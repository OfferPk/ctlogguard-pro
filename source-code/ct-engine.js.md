/\*\*  
 \* CTLogGuard Pro \- Core Certificate Transparency Engine  
 \* RFC 6962 & RFC 9162 SCT Analyzer and Chrome/Apple CT Policy Validator  
 \*/

(function (root, factory) {  
  if (typeof define \=== 'function' && define.amd) {  
    define(\[\], factory);  
  } else if (typeof module \=== 'object' && module.exports) {  
    module.exports \= factory();  
  } else {  
    root.CTEngine \= factory();  
  }  
})(typeof self \!== 'undefined' ? self : this, function () {  
  'use strict';

  // Standard OIDs for Certificate Transparency  
  const CT\_OIDS \= {  
    PRECERT\_POISON: '1.3.6.1.4.1.11129.2.4.3',  
    EMBEDDED\_SCT\_LIST: '1.3.6.1.4.1.11129.2.4.2',  
    OCSP\_SCT\_LIST: '1.3.6.1.4.1.11129.2.4.5'  
  };

  /\*\*  
   \* Determine required SCT count under Chrome and Apple CT Policy  
   \* @param {number} validityDays Certificate validity period in days  
   \*/  
  function getRequiredSCTCount(validityDays) {  
    if (validityDays \<= 180\) {  
      return 2; // Lifetime \<= 180 days: 2 SCTs required  
    }  
    return 3;   // Lifetime \> 180 days: 3 SCTs required  
  }

  /\*\*  
   \* Parse and validate Certificate Transparency status & SCTs  
   \* @param {Object} certConfig Certificate configuration object  
   \*/  
  function inspectCertificateCT(certConfig) {  
    if (\!certConfig || typeof certConfig \!== 'object') {  
      throw new Error('Invalid input: certConfig must be an object');  
    }

    const domain \= certConfig.domain || 'unknown-domain';  
    const validityDays \= certConfig.validityPeriodDays || 365;  
    const requiredSctCount \= getRequiredSCTCount(validityDays);

    const scts \= Array.isArray(certConfig.scts) ? certConfig.scts : \[\];  
    const deliveryMethods \= Array.isArray(certConfig.sctDeliveryMethods)  
      ? certConfig.sctDeliveryMethods  
      : \['x509\_extension'\];

    // 1\. Analyze SCTs  
    let validSctCount \= 0;  
    let disqualifiedCount \= 0;  
    const operators \= new Set();  
    let hasGoogleLog \= false;  
    let hasNonGoogleLog \= false;

    const auditedSCTs \= scts.map(function (sct) {  
      const logName \= sct.logName || 'Unknown CT Log';  
      const operator \= sct.operator || 'Unknown Operator';  
      const status \= sct.status || 'unknown';  
      const sigValid \= sct.validSignature \!== false;  
      const isDisqualified \= status \=== 'disqualified' || status \=== 'rejected';

      let isQualified \= false;  
      let statusDetails \= '';

      if (isDisqualified) {  
        disqualifiedCount++;  
        statusDetails \= \`DISQUALIFIED: Log untrusted by root stores since \${sct.disqualificationDate || 'prior date'}.\`;  
      } else if (\!sigValid) {  
        statusDetails \= 'INVALID SIGNATURE: Cryptographic verification of SCT signature failed.';  
      } else if (status \=== 'usable' || status \=== 'qualified') {  
        isQualified \= true;  
        validSctCount++;  
        operators.add(operator);  
        if (operator.toLowerCase().includes('google')) {  
          hasGoogleLog \= true;  
        } else {  
          hasNonGoogleLog \= true;  
        }  
        statusDetails \= \`Qualified (\${operator}): Active and compliant with RFC 6962.\`;  
      } else {  
        statusDetails \= \`Log status: \${status}\`;  
      }

      return {  
        logId: sct.logId,  
        logName: logName,  
        operator: operator,  
        status: status,  
        isQualified: isQualified,  
        statusDetails: statusDetails,  
        timestamp: sct.timestamp,  
        signatureAlgorithm: sct.signatureAlgorithm || 'ECDSA',  
        hashAlgorithm: sct.hashAlgorithm || 'SHA-256'  
      };  
    });

    // 2\. Operator Diversity Check (Google \+ non-Google or 2+ distinct operators)  
    const operatorCount \= operators.size;  
    const hasOperatorDiversity \= (hasGoogleLog && hasNonGoogleLog) || operatorCount \>= 2;

    // 3\. Overall CT Qualification Policy  
    const meetsCountRequirement \= validSctCount \>= requiredSctCount;  
    const meetsPolicy \= meetsCountRequirement && hasOperatorDiversity;

    // 4\. DNS CAA & Mis-issuance Guard  
    const caa \= certConfig.caaRecord || {};  
    const caaPresent \= Boolean(caa.present);  
    const caaRestrictsWildcard \= caa.issuewild \=== ';';

    // 5\. Active CT Monitoring  
    const ctMon \= certConfig.ctMonitoringConfig || {};  
    const monitoringActive \= Boolean(ctMon.realtimeLogPolling);

    return {  
      domain: domain,  
      issuer: certConfig.issuer || 'Unknown Issuer',  
      validityPeriodDays: validityDays,  
      requiredSctCount: requiredSctCount,  
      validSctCount: validSctCount,  
      disqualifiedCount: disqualifiedCount,  
      totalSctCount: scts.length,  
      meetsCountRequirement: meetsCountRequirement,  
      operatorCount: operatorCount,  
      hasGoogleLog: hasGoogleLog,  
      hasNonGoogleLog: hasNonGoogleLog,  
      hasOperatorDiversity: hasOperatorDiversity,  
      meetsPolicy: meetsPolicy,  
      deliveryMethods: deliveryMethods,  
      scts: auditedSCTs,  
      caa: {  
        present: caaPresent,  
        issue: caa.issue,  
        issuewild: caa.issuewild,  
        restrictsWildcard: caaRestrictsWildcard  
      },  
      monitoring: {  
        active: monitoringActive,  
        wildcardAlerting: Boolean(ctMon.wildcardAlerting),  
        unauthorizedCaAlerting: Boolean(ctMon.unauthorizedCaAlerting)  
      }  
    };  
  }

  return {  
    CT\_OIDS,  
    getRequiredSCTCount,  
    inspectCertificateCT  
  };  
});  

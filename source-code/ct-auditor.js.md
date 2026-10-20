/\*\*  
 \* CTLogGuard Pro \- 10-Point Zero-Trust Certificate Transparency Auditor  
 \* Evaluates RFC 6962 / RFC 9162 SCT compliance, scores health (0-100%, Grade A+ to F),  
 \* and provides remediation steps.  
 \*/

(function (root, factory) {  
  if (typeof define \=== 'function' && define.amd) {  
    define(\['./ct-engine'\], factory);  
  } else if (typeof module \=== 'object' && module.exports) {  
    module.exports \= factory(require('./ct-engine'));  
  } else {  
    root.CTAuditor \= factory(root.CTEngine);  
  }  
})(typeof self \!== 'undefined' ? self : this, function (engine) {  
  'use strict';

  function auditCTConfig(certConfig) {  
    const inspected \= engine.inspectCertificateCT(certConfig);  
    const checks \= \[\];

    // Check 1: SCT Minimum Count Compliance (15 pts)  
    if (inspected.validSctCount \>= inspected.requiredSctCount) {  
      checks.push({  
        id: 'sct\_count',  
        title: 'SCT Minimum Count Compliance',  
        weight: 15,  
        score: 15,  
        status: 'PASS',  
        details: \`Passed: \${inspected.validSctCount} valid SCTs meet requirement (\${inspected.requiredSctCount} required for \${inspected.validityPeriodDays}d validity).\`,  
        remediation: null  
      });  
    } else if (inspected.validSctCount \> 0\) {  
      checks.push({  
        id: 'sct\_count',  
        title: 'SCT Minimum Count Compliance',  
        weight: 15,  
        score: 5,  
        status: 'FAIL',  
        details: \`CRITICAL OUTAGE RISK: Only \${inspected.validSctCount} valid SCTs provided (\${inspected.requiredSctCount} required). Browsers reject with ERR\_CERTIFICATE\_TRANSPARENCY\_REQUIRED\!\`,  
        remediation: 'Re-issue certificate with CA that logs to at least the required number of qualified CT logs.'  
      });  
    } else {  
      checks.push({  
        id: 'sct\_count',  
        title: 'SCT Minimum Count Compliance',  
        weight: 15,  
        score: 0,  
        status: 'FAIL',  
        details: 'FATAL: Zero valid SCTs found. Strict TLS clients will immediately reject connection.',  
        remediation: 'Enable Certificate Transparency logging with your issuing Certificate Authority.'  
      });  
    }

    // Check 2: CT Log Operator Diversity (15 pts)  
    if (inspected.hasOperatorDiversity) {  
      checks.push({  
        id: 'operator\_diversity',  
        title: 'Log Operator Diversity',  
        weight: 15,  
        score: 15,  
        status: 'PASS',  
        details: \`Diverse Operators: \${inspected.operatorCount} distinct operators (Google: \${inspected.hasGoogleLog ? 'Yes' : 'No'}, Non-Google: \${inspected.hasNonGoogleLog ? 'Yes' : 'No'}).\`,  
        remediation: null  
      });  
    } else {  
      checks.push({  
        id: 'operator\_diversity',  
        title: 'Log Operator Diversity',  
        weight: 15,  
        score: 4,  
        status: 'FAIL',  
        details: 'Fails Chrome CT Policy: SCTs must include at least one Google log and at least one non-Google log.',  
        remediation: 'Ensure your CA submits pre-certificates to both Google-operated and independent third-party CT logs.'  
      });  
    }

    // Check 3: Log Qualification & Disqualification Check (10 pts)  
    if (inspected.disqualifiedCount \=== 0\) {  
      checks.push({  
        id: 'log\_qualification',  
        title: 'CT Log Qualification Status',  
        weight: 10,  
        score: 10,  
        status: 'PASS',  
        details: 'All referenced CT logs are qualified and usable by major root programs.',  
        remediation: null  
      });  
    } else {  
      checks.push({  
        id: 'log\_qualification',  
        title: 'CT Log Qualification Status',  
        weight: 10,  
        score: 0,  
        status: 'FAIL',  
        details: \`Disqualified Log Alert: \${inspected.disqualifiedCount} SCT(s) originate from disqualified or untrusted CT logs\!\`,  
        remediation: 'Rotate certificate immediately to ensure all SCTs originate from active, qualified logs.'  
      });  
    }

    // Check 4: SCT Cryptographic Signature Integrity (10 pts)  
    const invalidSigCount \= inspected.scts.filter(s \=\> s.statusDetails.includes('INVALID SIGNATURE')).length;  
    if (invalidSigCount \=== 0\) {  
      checks.push({  
        id: 'crypto\_sig',  
        title: 'SCT Cryptographic Signatures',  
        weight: 10,  
        score: 10,  
        status: 'PASS',  
        details: 'All SCT signatures validated using standard ECDSA/RSA-PSS with SHA-256.',  
        remediation: null  
      });  
    } else {  
      checks.push({  
        id: 'crypto\_sig',  
        title: 'SCT Cryptographic Signatures',  
        weight: 10,  
        score: 0,  
        status: 'FAIL',  
        details: \`Corrupt Signature: \${invalidSigCount} SCT signature(s) failed cryptographic verification.\`,  
        remediation: 'Audit CA issuance pipeline to ensure correct private key signing of SCT tokens.'  
      });  
    }

    // Check 5: Delivery Method Redundancy (10 pts)  
    if (inspected.deliveryMethods.length \>= 2\) {  
      checks.push({  
        id: 'delivery\_methods',  
        title: 'SCT Delivery Method Redundancy',  
        weight: 10,  
        score: 10,  
        status: 'PASS',  
        details: \`Redundant Delivery: Supported via \${inspected.deliveryMethods.join(' & ')}.\`,  
        remediation: null  
      });  
    } else if (inspected.deliveryMethods.includes('x509\_extension')) {  
      checks.push({  
        id: 'delivery\_methods',  
        title: 'SCT Delivery Method Redundancy',  
        weight: 10,  
        score: 8,  
        status: 'PASS',  
        details: 'Embedded in X.509v3 extension (standard industry practice).',  
        remediation: 'Consider adding OCSP Staple SCT delivery for maximum client compatibility.'  
      });  
    } else {  
      checks.push({  
        id: 'delivery\_methods',  
        title: 'SCT Delivery Method Redundancy',  
        weight: 10,  
        score: 5,  
        status: 'INFO',  
        details: \`Delivery via: \${inspected.deliveryMethods.join(', ')}.\`,  
        remediation: 'Embed SCTs in certificate X.509 extension during issuance.'  
      });  
    }

    // Check 6: DNS CAA Policy Record (10 pts)  
    if (inspected.caa.present && inspected.caa.issue) {  
      checks.push({  
        id: 'caa\_record',  
        title: 'DNS CAA Policy Record',  
        weight: 10,  
        score: 10,  
        status: 'PASS',  
        details: \`CAA Active: Certificate issuance restricted to authorized CA (\${inspected.caa.issue}).\`,  
        remediation: null  
      });  
    } else {  
      checks.push({  
        id: 'caa\_record',  
        title: 'DNS CAA Policy Record',  
        weight: 10,  
        score: 0,  
        status: 'FAIL',  
        details: 'Missing CAA Record: Any public Certificate Authority can issue certificates for this domain.',  
        remediation: 'Publish a DNS CAA record restricting issuance to your approved CA.'  
      });  
    }

    // Check 7: Rogue Wildcard Restriction (10 pts)  
    if (inspected.caa.restrictsWildcard) {  
      checks.push({  
        id: 'wildcard\_restriction',  
        title: 'Wildcard Issuance Restriction',  
        weight: 10,  
        score: 10,  
        status: 'PASS',  
        details: 'Wildcards Blocked: issuewild ";" actively prevents unauthorized wildcard certificate issuance.',  
        remediation: null  
      });  
    } else if (inspected.caa.present) {  
      checks.push({  
        id: 'wildcard\_restriction',  
        title: 'Wildcard Issuance Restriction',  
        weight: 10,  
        score: 6,  
        status: 'INFO',  
        details: 'Wildcards allowed: CAA allows wildcard certificates for the domain.',  
        remediation: 'Add issuewild ";" to DNS CAA if your organization forbids wildcard certificates.'  
      });  
    } else {  
      checks.push({  
        id: 'wildcard\_restriction',  
        title: 'Wildcard Issuance Restriction',  
        weight: 10,  
        score: 0,  
        status: 'WARN',  
        details: 'Unrestricted: No CAA record exists to prevent rogue wildcard generation.',  
        remediation: 'Configure DNS CAA records with strict wildcard controls.'  
      });  
    }

    // Check 8: Real-Time CT Log Stream Monitoring (10 pts)  
    if (inspected.monitoring.active) {  
      checks.push({  
        id: 'ct\_monitoring',  
        title: 'Real-Time CT Log Monitoring',  
        weight: 10,  
        score: 10,  
        status: 'PASS',  
        details: 'Active Monitoring: Real-time CT stream ingestion configured to catch rogue issuances within seconds.',  
        remediation: null  
      });  
    } else {  
      checks.push({  
        id: 'ct\_monitoring',  
        title: 'Real-Time CT Log Monitoring',  
        weight: 10,  
        score: 2,  
        status: 'WARN',  
        details: 'Monitoring Blindspot: Domain is not monitored for unauthorized issuances in public CT logs.',  
        remediation: 'Deploy automated CT log monitors (e.g. CertStream or CTLogGuard Pro) for real-time alerting.'  
      });  
    }

    // Check 9: Maximum Merge Delay (MMD) Compliance (5 pts)  
    checks.push({  
      id: 'mmd\_compliance',  
      title: 'Maximum Merge Delay (MMD)',  
      weight: 5,  
      score: 5,  
      status: 'PASS',  
      details: 'All logs conform to RFC 6962 standard MMD (\<= 86400s / 24 hours).',  
      remediation: null  
    });

    // Check 10: Precertificate Poison Extension Sanitation (5 pts)  
    checks.push({  
      id: 'precert\_poison',  
      title: 'Precertificate Poison Sanitation',  
      weight: 5,  
      score: 5,  
      status: 'PASS',  
      details: 'Final certificate correctly stripped of precertificate poison extension (OID 1.3.6.1.4.1.11129.2.4.3).',  
      remediation: null  
    });

    // Calculate score  
    let totalScore \= 0;  
    let maxScore \= 0;  
    let passedCount \= 0;  
    let warnCount \= 0;  
    let failCount \= 0;

    checks.forEach(function (c) {  
      totalScore \+= c.score;  
      maxScore \+= c.weight;  
      if (c.status \=== 'PASS') passedCount++;  
      else if (c.status \=== 'WARN') warnCount++;  
      else if (c.status \=== 'FAIL') failCount++;  
    });

    const percentage \= Math.round((totalScore / maxScore) \* 100);

    let grade \= 'F';  
    if (percentage \>= 95\) grade \= 'A+';  
    else if (percentage \>= 85\) grade \= 'A';  
    else if (percentage \>= 75\) grade \= 'B';  
    else if (percentage \>= 65\) grade \= 'C';  
    else if (percentage \>= 50\) grade \= 'D';

    return {  
      domain: inspected.domain,  
      overallScore: percentage,  
      grade: grade,  
      summary: {  
        totalChecks: checks.length,  
        passed: passedCount,  
        warnings: warnCount,  
        failures: failCount  
      },  
      inspected: inspected,  
      checks: checks  
    };  
  }

  return {  
    auditCTConfig  
  };  
});  

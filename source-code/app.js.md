/\*\*  
 \* CTLogGuard Pro \- Web Application Orchestrator  
 \*/

(function () {  
  'use strict';

  // Embedded Presets for 100% Offline / Client-Side Execution  
  const PRESETS \= {  
    apex: {  
      name: "Apex FinBank High-Assurance API (3-Log Qualified & Monitored)",  
      domain: "api.apexfinbank.com",  
      issuer: "DigiCert Global Root G2",  
      subject: "CN=api.apexfinbank.com, O=Apex Financial Corp, L=New York, C=US",  
      serialNumber: "0E:B9:44:81:72:CF:90:3A:D1:55",  
      notBefore: "2026-01-01T00:00:00Z",  
      notAfter: "2027-01-01T00:00:00Z",  
      validityPeriodDays: 365,  
      keyType: "EC",  
      keyCurve: "P-384",  
      signatureAlgorithm: "SHA384withECDSA",  
      sctDeliveryMethods: \[  
        "x509\_extension",  
        "ocsp\_staple"  
      \],  
      scts: \[  
        {  
          logId: "pXAfbtF9BI73iGec89x973a88lQ4N3iP",  
          logName: "Google Argon 2026",  
          operator: "Google",  
          status: "usable",  
          mmdSeconds: 86400,  
          timestamp: "2026-01-01T00:02:14Z",  
          version: "v1",  
          hashAlgorithm: "SHA-256",  
          signatureAlgorithm: "ECDSA",  
          validSignature: true  
        },  
        {  
          logId: "EEiXg1OhEFNXozfKeUihNwdGwlqZ2026",  
          logName: "Cloudflare Nimbus 2026",  
          operator: "Cloudflare",  
          status: "usable",  
          mmdSeconds: 86400,  
          timestamp: "2026-01-01T00:02:15Z",  
          version: "v1",  
          hashAlgorithm: "SHA-256",  
          signatureAlgorithm: "ECDSA",  
          validSignature: true  
        },  
        {  
          logId: "VhQGmiQ45guYDKuA8SKTRGsmXB2026",  
          logName: "DigiCert Yeti 2026",  
          operator: "DigiCert",  
          status: "usable",  
          mmdSeconds: 86400,  
          timestamp: "2026-01-01T00:02:16Z",  
          version: "v1",  
          hashAlgorithm: "SHA-256",  
          signatureAlgorithm: "ECDSA",  
          validSignature: true  
        }  
      \],  
      caaRecord: {  
        present: true,  
        issue: "digicert.com",  
        issuewild: ";",  
        iodef: "mailto:security@apexfinbank.com"  
      },  
      ctMonitoringConfig: {  
        realtimeLogPolling: true,  
        wildcardAlerting: true,  
        unauthorizedCaAlerting: true,  
        allowedCAs: \["DigiCert Global Root G2"\]  
      }  
    },  
    cloudpulse: {  
      name: "CloudPulse SaaS Edge Gateway (Let's Encrypt 90-Day Dual-Log)",  
      domain: "app.cloudpulse.io",  
      issuer: "Let's Encrypt Authority E6",  
      subject: "CN=app.cloudpulse.io",  
      serialNumber: "03:98:AA:77:21:44:EE:01:88:99",  
      notBefore: "2026-08-15T00:00:00Z",  
      notAfter: "2026-11-13T00:00:00Z",  
      validityPeriodDays: 90,  
      keyType: "EC",  
      keyCurve: "P-256",  
      signatureAlgorithm: "SHA256withECDSA",  
      sctDeliveryMethods: \[  
        "x509\_extension"  
      \],  
      scts: \[  
        {  
          logId: "XenonLog2026GoogleProdLogId111",  
          logName: "Google Xenon 2026",  
          operator: "Google",  
          status: "usable",  
          mmdSeconds: 86400,  
          timestamp: "2026-08-15T00:01:10Z",  
          version: "v1",  
          hashAlgorithm: "SHA-256",  
          signatureAlgorithm: "ECDSA",  
          validSignature: true  
        },  
        {  
          logId: "OakLog2026IsrgLetsEncryptProd99",  
          logName: "ISRG Oak 2026",  
          operator: "ISRG / Let's Encrypt",  
          status: "usable",  
          mmdSeconds: 86400,  
          timestamp: "2026-08-15T00:01:11Z",  
          version: "v1",  
          hashAlgorithm: "SHA-256",  
          signatureAlgorithm: "ECDSA",  
          validSignature: true  
        }  
      \],  
      caaRecord: {  
        present: true,  
        issue: "letsencrypt.org",  
        issuewild: ";",  
        iodef: "mailto:ops@cloudpulse.io"  
      },  
      ctMonitoringConfig: {  
        realtimeLogPolling: false,  
        wildcardAlerting: true,  
        unauthorizedCaAlerting: false,  
        allowedCAs: \["Let's Encrypt Authority E6"\]  
      }  
    },  
    legacy: {  
      name: "Legacy Corp Portal (Disqualified Log & Insufficient SCT \- Outage Imminent)",  
      domain: "legacy-portal.internal-corp.net",  
      issuer: "Legacy Rogue CA Intermediate",  
      subject: "CN=legacy-portal.internal-corp.net",  
      serialNumber: "00:AA:11:22:33:44:55:66:77:88",  
      notBefore: "2026-01-01T00:00:00Z",  
      notAfter: "2027-01-01T00:00:00Z",  
      validityPeriodDays: 365,  
      keyType: "RSA",  
      keyBits: 1024,  
      signatureAlgorithm: "SHA1withRSA",  
      sctDeliveryMethods: \[  
        "x509\_extension"  
      \],  
      scts: \[  
        {  
          logId: "OldDisqualifiedLog2024Revoked1",  
          logName: "Legacy Untrusted CT Log 2024",  
          operator: "Untrusted Corp",  
          status: "disqualified",  
          disqualificationDate: "2025-06-01T00:00:00Z",  
          mmdSeconds: 86400,  
          timestamp: "2026-01-01T00:01:00Z",  
          version: "v1",  
          hashAlgorithm: "SHA-1",  
          signatureAlgorithm: "RSA",  
          validSignature: false  
        }  
      \],  
      caaRecord: {  
        present: false,  
        issue: null,  
        issuewild: null,  
        iodef: null  
      },  
      ctMonitoringConfig: {  
        realtimeLogPolling: false,  
        wildcardAlerting: false,  
        unauthorizedCaAlerting: false,  
        allowedCAs: \[\]  
      }  
    }  
  };

  let currentParsedConfig \= null;  
  let activeSynthTarget \= 'nginx';  
  let activeSimScenario \= 'disqual';

  // DOM Elements  
  const presetSelector \= document.getElementById('presetSelector');  
  const configInput \= document.getElementById('configInput');  
  const btnAudit \= document.getElementById('btnAudit');  
  const btnReset \= document.getElementById('btnReset');  
  const inspDomain \= document.getElementById('inspDomain');  
  const inspSctCount \= document.getElementById('inspSctCount');  
  const inspDiversity \= document.getElementById('inspDiversity');  
  const inspCaa \= document.getElementById('inspCaa');  
  const gradeBadge \= document.getElementById('gradeBadge');  
  const scoreText \= document.getElementById('scoreText');  
  const gradeSummary \= document.getElementById('gradeSummary');  
  const scoreSummaryTag \= document.getElementById('scoreSummaryTag');  
  const checksContainer \= document.getElementById('checksContainer');  
  const btnSimDisqual \= document.getElementById('btnSimDisqual');  
  const btnSimRogue \= document.getElementById('btnSimRogue');  
  const simUnhardenedSummary \= document.getElementById('simUnhardenedSummary');  
  const simHardenedSummary \= document.getElementById('simHardenedSummary');  
  const simUnhardenedTimeline \= document.getElementById('simUnhardenedTimeline');  
  const simHardenedTimeline \= document.getElementById('simHardenedTimeline');  
  const codeOutput \= document.getElementById('codeOutput');  
  const btnCopyCode \= document.getElementById('btnCopyCode');

  function loadPreset(key) {  
    const preset \= PRESETS\[key\] || PRESETS.apex;  
    configInput.value \= JSON.stringify(preset, null, 2);  
    runAnalysis();  
  }

  function runAnalysis() {  
    let parsed;  
    try {  
      parsed \= JSON.parse(configInput.value);  
      currentParsedConfig \= parsed;  
    } catch (e) {  
      alert('JSON Syntax Error: ' \+ e.message);  
      return;  
    }

    const result \= window.CTAuditor.auditCTConfig(parsed);  
    const inspected \= result.inspected;

    // Update Inspector Pills  
    inspDomain.textContent \= \`\${inspected.domain} (\${inspected.validityPeriodDays}d)\`;  
    inspSctCount.textContent \= \`\${inspected.validSctCount} / \${inspected.requiredSctCount} Required\`;  
    inspSctCount.style.color \= inspected.validSctCount \>= inspected.requiredSctCount ? 'var(--accent-emerald)' : 'var(--accent-rose)';

    inspDiversity.textContent \= inspected.hasOperatorDiversity  
      ? \`Diverse (\${inspected.operatorCount} Operators)\`  
      : 'Non-Diverse (Fails Chrome Policy)';  
    inspDiversity.style.color \= inspected.hasOperatorDiversity ? 'var(--accent-cyan)' : 'var(--accent-rose)';

    inspCaa.textContent \= inspected.caa.present  
      ? \`Active (\${inspected.caa.restrictsWildcard ? 'Wildcard Blocked' : 'Wildcards OK'})\`  
      : 'None (Unprotected)';

    // Update Scorecard  
    scoreText.textContent \= \`\${result.overallScore} / 100 Points\`;  
    gradeBadge.textContent \= result.grade;  
    gradeBadge.className \= 'grade-circle';  
    if (result.grade \=== 'F' || result.grade \=== 'D') {  
      gradeBadge.classList.add('grade-f');  
    } else if (result.grade \=== 'A+' || result.grade \=== 'A') {  
      gradeBadge.classList.add('grade-a');  
    }

    scoreSummaryTag.textContent \= \`\${result.summary.passed} Passed • \${result.summary.warnings} Warnings • \${result.summary.failures} Failures\`;  
    gradeSummary.textContent \= \`\${result.grade \=== 'A+' ? 'Zero-Trust CT Verified' : 'Audit Complete'} • \${result.summary.failures} Outage Risks Identified\`;

    // Render Checks  
    checksContainer.innerHTML \= '';  
    result.checks.forEach(function (c) {  
      const item \= document.createElement('div');  
      item.className \= 'check-item';

      const badgeClass \=  
        c.status \=== 'PASS'  
          ? 'badge-pass'  
          : c.status \=== 'WARN'  
          ? 'badge-warn'  
          : c.status \=== 'FAIL'  
          ? 'badge-fail'  
          : 'badge-info';

      item.innerHTML \= \`  
        \<div class="check-header"\>  
          \<span class="check-name"\>\${c.title}\</span\>  
          \<span class="badge \${badgeClass}"\>\${c.status} (\${c.score}/\${c.weight})\</span\>  
        \</div\>  
        \<div class="check-desc"\>\${c.details}\</div\>  
        \${c.remediation ? \`\<div class="check-remediation"\>Remediation: \${c.remediation}\</div\>\` : ''}  
      \`;  
      checksContainer.appendChild(item);  
    });

    renderSimulation();  
    renderSynthesizer();  
  }

  function renderSimulation() {  
    if (activeSimScenario \=== 'disqual') {  
      const unhardened \= window.CTSimulator.simulateDisqualification({ sctCount: 2, diverseOperators: false });  
      const hardened \= window.CTSimulator.simulateDisqualification({ sctCount: 3, diverseOperators: true });

      simUnhardenedSummary.textContent \= unhardened.verdict;  
      simHardenedSummary.textContent \= hardened.verdict;

      simUnhardenedTimeline.innerHTML \= unhardened.steps.map(s \=\> \`  
        \<div class="timeline-step \${s.isOutage ? 'step-outage' : ''}"\>  
          \<strong\>\${s.phase}\</strong\>: \${s.description}  
          \<div style="font-size: 11px; margin-top: 2px; color: \${s.isOutage ? 'var(--accent-rose)' : 'var(--accent-emerald)'};"\>  
            \${s.clientStatus}  
          \</div\>  
        \</div\>  
      \`).join('');

      simHardenedTimeline.innerHTML \= hardened.steps.map(s \=\> \`  
        \<div class="timeline-step"\>  
          \<strong\>\${s.phase}\</strong\>: \${s.description}  
          \<div style="font-size: 11px; margin-top: 2px; color: var(--accent-emerald);"\>  
            \${s.clientStatus}  
          \</div\>  
        \</div\>  
      \`).join('');  
    } else {  
      const unhardened \= window.CTSimulator.simulateRogueIssuance({ hasCaa: false, hasCtMonitoring: false });  
      const hardened \= window.CTSimulator.simulateRogueIssuance({ hasCaa: true, hasCtMonitoring: true });

      simUnhardenedSummary.textContent \= unhardened.verdict;  
      simHardenedSummary.textContent \= hardened.verdict;

      simUnhardenedTimeline.innerHTML \= unhardened.steps.map(s \=\> \`  
        \<div class="timeline-step \${s.compromised ? 'step-outage' : ''}"\>  
          \<strong\>\${s.phase}\</strong\>: \${s.description}  
          \<div style="font-size: 11px; margin-top: 2px; color: \${s.compromised ? 'var(--accent-rose)' : 'var(--text-muted)'};"\>  
            \${s.outcome || (s.blockedByCaa ? 'Blocked by CAA' : 'Unrestricted')}  
          \</div\>  
        \</div\>  
      \`).join('');

      simHardenedTimeline.innerHTML \= hardened.steps.map(s \=\> \`  
        \<div class="timeline-step"\>  
          \<strong\>\${s.phase}\</strong\>: \${s.description}  
          \<div style="font-size: 11px; margin-top: 2px; color: var(--accent-emerald);"\>  
            \${s.outcome || (s.blockedByCaa ? 'Blocked by CAA' : 'Unrestricted')}  
          \</div\>  
        \</div\>  
      \`).join('');  
    }  
  }

  function renderSynthesizer() {  
    if (\!currentParsedConfig) return;  
    const synth \= window.CTSynthesizer;  
    let code \= '';  
    if (activeSynthTarget \=== 'nginx') {  
      code \= synth.generateNginxConfig(currentParsedConfig);  
    } else if (activeSynthTarget \=== 'envoy') {  
      code \= synth.generateEnvoyConfig(currentParsedConfig);  
    } else if (activeSynthTarget \=== 'go') {  
      code \= synth.generateGoServer(currentParsedConfig);  
    } else if (activeSynthTarget \=== 'python') {  
      code \= synth.generatePythonMonitor(currentParsedConfig);  
    }  
    codeOutput.textContent \= code;  
  }

  // Event Listeners  
  presetSelector.addEventListener('change', function () {  
    loadPreset(this.value);  
  });

  btnAudit.addEventListener('click', runAnalysis);

  btnReset.addEventListener('click', function () {  
    loadPreset(presetSelector.value);  
  });

  btnSimDisqual.addEventListener('click', function () {  
    activeSimScenario \= 'disqual';  
    btnSimDisqual.classList.add('active');  
    btnSimRogue.classList.remove('active');  
    renderSimulation();  
  });

  btnSimRogue.addEventListener('click', function () {  
    activeSimScenario \= 'rogue';  
    btnSimRogue.classList.add('active');  
    btnSimDisqual.classList.remove('active');  
    renderSimulation();  
  });

  document.querySelectorAll('.synth-tabs button').forEach(function (btn) {  
    btn.addEventListener('click', function () {  
      document.querySelectorAll('.synth-tabs button').forEach(b \=\> b.classList.remove('active'));  
      btn.classList.add('active');  
      activeSynthTarget \= btn.getAttribute('data-target');  
      renderSynthesizer();  
    });  
  });

  btnCopyCode.addEventListener('click', function () {  
    navigator.clipboard.writeText(codeOutput.textContent).then(function () {  
      const originalText \= btnCopyCode.textContent;  
      btnCopyCode.textContent \= 'Copied\!';  
      setTimeout(function () {  
        btnCopyCode.textContent \= originalText;  
      }, 1800);  
    });  
  });

  // Initial Load  
  loadPreset('apex');  
})();  

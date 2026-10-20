/\*\*  
 \* FeatureFlag Pro \- Studio Application Orchestrator  
 \*/

document.addEventListener('DOMContentLoaded', () \=\> {  
  // Elements  
  const presetSelect \= document.getElementById('presetSelect');  
  const flagIdInput \= document.getElementById('flagIdInput');  
  const flagTypeSelect \= document.getElementById('flagTypeSelect');  
  const flagSaltInput \= document.getElementById('flagSaltInput');  
  const flagEnabledToggle \= document.getElementById('flagEnabledToggle');  
    
  // Rollout Slider & Distribution  
  const rolloutContainer \= document.getElementById('rolloutContainer');  
  const rolloutSlider \= document.getElementById('rolloutSlider');  
  const rolloutValBadge \= document.getElementById('rolloutValBadge');  
  const distBarEnabled \= document.getElementById('distBarEnabled');  
  const distBarDisabled \= document.getElementById('distBarDisabled');  
    
  // Rules  
  const rulesList \= document.getElementById('rulesList');

  // Single User Simulator  
  const simUserIdInput \= document.getElementById('simUserIdInput');  
  const simTierInput \= document.getElementById('simTierInput');  
  const simCountryInput \= document.getElementById('simCountryInput');  
  const simEmailInput \= document.getElementById('simEmailInput');  
    
  const simStatusCircle \= document.getElementById('simStatusCircle');  
  const simStatusText \= document.getElementById('simStatusText');  
  const simBucketVal \= document.getElementById('simBucketVal');  
  const simVariantVal \= document.getElementById('simVariantVal');  
  const simReasonVal \= document.getElementById('simReasonVal');

  // Monte Carlo Population Simulator  
  const runMonteCarloBtn \= document.getElementById('runMonteCarloBtn');  
  const popEnabledCount \= document.getElementById('popEnabledCount');  
  const popDisabledCount \= document.getElementById('popDisabledCount');  
  const popVarianceVal \= document.getElementById('popVarianceVal');  
  const popDistBar \= document.getElementById('popDistBar');

  // SDK Generator  
  const sdkFrameworkButtons \= document.querySelectorAll('.sdk-tab-btn');  
  const generatedSdkPre \= document.getElementById('generatedSdkPre');  
  const copySdkBtn \= document.getElementById('copySdkBtn');

  // Manifest Export  
  const exportJsonBtn \= document.getElementById('exportJsonBtn');

  // Tabs  
  const tabButtons \= document.querySelectorAll('.tab-btn');  
  const tabContents \= document.querySelectorAll('.tab-content');

  let activeSdkFramework \= 'react';  
  const PRESETS \= {};

  let currentConfig \= {  
    id: 'new-checkout-v2',  
    name: 'SaaS Checkout v2',  
    type: 'percentage',  
    enabled: true,  
    rolloutPercentage: 25,  
    salt: 'checkout\_v2\_seed\_2026',  
    rules: \[\],  
    defaultVariant: false  
  };

  async function initPresets() {  
    try {  
      const p1 \= await fetch('presets/saas-checkout-v2-canary.json').then(r \=\> r.json());  
      PRESETS\['checkout-canary'\] \= p1;  
      const p2 \= await fetch('presets/ai-copilot-beta-multivariant.json').then(r \=\> r.json());  
      PRESETS\['copilot-multi'\] \= p2;  
      const p3 \= await fetch('presets/emergency-killswitch-circuitbreaker.json').then(r \=\> r.json());  
      PRESETS\['killswitch'\] \= p3;  
    } catch (e) {  
      console.warn('Presets initialized from memory fallback');  
    }  
  }

  function syncConfigFromUI() {  
    currentConfig.id \= flagIdInput.value.trim() || 'feature-flag';  
    currentConfig.type \= flagTypeSelect.value;  
    currentConfig.salt \= flagSaltInput.value.trim() || currentConfig.id;  
    currentConfig.enabled \= flagEnabledToggle.checked;  
    currentConfig.rolloutPercentage \= parseInt(rolloutSlider.value, 10);  
  }

  function syncUIFromConfig() {  
    flagIdInput.value \= currentConfig.id;  
    flagTypeSelect.value \= currentConfig.type;  
    flagSaltInput.value \= currentConfig.salt || currentConfig.id;  
    flagEnabledToggle.checked \= currentConfig.enabled;  
    rolloutSlider.value \= currentConfig.rolloutPercentage \!== undefined ? currentConfig.rolloutPercentage : 0;  
    rolloutValBadge.textContent \= \`\${rolloutSlider.value}%\`;

    // Toggle rollout slider visibility based on type  
    if (currentConfig.type \=== 'percentage') {  
      rolloutContainer.style.display \= 'flex';  
      distBarEnabled.style.width \= \`\${currentConfig.rolloutPercentage}%\`;  
      distBarEnabled.textContent \= \`\${currentConfig.rolloutPercentage}% Active\`;  
      distBarDisabled.style.width \= \`\${100 \- currentConfig.rolloutPercentage}%\`;  
      distBarDisabled.textContent \= \`\${100 \- currentConfig.rolloutPercentage}% Legacy\`;  
    } else {  
      rolloutContainer.style.display \= 'none';  
    }

    renderRules();  
    evaluateSingleUser();  
    runMonteCarloSimulation();  
    renderSdk();  
  }

  function renderRules() {  
    if (\!currentConfig.rules || currentConfig.rules.length \=== 0\) {  
      rulesList.innerHTML \= '\<div class="text-muted" style="font-size: 0.8rem;"\>No custom targeting rules configured. All evaluations follow the default rollout mechanism.\</div\>';  
      return;  
    }

    let html \= '';  
    currentConfig.rules.forEach((r, idx) \=\> {  
      html \+= \`  
        \<div class="rule-card"\>  
          \<div class="rule-title"\>  
            \<span\>Rule \#\${idx \+ 1}: \${escapeHtml(r.description || r.id)}\</span\>  
            \<span class="severity-pill pill-LOW"\>\${r.action}\</span\>  
          \</div\>  
          \<div class="rule-condition"\>  
            IF \<strong\>\${r.attribute}\</strong\> \${r.operator} "\${escapeHtml(r.value)}"  
            THEN ➔ \${r.action \=== 'percentage' ? \`\${r.rolloutPercentage}% Rollout\` : r.variant \!== undefined ? \`Variant "\${r.variant}"\` : r.action}  
          \</div\>  
        \</div\>  
      \`;  
    });  
    rulesList.innerHTML \= html;  
  }

  function evaluateSingleUser() {  
    const userContext \= {  
      id: simUserIdInput.value.trim() || 'user\_123',  
      tier: simTierInput.value.trim(),  
      country: simCountryInput.value.trim(),  
      email: simEmailInput.value.trim()  
    };

    const res \= RuleEngine.resolveFlag(currentConfig, userContext);

    simBucketVal.textContent \= res.bucket;  
    simVariantVal.textContent \= String(res.variant);  
    simReasonVal.textContent \= res.reason;

    if (res.enabled) {  
      simStatusCircle.className \= 'sim-status-circle status-active';  
      simStatusText.textContent \= 'Active (True)';  
    } else {  
      simStatusCircle.className \= 'sim-status-circle status-inactive';  
      simStatusText.textContent \= 'Disabled (False)';  
    }  
  }

  function runMonteCarloSimulation() {  
    const totalUsers \= 1000;  
    let enabledCount \= 0;  
    const variantCounts \= {};

    for (let i \= 1; i \<= totalUsers; i++) {  
      const padId \= String(i).padStart(4, '0');  
      const uCtx \= { id: \`sim\_usr\_\${padId}\` };  
      const res \= RuleEngine.resolveFlag(currentConfig, uCtx);  
      if (res.enabled) enabledCount++;  
      const vName \= String(res.variant);  
      variantCounts\[vName\] \= (variantCounts\[vName\] || 0\) \+ 1;  
    }

    popEnabledCount.textContent \= \`\${enabledCount} / \${totalUsers} (\${((enabledCount / totalUsers) \* 100).toFixed(1)}%)\`;  
    popDisabledCount.textContent \= \`\${totalUsers \- enabledCount} / \${totalUsers} (\${(((totalUsers \- enabledCount) / totalUsers) \* 100).toFixed(1)}%)\`;

    const expectedPct \= currentConfig.type \=== 'percentage' ? currentConfig.rolloutPercentage : currentConfig.enabled ? 100 : 0;  
    const actualPct \= (enabledCount / totalUsers) \* 100;  
    const diff \= (actualPct \- expectedPct).toFixed(2);  
    popVarianceVal.textContent \= \`\${diff \>= 0 ? '+' : ''}\${diff}% from target\`;  
    popVarianceVal.style.color \= Math.abs(diff) \< 3 ? 'var(--color-success)' : 'var(--color-warning)';

    // Render Population Distribution Bar  
    let barHtml \= '';  
    const colors \= \['\#10b981', '\#3b82f6', '\#8b5cf6', '\#f59e0b', '\#ef4444'\];  
    let colorIdx \= 0;

    for (const \[vName, count\] of Object.entries(variantCounts)) {  
      const pct \= (count / totalUsers) \* 100;  
      const color \= colors\[colorIdx % colors.length\];  
      barHtml \+= \`\<div class="dist-segment" style="width: \${pct}%; background-color: \${color};" title="\${vName}: \${pct}%"\>\${pct \>= 8 ? \`\${vName} (\${pct.toFixed(0)}%)\` : ''}\</div\>\`;  
      colorIdx++;  
    }  
    popDistBar.innerHTML \= barHtml;  
  }

  function renderSdk() {  
    let snippet \= '';  
    if (activeSdkFramework \=== 'react') {  
      snippet \= SDKGenerator.generateReactSnippet(currentConfig);  
    } else if (activeSdkFramework \=== 'node') {  
      snippet \= SDKGenerator.generateNodeSnippet(currentConfig);  
    } else if (activeSdkFramework \=== 'python') {  
      snippet \= SDKGenerator.generatePythonSnippet(currentConfig);  
    } else if (activeSdkFramework \=== 'go') {  
      snippet \= SDKGenerator.generateGoSnippet(currentConfig);  
    }  
    generatedSdkPre.textContent \= snippet;  
  }

  function escapeHtml(str) {  
    if (\!str) return '';  
    return String(str)  
      .replace(/&/g, '\&amp;')  
      .replace(/\</g, '\&lt;')  
      .replace(/\>/g, '\&gt;')  
      .replace(/"/g, '\&quot;')  
      .replace(/'/g, '&\#039;');  
  }

  // Event Listeners  
  presetSelect.addEventListener('change', (e) \=\> {  
    const key \= e.target.value;  
    if (PRESETS\[key\]) {  
      currentConfig \= JSON.parse(JSON.stringify(PRESETS\[key\]));  
      syncUIFromConfig();  
    }  
  });

  rolloutSlider.addEventListener('input', (e) \=\> {  
    rolloutValBadge.textContent \= \`\${e.target.value}%\`;  
    distBarEnabled.style.width \= \`\${e.target.value}%\`;  
    distBarEnabled.textContent \= \`\${e.target.value}% Active\`;  
    distBarDisabled.style.width \= \`\${100 \- e.target.value}%\`;  
    distBarDisabled.textContent \= \`\${100 \- e.target.value}% Legacy\`;  
    syncConfigFromUI();  
    evaluateSingleUser();  
    runMonteCarloSimulation();  
    renderSdk();  
  });

  flagEnabledToggle.addEventListener('change', () \=\> {  
    syncConfigFromUI();  
    evaluateSingleUser();  
    runMonteCarloSimulation();  
    renderSdk();  
  });

  flagIdInput.addEventListener('input', () \=\> {  
    syncConfigFromUI();  
    evaluateSingleUser();  
    renderSdk();  
  });

  flagSaltInput.addEventListener('input', () \=\> {  
    syncConfigFromUI();  
    evaluateSingleUser();  
    runMonteCarloSimulation();  
    renderSdk();  
  });

  flagTypeSelect.addEventListener('change', () \=\> {  
    syncConfigFromUI();  
    syncUIFromConfig();  
  });

  \[simUserIdInput, simTierInput, simCountryInput, simEmailInput\].forEach(el \=\> {  
    el.addEventListener('input', () \=\> evaluateSingleUser());  
  });

  runMonteCarloBtn.addEventListener('click', () \=\> {  
    runMonteCarloSimulation();  
  });

  tabButtons.forEach(btn \=\> {  
    btn.addEventListener('click', () \=\> {  
      tabButtons.forEach(b \=\> b.classList.remove('active'));  
      tabContents.forEach(c \=\> c.classList.remove('active'));  
      btn.classList.add('active');  
      const target \= btn.getAttribute('data-tab');  
      document.getElementById(target).classList.add('active');  
    });  
  });

  sdkFrameworkButtons.forEach(btn \=\> {  
    btn.addEventListener('click', () \=\> {  
      sdkFrameworkButtons.forEach(b \=\> b.classList.remove('active'));  
      btn.classList.add('active');  
      activeSdkFramework \= btn.getAttribute('data-sdk');  
      renderSdk();  
    });  
  });

  copySdkBtn.addEventListener('click', () \=\> {  
    navigator.clipboard.writeText(generatedSdkPre.textContent);  
    copySdkBtn.textContent \= 'Copied\!';  
    setTimeout(() \=\> copySdkBtn.textContent \= 'Copy SDK Code', 1500);  
  });

  exportJsonBtn.addEventListener('click', () \=\> {  
    const blob \= new Blob(\[JSON.stringify(currentConfig, null, 2)\], { type: 'application/json' });  
    const url \= URL.createObjectURL(blob);  
    const a \= document.createElement('a');  
    a.href \= url;  
    a.download \= \`featureflag-\${currentConfig.id}.json\`;  
    a.click();  
    URL.revokeObjectURL(url);  
  });

  // Init  
  initPresets().then(() \=\> {  
    if (PRESETS\['checkout-canary'\]) {  
      currentConfig \= JSON.parse(JSON.stringify(PRESETS\['checkout-canary'\]));  
    }  
    syncUIFromConfig();  
  });  
});  

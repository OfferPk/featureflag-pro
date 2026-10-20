# FeatureFlag Pro — User & Developer Guide

## 1\. Quickstart: Web Studio

1. Open `source-code/index.html` in any web browser.  
2. Select an experiment preset from the **Preset Experiment** dropdown, or configure a new flag:  
   - **Flag Key**: e.g., `new-checkout-v2`  
   - **Flag Type**: Percentage Rollout, Multivariant (A/B/n), or Operational Boolean.  
   - **Hash Salt**: A unique string seed (e.g. `checkout_v2_2026`).  
3. Adjust the **Rollout Slider** (0% to 100%).  
4. The studio updates in real time:  
   - **Distribution Bar**: Shows active vs legacy allocation.  
   - **User Test Sandbox**: Enter any User ID and custom attributes (`tier`, `country`, `email`) to see which bucket and variant the user resolves to.  
   - **1,000-User Simulator**: Runs a Monte Carlo simulation across 1,000 synthetic users to demonstrate statistical hash uniformity.  
   - **Zero-Latency SDKs**: Copy drop-in code for React, Node.js, Python, or Go.

---

## 2\. Setting Up Targeting Rules

Targeting rules allow you to bypass percentage rollouts for internal employees, QA teams, or specific geographical regions:

- **VIP Enterprise Override**:  
  - `tier equals vip` ➔ Action: `force_true` (100% enabled for VIPs).  
- **Internal Staff Beta**:  
  - `email contains @yourcompany.com` ➔ Action: `force_variant (hybrid-agent)`.  
- **Regional Rollout**:  
  - `country in_list AE,GB,US` ➔ Action: `percentage (50% rollout)`.

---

## 3\. Standalone Python CLI Tool (`featureflag-cli.py`)

The CLI utility runs on Python 3.8+ with zero external package dependencies.

### Evaluate a Specific User

\# Evaluate flag for a user

python3 build/featureflag-cli.py evaluate source-code/presets/saas-checkout-v2-canary.json \\

  \--user-id "usr\_1092" \--tier "vip" \--country "AE"

\# Output raw JSON

python3 build/featureflag-cli.py evaluate source-code/presets/saas-checkout-v2-canary.json \\

  \--user-id "usr\_1092" \--json

### Run Monte Carlo 1,000-User Simulation

python3 build/featureflag-cli.py simulate source-code/presets/saas-checkout-v2-canary.json \--users 1000

### CI/CD Pre-Deployment Audit

To audit flag manifests for missing fields, invalid percentage sums, or duplicate salts:

python3 build/featureflag-cli.py audit source-code/presets/saas-checkout-v2-canary.json \--fail-on-critical

\# Exits with status code 1 if errors exist
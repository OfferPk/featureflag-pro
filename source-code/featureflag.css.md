/\* FeatureFlag Pro \- Modern Dark UI & Rollout Simulator Stylesheet \*/

:root {  
  \--bg-primary: \#0b0f19;  
  \--bg-secondary: \#111827;  
  \--bg-tertiary: \#1f2937;  
  \--bg-card: \#161e2e;  
  \--bg-input: \#0d131f;

  \--border-color: \#2d3748;  
  \--border-focus: \#3b82f6;

  \--text-primary: \#f3f4f6;  
  \--text-secondary: \#9ca3af;  
  \--text-muted: \#6b7280;

  \--color-brand: \#3b82f6;  
  \--color-success: \#10b981;  
  \--color-danger: \#ef4444;  
  \--color-warning: \#f59e0b;  
  \--color-purple: \#8b5cf6;  
  \--color-cyan: \#06b6d4;

  \--font-sans: \-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;  
  \--font-mono: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;

  \--radius-sm: 6px;  
  \--radius-md: 10px;  
  \--radius-lg: 16px;  
  \--shadow-md: 0 4px 12px rgba(0, 0, 0, 0.5);  
  \--shadow-lg: 0 10px 25px rgba(0, 0, 0, 0.6);  
}

\* {  
  box-sizing: border-box;  
  margin: 0;  
  padding: 0;  
}

body {  
  background-color: var(--bg-primary);  
  color: var(--text-primary);  
  font-family: var(--font-sans);  
  line-height: 1.5;  
  min-height: 100vh;  
  display: flex;  
  flex-direction: column;  
}

/\* Header \*/  
header.app-header {  
  background-color: var(--bg-secondary);  
  border-bottom: 1px solid var(--border-color);  
  padding: 0.85rem 1.75rem;  
  display: flex;  
  justify-content: space-between;  
  align-items: center;  
  position: sticky;  
  top: 0;  
  z-index: 50;  
}

.brand-wrapper {  
  display: flex;  
  align-items: center;  
  gap: 0.75rem;  
}

.brand-icon {  
  width: 32px;  
  height: 32px;  
  background: linear-gradient(135deg, \#3b82f6, \#8b5cf6);  
  border-radius: var(--radius-sm);  
  display: flex;  
  align-items: center;  
  justify-content: center;  
  font-weight: 700;  
  font-size: 1.1rem;  
  color: white;  
}

.brand-title {  
  font-size: 1.25rem;  
  font-weight: 700;  
  letter-spacing: \-0.025em;  
  background: linear-gradient(90deg, \#60a5fa, \#c084fc);  
  \-webkit-background-clip: text;  
  \-webkit-text-fill-color: transparent;  
}

.brand-tag {  
  font-size: 0.7rem;  
  background-color: rgba(59, 130, 246, 0.15);  
  color: \#60a5fa;  
  padding: 0.15rem 0.5rem;  
  border-radius: 9999px;  
  border: 1px solid rgba(59, 130, 246, 0.3);  
  font-weight: 600;  
  text-transform: uppercase;  
}

.header-actions {  
  display: flex;  
  align-items: center;  
  gap: 1rem;  
}

.form-select {  
  background-color: var(--bg-input);  
  color: var(--text-primary);  
  border: 1px solid var(--border-color);  
  padding: 0.45rem 0.85rem;  
  border-radius: var(--radius-sm);  
  font-size: 0.85rem;  
  outline: none;  
  cursor: pointer;  
}

.form-select:focus {  
  border-color: var(--border-focus);  
}

.btn {  
  display: inline-flex;  
  align-items: center;  
  gap: 0.4rem;  
  padding: 0.45rem 0.95rem;  
  border-radius: var(--radius-sm);  
  font-size: 0.85rem;  
  font-weight: 600;  
  cursor: pointer;  
  border: 1px solid transparent;  
  transition: all 0.2s ease;  
}

.btn-primary {  
  background: linear-gradient(135deg, \#2563eb, \#3b82f6);  
  color: white;  
}  
.btn-primary:hover {  
  background: linear-gradient(135deg, \#1d4ed8, \#2563eb);  
}

.btn-secondary {  
  background-color: var(--bg-tertiary);  
  color: var(--text-primary);  
  border-color: var(--border-color);  
}  
.btn-secondary:hover {  
  background-color: \#374151;  
}

/\* Main Grid Layout \*/  
main.app-main {  
  flex: 1;  
  display: grid;  
  grid-template-columns: 1fr 1.25fr;  
  gap: 1.5rem;  
  padding: 1.5rem;  
  max-width: 1700px;  
  margin: 0 auto;  
  width: 100%;  
}

@media (max-width: 1100px) {  
  main.app-main {  
    grid-template-columns: 1fr;  
  }  
}

/\* Cards \*/  
.studio-card {  
  background-color: var(--bg-secondary);  
  border: 1px solid var(--border-color);  
  border-radius: var(--radius-md);  
  box-shadow: var(--shadow-md);  
  display: flex;  
  flex-direction: column;  
  overflow: hidden;  
}

.card-header {  
  padding: 0.85rem 1.25rem;  
  background-color: rgba(31, 41, 55, 0.5);  
  border-bottom: 1px solid var(--border-color);  
  display: flex;  
  justify-content: space-between;  
  align-items: center;  
}

.card-title {  
  font-size: 0.95rem;  
  font-weight: 700;  
  display: flex;  
  align-items: center;  
  gap: 0.5rem;  
}

.card-body {  
  padding: 1.25rem;  
  display: flex;  
  flex-direction: column;  
  gap: 1.25rem;  
}

/\* Form Controls \*/  
.form-group {  
  display: flex;  
  flex-direction: column;  
  gap: 0.4rem;  
}

.form-label {  
  font-size: 0.8rem;  
  font-weight: 600;  
  color: var(--text-secondary);  
}

.form-input {  
  background-color: var(--bg-input);  
  color: var(--text-primary);  
  border: 1px solid var(--border-color);  
  padding: 0.5rem 0.85rem;  
  border-radius: var(--radius-sm);  
  font-family: var(--font-mono);  
  font-size: 0.85rem;  
  outline: none;  
}

.form-input:focus {  
  border-color: var(--border-focus);  
}

/\* Rollout Slider & Distribution Bar \*/  
.rollout-slider-container {  
  background-color: var(--bg-card);  
  border: 1px solid var(--border-color);  
  border-radius: var(--radius-sm);  
  padding: 1rem;  
  display: flex;  
  flex-direction: column;  
  gap: 0.75rem;  
}

.slider-header {  
  display: flex;  
  justify-content: space-between;  
  align-items: center;  
}

.rollout-val-badge {  
  font-family: var(--font-mono);  
  font-size: 1.15rem;  
  font-weight: 800;  
  color: var(--color-success);  
}

input\[type="range"\].rollout-slider {  
  width: 100%;  
  accent-color: var(--color-brand);  
  cursor: pointer;  
}

.distribution-bar {  
  display: flex;  
  height: 20px;  
  border-radius: 4px;  
  overflow: hidden;  
  border: 1px solid var(--border-color);  
}

.dist-segment {  
  display: flex;  
  align-items: center;  
  justify-content: center;  
  font-size: 0.7rem;  
  font-weight: 700;  
  color: white;  
  transition: width 0.3s ease;  
}

.seg-enabled { background-color: var(--color-success); }  
.seg-disabled { background-color: \#374151; }

/\* Rules Section \*/  
.rules-container {  
  display: flex;  
  flex-direction: column;  
  gap: 0.75rem;  
}

.rule-card {  
  background-color: var(--bg-card);  
  border: 1px solid var(--border-color);  
  border-left: 4px solid var(--color-brand);  
  border-radius: var(--radius-sm);  
  padding: 0.85rem;  
  display: flex;  
  flex-direction: column;  
  gap: 0.4rem;  
}

.rule-title {  
  font-size: 0.85rem;  
  font-weight: 700;  
  display: flex;  
  justify-content: space-between;  
}

.rule-condition {  
  font-size: 0.8rem;  
  font-family: var(--font-mono);  
  color: \#60a5fa;  
  background-color: rgba(59, 130, 246, 0.1);  
  padding: 0.25rem 0.5rem;  
  border-radius: 4px;  
}

/\* Tabs \*/  
.tabs-nav {  
  display: flex;  
  border-bottom: 1px solid var(--border-color);  
  background-color: rgba(17, 24, 39, 0.7);  
}

.tab-btn {  
  padding: 0.75rem 1.25rem;  
  background: none;  
  border: none;  
  color: var(--text-secondary);  
  font-size: 0.85rem;  
  font-weight: 600;  
  cursor: pointer;  
  border-bottom: 2px solid transparent;  
  transition: all 0.2s;  
  display: flex;  
  align-items: center;  
  gap: 0.4rem;  
}

.tab-btn:hover {  
  color: var(--text-primary);  
}

.tab-btn.active {  
  color: \#60a5fa;  
  border-bottom-color: \#3b82f6;  
  background-color: rgba(59, 130, 246, 0.05);  
}

.tab-content {  
  display: none;  
  padding: 1.25rem;  
  flex-direction: column;  
  gap: 1.25rem;  
}

.tab-content.active {  
  display: flex;  
}

/\* Single User Test Simulator \*/  
.sim-result-box {  
  background-color: var(--bg-card);  
  border: 1px solid var(--border-color);  
  border-radius: var(--radius-md);  
  padding: 1.25rem;  
  display: flex;  
  align-items: center;  
  gap: 1.5rem;  
}

.sim-status-circle {  
  width: 80px;  
  height: 80px;  
  border-radius: 50%;  
  display: flex;  
  flex-direction: column;  
  align-items: center;  
  justify-content: center;  
  font-weight: 800;  
  border: 4px solid;  
}

.status-active { border-color: var(--color-success); color: var(--color-success); background: rgba(16, 185, 129, 0.1); }  
.status-inactive { border-color: var(--color-danger); color: var(--color-danger); background: rgba(239, 68, 68, 0.1); }  
.status-variant { border-color: var(--color-purple); color: var(--color-purple); background: rgba(139, 92, 246, 0.1); }

/\* Monte Carlo 1000 Population Visualizer \*/  
.population-stats-grid {  
  display: grid;  
  grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));  
  gap: 1rem;  
}

.pop-stat-card {  
  background-color: var(--bg-card);  
  border: 1px solid var(--border-color);  
  border-radius: var(--radius-sm);  
  padding: 1rem;  
  display: flex;  
  flex-direction: column;  
  gap: 0.3rem;  
}

.pop-stat-title {  
  font-size: 0.75rem;  
  color: var(--text-secondary);  
  text-transform: uppercase;  
  font-weight: 700;  
}

.pop-stat-val {  
  font-size: 1.3rem;  
  font-weight: 800;  
  font-family: var(--font-mono);  
}

/\* Code Pre \*/  
pre.code-preview {  
  background-color: var(--bg-input);  
  border: 1px solid var(--border-color);  
  border-radius: var(--radius-sm);  
  padding: 1rem;  
  font-family: var(--font-mono);  
  font-size: 0.8rem;  
  line-height: 1.5;  
  overflow-x: auto;  
  max-height: 400px;  
}

/\* Footer \*/  
footer.app-footer {  
  background-color: var(--bg-secondary);  
  border-top: 1px solid var(--border-color);  
  padding: 1rem 1.75rem;  
  font-size: 0.8rem;  
  color: var(--text-secondary);  
  display: flex;  
  justify-content: space-between;  
  align-items: center;  
  margin-top: auto;  
}

@media print {  
  header.app-header, .tabs-nav, .header-actions, button {  
    display: none \!important;  
  }  
  main.app-main {  
    display: block;  
    padding: 0;  
  }  
  .studio-card {  
    border: none;  
    box-shadow: none;  
  }  
  .tab-content {  
    display: block \!important;  
  }  
}  

---
name: ui-patterns
description: UI patterns for Chrome Extension interfaces covering popup design, options pages, side panel layouts, responsive design, dark mode, accessibility, form handling, and navigation patterns. Use when designing extension user interfaces.
---

# Chrome Extension UI Patterns

## Popup UI

### Popup Constraints

- **Max size:** 800px wide × 600px tall
- **Min size:** 25px × 25px
- **Behavior:** Closes on outside click
- **Lifecycle:** Recreated each time opened

### Basic Popup Structure

```html
<!-- popup.html -->
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <link rel="stylesheet" href="popup.css">
</head>
<body>
  <div class="popup-container">
    <header class="popup-header">
      <img src="icons/icon32.png" alt="Logo" class="logo">
      <h1>Extension Name</h1>
    </header>

    <main class="popup-content">
      <!-- Main content -->
    </main>

    <footer class="popup-footer">
      <button id="options">Settings</button>
    </footer>
  </div>
  <script src="popup.js"></script>
</body>
</html>
```

### Popup Styles

```css
/* popup.css */
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  width: 350px;
  min-height: 200px;
  font-family: system-ui, -apple-system, sans-serif;
  font-size: 14px;
  color: #333;
}

.popup-container {
  display: flex;
  flex-direction: column;
  min-height: 200px;
}

.popup-header {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 12px 16px;
  border-bottom: 1px solid #e0e0e0;
}

.popup-header h1 {
  font-size: 16px;
  font-weight: 600;
}

.popup-content {
  flex: 1;
  padding: 16px;
  overflow-y: auto;
}

.popup-footer {
  padding: 12px 16px;
  border-top: 1px solid #e0e0e0;
  display: flex;
  justify-content: flex-end;
  gap: 8px;
}
```

### Popup Patterns

**Quick Actions List:**
```html
<ul class="action-list">
  <li class="action-item" data-action="save">
    <span class="action-icon">💾</span>
    <span class="action-text">Save Page</span>
    <span class="action-shortcut">⌘S</span>
  </li>
  <li class="action-item" data-action="share">
    <span class="action-icon">📤</span>
    <span class="action-text">Share</span>
  </li>
</ul>
```

```css
.action-list {
  list-style: none;
}

.action-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 10px 12px;
  cursor: pointer;
  border-radius: 6px;
  transition: background 0.15s;
}

.action-item:hover {
  background: #f5f5f5;
}

.action-shortcut {
  margin-left: auto;
  font-size: 12px;
  color: #888;
}
```

---

## Side Panel UI

### Side Panel Constraints

- **Width:** Typically 320-400px (user can resize)
- **Height:** Full browser height
- **Behavior:** Persistent while browsing

### Side Panel Layout

```html
<!-- sidepanel.html -->
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <link rel="stylesheet" href="sidepanel.css">
</head>
<body>
  <div class="panel">
    <header class="panel-header">
      <div class="tab-info">
        <img id="favicon" class="favicon" alt="">
        <span id="site-name" class="site-name"></span>
      </div>
      <button class="header-action" aria-label="Refresh">🔄</button>
    </header>

    <nav class="panel-tabs">
      <button class="tab active" data-tab="main">Main</button>
      <button class="tab" data-tab="history">History</button>
      <button class="tab" data-tab="settings">Settings</button>
    </nav>

    <main class="panel-content">
      <section id="main" class="tab-content active">
        <!-- Main content -->
      </section>
      <section id="history" class="tab-content">
        <!-- History content -->
      </section>
      <section id="settings" class="tab-content">
        <!-- Settings content -->
      </section>
    </main>

    <footer class="panel-footer">
      <span class="status" id="status">Ready</span>
    </footer>
  </div>
  <script src="sidepanel.js"></script>
</body>
</html>
```

### Side Panel Styles

```css
/* sidepanel.css */
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: system-ui, -apple-system, sans-serif;
  font-size: 14px;
  color: #333;
  background: #fff;
}

.panel {
  display: flex;
  flex-direction: column;
  height: 100vh;
}

.panel-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 12px 16px;
  border-bottom: 1px solid #e0e0e0;
  flex-shrink: 0;
}

.tab-info {
  display: flex;
  align-items: center;
  gap: 8px;
  overflow: hidden;
}

.favicon {
  width: 16px;
  height: 16px;
}

.site-name {
  font-weight: 500;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.panel-tabs {
  display: flex;
  border-bottom: 1px solid #e0e0e0;
  flex-shrink: 0;
}

.tab {
  flex: 1;
  padding: 10px;
  border: none;
  background: none;
  cursor: pointer;
  font-size: 13px;
  color: #666;
  border-bottom: 2px solid transparent;
  transition: all 0.2s;
}

.tab:hover {
  color: #333;
  background: #f5f5f5;
}

.tab.active {
  color: #1a73e8;
  border-bottom-color: #1a73e8;
}

.panel-content {
  flex: 1;
  overflow-y: auto;
}

.tab-content {
  display: none;
  padding: 16px;
}

.tab-content.active {
  display: block;
}

.panel-footer {
  padding: 8px 16px;
  border-top: 1px solid #e0e0e0;
  font-size: 12px;
  color: #666;
  flex-shrink: 0;
}
```

---

## Options Page

### Options Structure

```html
<!-- options.html -->
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Extension Settings</title>
  <link rel="stylesheet" href="options.css">
</head>
<body>
  <div class="options-container">
    <header class="options-header">
      <h1>Extension Settings</h1>
    </header>

    <main class="options-content">
      <section class="settings-group">
        <h2>General</h2>

        <div class="setting-item">
          <div class="setting-info">
            <label for="autoSave">Auto-save pages</label>
            <p class="setting-description">Automatically save pages when visiting</p>
          </div>
          <input type="checkbox" id="autoSave" class="toggle">
        </div>

        <div class="setting-item">
          <div class="setting-info">
            <label for="theme">Theme</label>
          </div>
          <select id="theme" class="select">
            <option value="system">System</option>
            <option value="light">Light</option>
            <option value="dark">Dark</option>
          </select>
        </div>
      </section>

      <section class="settings-group">
        <h2>Keyboard Shortcuts</h2>
        <p class="section-description">
          Configure shortcuts in
          <a href="chrome://extensions/shortcuts" id="shortcuts-link">
            Chrome Extension Shortcuts
          </a>
        </p>
      </section>

      <section class="settings-group danger-zone">
        <h2>Danger Zone</h2>
        <button id="clearData" class="btn-danger">Clear All Data</button>
      </section>
    </main>

    <div id="toast" class="toast hidden">Settings saved</div>
  </div>
  <script src="options.js"></script>
</body>
</html>
```

### Options Styles

```css
/* options.css */
body {
  font-family: system-ui, -apple-system, sans-serif;
  font-size: 14px;
  color: #333;
  background: #f5f5f5;
  margin: 0;
  padding: 20px;
}

.options-container {
  max-width: 600px;
  margin: 0 auto;
}

.options-header {
  margin-bottom: 24px;
}

.options-header h1 {
  font-size: 24px;
  font-weight: 600;
}

.settings-group {
  background: white;
  border-radius: 8px;
  padding: 20px;
  margin-bottom: 16px;
  box-shadow: 0 1px 3px rgba(0,0,0,0.1);
}

.settings-group h2 {
  font-size: 16px;
  font-weight: 600;
  margin-bottom: 16px;
  padding-bottom: 8px;
  border-bottom: 1px solid #e0e0e0;
}

.setting-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 0;
  border-bottom: 1px solid #f0f0f0;
}

.setting-item:last-child {
  border-bottom: none;
}

.setting-info {
  flex: 1;
}

.setting-info label {
  font-weight: 500;
  cursor: pointer;
}

.setting-description {
  font-size: 12px;
  color: #666;
  margin-top: 4px;
}

/* Toggle Switch */
.toggle {
  appearance: none;
  width: 44px;
  height: 24px;
  background: #ccc;
  border-radius: 12px;
  position: relative;
  cursor: pointer;
  transition: background 0.2s;
}

.toggle::after {
  content: '';
  position: absolute;
  top: 2px;
  left: 2px;
  width: 20px;
  height: 20px;
  background: white;
  border-radius: 50%;
  transition: transform 0.2s;
}

.toggle:checked {
  background: #1a73e8;
}

.toggle:checked::after {
  transform: translateX(20px);
}

/* Select */
.select {
  padding: 8px 12px;
  border: 1px solid #ddd;
  border-radius: 6px;
  font-size: 14px;
  min-width: 120px;
}

/* Danger Zone */
.danger-zone {
  border: 1px solid #ffcdd2;
}

.btn-danger {
  background: #dc3545;
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 6px;
  cursor: pointer;
}

.btn-danger:hover {
  background: #c82333;
}

/* Toast Notification */
.toast {
  position: fixed;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%);
  background: #333;
  color: white;
  padding: 12px 24px;
  border-radius: 8px;
  transition: opacity 0.3s;
}

.toast.hidden {
  opacity: 0;
  pointer-events: none;
}
```

---

## Dark Mode Support

### CSS Variables Approach

```css
:root {
  --bg-primary: #ffffff;
  --bg-secondary: #f5f5f5;
  --text-primary: #333333;
  --text-secondary: #666666;
  --border-color: #e0e0e0;
  --accent-color: #1a73e8;
}

@media (prefers-color-scheme: dark) {
  :root {
    --bg-primary: #1e1e1e;
    --bg-secondary: #2d2d2d;
    --text-primary: #e0e0e0;
    --text-secondary: #a0a0a0;
    --border-color: #404040;
    --accent-color: #8ab4f8;
  }
}

body {
  background: var(--bg-primary);
  color: var(--text-primary);
}

.card {
  background: var(--bg-secondary);
  border: 1px solid var(--border-color);
}
```

### Manual Theme Toggle

```javascript
// options.js
async function applyTheme(theme) {
  if (theme === 'system') {
    document.documentElement.removeAttribute('data-theme');
  } else {
    document.documentElement.setAttribute('data-theme', theme);
  }
  await chrome.storage.sync.set({ theme });
}

// CSS
[data-theme="dark"] {
  --bg-primary: #1e1e1e;
  /* ... dark values */
}

[data-theme="light"] {
  --bg-primary: #ffffff;
  /* ... light values */
}
```

---

## Accessibility

### Focus Management

```css
/* Visible focus indicators */
button:focus-visible,
input:focus-visible,
select:focus-visible {
  outline: 2px solid var(--accent-color);
  outline-offset: 2px;
}

/* Skip links for keyboard navigation */
.skip-link {
  position: absolute;
  top: -40px;
  left: 0;
  background: var(--accent-color);
  color: white;
  padding: 8px;
  z-index: 100;
}

.skip-link:focus {
  top: 0;
}
```

### ARIA Labels

```html
<!-- Icon-only buttons -->
<button aria-label="Close" class="close-btn">×</button>
<button aria-label="Settings" class="icon-btn">⚙️</button>

<!-- Loading states -->
<div aria-live="polite" aria-busy="true">
  Loading...
</div>

<!-- Tabs -->
<div role="tablist">
  <button role="tab" aria-selected="true" aria-controls="panel1">Tab 1</button>
  <button role="tab" aria-selected="false" aria-controls="panel2">Tab 2</button>
</div>
<div role="tabpanel" id="panel1">Content 1</div>
<div role="tabpanel" id="panel2" hidden>Content 2</div>
```

### Keyboard Navigation

```javascript
// Keyboard shortcuts
document.addEventListener('keydown', (e) => {
  // Escape to close
  if (e.key === 'Escape') {
    closeModal();
  }

  // Tab navigation in lists
  if (e.key === 'ArrowDown' || e.key === 'ArrowUp') {
    navigateList(e.key === 'ArrowDown' ? 1 : -1);
    e.preventDefault();
  }
});

// Trap focus in modals
function trapFocus(element) {
  const focusables = element.querySelectorAll(
    'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
  );
  const first = focusables[0];
  const last = focusables[focusables.length - 1];

  element.addEventListener('keydown', (e) => {
    if (e.key !== 'Tab') return;

    if (e.shiftKey && document.activeElement === first) {
      last.focus();
      e.preventDefault();
    } else if (!e.shiftKey && document.activeElement === last) {
      first.focus();
      e.preventDefault();
    }
  });
}
```

---

## Loading States

```html
<!-- Skeleton Loading -->
<div class="skeleton-container">
  <div class="skeleton skeleton-text"></div>
  <div class="skeleton skeleton-text short"></div>
</div>

<!-- Spinner -->
<div class="spinner"></div>

<!-- Progress Bar -->
<div class="progress-bar">
  <div class="progress-fill" style="width: 60%"></div>
</div>
```

```css
/* Skeleton */
.skeleton {
  background: linear-gradient(90deg, #f0f0f0 25%, #e0e0e0 50%, #f0f0f0 75%);
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite;
  border-radius: 4px;
}

.skeleton-text {
  height: 16px;
  margin-bottom: 8px;
}

.skeleton-text.short {
  width: 60%;
}

@keyframes shimmer {
  0% { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}

/* Spinner */
.spinner {
  width: 24px;
  height: 24px;
  border: 3px solid #f0f0f0;
  border-top-color: var(--accent-color);
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

/* Progress Bar */
.progress-bar {
  height: 4px;
  background: #e0e0e0;
  border-radius: 2px;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background: var(--accent-color);
  transition: width 0.3s;
}
```

---

## Form Patterns

```html
<form class="form" id="settingsForm">
  <div class="form-group">
    <label for="apiKey">API Key</label>
    <input type="password" id="apiKey" placeholder="Enter your API key">
    <p class="form-hint">Get your key from the dashboard</p>
  </div>

  <div class="form-group">
    <label for="language">Language</label>
    <select id="language">
      <option value="en">English</option>
      <option value="es">Spanish</option>
    </select>
  </div>

  <div class="form-actions">
    <button type="button" class="btn-secondary">Cancel</button>
    <button type="submit" class="btn-primary">Save</button>
  </div>
</form>
```

```css
.form-group {
  margin-bottom: 16px;
}

.form-group label {
  display: block;
  margin-bottom: 6px;
  font-weight: 500;
}

.form-group input,
.form-group select {
  width: 100%;
  padding: 10px 12px;
  border: 1px solid var(--border-color);
  border-radius: 6px;
  font-size: 14px;
}

.form-group input:focus,
.form-group select:focus {
  border-color: var(--accent-color);
  outline: none;
}

.form-hint {
  font-size: 12px;
  color: var(--text-secondary);
  margin-top: 4px;
}

.form-actions {
  display: flex;
  gap: 8px;
  justify-content: flex-end;
  margin-top: 24px;
}

.btn-primary {
  background: var(--accent-color);
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 6px;
  font-weight: 500;
  cursor: pointer;
}

.btn-secondary {
  background: transparent;
  color: var(--text-primary);
  border: 1px solid var(--border-color);
  padding: 10px 20px;
  border-radius: 6px;
  cursor: pointer;
}
```

---

## Best Practices

1. **Set explicit dimensions** - Prevent layout shifts
2. **Use system fonts** - Faster loading, native feel
3. **Support dark mode** - Match user preference
4. **Test at different sizes** - Popup can be resized
5. **Provide keyboard navigation** - Accessibility
6. **Show loading states** - User feedback
7. **Use CSS variables** - Easy theming
8. **Minimize external resources** - Faster loading
9. **Test with screen readers** - Full accessibility
10. **Follow platform conventions** - Familiar patterns

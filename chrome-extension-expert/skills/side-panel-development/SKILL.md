---
name: side-panel-development
description: Side Panel development patterns for Chrome Extensions covering manifest configuration, chrome.sidePanel API, programmatic control, per-tab vs global panels, lifecycle management, and communication patterns. Use when building side panel features.
---

# Chrome Extension Side Panel Development

## Overview

Side panels provide a persistent UI alongside web pages, offering richer interaction than popups.

**Key Characteristics:**
- Persistent while browsing (unlike popups)
- Can be per-tab or global
- Requires `sidePanel` permission
- Available in Chrome 114+

---

## Manifest Configuration

### Basic Setup

```json
{
  "manifest_version": 3,
  "name": "Side Panel Extension",
  "version": "1.0.0",
  "permissions": ["sidePanel"],
  "side_panel": {
    "default_path": "sidepanel.html"
  },
  "action": {
    "default_title": "Open Side Panel"
  },
  "background": {
    "service_worker": "background.js"
  }
}
```

### Side Panel HTML

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
  <div id="app">
    <header>
      <h1>Side Panel</h1>
    </header>
    <main id="content">
      <!-- Content here -->
    </main>
  </div>
  <script src="sidepanel.js"></script>
</body>
</html>
```

---

## chrome.sidePanel API

### Opening Side Panel

```javascript
// background.js

// Open on action click
chrome.sidePanel.setPanelBehavior({
  openPanelOnActionClick: true
});

// Or manually open on click
chrome.action.onClicked.addListener(async (tab) => {
  await chrome.sidePanel.open({ tabId: tab.id });
});

// Open for entire window
chrome.action.onClicked.addListener(async (tab) => {
  await chrome.sidePanel.open({ windowId: tab.windowId });
});
```

### Setting Panel Options

```javascript
// Set panel for specific tab
await chrome.sidePanel.setOptions({
  tabId: tab.id,
  path: 'sidepanel.html',
  enabled: true
});

// Disable panel for specific tab
await chrome.sidePanel.setOptions({
  tabId: tab.id,
  enabled: false
});

// Set default panel (all tabs)
await chrome.sidePanel.setOptions({
  path: 'sidepanel.html',
  enabled: true
});
```

### Getting Panel Options

```javascript
// Get options for specific tab
const options = await chrome.sidePanel.getOptions({ tabId: tab.id });
console.log('Panel path:', options.path);
console.log('Enabled:', options.enabled);

// Get default options
const defaultOptions = await chrome.sidePanel.getOptions({});
```

### Panel Behavior

```javascript
// Auto-open on extension icon click
await chrome.sidePanel.setPanelBehavior({
  openPanelOnActionClick: true
});

// Get current behavior
const behavior = await chrome.sidePanel.getPanelBehavior();
console.log('Opens on click:', behavior.openPanelOnActionClick);
```

---

## Per-Tab vs Global Side Panels

### Global Panel (Same for All Tabs)

```javascript
// background.js - Set once
chrome.runtime.onInstalled.addListener(() => {
  chrome.sidePanel.setOptions({
    path: 'sidepanel.html',
    enabled: true
  });
});
```

### Per-Tab Panel (Different Content per Tab)

```javascript
// background.js
chrome.tabs.onActivated.addListener(async ({ tabId }) => {
  const tab = await chrome.tabs.get(tabId);

  // Different panel based on URL
  if (tab.url?.includes('github.com')) {
    await chrome.sidePanel.setOptions({
      tabId,
      path: 'sidepanel-github.html'
    });
  } else if (tab.url?.includes('docs.google.com')) {
    await chrome.sidePanel.setOptions({
      tabId,
      path: 'sidepanel-docs.html'
    });
  } else {
    await chrome.sidePanel.setOptions({
      tabId,
      path: 'sidepanel-default.html'
    });
  }
});

// Also handle URL changes within tab
chrome.tabs.onUpdated.addListener(async (tabId, changeInfo, tab) => {
  if (changeInfo.url) {
    // Update panel based on new URL
    await updatePanelForTab(tabId, changeInfo.url);
  }
});
```

---

## Side Panel Lifecycle

### Panel Load/Unload

```javascript
// sidepanel.js
document.addEventListener('DOMContentLoaded', async () => {
  console.log('Side panel loaded');

  // Get current tab
  const [tab] = await chrome.tabs.query({
    active: true,
    currentWindow: true
  });

  // Initialize with tab data
  await initializePanel(tab);
});

// Clean up before unload
window.addEventListener('beforeunload', () => {
  // Save state
  saveCurrentState();
});

// Visibility change (panel minimized/restored)
document.addEventListener('visibilitychange', () => {
  if (document.hidden) {
    console.log('Panel hidden');
  } else {
    console.log('Panel visible');
  }
});
```

### State Persistence

```javascript
// sidepanel.js

// Save state when it changes
async function saveState(state) {
  await chrome.storage.session.set({ sidePanelState: state });
}

// Restore state on load
async function restoreState() {
  const { sidePanelState } = await chrome.storage.session.get('sidePanelState');
  if (sidePanelState) {
    applyState(sidePanelState);
  }
}

// Tab-specific state
async function saveTabState(tabId, state) {
  const key = `sidePanelState_${tabId}`;
  await chrome.storage.session.set({ [key]: state });
}

async function getTabState(tabId) {
  const key = `sidePanelState_${tabId}`;
  const result = await chrome.storage.session.get(key);
  return result[key];
}
```

---

## Communication Patterns

### Side Panel → Background

```javascript
// sidepanel.js
async function fetchData() {
  const response = await chrome.runtime.sendMessage({
    type: 'FETCH_DATA',
    query: 'search term'
  });

  if (response.success) {
    displayData(response.data);
  }
}
```

### Background → Side Panel

```javascript
// background.js
function notifySidePanel(data) {
  chrome.runtime.sendMessage({
    type: 'DATA_UPDATE',
    data
  }).catch(() => {
    // Side panel might not be open
  });
}
```

### Side Panel ↔ Content Script (via Background)

```javascript
// sidepanel.js
async function getPageData() {
  const [tab] = await chrome.tabs.query({
    active: true,
    currentWindow: true
  });

  // Request goes through background
  const response = await chrome.runtime.sendMessage({
    type: 'GET_PAGE_DATA',
    tabId: tab.id
  });

  return response;
}

// background.js
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (message.type === 'GET_PAGE_DATA') {
    chrome.tabs.sendMessage(message.tabId, { type: 'EXTRACT_DATA' })
      .then(sendResponse)
      .catch(error => sendResponse({ error: error.message }));
    return true;
  }
});
```

### Long-Lived Connection

```javascript
// sidepanel.js
let port;

function connectToBackground() {
  port = chrome.runtime.connect({ name: 'sidepanel' });

  port.onMessage.addListener((message) => {
    handleBackgroundMessage(message);
  });

  port.onDisconnect.addListener(() => {
    console.log('Disconnected, reconnecting...');
    setTimeout(connectToBackground, 1000);
  });
}

document.addEventListener('DOMContentLoaded', connectToBackground);

function sendToBackground(message) {
  if (port) {
    port.postMessage(message);
  }
}
```

---

## Side Panel vs Popup Decision Guide

| Feature | Side Panel | Popup |
|---------|------------|-------|
| **Persistence** | Stays open while browsing | Closes on outside click |
| **Size** | Full height, ~400px width | Max 800x600 |
| **Use Case** | Ongoing tasks, reference | Quick actions |
| **Tab Context** | Can track active tab | Single tab context |
| **Performance** | Always loaded when open | Loaded on demand |
| **Minimum Chrome** | 114+ | All versions |

**Choose Side Panel when:**
- User needs persistent workspace
- Content relates to multiple tabs
- Complex interactions required
- Reference material while browsing

**Choose Popup when:**
- Quick, single action
- Simple settings toggle
- Compatibility with older Chrome
- Minimal resource usage

---

## UI Patterns for Side Panels

### Responsive Width

```css
/* sidepanel.css */
:root {
  --panel-padding: 16px;
}

body {
  margin: 0;
  padding: var(--panel-padding);
  font-family: system-ui, sans-serif;
  font-size: 14px;
  /* Panel width is typically 320-400px */
  min-width: 280px;
  max-width: 100%;
}

/* Stack layout for narrow panels */
.container {
  display: flex;
  flex-direction: column;
  gap: 12px;
}
```

### Header with Tab Info

```javascript
// sidepanel.js
async function updateHeader() {
  const [tab] = await chrome.tabs.query({
    active: true,
    currentWindow: true
  });

  document.getElementById('site-name').textContent =
    new URL(tab.url).hostname;
  document.getElementById('favicon').src = tab.favIconUrl || '';
}

// Update when tab changes
chrome.tabs.onActivated.addListener(updateHeader);
chrome.tabs.onUpdated.addListener((tabId, changeInfo) => {
  if (changeInfo.title || changeInfo.favIconUrl) {
    updateHeader();
  }
});
```

### Loading States

```javascript
// sidepanel.js
function showLoading() {
  document.getElementById('content').innerHTML = `
    <div class="loading">
      <div class="spinner"></div>
      <p>Loading...</p>
    </div>
  `;
}

function showError(message) {
  document.getElementById('content').innerHTML = `
    <div class="error">
      <p>${message}</p>
      <button onclick="retry()">Retry</button>
    </div>
  `;
}
```

### Scrollable Content

```css
/* sidepanel.css */
#app {
  display: flex;
  flex-direction: column;
  height: 100vh;
}

header {
  flex-shrink: 0;
  padding: 12px;
  border-bottom: 1px solid #e0e0e0;
}

main {
  flex: 1;
  overflow-y: auto;
  padding: 12px;
}

footer {
  flex-shrink: 0;
  padding: 12px;
  border-top: 1px solid #e0e0e0;
}
```

---

## Advanced Patterns

### Tab-Specific Side Panels

```javascript
// background.js
const tabPanelStates = new Map();

chrome.tabs.onActivated.addListener(async ({ tabId, windowId }) => {
  // Get or create state for this tab
  let state = tabPanelStates.get(tabId);
  if (!state) {
    state = { initialized: false, data: null };
    tabPanelStates.set(tabId, state);
  }

  // Notify side panel of tab change
  chrome.runtime.sendMessage({
    type: 'TAB_CHANGED',
    tabId,
    state
  }).catch(() => {});
});

// Clean up when tab closes
chrome.tabs.onRemoved.addListener((tabId) => {
  tabPanelStates.delete(tabId);
});
```

### Conditional Side Panel Enabling

```javascript
// background.js
chrome.tabs.onUpdated.addListener(async (tabId, changeInfo, tab) => {
  if (changeInfo.status === 'complete' && tab.url) {
    // Only enable for supported sites
    const supportedSites = [
      'github.com',
      'stackoverflow.com',
      'developer.mozilla.org'
    ];

    const isSupported = supportedSites.some(site =>
      tab.url.includes(site)
    );

    await chrome.sidePanel.setOptions({
      tabId,
      enabled: isSupported
    });
  }
});
```

### Side Panel with Content Script Sync

```javascript
// content.js
function setupMutationObserver() {
  const observer = new MutationObserver((mutations) => {
    // Detect relevant page changes
    const relevantChanges = filterRelevantMutations(mutations);

    if (relevantChanges.length > 0) {
      chrome.runtime.sendMessage({
        type: 'PAGE_CONTENT_CHANGED',
        changes: relevantChanges
      });
    }
  });

  observer.observe(document.body, {
    childList: true,
    subtree: true
  });
}

// sidepanel.js - Listen for updates
chrome.runtime.onMessage.addListener((message) => {
  if (message.type === 'PAGE_CONTENT_CHANGED') {
    refreshData(message.changes);
  }
});
```

---

## Best Practices

1. **Save state frequently** - Users expect panel state to persist
2. **Handle tab switches gracefully** - Update content for new tab
3. **Use efficient layouts** - Narrow width requires careful design
4. **Communicate via background** - Never assume direct content script access
5. **Show loading states** - Panel operations can be async
6. **Handle disconnections** - Reconnect ports automatically
7. **Test across window sizes** - Users may resize browser
8. **Consider dark mode** - Match system preferences
9. **Minimize background wake-ups** - Battery/performance impact
10. **Provide clear close/minimize options** - User control

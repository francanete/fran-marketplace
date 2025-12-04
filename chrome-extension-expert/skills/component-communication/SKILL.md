---
name: component-communication
description: Inter-component messaging patterns for Chrome Extensions covering background ↔ content script ↔ popup ↔ side panel communication, ports, message passing, state synchronization, and error handling. Essential for multi-component extensions.
---

# Chrome Extension Component Communication

## Overview

Chrome extensions have isolated components that must communicate via message passing:

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   Side Panel    │     │     Popup       │     │  Options Page   │
└────────┬────────┘     └────────┬────────┘     └────────┬────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                    ┌────────────┴────────────┐
                    │   Background Service    │
                    │       Worker            │
                    └────────────┬────────────┘
                                 │
         ┌───────────────────────┼───────────────────────┐
         │                       │                       │
┌────────┴────────┐     ┌────────┴────────┐     ┌────────┴────────┐
│ Content Script  │     │ Content Script  │     │ Content Script  │
│    (Tab 1)      │     │    (Tab 2)      │     │    (Tab 3)      │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

---

## One-Time Messages

### Background ↔ Content Script

**Content Script → Background:**
```javascript
// content.js
const response = await chrome.runtime.sendMessage({
  type: 'SAVE_DATA',
  data: { url: window.location.href }
});
console.log('Response:', response);
```

**Background → Content Script:**
```javascript
// background.js
const response = await chrome.tabs.sendMessage(tabId, {
  type: 'GET_PAGE_DATA'
});
```

**Background Listener:**
```javascript
// background.js
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  // sender.tab exists if from content script
  if (sender.tab) {
    console.log('From tab:', sender.tab.id, sender.tab.url);
  }

  switch (message.type) {
    case 'SAVE_DATA':
      saveData(message.data)
        .then(result => sendResponse({ success: true, result }))
        .catch(error => sendResponse({ success: false, error: error.message }));
      return true; // CRITICAL: Keep channel open for async response

    case 'GET_STATUS':
      sendResponse({ status: 'ready' }); // Sync response
      break;
  }
});
```

### Background ↔ Popup / Side Panel

**Popup/Side Panel → Background:**
```javascript
// popup.js or sidepanel.js
const response = await chrome.runtime.sendMessage({
  type: 'FETCH_DATA',
  query: 'search term'
});
updateUI(response.data);
```

**Background → Popup/Side Panel:**
```javascript
// background.js - Broadcast to all extension pages
chrome.runtime.sendMessage({
  type: 'DATA_UPDATED',
  data: newData
}).catch(() => {
  // No listeners - popup/panel might be closed
});
```

---

## Long-Lived Connections (Ports)

Use for streaming data or maintaining connection state.

### Creating Ports

**Content Script → Background:**
```javascript
// content.js
const port = chrome.runtime.connect({ name: 'content-channel' });

port.onMessage.addListener((message) => {
  console.log('Received:', message);
});

port.postMessage({ type: 'INIT', url: location.href });

// Handle disconnect
port.onDisconnect.addListener(() => {
  console.log('Port disconnected');
  if (chrome.runtime.lastError) {
    console.error(chrome.runtime.lastError.message);
  }
});
```

**Background Port Listener:**
```javascript
// background.js
const connectedPorts = new Map();

chrome.runtime.onConnect.addListener((port) => {
  console.log('New connection:', port.name);

  if (port.name === 'content-channel') {
    const tabId = port.sender.tab.id;
    connectedPorts.set(tabId, port);

    port.onMessage.addListener((message) => {
      handleContentMessage(message, port, tabId);
    });

    port.onDisconnect.addListener(() => {
      connectedPorts.delete(tabId);
    });
  }
});

// Send to specific tab's content script
function sendToTab(tabId, message) {
  const port = connectedPorts.get(tabId);
  if (port) {
    port.postMessage(message);
  }
}
```

### Background ↔ Side Panel Port

```javascript
// sidepanel.js
let backgroundPort;

function connectToBackground() {
  backgroundPort = chrome.runtime.connect({ name: 'sidepanel' });

  backgroundPort.onMessage.addListener((message) => {
    switch (message.type) {
      case 'UPDATE':
        updateUI(message.data);
        break;
      case 'STATUS':
        showStatus(message.status);
        break;
    }
  });

  backgroundPort.onDisconnect.addListener(() => {
    // Reconnect after delay
    setTimeout(connectToBackground, 1000);
  });
}

document.addEventListener('DOMContentLoaded', connectToBackground);
```

---

## Content Script ↔ Side Panel Communication

Content scripts and side panel cannot communicate directly. Use background as relay.

### Pattern: Background Relay

```javascript
// content.js
chrome.runtime.sendMessage({
  type: 'TO_SIDEPANEL',
  payload: { selectedText: selection }
});

// background.js
const sidePanelPorts = new Map();

chrome.runtime.onConnect.addListener((port) => {
  if (port.name === 'sidepanel') {
    const windowId = port.sender.tab?.windowId;
    sidePanelPorts.set(windowId, port);

    port.onDisconnect.addListener(() => {
      sidePanelPorts.delete(windowId);
    });
  }
});

chrome.runtime.onMessage.addListener((message, sender) => {
  if (message.type === 'TO_SIDEPANEL') {
    const windowId = sender.tab.windowId;
    const sidePanelPort = sidePanelPorts.get(windowId);
    if (sidePanelPort) {
      sidePanelPort.postMessage({
        type: 'FROM_CONTENT',
        tabId: sender.tab.id,
        payload: message.payload
      });
    }
  }
});

// sidepanel.js
const port = chrome.runtime.connect({ name: 'sidepanel' });
port.onMessage.addListener((message) => {
  if (message.type === 'FROM_CONTENT') {
    handleContentData(message.payload, message.tabId);
  }
});
```

---

## Broadcasting to All Tabs

```javascript
// background.js
async function broadcastToAllTabs(message) {
  const tabs = await chrome.tabs.query({});

  for (const tab of tabs) {
    try {
      await chrome.tabs.sendMessage(tab.id, message);
    } catch (error) {
      // Tab doesn't have content script
    }
  }
}

// Broadcast to tabs matching pattern
async function broadcastToMatchingTabs(pattern, message) {
  const tabs = await chrome.tabs.query({ url: pattern });

  const results = await Promise.allSettled(
    tabs.map(tab => chrome.tabs.sendMessage(tab.id, message))
  );

  return results
    .filter(r => r.status === 'fulfilled')
    .map(r => r.value);
}
```

---

## State Synchronization

### Using Storage for State Sync

```javascript
// Any component - update state
async function updateSharedState(updates) {
  const { state } = await chrome.storage.local.get('state');
  const newState = { ...state, ...updates };
  await chrome.storage.local.set({ state: newState });
}

// Any component - listen for changes
chrome.storage.onChanged.addListener((changes, areaName) => {
  if (areaName === 'local' && changes.state) {
    const newState = changes.state.newValue;
    updateUI(newState);
  }
});
```

### Centralized State in Background

```javascript
// background.js
let appState = {
  user: null,
  data: [],
  settings: {}
};

chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  switch (message.type) {
    case 'GET_STATE':
      sendResponse({ state: appState });
      break;

    case 'UPDATE_STATE':
      appState = { ...appState, ...message.updates };
      // Notify all listeners
      broadcastStateUpdate(appState);
      sendResponse({ success: true, state: appState });
      break;

    case 'SUBSCRIBE':
      // Add to subscribers list
      break;
  }
  return true;
});

function broadcastStateUpdate(state) {
  chrome.runtime.sendMessage({
    type: 'STATE_CHANGED',
    state
  }).catch(() => {});
}
```

---

## Error Handling

### Common Errors

**"Receiving end does not exist"**
```javascript
async function safeSendToTab(tabId, message) {
  try {
    return await chrome.tabs.sendMessage(tabId, message);
  } catch (error) {
    if (error.message.includes('Receiving end does not exist')) {
      // Inject content script first
      await chrome.scripting.executeScript({
        target: { tabId },
        files: ['content.js']
      });
      // Retry
      return await chrome.tabs.sendMessage(tabId, message);
    }
    throw error;
  }
}
```

**"Extension context invalidated"**
```javascript
// content.js
function isContextValid() {
  try {
    chrome.runtime.id;
    return true;
  } catch {
    return false;
  }
}

async function safeSendMessage(message) {
  if (!isContextValid()) {
    console.warn('Extension context invalidated - reload page');
    return null;
  }
  return chrome.runtime.sendMessage(message);
}
```

**"Could not establish connection"**
```javascript
// With retry logic
async function sendWithRetry(message, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await chrome.runtime.sendMessage(message);
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(r => setTimeout(r, 100 * (i + 1)));
    }
  }
}
```

---

## Message Protocol Design

### Structured Message Format

```typescript
interface Message {
  type: string;
  payload?: any;
  requestId?: string;  // For request/response tracking
  timestamp?: number;
}

interface Response {
  success: boolean;
  data?: any;
  error?: string;
  requestId?: string;
}
```

### Type-Safe Messaging

```javascript
// messages.js - Shared message types
const MessageTypes = {
  // Content → Background
  SAVE_SELECTION: 'SAVE_SELECTION',
  GET_PAGE_DATA: 'GET_PAGE_DATA',

  // Background → Content
  HIGHLIGHT_TEXT: 'HIGHLIGHT_TEXT',
  UPDATE_CONFIG: 'UPDATE_CONFIG',

  // Background → UI (Popup/SidePanel)
  DATA_UPDATED: 'DATA_UPDATED',
  STATUS_CHANGED: 'STATUS_CHANGED'
};

// Create message helper
function createMessage(type, payload = null) {
  return {
    type,
    payload,
    requestId: crypto.randomUUID(),
    timestamp: Date.now()
  };
}
```

---

## External Messaging

### From Web Pages

**manifest.json:**
```json
{
  "externally_connectable": {
    "matches": ["https://mywebsite.com/*"]
  }
}
```

**Web page:**
```javascript
// On https://mywebsite.com
chrome.runtime.sendMessage(
  'extension-id-here',
  { type: 'FROM_WEBSITE', data: 'hello' },
  (response) => {
    console.log('Extension responded:', response);
  }
);
```

**Background listener:**
```javascript
chrome.runtime.onMessageExternal.addListener(
  (message, sender, sendResponse) => {
    // Verify sender
    if (!sender.url.startsWith('https://mywebsite.com')) {
      sendResponse({ error: 'Unauthorized' });
      return;
    }

    handleExternalMessage(message)
      .then(sendResponse);
    return true;
  }
);
```

### Between Extensions

```javascript
// Send to another extension
chrome.runtime.sendMessage(
  'other-extension-id',
  { type: 'CROSS_EXT_MESSAGE' },
  (response) => {}
);

// Receive from other extensions
chrome.runtime.onMessageExternal.addListener(
  (message, sender, sendResponse) => {
    console.log('From extension:', sender.id);
  }
);
```

---

## Best Practices

1. **Always return true for async responses**
2. **Validate message sources** - Check `sender.id` and `sender.url`
3. **Handle disconnections gracefully**
4. **Use structured message types** - Avoid magic strings
5. **Implement retry logic** - Network can be flaky
6. **Clean up ports on disconnect**
7. **Don't assume listeners exist** - Wrap in try-catch
8. **Use storage for persistent state** - Messages are ephemeral
9. **Rate limit messages** - Avoid flooding
10. **Log message flow in development** - Easier debugging

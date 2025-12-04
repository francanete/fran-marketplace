---
name: migration-guide
description: Manifest V2 to V3 migration guide covering background page to service worker, API changes, webRequest to declarativeNetRequest, remote code removal, executeScript changes, and persistence patterns. Use when upgrading extensions to Manifest V3.
---

# Manifest V2 to V3 Migration Guide

## Overview of Changes

| Feature | Manifest V2 | Manifest V3 |
|---------|-------------|-------------|
| Background | Persistent page | Service worker |
| Remote code | Allowed | Forbidden |
| Web requests | webRequest API | declarativeNetRequest |
| Host permissions | In permissions | Separate field |
| Content scripts | executeScript string | executeScript files/func |
| CSP | Customizable | Restricted |
| Action | browser_action/page_action | action |

---

## Manifest Changes

### Basic Manifest Update

**MV2:**
```json
{
  "manifest_version": 2,
  "name": "My Extension",
  "version": "1.0",
  "browser_action": {
    "default_popup": "popup.html",
    "default_icon": "icon.png"
  },
  "background": {
    "scripts": ["background.js"],
    "persistent": false
  },
  "permissions": [
    "tabs",
    "storage",
    "*://*.example.com/*"
  ]
}
```

**MV3:**
```json
{
  "manifest_version": 3,
  "name": "My Extension",
  "version": "1.0",
  "action": {
    "default_popup": "popup.html",
    "default_icon": "icon.png"
  },
  "background": {
    "service_worker": "background.js",
    "type": "module"
  },
  "permissions": [
    "tabs",
    "storage"
  ],
  "host_permissions": [
    "*://*.example.com/*"
  ]
}
```

### Key Changes

1. `browser_action` / `page_action` → `action`
2. `background.scripts` → `background.service_worker`
3. Host permissions moved to `host_permissions`
4. Add `"type": "module"` for ES modules

---

## Background Page → Service Worker

### Key Differences

| Background Page | Service Worker |
|-----------------|----------------|
| Persistent (optional) | Always terminates |
| DOM access | No DOM |
| window object | No window |
| localStorage | No localStorage |
| XMLHttpRequest | fetch only |
| setTimeout reliable | setTimeout may not fire |

### Migration Steps

**1. Remove DOM dependencies:**

```javascript
// MV2 - Using DOM
const parser = new DOMParser();
const doc = parser.parseFromString(html, 'text/html');

// MV3 - Use offscreen document
await chrome.offscreen.createDocument({
  url: 'offscreen.html',
  reasons: ['DOM_PARSER'],
  justification: 'Parse HTML content'
});
// Send HTML to offscreen document for parsing
```

**2. Replace window with self:**

```javascript
// MV2
window.addEventListener('message', handler);

// MV3
self.addEventListener('message', handler);
```

**3. Handle termination:**

```javascript
// MV2 - Persistent state
let cachedData = null;

// MV3 - Must persist to storage
async function getCachedData() {
  const { cachedData } = await chrome.storage.session.get('cachedData');
  return cachedData;
}

async function setCachedData(data) {
  await chrome.storage.session.set({ cachedData: data });
}
```

**4. Register listeners at top level:**

```javascript
// MV2 - Could add listeners anytime
setTimeout(() => {
  chrome.runtime.onMessage.addListener(handler);
}, 1000);

// MV3 - Must be at top level
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  // Can still handle asynchronously
  handleMessage(message).then(sendResponse);
  return true;
});
```

---

## Persistence Patterns

### Using Alarms

```javascript
// MV2 - setInterval
setInterval(checkForUpdates, 60000);

// MV3 - Alarms
chrome.alarms.create('checkUpdates', { periodInMinutes: 1 });

chrome.alarms.onAlarm.addListener((alarm) => {
  if (alarm.name === 'checkUpdates') {
    checkForUpdates();
  }
});
```

### State Persistence

```javascript
// Service worker can terminate anytime
// Must save state to storage

// On state change
async function updateState(newState) {
  state = { ...state, ...newState };
  await chrome.storage.session.set({ state });
}

// On service worker start
async function restoreState() {
  const { state } = await chrome.storage.session.get('state');
  return state || initialState;
}

// Restore state immediately on load
let state;
restoreState().then(s => state = s);
```

### Keep-Alive Patterns (Use Sparingly)

```javascript
// For long-running operations
// Create an alarm to keep service worker alive
async function startLongOperation() {
  chrome.alarms.create('keepAlive', { periodInMinutes: 0.5 });

  try {
    await longOperation();
  } finally {
    chrome.alarms.clear('keepAlive');
  }
}

chrome.alarms.onAlarm.addListener((alarm) => {
  if (alarm.name === 'keepAlive') {
    // Just keeps worker alive
  }
});
```

---

## webRequest → declarativeNetRequest

### Migration Comparison

**MV2 - webRequest (blocking):**
```javascript
chrome.webRequest.onBeforeRequest.addListener(
  (details) => {
    if (shouldBlock(details.url)) {
      return { cancel: true };
    }
  },
  { urls: ['<all_urls>'] },
  ['blocking']
);
```

**MV3 - declarativeNetRequest:**

**manifest.json:**
```json
{
  "permissions": ["declarativeNetRequest"],
  "declarative_net_request": {
    "rule_resources": [{
      "id": "ruleset_1",
      "enabled": true,
      "path": "rules.json"
    }]
  }
}
```

**rules.json:**
```json
[
  {
    "id": 1,
    "priority": 1,
    "action": { "type": "block" },
    "condition": {
      "urlFilter": "||ads.example.com",
      "resourceTypes": ["script", "image", "xmlhttprequest"]
    }
  },
  {
    "id": 2,
    "priority": 1,
    "action": {
      "type": "redirect",
      "redirect": { "extensionPath": "/blocked.html" }
    },
    "condition": {
      "urlFilter": "||tracking.com/*",
      "resourceTypes": ["main_frame"]
    }
  }
]
```

### Dynamic Rules

```javascript
// Add rules at runtime
await chrome.declarativeNetRequest.updateDynamicRules({
  addRules: [{
    id: 100,
    priority: 1,
    action: { type: 'block' },
    condition: {
      urlFilter: userBlockedDomain,
      resourceTypes: ['main_frame', 'sub_frame']
    }
  }],
  removeRuleIds: [99]  // Remove old rule
});

// Session rules (cleared on browser restart)
await chrome.declarativeNetRequest.updateSessionRules({
  addRules: [/* ... */]
});
```

### When webRequest is Still Needed

Some use cases still require webRequest (without blocking):
- Observing requests (non-blocking)
- Reading response headers
- Authentication handling

```json
{
  "permissions": ["webRequest"],
  "host_permissions": ["*://*.example.com/*"]
}
```

```javascript
// Non-blocking observation still works
chrome.webRequest.onCompleted.addListener(
  (details) => {
    logRequest(details);
  },
  { urls: ['*://*.example.com/*'] }
);
```

---

## Remote Code Removal

### Identifying Remote Code

**Not Allowed in MV3:**
```javascript
// Loading external scripts
const script = document.createElement('script');
script.src = 'https://external.com/script.js';  // Blocked

// Eval and Function constructor
eval(code);  // Blocked
new Function(code);  // Blocked

// External script tags in HTML
<script src="https://cdn.example.com/lib.js"></script>  // Blocked
```

### Solutions

**1. Bundle all dependencies:**
```bash
npm install library
# Bundle with webpack/rollup/esbuild
```

**2. For dynamic configuration:**
```javascript
// MV2 - Fetch and eval config
const config = await fetch('https://api.com/config.js');
eval(config);

// MV3 - Fetch JSON data only
const response = await fetch('https://api.com/config.json');
const config = await response.json();
// Use config data, don't execute code
```

**3. For user scripts (sandbox):**
```json
{
  "sandbox": {
    "pages": ["sandbox.html"]
  }
}
```

```javascript
// sandbox.html can use eval
// Communicate via postMessage
const frame = document.getElementById('sandbox');
frame.contentWindow.postMessage({ code: userCode }, '*');
```

---

## executeScript Changes

### MV2 Style

```javascript
// Execute string of code
chrome.tabs.executeScript(tabId, {
  code: 'document.body.style.background = "red"'
});

// Execute file
chrome.tabs.executeScript(tabId, {
  file: 'content.js'
});
```

### MV3 Style

```javascript
// Execute function
await chrome.scripting.executeScript({
  target: { tabId },
  func: () => {
    document.body.style.background = 'red';
  }
});

// Execute function with arguments
await chrome.scripting.executeScript({
  target: { tabId },
  func: (color) => {
    document.body.style.background = color;
  },
  args: ['red']
});

// Execute file
await chrome.scripting.executeScript({
  target: { tabId },
  files: ['content.js']
});

// Specify world
await chrome.scripting.executeScript({
  target: { tabId },
  world: 'MAIN',  // Access page's JavaScript context
  func: () => window.somePageVariable
});
```

### insertCSS Changes

```javascript
// MV2
chrome.tabs.insertCSS(tabId, { code: 'body { color: red; }' });

// MV3
await chrome.scripting.insertCSS({
  target: { tabId },
  css: 'body { color: red; }'
});

// Or file
await chrome.scripting.insertCSS({
  target: { tabId },
  files: ['styles.css']
});
```

---

## Action API Changes

### browser_action / page_action → action

```javascript
// MV2
chrome.browserAction.setIcon({ path: 'icon.png' });
chrome.browserAction.setBadgeText({ text: '5' });
chrome.browserAction.onClicked.addListener(handler);

chrome.pageAction.show(tabId);

// MV3
chrome.action.setIcon({ path: 'icon.png' });
chrome.action.setBadgeText({ text: '5' });
chrome.action.onClicked.addListener(handler);

// No separate pageAction - use action for all
chrome.action.enable(tabId);
chrome.action.disable(tabId);
```

---

## Web Accessible Resources

### MV2

```json
{
  "web_accessible_resources": [
    "images/*",
    "script.js"
  ]
}
```

### MV3

```json
{
  "web_accessible_resources": [{
    "resources": ["images/*", "script.js"],
    "matches": ["*://*.example.com/*"]
  }, {
    "resources": ["public/*"],
    "matches": ["<all_urls>"],
    "use_dynamic_url": true
  }]
}
```

---

## Content Security Policy

### MV2 (Flexible)

```json
{
  "content_security_policy": "script-src 'self' 'unsafe-eval'; object-src 'self'"
}
```

### MV3 (Restricted)

```json
{
  "content_security_policy": {
    "extension_pages": "script-src 'self'; object-src 'self'"
  }
}
```

**Restrictions:**
- No `unsafe-eval`
- No `unsafe-inline`
- No remote script sources
- `wasm-unsafe-eval` allowed for WebAssembly

---

## Migration Checklist

### Manifest
- [ ] Change `manifest_version` to 3
- [ ] Replace `browser_action`/`page_action` with `action`
- [ ] Move host permissions to `host_permissions`
- [ ] Update `background` to `service_worker`
- [ ] Update `web_accessible_resources` format
- [ ] Remove forbidden CSP directives

### Background Script
- [ ] Convert to service worker
- [ ] Remove DOM dependencies
- [ ] Replace `window` with `self`
- [ ] Move state to storage
- [ ] Register listeners at top level
- [ ] Replace setInterval with alarms
- [ ] Handle service worker termination

### Content Scripts
- [ ] Update `executeScript` calls
- [ ] Update `insertCSS` calls
- [ ] Use `chrome.scripting` API

### Network
- [ ] Replace `webRequest` blocking with `declarativeNetRequest`
- [ ] Create static rule files
- [ ] Implement dynamic rules if needed

### Remote Code
- [ ] Bundle all external scripts
- [ ] Remove eval/Function usage
- [ ] Use sandbox for dynamic code
- [ ] Convert remote config to JSON

### Testing
- [ ] Test all features
- [ ] Verify service worker lifecycle
- [ ] Check permission functionality
- [ ] Test on multiple sites
- [ ] Verify no console errors

---

## Common Migration Issues

### Issue: Service Worker Terminates

**Problem:** State lost when worker terminates

**Solution:**
```javascript
// Use storage instead of variables
chrome.storage.session.set({ key: value });

// Restore on start
chrome.storage.session.get('key').then(({ key }) => {
  // Use restored value
});
```

### Issue: DOM Parser Needed

**Problem:** No DOMParser in service worker

**Solution:** Use offscreen document
```javascript
await chrome.offscreen.createDocument({
  url: 'offscreen.html',
  reasons: ['DOM_PARSER'],
  justification: 'Parse HTML'
});
```

### Issue: Blocking Requests

**Problem:** webRequest blocking not available

**Solution:** Use declarativeNetRequest with static rules

### Issue: Dynamic Code

**Problem:** Can't execute user-provided code

**Solution:** Sandbox page with postMessage communication

### Issue: External Libraries

**Problem:** Can't load from CDN

**Solution:** Bundle with npm/webpack

---
name: debugging-guide
description: Comprehensive debugging guide for Chrome Extensions covering DevTools usage, service worker inspection, content script debugging, storage inspection, network analysis, performance profiling, and common error solutions. Use when troubleshooting extension issues.
---

# Chrome Extension Debugging Guide

## Accessing DevTools

### Extension Management Page
1. Go to `chrome://extensions`
2. Enable **Developer mode** (top right toggle)
3. Click **Details** on your extension
4. Available debugging entry points:
   - **Service Worker** link → Service worker DevTools
   - **Errors** button → View runtime errors

### Different Context DevTools

| Component | How to Access |
|-----------|---------------|
| Service Worker | `chrome://extensions` → Click "Service Worker" link |
| Popup | Right-click popup → "Inspect" |
| Side Panel | Right-click side panel → "Inspect" |
| Options Page | Right-click page → "Inspect" |
| Content Script | Page DevTools → Console → Select extension context |

---

## Service Worker Debugging

### Opening Service Worker DevTools

1. Go to `chrome://extensions`
2. Find your extension
3. Click the "Service Worker" link (shows as "Inactive" or "Active")

### Service Worker Lifecycle

```javascript
// Add lifecycle logging
self.addEventListener('install', (event) => {
  console.log('[SW] Installing...');
});

self.addEventListener('activate', (event) => {
  console.log('[SW] Activated');
});

// Log when service worker starts
console.log('[SW] Service worker loaded at', new Date().toISOString());

// Monitor fetch events
self.addEventListener('fetch', (event) => {
  console.log('[SW] Fetch:', event.request.url);
});
```

### Common Service Worker Issues

**Issue: Service Worker Not Loading**
```javascript
// Check manifest.json
{
  "background": {
    "service_worker": "background.js",  // Verify path
    "type": "module"  // Add if using ES imports
  }
}

// Check for syntax errors
// Open chrome://extensions and look for error badge
```

**Issue: Service Worker Terminating Early**
```javascript
// Keep alive with alarms (not recommended for all cases)
chrome.alarms.create('keepAlive', { periodInMinutes: 1 });

// Better: Design for termination
// - Store state in chrome.storage
// - Re-register listeners at top level
// - Don't rely on global variables
```

**Issue: Events Not Firing**
```javascript
// WRONG - Listener added async
setTimeout(() => {
  chrome.runtime.onMessage.addListener(handler);
}, 1000);

// CORRECT - Top-level registration
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  // Handle asynchronously inside
  handleAsync(message).then(sendResponse);
  return true;
});
```

---

## Content Script Debugging

### Finding Content Script Console

1. Open DevTools on the web page (F12)
2. Click the **Console** tab
3. Click the dropdown next to "top" (usually says "top")
4. Select your extension's context

### Content Script Debugging Tips

```javascript
// Verify injection
console.log('[Content] Script loaded on:', window.location.href);
console.log('[Content] Document state:', document.readyState);

// Check for multiple injections
if (window.__myExtensionLoaded) {
  console.warn('[Content] Already loaded!');
} else {
  window.__myExtensionLoaded = true;
  // Initialize
}

// Debug DOM queries
const element = document.querySelector('#target');
console.log('[Content] Target element:', element);
if (!element) {
  console.log('[Content] Available elements:', document.body.innerHTML.slice(0, 500));
}
```

### Content Script Injection Failures

```javascript
// Check manifest matches
{
  "content_scripts": [{
    "matches": ["*://*.example.com/*"],  // Check pattern
    "js": ["content.js"],
    "run_at": "document_idle"
  }]
}

// Verify URL matches pattern
// chrome://extensions - Content scripts won't run on chrome:// URLs

// Programmatic injection alternative
chrome.scripting.executeScript({
  target: { tabId },
  files: ['content.js']
}).catch(error => {
  console.error('Injection failed:', error);
  // Common errors:
  // - "Cannot access chrome:// URLs"
  // - "Missing host_permissions"
});
```

---

## Message Passing Debugging

### Logging All Messages

```javascript
// background.js
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  console.log('[BG] Message received:', {
    message,
    from: sender.tab ? `Tab ${sender.tab.id}` : 'Extension',
    url: sender.url,
    frameId: sender.frameId
  });

  // Your handling logic
  return true;
});

// content.js
const originalSendMessage = chrome.runtime.sendMessage;
chrome.runtime.sendMessage = function(message, ...args) {
  console.log('[CS] Sending message:', message);
  return originalSendMessage.call(this, message, ...args);
};
```

### Common Message Errors

**"Receiving end does not exist"**
```javascript
// The target doesn't have a listener
// Solutions:
// 1. Content script not injected
// 2. Tab navigated away
// 3. Extension context invalidated

async function safeSend(tabId, message) {
  try {
    return await chrome.tabs.sendMessage(tabId, message);
  } catch (error) {
    if (error.message.includes('Receiving end does not exist')) {
      // Inject content script first
      await chrome.scripting.executeScript({
        target: { tabId },
        files: ['content.js']
      });
      return await chrome.tabs.sendMessage(tabId, message);
    }
    throw error;
  }
}
```

**Async Response Not Working**
```javascript
// WRONG - Missing return true
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  fetchData().then(sendResponse);
  // Missing return true!
});

// CORRECT
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  fetchData().then(sendResponse);
  return true;  // CRITICAL for async
});
```

---

## Storage Debugging

### Inspecting Storage

```javascript
// Dump all storage contents
async function debugStorage() {
  const local = await chrome.storage.local.get(null);
  const sync = await chrome.storage.sync.get(null);
  const session = await chrome.storage.session.get(null);

  console.group('Storage Contents');
  console.log('Local:', local);
  console.log('Sync:', sync);
  console.log('Session:', session);
  console.groupEnd();

  // Check quotas
  const localBytes = await chrome.storage.local.getBytesInUse(null);
  const syncBytes = await chrome.storage.sync.getBytesInUse(null);

  console.group('Storage Usage');
  console.log(`Local: ${localBytes} / 10,485,760 bytes`);
  console.log(`Sync: ${syncBytes} / 102,400 bytes`);
  console.groupEnd();
}

// Call from DevTools console
debugStorage();
```

### Storage Change Monitoring

```javascript
// Add to background.js for debugging
chrome.storage.onChanged.addListener((changes, areaName) => {
  console.group(`Storage changed [${areaName}]`);
  for (const [key, { oldValue, newValue }] of Object.entries(changes)) {
    console.log(`${key}:`, oldValue, '→', newValue);
  }
  console.groupEnd();
});
```

### Application Panel

1. Open DevTools
2. Go to **Application** tab
3. Expand **Storage** → **Extension storage**
4. View local/sync/session storage contents

---

## Network Debugging

### Network Panel

1. Open DevTools on the relevant context
2. Go to **Network** tab
3. Filter by extension requests

### Logging Network Requests

```javascript
// Wrap fetch for debugging
const originalFetch = fetch;
window.fetch = async function(...args) {
  console.log('[Fetch]', args[0], args[1]);
  try {
    const response = await originalFetch.apply(this, args);
    console.log('[Fetch Response]', response.status, response.url);
    return response;
  } catch (error) {
    console.error('[Fetch Error]', error);
    throw error;
  }
};
```

### declarativeNetRequest Debugging

```javascript
// Enable rule matching display
chrome.declarativeNetRequest.setExtensionActionOptions({
  displayActionCountAsBadgeText: true
});

// Get matched rules for a tab
const rules = await chrome.declarativeNetRequest.getMatchedRules({
  tabId: tab.id
});
console.log('Matched rules:', rules);

// Test a URL against rules
const result = await chrome.declarativeNetRequest.testMatchOutcome({
  url: 'https://example.com/script.js',
  type: 'script',
  initiator: 'https://example.com'
});
console.log('Would match:', result);
```

---

## Performance Debugging

### Performance Timing

```javascript
// Measure operation time
console.time('operation');
await someOperation();
console.timeEnd('operation');

// Performance marks
performance.mark('start');
await someOperation();
performance.mark('end');
performance.measure('operation', 'start', 'end');
console.log(performance.getEntriesByName('operation'));
```

### Memory Profiling

1. Open DevTools for component
2. Go to **Memory** tab
3. Take heap snapshot
4. Compare snapshots to find leaks

```javascript
// Force garbage collection (DevTools must be open)
// Click the trash can icon or use:
// Settings → Experiments → Enable "Timeline: show runtime call stats"
```

### Performance Panel

1. Open DevTools
2. Go to **Performance** tab
3. Click Record
4. Perform actions
5. Stop recording
6. Analyze flame chart

---

## Common Errors & Solutions

### "Extension context invalidated"

**Cause:** Extension was reloaded/updated while script was running

**Solution:**
```javascript
// Check before Chrome API calls
function isContextValid() {
  try {
    chrome.runtime.id;
    return true;
  } catch {
    return false;
  }
}

// Notify user
if (!isContextValid()) {
  showNotification('Extension updated. Please refresh the page.');
}
```

### "Cannot read properties of undefined"

**Cause:** API doesn't exist or lacks permission

```javascript
// Safe API access
if (chrome.sidePanel) {
  chrome.sidePanel.open({ tabId });
} else {
  console.warn('sidePanel API not available');
}

// Check manifest permissions
console.log('Permissions:', chrome.runtime.getManifest().permissions);
```

### "Uncaught (in promise)"

**Cause:** Unhandled promise rejection

```javascript
// Always add catch
chrome.storage.local.get('key')
  .then(handleResult)
  .catch(handleError);

// Or use try-catch with async/await
try {
  const result = await chrome.storage.local.get('key');
} catch (error) {
  console.error('Storage error:', error);
}
```

### lastError Handling

```javascript
// Callback style - check lastError
chrome.tabs.sendMessage(tabId, message, (response) => {
  if (chrome.runtime.lastError) {
    console.error('Error:', chrome.runtime.lastError.message);
    return;
  }
  // Process response
});

// Promise style - use try-catch
try {
  const response = await chrome.tabs.sendMessage(tabId, message);
} catch (error) {
  console.error('Error:', error.message);
}
```

---

## Debugging Checklist

### Quick Diagnosis
- [ ] Check `chrome://extensions` for error badge
- [ ] Click "Errors" to see details
- [ ] Reload extension after changes
- [ ] Clear storage and retry

### Service Worker
- [ ] Service worker link shows "Active"
- [ ] Console shows no errors
- [ ] Listeners registered at top level
- [ ] State persisted to storage

### Content Scripts
- [ ] Correct console context selected
- [ ] URL matches manifest pattern
- [ ] Script confirms injection
- [ ] No CSP violations

### Messages
- [ ] Sender/receiver both have listeners
- [ ] `return true` for async responses
- [ ] Check `chrome.runtime.lastError`
- [ ] Tab ID is correct

### Storage
- [ ] Correct storage area used
- [ ] Within quota limits
- [ ] Keys spelled correctly
- [ ] Data serializable (no functions)

---

## Hot Reload Development

```javascript
// Development helper in background.js
if (process.env.NODE_ENV === 'development') {
  // Watch for file changes
  const ws = new WebSocket('ws://localhost:8080');
  ws.onmessage = (event) => {
    if (event.data === 'reload') {
      chrome.runtime.reload();
    }
  };

  // Or simpler: manual reload
  console.log('Press F5 in chrome://extensions to reload');
}
```

### File Watcher Script

```javascript
// watch.js (Node.js)
const WebSocket = require('ws');
const chokidar = require('chokidar');

const wss = new WebSocket.Server({ port: 8080 });

chokidar.watch('./src').on('change', () => {
  wss.clients.forEach(client => {
    client.send('reload');
  });
});
```

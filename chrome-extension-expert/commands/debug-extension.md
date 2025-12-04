---
name: debug-extension
description: Guided debugging for common Chrome Extension issues including service worker, message passing, content script, and storage problems
arguments:
  - name: issue
    description: Type of issue to debug (service-worker, messaging, content-script, storage, side-panel, general)
    required: false
    default: general
---

# Debug Chrome Extension

Guided debugging for issue type: **{{issue}}**

## Instructions

Based on the issue type, follow the appropriate debugging guide and analyze the extension code.

---

## Service Worker Issues ({{issue}} = service-worker)

### Common Problems

**1. Service Worker Not Registering**
Check:
- [ ] manifest.json has correct `background.service_worker` path
- [ ] Service worker file exists at specified path
- [ ] No syntax errors in service worker file
- [ ] Using `type: "module"` if using ES imports

```json
// Correct manifest configuration
{
  "background": {
    "service_worker": "background.js",
    "type": "module"  // Only if using ES modules
  }
}
```

**2. Service Worker Terminating**
- Service workers terminate after ~30 seconds of inactivity
- Use `chrome.alarms` for periodic tasks
- Don't rely on global variables persisting

```javascript
// BAD - Will lose state
let cachedData = null;

// GOOD - Persist to storage
chrome.storage.session.set({ cachedData: data });
```

**3. Events Not Firing**
- Listeners must be registered synchronously at top level
- Can't add listeners in async callbacks

```javascript
// BAD - Won't work
setTimeout(() => {
  chrome.runtime.onMessage.addListener(...);
}, 1000);

// GOOD - Top level registration
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  // Handle asynchronously inside
});
```

### Debugging Steps
1. Go to `chrome://extensions`
2. Find your extension
3. Click "Service Worker" link
4. Check console for errors
5. Set breakpoints in Sources panel

---

## Message Passing Issues ({{issue}} = messaging)

### Common Problems

**1. "Receiving end does not exist"**
Causes:
- Content script not injected
- Tab navigated away
- Extension context invalidated

```javascript
// Safe message sending
async function sendToTab(tabId, message) {
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

**2. Async Response Not Working**
Must return `true` from listener for async responses:

```javascript
// WRONG - Response never sent
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  fetch(url).then(data => sendResponse(data));
  // Missing return true!
});

// CORRECT
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  fetch(url).then(data => sendResponse(data));
  return true; // CRITICAL for async
});
```

**3. Port Disconnected**
Long-lived connections close after 5 minutes of inactivity or when the other end closes.

### Debugging Steps
1. Add logging to both sender and receiver
2. Check sender ID matches extension ID
3. Verify content script is injected (check page console)
4. Check for `chrome.runtime.lastError`

---

## Content Script Issues ({{issue}} = content-script)

### Common Problems

**1. Script Not Injecting**
Check:
- [ ] `matches` pattern in manifest is correct
- [ ] URL matches the pattern
- [ ] Page isn't chrome://, chrome-extension://, or other restricted URL

```json
{
  "content_scripts": [{
    "matches": ["*://*.example.com/*"],
    "js": ["content.js"],
    "run_at": "document_idle"
  }]
}
```

**2. Script Runs But Nothing Happens**
- Check correct console context (select extension from dropdown)
- Verify DOM is ready
- Check for CSS conflicts

```javascript
// Ensure DOM is ready
if (document.readyState === 'loading') {
  document.addEventListener('DOMContentLoaded', init);
} else {
  init();
}
```

**3. Cross-Origin Restrictions**
Content scripts can't make cross-origin requests. Use background script as relay.

### Debugging Steps
1. Open page DevTools (F12)
2. Go to Console tab
3. Select extension context from dropdown
4. Check for errors
5. Verify script loaded with console.log at top

---

## Storage Issues ({{issue}} = storage)

### Common Problems

**1. Data Not Persisting**
```javascript
// Check what's actually stored
const all = await chrome.storage.local.get(null);
console.log('All storage:', all);
```

**2. Sync Storage Quota Exceeded**
- Sync: 102,400 bytes total, 8,192 bytes per item
- Local: 10,485,760 bytes

```javascript
// Check usage
const usage = await chrome.storage.sync.getBytesInUse(null);
console.log('Sync usage:', usage, '/ 102400');
```

**3. Storage Not Syncing**
- User must be signed into Chrome
- Sync must be enabled
- Check `chrome.storage.sync` specifically

### Debugging Steps
1. Open extension DevTools
2. Go to Application > Storage
3. Check chrome.storage.local and chrome.storage.sync
4. Monitor with `chrome.storage.onChanged`

---

## Side Panel Issues ({{issue}} = side-panel)

### Common Problems

**1. Side Panel Not Appearing**
Check:
- [ ] `sidePanel` permission in manifest
- [ ] `side_panel.default_path` set correctly
- [ ] HTML file exists

```json
{
  "permissions": ["sidePanel"],
  "side_panel": {
    "default_path": "sidepanel.html"
  }
}
```

**2. Can't Open Programmatically**
```javascript
// Must be in response to user action
chrome.action.onClicked.addListener((tab) => {
  chrome.sidePanel.open({ tabId: tab.id });
});
```

**3. State Lost When Panel Closes**
Side panel is recreated each time. Use storage:
```javascript
// Save state before unload
window.addEventListener('beforeunload', () => {
  chrome.storage.session.set({ panelState: currentState });
});

// Restore on load
const { panelState } = await chrome.storage.session.get('panelState');
```

---

## General Debugging ({{issue}} = general)

### Quick Diagnosis
1. Check `chrome://extensions` for errors (red badge)
2. Click "Errors" to see details
3. Reload extension after changes
4. Check all relevant consoles:
   - Service worker console
   - Popup/side panel DevTools
   - Page console (content scripts)

### Common Fixes
- Clear extension storage and reload
- Uninstall and reinstall extension
- Check for typos in file paths
- Verify all referenced files exist
- Check permissions are correct

---

Now analyze the extension code for {{issue}} issues and provide specific debugging recommendations.

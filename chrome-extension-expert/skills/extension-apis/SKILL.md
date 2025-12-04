---
name: extension-apis
description: Chrome Extension APIs reference covering runtime, storage, tabs, scripting, action, alarms, notifications, contextMenus, sidePanel, offscreen, and identity APIs. Use when implementing Chrome extension functionality.
---

# Chrome Extension APIs Reference

## chrome.runtime

### Lifecycle Events

```javascript
// Extension installed or updated
chrome.runtime.onInstalled.addListener((details) => {
  if (details.reason === 'install') {
    // First install
  } else if (details.reason === 'update') {
    // Updated from details.previousVersion
  }
});

// Browser started with extension enabled
chrome.runtime.onStartup.addListener(() => {
  // Browser startup
});

// Extension suspended (MV3)
chrome.runtime.onSuspend.addListener(() => {
  // Clean up before suspension
});
```

### Messaging

```javascript
// Send message to background
const response = await chrome.runtime.sendMessage({
  type: 'ACTION',
  data: payload
});

// Listen for messages
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  console.log('From:', sender.tab?.id || 'extension');

  // Async response - MUST return true
  handleAsync(message).then(sendResponse);
  return true;
});

// Connect for long-lived connection
const port = chrome.runtime.connect({ name: 'channel' });
port.postMessage({ type: 'HELLO' });
port.onMessage.addListener((message) => {
  console.log('Received:', message);
});
port.onDisconnect.addListener(() => {
  console.log('Port disconnected');
});
```

### Extension Info

```javascript
// Get extension info
const manifest = chrome.runtime.getManifest();
const id = chrome.runtime.id;
const url = chrome.runtime.getURL('page.html');

// Reload extension
chrome.runtime.reload();

// Open options page
chrome.runtime.openOptionsPage();
```

---

## chrome.storage

### Storage Areas

```javascript
// Local storage (10MB limit, persistent)
await chrome.storage.local.set({ key: 'value' });
const { key } = await chrome.storage.local.get('key');
const all = await chrome.storage.local.get(null);
await chrome.storage.local.remove('key');
await chrome.storage.local.clear();

// Sync storage (100KB limit, synced across devices)
await chrome.storage.sync.set({ settings: { theme: 'dark' } });
const { settings } = await chrome.storage.sync.get('settings');

// Session storage (cleared on browser restart)
await chrome.storage.session.set({ tempData: data });
const { tempData } = await chrome.storage.session.get('tempData');

// Check usage
const bytes = await chrome.storage.local.getBytesInUse(null);
const syncBytes = await chrome.storage.sync.getBytesInUse(null);
```

### Storage Limits

| Area | Total Limit | Per Item |
|------|-------------|----------|
| local | 10 MB | Unlimited |
| sync | 102,400 bytes | 8,192 bytes |
| session | 10 MB | Unlimited |

### Change Listener

```javascript
chrome.storage.onChanged.addListener((changes, areaName) => {
  for (const [key, { oldValue, newValue }] of Object.entries(changes)) {
    console.log(`${areaName}.${key}: ${oldValue} → ${newValue}`);
  }
});
```

---

## chrome.tabs

### Query Tabs

```javascript
// Get current tab
const [tab] = await chrome.tabs.query({
  active: true,
  currentWindow: true
});

// Get all tabs
const tabs = await chrome.tabs.query({});

// Query with filters
const tabs = await chrome.tabs.query({
  url: '*://*.example.com/*',
  status: 'complete',
  pinned: false
});
```

### Tab Operations

```javascript
// Create new tab
const tab = await chrome.tabs.create({
  url: 'https://example.com',
  active: true,
  index: 0
});

// Update tab
await chrome.tabs.update(tabId, {
  url: 'https://new-url.com',
  active: true,
  pinned: true
});

// Remove tab
await chrome.tabs.remove(tabId);
await chrome.tabs.remove([tabId1, tabId2]);

// Reload tab
await chrome.tabs.reload(tabId);

// Duplicate tab
const newTab = await chrome.tabs.duplicate(tabId);

// Move tab
await chrome.tabs.move(tabId, { index: 0 });

// Get tab
const tab = await chrome.tabs.get(tabId);
```

### Tab Messaging

```javascript
// Send message to content script in tab
try {
  const response = await chrome.tabs.sendMessage(tabId, {
    type: 'GET_DATA'
  });
} catch (error) {
  // Content script not loaded
}

// Send to specific frame
await chrome.tabs.sendMessage(tabId, message, { frameId: 0 });
```

### Tab Events

```javascript
chrome.tabs.onCreated.addListener((tab) => {});
chrome.tabs.onUpdated.addListener((tabId, changeInfo, tab) => {
  if (changeInfo.status === 'complete') {
    // Page loaded
  }
});
chrome.tabs.onRemoved.addListener((tabId, removeInfo) => {});
chrome.tabs.onActivated.addListener(({ tabId, windowId }) => {});
```

---

## chrome.scripting

### Execute Script

```javascript
// Execute function in tab
const results = await chrome.scripting.executeScript({
  target: { tabId },
  func: () => document.title
});
console.log(results[0].result);

// Execute with arguments
await chrome.scripting.executeScript({
  target: { tabId },
  func: (arg1, arg2) => {
    console.log(arg1, arg2);
  },
  args: ['hello', 'world']
});

// Execute file
await chrome.scripting.executeScript({
  target: { tabId },
  files: ['content.js']
});

// Execute in specific frames
await chrome.scripting.executeScript({
  target: { tabId, frameIds: [0] },
  files: ['content.js']
});

// Execute in all frames
await chrome.scripting.executeScript({
  target: { tabId, allFrames: true },
  files: ['content.js']
});

// Execute in main world (access page variables)
await chrome.scripting.executeScript({
  target: { tabId },
  world: 'MAIN',
  func: () => window.somePageVariable
});
```

### Insert CSS

```javascript
// Insert CSS string
await chrome.scripting.insertCSS({
  target: { tabId },
  css: 'body { background: red !important; }'
});

// Insert CSS file
await chrome.scripting.insertCSS({
  target: { tabId },
  files: ['styles.css']
});

// Remove CSS
await chrome.scripting.removeCSS({
  target: { tabId },
  css: 'body { background: red !important; }'
});
```

### Register Content Scripts

```javascript
// Register dynamic content script
await chrome.scripting.registerContentScripts([{
  id: 'my-script',
  matches: ['*://*.example.com/*'],
  js: ['content.js'],
  runAt: 'document_idle'
}]);

// Update registered script
await chrome.scripting.updateContentScripts([{
  id: 'my-script',
  matches: ['*://*.new-domain.com/*']
}]);

// Unregister
await chrome.scripting.unregisterContentScripts({
  ids: ['my-script']
});

// Get registered scripts
const scripts = await chrome.scripting.getRegisteredContentScripts();
```

---

## chrome.action

```javascript
// Set popup
chrome.action.setPopup({ popup: 'popup.html' });
chrome.action.setPopup({ tabId, popup: 'tab-popup.html' });

// Set icon
chrome.action.setIcon({
  path: 'icons/active.png'
});
chrome.action.setIcon({
  tabId,
  path: { 16: 'icon16.png', 32: 'icon32.png' }
});

// Set badge
chrome.action.setBadgeText({ text: '5' });
chrome.action.setBadgeText({ tabId, text: '!' });
chrome.action.setBadgeBackgroundColor({ color: '#FF0000' });
chrome.action.setBadgeTextColor({ color: '#FFFFFF' });

// Set title (tooltip)
chrome.action.setTitle({ title: 'Custom tooltip' });

// Enable/disable
chrome.action.enable(tabId);
chrome.action.disable(tabId);

// Click handler (when no popup)
chrome.action.onClicked.addListener((tab) => {
  // Handle click
});

// Open popup programmatically (Chrome 127+)
await chrome.action.openPopup();
```

---

## chrome.alarms

```javascript
// Create alarm
chrome.alarms.create('myAlarm', {
  delayInMinutes: 1,           // First trigger
  periodInMinutes: 5           // Repeat interval
});

chrome.alarms.create('oneTime', {
  when: Date.now() + 60000     // Specific time (ms)
});

// Listen for alarm
chrome.alarms.onAlarm.addListener((alarm) => {
  if (alarm.name === 'myAlarm') {
    // Handle alarm
  }
});

// Get alarms
const alarm = await chrome.alarms.get('myAlarm');
const allAlarms = await chrome.alarms.getAll();

// Clear alarms
await chrome.alarms.clear('myAlarm');
await chrome.alarms.clearAll();
```

---

## chrome.notifications

```javascript
// Create notification
const notificationId = await chrome.notifications.create({
  type: 'basic',
  iconUrl: 'icon128.png',
  title: 'Notification Title',
  message: 'Notification message',
  priority: 2,
  buttons: [
    { title: 'Button 1' },
    { title: 'Button 2' }
  ]
});

// Progress notification
await chrome.notifications.create({
  type: 'progress',
  iconUrl: 'icon128.png',
  title: 'Downloading',
  message: 'Please wait...',
  progress: 50
});

// Update notification
await chrome.notifications.update(notificationId, {
  progress: 100,
  message: 'Complete!'
});

// Clear notification
await chrome.notifications.clear(notificationId);

// Event listeners
chrome.notifications.onClicked.addListener((notificationId) => {});
chrome.notifications.onButtonClicked.addListener((notificationId, buttonIndex) => {});
chrome.notifications.onClosed.addListener((notificationId, byUser) => {});
```

---

## chrome.contextMenus

```javascript
// Create context menu
chrome.contextMenus.create({
  id: 'myMenu',
  title: 'Do something with "%s"',
  contexts: ['selection', 'link', 'image'],
  documentUrlPatterns: ['*://*.example.com/*']
});

// Nested menu
chrome.contextMenus.create({
  id: 'parent',
  title: 'Parent Menu',
  contexts: ['all']
});
chrome.contextMenus.create({
  id: 'child',
  parentId: 'parent',
  title: 'Child Item',
  contexts: ['all']
});

// Handle click
chrome.contextMenus.onClicked.addListener((info, tab) => {
  if (info.menuItemId === 'myMenu') {
    console.log('Selected text:', info.selectionText);
    console.log('Link URL:', info.linkUrl);
    console.log('Image URL:', info.srcUrl);
  }
});

// Update menu
chrome.contextMenus.update('myMenu', { title: 'New Title' });

// Remove menu
chrome.contextMenus.remove('myMenu');
chrome.contextMenus.removeAll();
```

### Context Types
`all`, `page`, `frame`, `selection`, `link`, `editable`, `image`, `video`, `audio`, `launcher`, `browser_action`, `page_action`, `action`

---

## chrome.sidePanel

```javascript
// Open side panel
await chrome.sidePanel.open({ tabId: tab.id });
await chrome.sidePanel.open({ windowId: window.id });

// Set panel options
await chrome.sidePanel.setOptions({
  tabId,
  path: 'sidepanel.html',
  enabled: true
});

// Get panel options
const options = await chrome.sidePanel.getOptions({ tabId });

// Set panel behavior
await chrome.sidePanel.setPanelBehavior({
  openPanelOnActionClick: true
});
```

---

## chrome.offscreen

For tasks requiring DOM but not visible UI.

```javascript
// Create offscreen document
await chrome.offscreen.createDocument({
  url: 'offscreen.html',
  reasons: ['DOM_SCRAPING', 'AUDIO_PLAYBACK'],
  justification: 'Parse HTML content'
});

// Check if exists
const existing = await chrome.offscreen.hasDocument();

// Close offscreen document
await chrome.offscreen.closeDocument();
```

### Reasons
`TESTING`, `AUDIO_PLAYBACK`, `IFRAME_SCRIPTING`, `DOM_SCRAPING`, `BLOBS`, `DOM_PARSER`, `USER_MEDIA`, `DISPLAY_MEDIA`, `WEB_RTC`, `CLIPBOARD`, `LOCAL_STORAGE`, `WORKERS`, `BATTERY_STATUS`, `MATCH_MEDIA`, `GEOLOCATION`

---

## chrome.identity

```javascript
// OAuth2 authentication
const token = await chrome.identity.getAuthToken({
  interactive: true
});

// Launch web auth flow
const responseUrl = await chrome.identity.launchWebAuthFlow({
  url: 'https://auth.provider.com/oauth?...',
  interactive: true
});

// Remove cached token
await chrome.identity.removeCachedAuthToken({ token });

// Get profile info
const userInfo = await chrome.identity.getProfileUserInfo({
  accountStatus: 'ANY'
});
```

---

## chrome.permissions

```javascript
// Check permissions
const hasPermission = await chrome.permissions.contains({
  permissions: ['tabs'],
  origins: ['*://*.example.com/*']
});

// Request permissions (must be user gesture)
const granted = await chrome.permissions.request({
  permissions: ['history'],
  origins: ['*://*.new-site.com/*']
});

// Remove permissions
await chrome.permissions.remove({
  permissions: ['history']
});

// Get all permissions
const all = await chrome.permissions.getAll();

// Listen for changes
chrome.permissions.onAdded.addListener((permissions) => {});
chrome.permissions.onRemoved.addListener((permissions) => {});
```

---

## chrome.declarativeNetRequest

```javascript
// Update dynamic rules
await chrome.declarativeNetRequest.updateDynamicRules({
  removeRuleIds: [1],
  addRules: [{
    id: 2,
    priority: 1,
    action: { type: 'block' },
    condition: {
      urlFilter: '*://ads.example.com/*',
      resourceTypes: ['script', 'image']
    }
  }]
});

// Get rules
const rules = await chrome.declarativeNetRequest.getDynamicRules();
const sessionRules = await chrome.declarativeNetRequest.getSessionRules();

// Test rules
const result = await chrome.declarativeNetRequest.testMatchOutcome({
  url: 'https://ads.example.com/script.js',
  type: 'script'
});

// Enable/disable rulesets
await chrome.declarativeNetRequest.updateEnabledRulesets({
  enableRulesetIds: ['ruleset_1'],
  disableRulesetIds: ['ruleset_2']
});
```

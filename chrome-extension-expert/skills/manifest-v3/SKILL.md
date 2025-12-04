---
name: manifest-v3
description: Comprehensive Manifest V3 reference covering manifest.json structure, all fields, service workers, content scripts, permissions, CSP, icons, and side panel configuration. Use when creating or modifying Chrome Extension manifests.
---

# Manifest V3 Complete Reference

## Basic Structure

```json
{
  "manifest_version": 3,
  "name": "Extension Name",
  "version": "1.0.0",
  "description": "Brief description (max 132 chars for store)"
}
```

## Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `manifest_version` | integer | Must be `3` |
| `name` | string | Extension name (max 45 chars) |
| `version` | string | Version string (e.g., "1.0.0") |

## Recommended Fields

| Field | Type | Description |
|-------|------|-------------|
| `description` | string | Brief description |
| `icons` | object | Extension icons |
| `action` | object | Toolbar button config |

---

## Icons

```json
{
  "icons": {
    "16": "icons/icon16.png",
    "32": "icons/icon32.png",
    "48": "icons/icon48.png",
    "128": "icons/icon128.png"
  }
}
```

| Size | Usage |
|------|-------|
| 16x16 | Favicon, toolbar |
| 32x32 | Windows computers |
| 48x48 | Extensions management page |
| 128x128 | Chrome Web Store |

---

## Background Service Worker

```json
{
  "background": {
    "service_worker": "background.js",
    "type": "module"
  }
}
```

| Property | Description |
|----------|-------------|
| `service_worker` | Path to service worker file |
| `type` | Set to `"module"` for ES module support |

**Key Differences from MV2:**
- No persistent background pages
- Service worker terminates after ~30s idle
- Must use `chrome.alarms` for periodic tasks
- Event listeners must be registered at top level

---

## Action (Toolbar Button)

```json
{
  "action": {
    "default_icon": {
      "16": "icons/icon16.png",
      "32": "icons/icon32.png"
    },
    "default_title": "Click to activate",
    "default_popup": "popup.html"
  }
}
```

| Property | Description |
|----------|-------------|
| `default_icon` | Icon shown in toolbar |
| `default_title` | Tooltip text |
| `default_popup` | Popup HTML file |

**Note:** Replaces both `browser_action` and `page_action` from MV2.

---

## Content Scripts

```json
{
  "content_scripts": [
    {
      "matches": ["*://*.example.com/*"],
      "js": ["content.js"],
      "css": ["content.css"],
      "run_at": "document_idle",
      "all_frames": false,
      "match_about_blank": false,
      "match_origin_as_fallback": false
    }
  ]
}
```

| Property | Description |
|----------|-------------|
| `matches` | URL patterns to match (required) |
| `exclude_matches` | URL patterns to exclude |
| `js` | JavaScript files to inject |
| `css` | CSS files to inject |
| `run_at` | When to inject (`document_start`, `document_end`, `document_idle`) |
| `all_frames` | Inject into all frames |
| `match_about_blank` | Match about:blank frames |
| `world` | Execution world (`ISOLATED` or `MAIN`) |

### Match Patterns

```
<scheme>://<host><path>

Examples:
*://*.google.com/*       - All Google domains
https://example.com/*    - Only HTTPS example.com
*://*/api/*              - Any URL with /api/ path
<all_urls>               - All URLs (requires justification)
```

---

## Side Panel

```json
{
  "side_panel": {
    "default_path": "sidepanel.html"
  }
}
```

**Required Permission:** `"sidePanel"`

---

## Permissions

### API Permissions

```json
{
  "permissions": [
    "storage",
    "tabs",
    "activeTab",
    "alarms",
    "notifications",
    "contextMenus",
    "sidePanel"
  ]
}
```

| Permission | Description |
|------------|-------------|
| `activeTab` | Access active tab on user gesture |
| `alarms` | Schedule code to run |
| `bookmarks` | Read/write bookmarks |
| `contextMenus` | Add context menu items |
| `cookies` | Read/write cookies |
| `history` | Read/write history |
| `identity` | OAuth authentication |
| `notifications` | Show notifications |
| `scripting` | Execute scripts programmatically |
| `sidePanel` | Use side panel |
| `storage` | Use chrome.storage API |
| `tabs` | Access tab information |
| `webNavigation` | Track page navigation |

### Host Permissions

```json
{
  "host_permissions": [
    "*://*.example.com/*",
    "https://api.service.com/*"
  ]
}
```

**Separated from permissions in MV3** for clearer security model.

### Optional Permissions

```json
{
  "optional_permissions": ["tabs", "history"],
  "optional_host_permissions": ["*://*.optional-site.com/*"]
}
```

Request at runtime:
```javascript
chrome.permissions.request({
  permissions: ['tabs'],
  origins: ['*://*.optional-site.com/*']
});
```

---

## Web Accessible Resources

```json
{
  "web_accessible_resources": [
    {
      "resources": ["images/*.png", "scripts/inject.js"],
      "matches": ["*://*.example.com/*"]
    },
    {
      "resources": ["fonts/*"],
      "matches": ["<all_urls>"],
      "use_dynamic_url": true
    }
  ]
}
```

| Property | Description |
|----------|-------------|
| `resources` | Files accessible from web pages |
| `matches` | Which pages can access |
| `extension_ids` | Which extensions can access |
| `use_dynamic_url` | Generate unique URL per session |

---

## Content Security Policy

```json
{
  "content_security_policy": {
    "extension_pages": "script-src 'self'; object-src 'self'",
    "sandbox": "sandbox allow-scripts; script-src 'self'"
  }
}
```

**MV3 Restrictions:**
- No `unsafe-eval`
- No remote code execution
- No inline scripts in HTML

---

## Declarative Net Request

```json
{
  "declarative_net_request": {
    "rule_resources": [
      {
        "id": "ruleset_1",
        "enabled": true,
        "path": "rules.json"
      }
    ]
  },
  "permissions": ["declarativeNetRequest"]
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
      "resourceTypes": ["script", "image"]
    }
  }
]
```

---

## Options Page

```json
{
  "options_page": "options.html",

  // OR embedded in extensions page:
  "options_ui": {
    "page": "options.html",
    "open_in_tab": false
  }
}
```

---

## Commands (Keyboard Shortcuts)

```json
{
  "commands": {
    "_execute_action": {
      "suggested_key": {
        "default": "Ctrl+Shift+Y",
        "mac": "Command+Shift+Y"
      },
      "description": "Activate extension"
    },
    "custom_command": {
      "suggested_key": {
        "default": "Ctrl+Shift+U"
      },
      "description": "Custom action"
    }
  }
}
```

---

## Externally Connectable

```json
{
  "externally_connectable": {
    "ids": ["other_extension_id"],
    "matches": ["https://example.com/*"]
  }
}
```

Allows external websites or extensions to send messages.

---

## Incognito Mode

```json
{
  "incognito": "spanning"
}
```

| Value | Description |
|-------|-------------|
| `spanning` | Single instance for normal and incognito |
| `split` | Separate instances |
| `not_allowed` | Disabled in incognito |

---

## Minimum Chrome Version

```json
{
  "minimum_chrome_version": "88"
}
```

---

## Localization

```json
{
  "name": "__MSG_extension_name__",
  "default_locale": "en"
}
```

Requires `_locales/en/messages.json`:
```json
{
  "extension_name": {
    "message": "My Extension",
    "description": "Extension name"
  }
}
```

---

## Complete Example

```json
{
  "manifest_version": 3,
  "name": "My Extension",
  "version": "1.0.0",
  "description": "A helpful Chrome extension",

  "icons": {
    "16": "icons/icon16.png",
    "48": "icons/icon48.png",
    "128": "icons/icon128.png"
  },

  "action": {
    "default_popup": "popup.html",
    "default_icon": {
      "16": "icons/icon16.png",
      "32": "icons/icon32.png"
    }
  },

  "background": {
    "service_worker": "background.js",
    "type": "module"
  },

  "content_scripts": [{
    "matches": ["*://*.example.com/*"],
    "js": ["content.js"],
    "css": ["content.css"]
  }],

  "side_panel": {
    "default_path": "sidepanel.html"
  },

  "permissions": [
    "storage",
    "activeTab",
    "sidePanel",
    "alarms"
  ],

  "host_permissions": [
    "*://*.example.com/*"
  ],

  "web_accessible_resources": [{
    "resources": ["images/*"],
    "matches": ["*://*.example.com/*"]
  }],

  "options_ui": {
    "page": "options.html",
    "open_in_tab": false
  },

  "commands": {
    "_execute_action": {
      "suggested_key": {
        "default": "Ctrl+Shift+E"
      }
    }
  }
}
```

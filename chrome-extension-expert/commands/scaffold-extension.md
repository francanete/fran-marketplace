---
name: scaffold-extension
description: Create a new Chrome Extension project structure with Manifest V3, complete file setup, and optional TypeScript/build configuration
arguments:
  - name: name
    description: Name of the extension project
    required: true
  - name: type
    description: Extension type (popup, sidepanel, content, or full)
    required: false
    default: popup
---

# Scaffold Chrome Extension

Create a new Chrome Extension project with Manifest V3 best practices.

## Parameters
- **name**: {{name}} (required)
- **type**: {{type}} (popup | sidepanel | content | full)

## Instructions

Based on the provided parameters, create a complete extension project structure.

### For ALL extension types, create:

1. **manifest.json** - Manifest V3 configuration
2. **icons/** - Placeholder icons (16, 32, 48, 128)
3. **README.md** - Basic documentation

### Type-specific files:

**popup** (default):
- popup.html
- popup.css
- popup.js
- background.js (service worker)

**sidepanel**:
- sidepanel.html
- sidepanel.css
- sidepanel.js
- background.js (service worker)

**content**:
- content.js
- content.css
- background.js (service worker)

**full** (all components):
- popup.html, popup.css, popup.js
- sidepanel.html, sidepanel.css, sidepanel.js
- content.js, content.css
- options.html, options.css, options.js
- background.js (service worker)

## Manifest Template

```json
{
  "manifest_version": 3,
  "name": "{{name}}",
  "version": "1.0.0",
  "description": "Description of {{name}}",
  "permissions": ["storage"],
  "icons": {
    "16": "icons/icon16.png",
    "32": "icons/icon32.png",
    "48": "icons/icon48.png",
    "128": "icons/icon128.png"
  }
}
```

Add appropriate sections based on type:

**For popup:**
```json
{
  "action": {
    "default_popup": "popup.html",
    "default_icon": {
      "16": "icons/icon16.png",
      "32": "icons/icon32.png"
    }
  },
  "background": {
    "service_worker": "background.js"
  }
}
```

**For sidepanel:**
```json
{
  "side_panel": {
    "default_path": "sidepanel.html"
  },
  "permissions": ["storage", "sidePanel"],
  "background": {
    "service_worker": "background.js"
  },
  "action": {
    "default_title": "Open Side Panel"
  }
}
```

**For content:**
```json
{
  "content_scripts": [{
    "matches": ["<all_urls>"],
    "js": ["content.js"],
    "css": ["content.css"]
  }],
  "background": {
    "service_worker": "background.js"
  }
}
```

## After Scaffolding

1. Replace placeholder icons with actual icons
2. Update manifest description
3. Adjust permissions based on needs
4. Update content_scripts matches for specific domains
5. Test by loading unpacked extension

## Directory Structure Output

```
{{name}}/
├── manifest.json
├── background.js
├── icons/
│   ├── icon16.png (placeholder)
│   ├── icon32.png (placeholder)
│   ├── icon48.png (placeholder)
│   └── icon128.png (placeholder)
├── [type-specific files]
└── README.md
```

Now create the extension structure for "{{name}}" with type "{{type}}".

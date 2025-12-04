---
name: validate-manifest
description: Validate a Chrome Extension manifest.json for Manifest V3 compliance, required fields, and store submission readiness
---

# Validate Chrome Extension Manifest

Analyze the manifest.json in the current project for:
1. Manifest V3 compliance
2. Required fields
3. Permission issues
4. Store submission readiness
5. Common mistakes

## Instructions

1. First, find and read the manifest.json file in the project
2. Perform the validation checks below
3. Report findings in a structured format

## Validation Checks

### Required Fields (MV3)
- [ ] `manifest_version` = 3
- [ ] `name` (max 45 characters)
- [ ] `version` (valid semver format)
- [ ] `description` (max 132 characters for store)

### Icons
- [ ] 16x16 icon present
- [ ] 48x48 icon present
- [ ] 128x128 icon present
- [ ] Icon files exist at specified paths

### Background (if present)
- [ ] Uses `service_worker` (not `scripts` or `page`)
- [ ] Service worker file exists
- [ ] `type: "module"` if using ES modules

### Content Scripts (if present)
- [ ] `matches` array defined
- [ ] Referenced JS/CSS files exist
- [ ] `run_at` is valid (document_idle, document_start, document_end)

### Action (if present)
- [ ] `default_popup` file exists (if specified)
- [ ] `default_icon` files exist

### Side Panel (if present)
- [ ] `sidePanel` permission included
- [ ] `default_path` file exists

### Permissions Analysis
- [ ] No deprecated permissions
- [ ] Permissions match functionality
- [ ] `host_permissions` properly separated from `permissions`
- [ ] No overly broad permissions without justification

### MV3 Compliance
- [ ] No `background.page` or `background.scripts`
- [ ] No `browser_action` (use `action` instead)
- [ ] No `page_action` (use `action` instead)
- [ ] CSP compliant (no unsafe-eval, no remote code)

### Store Readiness
- [ ] Name is unique and descriptive
- [ ] Description is clear and complete
- [ ] All required icons present
- [ ] No policy-violating permissions

## Output Format

```markdown
## Manifest Validation Report

### Status: [PASS/FAIL/WARNINGS]

### Required Fields
| Field | Status | Value/Issue |
|-------|--------|-------------|
| manifest_version | ✅/❌ | |
| name | ✅/❌ | |
| version | ✅/❌ | |
| description | ✅/❌ | |

### Icons
[Status of icon validation]

### Background Service Worker
[Status and issues]

### Permissions
| Permission | Necessary | Justification Needed |
|------------|-----------|---------------------|
| [perm] | Yes/No/Maybe | Yes/No |

### MV3 Compliance Issues
[List any MV2 patterns found]

### Recommendations
1. [High priority fixes]
2. [Improvements]
3. [Suggestions]

### Store Submission Readiness
[Ready/Not Ready] - [Explanation]
```

Now find and validate the manifest.json file.

---
name: store-requirements
description: Chrome Web Store guidelines and requirements covering Developer Program Policies, single purpose requirement, permission justifications, privacy policy, branding, screenshots, common rejection reasons, and appeal process. Essential for successful store submission.
---

# Chrome Web Store Requirements

## Developer Program Policies

### Core Principles

1. **Single Purpose** - Extension must have one clear purpose
2. **Transparency** - Clearly describe functionality
3. **Privacy** - Respect user data
4. **Security** - No malicious behavior
5. **Quality** - Work as advertised

### Policy Categories

| Category | Key Requirements |
|----------|-----------------|
| Content | No malware, spam, hate, or illegal content |
| Data | Minimal collection, clear disclosure |
| Functionality | Works as described, no deception |
| Monetization | Clear about paid features |
| User Experience | No unwanted behavior |

---

## Single Purpose Requirement

### What It Means

Your extension should do **ONE thing well**, not bundle unrelated features.

**ACCEPTABLE:**
- Tab manager with grouping, search, and organization
- Note-taking with formatting, tags, and sync
- Ad blocker with filter customization

**NOT ACCEPTABLE:**
- Tab manager + weather widget + game
- Coupon finder + screenshot tool
- Anything with "multi-tool" bundled features

### How to Define Purpose

```markdown
## Single Purpose Statement
[Extension Name] helps users [primary action] by [method].

Example:
"TabMaster helps users organize their browser tabs by providing
grouping, search, and session management features."
```

### Feature Justification

All features should relate to the core purpose:

| Feature | Core: Tab Management | Justified? |
|---------|---------------------|------------|
| Tab grouping | Direct | ✅ |
| Tab search | Direct | ✅ |
| Session save | Related | ✅ |
| Tab statistics | Enhancement | ✅ |
| Weather widget | Unrelated | ❌ |
| Ad blocking | Unrelated | ❌ |

---

## Permission Justifications

### When Required

Justifications required for:
- `tabs`
- `webNavigation`
- `history`
- `bookmarks`
- `topSites`
- `browsingData`
- `<all_urls>` or broad host permissions

### Writing Good Justifications

**Structure:**
```
Permission: [permission name]

1. What we use it for:
   [Specific feature that needs this permission]

2. Why it's necessary:
   [Why the feature can't work without it]

3. What we don't do:
   [Reassurance about data not collected/shared]
```

**Example - tabs permission:**
```
Permission: tabs

1. What we use it for:
   - Display tab titles in our tab search feature
   - Show favicon in the tab list
   - Enable "Switch to Tab" functionality

2. Why it's necessary:
   Our core tab management features require reading tab metadata
   to provide search and organization capabilities.

3. What we don't do:
   - We do not track browsing history
   - We do not transmit URLs to external servers
   - Tab data is only stored locally on the user's device
```

### Common Permission Justifications

**activeTab:**
```
Required to read the current page content when the user clicks
the extension icon. Only activated on user gesture, no passive
access to browsing data.
```

**storage:**
```
Used to save user preferences and extension state locally.
No data is transmitted to external servers.
```

**alarms:**
```
Required for scheduling periodic background tasks like
syncing saved data and checking for updates.
```

---

## Privacy Policy Requirements

### When Required

Privacy policy is **required** if your extension:
- Collects any user data
- Uses analytics
- Has any network requests
- Uses certain permissions (tabs, history, etc.)

### Required Sections

```markdown
# Privacy Policy for [Extension Name]

Last updated: [Date]

## What Data We Collect
- [List all data collected]
- [Include data from permissions]
- [Include any analytics]

## How We Use Data
- [Purpose for each data type]
- [Who processes the data]

## Data Storage
- Where data is stored (local/cloud)
- How long data is retained
- Encryption methods used

## Data Sharing
- Third parties who receive data
- Purpose of sharing
- Or: "We do not share data with third parties"

## User Rights
- How to access your data
- How to delete your data
- How to opt out

## Changes to Policy
- How users will be notified

## Contact
- Email for privacy concerns
```

### Hosting Requirements

- Must be hosted on accessible URL
- HTTPS required
- Should be stable (don't use temporary hosts)
- Good options: GitHub Pages, your website

---

## Store Listing Requirements

### Extension Name

- **Max:** 45 characters
- **Must:** Be unique and descriptive
- **Avoid:** Keyword stuffing, misleading names
- **No:** Trademark infringement

### Description

**Short description (shown in search):**
- First ~132 characters are crucial
- Clear value proposition
- No keyword stuffing

**Full description:**
```markdown
[One-line value proposition]

Key Features:
• [Feature 1]
• [Feature 2]
• [Feature 3]

How to Use:
1. [Step 1]
2. [Step 2]
3. [Step 3]

[Additional details about the extension]

Support: [contact info or support URL]
```

### Screenshots

**Requirements:**
- Size: 1280x800 or 640x400 pixels
- Format: PNG or JPEG
- Minimum: 1
- Maximum: 5

**Best Practices:**
- Show actual functionality
- Include UI elements
- Add brief annotations if needed
- Avoid excessive text/branding
- High quality, no compression artifacts

### Promotional Images

| Image | Size | Required |
|-------|------|----------|
| Small tile | 440x280 | Yes (for featuring) |
| Marquee | 1400x560 | No (for featuring) |

### Icons

| Size | Usage | Required |
|------|-------|----------|
| 16x16 | Toolbar | Yes |
| 32x32 | Windows | Recommended |
| 48x48 | Extensions page | Yes |
| 128x128 | Store listing | Yes |

---

## Common Rejection Reasons

### 1. Single Purpose Violation

**Error:** "Does not comply with the single purpose policy"

**Fix:**
- Remove unrelated features
- Split into multiple extensions
- Update description to clarify purpose

### 2. Missing/Inadequate Privacy Policy

**Error:** "Missing or inadequate privacy policy"

**Fix:**
- Create comprehensive privacy policy
- Host on accessible HTTPS URL
- Cover all data practices

### 3. Permission Overreach

**Error:** "Requesting permissions that exceed functionality"

**Fix:**
- Remove unnecessary permissions
- Use activeTab instead of tabs
- Add optional_permissions
- Provide clear justifications

### 4. Misleading Functionality

**Error:** "Does not work as described"

**Fix:**
- Match description to actual features
- Test all advertised functionality
- Remove claims about unimplemented features

### 5. Missing Justifications

**Error:** "Missing permission justification"

**Fix:**
- Add justifications in Developer Dashboard
- Be specific about each permission use
- Explain user benefit

### 6. Quality Issues

**Error:** "Extension does not meet quality standards"

**Fix:**
- Fix all bugs
- Test on multiple sites
- Handle errors gracefully
- Improve performance

### 7. Deceptive Behavior

**Error:** "Extension engages in deceptive practices"

**Fix:**
- Remove hidden functionality
- Be transparent about monetization
- Don't change browser settings unexpectedly

---

## Submission Process

### Developer Account Setup

1. Go to [Chrome Developer Dashboard](https://chrome.google.com/webstore/devconsole)
2. Pay one-time $5 registration fee
3. Verify email address
4. Complete account profile

### Package Preparation

```bash
# Create ZIP file (not .crx)
zip -r extension.zip \
  manifest.json \
  background.js \
  content.js \
  popup/ \
  icons/ \
  # ... all files except:
  #   - .git/
  #   - node_modules/
  #   - source files (if bundled)
```

### Submission Steps

1. **Upload** - Upload ZIP file
2. **Store Listing** - Fill in all fields
3. **Privacy** - Complete privacy questionnaire
4. **Screenshots** - Upload required images
5. **Distribution** - Choose visibility
6. **Submit** - Send for review

### Review Timeline

| Submission Type | Typical Time |
|----------------|--------------|
| New extension | 1-3 business days |
| Update | 1-2 business days |
| After rejection | Varies |

---

## Update Strategy

### Version Management

```json
{
  "version": "1.2.3"
  // Major.Minor.Patch
  // Major: Breaking changes, significant features
  // Minor: New features, improvements
  // Patch: Bug fixes
}
```

### Update Best Practices

1. **Test thoroughly** before submission
2. **Stage rollout** if available
3. **Document changes** in store listing
4. **Maintain backwards compatibility**
5. **Respond to user feedback**

### Update Notes Template

```markdown
Version X.Y.Z - [Date]

New Features:
• [Feature description]

Improvements:
• [Improvement description]

Bug Fixes:
• [Bug fix description]

Thank you for your feedback!
```

---

## Appeal Process

### When to Appeal

- Believe rejection was incorrect
- Have additional information
- Made changes to address concerns

### How to Appeal

1. Reply to rejection email
2. Be professional and factual
3. Provide specific information:

```
Subject: Appeal for [Extension Name] - [Submission ID]

Dear Chrome Web Store Team,

I am writing to appeal the rejection of [Extension Name].

Rejection Reason: [Quote their reason]

My Response:
[Explain why you believe this is incorrect, OR]
[Explain changes you've made to address the issue]

Evidence:
[Screenshots, code snippets, or documentation]

I believe this addresses the concern because:
[Clear reasoning]

Please let me know if you need additional information.

Best regards,
[Your name]
```

### Appeal Timeline

- Initial response: 3-5 business days
- Complex cases: 1-2 weeks
- May require multiple exchanges

---

## Compliance Checklist

### Before Submission
- [ ] Extension has single, clear purpose
- [ ] All features relate to core purpose
- [ ] Permissions are minimal and justified
- [ ] Privacy policy covers all data practices
- [ ] Description accurately reflects functionality
- [ ] All screenshots show real functionality
- [ ] Icons are clear and professional
- [ ] Extension tested and working
- [ ] No console errors
- [ ] No policy violations

### Store Listing
- [ ] Name is unique and descriptive (≤45 chars)
- [ ] Description explains value clearly
- [ ] Category is appropriate
- [ ] Screenshots are high quality (1280x800)
- [ ] Small tile provided (440x280)
- [ ] Privacy policy URL accessible

### Technical
- [ ] Manifest V3 compliant
- [ ] No remote code
- [ ] Proper error handling
- [ ] CSP compliant
- [ ] No hardcoded secrets

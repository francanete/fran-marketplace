---
name: store-checklist
description: Generate a comprehensive Chrome Web Store submission checklist for your extension, including policy compliance and required assets
---

# Chrome Web Store Submission Checklist

Generate a comprehensive checklist for submitting your extension to the Chrome Web Store.

## Instructions

1. Analyze the current extension project
2. Check for required assets and compliance items
3. Generate a personalized checklist based on the extension's features

## Pre-Submission Checklist

### 1. Technical Requirements

#### Manifest
- [ ] manifest_version is 3
- [ ] name is descriptive (max 45 chars)
- [ ] version follows semver (e.g., 1.0.0)
- [ ] description is clear (max 132 chars for store display)
- [ ] All referenced files exist

#### Icons (Required)
- [ ] 16x16 PNG icon
- [ ] 32x32 PNG icon (for Windows)
- [ ] 48x48 PNG icon (extensions page)
- [ ] 128x128 PNG icon (store listing)
- [ ] Icons are clear at small sizes
- [ ] No trademark/copyright violations

#### Functionality
- [ ] Extension works on Chrome stable
- [ ] All advertised features functional
- [ ] Error handling implemented
- [ ] No console errors in normal use
- [ ] Tested on multiple websites (if applicable)

### 2. Store Listing Assets

#### Screenshots (Required)
- [ ] At least 1 screenshot
- [ ] Size: 1280x800 or 640x400 pixels
- [ ] PNG or JPEG format
- [ ] Shows actual functionality
- [ ] No excessive promotional text
- [ ] High quality, no artifacts

#### Promotional Images
- [ ] Small tile: 440x280 (required for featuring)
- [ ] Marquee: 1400x560 (optional)

#### Store Listing Content
- [ ] Detailed description written
- [ ] Key features highlighted
- [ ] Usage instructions included
- [ ] Category selected appropriately
- [ ] Language/localization set

### 3. Privacy & Compliance

#### Privacy Policy (Required if collecting data)
- [ ] Privacy policy URL accessible
- [ ] Covers what data is collected
- [ ] Explains how data is used
- [ ] Describes data storage
- [ ] Third-party sharing disclosed
- [ ] User deletion rights explained
- [ ] Contact information provided

#### Permission Justifications
For each sensitive permission, prepare justification:

| Permission | Justification Required |
|------------|----------------------|
| tabs | Yes - explain tab access |
| history | Yes - explain history access |
| bookmarks | Yes - explain bookmark access |
| <all_urls> | Yes - explain broad access |
| cookies | Yes - explain cookie access |
| webNavigation | Yes - explain navigation tracking |

#### Single Purpose Policy
- [ ] Extension has ONE clear purpose
- [ ] All features relate to core purpose
- [ ] No bundled unrelated functionality

#### Data Handling
- [ ] Minimal data collection
- [ ] No unnecessary permissions
- [ ] Secure data transmission (HTTPS)
- [ ] No selling of user data

### 4. Policy Compliance

#### Content Policies
- [ ] No malware or deceptive behavior
- [ ] No spam or repetitive content
- [ ] No sexually explicit material
- [ ] No hate speech or violence
- [ ] No illegal activities

#### Developer Policies
- [ ] No impersonation
- [ ] No misleading functionality
- [ ] No unauthorized data collection
- [ ] No interfering with other extensions
- [ ] No cryptocurrency mining

### 5. Quality Checks

#### User Experience
- [ ] Clear onboarding for new users
- [ ] Intuitive interface
- [ ] Responsive to user actions
- [ ] Appropriate loading states
- [ ] Clear error messages

#### Performance
- [ ] Fast popup/panel load time
- [ ] Minimal memory usage
- [ ] No impact on browser performance
- [ ] Efficient background processing

### 6. Developer Account

- [ ] Developer account created
- [ ] $5 registration fee paid
- [ ] Contact email verified
- [ ] Developer name set

### 7. Final Review

- [ ] Test extension after packaging
- [ ] ZIP file created (not .crx)
- [ ] All files included in package
- [ ] No development/debug code
- [ ] Version number updated

## Submission Steps

1. Go to [Chrome Developer Dashboard](https://chrome.google.com/webstore/devconsole)
2. Click "New Item"
3. Upload ZIP file
4. Fill in store listing
5. Upload screenshots
6. Complete privacy practices
7. Add privacy policy URL
8. Submit for review

## Expected Timeline

- Initial review: 1-3 business days
- Complex extensions: Up to 7 days
- Rejections require re-submission

## After Approval

- [ ] Test published extension
- [ ] Monitor user reviews
- [ ] Set up crash/error reporting
- [ ] Plan update schedule

---

Now analyze the current project and generate a customized checklist highlighting what's done and what's missing.

# Chrome Extension Style Guide

Standards for Chrome extensions (Manifest V3).

## File Structure

### Top-Level Structure
```
extension_name/
├── _locales/
│   ├── en/
│   │   └── messages.json
│   └── zh_CN/
│       └── messages.json
├── images/
│   ├── icon-16.png
│   ├── icon-32.png
│   ├── icon-48.png
│   ├── icon-128.png
│   └── icon-*.png
├── background/
│   └── service_worker.js
├── content_scripts/
│   └── content.js
├── popup/
│   ├── popup.html
│   ├── popup.js
│   └── popup.css
├── options/
│   ├── options.html
│   ├── options.js
│   └── options.css
├── newtab/
│   ├── newtab.html
│   ├── newtab.js
│   └── newtab.css
├── docs/
│   └── (documentation)
├── lib/
│   └── (third-party libraries)
├── src/
│   └── (source files if using TypeScript/webpack)
├── manifest.json
├── README.md
└── LICENSE
```

### Manifest (manifest.json)
```json
{
  "manifest_version": 3,
  "name": "__MSG_extension_name__",
  "version": "1.0.0",
  "description": "__MSG_extension_description__",
  "default_locale": "en",
  "icons": {
    "16": "images/icon-16.png",
    "32": "images/icon-32.png",
    "48": "images/icon-48.png",
    "128": "images/icon-128.png"
  },
  "background": {
    "service_worker": "background/service_worker.js"
  },
  "action": {
    "default_popup": "popup/popup.html",
    "default_icon": {
      "16": "images/icon-16.png",
      "32": "images/icon-32.png"
    }
  },
  "permissions": [
    "storage",
    "tabs"
  ],
  "host_permissions": [
    "<all_urls>"
  ]
}
```

## Formatting

- **Language**: JavaScript (ES6+), TypeScript
- **Indent**: 2 spaces
- **Max line length**: 100 chars
- **Line endings**: LF (Unix-style)
- **Quotes**: Single quotes for strings, double for HTML

## Naming Conventions

| Type | Convention | Example |
|:-----|:-----------|:--------|
| Files | `snake_case` | `service_worker.js`, `popup.js` |
| Functions | `camelCase` | `handleClick`, `fetchData` |
| Classes | `PascalCase` | `MyClass` |
| Constants | `UPPER_SNAKE_CASE` | `DEFAULT_TIMEOUT` |
| DOM IDs | `kebab-case` | `submit-button` |
| CSS Classes | `kebab-case` | `.main-content` |
| Message Keys | `lowercase_underscore` | `btn_submit`, `msg_error` |

## Code Organization

### Service Worker
- Keep background script minimal
- Use event-driven architecture
- Use chrome.storage for persistence

```javascript
// Background script pattern
chrome.runtime.onInstalled.addListener(() => {
  // Initialize extension
});

chrome.runtime.onMessage.addListener((request, sender, sendResponse) => {
  if (request.action === 'fetch') {
    handleFetch(request.url).then(sendResponse);
    return true; // async response
  }
});
```

### Content Script
- Run at document_idle
- Communicate with background via messaging
- Avoid page script conflicts

```javascript
// Content script
chrome.runtime.sendMessage({ action: 'getData' }, (response) => {
  processData(response.data);
});
```

### Popup
- Use separate JS file from HTML
- Use chrome.storage API
- Minimize heavy operations

```javascript
// popup.js
document.addEventListener('DOMContentLoaded', () => {
  loadSettings();
});

function loadSettings() {
  chrome.storage.local.get(['key'], (result) => {
    document.getElementById('input').value = result.key || '';
  });
}
```

## UI Development

### Popup HTML
- Keep minimal and fast loading
- Inline CSS for single file, separate for complex UI

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <link rel="stylesheet" href="popup.css">
</head>
<body>
  <div class="container">
    <input type="text" id="input-field" placeholder="Enter value">
    <button id="submit" class="btn btn-primary">Submit</button>
  </div>
  <script src="popup.js"></script>
</body>
</html>
```

### Popup CSS
```css
.container {
  width: 300px;
  padding: 10px;
}

.btn {
  padding: 8px 16px;
  border: none;
  cursor: pointer;
}

.btn-primary {
  background: #4285f4;
  color: white;
}
```

## Permissions and Host Permissions

### Common Permissions
- `storage` - Use chrome.storage API
- `tabs` - Access tab information
- `activeTab` - Access active tab when clicked
- `scripting` - Execute content scripts
- `declarativeNetRequest` - Modify network requests

### Best Practices
- Request only necessary permissions
- Use optional permissions when possible
- Explain permission need in description

```json
{
  "permissions": ["storage"],
  "optional_permissions": ["tabs"],
  "host_permissions": ["https://example.com/*"]
}
```

## Internationalization (i18n)

### Message Keys
```json
{
  "extension_name": {
    "message": "My Extension",
    "description": "Name of the extension"
  },
  "btn_submit": {
    "message": "Submit",
    "description": "Submit button text"
  }
}
```

### Usage
```javascript
// In JavaScript
chrome.i18n.getMessage('btn_submit')

// In HTML
<span data-i18n="btn_submit"></span>
```

## Preferred APIs

| Purpose | API |
|:--------|:---|
| Storage | chrome.storage |
| Tabs | chrome.tabs |
| Messages | chrome.runtime |
| Context Menus | chrome.contextMenus |
| Commands | chrome.commands |
| Web Navigation | chrome.webNavigation |

## Common Mistakes and Correct Implementation

### 1. DON'T: Execute heavy operations in popup

```javascript
// WRONG - popup closes immediately
document.getElementById('btn').addEventListener('click', () => {
  fetch('https://api.example.com/data'); // This won't complete
});
```

### ✅ DO: Use background script for heavy operations

```javascript
// popup.js
document.getElementById('btn').addEventListener('click', () => {
  chrome.runtime.sendMessage({ action: 'fetch' });
});

// background.js
chrome.runtime.onMessage.addListener((request, sender, sendResponse) => {
  if (request.action === 'fetch') {
    fetch('https://api.example.com/data').then(sendResponse);
    return true;
  }
});
```

### 2. DON'T: Use remote code execution

```javascript
// WRONG - violates manifest v3
chrome.scripting.executeScript({
  code: 'console.log(eval("alert(1)"))'
});
```

### ✅ DO: Use local files only

```javascript
// CORRECT
chrome.scripting.executeScript({
  files: ['content_script.js']
});
```

### 3. DON'T: Store sensitive data in localStorage

```javascript
// WRONG
localStorage.setItem('token', 'secret');
```

### ✅ DO: Use chrome.storage with encryption or avoid storing sensitive data

```javascript
// CORRECT
chrome.storage.local.set({ settings: { theme: 'dark' } });
```

### 4. DON'T: Use deprecated APIs

```javascript
// WRONG - deprecated in MV3
chrome.extension.getBackgroundPage()
```

### ✅ DO: Use message passing

```javascript
// CORRECT - background uses onMessage
chrome.runtime.onMessage.addListener((request, sender, sendResponse) => {
  // Handle message
});
```
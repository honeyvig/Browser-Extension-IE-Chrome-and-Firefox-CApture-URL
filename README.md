# Browser-Extension-IE-Chrome-and-Firefox-CApture-URL
a browser extension (works for IE, Chrome & Fire Fox) That will capture the URL of the current site and store it as a cookie for retrieval by website
---------------
To create a browser extension that captures the URL of the current site and stores it as a cookie, we'll need to follow different approaches based on the browser but generally, it will be the same logic for Chrome, Firefox, and Internet Explorer (though IE is less common today).

Here's the general breakdown:

    Capture the Current URL: We'll use JavaScript to grab the current URL of the page.
    Store the URL as a Cookie: Store the URL in a browser cookie, which can be accessed later by the website.
    Browser Compatibility: We'll make it work across Chrome, Firefox, and IE. For modern browsers, the code is similar, but we'll handle each browser separately using different methods for cookies, especially in the background scripts.

1. Manifest File (manifest.json)

For the extension to work in Chrome, Firefox, and Internet Explorer, we'll define the manifest.json to specify permissions, background scripts, and any content scripts that need to interact with the page.

{
  "manifest_version": 2,
  "name": "URL Capture and Store",
  "description": "Capture the current URL and store it as a cookie for retrieval by the website.",
  "version": "1.0",
  "permissions": [
    "tabs",
    "cookies",
    "<all_urls>"
  ],
  "background": {
    "scripts": ["background.js"],
    "persistent": false
  },
  "browser_action": {
    "default_popup": "popup.html"
  },
  "content_scripts": [
    {
      "matches": ["<all_urls>"],
      "js": ["content.js"]
    }
  ],
  "icons": {
    "16": "icon16.png",
    "48": "icon48.png",
    "128": "icon128.png"
  }
}

2. Background Script (background.js)

The background script will listen for the browser's active tab, capture its URL, and store it in a cookie for later use.

// background.js

// Function to store the current URL in a cookie
function setCookie(url) {
  let cookieDetails = {
    url: 'http://example.com', // Replace with the domain that will access the cookie
    name: 'current_url',
    value: url,
    path: '/',
    secure: true,
    httpOnly: false,  // You can make it true if you're dealing with server-side access
    sameSite: 'Strict',
    expirationDate: (new Date().getTime() / 1000) + 3600, // expires in 1 hour
  };

  chrome.cookies.set(cookieDetails, function (cookie) {
    console.log("Cookie set:", cookie);
  });
}

// Listen for tab changes and capture the URL
chrome.tabs.onActivated.addListener(function (activeInfo) {
  chrome.tabs.get(activeInfo.tabId, function (tab) {
    if (tab && tab.url) {
      setCookie(tab.url);
    }
  });
});

// Capture URL when the extension icon is clicked
chrome.browserAction.onClicked.addListener(function (tab) {
  if (tab && tab.url) {
    setCookie(tab.url);
  }
});

3. Content Script (content.js)

This content script ensures the URL is accessible for the website to retrieve when needed. You can also use this script to display or modify the page based on the stored cookie.

// content.js

// Retrieve the stored cookie (current_url)
chrome.cookies.get({ url: 'http://example.com', name: 'current_url' }, function (cookie) {
  if (cookie) {
    console.log('Stored URL:', cookie.value);
    // Now you can inject the URL into your website or use it for any logic
  }
});

4. Popup HTML (popup.html)

The popup file (optional) can display the stored URL for the user to view when clicking on the extension's icon.

<!DOCTYPE html>
<html>
  <head>
    <title>URL Capture</title>
  </head>
  <body>
    <h1>Stored URL</h1>
    <p id="url"></p>
    <script>
      chrome.cookies.get({ url: 'http://example.com', name: 'current_url' }, function (cookie) {
        if (cookie) {
          document.getElementById('url').innerText = cookie.value;
        } else {
          document.getElementById('url').innerText = 'No URL stored';
        }
      });
    </script>
  </body>
</html>

5. Manifest for Internet Explorer

Unfortunately, Internet Explorer doesn’t use the same extension system as Chrome or Firefox. You would need to create a different kind of extension for IE, typically a browser helper object (BHO), which is quite complex compared to modern browser extensions. But in modern environments, this approach would mainly apply to Chrome and Firefox. For IE, you'd need to focus on ActiveX controls or other legacy methods which are deprecated.
6. How It Works

    Capture URL: When a user visits a webpage, the extension captures the URL of the active tab.
    Store in Cookie: The URL is then stored in a cookie with the domain where you need to retrieve it.
    Retrieve URL: Websites that have access to the domain can retrieve this cookie via JavaScript and use it as needed.

7. Installation and Usage

    Go to chrome://extensions or about:addons in Firefox.
    Enable Developer Mode (for Chrome).
    Click Load Unpacked and select the directory where your extension is stored.
    The extension should now be available in the toolbar, and whenever you visit a page, it captures and stores the URL in a cookie.

Conclusion:

This basic extension captures the URL of the current site and stores it as a cookie for retrieval by websites. While it works for modern browsers (Chrome, Firefox), Internet Explorer would require a different and more complex approach with legacy tools. If you're aiming for widespread use, focusing on Chrome and Firefox is advisable.

# Cross-Site Scripting (XSS)

## Overview

Cross-Site Scripting (XSS) is a type of web vulnerability that allows attackers to inject malicious scripts into trusted websites. This occurs when an application includes user input in its output without proper validation or encoding.

An attacker can use XSS to execute scripts in a victim’s browser, leading to:

- Theft of cookies, session tokens, or other sensitive data.
- Defacement or modification of webpage content.
- Redirecting users to malicious websites.
---
## Types of XSS

### 1. **Reflected XSS

Reflected XSS occurs when a malicious script is immediately reflected off a web application and executed in a victim’s browser. This often happens through manipulated URLs, search forms, or error messages.

**Attack flow:**

1. The attacker crafts a malicious URL containing a script.
2. The victim clicks the URL, sending the request to the vulnerable website.
3. The website reflects the script in its response.
4. The browser executes the script, believing it to be from a trusted source.

---

### 2. **Stored XSS 

Stored XSS happens when the injected script is permanently stored on a server (e.g., in a database, comment section, or forum post). When another user views the stored data, the script executes in their browser.

**Attack flow:**

1. The attacker submits a script as a comment, forum post, or profile description.
2. The web application saves the malicious input without proper sanitization.
3. When a victim views the stored content, the script runs in their browser.

---
### 3. **Blind XSS**

Blind XSS is a type of stored XSS that affects backend users, such as administrators or moderators. The attacker injects a script into a form or other input field, and when an admin reviews the input in their control panel, the script executes.

**Example Attack Vector:**

- Submitting a malicious payload via a feedback form.
- The admin views the feedback in the backend panel, unknowingly triggering the XSS.
- This can lead to account takeover or backend exploitation.

---
### 4. **DOM-Based XSS**

DOM XSS occurs when the script execution happens entirely in the victim's browser without any server interaction. The vulnerability exists in JavaScript code that directly modifies the DOM based on user input.

**Attack flow:**

1. A website dynamically modifies the page based on URL fragments (e.g., `window.location.hash`).
2. The attacker crafts a URL with malicious JavaScript.
3. When the victim visits the URL, the script executes within their browser.

**Example Code (Vulnerable JavaScript):**

```javascript
document.write("<p>" + window.location.hash + "</p>");
```

If the user visits:

```
http://example.com/#<script>alert('XSS');</script>
```

The script gets executed because `document.write()` directly injects user-controlled input into the DOM.

---

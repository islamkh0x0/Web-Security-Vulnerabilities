# Cross-Site Request Forgery (CSRF) 
## What’s CSRF?
It’s an attack where we can trick the victim into sending a request with some malicious code that lets us change stuff about the victim, like their (email, username, password). 
1. I look for it when I find it in something like "forget password" so it turns into an account takeover. They send an email with content the victim likes, so they click it, and there’s an invisible iframe in there that runs the malicious request. 
2. Or if it’s a shopping site, I can force them to buy something without them knowing. 
--- 
## When Can I Exploit CSRF? 
- If there’s an important function like changing the password or email. 
- If the system only checks stuff through the cookie. 
- If there’s no CSRF parameter. 
--- 
## How Does a Site Protect Itself from CSRF? 
### Through CSRF Token 
Okay, so how does this token thing work? It’s all done in PHP, and there are 3 pages to get it right. It starts with the page where the session gets created. 
1. This code checks if the token isn’t in the session yet, it generates a new one. That means a new one gets made for every session. 
   
```php
session_start(); if (!isset($_SESSION['csrf_token'])) { $_SESSION['csrf_token'] = bin2hex(random_bytes(32)); }
```

2. The second page is where the process gets tested.  
    Like, if the request came from another page, does it have the token that’s registered on the server for the current session? Is it matching or not? Does it even exist?

```php
<?php
session_start();
if (!isset($_POST['csrf_token']) || $_POST['csrf_token'] !== $_SESSION['csrf_token']) {
    // Token doesn’t match or isn’t there
    die("Invalid CSRF token");
}
// If the token matches, we go ahead with the process
?>
```

3. Then comes the form itself.

```php
<?php session_start(); ?>
<form method="POST" action="process.php">
    <!-- Other fields -->
    <input type="hidden" name="csrf_token" value="<?php echo $_SESSION['csrf_token']; ?>">
    <button type="submit">Send</button>
</form>
```
### Through Same-Site Cookies

This is a setting that makes cookies get sent only in specific cases:

1. In **Strict** mode: Cookies only get sent if the request comes from the exact same domain.
2. In **Lax** mode: Cookies get sent in most cases, but not in some sensitive ones (like POST requests or requests from other sites); it’s a middle ground.
3. If it’s **None**: That means the cookies get sent with all requests, even from another site, but it has to have the Secure flag (so it’s through HTTPS).  
   Here’s some code to show how it works:

```php
// Example in PHP to set cookies with SameSite
session_set_cookie_params([
    'lifetime' => 0,      // Cookie lasts for the session
    'path' => '/',
    'domain' => 'example.com',
    'secure' => true,     // Cookies only sent over HTTPS
    'httponly' => true,   // Cookies not accessible by JavaScript
    'samesite' => 'Strict' // Or 'Lax' depending on what you need
]);
session_start();
```

When the cookie setting is SameSite=Strict or Lax, the browser makes sure before sending the cookies with any request that it’s coming from the same domain. So if an attacker tries to do CSRF from another site, the cookies (like the user’s session) won’t get sent with the request, and the server won’t recognize the user or process it.

### Through Referrer Header

Some sites check if the request is coming from the same site or a different one. If it’s from another site, they reject it.

### Through Check Content Type

If the body is JSON, either you make the content type JSON and check it, or it’s just any content type, and you know it’s gonna be JSON, so you treat it like JSON.

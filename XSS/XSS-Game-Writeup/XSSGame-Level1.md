### **XSSGame Level 1 Writeup**  

After learning about **XSS**, the first step in finding a vulnerability is testing which characters are allowed and bypass the filter.  

In this challenge, we have an **input field**, as shown in the image. 
![Image](https://github.com/user-attachments/assets/7566ead3-f20a-4015-86db-e829c5dc344e) 
  
Let's enter the following payload:  

```
eslam"<>
```

Now, we check what gets reflected on the page.  

Looking at the source code, we see that **all the characters we entered were reflected** without any encoding or filtering:  

```html
<!DOCTYPE html>
<html>
  <head>
    <style>
      body {
        background-color: #ffffff;
      }
    </style>
    <script src="/static/js/js_frame.js"></script>
  </head>
  <body>
    <center>
      <img src="/static/img/foogle.png">
      <br><br>
      Sorry, no results were found for <b>x"<>;</b>.
      <a href="?">Try again</a>.
      <br>
    </center>
  </body>
</html>
```

Since our input is **reflected**, we can inject a simple XSS payload to trigger an alert box:  

```html
<script>alert()</script>
```

Once executed, an alert box appears, confirming the **XSS vulnerability**, as shown in the image.
![Image](https://github.com/user-attachments/assets/2c924616-8081-4d69-918c-ba2f192c95aa)

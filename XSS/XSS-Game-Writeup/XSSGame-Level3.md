### XSSGame Level 3 Writeup

![Gallery Screenshot](https://github.com/user-attachments/assets/c75b0195-6a8c-4c80-8a36-35540fa2548b)

---

When I opened this challenge, I didn’t find any **input fields**.  
Instead, the page was a simple **image gallery** showing cat pictures.

images.

So I checked the source code to understand how the gallery works. Here's what I found:

The function chooseTab(name) takes a value (which is expected to be a number), and then constructs a string of HTML to render the image:

```javascript
var html = "Cat " + parseInt(name) + "<br>";
html += "<img src='/static/img/cat" + name + ".jpg' />";
document.getElementById('tabContent').innerHTML = html;
```
I searched for how the name value is passed and found this line:

```javascript
window.location.hash = name;
```
This told me that the value used in the chooseTab() function is taken directly from the URL hash (the part after #).

So now I knew that the injection point is in the `window.location.hash`.

Since the value is inserted directly into the src attribute of the <img> tag, I thought about using the classic XSS vector via the onerror attribute.

So I crafted the following payload:

`4'onerror="alert()"`
And it worked — the alert popped up as expected 
![Image](https://github.com/user-attachments/assets/fc0d0a4e-1ff0-47ba-8d76-3a1206e7ef1d)


Here’s how the final HTML looked inside the Developer Tools:

```html
<img src='/static/img/cat4'onerror="alert()" />
```
![Image](https://github.com/user-attachments/assets/4feb05cf-54ed-4a1f-bf98-02bb1f15fe42)
Which proves that our payload was successfully injected and executed.







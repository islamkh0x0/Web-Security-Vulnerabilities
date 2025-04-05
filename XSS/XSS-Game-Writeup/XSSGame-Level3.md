### XSSGame Level 3 Writeup
![3-1](https://github.com/user-attachments/assets/a2755d1a-bf65-4977-a719-25114a4a69ea)


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
![3-2](https://github.com/user-attachments/assets/f6427ed7-de53-4f4c-9c9c-2b60459cc32d)


Here’s how the final HTML looked inside the Developer Tools:

```html
<img src='/static/img/cat4'onerror="alert()" />
```
![3-3](https://github.com/user-attachments/assets/5a2d2bc5-8a91-4c06-afa3-0156c92a35b6)

Which proves that our payload was successfully injected and executed.







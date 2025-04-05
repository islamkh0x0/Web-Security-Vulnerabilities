# XSS Game Write-Up: Level 5 
![5-1](https://github.com/user-attachments/assets/3973d72b-5d20-4e47-8174-7ddc57cc0bea)


When I opened the challenge, I found a search input field that accepts up to 140 characters. So, I typed `eslam"<>;` to see where the reflection happens and what happens to the special characters, to figure out which ones pass through the filter and which ones get filtered out.

I found that the reflection happens in just one place, and it’s right here:
```html
<p ng-non-bindable>Sorry, no results were found for <b>eslam&quot;&lt;&gt;</b>.</p>
``` 
Of course, all the special characters were HTML encoded. 
But what grabbed my attention was this thing at the start of the tag: 
 
`<p ng-non-bindable>`  

So, I did some searching and figured out that the developer here is using the AngularJS library. From what I know, some versions of AngularJS have vulnerabilities, and payloads for those are floating around online, easy to find. 
I checked the version of AngularJS being used, and it turned out to be AngularJS v1.5.8. 

After digging around, I couldn’t find a payload that worked for this version. 
So, I went back to the source code and stumbled upon this script: 
```html
<script>
  angular.module('myApp', [])
  .controller('myController', ['$scope', function ($scope) {
    $scope.query = "";
    $scope.alert = window.alert;
  }]);

  var UTM_PARAMS = ["utm_content", "utm_medium", "utm_source",
      "utm_campaign", "utm_term"]

  if (location.search)
  {
    var params = location.search.substring(1).split('&');

    for (var p in params) {
      var r = params[p].split('=');

      if (r.length == 2 && UTM_PARAMS.indexOf(r[0]) != -1) {
        var el = document.getElementsByName(r[0]);
        if (el.length) el[0].value = decodeURIComponent(r[1]);
      }
    }
  }
</script>
``` 
I noticed that this script builds a small app with AngularJS and adds a controller to it. It checks if there are any parameters in the URL, specifically UTM parameters (you know, the ones used in marketing). 
If it finds any, it splits them up and looks at their names and values. 
Then it searches the page for input fields with the same names as those parameters and fills them with the values from the URL. 
So, I thought, why not try one of those parameters and sneak a payload into it? 
Since we’re dealing with AngularJS here, the payload would need to look like {{expression}}. 
I tested this payload in the URL: ?utm_campaign={{alert()}}, and guess what? It worked. 

The final URL was: 
`http://www.xssgame.com/f/JFTG_t7t3N-P/?utm_campaign={{alert()}}` 
![5-2](https://github.com/user-attachments/assets/2dbe83aa-76d8-4cf8-b2d2-e58e76dbedbf)

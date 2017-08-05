---
layout: post
title:  Don't rely on $watch oldValue newValue to initialize changes
date:   2017-08-05 20:13:30 +0800
categories: angular
---

TLDR: 
When you add $watch is added to your code, it runs for the first time, showing you that oldValue === newValue. 

This is NOT a bug. Although it may be mistaken as one, as I first encountered it. See [described](https://github.com/angular/angular.js/issues/11565).


Consider the following code. There is a portion of HTML we wish to toggle (show or hide) based on a controller property.

```html
<!-- page.html -->
<div class="div" ng-if="ctrl.toggleHide">
        <!-- some content that you wish to hide first ... -->
</div>

```
So far so good. Here's the JS.

```javascript
//controller.js
function MyController($scope, MyService) {
        var vm = this;

        vm.toggleHide = false;

        $scope.$watch( function () {
            return MyService.id;
        }, function (newValue, oldValue) {
            vm.toggleHide = true;
            console.log('The old value is' + oldValue);
            console.log('The new value is' + newValue);
            console.log('Is old value equals to new value? :' + (oldValue === newValue));
        });
}
```

What you will realise is that upon loading, `toggleHide` will be set to true. The scary part is that `MyService.id` never changed. Your console statements will tell you that `oldValue` is indeed equals to `newValue`. 

Instead, you can try to modify your code this way, by setting a default value for `MyService.id`, for instance to the string "abc". And only watch for changes away from this value.

Like so:

```javascript
//controller.js
function MyController($scope, MyService) {
        var vm = this;

        vm.toggleHide = false;

        $scope.$watch( function () {
            return MyService.id; // this will default to "abc"
        }, function (newValue, oldValue) {
            if (newValue === "abc") {
                return
            }
            vm.toggleHide = true;
        });
}       
```
Hope this helps!




---
layout: post
title:  $transitions instead of $stateChange for Angular authentication
date:   2017-08-04 20:55:12 +0800
categories: angular
---

A common way of implementing Angular authentication is to add properties to your states, as shown below. 

See examples [here](https://medium.com/@mattlanham/authentication-with-angularjs-4e927af3a15f) and [here](https://solidfoundationwebdev.com/blog/posts/require-authentication-for-certain-routes-with-ui-router-in-angularjs).

Notice that on Admin state we add an authenticate property, whereas on the Login state we don't have it - it's meant to be open to all. 

```javascript
//route.js
function uiRouteConfig($stateProvider, $urlRouterProvider) {
    $stateProvider
        .state('admin',{
            url: '/admin',
            templateUrl: '/app/admin/admin.html',
            controller: 'AdminC as con',
            authenticate: true
        })
        .state('login',{
            url: '/login',
            templateUrl: '/app/login/login.html',
            controller: 'LoginC as con'
        });

    $urlRouterProvider .otherwise('/home');
}
```

To implement authentication, we previously use $stateChangeStart property. 

But this has been deprecated, instead we now use $transitions. 


Before: 

```javascript
//app.js
angular.module("MyApp", ["ui.router"]).run(init);

init.$inject = ['$rootScope', '$state', 'passportService'];

function init($rootScope, $state, passportService){
    $rootScope.$on("$stateChangeStart", function(event, toState, toParams, fromState, fromParams){
        if (toState.authenticate) {
            // perform my authentication which returns a promise
                return passportService.getAccessToken().then(function(token){
                    // do anything needed
                })
                .catch(function(err){ 
                    $state.transitionTo("login");
                    event.preventDefault();
                });
    });
}
```


Here is AFTER, just replacing the part defining init().

```javascript
//app.js

init.$inject = ['$transitions', '$state', 'passportService'];
 
function init ($transitions, $state, passportService) {
    $transitions.onStart( { to: '**' }, 
        function(trans){
            var toState = trans.to(); 
            if (toState.authenticate) { 
                // perform my authentication which returns a promise
                return passportService.getAccessToken().then(function(token){
                    // do anything needed
                })
                .catch(function(err){ 
                    $state.transitionTo("login");
                    event.preventDefault();
                });
            }
    });
}
```

We just $transitions directly, without attaching it to $rootScope. Instead, we call `.onStart()` which takes an object and the callback. 

You can use wildcards in the object, or filter to specific routes such as `{ to: 'auth.**' }` - where you would need to defined nested states under the auth parent state.

The callback function is the object `trans` as the parameter. Which we can call properties on, such as `trans.to()` for the toState.

Hope you find this helpful!


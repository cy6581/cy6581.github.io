---
layout: post
title:  Common mistake in selecting nested objects in JS
date:   2017-08-30 18:07:12 +0800
categories: 'JavaScript'
---

Wonder why your nested object selections are returning "undefined" or throwing errors?

This is a common error in JS. I have made this mistake a few times over, and helped troubleshoot it for my friends.

Say you have a JavaScript object like so, which you wish to loop through for every user within. 

```javascript
const awesome = {
    url: "https://myawseomesite.io",
    users: {
        john: {
            password: "abc123",
            email: "john@example.com"
        },
        jane: {
            password: "abc123",
            email: "jane@example.com"
        }
        // assume we have other users
    }
}
```

Let's say you'd like to automate a login for each user, using an automator tool such as Selenium. You might write a script like this: 

```javascript 
// assumes Selenium web-driver loaded
let d = new webdriver.Builder().build();
d.get(awesome.url);

for (let each in awesome.users) {
    let user = awesome.users.each;
    d.findElement(selectors.userIdField).sendKeys(user.email);
    d.findElement(selectors.userPassword).sendKeys(user.password);
}
```

And guess what - "email is undefined". Why? Because `each` is a variable. You cannot call it with the . (dot syntax). You need the [] bracket notation.

This may lead to another common error.

```javascript 
for (let each in awesome[users]) {
    let user = awesome[users][each];
    d.findElement(selectors.userIdField).sendKeys(user[email]);
    d.findElement(selectors.userPassword).sendKeys(user[password]);
}
```

Now everything is in square brackets... and you get another error! Bracket syntax needs to be in a string for non-variable properties. 

Finally, like so: 

```javascript
for (let each in awesome.users) {
    let user = awesome.users[each];
    d.findElement(selectors.userIdField).sendKeys(user.email);
    d.findElement(selectors.userPassword).sendKeys(user.password);
}
```

Hope this helps! Till next time.
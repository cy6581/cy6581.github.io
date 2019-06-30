---
layout: post
title: JS Scrolling in Angular or Selenium
date: 2017-08-31 10:44:07 +0800
categories: ['HTML', 'Javascript', 'Angular', 'Selenium', 'Testing']
---

While making our first MVP I came across the [Angular scroll library](https://github.com/oblador/angular-scroll).

We wanted an 'onclick' function to scroll the screen down to an image loaded dynamically through AJAX. But we didn't know the exact size of the images as they were fetched from Facebook's CDN.

The scrollTo() gave us unpredictable results because the image usually rendered after the scroll completed, usually with half of off-screen. How then?

### Workaround

What we did was to hardscroll the element to page bottom, regardless of size. An easy way to do this was an additional scrollTo() offset of Y pixels below the element.

Recall how offsets work in CSS. With reference to the Window object, it counts the number of pixels below the TOP (y-offset) and number of pixels right of the LEFT (x-offset).

### Under the hood of Angular Scroll

Later, I found that Angular Scroll uses the [.getBoundingClientRect() method](https://developer.mozilla.org/en/docs/Web/API/Element/getBoundingClientRect) to figure out the element's position. It returns you properties like .top, .bottom, .left, .right - see what these mean at [this nice diagram](https://javascript.info/coordinates).

You can combine this with the native JS method of [window.scrollBy(xOffset, yOffset)](https://developer.mozilla.org/en-US/docs/Web/API/Window/scrollBy).

In fact, you may not need Angular Scroll for simple cases.

### Scrolling in Selenium

I found myself using this in Selenium, when the scrollIntoView() didn't centre the button that I wanted to click properly, which sometimes led to problems clicking.

[Bill Agee here](http://blog.likewise.org/2015/04/scrolling-to-an-element-with-the-python-bindings-for-selenium-webdriver/) shared a similar way to hardcode an additional scroll offset to centre the element, for Python Selenium bindings.

But this did not capture the edge cases where my buttons are already at the bottom of the page. Initiating an upward scroll (-Y offset) would cover my button.

Adding an If-statement to test the button's position solved the issue. Code in NodeJs:

```javascript
const webd = require('selenium-webdriver');
const d = new webd.Builder().withCapabilities(webd.Capabilities.chrome()).build();

let bookbutton = d.wait(webd.until.elementLocated(chosenClass), 15000);

//scrollIntoView default param true, will push the elem to the top of the page
d.executeScript('arguments[0].scrollIntoView()', bookbutton);

// check if button is less than 300px from top, scroll up 350)
let checkHeightScript =
	'if(arguments[0].getBoundingClientRect().top < 300){ window.scrollBy(0, -350)}';
d.executeScript(checkHeightScript, bookbutton);
bookbutton.click();
```

Note: The [Selenium documents](https://seleniumhq.github.io/selenium/docs/api/javascript/module/selenium-webdriver/lib/until.html) are not that clear. But you need `.until()` method on the Webdriver object, i.e. `webdriver.until` or `webd.until` in my code above.

Till next time!

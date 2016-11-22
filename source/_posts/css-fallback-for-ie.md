---
title: css fallback for IE(11+)
date: 2016-11-21 16:45:40
tags:
  - css
  - ie
---

*我也不知道我當初為什麼會硬著頭皮寫英文的，反正就是英文的了哈哈哈*

## 1. grayscale
  ```css
    -webkit-filter:grayscale(100%);
    filter:grayscale(100%);
   ```
  - use javascript. see tutorial [here](http://www.majas-lapu-izstrade.lv/useful/cross-browser-grayscale-image-example-using-css3-js).
  - or just simply use picture in gray. (haha)

## 2. object-fit / object-position
```css
-o-object-fit:cover;
   object-fit:cover;
-o-object-position:center;
   object-position:center;
```
  - IE doesn't support `object-fit` after IE10, 
  so use `background-size` and `background-position` instead.

## 3. vertical centering with css translate
As the question asked [here](https://stackoverflow.com/questions/27990347/ie-showing-horizontal-scrollbar-after-dom-element-transformed/27990348#27990348),
when using css translate to center contents, 
after scrolling the webpage, IE would somehow translate dom elements in the way that were not expected.


This is what i encountered:

The original page looks like this.
![](https://i.imgur.com/Xk1cv6S.jpg)
After scrolling to other section(with jquery), the first section was "moved up".
![](https://i.imgur.com/980k1Fm.jpg)

The html and css:
```html
<!-- the first section in the picture -->
<div class="page" id="first-section">
    <div class="page-content">
    <!--  some contents here  -->
    </div>
</div>
<!-- some other sections -->
<div class="page">
    <div class="page-content" id="other-section">
    <!--  some contents here  -->
    </div>
</div>
...
```
```css
.page {
    height: 100vh;
    min-height: 850px;
}
.page-content {
    position: absolute;
    top: 50%; 
    left: 50%;
    transform: translate(-50%, -50%);
    -webkit-transform: translate(-50%, -50%);
}
```

I found out it's the css in `.page-content` that caused the unexpected translation (in IE only of course),
so after using `bottom` and `right` instead of `top` and `right` as suggested,
everything works fine magically again!

```css
/* this works in IE (and all other browsers) */
.page-content {
    position: absolute;
    bottom: 50%; 
    right: 50%;
    transform: translate(50%,50%);
    -webkit-transform: translate(50%,50%);
}
```
Guess this is just some kind of bug in IE, since this doesn't happen in other browsers. Stupid IE.
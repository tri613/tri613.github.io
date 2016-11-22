---
title: :nth-child v.s. :nth-of-type
date: 2016-11-21 15:55:37
tags:
- css
---

## :nth-child

`:nth-child` 最容易誤會的地方是，它並不是搜尋父元素下的第幾個順序的selector子元素，而是單純在父元素下的**第幾個子元素**，基本上跟元素本身type或selector無關，*selector只是附加限制條件*。

```html
<div class="parent">
    <div class="child-div"></div> 
    <!-- .parent .child-div:nth-child(1) -->
    <div class="child-div"></div>
    <!-- .parent .child-div:nth-child(2) -->
    <a class="child-a"></a>
    <!-- .parent .child-a:nth-child(3) -->
    <a class="child-a"></a>
    <!-- .parent .child-a:nth-child(4) -->
</div>
```
```css
.parent .child-div:nth-child(2) {
    color:red; /* this will work */
}
.parent .child-a:nth-child(1) {
    color:blue; /* this won't work */
}
```

## :nth-of-type

`:nth-of-type` 就比較偏向搜尋某個父元素下面的**第幾個指定類型的子元素**，但要注意的是它只能指定html的本身的種類（像是`<a>`、`<div>`等等），而*不能用class名稱做為selector*。

```html
<div class="parent">
    <div class="child-div"></div> 
    <!-- .parent div:nth-of-type(1) -->
    <div class="child-div"></div>
    <!-- .parent div:nth-of-type(2) -->
    <a class="child-a"></a>
    <!-- .parent a:nth-of-type(1) -->
    <a class="child-a"></a>
    <!-- .parent a:nth-of-type(2) -->
</div>
```
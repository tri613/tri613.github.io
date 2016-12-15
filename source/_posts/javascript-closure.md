---
title: Javascript closure
date: 2016-12-15 12:50:54
tags:
- javascript
---

Closure一直是我覺得JS裡面很玄的東西，
最近又熊熊想到(?)回去好好看它，
這篇主要是記錄自己卡點的地方，如果有錯歡迎指正。


## 定義

{% blockquote Mozilla https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures %}
Closures are functions that refer to independent (free) variables (variables that are used locally, but defined in an enclosing scope). In other words, these functions 'remember' the environment in which they were created.
{% endblockquote %}

{% blockquote Programming Concepts Help http://conceptf1.blogspot.tw/2013/11/javascript-closures.html %}
Closures can simply be defined as an inner function in JavaScript has the access to all variable define in outer functions.
So when a function is invoked in JavaScript , it creates a new execution context. This context has access to Parent objects with the arguments for current function get invoked, this execution context also has access to the variables declared outside of its scope.
{% endblockquote %}

考慮以下的例子：

{% jsfiddle rLhfxgo7 js,result dark %}

`greeting` 被賦值的時候，會觸發並執行 anonymous function，
這個anonymous function則會回傳另一個function。
而這個function，也就是閉包，它記住了它的環境上下文，
所以它可以取得在它scope之外的變數 `me`，
即使 `me` 的值被改變，`greeting`仍然可以取得 `me` 正確的值。

這就是一個最簡單的closure。

## Creating closures in loops: A common mistake 

Mozilla關於Closure的文件上有一段[Creating closures in loops: A common mistake](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures#Creating_closures_in_loops_A_common_mistake)，
裡面的例子我卡了很久才想通：(這邊是直接用Mozilla上面的fiddle)

{% jsfiddle v7gjv js,html,result dark %}

照理來說，我們希望在focus不同input的時候，會出現各自的提示訊息，
但這邊卻只會顯示最後一筆 `Your age (you must be over 16)`。
原因是在創建onfocus的callback的時候，它其實是一個closure，
所以它記得的是被創建時的**上下文**，
也就是說onfocus被觸發的時候，它的callback會知道去哪裡找 `item.help` 的值，
但在for loop完了之後，`item.help`的值已經變成 `Your age (you must be over 16)`了，
而這也是為什麼上面的範例中，不管選哪個它都只會顯示同一個訊息。

```javascript
for (var i = 0; i < helpText.length; i++) {
    var item = helpText[i];
    document.getElementById(item.id).onfocus = function() {
      showHelp(item.help); 
    }
}
// 在for loop外面印出item
console.log(item); // {'id': 'age', 'help': 'Your age (you must be over 16)'}
```
> 所以說Closure基本上就是一個inner function會記得被創建時的外層上下文環境，
而這個inner function可以取得parent scope的變數，
它知道變數該指向的`對象`，但並不包含變數當下的`值`。

## Creating closures in loops: Solutions 

解法在 [Mozilla](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures#Creating_closures_in_loops_A_common_mistake) 上面都有提供，這邊稍微再補充那裡面提到的第一種方式。

{% jsfiddle v7gjv/1 js,html,result dark %}

根據 Mozilla 的說明，之前 for loop 會出問題，是因為所有的callback function在創建時，
全部都指向了同一個環境，所以指向的`item`都會是同一個，
不像上面是各自創了一個獨立環境。

這個說法對我來說有點抽象，我比較喜歡[Programming Concepts Help](http://conceptf1.blogspot.tw/2013/11/javascript-closures.html)的解釋：
> If we pass a parameter function makes its own local copy of the variable (if it is not object type which pass by reference). So each time function has its own local copy of variable which not get updated by loop iteration.

豁然開朗啊！JS真是博大精深。(謎)

## 參考資料

- [Mozilla - Closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures)
- [Programming Concepts Help](http://conceptf1.blogspot.tw/2013/11/javascript-closures.html)
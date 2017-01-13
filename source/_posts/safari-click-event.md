---
title: Click events not working on Safari
date: 2017-01-12 23:54:29
tags:
- ios
- safari
---

## Safari 無法觸發點擊事件

今天準備要下班的時候被同事抓住說safari上出現了一個bug，
說按鈕怎樣點擊都沒有反應，並且只有safari有這個問題。

原本以為是js又出了什麼問題，殊不知用console直接trigger點擊事件完全ＯＫ
證明了是safari無法捕捉到使用者點擊的事件。

後來一查發現這根本是個safari的bug，官方還偷偷把這部分的說明拿掉（囧），
根本就是想偷偷把bug隱藏起來啊！！！（重點是這個bug你又沒修掉！！！）

僅此紀錄解決方法，以後大家少走點冤枉路。ＱＱ

## 原因

 **其實它就是bug，結束。**（欸）

看mdn上面的說法是因為safari自己有定義哪些elements是可點擊哪些是不可點擊的，
所以如果點擊對象是nonclickable，自然就不會觸發click event。
（啊結果你官方文件又沒說你定義哪些是可點擊的！！！簡直要爆氣！！！）

還好MDN上面有列出來，可以看[這邊](https://developer.mozilla.org/en-US/docs/Web/Events/click#Safari_Mobile)參考。

## 解法

紀錄一下兩個我覺得比較實用的解法備存。（不然哪天文件消失又要哭哭了ＱＱ）

- 為點擊對象添加 `cursor: pointer` 的css style
- 添加`onclick="void(0)"`的屬性

我今天是使用加css的方式，然後就好了。（囧）
Apple我明明這麼相信你的技術力啊！！！！！

## 參考文件

- [iOS Safari 点击事件无效](https://www.zfanw.com/blog/ios-safari-click-not-working.html)
  (我今天倉促的找到第一個解法來源是這裡，超級感謝ＱＱ)
- [MDN: click#Safari_Mobile](https://developer.mozilla.org/en-US/docs/Web/Events/click#Safari_Mobile)

至於Apple自己的官方文件則是都被移掉了⋯⋯徹底無言⋯⋯

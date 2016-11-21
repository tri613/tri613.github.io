---
title: APNs with HTTP/2
date: 2016-11-21 17:02:24
tags: 
  - apns
  - ios 
  - push

---

*這篇是從tumblr上面搬來的，當時發表時間是2016/08/12。*

---

其實我也沒有接過第一版的iOS推播，不過被交付survey新版文件的工作， 就稍微記錄一下幾項看完的重點XD

---

1. 因為anps新版api用的是 HTTP/2，為了能夠與他們的server溝通，
	- PHP版本需要 >= 5.5.24 (我們只有5.4.40)
	- Curl的library版本需要 >= 7.38.0 (我們只有7.19.7)
	- OpenSSL >= 1.0.2e (我們只有1.0.1e)

2. 看起來它需要新的憑證，而這個新版憑證沒有所謂的 ‘test’ 或者是 ‘production’之分，只要建好了， 不管是測試環境還是正式環境都可以使用。以下是官方文件中的摘錄：
	> The HTTP/2-based provider API lets you use a single certificate for both development and production environments. For details on obtaining an Apple Push Notification service (APNs) certificate that works in both environments, read [Creating a Universal Push Notification Client SSL Certificate](https://developer.apple.com/library/content/documentation/IDEs/Conceptual/AppDistributionGuide/AddingCapabilities/AddingCapabilities.html#//apple_ref/doc/uid/TP40012582-CH26-SW11).

3. 新版的運作方式可以不用建立socket，可以改用curl POST方法， 除了接口 & 憑證 (還有程式環境版本問題)，其他內容格式(ex. payload)應該是差不多的。

4. payload的大小限制從2kb -> 4kb

5. 如果在短時間內一直瘋狂重開連線，apple會認定你是在攻擊然後就把連線關閉。(官方文件好像沒有特別提到後續的處理或結果)

6. 現在新版api會立即回傳response給你，不像之前舊版如果發生錯誤還會有時間差。 至於舊版的錯誤處理，官方文件有提供一些建議 基本上應該就是一直寫一直寫，寫到壞掉的時候，(fwrite的結果為false?) 有可能是滿了或怎樣，就再試著寫一次， 如果還是寫不進去就讀讀看，如果回傳大小為0，就表示連線可能因為是大小問題or格試錯誤等等被關掉了， 但假如大小為6bytes，就表示他們有回傳錯誤訊息，然後就可以根據錯誤訊息debug一下這樣。 －－っと、原本是這樣想的，但在stackoverflow看到有人說
	> $result is always true not mathers if sending was successful or not

	看來是我想得太天真ㄌ…

※備註：
在[這裡](https://onesignal.com/blog/apple-just-released-http2-support-for-their-push-notification-api/)有人提到：
> Any errors cause the TCP socket to be closed, even if valid notifications had been sent over the connection after a single invalid one. This often happens in cases where developers might accidentally mixed push identifiers between Development and Production versions of their app.

感覺我們舊版偶爾會爆炸可能就是因為這個…… 
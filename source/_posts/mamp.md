---
title: 在mac上設置apache + php + mysql
date: 2016-11-21 16:52:52
tags: 
  - mamp
  - osx
---

## 前前提要

*這篇是從tumblr上面搬來的，當時發表時間是2016/08/25。*

## 前情提要一下

本人上上禮拜敗了一台mac pro，想說架個伺服器以便未來開發 

原本在windows下我都是懶惰直接灌wamp或xampp這種一次打包好的東西， 

所以本來想說mac也來找找有沒有類似的就好了（←很懶 

殊不知估狗一下才發現原來mac有內建阿帕契！甚至連PHP都已經裝好了！！ 

只有mysql需要自己灌，真是太神拉~~~（灑花）

不過話是這樣說，實際上在setup的過程中還是碰到了不少問題（中途甚至不小心玩壞怒重灌了一次XD），所以還是記錄一下整個過程好了。

<!-- more -->

\# 環境　 

- 作業系統  OSX 10.11.6 

- Apache 2.4.18 (Unix) 

- PHP 5.5.36

<!--more-->

* * *

## 設定 apache

既然mac都已經內建好apache了，就先來啟動apache server看看吧！

```bash
$ sudo apachectl start
```
如果可以連到 [localhost](http://localhost) 就表示成功了！

接下來就是一些apache的設定等等了， 

像我就會把port設成1337然後引入一些需要的module等等。 

設定檔位置在 `/etc/apache2/httpd.conf` ， 

在finder下 cmd + shift + g 直接輸入上述位置即可。

我自己的httpd.conf主要是改這些東西：

*   port 改成 1337

```apacheconfig
Listen 1337
```

*   載入需要的模組 （也就是把前面的＃拿掉）

```apacheconfig
LoadModule rewrite_module libexec/apache2/mod_rewrite.so</span>
LoadModule php5_module libexec/apache2/libphp5.so
```

*   啟用 Virtual hosts

```apacheconfig
# Virtual hosts
Include /private/etc/apache2/extra/httpd-vhosts.conf
```

*   加入路徑讀取權限 

    如果不加，會發生  因為沒有權限而存取被拒的問題。（**client denied by server configuration** ） 

    這好像是比較偷懶的權限設法，因為是本地開發沒關係， 

    對外可能就不是這樣寫了XD 

    路徑我自己是設成這樣 /Users/tri613/workspace/www/

```apacheconfig
<Directory "/your/path/to/site/documents">
  Options Indexes MultiViews FollowSymLinks
  AllowOverride All
  # Apache 2.4
  Require all granted
</Directory> 
```

*   設定user / group 

    也是跟權限有關的設定，對linux(unix)不是特熟所以只是先設著保險（？）
	
```apacheconfig
User your_user_name 
Group staff
```

最後記得伺服器要重開剛才套用的東西才會有作用！

```bash
$ sudo apachectl restart
```
* * *

## 設置 virtual host

設置virtual hosts就只要修改 `etc/apache2/extra/httpd-vhosts.conf` 這個檔案就可以了！

```apacheconfig
# httpd-vhosts.conf

# 第一個是如果沒有符合的網址，就會導到這裡
<VirtualHost *:1337>
    DocumentRoot "/your/project/site"
    ServerName my.default.site.local
    ErrorLog "/your/logs/error.log"
</VirtualHost>

# 之後的就隨意了
<VirtualHost *:1337>
    DocumentRoot "/Users/tri613/workspace/www/my-project"
    ServerName myproject.local
    ErrorLog "/Users/tri613/workspace/www/log/myproject@error_log"
</VirtualHost>
```
以上內容只是範例啦，我都很懶惰只有指定這幾項而已ＸＤ

另外就是要注意 `etc/hosts` 這隻檔案要加上hosts， 

不然打剛剛自己設定的網址是會找不到的。
```apacheconfig
# etc/hosts
# 以上面的virtual hosts為例
127.0.0.1   myproject.local
```
大致上就是這些！

* * *

## 安裝 Mysql

先去官方網站下載，然後安裝，結束～～～（誤） 

安裝過程會有一串預設的密碼要記著， 

不然到時候沒密碼會登不進去。

安裝好了以後，要記得先到 **系統偏好設定** 裡面找到mysql並把mysql server開啟，不然不能用的。

接下來就先加環境變數吧！<strike>（我絕對不會說我為了加這個把其他的指令都蓋掉惹）</strike>
```bash
# 一開始應該都不會有這個檔案，所以就建一個新的
$ touch ~/.bash_profile
```
然後在 `.bash_profile` 裡面加上
```bash
export PATH=/usr/local/mysql/bin:$PATH
```
這行就可以了！

接下來就是一些mysql的初始設定等等摟。
```bash
//首次登入
$ mysql -u root -p 
//輸入剛才安裝中給的預設密碼
//登入成功後，要先換密碼
mysql > SET PASSWORD = PASSWORD('your_new_password');
```
到這裡就可以用root身份進行一般mysql操作了，通常後續還會創root以外的使用者等等的，但因為是本地開發所以就先忽略哈哈。

確定mysql可以用了以後，就是把PHP跟mysql連在一起啦～
```bash
cd /var
mkdir mysql
cd mysql
ln -s /tmp/mysql.sock mysql.sock
```
到這裡就完成基本的MAMP架設啦～ 

雖然感覺很簡單但我也是裝了兩遍啊哈哈哈～～～（謎

* * *

## 參考資料

*   [[Mac]不用懶人包，在 OS X 上安裝 Apache, PHP, MySQL](https://blog.allenchou.cc/mac-apache-php-mysql-setup/)
*   [Apache2: ‘AH01630: client denied by server configuration’](http://stackoverflow.com/questions/18392741/apache2-ah01630-client-denied-by-server-configuration)
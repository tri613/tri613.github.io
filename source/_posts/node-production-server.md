---
title: Node production server
date: 2017-07-20 11:17:13
tags:
    - node.js
    - nginx
    - linux
---

看了一下大家似乎是用nginx + pm2在作這些事情
但我覺得多一個pm2就好麻煩喔為什麼要這樣逼大家嗚嗚
不曉得如果要弄到ec2上又是怎樣
MIS真的不是人在幹的(。

<!-- more -->

## 環境

- Ubuntu 16.04
- node v7.10.1
- npm v4.2.0

### 安裝各種東西

#### 設定網路 & 安裝 openssh server
```
$ sudo apt install openssh-server
```
- 沒有CURL的話
```
sudo apt-get update; sudo apt-get install curl
```

#### 安裝 node

- 推薦方法

```
# Install Node.js 7.x repository
curl -sL https://deb.nodesource.com/setup_7.x | bash -

# Install Node.js and npm
apt-get install -y nodejs
```

- 用nvm (buggy)

會buggy的原因是nvm都灌在使用者自己的根目錄下面
如果到別的地方要用or sudo就會怪怪的
但我不想修這種bug所以果斷放棄^^

```
$ apt-get update
$ curl https://raw.githubusercontent.com/creationix/nvm/v0.25.0/install.sh | bash
```
如果出現 `Close and reopen your terminal to start using nvm`
不想重開可以打下面的指令

```
$ source ~/.profile
```

```
$ nvm install 7
```

#### 安裝 nginx
```
$ sudo apt-get install nginx
```

#### 安裝 pm2
```
$ sudo npm install -g pm2
```

## 設定 nginx

```
$ sudo vim /etc/nginx/sites-available/default
```

寫成像下面這樣

```yaml
server {
    listen 80;
    return 301 https://$host$request_uri;
}

server {
    listen 443;
    server_name tri.example.com;

    ssl_certificate           /etc/nginx/cert.crt;
    ssl_certificate_key       /etc/nginx/cert.key;

    ssl on;
    ssl_session_cache  builtin:1000  shared:SSL:10m;
    ssl_protocols  TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers HIGH:!aNULL:!eNULL:!EXPORT:!CAMELLIA:!DES:!MD5:!PSK:!RC4;
    ssl_prefer_server_ciphers on;

    access_log            /var/log/nginx/example.access.log;

    location / {
      proxy_pass          http://localhost:8080;
      # proxy_read_timeout  90;
      proxy_http_version 1.1;

      proxy_set_header        Host $host;
      proxy_set_header        X-Real-IP $remote_addr;
      proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header        X-Forwarded-Proto $scheme;

      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection 'upgrade';
      proxy_set_header Host $host;
      proxy_cache_bypass $http_upgrade;

      proxy_redirect      http://localhost:8080 https://tri.example.com;
    }
}
```

設定完記得重啟一下就行。

```
$ sudo service nginx restart
```

### SSL

上面那個有設定ssl，憑證開發用的可以自簽
雖然瀏覽器會說你這個是可怕的網站但沒關係
Production再換成正式憑證就好
自簽憑證：
```
$ sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/nginx/cert.key -out /etc/nginx/cert.crt
```

參考資料：https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-nginx-in-ubuntu-16-04


### IP判斷

在 Server block裡面簡單加上 
```
allow 192.168.13.13
deny all
```
之類的就可以了。

但如果有load balance什麼東西等等，就要透過 `ngx_http_realip_module` 來獲取client真實IP，
這我沒設過但下面有兩篇蠻清楚的講解文，下次再試。

- https://leo108.com/pid-2132.asp
- https://www.x4b.net/kb/RealIP-Nginx

## 架 node server

我都放在 `/var/www/` 下面，這邊就只是把code clone下來放著(?)

```
$ cd /var/www
$ sudo git clone https://some-git-repository.git myproject
```

### 備註

Nginx 基本上就是真正對外的server，
所以為了不讓外面的人有辦法直接連到node server，
可以把 node server綁在一個IP上 (像是localhost)

express範例如下：

```javascript
app.listen(port, '127.0.0.1', () => console.log(`Express server is listening on *:${port}`));
```

這樣外面的人就無法透過IP直接連到node server了。
(也有人說你就直接設防火牆就好了ㄚ)

## 使用 pm2 啟動 node server

現在有了node server的程式碼，就只差把它啟動而已
雖然我也很想直接 `node index.js` 就好
但如果它突然中斷我會很害怕
所以只好靠 `pm2` 來幫忙

### 寫 pm2 設定檔

因為我有用到 `NODE_ENV` 去區分我要連線的DB，
所以寫一個config檔放在專案的根目錄裡

```javascript
// pm2.config.js
module.exports = {
  apps : [
      {
        name: "push-usagi",
        script: "./index.js",
        watch: true,
        ignore_watch: ["node_modules"],
        env: {
            "NODE_ENV": "development"
        },
        env_testing: {
            "NODE_ENV": "testing"
        },
        env_production: {
            "NODE_ENV": "production"
        }
      }
  ]
};
```

### 啟動

因為我有寫config檔，所以就用config檔來作為啟動設定(?)

先切到專案目錄下

```
$ cd /var/www/myproject
```

然後執行pm2啟動 node server

```
$ pm2 start pm2.config.js --env production
```

看一下它的狀態

```
$ pm2 ls
```

好了，它開起來了。
現在它如果意外被關閉會自己重啟，
而因為有設定 `watch`的選項，
更新程式的時候它會自己reload server。

### 開機時自動啟動pm2程序

```
$ pm2 startup systemd
```

這一步都是已經把我要自動開的service開起來後才作的，
照我的理解是，如果作這步的時候沒有service
之後開完要再輸入

```
$ pm2 save
```
才可以，但我沒有試成功過就是了。

上面做完如果重開VM不用作任何事情就可以連到網頁就大功告成啦。:v: 

### 其他

pm2還有支援一些deploy & load balance等等的機制，
但這對我來說太over了所以暫時就這樣吧
有需要可以翻翻他們的github
[pm2 on github](https://github.com/Unitech/pm2)

## SO? Why nginx?

> ~~都裝完了才在問~~ (。

看起來nginx在以下項目上的表現會比直接用node server好，或者是寫node處理會太繁瑣：

- serve static files
- gzip
- load balance (across machine or port的那種，不是node本身的cluster，而是真的開很多node server的load balance)
- 同一台server要架很多站 (virtual host)
- 綁 80 or 443 port (主要是權限問題，如果可以sudo執行好像就沒差)
- SSL ~~(但node本身也可以設SSL，大家是說這樣比較不好，但到底哪裡不好也沒特別提囧)~~
  - 關於SSL這邊有提到是效率問題：[Does your Node.js code need a web server to run?](https://www.quora.com/Does-your-Node-js-code-need-a-web-server-to-run)
- a more proper error handling (至少當你node server crash的時候大家連進來還會有 502 bad gateway)
- 減少application server (like node, python...) 本身的loading

所以說不是不能直接裸開node server，只是使用nginx來處理某些事情會更輕鬆可靠，大家才會建議這麼做吧。

(那我的API server到底有沒有必要呢...感覺除了SSL好像都沒差...)

## 參考資料

- [How To Set Up a Node.js Application for Production on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-node-js-application-for-production-on-ubuntu-16-04)
- [How To Use PM2 to Setup a Node.js Production Environment On An Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-use-pm2-to-setup-a-node-js-production-environment-on-an-ubuntu-vps)
- [NGINX 設定 HTTPS 網頁加密連線，建立自行簽署的 SSL 憑證](https://blog.gtwang.org/linux/nginx-create-and-install-ssl-certificate-on-ubuntu-linux/)
- [5 Performance Tips for Node.js Applications](https://www.nginx.com/blog/5-performance-tips-for-node-js-applications)
- [Deploying django in a production server](https://stackoverflow.com/questions/20210495/deploying-django-in-a-production-server)
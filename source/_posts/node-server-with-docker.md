---
title: Node server with Docker
date: 2017-08-03 18:18:27
tags:
    - docker
    - node.js
    - nginx
    - linux
---

我發現Docker基本原則是這樣的，就是

**一個container只運行一個服務**
**一個container只運行一個服務**
**一個container只運行一個服務**

（很重要所以說三遍）

所以以前我們可能會把 nginx + node 全部都一起裝在一台VM上，
但現在如果要把它docker化，就變成要開兩個container，一個`nginx`一個`node server`的服務。
而不是直接開一個ububtu(之類的)的container然後把全部東西灌在上面build成一個image。

<!-- more -->

所以如果要把上述的東西docker化，檔案結構會長成這樣
```
/ (專案根目錄)
| -- docker-compose.yml
` -- node
|    | -- index.js
|    | -- Dockerfile
`-- nginx
|    | -- nginx.conf
|    | -- Dockerfile
```

完整範例可以參考我的 [github](https://github.com/tri613/simple-docker-playground)

Docker安裝走這邊：(ubuntu 16.04)
https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-16-04

## 一些名詞簡介

OK，所以到底啥是 `Dockerfile` ? 啥又是 `docker-compose` ?

### Dockerfile

> 其實它就是用來 build image 的步驟描述檔。

如果你要建立image檔，有兩種方式：

1. 先下載一個base image (像是ubuntu or centos那種)，就像是開一台VM一樣，
然後在你開起來的那個container裡面做平常你會在VM裡面做的事情，
再把這個container的狀態`commit`上去變成一個image檔。

2. 把你所有要做的事情寫成一個 `Dockerfile`，然後直接下指令build就好!!

我覺得寫 Dockerfile 的好處是，你不用真的把你的image推到像是docker hub的地方，
而是直接給別人你的 Dockerfile + 需要的源碼讓人家去build就好
感覺這樣進版本庫也比較輕量(?)

### docker-compose

一開始提到了，Docker的基本原則是，**一個container只負責一項服務**
但基本上不太可能一個APP只用到一個服務 (像是本文就用到了 node + nginx)
所以需要 `docker-compose` 來一次幫你把你的 **APP** (服務群?) 開好
(不然也是可以自己先把所有的image build好再一個個開起來啦只是會開到起笑)

那怎麼知道要開哪些服務哩?

這時候就需要用到 `docker-compose.yml` 這個設定檔了
而這個設定檔怎麼寫，下面會有範例所以這邊不多說了

所以兩者關係是這樣的：
> 一個 `Dockerfile` 可以build成一個 `image`，
一個 `container` 則是一個 `image` 的實體，
而 `docker-compose.yml` 就是描述很多個 `container` (or service) 之間的交互作用。(吧)

好像沒有很總結到的感覺XDDDD，直接往下看程式比較快(ㄜ)

## node

### index.js

（毫無反應，就只是個node server）

陽春到極致的簡單範例：

```javascript=
const http = require('http')
const port = process.env.PORT || 8080

const requestHandler = function handler(req, res) {
  res.end("Hello! This is a simple node server!")
}

const server = http.createServer(requestHandler)

server.listen(port, () => console.log(`Node simple server is now listening on *:${port}`))
```

### Dockerfile

- building from ubuntu

```dockerfile=
FROM ubuntu:trusty

# install curl & node
RUN apt-get update && \
    apt-get -y install curl && \
    curl -sL https://deb.nodesource.com/setup_7.x | bash - && \
    apt-get install -y nodejs

WORKDIR  /src
ADD . /src

EXPOSE 8080

CMD ["node", "/src/index.js"]
```

- building from node (with pm2)

直接用官方的node image環境當底，
這個方法build起來速度快很多XD

```dockerfile=
FROM node:7.10.1

RUN npm install pm2 -g

WORKDIR  /src
COPY . /src

EXPOSE 8080

CMD ["pm2-docker", "process.yml"]
```

- process.yml

這個是 pm2 在用的啦，順便列一下

```yaml=
apps:
  - script   : './index.js'
    name     : 'node-server'
    exec_mode: 'cluster'
    instances: 4
```

`EXPOSE` 好像只是讓大家知道 **container** 打算要使用哪個port，
真正要運行的時候，還是需要加上類似`-p xxxx:8080`這樣的東西。

但因為我們不會直接build images出來，而是透過 `docker-compose` 來建立整套服務，
所以到時候寫在 `docker-compose.yml`裡面就可以了。

#### 手動 build image 並啟動

假如沒有要透過 `docker-compose` 而只是想要啟用單一服務的話，
也可以先手動建立image然後啟動。

`Dockerfile` 就是拿來build image用的，
所以我們現在其實已經擁有所有需要的素材了。

先切到 `node` 本身的目錄下面：
```
$ cd your-project/node
```

建立image：
```
$ docker build -t your-namespace/node .
```

build完以後執行啟動

```
$ docker run -d -p 3000:8080 your-namespace/node
```

連到 `*:3000` 就可以看到我們的node伺服器正常運作中拉，Hooray!

## nginx

### nginx.conf

這 nginx 的 config 也是直接複製網路上範例，(所以我沒有深究它在幹嘛XD)
注意連到 node server 的服務的時候用的是 `http://nodejs:8080`
其中 `nodejs` 是寫在 `docker-compose.yml`裡面 node server的 `name`
port 則是當初在 node Dockerfile 裡面`EXPOSE`指定的 port

```config=
worker_processes 4;

events { worker_connections 1024; }

http {
    server {
            listen 80;
        
            location / {
                proxy_pass http://nodejs:8080;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection 'upgrade';
                proxy_set_header Host $host;
                proxy_cache_bypass $http_upgrade;
            }
    }
}
```

#### nginx config 參數配置參考

- http://www.ha97.com/5194.html

### Dockerfile

```dockerfile=
FROM nginx

COPY nginx.conf /etc/nginx/nginx.conf
```

## docker-compose

### docker-compose.yml
首先在根目綠下面加上 `docker-compose.yml` 這個檔案
它主要就是來告訴 `docker-compose` 說要開哪些服務 
然後這些服務彼此之間的關係等等

```yaml=
nginx:
    build: ./nginx
    links:
        - nodejs:nodejs
    ports:
        - "80:80"
nodejs:
    build: ./node
    ports:
        - "8080"  # 對外沒有port!!!!! 所以外面的人是無法直接連到 node 服務的
```

### 啟動

在專案根目錄下面執行：
```
$ docker-compose up -d
```

注意 `docker-compose` 是要另外裝的，所以沒有要記得裝。

詳細安裝教學：(ubuntu 16.04)
https://www.digitalocean.com/community/tutorials/how-to-install-docker-compose-on-ubuntu-16-04

然後就好啦!!!

### 更新

假如更新程式的話怎麼辦?????
像是我的 `node` 裡面的程式碼如果更新了要怎麼辦???

因為我這邊用的是 `COPY`，所以基本上程式碼算是和docker分開管理(的吧)

我現在的想像是整個project都進版本庫管理
然後在專案底下執行 `git pull` or `svn up` 之類的完畢後，使用

```
$ docker-compose up --build -d
```
來建立新版本並更新。

(而且我發現這可以在已經執行中的狀態下使用，可以無痛更新ㄚ!!!!!)

注意 `--build` 這個參數，如果沒有用的話他就會使用已經建立過的版本下去跑，
這樣你更新的程式碼就完全沒用了~

其他很多像是 `ADD` 跟 `COPY` 差在哪啦，什麼樣的更新方式比較OK啦
這類的細節和討論我不是很清楚，
總之先可以跑起來就好了XD (欸)

## 參考資料

- http://anandmanisankar.com/posts/docker-container-nginx-node-redis-example/
- http://schempy.com/2015/08/25/docker_nginx_nodejs/
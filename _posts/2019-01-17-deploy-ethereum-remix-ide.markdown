---
layout: post
title:  "Deploy Ethereum remix-ide"
date:   2019-01-17 12:12 +0800
categories: Ethereum
---
Recently I have to deploy a modified ethereum remix IDE on the public network. From the [github repo][remix-ide-github] and most blogs/tech forums, they talked about `localhost`. At first I tried to change the ip or port to my vps ip, but they won't work. finally I found that the port or ip do not need to be changed. I used Ubuntu linux.

For GNU/Linux:

### Install from npm

``` sh
npm install remix-ide -g
remix-ide
```

#### Install from source

``` sh
git clone https://github.com/ethereum/remix-ide.git
git clone https://github.com/ethereum/remix.git # only if you plan to link remix and remix-ide repositories and develop on it.
cd remix-ide
npm install
npm run setupremix  # only if you plan to link remix and remix-ide repositories and develop on it.
npm start
```

Then you can access from `http://localhost:8080`

if you deploy in the cloud, just open port 8080, and access from `http://you.ip.or.domain:8080`


[remix-ide-github]:https://github.com/ethereum/remix-ide
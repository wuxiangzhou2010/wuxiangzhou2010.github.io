---
layout: post
title: "nvm node version manager"
date: 2019-03-15 11:15 +0800
categories: tools
published: false
---

- [NVM node version manager](https://github.com/creationix/nvm)

  nvm is a great tool to manange node version, this means download install and select which version to use on the fly.

[How to use nvm](https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-an-ubuntu-14-04-server)

```sh
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash
source ~/.bashrc
nvm install node
nvm install 8.14.1
nvm list
nvm list-remote
nvm use 8.14.1
nvm install --lts
```

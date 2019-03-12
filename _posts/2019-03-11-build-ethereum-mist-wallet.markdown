---
layout: post
title: "Build mist wallet 0.10.0"
date: 2019-03-11 16:52 +0800
categories: Ethereum
published: false
---

## config foleder

linux: `~/.config/Mist`

```sh
rm -rf ~/.config/Mist
```

## dependencies

- nodejs 8
- meteor
- yarn
- Electron
- Gulp

```sh
curl https://install.meteor.com/ | sh
curl -o- -L https://yarnpkg.com/install.sh | bash
yarn global add electron@1.8.4
yarn global add gulp
```

## get repo

```sh
git clone https://github.com/ethereum/mist.git

cd mist
git reset --hard v0.10.0
yarn
```

## for dev

```sh
# 1. first terminal

cd mist/interface && meteor --no-release-check

# 2. second terminal

cd my/path/meteor-dapp-wallet/app && meteor --port 3050
# npm install @babel/runtime@7.0.0-beta.49

# 3. third terminal

cd mist
yarn dev:electron --mode wallet
```

## generate wallet

```sh
# install multi lib for ia32 platform
sudo apt-get install gcc-multilib g++-multilib

npm install -g meteor-build-client
gulp --linux --wallet --x64
```

reference:

- [ethereum mist v0.10.0](https://github.com/ethereum/mist/tree/v0.10.0)

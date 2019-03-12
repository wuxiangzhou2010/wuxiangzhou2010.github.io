---
layout: post
title: "Build mist wallet 0.10.0"
date: 2019-03-11 16:52 +0800
categories: Ethereum
published: true
---

Mist `0.10.0` it is the last version that use web3.js `0.xx.xx` series. To not break dependencies in my other projects, I need to use this version rather the newer ones.

As far as I know, the major difference between web3.js 0.xx and 1.xx is that 1.xx is mostly using async mechanism and web3 is divided into several packages while 0.xx can use async and sync mechanism and 0.xx web3.js is packed into a single package.

1. Config foleder

   linux: `~/.config/Mist`

   ```sh
   rm -rf ~/.config/Mist
   ```

2. Dependencies

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

3. Get repo

   ```sh
   git clone https://github.com/ethereum/mist.git

   cd mist
   git reset --hard v0.10.0
   yarn
   ```

4. For dev

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

5. Generate wallet

   ```sh
   # install multi lib for ia32 platform
   sudo apt-get install gcc-multilib g++-multilib

   npm install -g meteor-build-client
   gulp --linux --wallet --x64
   ```

Reference:

- [ethereum mist v0.10.0](https://github.com/ethereum/mist/tree/v0.10.0)

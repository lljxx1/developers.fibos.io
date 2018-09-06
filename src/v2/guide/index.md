---
title: 介绍
type: guide
order: 2
---


## 起步

<p class="tip">该指南假设你已了解关于 fibos, fibos.js的中级知识。</p>


## 启动一个fibos单机节点
在开始写合约之前我们需要先启动一个**开启了Javascript合约**的单机节点，或者使用一个已经开启了Javascript合约的**fibos(测试网/主网)**的节点

目前主网和测试网络暂时还未开启Javascript合约，所以我们需要在本机运行一个单节点的fibos
``` js

// 引入fibos包
var fibos = require('fibos');
var fs = require('fs');

function clean_folder(path) {
    var dir = fs.readdir(path);
    console.log("clean", path);
    dir.forEach(function (name) {
        var fname = path + '/' + name;
        var f = fs.stat(fname);
        if (f.isDirectory()) {
            clean_folder(fname);
            fs.rmdir(fname);
        } else
            fs.unlink(fname);
    });
}

// 每次重启清空data-dir
console.log(fibos.data_dir);
try {
    clean_folder(fibos.data_dir);
} catch (e) {}

console.log(fibos.config_dir);
try {
    clean_folder(fibos.config_dir);
} catch (e) {}


// 配置rpc监听的端口
fibos.load("http", {
    "http-server-address": "0.0.0.0:8889"
});

// 开启合约console的支持
fibos.load("chain", {
    "contracts-console": true
});

// 配置p2p监听端口
fibos.load("net", {
    "p2p-listen-endpoint": "0.0.0.0:9877",
});

// 开启js合约
fibos.enableJSContract = true;

// 配置producer 为eosio
fibos.load("producer", {
    'producer-name': 'eosio',
    'enable-stale-production': true,
    'max-transaction-time': 3000
});

fibos.load("chain_api");
fibos.load("history_api");

// 启动
fibos.start();

```

我们已经成功启动了一个fibos单机节点，接下来需要创建一个作为合约部署的帐号


## 创建合约帐号

为合约帐号生成一对keypair

``` js
var FIBOS = require('fibos.js');
var prikey = FIBOS.modules.ecc.randomKeySync(); //私钥
var pubkey = FIBOS.modules.ecc.privateToPublic(prikey); //公钥

console.log(prikey, pubkey);

// 5HsFtSEjcqWvtyprTodwfJLcLATjF6kLxyPRWj3sVY86eJPnuad 
// FO5NUW1YZWsjFJK42QWbZL8S4rSmcRRMG7tsHd3wKRAyQQHRXFEg

```

使用eosio创建合约部署帐号**testoooooooo**

``` js

// 引入fibos.js
const FIBOS = require("fibos.js");
const config = {
  // 默认chainId
  chainId: "cf057bbfb72640471fd910bcb67639c22df9f92470936cddc1ade0e2f2e7dc4f",
  // 默认KEY
  priKey: "5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3",
  httpEndpoint: "http://127.0.0.1:8889",
  verbose: false,
}

const fibos = FIBOS({
  chainId: config.chainId,
  httpEndpoint: config.httpEndpoint,
  keyProvider: config.priKey,
  verbose: false,
  logger: {
      log: null,
      error: null
  }
})


const name = 'testoooooooo';

// publickey
const pubkey = 'FO5NUW1YZWsjFJK42QWbZL8S4rSmcRRMG7tsHd3wKRAyQQHRXFEg';

fibos.newaccountSync({
  creator: 'eosio',
  name: name,
  owner: pubkey,
  active: pubkey
});

fibos.buyrambytesSync({
  payer: 'eosio',
  receiver: name,
  bytes: 8192
});

fibos.delegatebwSync({
  from: 'eosio',
  receiver: name,
  stake_net_quantity: '10.0000 SYS',
  stake_cpu_quantity: '10.0000 SYS',
  transfer: 0
});

```

好了，你现在有一个帐号了，接下来就是编写合约代码，再以这个帐号部署合约



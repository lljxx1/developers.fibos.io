---
title: 合约编写
type: guide
order: 3
---


## 合约ABI定义

什么是ABI
ABI相当于是合约的接口定义文档， ABI里包含了一些`action`, `table`, `struct` 的定义



``` js
var abi = {
    "version": "eosio::abi/1.0",
    "structs": [
    // 结构体 hi定义 
    {
        "name": "hi",
        "base": "",
        "fields": [{
            "name": "user",
            "type": "name"
        }, {
            "name": "content",
            "type": "string"
        }]
    }],
    "actions": [
        // 定义了一个叫hi的action, 类型结构体为 hi
        {
            "name": "hi",
            "type": "hi",
            "ricardian_contract": ""
        }
    ]
};
```


`fibos.js`可以根据abi定义生成相应的接口对象

``` js
// fibos.js client会尝试调用远程rpc接口获取该合约帐号的abi定义
const customContract = fibos.contractSync('testoooooooo');
console.log(customContract.hi);

```



## 编写合约

以上ABI定义了名为`hi`的一个`action`，我们需要在合约代码中实现这个方法。
该`action`可以接收两个`fields`。


分别是
 - `name` | `user`
 - `string`  |`content`


``` js
exports.hi = (user, content) => {
    console.log(user, content)
}
```


## 编译+部署合约


编译合约主要需要用到`fibos.js`库中的`compileCode/compileModule`来对javascript代码进行编译
部署合约主要用到`setcode`,`setabi`这两个RPC API

``` js
// 引入fibos.js
var FIBOS = require("fibos.js");
var fs = require('fs');

var config = {
   chainId: "cf057bbfb72640471fd910bcb67639c22df9f92470936cddc1ade0e2f2e7dc4f",
   // 刚刚创建testoooooooo 帐号的privatekey
   priKey: "5HsFtSEjcqWvtyprTodwfJLcLATjF6kLxyPRWj3sVY86eJPnuad",
   httpEndpoint: "http://127.0.0.1:8889",
   verbose: false,
}

// 初始化一个rpc client
var fibosClient = FIBOS({
  chainId: config.chainId,
  httpEndpoint: config.httpEndpoint,
  keyProvider: config.priKey,
  verbose: false,
  logger: {
      log: null,
      error: null
  }
})


// 合约abi定义
var abi = {
    "version": "eosio::abi/1.0",
    "structs": [
    // 结构体 hi定义 
    {
        "name": "hi",
        "base": "",
        "fields": [{
            "name": "user",
            "type": "name"
        }, {
            "name": "content",
            "type": "string"
        }]
    }],
    "actions": [
        // 定义了一个叫hi的action, 类型结构体为 hi
        {
            "name": "hi",
            "type": "hi",
            "ricardian_contract": ""
        }
    ]
};


// 这里也可以通过 fs.readFileSync() 来读取外部文件的代码
// 或者使用 fibosClient.compileModule(path) 来编译一个合约目录
var js_code = `exports.hi = (user, content) => {  console.log(user, content) }`;

fibosClient.setcodeSync('testoooooooo', 0, 0, fibosClient.compileCode(js_code))
fibosClient.setabiSync('testoooooooo', abi);

```



## 调用合约

到目前为止你已经学会了怎么定义+编译+部署一个合约，接下来就是在（`browser/node/fibos`）客户端执行合约了

``` js
// 引入fibos.js
var FIBOS = require("fibos.js");
var fs = require('fs');

var config = {
   chainId: "cf057bbfb72640471fd910bcb67639c22df9f92470936cddc1ade0e2f2e7dc4f",
   // 刚刚创建testoooooooo 帐号的privatekey
   priKey: "5HsFtSEjcqWvtyprTodwfJLcLATjF6kLxyPRWj3sVY86eJPnuad",
   httpEndpoint: "http://127.0.0.1:8889",
   verbose: false,
}

// 初始化一个rpc client
var fibosClient = FIBOS({
  chainId: config.chainId,
  httpEndpoint: config.httpEndpoint,
  keyProvider: config.priKey,
  verbose: false,
  logger: {
      log: null,
      error: null
  }
})

var myContract = fibosClient.contractSync('testoooooooo');
var res = myContract.hiSync('funis', '这是一个测试', {
    authorization: 'testoooooooo'
});

// 合约action内部执行console.log的日志
console.log(res.processed.action_traces[0].console)

console.log(res);
// 返回
{
  "broadcast": true,
  "transaction": {
    "compression": "none",
    "transaction": {
      "expiration": "2018-09-06T03:39:02",
      "ref_block_num": 21394,
      "ref_block_prefix": 1541296618,
      "max_net_usage_words": 0,
      "max_cpu_usage_ms": 0,
      "delay_sec": 0,
      "context_free_actions": [],
      "actions": [
        {
          "account": "testoooooooo",
          "name": "hi",
          "authorization": [
            {
              "actor": "testoooooooo",
              "permission": "active"
            }
          ],
          "data": "0000000000eca65e12e8bf99e698afe4b880e4b8aae6b58be8af95"
        }
      ],
      "transaction_extensions": []
    },
    "signatures": [
      "SIG_K1_KhJFsoSgqDgYFqoWX36ck4stPJFqFwhQ37DP7QNhKDfyb3xmpP156vdCjs45gq4xT8mjHfrtxnP7DrBBRKxFqfHgakAmqv"
    ]
  },
  "transaction_id": "12e708d89d204361a7be65fa9601d75bbbfd2ad7e5c024c26d9acc3a3efe237f",
  "processed": {
    "id": "12e708d89d204361a7be65fa9601d75bbbfd2ad7e5c024c26d9acc3a3efe237f",
    "receipt": {
      "status": "executed",
      "cpu_usage_us": 1080,
      "net_usage_words": 15
    },
    "elapsed": 1080,
    "net_usage": 120,
    "scheduled": false,
    "action_traces": [
      {
        "receipt": {
          "receiver": "testoooooooo",
          "act_digest": "ba7d8a0abc0f12c93e57f0d9e3ae671ac3e9e32089402c5db9a037b67e0c41cc",
          "global_sequence": 86948,
          "recv_sequence": 4,
          "auth_sequence": [
            [
              "testoooooooo",
              13
            ]
          ],
          "code_sequence": 3,
          "abi_sequence": 6
        },
        "act": {
          "account": "testoooooooo",
          "name": "hi",
          "authorization": [
            {
              "actor": "testoooooooo",
              "permission": "active"
            }
          ],
          "data": {
            "user": "funis",
            "content": "这是一个测试"
          },
          "hex_data": "0000000000eca65e12e8bf99e698afe4b880e4b8aae6b58be8af95"
        },
        "elapsed": 906,
        "cpu_usage": 0,
        "console": "funis 这是一个测试\n",
        "total_cpu_usage": 0,
        "trx_id": "12e708d89d204361a7be65fa9601d75bbbfd2ad7e5c024c26d9acc3a3efe237f",
        "inline_traces": []
      }
    ],
    "except": null
  }
}

```


## 调试合约

节点需要开启`contracts-console`的支持
在合约内部执行的`console.log` 都会收集在返回值的`processed.action_traces`里的 `console`属性里

``` js
var myContract = fibosClient.contractSync('testoooooooo');
var res = myContract.hiSync('funis', '这是一个测试', {
    authorization: 'testoooooooo'
});

// 合约action内部执行console.log的日志
console.log(res.processed.action_traces[0].console)
funis 这是一个测试
```
---
title: 合约进阶
type: guide
order: 4
---


## 用户权限

在合约内部中我们会需用判断用户的权限，判断用户是否存在之类的一些情况
这个时候我们需要用到`action`模块，这是一个全局对象


``` js
exports.hi = (user, content) => {
    console.log(action);

    if(!action.has_auth(user)){
      console.log('无权限')
    }

    if(!action.is_account(user)){
        console.log('请提供一个存在的帐号')
    }else{
        console.log('帐号存在')
    }
}
```



- 直接调用hi 

``` js
var res = myContract.hiSync('funis', '这是一个测试', {
    authorization: 'testoooooooo'
});

console.log(res.processed.action_traces[0].console)
请提供一个存在的帐号

```



- 我们换`eosio`作为 `user`参数

``` js
var res = myContract.hiSync('eosio', '这是一个测试', {
    authorization: 'testoooooooo'
});
console.log(res.processed.action_traces[0].console)
无权限
帐号存在
```


- **参考** [action](../api/index.html#action)



## 数据库

在合约内部中我们可能会需要用到数据库
abi里定义的table都会注入到db对象下


- ABI定义tables
``` js
var db_abi = {
    "version": "eosio::abi/1.0",
    "types": [{
        "new_type_name": "my_account_name",
        "type": "name"
    }],
    "structs": [
    //  player结构体
    {
        "name": "player",
        "base": "",
        "fields": [{
            "name": "title",
            "type": "string"
        }, {
            "name": "age",
            "type": "int32"
        }]
    }, {
        "name": "hi",
        "base": "",
        "fields": [{
            "name": "user",
            "type": "name"
        }]
    }],
    "actions": [{
        "name": "hi",
        "type": "hi",
        "ricardian_contract": ""
    }],
    "tables": [
    // 定义players表 type为player结构体,以 my_account_name作为主键
    {
        "name": "players",
        "type": "player",
        "index_type": "i64",
        "key_names": ["nickname"],
        "key_types": ["my_account_name"]
    }, {
        "name": "players1",
        "type": "player",
        "index_type": "i64",
        "key_names": ["id"],
        "key_types": ["int64"]
    }]
};

```
- 在合约里操作数据表 

``` js
exports.hi = (user) => {
    // 初始化数据库
    // 执行帐号的scope, code
    var players = db.players(action.account, action.account);
    console.log((players.name, players.code, players.scope);
    players.emplace(action.account, {
        title: "ceo",
        age: 48,
        nickname: "lion1"
    });
}
```
- 在fibos.js查询数据表

``` js
fibosClient.getTableRowsSync({
    json: true,
    scope: 'testoooooooo',
    code: 'testoooooooo',
    table: "players"
})

{
  "rows": [
    {
      "title": "ceo",
      "age": 48
    }
  ],
  "more": false
}
```


- **关于table请参考** [table](../api/index.html#action)



## inline action

有的时候我们可能需要在合约中调用其它合约的action，比如给用户发送token，这个时候需要用到 `trans`模块

``` js
exports.hi = (user) => {
    var players = db.players(action.account, action.account);
    console.log((players.name, players.code, players.scope);
    players.emplace(action.account, {
        title: "ceo",
        age: 48,
        nickname: "lion1"
    });

    trans.send_inline(
      "testooooo2", 
      "hi", 
      {
        user:"user1", 
        friend:"user2"
      }, 
      [{
        "actor": "testooooo", 
        "permission": "active"
      }]
    );
}
```

- **关于trans请参考** [trans](../api/index.html#trans)
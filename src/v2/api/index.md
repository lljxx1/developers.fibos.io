---
title: API
type: api
---


## fibos


### fibos.compileCode(code)

- **参数**：
  - `{String} code` 合约代码

- **用法**：

  编译字符串合约代码


  ``` js
  
  var FIBOS = require('fibos.js');
  var fibos = FIBOS();

  // 合约
  var code = `exports.hi = v => console.error(typeof v, v);`;

  // 编译合约
  var complieCode = fibos.compileCode(code);

  // 部署合约
  fibos.setcodeSync('contractaccount', 0, 0, complieCode);

  ```

### fibos.compileModule(path)

- **参数**：
  - `{String} path`  模块路径

- **用法**：

  编译模块化的合约

  ``` js
  // module 目录


  // /module/test/test2.js  
  ` module.exports = " test2";`

  // /module/test1.js 
  `var test2 = require("./test/test2"); 
    module.exports = " test1" + test2;`

  // 合约入口
  // /module/index.js 
  `var test1 = require('./test1');  
    exports.hi = user => console.log(user, test1);`

  var FIBOS = require('fibos.js');
  var fibos = FIBOS();

  // 编译模块合约
  var complieCode = fibos.compileModule(__dirname + '/module');

  // 部署合约
  fibos.setcodeSync('contractaccount', 0, 0, fibos.compileCode(js_code));

  ```




## trans

### trans.send_inline( account, name, args, authorization)

- **参数**：
  - `{String} account` 发送者的帐号名称
  - `{String} name` 名称
  - `{Object} args` 附带的数据
  - `{Array} authorization` action 的权限

- **用法**：

  在合约内部向特定的帐号发送inline action


  ``` js
  // hi acction
  exports.hi => (user) {
    // 触发hi2 action
    trans.send_inline(
      "test", 
      "hi2", 
      {
        user:"user1", 
        friend:"user2"
      }, 
      [{
        "actor": "${name}", 
        "permission": "active"
    }])
  };

  // hi2 action
  exports.hi2 = (user, friend) => {
    console.log(user, friend);
  }

  ```

  

- **参考**：[组件](../guide/components.html)

### trans.send_context_free_inline(account, name, args )

- **参数**：
  - `{String} account` 发送者的帐号名称
  - `{String} name` 名称
  - `{Object} args` 附带的数据

- **用法**：

  在合约内部向特定的帐号发送context_free inline action

  ``` js
  // hi acction
  exports.hi => (user) {
    // 触发hi2 action
    trans.send_context_free_inline(
      "test", 
      "hi2", 
      { 
        user:"user1", 
        friend:"user2" 
      }
    );
  };

  // hi2 action
  exports.hi2 = (user, friend) => {
    console.log(user, friend);
  }

  ```


## action

### name

- **类型**：`String`
- **详细**：

  action的名称

- **示例**：

  ``` js
  exports.hi = v => {
    console.log(action.name)
  };

  ```


### account

- **类型**：`String`
- **详细**：
  执行action的账户名称
  

- **示例**：
  ``` js
  exports.hi = v => {
    console.log(action.account)
  };
  ```

### receiver

- **类型**：`String`
- **详细**：

  action接收者

- **示例**：

  ``` js
  exports.hi = v => {
    console.log(action.receiver)
  };
  ```


### publication_time

- **类型**：`Long`
- **详细**：
  返回从1970年1月1日0时0分0秒（UTC，即协调世界时）距离出块时间的毫秒数

- **示例**：

  ``` js
  exports.hi = v => {
    console.log(action.publication_time)
  };
  ```

### authorization

- **类型**：`Array`
- **详细**：

执行该 action 需要得到数组中所有账户的授权

- **示例**：

  ``` js
  exports.hi = v => {
    console.log(action.authorization)
  };
  ```



### action.is_account(name )

- **参数**：
  - `{String} name`

- **返回**：
  - `{Boolean `

- **用法**：

  判断账户是否存在, 返回true 或者 false

  ``` js
  exports.hi = v => {
    console.error(
      action.is_account(action.account), 
      action.is_account("notexists")
    )
  };

  ```

### action.require_recipient(name)

- **参数**：
  - `{String} name`

- **用法**：

  向通知列表增加特定账号

  ``` js
  exports.hi = v => {
    action.require_recipient(action.receiver);
  };
  ```


### action.has_recipient(name )

- **参数**：
  - `{String} name`

- **用法**：

  action 执行成后，名为 name 的账号是否会收到通知

  ``` js
  exports.hi = v => {
     console.error(
       action.has_recipient(action.receiver), 
       action.has_recipient("test")
    );
  };
  ```



### action.has_auth(name )
- **参数**：
  - `{String} name`
- **用法**：
  验证 action 是否需要特定账户的授权

  ``` js
  exports.hi = v => {
    console.error(action.has_auth(action.receiver));
  };
  ```

### action.require_auth(name, permission)

- **参数**：
  - `{String} name`
  - `{String} permission`

- **用法**：

  向 action 的授权列表中添加特定账户及对应的权限，若添加失败则会抛出异

  ``` js
  // hi acction
  exports.hi = v => {
    console.error(action.require_auth(action.receiver))
  };
  ```

## table

ABI定义数据表

``` js
var abi =  {
    "version": "eosio::abi/1.0",
    "types": [{
        "new_type_name": "my_account_name",
        "type": "name"
    }],
    "structs": [{
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
    "tables": [{
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
}

```


表需要在abi里定义，相关的table才能在db对象访问到, 如db.players
- `players`以`nickname`作为索引，
- `players1`以`id`作为索引

``` js
// 初始化table对象
var players = db.players("code", "scope");
```

### name

- **类型**：`String`
- **详细**：表名


### code

- **类型**：`String`
- **详细**：指向合约发布者的名称

### scope

- **类型**：`String`
- **详细**：table 中数据所属的 account_name


### table.emplace(name, args)

- **参数**：
  - `{String} name`
  - `{Object} args`

- **用法**：

  向表中插入数据

  ``` js
  exports.hi = v => {
    var players = db.players(action.account, action.account);
    players.emplace(action.account, { 
      title: "ceo",
      age:48, 
      nickname:"lion1",
      id:123
    });
  };
  ```


### table.get(id )

- **参数**：
  - `{Value} id`

- **用法**：
  获取索引值为 id 的数据

  ``` js
  // hi acction
  exports.hi = v => {
    var players = db.players1(action.account, action.account);
    console.log(players.get(v))
  };
  ```


### table.modify(id, name, args)

- **参数**：
  - `{Value} id`
  - `{String} name`
  - `{Object} args`

- **用法**：
  修改索引值为 id 的对应的数据

  ``` js
  // hi acction
  exports.hi = (user) {
    var players = db.players(action.account, action.account);
    players.modify(
      123, 
      action.account, 
      { 
        title: "cto", 
        age:23, 
        id:123 
      }
    );
  };

  ```



### table.erase(id )

- **参数**：
  - `{Value} id`

- **用法**：
  删除索引值为 id 的数据

  ``` js
  // hi acction
  exports.hi => (user) {
    var players = db.players1(action.account, action.account);
    players.erase(123);
  };

  ```


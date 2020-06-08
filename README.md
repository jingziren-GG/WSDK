[发送消息| WSDK
im.taobao.com › wsdk_doc › Tribe › SendMsg](http://im.taobao.com/wsdk_doc/Tribe/SendMsg.html)

发送消息. Object.*Tribe*.*sendMsg*(param);. param.tid 群id param.msg 消息内容[param.msgTime] 发送消息的时间（单位：秒） [param.msgType] 消息类型，详情见下



## 监听事件

### Object.Event.on(event, callback, [context]);

```
event: 事件名称， 可以传递多个，以空格隔开
callback: 回调函数
[context]: callback中的this指向context
```

### DEMO

```js
    var sdk = new WSDK();

    var context = {
        name: 'wsdk'
    };

    sdk.Event.on('WSDK_EVENT1 WSDK_EVENT2 WSDK_EVENT3', function(data){
        console.log(data);
        console.log(this.name); // wsdk
    }, context);
```

## 取消监听事件

### Object.Event.off([event], [callback]);

```
[event]: 事件名称， 可以传递多个，以空格隔开
[callback]: 监听事件时传的回调函数
```

### DEMO

```js
    var sdk = new WSDK();

    var callback = function(data){
        console.log(data.eventName);
    };

    sdk.Event.on('WSDK_EVENT', callback);

    sdk.Event.fire('WSDK_EVENT', {eventName: 'WSDK_EVENT'}); // WSDK_EVENT

    // 删除指定事件名称，回调的监听
    sdk.Event.off('WSDK_EVENT', callback);
    // 删除指定事件名称下的所有监听
    sdk.Event.off('WSDK_EVENT');
    // 删除所有事件监听
    sdk.Event.off();

    sdk.Event.fire('WSDK_EVENT1', {eventName: 'WSDK_EVENT'}); // 不会console;
```



## 触发监听的事件

### Object.Event.fire(event, [data]);

```
event: 事件名称， 可以传递多个，以空格隔开
[data]: 触发事件时callback中第一个参数
```

### DEMO

```js
    var sdk = new WSDK();

    var callback = function(data){
        console.log(data.eventName);
    };

    sdk.Event.on('WSDK_EVENT', callback);

    sdk.Event.fire('WSDK_EVENT', {eventName: 'WSDK_EVENT'}); // WSDK_EVENT
```



## 登录

### Object.Base.login(param);

```
param.uid            要登录的用户id
param.appkey         appkey
param.credential     登录用户的校验码
[param.anonymous]    使用匿名的方式登录
[param.toAppkey]     当需要跨appkey聊天时使用
[param.timeout]      登录超时时间，默认5000ms
[param.success]      登录成功回调
[param.error]        登录失败回调
```

**使用匿名方式登录时，可以不传uid与credential**

[如何获取appkey](http://baichuan.taobao.com/doc2/detail?spm=0.0.0.0.ClNuWh&treeId=28&articleId=102565&docType=1#s2)

**由于部分浏览器不完全支持postMessenger, 比如内嵌了IE内核的QQ浏览器7系列，使用postMessenger时会抛出"没有权限"的错误，导致无法跨iframe通信而登录错误返回TIMEOUT，目前也没有很好的解决方案来解决此问题**
**WSDK会将这种不支持的错误拋出，返回给开发者，开发者可以自行决定如何提示用户**
**捕捉到此错误的方法：**
**全局设置__WSDK__POSTMESSAGE__DEBUG__，设置__WSDK__POSTMESSAGE__DEBUG__为布尔类型时，WSDK会alert出“对不起,当前浏览器不支持聊天,请更换浏览器”的错误；**
**设置__WSDK__POSTMESSAGE__DEBUG__为函数时，WSDK会调用此函数，并将错误信息通过第一个参数返回给开发者**

### DEMO

```js
    var sdk = new WSDK();

    sdk.Base.login({
        uid: 'uid',
        appkey: 'appkey',
        credential: 'credential',
        timeout: 5000,
        success: function(data){
           // {code: 1000, resultText: 'SUCCESS'}
           console.log('login success', data);
        },
        error: function(error){
           // {code: 1002, resultText: 'TIMEOUT'}
           console.log('login fail', error);
        }
    });
```

### 匿名方式登录

```js
    var sdk = new WSDK();
    sdk.Base.login({
        anonymous: true,
        appkey: 'appkey',
        timeout: 5000,
        success: function(data){
           // {code: 1000, resultText: 'SUCCESS'}
           console.log('login success', data);
        },
        error: function(error){
           // {code: 1002, resultText: 'TIMEOUT'}
           console.log('login fail', error);
        }
    });
```

### 捕捉浏览器不支持错误，使用WSDK默认的提示方式

```js
    // 当浏览器不支持时，会alert”对不起,当前浏览器不支持聊天,请更换浏览器“
    window.__WSDK__POSTMESSAGE__DEBUG__ = true;

    var sdk = new WSDK();

    sdk.Base.login({
        uid: 'uid',
        appkey: 'appkey',
        credential: 'credential',
        timeout: 5000,
        success: function(data){
           // {code: 1000, resultText: 'SUCCESS'}
           console.log('login success', data);
        },
        error: function(error){
           // {code: 1002, resultText: 'TIMEOUT'}
           console.log('login fail', error);
        }
    });
```

### 捕捉浏览器不支持错误，自定义错误处理

```js
    window.__WSDK__POSTMESSAGE__DEBUG__ = function(error){
        console.log(error); 
        alert('浏览器不支持，请切换浏览器');
    };

    var sdk = new WSDK();

    sdk.Base.login({
        uid: 'uid',
        appkey: 'appkey',
        credential: 'credential',
        timeout: 5000,
        success: function(data){
           // {code: 1000, resultText: 'SUCCESS'}
           console.log('login success', data);
        },
        error: function(error){
           // {code: 1002, resultText: 'TIMEOUT'}
           console.log('login fail', error);
        }
    });
```

## 获取未读消息的条数

### Object.Base.getUnreadMsgCount(param);

```
[param.count]        一次获取的条数，默认30条
[param.success]      获取成功回调
[param.error]        获取失败回调
```

**param.count解释：**比如登录用户有90个人给他发的消息都没读，其中有一个人给登录用户发了50条消息，那么param.count设置的是90这个维度的数量，而这个接口是用来获取50这个维度的数量

### 用法及功能

1. 第一个功能就是显而易见的功能，获取未读消息的条数
2. 由于目前没有获取最近联系群的接口，可以通过这个接口获取到最近联系的群

### DEMO

```js
    var sdk = new WSDK();
    var recentTribe = [];

    sdk.Base.getUnreadMsgCount({
        count: 10,
        success: function(data){
            console.log(data);
            var list = data.data;
            list.forEach(function(item){
                if(item.contact.substring(0, 8) === 'chntribe'){
                    recentTribe.push(item);
                }else{
                    console.log(item.contact.substring(8) + '在' + new Date(parseInt(item.timestamp)*1000) + ',');
                    console.log('给我发了' + item.msgCount + '条消息，最后一条消息是在' + new Date(parseInt(item.lastmsgTime)*1000) + '发的');
                }
            });

            recentTribe.length && console.log('最近给我发消息的群', recentTribe);
        },
        error: function(error){
            console.log('获取未读消息的条数失败' ,error);
        }
    });
```

### 返回的数据结构

![image](https://img.alicdn.com/tps/TB1dn_wLpXXXXbrXpXXXXXXXXXX-926-666.png)

### 返回的数据字段解释

|   字段名    | 解释                                                         |
| :---------: | :----------------------------------------------------------- |
|    code     | 返回值                                                       |
| resultText  | 返回值说明                                                   |
|    data     | 接口返回数据                                                 |
|     @t      | 忽略                                                         |
|   contact   | 给登录用户发消息的人或者群(前8位是chntribe表示群)            |
| lastmsgTime | 最后一条消息的发送时间，**注意单位是秒，操作前请自己乘以1000转成毫秒** |
|  msgCount   | 未读的消息条数                                               |
|    msgId    | 消息的id，**群的消息id总是为0**                              |
|  timestamp  | 第一条未读消息的时间 **注意单位是秒，操作前请自己乘以1000转成毫秒** |

## 获取最近联系人及最后一条消息内容

### Object.Base.getRecentContact(param);

```
[param.count]        一次获取的条数，默认30条
[param.success]      获取成功回调
[param.error]        获取失败回调
```

### DEMO

```js
    var sdk = new WSDK();

    sdk.Base.getRecentContact({
        count: 10,
        success: function(data){
            console.log(data);
            data = data.data;
            var list = data.cnts,
                type = '';

            list.forEach(function(item){
                console.log(item.uid.substring(8) + '在' + new Date(parseInt(item.time)*1000) + '联系了你');
                console.log('他说：' + item.type == 2 ? '[语音]' : item.type == 1 ? '[图片]' : (item.msg && item.msg[1]);
            });
        },
        error: function(error){
            console.log('获取最近联系人及最后一条消息内容失败' ,error);
        }
    });
```

### 返回的数据结构

![image](https://gw.alicdn.com/tps/TB17g6uJFXXXXchXXXXXXXXXXXX-936-1196.png)

### 返回的数据字段解释

|   字段名   | 解释                                                        |
| :--------: | :---------------------------------------------------------- |
|    code    | 返回值                                                      |
| resultText | 返回值说明                                                  |
|    data    | 接口返回数据                                                |
|   actor    | 请求发起者                                                  |
|    cnts    | 联系人列表                                                  |
| direction  | 忽略                                                        |
|    msg     | 最后一条消息                                                |
|   msg[0]   | 消息类型 T表示文本 P表示图片                                |
|   msg[1]   | 消息内容                                                    |
|    time    | 消息发送时间 **注意单位是秒，操作前请自己乘以1000转成毫秒** |
|    type    | 消息类型（与msg[0]重复） 0文本 1图片 2语音                  |
|    uid     | 消息发送者                                                  |
|    uuid    | 忽略                                                        |
|  uuid_str  | 消息id                                                      |
| timestamp  | 时间 **注意单位是秒，操作前请自己乘以1000转成毫秒**         |

## 开始接收当前登录用户的所有消息（单聊，群聊）

### Object.Base.startListenAllMsg();

**注意：此方法在网络波动(网络切换或者断网后又自动连上)时会返回code为1005的错误(只有监听START_RECEIVE_ALL_MSG事件才能接收到)，后续将会收不到消息，请在收到此错误后，自己处理重连的逻辑**
**推荐的重连逻辑：**
**1. 每隔10-20秒重新调用startListenAllMsg方法重试，设置调用上限比如3次**
**2. 如果可以知道网络已经恢复，那么只需要在网络恢复的时候，调用startListenAllMsg方法即可**

### 监听事件名称

- `START_RECEIVE_ALL_MSG`: 所有消息, 包括成功的消息与失败的消息
- `MSG_RECEIVED`: 只接收成功的消息
- `CHAT.MSG_RECEIVED`: 只接收成功的单聊消息
- `TRIBE.MSG_RECEIVED`: 只接收成功的群聊消息（群聊天消息，群系统消息）
- `KICK_OFF`: 收到被踢的通知

**注意：无法监听到自己当前发送的消息**

### DEMO

```js
    var sdk = new WSDK();

    var Event = sdk.Event;

    Event.on('START_RECEIVE_ALL_MSG', function(data){
        console.log('我所有消息都能收到', data);
    });

    Event.on('MSG_RECEIVED', function(data){
        console.log('我能收到成功的消息', data);
    });

    Event.on('CHAT.MSG_RECEIVED', function(data){
        console.log('我能收到成功的单聊消息', data);
    });

    Event.on('TRIBE.MSG_RECEIVED', function(data){
        console.log('我能收到成功的群聊消息', data);
    });

    Event.on('KICK_OFF', function(data){
        console.log('啊，我被踢了', data);
    });

    sdk.Base.startListenAllMsg();
```

### 返回的数据结构

![image](https://img.alicdn.com/tps/TB1TkjALpXXXXaJXpXXXXXXXXXX-936-574.png)

### 返回的数据字段解释

|    字段名    | 解释                        |
| :----------: | :-------------------------- |
|     code     | 返回值                      |
|  resultText  | 返回值说明                  |
|     data     | 接口返回数据                |
|    cmdid     | 忽略                        |
|     msgs     | 消息列表                    |
|  direction   | 忽略                        |
|     msg      | 最后一条消息                |
|    touid     | 相对于当前登录用户的对方uid |
|     uid      | 当前登录用户uid             |
|     time     | 消息发送时间                |
|     type     | 消息类型 **具体见下方表格** |
|     from     | 消息发送者                  |
|      to      | 消息接收者                  |
|     msg      | 消息内容                    |
| receiverFlag | 忽略                        |

### 单聊消息类型表

| 类型 | 描述     |
| :--: | :------- |
|  0   | 文本消息 |
|  1   | 图片消息 |
|  2   | 语音消息 |

### 群聊消息类型表

| 类型 | 描述                                                       |
| :--: | :--------------------------------------------------------- |
|  1   | 普通文本消息                                               |
|  3   | 用户加入群                                                 |
|  5   | 用户退出群                                                 |
|  7   | 群主转让                                                   |
|  8   | 取消管理员                                                 |
|  9   | 用户被请出群                                               |
|  12  | 停用群                                                     |
|  14  | 更新群信息                                                 |
|  15  | 通知群名片修改                                             |
|  16  | 图片消息                                                   |
|  35  | 被邀请者拒绝管理员加入群（用户手动拒绝）                   |
|  98  | 管理员拒绝用户加入群                                       |
| 101  | 被邀请者拒绝管理员加入群(用户在客户端有设置，系统自动拒绝) |
| 212  | 申请加入群消息                                             |

## 停止接收当前登录用户的所有消息

### Object.Base.stopListenAllMsg();

### DEMO

```js
    var sdk = new WSDK();

    var Event = sdk.Event;

    Event.on('START_RECEIVE_ALL_MSG', function(data){
        console.log('我所有消息都能收到', data);
    });

    Event.on('MSG_RECEIVED', function(data){
        console.log('我能收到成功的消息', data);
    });

    Event.on('CHAT.MSG_RECEIVED', function(data){
        console.log('我能收到成功的单聊消息', data);
    });

    Event.on('TRIBE.MSG_RECEIVED', function(data){
        console.log('我能收到成功的群聊消息', data);
    });

    Event.on('KICK_OFF', function(data){
        console.log('啊，我被踢了', data);
    });

    sdk.Base.startListenAllMsg();

    sdk.Base.stopListenAllMsg();
```

## 去除用户长id的前缀，即前8位

### Object.Base.getNick(str);

```
str     用户长id
```

[什么是前缀](http://im.taobao.com/wsdk_doc/faq.html)

### DEMO

```js
    var sdk = new WSDK();
    var luid = 'iwangxinvisitor4';

    var suid = sdk.Base.getNick(luid); // visitor4

    sdk.Base.getNick(suid); // visitor4
```

## 本地销毁WSDK

### Object.Base.destroy();

**调用此方法会将本地销毁WSDK，停止接收消息等，但是登录的账号在服务端还会保持20分钟的在线状态，20分钟后才会在服务端真正下线**

### DEMO

```js
    var sdk = new WSDK();

    sdk.Base.destroy();

    sdk = null;
```



## 登出

### Object.Base.logout();

```
[param.success]     登出成功回调
[param.error]       登出失败回调
```

登出后，将不会收到任何消息，也无法再调用需要登录的接口

### DEMO

```js
    var sdk = new WSDK();

    sdk.Base.login({
        uid: 'uid',
        appkey: 'appkey',
        credential: 'credential',
        timeout: 5000,
        success: function(data){
           // {code: 1000, resultText: 'SUCCESS'}
           console.log('login success', data);

           setTimeout(function(){
                sdk.Base.logout();
            }, 2000);
        },
        error: function(error){
           // {code: 1002, resultText: 'TIMEOUT'}
           console.log('login fail', error);
        }
    });
```



## 发送消息

### Object.Chat.sendMsg(param);

```
param.touid         对方的id
param.msg           消息内容
[param.msgType]     消息类型，详情见下表
[param.hasPrefix]   传入的touid是否已经带有前缀
[param.success]     发送成功回调
[param.error]       发送失败回调
```

[什么是前缀](http://im.taobao.com/wsdk_doc/faq.html)

### DEMO

```js
    var sdk = new WSDK();

    sdk.Chat.sendMsg({
       touid: 'touid',
       msg: '你好啊',
       success: function(data){
         console.log('消息发送成功', data);
       },
       error: function(error){
         console.log('消息发送失败', error);
       }
    });
```

### 单聊消息类型表

| 类型 | 描述     |
| :--: | :------- |
|  0   | 文本消息 |
|  1   | 图片消息 |
|  2   | 语音消息 |



## 发送自定义消息

### Object.Chat.sendCustomMsg(param);

```
param.touid         对方的id
param.msg           消息内容
[param.summary]     消息摘要，一般会用在native端收到推送消息时显示的文案，默认为msg
[param.hasPrefix]   传入的touid是否已经带有前缀
[param.success]     发送成功回调
[param.error]       发送失败回调
```

[什么是前缀](http://im.taobao.com/wsdk_doc/faq.html)

### DEMO

```js
    var sdk = new WSDK();

    sdk.Chat.sendCustomMsg({
       touid: 'touid',
       msg: 'xxx已经添加你为好友',
       summary: '添加好友',
       success: function(data){
         console.log('发送成功', data);
       },
       error: function(error){
         console.log('发送失败', error);
       }
    });
```



## 给千牛端客服发送消息

### Object.Chat.sendMsgToCustomService(param);

```
param.touid         对方的id
param.msg           消息内容
[param.msgType]     消息类型，详情见下表
[param.groupid]     在千牛开启了分流后，得到的groupid
[param.hasPrefix]   传入的touid是否已经带有前缀
[param.nocache]     不缓存联系人，每次都会重新走分流接口获取客服（不推荐）
[param.success]     发送成功回调
[param.error]       发送失败回调
```

[什么是前缀](http://im.taobao.com/wsdk_doc/faq.html)

**param.nocache解释：**sdk中会缓存上次传入的touid与子账号，不会再调用内部的接口获取子账号，如果设置了nocache为true, 那么会强制再次调用内部接口获取子账号，但是要注意，服务端其实也做了缓存，如果没有特殊情况(子账号离线、中途跟其他子账号聊过天等)，重新调接口拿到的子账号还是会跟之前的一样

### DEMO

```js
    var sdk = new WSDK();

    sdk.Chat.sendMsgToCustomService({
       touid: 'touid',
       msg: '你好啊',
       groupid: 123,
       success: function(data){
         console.log('send success', data);
       },
       error: function(error){
         console.log('send fail', error);
       }
    });
```

### 消息类型表

| 类型 | 描述     |
| :--: | :------- |
|  0   | 文本消息 |
|  1   | 图片消息 |
|  2   | 语音消息 |



## 获取历史消息

### Object.Chat.getHistory(param);

```
param.touid           对方nick    
[param.hasPrefix]     传入的touid中是否已带前缀  
[param.nextkey]       分页获取漫游历史消息时，服务端返回的下一页分页标示，如果要获取下一页，需要将这个标示带上，默认为''
[param.count]         一次获取的消息条数，默认20 
[param.success]       获取成功回调
[param.error]         获取失败回调
```

[什么是前缀](http://im.taobao.com/wsdk_doc/faq.html)

**关于count字段：**实际返回的消息条数，可能会比设置的count条数大，因为pc客户端发送的**图文/文本与链接**消息由于消息协议的原因会包含在一条消息内，但是jssdk会把图文消息拆开成多条，所以需要根据返回的nextkey来判断是否还有下一页消息

### DEMO

```js
    var sdk = new WSDK(),
    nextkey = '';

    sdk.Chat.getHistory({
        touid: 'touid',
        nextkey, nextkey,
        count: 10,
        success: function(data){
            console.log('获取历史消息成功', data);
            nextkey = data.data && data.data.next_key;
        },
        error: function(error){
            console.log('获取历史消息失败', error);
        }
     });
```

### 返回的数据结构

![image](https://img.alicdn.com/tps/TB1SwwcLpXXXXXxapXXXXXXXXXX-934-1046.png)

### 返回的数据字段解释

|   字段名   | 解释                                          |
| :--------: | :-------------------------------------------- |
|    code    | 返回值                                        |
| resultText | 返回值说明                                    |
|    data    | 接口返回数据                                  |
|  nextKey   | 下一页的标示，如果为空，则标示没有下一页      |
|   touid    | 对方的用户id，与调用接口是传入的touid保持一致 |
|    uid     | 自己的用户id                                  |
|    msgs    | 消息列表                                      |
|    from    | 消息的发送者                                  |
|     to     | 消息的接收者                                  |
|    time    | 消息发送时间 **注意单位是毫秒**               |
|    type    | 消息类型 0文本 1图片 2语音                    |
|    msg     | 消息内容                                      |
|   msgId    | 消息的唯一标示                                |

### 返回的消息类型表

| 类型 | 描述     |
| :--: | :------- |
|  0   | 文本消息 |
|  1   | 图片消息 |
|  2   | 语音消息 |

## 设置与某人的消息已读

### Object.Chat.setReadState(param);

```
param.touid         对方的id
[param.timestamp]   当前时间的秒数
[param.hasPrefix]   传入的touid是否已经带有前缀
[param.success]     发送成功回调
[param.error]       发送失败回调
```

[什么是前缀](http://im.taobao.com/wsdk_doc/faq.html)

**注意：许多开发者会碰到调用此方法无效，是由于调用的机器本地时间不准确，导致timestamp传入的时间不对，没有真正设置成功，推荐的方法是开发者自己保证时间的准确，或者将时间往后推迟，比如当前时间是11点，那么传入的时间设置成12点，一般可以保证消息可以正确的被设置为已读**

### DEMO

```js
    var sdk = new WSDK();

    sdk.Chat.setReadState({
        touid: 'touid',
        timestamp: Math.floor((+new Date())/1000),
        success: function(data){
            console.log('设置已读成功', data);
        },
        error: function(error){
            console.log('设置已读失败', error);
        }
    });
```

## 开始接收与某人的消息

### Object.Chat.startListenMsg(param);

```
param.touid         对方的id
[param.hasPrefix]   传入的touid是否已经带有前缀
```

[什么是前缀](http://im.taobao.com/wsdk_doc/faq.html)

**注意：此方法在网络波动(网络切换或者断网后又自动连上)时会返回code为1005的错误(只有监听CHAT_START_RECEIVE_MSG事件才能接收到)，后续将会收不到消息，请在收到此错误后，自己处理重连的逻辑**
**推荐的重连逻辑：**
**1. 每隔10-20秒重新调用startListenMsg方法重试，设置调用上限比如3次**
**2. 如果可以知道网络已经恢复，那么只需要在网络恢复的时候，调用startListenMsg方法即可**

### 监听事件名称

- `CHAT_START_RECEIVE_MSG`: 与传入的touid之间的所有消息, 包括成功的消息与失败的消息
- `MSG_RECEIVED`: 只接收成功的消息
- `CHAT.MSG_RECEIVED`: 只接收成功的单聊消息
- `KICK_OFF`: 收到被踢的通知

**注意：无法监听到自己当前发送的消息**

### DEMO

```js
    var sdk = new WSDK();

    Event.on('CHAT_START_RECEIVE_MSG', function(data){
        console.log('我与touid之间的所有消息', data);
    });

    Event.on('MSG_RECEIVED', function(data){
        console.log('我与touid之间能收到成功的消息', data);
    });

    Event.on('CHAT.MSG_RECEIVED', function(data){
        console.log('我与touid之间能收到成功的单聊消息', data);
    });

    Event.on('KICK_OFF', function(data){
        console.log('啊，我被踢了', data);
    });

    sdk.Chat.startListenMsg({
        touid: 'touid'
    });
```

### 返回的数据结构

![image](https://img.alicdn.com/tps/TB1ZbspLpXXXXbjXVXXXXXXXXXX-908-598.png)

### 返回的数据字段解释

|    字段名    | 解释                                          |
| :----------: | :-------------------------------------------- |
|     code     | 返回值                                        |
|  resultText  | 返回值说明                                    |
|     data     | 接口返回数据                                  |
|   nextKey    | 下一页的标示，如果为空，则标示没有下一页      |
|    touid     | 对方的用户id，与调用接口是传入的touid保持一致 |
|     uid      | 自己的用户id                                  |
|     msgs     | 消息列表                                      |
|     from     | 消息的发送者                                  |
|      to      | 消息的接收者                                  |
|     time     | 消息发送时间 **注意单位是毫秒**               |
|     type     | 消息类型 0文本 1图片 2语音                    |
|     msg      | 消息内容                                      |
|    msgId     | 消息的唯一标示                                |
| recevierFlag | 忽略                                          |

### 收到的消息类型表

| 类型 | 描述     |
| :--: | :------- |
|  0   | 文本消息 |
|  1   | 图片消息 |
|  2   | 语音消息 |

## 停止接收与某人的消息

### Object.Chat.stopListenMsg();

### DEMO

```js
    var sdk = new WSDK();

    Event.on('CHAT_START_RECEIVE_MSG', function(data){
        console.log('我与touid之间的所有消息', data);
    });

    Event.on('MSG_RECEIVED', function(data){
        console.log('我与touid之间能收到成功的消息', data);
    });

    Event.on('CHAT.MSG_RECEIVED', function(data){
        console.log('我与touid之间能收到成功的单聊消息', data);
    });

    Event.on('KICK_OFF', function(data){
        console.log('啊，我被踢了', data);
    });

    sdk.Chat.startListenMsg({
        touid: 'touid'
    });

    sdk.Chat.stopListenMsg();
```

## 获取用户在线状态

### Object.Chat.getUserStatus(param);

```
param.uids            需要获取的用户nick数组   
[param.hasPrefix]     传入的uid中是否已带前缀  
[param.appkey]        传入的uid中的appkey
[param.charset]       传入的uid的编码，默认utf-8; 可选：gbk
[param.success]       获取成功回调
[param.error]         获取失败回调
```

**param.uids解释：**uids必须为数组，最多传入30个用户 **param.appkey解释：**只有在`hasPrefix`为`false`时，`appkey`为必填项

**uids的前缀：**如果uids中的uid不带前缀，会根据传入的appkey对应的前缀；如果uids中的uid带前缀，请设置hasPrefix字段为true

**接口返回：**获取成功后，返回的状态也是数组，数组的顺序与你传入的uids的顺序一致

### DEMO

```js
    var sdk = new WSDK(),
          statusMap = {
             0: '离线',
             1: '在线'
         };

    sdk.Chat.getUserStatus({
        uids: ['iwangxinvisitor1', 'iwangxinvisitor2', 'iwangxinvisitor3'],
        hasPrefix: true,
        success: function(data){
              console.log('visitor1的状态为': statusMap[data.result.status[0]]);
              console.log('visitor2的状态为': statusMap[data.result.status[1]]);
              console.log('visitor3的状态为': statusMap[data.result.status[2]]);
        },
        error: function(){
            console.log('批量获取用户在线状态失败');
        }
     });
```

### 返回的数据结构

![image](https://img.alicdn.com/tps/TB1CtUKLpXXXXcaXXXXXXXXXXXX-896-362.png)

### 返回的数据字段解释

|   字段名   | 解释                                                   |
| :--------: | :----------------------------------------------------- |
|    code    | 返回值                                                 |
| resultText | 返回值说明                                             |
|    data    | 接口返回数据                                           |
|   status   | 与传入的uids顺序一致的在线状态值，0表示离线，1表示在线 |

## 发送消息

### Object.Tribe.sendMsg(param);

```
param.tid           群id
param.msg           消息内容
[param.msgTime]     发送消息的时间（单位：秒）
[param.msgType]     消息类型，详情见下表
[param.hasPrefix]   传入的touid是否已经带有前缀
[param.success]     发送成功回调
[param.error]       发送失败回调
```

### DEMO

```js
    var sdk = new WSDK();

    sdk.Tribe.sendMsg({
       tid: 'tid',
       msg: '你好啊',
       success: function(data){
         console.log('发送群消息成功', data);
       },
       error: function(error){
         console.log('发送群消息失败', error);
       }
    });
```

### 单聊消息类型表

| 类型 | 描述     |
| :--: | :------- |
|  0   | 文本消息 |
|  1   | 图片消息 |
|  2   | 语音消息 |



## 获取历史消息

### Object.Tribe.getHistory(param);

```
param.tid             群号   
[param.nextkey]       分页获取漫游历史消息时，服务端返回的下一页分页标示，如果要获取下一页，需要将这个标示带上，默认为''
[param.count]         一次获取的消息条数，默认20 
[param.success]       获取成功回调
[param.error]         获取失败回调
```

**关于count字段：**实际返回的消息条数，可能会比设置的count条数大，因为pc客户端发送的**图文/文本与链接**消息由于消息协议的原因会包含在一条消息内，但是jssdk会把图文消息拆开成多条，所以需要根据返回的nextkey来判断是否还有下一页消息

### DEMO

```js
    var sdk = new WSDK(),
    nextkey = '';

    sdk.Tribe.getHistory({
        tid: 'tid',
        nextkey, nextkey,
        count: 10,
        success: function(data){
            console.log('获取历史消息成功', data);
            nextkey = data.data && data.data.next_key;
        },
        error: function(error){
            console.log('获取历史消息失败', error);
        }
     });
```

### 返回的数据结构

![image](https://img.alicdn.com/tps/TB1bncJLpXXXXawXpXXXXXXXXXX-898-1100.png)

### 返回的数据字段解释

|   字段名   | 解释                                     |
| :--------: | :--------------------------------------- |
|    code    | 返回值                                   |
| resultText | 返回值说明                               |
|    data    | 接口返回数据                             |
|  nextKey   | 下一页的标示，如果为空，则标示没有下一页 |
|   touid    | 群id，与调用接口是传入的tid保持一致      |
|    uid     | 自己的用户id                             |
|    msgs    | 消息列表                                 |
|    from    | 消息的发送者                             |
|     to     | 群id                                     |
|    time    | 消息发送时间 **注意单位是毫秒**          |
|    type    | 消息类型                                 |
|    msg     | 消息内容                                 |
|   msgId    | 消息的唯一标示                           |

### 返回的消息类型表

| 类型 | 描述                                                       |
| :--: | :--------------------------------------------------------- |
|  1   | 普通文本消息                                               |
|  3   | 用户加入群                                                 |
|  5   | 用户退出群                                                 |
|  7   | 群主转让                                                   |
|  8   | 取消管理员                                                 |
|  9   | 用户被请出群                                               |
|  12  | 停用群                                                     |
|  14  | 更新群信息                                                 |
|  15  | 通知群名片修改                                             |
|  16  | 图片消息                                                   |
|  35  | 被邀请者拒绝管理员加入群（用户手动拒绝）                   |
|  98  | 管理员拒绝用户加入群                                       |
| 101  | 被邀请者拒绝管理员加入群(用户在客户端有设置，系统自动拒绝) |
| 212  | 申请加入群消息                                             |

## 获取群信息

### Object.Tribe.getTribeInfo(param);

```
param.tid              群号   
[param.excludeFlag]    群的类型(默认：0) 0: 所有类型的群的群信息  1: 普通群的群信息
[param.success]        获取成功回调
[param.error]          获取失败回调
```

### DEMO

```js
    var sdk = new WSDK();

    sdk.Tribe.getTribeInfo({
        tid: 'tid',
        success: function(data){
            console.log('获取群信息成功', data);
        },
        error: function(error){
            console.log('获取群信息失败', error);
        }
     });
```

### 返回的数据结构

![image](https://img.alicdn.com/tps/TB1BOElLpXXXXaPaXXXXXXXXXXX-1176-440.png)

### 返回的数据字段解释

|    字段名    | 解释                                      |
| :----------: | :---------------------------------------- |
|     code     | 返回值                                    |
|  resultText  | 返回值说明                                |
|     data     | 接口返回数据                              |
|   bulletin   | 群公告                                    |
|  checkMode   | 入群的方式                                |
|     icon     | 群头像                                    |
| lastModified | 群头像/公告最后更新的时间**注意单位是秒** |
| memberCount  | 群成员数量                                |
|     name     | 群名称                                    |
|     tid      | 群号                                      |
|  tribeType   | 群类型                                    |

### 群类型表

| 类型 | 描述                                               |
| :--: | :------------------------------------------------- |
|  0   | 普通群                                             |
|  1   | 讨论组，**注意用这个类型，将无法获取到群历史消息** |
|  2   | 企业群                                             |



## 获取群列表

### Object.Tribe.getTribeList(param);

```
param.tribeTypes       群类型，具体见下表格
[param.success]        获取成功回调
[param.error]          获取失败回调
```

### DEMO

```js
    var sdk = new WSDK();

    sdk.Tribe.getTribeInfo({
        tribeTypes: [0,1,2],
        success: function(data){
            console.log('获取群列表成功', data);
        },
        error: function(error){
            console.log('获取群列表失败', error);
        }
     });
```

### 返回的数据结构

![image](https://img.alicdn.com/tps/TB1j8oiLpXXXXctaXXXXXXXXXXX-1008-966.png)

### 返回的数据字段解释

|   字段名   | 解释                                                        |
| :--------: | :---------------------------------------------------------- |
|    code    | 返回值                                                      |
| resultText | 返回值说明                                                  |
|    data    | 接口返回数据                                                |
|   atFlag   | 忽略                                                        |
|  recvFlag  | 忽略                                                        |
|    icon    | 群头像                                                      |
|    role    | 在群中的角色，群主(host)，管理员(manager)，普通成员(normal) |
|    sign    | 群签名                                                      |
|    name    | 群名称                                                      |
|    tid     | 群号                                                        |
| tribeType  | 群类型                                                      |

### 群类型表

| 类型 | 描述                                               |
| :--: | :------------------------------------------------- |
|  0   | 普通群                                             |
|  1   | 讨论组，**注意用这个类型，将无法获取到群历史消息** |
|  2   | 企业群                                             |

## 获取群成员

### Object.Tribe.getTribeMembers(param);

```
param.tid             群号
[param.success]       获取成功回调
[param.error]         获取失败回调
```

### DEMO

```js
    var sdk = new WSDK();

    sdk.Tribe.getTribeMembers({
        tid: 'tid',
        success: function(data){
            console.log('获取群成员成功', data);
        },
        error: function(error){
            console.log('获取群成员失败', error);
        }
     });
```

### 返回的数据结构

![image](https://img.alicdn.com/tps/TB14EwCLpXXXXanXFXXXXXXXXXX-914-520.png)

### 返回的数据字段解释

|   字段名   | 解释                                                        |
| :--------: | :---------------------------------------------------------- |
|    code    | 返回值                                                      |
| resultText | 返回值说明                                                  |
|    data    | 接口返回数据                                                |
|    nick    | 用户昵称                                                    |
|    role    | 在群中的角色，群主(host)，管理员(manager)，普通成员(normal) |
|    uid     | 用户id                                                      |

## 回复邀请加入群的请求

### Object.Tribe.responseInviteIntoTribe(param);

```
param.tid              群号   
param.validatecode     收到消息中的validatecode值 与收到的邀请消息中的validatecode值相同表示同意加入,否则表示拒绝加入
param.recommender      收到消息中的recommender值
param.manager          收到消息中的manager值
[param.success]        获取成功回调
[param.error]          获取失败回调
```

### DEMO

```js
    var sdk = new WSDK();

    sdk.Tribe.responseInviteIntoTribe({
        tid: 'tid',
        recommender: '',
        validatecode: '',
        manager: '',
        success: function(data){
            console.log(data);
        },
        error: function(error){
            console.log(error);
        }
    });
```



## 上传插件

### Object.Plugin.Uploader.upload(param);

```
param.base64Str     图片/语音base64后的字符串
param.ext           图片/语音的格式 如：png, amr
[param.type]        上传的类型,默认1 1：图片 2：语音
[param.maxSize]     最大文件大小
[param.timeout]     超时时间，默认30000ms  
[param.success]     上传成功回调
[param.error]       上传失败回调
```

**maxSize解释：** WSDK内部做了一个最大文件大小(1024 * 1024 * 5K)的限制，如果你传入的maxSize比5M大，则maxSize为5M

**关于接口返回：** 上传成功后，success中返回的data中返回了base64Str字段，用来给你做本地图片/语音的展示，节省流量

### DEMO

```js
    var sdk = new WSDK();

    sdk.Plugin.Uploader.upload({
        base64Str: 'base64',
        ext: 'ext',
        type: 1,
        success: function(data){
            /*
            {
                code: 1000,
                resultCode: 'SUCCESS',
                data: {
                    url: '',
                    base64Str: ''
                }
            }
            */
            console.log('上传成功', data);
        },
        error: function(error){
            console.log('上传失败', error);
        }
    });
```

## 图片上传插件

### Object.Plugin.Image.upload(param);

```
param.target        上传的input[file] dom元素，请传入原生dom
param.base64Img     图片base64后的字符串
param.ext           图片的格式 如：png, jpg
[param.maxSize]     最大文件大小
[param.timeout]     超时时间，默认30000ms  
[param.success]     上传成功回调
[param.error]       上传失败回调
```

**入参：** target 或者 base64Img + ext 参数二选其一必填

**maxSize解释：** WSDK内部做了一个最大文件大小(1024 * 1024 * 5K)的限制，如果你传入的maxSize比5M大，则maxSize为5M

**关于接口返回：** 上传成功后，success中返回的data中返回了base64Str字段，用来给你做本地图片/语音的展示，节省流量

### DEMO

#### 用target参数上传

```html
    <input type="file" id="J_fileInput" />
    var sdk = new WSDK(),
        fileInput = document.getElementById('J_fileInput');

    fileInput.onchange = function(){
        sdk.Plugin.Image.upload({
            target: fileInput,
            success: function(data){
                /*
                {
                    code: 1000,
                    resultCode: 'SUCCESS',
                    data: {
                        url: '',
                        base64Str: ''
                    }
                }
                */
                console.log('上传成功', data);
            },
            error: function(error){
                console.log('上传失败', error);
            }
        });
    }
```

#### 用 base64Img 与 ext 上传

```js
    var sdk = new WSDK();

    sdk.Plugin.Image.upload({
        base64Img: 'base64',
        ext: 'png',
        success: function(data){
            /*
            {
                code: 1000,
                resultCode: 'SUCCESS',
                data: {
                    url: '',
                    base64Str: ''
                }
            }
            */
            console.log('上传成功', data);
        },
        error: function(error){
            console.log('上传失败', error);
        }
    });
```



## 表情插件

**object.Plugin.Emot.emotTitles** [Array]
`默认的表情title列表，共99个`

**object.Plugin.Emot.emotW320** [String]
`默认的表情图片集合(宽度320)`

**object.Plugin.Emot.emotW640** [String]
`默认的表情图片集合(宽度640)`

**object.Plugin.Emot.htmlEncode(str)** [Function]
`将表情文字转成多端通用的表情符号，一般在发送消息之前调用，返回表情通用符号，例如：你好[加油]=>你好/:012`

**object.Plugin.Emot.decode(str)** [Function]
`将通用的表情符号转换成表情图片，一般在收到消息时，展示消息时调用，返回表情图片html, 例如 <img class="wsdk-emot" src="https://g.alicdn.com/aliww/h5-openim/0.0.1/faces/s077.png" alt="惊声尖叫">`

**object.Plugin.Emot.splitByEmot(str)** [Function]
`将消息中的表情分隔开，方便转换 返回数组,例如：你好[加油]=>[‘你好’, ‘[加油]’]`

**object.Plugin.Emot.isEmot(str)** [Function]
`判断传入的str是否为表情 返回true/false`

**object.Plugin.Emot.render(config)** [Function]
`渲染默认表情选择框`

**这个渲染只是最基本的pc端样式，如果要在移动端使用或者要修改交互方式，可以使用上面提供的常量与方法，自己组织代码**

```
config.container: 表情框渲染的容器，原生dom   
[config.emots]: 表情的title文案列表，默认object.Plugin.Emot.emotTitles
[config.emotsImg]: 表情的图片集合地址, 默认object.Plugin.Emot.emotW320    
[config.emotSize]: 单个表情的大小，默认45（px）    
[config.row]: 一次展示的表情行数，默认7   
[config.col]: 一次展示的表情列数，默认3   
[config.trigger]: 切换表情的触发按钮，默认：true
[config.onEmotClick]: 当表情被点击时的回调 
[config.customStyle]: 当默认的样式不满足需求时，将customStyle设置为true，可以去除默认样式，自己在css文件中定义样式
```

### DEMO

#### HTML

```html
<div class="my-msg-content" id="myMsgCon"></div>
<div class="my-soft-input">
    <a href="javascript:;" id="myEmotTrigger">表情</a>
    <div id="myEmotBox" style="display:none"></div>
</div>
<div class="my-msg-input">
    <textarea id="myMsgInput"></textarea>
</div>
```

#### CSS

```css
*{margin:0;padding:0;}
.my-msg-content{
    width: 400px;
    height: 200px;
}
.my-soft-input{
    position: relative;
    width: 400px;
    height: 20px;
}
#myEmotBox{
    position: absolute;
    width: 315px;
    height: 155px;
    border: 1px solid #ccc;
    background: #fff;
    bottom: 30px;
}
.my-msg-input textarea{
    width: 400px;
    height: 100px;
    border: 1px solid #ccc;
}
/* 表情切换触发按钮样式 */
.wsdk-emot-trigger {
   text-align: center;
}
.wsdk-emot-trigger .wsdk-emot-trigger-item {
    display: inline-block;
    width: 8px;
    height: 8px;
    overflow: hidden;
    border-radius: 100%;
    margin-right: 8px;
    *zoom: 1;
    *display: inline;
    background: #ccc;
}
.wsdk-emot-trigger .wsdk-active {
    background: #f00;
}
```

#### JS

```js
var trigger = document.getElementById('myEmotTrigger'),
    box = document.getElementById('myEmotBox'),
    textarea = document.getElementById('myMsgInput'),
    msgCon = document.getElementById('myMsgCon'),
    isEmotInited = false,
    isEmotBoxShow = false,
    sdk = new WSDK(),
    Emot = sdk.Plugin.Emot;

trigger.onclick = function(){

    if(!isEmotInited){
        Emot.render({
            container: box,
            onEmotClick: emotClickHandler
        });
        isEmotInited = true;
    }
    if(isEmotBoxShow){
        box.style.display = 'none';
    }else{
        box.style.display = 'block';
    }

    isEmotBoxShow = !isEmotBoxShow;

};

textarea.onkeydown = function(ev){
    if(ev.keyCode == 13 && this.value){
        ev.preventDefault();
        var encodeMsg = Emot.htmlEncode(this.value);
        var decodeMsg = Emot.decode(encodeMsg);
        console.log('用来发送的消息:', encodeMsg);
        console.log('用来显示的消息:', decodeMsg);
        renderMsg(decodeMsg);
        this.value = '';
    }
};

/**
 * 点击表情时的事件
 */
var emotClickHandler = function(emotTitle){
    textarea.value += emotTitle;
};

/**
 * 渲染消息
 */
var renderMsg = function(msg){
    var div = document.createElement('div');
    div.innerHTML = msg;

    msgCon.appendChild(div);
}
```

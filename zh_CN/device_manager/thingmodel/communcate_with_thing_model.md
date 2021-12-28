## 使用说明
注意：设备与平台的通信格式中各能力需要携带自定模块与标识符，格式为：`模块标识符:能力标识符`，中间为英文冒号。  
例如，在物模型 TSL 中，自定义模块标识符为：`test_model_id`，能力标识符为：`test_ability_id`，  
则通信数据中的格式为：`test_model_id:test_ability_id`

## 设备属性上报

* 设备端请求的topic: `/fabric/sys/${productKey}/${deviceName}/thing/property/post`
* 请求数据格式：

```json
{
  "id": "123456",
  "version": "1.0",
  "sys":{
      "ack":0
  },
  "params": {
    "model_id:Switch": {
      "value": "on",
      "time": 1634841971000
    },
    "model_id:Temperature": {
      "value": 35.6,
      "time": 1634841971000
    }
  },
  "method": "thing.property.post"
}
```  

* 参数说明

| 参数  | 类型  | 说明  |
| :------------ | :------------ | :------------ |
| params  | object  | value 可以是普通的值类型，比如: <br/>`"Switch:"on"` <br/>也可以是结构体，可以附带其他参数，比如时间戳: <br/> `"Switch": { "value": "on",  "time": 1634841971000}` |
| sys  | object  | 扩展功能  |
| ack  | int  | sys扩展功能中的参数，表示是否需要平台返回响应 <br/> - 1: 需要平台返回响应 <br/> - 2: 不需要平台返回响应 |

* 平台端响应的topic: `/fabric/sys/${productKey}/${deviceName}/thing/property/post_reply`
* 响应数据格式：

```json
{
  "id": "123456",
  "code": 0,
  "data": {}
}
```

* 参数说明

| 参数  | 类型  | 说明  |
| :------------ | :------------ | :------------ |
| id  | string  | 响应 payload 中的 id 与请求 payload 中的 id 一致，用于平台匹配命令请求与响应的关系  |


## 属性设置

* 平台端请求的 topic: `/fabric/sys/${productKey}/${deviceName}/thing/property/set`
* 请求数据格式：

```json
{
  "id": "123456",
  "version": "1.0",
  "params": {
    "model_id:Wifi": "close"
  },
  "method": "thing.property.set"
}
```

* 参数说明

<table>
  <tr> <td> 参数 </td>  <td> 类型 </td> <td> 说明 </td> </tr>
   <tr> <td> params </td> <td> object </td> <td> params 可以用键值对的形式传递参数，比如：<br/> 

```
 "params": {
     "Switch": "on", 
     "Wifi": "close"
 } 
```

<br/> 当需要额外传递时间戳时，需使用结构体将所有参数包装起来：<br/> 

```
"params":{
    "value": {
        "Switch": "on",
        "Wifi": "close"
    },
    "time": 1634841971000
}
```

</td> </tr>
</table>

* 设备端响应的 topic: `/fabric/sys/${productKey}/${deviceName}/thing/property/set_reply`
* 响应数据格式：

```json
{
  "id": "123456",
  "code": 0,
  "data": {}
}
```
## 设备事件上报

* 设备端请求的topic: `/fabric/sys/${productKey}/${deviceName}/thing/event/model_id:identifier/post`
* 请求数据格式：

```json
{
  "id": "123456",
  "version": "1.0",
  "sys":{
      "ack":0
  },
  "params": {
    "value": {
      "Switch": "on",
      "Wifi": "open"
    },
    "time": 1634841971000
  },
  "method": "thing.event.post.model_id:identifier"
}
```

* 参数说明

<table>
  <tr> <td> 参数 </td>  <td> 类型 </td> <td> 说明 </td> </tr>
   <tr> <td> params </td> <td> object </td> <td> params 可以用键值对的形式传递参数，比如：<br/> 

```
 "params": {
     "Switch": "on", 
     "Wifi": "close"
 } 
```

<br/> 当需要额外传递时间戳时，需使用结构体将所有参数包装起来：<br/> 

```
"params":{
    "value": {
        "Switch": "on",
        "Wifi": "close"
    },
    "time": 1634841971000
}
```

</td> </tr>
</table>

* 平台端响应的 topic: `/fabric/sys/${productKey}/${deviceName}/thing/event/model_id:identifier/post_reply`
* 响应数据格式：
```json
{
  "id": "123456",
  "code": 0,
  "data": {}
}
```

## 设备方法调用（异步）

* 平台端请求的 topic: `/fabric/sys/${productKey}/${deviceName}/thing/action/model_id:identifier/exec`
* 请求数据格式：

```json
{
  "id": "123456",
  "version": "1.0",
  "params": {
    "Switch": "on",
    "Wifi": "close"
  },
  "method": "thing.action.exec.model_id:identifier"
}
```

* 设备端响应的 topic: `/fabric/sys/${productKey}/${deviceName}/thing/action/model_id:identifier/exec_reply`
* 响应数据格式：

```json
{
  "id": "123456",
  "code": 0,
  "data": {
    "Switch": "on",
    "Wifi": "close"
  }
}
```
## 设备方法调用（同步）

* 平台端请求的 topic: `/fabric/sys/${productKey}/${deviceName}/rrpc/request/${messageId}`
* 请求数据格式：

```json
{
  "id": "123456",
  "version": "1.0",
  "params": {
    "Switch": "on",
    "Wifi": "close"
  },
  "method": "thing.action.exec.[model_id:]identifier"
}
```

* 设备端响应的 topic: `/fabric/sys/${productKey}/${deviceName}/rrpc/response/${messageId}`
* 响应数据格式：
```json
{
  "id": "123456",
  "code": 0,
  "data": {
    "Switch": "on",
    "Wifi": "close"
  }
}
```

## 设备获取期望属性

* 设备端请求的 topic: `/fabric/sys/${productKey}/${deviceName}/thing/property/desired/get`
* 请求数据格式：

```json
{
    "id" : "123456",
    "version":"1.0",
    "params" : [
        "model_id:Switch",
        "model_id:Wifi"
    ],
    "method":"thing.property.desired.get"
}
```

* 平台端响应的 topic: `/fabric/sys/${productKey}/${deviceName}/thing/property/desired/get_reply`
* 响应数据格式：

```json
{
    "id":"123456",
    "code":0,
    "data":{
        "model_id:Switch": {
            "value": "on",
            "version": 1
        },
        "model_id:Wifi": {
            "value": "close",
            "version": 2
        }
    }
}
```

## 设备删除期望属性

* 设备端请求的 topic: `//fabric/sys/${productKey}/${deviceName}/thing/property/desired/delete`
* 请求数据格式：

```json
{
    "id": "123456",
    "version": "1.0",
    "sys":{
      "ack":0
  },
    "params": {
        "model_id:Switch": {
            "version": 1
        },
        "model_id:Wifi": {}
    },
    "method":"thing.property.desired.delete"  
}
```

* 参数说明

<table>
<tr> <td> 参数 </td>  <td> 类型 </td> <td> 说明 </td> </tr>
<tr> <td> params </td> <td> object </td> <td> 清除属性时，可以指定清除特定的版本号，如：<br/> 

```
"params": {
    "model_id:Switch": {
        "version": 1
    }
}
```

<br/> 也可以不指定版本号，直接清除整个属性，如：<br/>

```
"params": {
    "model_id:Wifi": {}
}
```

</td> </tr>
</table>

* 平台端响应的 topic: `/fabric/sys/${productKey}/${deviceName}/thing/property/desired/delete_reply`
* 响应数据格式：

```json
{
    "id": "123456",
    "code": 0,
    "data": {}
}
```
# JavaScript脚本示例

本文将介绍自定义 topic 的脚本模板

## 脚本模板
```javascript
/**
* 入参：topic，字符串，设备上报消息的topic。
* 入参：bytes，byte[]数组，不能为空。
* 出参：jsonObj，对象，不能为空。
*/
function transformPayload(topic, bytes) {
  var jsonObj = {};
  return jsonObj;
}
```

**说明** _以上脚本仅用于解析自定义 topic 的数据。如果产品定义了物模型，
且设备使用自定义数据格式上报物模型数据，则还需编写物模型数据解析函数。_  
物模型数据解析脚本的编写，请参考[物模型数据解析使用示例](../user_defined_thing_model_data_parsing/javascript_use_case.md)

## 脚本示例
```javascript
var SELF_DEFINE_TOPIC_UPDATE_FLAG = '/user/update'  //自定义topic后缀：/user/update。
var SELF_DEFINE_TOPIC_ERROR_FLAG = '/user/update/error' //自定义topic后缀：/user/update/error。
/*
  示例数据：
  自定义topic：
     /user/update，上报数据。
  输入参数：
     topic: /fabric/sys/${productKey}/${deviceName}/user/update
     bytes: 0x00020100c203110213010400
  输出参数：
    {
      "param_int1": 785,
      "param_int2": 256,
      "param_int3": 513
    }
 */
function transformPayload(topic, bytes) {
    var uint8Array = new Uint8Array(bytes.length);
    for (var i = 0; i < bytes.length; i++) {
        uint8Array[i] = bytes[i];
    }
    var dataView = new DataView(uint8Array.buffer, 0);
    var jsonMap = {};

    if(topic.includes(SELF_DEFINE_TOPIC_ERROR_FLAG)) {
        jsonMap['code'] = dataView.getInt8(0)
    } else if (topic.includes(SELF_DEFINE_TOPIC_UPDATE_FLAG)) {
        jsonMap['param_int1'] = dataView.getInt16(5);
        jsonMap['param_int2'] = dataView.getInt16(2);
        jsonMap['param_int3'] = dataView.getInt16(1);
    }
    return jsonMap;
}
```
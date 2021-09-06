# JavaScript脚本示例

本文将介绍自定义 topic 的脚本模板

## 脚本模板
```javascript
/**
* 入参：topic，字符串，设备上报消息的Topic。
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
     topic: /${productKey}/${deviceName}/user/update
     bytes: 0x000000000100320100000000
  输出参数：
  {
     "prop_float": 0,
     "prop_int16": 50,
     "prop_bool": 1
   }
 */
function transformPayload(topic, bytes) {
    var uint8Array = new Uint8Array(bytes.length);
    for (var i = 0; i < bytes.length; i++) {
        uint8Array[i] = bytes[i] & 0xff;
    }
    var dataView = new DataView(uint8Array.buffer, 0);
    var jsonMap = {};

    if(topic.includes(SELF_DEFINE_TOPIC_ERROR_FLAG)) {
        jsonMap['code'] = dataView.getInt8(0)
    } else if (topic.includes(SELF_DEFINE_TOPIC_UPDATE_FLAG)) {
        jsonMap['param_int16'] = dataView.getInt16(5);
        jsonMap['param_bool'] = uint8Array[7];
        jsonMap['param_float'] = dataView.getFloat32(8);
    }
    return jsonMap;
}
```
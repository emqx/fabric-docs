# JavaScript脚本示例

## 脚本模板
```javascript
/**
 *  将平台下发的 JSON 格式的物模型数据转换为设备能识别的自定义格式数据，
 *  比如设置物模型属性、设备方法调用、设备上报物模型数据后平台返回的响应
 *  入参：jsonObj，对象，不能为空。
 *  出参：设备能识别的自定义格式数据，byte[]数组，不能为空。
 */
function thingProtocolToRawData(jsonObj) {
    var payloadArray = [];
    return payloadArray;
}

/**
 * 将设备上报的自定义格式的物模型数据转换为 JSON 格式的数据。
 * 入参：bytes 设备上报的数据，byte[]数组，不能为空。
 * 出参：jsonObj，对象，不能为空。
 */
function rawDataToThingProtocol(bytes) {
    var jsonObj ={};
    return jsonObj;
}
```

## 脚本示例
```javascript
var COMMAND_REPORT = 0x00; //属性上报。
var COMMAND_SET = 0x01; //属性设置。
var COMMAND_REPORT_REPLY = 0x02; //上报数据返回结果。
var COMMAND_SET_REPLY = 0x03; //属性设置设备返回结果。
var COMMAD_UNKOWN = 0xff;    //未知的命令。
var THING_PROP_REPORT_METHOD = 'thing.property.post'; //物联网平台Topic，设备上传属性数据到云端。
var THING_PROP_SET_METHOD = 'thing.property.set'; //物联网平台Topic，云端下发属性控制指令到设备端。
var THING_PROP_SET_REPLY_METHOD = 'thing.property.set'; //物联网平台Topic，设备上报属性设置的结果到云端。
var SELF_DEFINE_TOPIC_UPDATE_FLAG = '/user/update'  //自定义Topic：/user/update。
var SELF_DEFINE_TOPIC_ERROR_FLAG = '/user/update/error' //自定义Topic：/user/update/error。
/*
示例数据：
设备上报属性数据：
传入参数：
    0x000000000100320100000000
输出结果：
    {"method":"thing.property.post","id":"1","params":{"param_float":0,"param_int16":50,"param_bool":1},"version":"1.0"}

属性设置的返回结果：
传入参数：
    0x0300223344c8
输出结果：
    {"code":"200","data":{},"id":"2241348","version":"1.0"}
*/
function rawDataToThingProtocol(bytes) {
    var uint8Array = new Uint8Array(bytes.length);
    for (var i = 0; i < bytes.length; i++) {
        uint8Array[i] = bytes[i] & 0xff;
    }
    var dataView = new DataView(uint8Array.buffer, 0);
    var jsonMap = new Object();
    var fHead = uint8Array[0]; // command
    if (fHead == COMMAND_REPORT) {
        jsonMap['method'] = THING_PROP_REPORT_METHOD; //JSON格式，属性上报topic。
        jsonMap['version'] = '1.0'; //JSON格式，协议版本号固定字段。
        jsonMap['id'] = '' + dataView.getInt32(1); //JSON格式，标识该次请求id值。
        var params = {};
        params['param_int16'] = dataView.getInt16(5); //对应产品属性中param_int16。
        params['param_bool'] = uint8Array[7]; //对应产品属性中param_bool。
        params['param_float'] = dataView.getFloat32(8); //对应产品属性中param_float。
        jsonMap['params'] = params; //JSON格式，params标准字段。
    } else if(fHead == COMMAND_SET_REPLY) {
        jsonMap['version'] = '1.0'; //JSON格式，协议版本号固定字段。
        jsonMap['id'] = '' + dataView.getInt32(1); //JSON格式，标识该次请求id值。
        jsonMap['code'] = ''+ dataView.getUint8(5);
        jsonMap['data'] = {};
    }

    return jsonMap;
}
/*
示例数据：
云端下发属性设置指令：
传入参数：
    {"method":"thing.property.set","id":"12345","version":"1.0","params":{"param_float":123.452, "param_int16":333, "param_bool":1}}
输出结果：
    0x0100003039014d0142f6e76d

设备上报的返回结果：
传入数据：
    {"method":"thing.property.post","id":"12345","version":"1.0","code":200,"data":{}}
输出结果：
    0x0200003039c8
*/
function thingProtocolToRawData(json) {
    var method = json['method'];
    var id = json['id'];
    var version = json['version'];
    var payloadArray = [];
    if (method == THING_PROP_SET_METHOD) //属性设置。
    {
        var params = json['params'];
        var param_float = params['param_float'];
        var param_int16 = params['param_int16'];
        var param_bool = params['param_bool'];
        //按照自定义协议格式拼接 rawData。
        payloadArray = payloadArray.concat(buffer_uint8(COMMAND_SET)); //command字段。
        payloadArray = payloadArray.concat(buffer_int32(parseInt(id))); //JSON格式 'id'。
        payloadArray = payloadArray.concat(buffer_int16(param_int16)); //属性'param_int16'的值。
        payloadArray = payloadArray.concat(buffer_uint8(param_bool)); //属性'param_bool'的值。
        payloadArray = payloadArray.concat(buffer_float32(param_float)); //属性'param_float'的值。
    } else if (method ==  THING_PROP_REPORT_METHOD) { //设备上报数据返回结果。
        var code = json['code'];
        payloadArray = payloadArray.concat(buffer_uint8(COMMAND_REPORT_REPLY)); //command字段。
        payloadArray = payloadArray.concat(buffer_int32(parseInt(id))); //JSON格式'id'。
        payloadArray = payloadArray.concat(buffer_uint8(code));
    } else { //未知命令，对于这些命令不做处理。
        var code = json['code'];
        payloadArray = payloadArray.concat(buffer_uint8(COMMAD_UNKOWN)); //command字段。
        payloadArray = payloadArray.concat(buffer_int32(parseInt(id))); //JSON格式'id'。
        payloadArray = payloadArray.concat(buffer_uint8(code));
    }
    return payloadArray;
}

/*
  示例数据
  自定义Topic：
     /user/update，上报数据。
  输入参数：
     topic:/${productKey}/${deviceName}/user/update
     bytes: 0x000000000100320100000000
  输出参数：
  {
     "param_float": 0,
     "param_int16": 50,
     "param_bool": 1,
     "topic": "/${productKey}/${deviceName}/user/update"
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
        jsonMap['topic'] = topic;
        jsonMap['errorCode'] = dataView.getInt8(0)
    } else if (topic.includes(SELF_DEFINE_TOPIC_UPDATE_FLAG)) {
        jsonMap['topic'] = topic;
        jsonMap['param_int16'] = dataView.getInt16(5);
        jsonMap['param_bool'] = uint8Array[7];
        jsonMap['param_float'] = dataView.getFloat32(8);
    }

    return jsonMap;
}

//以下是部分辅助函数。
function buffer_uint8(value) {
    var uint8Array = new Uint8Array(1);
    var dv = new DataView(uint8Array.buffer, 0);
    dv.setUint8(0, value);
    return [].slice.call(uint8Array);
}
function buffer_int16(value) {
    var uint8Array = new Uint8Array(2);
    var dv = new DataView(uint8Array.buffer, 0);
    dv.setInt16(0, value);
    return [].slice.call(uint8Array);
}
function buffer_int32(value) {
    var uint8Array = new Uint8Array(4);
    var dv = new DataView(uint8Array.buffer, 0);
    dv.setInt32(0, value);
    return [].slice.call(uint8Array);
}
function buffer_float32(value) {
    var uint8Array = new Uint8Array(4);
    var dv = new DataView(uint8Array.buffer, 0);
    dv.setFloat32(0, value);
    return [].slice.call(uint8Array);
}
```
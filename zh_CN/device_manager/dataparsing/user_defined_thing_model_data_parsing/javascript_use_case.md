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
var COMMAND_REPORT = 0x00; //表示当前消息为设备上报属性。
var COMMAND_SET = 0x01; //表示当前消息为平台下发属性设置。
var COMMAND_REPORT_REPLY = 0x02; //表示当前消息为设备上报属性后，平台返回的响应。
var COMMAND_SET_REPLY = 0x03; //表示当前消息为属性设置后设备的响应。
var THING_PROP_REPORT_METHOD = 'thing.property.post'; //设备上传属性数据到平台时，需要在payload中添加的method字段。
var THING_PROP_SET_METHOD = 'thing.property.set'; //平台下发属性控制命令到设备端时，在payload中添加的method字段。
var THING_PROP_SET_REPLY_METHOD = 'thing.property.set'; //设备响应属性设置的结果到平台，需要在payload中添加的method字段。
var SELF_DEFINE_TOPIC_UPDATE_FLAG = '/user/update'  //自定义topic后缀：/user/update。
var SELF_DEFINE_TOPIC_ERROR_FLAG = '/user/update/error' //自定义topic后缀：/user/update/error。
/*
示例：
设备上报属性：
传入参数：
    0x0010100c0200410600080000
输出结果：
    {
      "id": "269487106",
      "method": "thing.property.post",
      "params": {
        "param_int1": 65,
        "param_int2": 4108,
        "param_int3": 4112
      },
      "version": "1.0"
    }

属性设置后，设备返回的响应：
传入参数：
    0x0000117a18b5184781732a
输出结果：
    {
      "id": "1145368",
      "method": "thing.property.post",
      "params": {
        "param_int1": -19176,
        "param_int2": 4474,
        "param_int3": 17
      },
      "version": "1.0"
    }
*/
function rawDataToThingProtocol(bytes) {
    var uint8Array = new Uint8Array(bytes.length);
    for (var i = 0; i < bytes.length; i++) {
        uint8Array[i] = bytes[i];
    }
    var dataView = new DataView(uint8Array.buffer, 0);
    var jsonMap = new Object();
    var fHead = uint8Array[0]; // command
    if (fHead == COMMAND_REPORT) {
        jsonMap['method'] = THING_PROP_REPORT_METHOD; //属性上报的method。
        jsonMap['version'] = '1.0'; //协议版本号固定字段。
        jsonMap['id'] = '' + dataView.getInt32(1); //标识该次请求id值。
        var params = {};
        params['param_int1'] = dataView.getInt16(5); //对应物模型属性中param_int1。
        params['param_int2'] = dataView.getInt16(2); //对应物模型属性中param_int2。
        params['param_int3'] = dataView.getInt16(1); //对应物模型属性中param_int3。
        jsonMap['params'] = params; //params标准字段。
    } else if(fHead == COMMAND_SET_REPLY) {
        jsonMap['version'] = '1.0'; //协议版本号固定字段。
        jsonMap['id'] = '' + dataView.getInt32(1); //标识该次请求id值
        jsonMap['code'] = ''+ dataView.getUint8(5);
        jsonMap['data'] = {};
    }

    return jsonMap;
}
/*
示例数据：
平台下发属性设置命令：
传入参数：
    {"method":"thing.property.set","id":"123456","version":"1.0","params":{"param_int1":123, "param_int2":456, "param_int3":789}}
输出结果（十六进制表示）：
    010001e240007b01c80315

设备上报的返回结果：
传入数据：
    {"method":"thing.property.post","id":"123456","version":"1.0","code":0,"data":{}}
输出结果（十六进制表示）：
    0x020001e24000
*/
function thingProtocolToRawData(json) {
    var method = json['method'];
    var id = json['id'];
    var version = json['version'];
    var rawData = [];
    if (method == THING_PROP_SET_METHOD) //属性设置。
    {
        var params = json['params'];
        var param_int1 = params['param_int1'];
        var param_int2 = params['param_int2'];
        var param_int3 = params['param_int3'];
        //按照自定义协议格式拼接 rawData。
        rawData = rawData.concat(buffer_uint8(COMMAND_SET)); //command字段
        rawData = rawData.concat(buffer_int32(parseInt(id))); //标识该次请求id值
        rawData = rawData.concat(buffer_int16(param_int1)); //属性'param_int1'的值
        rawData = rawData.concat(buffer_int16(param_int2)); //属性'param_int2'的值
        rawData = rawData.concat(buffer_int16(param_int3)); //属性'param_int3'的值
    } else if (method ==  THING_PROP_REPORT_METHOD) { //设备上报数据返回结果。
        var code = json['code'];
        rawData = rawData.concat(buffer_uint8(COMMAND_REPORT_REPLY)); //command字段
        rawData = rawData.concat(buffer_int32(parseInt(id))); //标识该次请求id值
        rawData = rawData.concat(buffer_uint8(code));
    }
    return rawData;
}

/*
  示例数据
  自定义topic：
     /user/update，上报数据。
  输入参数：
     topic:/fabric/sys/${productKey}/${deviceName}/user/update
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
    var json = {};

    if(topic.includes(SELF_DEFINE_TOPIC_ERROR_FLAG)) {
        json['code'] = dataView.getInt8(0)
    } else if (topic.includes(SELF_DEFINE_TOPIC_UPDATE_FLAG)) {
        json['param_int1'] = dataView.getInt16(5);
        json['param_int2'] = dataView.getInt16(2);
        json['param_int3'] = dataView.getInt16(1);
    }
    return json;
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
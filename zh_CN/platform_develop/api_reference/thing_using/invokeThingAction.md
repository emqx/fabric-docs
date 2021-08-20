# invokeThingAction

调用该接口在单个设备上调用指定方法。

## 使用说明
定义产品物模型方法时，已设置方法调用方式。因此，调用本接口时，平台会根据方法的定义选择对应的调用方式。
同步方式：平台直接使用 RRPC 同步方式下行发送请求，设备返回 RRPC 响应消息。RRPC使用详情，请参见`[什么是RRPC](待补充)`。
异步方式：平台采用异步方式下行发送请求，设备采用异步方式返回结果。订阅的Topic详情，请参见[物模型通信](../../device_manager/thingmodel/communcate_with_thing_model.md)。

## 限制说明
同步调用方法，最大超时时间为5秒。若5秒内平台未收到回复，则返回超时错误。异步调用方法没有时间限制。

## 请求参数

<table>
<tr> <td>参数</td> <td>类型</td> <td>是否是必选</td> <td>示例值</td> <td>说明</td> </tr>
<tr>
    <td>product_key</td>
    <td>string</td>
    <td>是</td>
    <td>ANOWAGVC**</td>
    <td>设备所属的产品ProductKey</td>
</tr>
<tr>
    <td>device_name</td>
    <td>string</td>
    <td>是</td>
    <td>AP0DNAP**</td>
    <td>设备名称</td>
</tr>
<tr>
    <td>model_id</td>
    <td>string</td>
    <td>否</td>
    <td>test_model</td>
    <td>方法所属的模块ID </br>
    <span style="border-radius: 50%; height: 20px; width: 20px; display: inline-block; background: #238ff9; vertical-align: center;">
       <span style="display: block; color: #FFFFFF; height: 20px; line-height: 20px; text-align: center">?</span>
    </span>
    <b>说明</b> 自定义模块才需要填写该参数（当前不支持默认模块）  
    </td>
</tr>
<tr>
    <td>identifier</td>
    <td>string</td>
    <td>是</td>
    <td>Switch</td>
    <td>方法标识符</td>
</tr>
<tr>
    <td>params_json</td>
    <td>string</td>
    <td>是</td>
    <td>{\"Fan\":"ON"}</td>
    <td>要启用服务的入参信息，数据格式为 JSON String，例如 params_json={"param1":1}。
        若此参数为空时，需传入 params_json={}
    </td>
</tr>
</table>

## 返回数据

<table>
<tr> <td>参数</td> <td>类型</td> <td>说明</td> </tr>
<tr>
    <td>code</td>
    <td>int</td>
    <td>错误码，0表示成功</td>
</tr>
<tr>
    <td>message</td>
    <td>string</td>
    <td>调用失败时的错误描述</td>
</tr>
<tr>
    <td>data</td>
    <td>struct</td>
    <td>调用成功时返回的数据</td>
</tr>
</table>

## 示例

请求示例
```
curl host:8082/device-thing-service/v0/thingModel/invokeThingAction -X POST \
-d '{"product_key":"KS9ZBEOR","device_name":"demo","model_id":"thing_model","identifier":"Switch","params_json":"{\"Fan\":"ON"}"}' \
-H "Content-Type: application/json" \
-H "Cookie: FABRIC_TOKEN=fabric.1.4b84b15bff6ee5796152495a230e45e3d7e947d9.d5527e81265b78e47b7891e3ad18d4cdddd3fda2.1629086990.c24c2f07" 
```

返回数据
```json
{
    "code": 0,
    "message": "",
    "data": {}
}
```

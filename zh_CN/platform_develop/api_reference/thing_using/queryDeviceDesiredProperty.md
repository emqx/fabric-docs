# queryDeviceDesiredProperty
调用该接口查询指定设备的期望属性值。

## 限制说明
* 只读属性不支持设置期望属性值。
* 一次调用最多可设置10个期望属性值。
* 仅支持查询读写模式的属性，如果您指定了只读模式的属性，该属性将被忽略。

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
    <td>identifiers</td>
    <td>array</td>
    <td>是</td>
    <td>["test:Level"]</td>
    <td>要查询期望值的属性的标识符（identifier）列表。最多可输入10个期望属性值。
        <li>单次调用，最多能传入10个标识符</li>
        <li>不可输入重复的属性标识符</li>
        <li>若不传入此参数，将返回该设备所有属性的期望值</li>
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
curl host:8082/device-thing-service/v0/thingModel/setDeviceDesiredProperty -X POST \
-d '{"product_key": "KS9ZBEOR","device_name": "tddemoemost","properties_json": "{\"test:Level\":\"HIGH\"}","versions_json": "{\"test:Level\":0}"}' \
-H "Content-Type: application/json" \
-H "Cookie: FABRIC_TOKEN=fabric.1.4b84b15bff6ee5796152495a230e45e3d7e947d9.d5527e81265b78e47b7891e3ad18d4cdddd3fda2.1629086990.c24c2f07" 
```

返回数据
```json
{
    "code": 0,
    "message": "",
    "data": {
      "desired_property_datas":[
        {
          "identifier": "test:Level",
          "version": 1,
          "value": "{\"HIGH\"}",
          "data_type": "text",
          "time": 1629100828173
        }
      ]
    }
}
```
<div>
<b>desired_property_datas</b> 数组类型，返回指定属性的期望值
</div>

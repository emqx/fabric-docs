# setThingProperty

调用该接口为指定设备设置属性值。  

## 返回结果说明
因为平台下发属性设置命令和设备收到并执行该命令为异步机制，所有调用该接口时，返回的成功结果只
标识平台下发属性设置的请求成功，不能保证设备端收到并执行难了该请求。  

## 请求参数

<table>
<tr> <td>参数</td> <td>类型</td> <td>示例值</td> <td>说明</td> </tr>
<tr>
    <td>product_key</td>
    <td>string</td>
    <td>ANOWAGVC**</td>
    <td>设备所属的产品ProductKey</td>
</tr>
<tr>
    <td>device_name</td>
    <td>string</td>
    <td>AP0DNAP**</td>
    <td>设备名称</td>
</tr>
<tr>
    <td>params_json</td>
    <td>string</td>
    <td>{"Switch":"ON","WIFI":"CLOSE"}</td>
    <td> 
        <div style="line-height:30px;">
            要设置的属性信息，数据格式为JSON。每个属性信息由标识符与属性值（key:value）构成，多个属性用英文逗号隔开。
            例如，设置智能灯的如下两个属性：</br>
            <li>标识符为Switch的开关属性，数据类型为string，设置值为"ON"（开）</li>
            <li>标识符为WIFI的无线开关属性，数据类型为String，设置值为"CLOSE"（关）</li>
            那么要设置的属性信息，JSON格式如下：</br>
            params_json={"Switch":"ON","WIFI":"CLOSE"}</br>
            <span style="border-radius: 50%; height: 20px; width: 20px; display: inline-block; background: #238ff9; vertical-align: center;">
                <span style="display: block; color: #FFFFFF; height: 20px; line-height: 20px; text-align: center">?</span>
            </span>
            **说明** 如果设置自定义模块testFb（非默认模块）的属性，则格式为： Items={"testFb:Switch":1,"testFb:Color":"blue"}
        </div>
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
curl host:8082/device-thing-service/v0/thingModel/setThingProperty -X POST \
-d '{"testFb:Switch":1,"testFb:Color":"blue"}' \
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
# setDeviceDesiredProperty
调用该接口为指定设备批量设置期望属性值。

## 限制说明
* 只读属性不支持设置期望属性值。
* 一次调用最多可设置10个期望属性值。
* 设备创建后，期望属性值的版本（version）为0。首次设置期望属性值时，如果要指定 version 参数，则必须指定 version 值为0。

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
    <td>properties_json</td>
    <td>string</td>
    <td>是</td>
    <td>{\"Level\":"HIGH"}</td>
    <td>要设置的期望属性值，组成为属性的Key:Value，数据格式为JSON String，例如{"Level":"HIGH"}。最多可输入10个期望属性值。
        <li>Key取值为属性的标识符（identifier）。如果是自定义模块test下属性Level，则参数值为 {"test:Level":"HIGH"}。</li>
        <span style="border-radius: 50%; height: 20px; width: 20px; display: inline-block; background: #238ff9; vertical-align: center;">
           <span style="display: block; color: #FFFFFF; height: 20px; line-height: 20px; text-align: center">?</span>
        </span>
        <b>说明</b> 指定属性必须是读写模式。如果您指定了一个只读模式的属性，设置将会失败。并且，一次调用中，不能传入重复的属性标识符。
        <li>Value取值为要设置的期望属性值。</li>
        <span style="border-radius: 50%; height: 20px; width: 20px; display: inline-block; background: #238ff9; vertical-align: center;">
           <span style="display: block; color: #FFFFFF; height: 20px; line-height: 20px; text-align: center">?</span>
        </span>
        <b>说明</b> 若属性值设置为null，则表示清空期望属性值。
    </td>
</tr>
<tr>
    <td>versions_json</td>
    <td>string</td>
    <td>是</td>
    <td>{\"Level\":0}</td>
    <td>当前期望属性值版本，组成为Key:Value，数据格式为JSON String，例如{"Temperature":2}。
        <li>Key取值为属性的标识符（identifier）。</li>
        <span style="border-radius: 50%; height: 20px; width: 20px; display: inline-block; background: #238ff9; vertical-align: center;">
           <span style="display: block; color: #FFFFFF; height: 20px; line-height: 20px; text-align: center">?</span>
        </span>
        <b>说明</b> 一次调用中，key的取值（即属性的identifier）不能重复。
        <li>Value取值为当前期望属性值的版本号。首次设置期望属性值时，指定该参数值为0。
            首次设置期望属性值后，期望值版本号为1。以后每次设置期望值后，
            平台自动将期望值版本加1（即第二次设置期望属性值时，指定该参数值为1。
            设置成功后，版本号自动变为2；第三次设置时，指定该参数值为2。设置成功后，
            版本号自动变为3；以此类推）。</li>
        <span style="border-radius: 50%; height: 20px; width: 20px; display: inline-block; background: #238ff9; vertical-align: center;">
           <span style="display: block; color: #FFFFFF; height: 20px; line-height: 20px; text-align: center">?</span>
        </span>
        <b>说明</b> 如果传入的版本号与当前版本不符，平台将拒绝此次请求。若您不确定当前期望值的版本号，可以不传入版本号，但仍需传入有效的JSON，即传入{}。
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
      "versions":"{\"test:Level\":1}"
    }
}
```
<div>
<span style="border-radius: 50%; height: 20px; width: 20px; display: inline-block; background: #238ff9; vertical-align: center;">
  <span style="display: block; color: #FFFFFF; height: 20px; line-height: 20px; text-align: center">?</span>
</span>
**versions** 为设置成功后期望属性的最新版本号
</div>

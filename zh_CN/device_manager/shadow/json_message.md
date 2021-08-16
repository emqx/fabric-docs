## 设备影子 JSON 格式解析

设备影子 JSON 格式示例：
```json
{
    "state": {
        "desired": {
            "power": "ON", 
            "light": [
                "RED", 
                "50", 
                "0xfa349g"
            ]
        }, 
        "reported": {
            "power": "CLOSE"
        }
    }, 
    "metadata": {
        "desired": {
            "color": {
                "timestamp": 1629082191
            }, 
            "sequence": {
                "timestamp": 1629082191
            }
        }, 
        "reported": {
            "color": {
                "timestamp": 1629082191
            }
        }
    }, 
    "timestamp": 1629082191, 
    "version": 1
}
```

| 参数 | 说明 |
| :--------- | :--------- |
| desired | 设备的预期状态。</br> 应用程序设置设备的 desired 状态，该状态将保存在设备影子中。|
| repoted | 设备最近一次上报的状态。</br> 应用程序可以通过读取该参数值，获取设备的状态 |
| metadata | 当应用程序更新 desired 或者设备上报状态后，设备影子服务会自动更新 metadata 的值。</br> 设备状态的元数据的信息包含间每个状态的时间戳 |
| timestamp | 设备影子的最新更新时间 |
| version | 应用程序更新 desired 时，设备影子会检查请求中的 version 值是否大于当前版本号。</br> 如果大于当前版本号，则更新设备影子，反之则会拒绝更新设备影子。</br> version参数为 int64 型。为防止参数溢出，您可以手动传入-1将版本号重置为0 |

_注：_ 设备状态支持更新数组类型的值，但更新数组类型的值，必须全量更新，不能只更新其中一部分。设备影子对同一状态的更新策略为全量覆盖  
例如：  
* 初始状态
```json

{
"reported" : { "temps" : ["23.5", "24.1", "22.9" ] }
}
```

* 更新
```json

{
"reported" : { "temps" : ["24.2", "23.9" ] }
}
```

* 最终状态
```json

{
"reported" : { "temps" : ["24.2", "23.9" ] }
}
```
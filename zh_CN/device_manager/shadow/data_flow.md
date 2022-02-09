# 设备影子通信协议
设备影子数据通过 topic 进行流转，主要包括：设备上报状态到设备影子，应用程序更改设备状态，
设备离线再上线后主动获取设备影子信息，和设备端请求删除设备影子中的属性信息

## 设备影子 topic
* `/fabric/shadow/update/${productKey}/${deviceName}`  
设备和用户端发布消息到此 topic，shadow 收到该 topic 的消息后，将消息中的状态更新到影子信息中。 

* `/fabric/shadow/get/${productKey}/${deviceName}`  
设备影子更新状态到该 topic，设备订阅此 topic 获取最新消息。

## 设备主动上报状态
1. 设备上线后，使用 topic `/fabric/shadow/update/${productKey}/${deviceName}` 上报最新状态到影子  
发送的 JSON 消息格式：
```json
{
  "method": "update", 
  "state": {
    "reported": {
      "color": "red"
    }
  }, 
  "version": 1
}
```

2.设备影子接收到设备上报的状态数据后，更新影子信息 
```json
{
  "state": {
    "reported": {
      "color": "red"
    }
  }, 
  "metadata": {
    "reported": {
      "color": {
        "timestamp": 1629086285
      }
    }
  }, 
  "timestamp": 1629086285, 
  "version": 1
}
```

3. 影子信息更新后，设备影子会返回结果给设备，即发送消息到设备订阅的 topic `/fabric/shadow/get/${productKey}/${deviceName}`
    * 若更新成功，发送到该 topic 中的消息为：
      ```json
      {
         "method": "reply", 
         "payload": {
            "status": "success", 
            "version": 1
          }, 
         "timestamp": 1469564576
      }
      ```
    * 若更新失败，发送到该 topic 中的消息为：
      ```json
      {
        "method": "reply",
        "payload": {
           "status": "error",
           "content": {
              "errorcode": "${errorcode}",
              "errormessage": "${errormessage}"
           }
        },
        "timestamp": 1469564576
      }
      ```
      
## 应用程序设置期望状态
应用程序通过调用云端API UpdateDeviceShadow 下发期望状态给设备影子，
设备影子再将影子信息下发给设备端。设备根据影子更新状态，并上报最新状态至影子  

1. 应用程序调用云端API UpdateDeviceShadow，下发消息更改设备状态
```json
{
  "method": "update", 
  "state": {
    "desired": {
      "color": "green"
    }
  }, 
  "version": 2
}
```

2. 设备影子接收到更新请求，更新其影子信息为：

```json
{
  "state": {
    "reported": {
      "color": "red"
    }, 
    "desired": {
      "color": "green"
    }
  }, 
  "metadata": {
    "reported": {
      "color": {
        "timestamp": 1629086285
      }
    }, 
    "desired": {
      "color": {
        "timestamp": 1469564576
      }
    }
  }, 
  "timestamp": 1469564576, 
  "version": 2
}
```

3. 设备影子更新完成后，将影子信息发送到 topic `/fabric/shadow/get/${productKey}/${deviceName}`。

```json
{
  "method": "control", 
  "payload": {
    "state": {
      "reported": {
        "color": "red"
      }, 
      "desired": {
        "color": "green"
      }
    }, 
    "metadata": {
      "reported": {
        "color": {
          "timestamp": 1629086285
        }
      }, 
      "desired": {
        "color": {
          "timestamp": 1469564576
        }
      }
    }
  }, 
  "version": 2, 
  "timestamp": 1469564576
}
```

4. 如果设备在线，并且订阅了 topic `/fabric/shadow/get/${productKey}/${deviceName}`，则会立即收到消息。  
设备收到消息后，根据影子信息中的 desired 的值更新状态。  
设备更新完状态后，上报最新状态到设备影子。 

```json
{
  "method": "update", 
  "state": {
    "reported": {
      "color": "green"
    }
  }, 
  "version": 3
}
```

5. 最新状态上报成功后，设备端和设备影子进行以下操作
    * 设备端发消息到 topic `/fabric/shadow/update/${productKey}/${deviceName}` 中清空 desired 属性。消息如下
    ```json
    {
      "method": "update",
      "state": {
      "desired": "null"
      },
      "version": 4
    }
    ```
    * 设备影子会同步更新影子文档，此时的影子文档如下
    ```json
    {
      "state": {
        "reported": {
        "color": "green"
        }
      },
      "metadata": {
        "reported": {
          "color": {
          "timestamp": 1469564577
          }
        },
        "desired": {
          "timestamp": 1469564576
        }
      },
      "version": 4
    }
    ```

## 设备主动获取期望状态
若应用程序发送指令时，设备离线。设备再次上线后，将主动获取设备影子内容  

1. 设备主动发送以下消息到 topic `/fabric/shadow/update/${productKey}/${deviceName}`中，请求获取设备影子中保存的最新状态
```json
{
  "method": "get"
}
```

2. 当设备影子收到这条消息后，发送最新状态到 topic `/fabric/shadow/get/${productKey}/${deviceName}`。  
设备通过订阅该 topic 获取最新状态。消息内容如下：
```json
{
  "method": "reply", 
  "payload": {
    "status": "success", 
    "state": {
      "reported": {
        "color": "red"
      }, 
      "desired": {
        "color": "green"
      }
    }, 
    "metadata": {
      "reported": {
        "color": {
          "timestamp": 1629086285
        }
      }, 
      "desired": {
        "color": {
          "timestamp": 1629086285
        }
      }
    }
  }, 
  "version": 2, 
  "timestamp": 1469564576
}
```

## 设备主动删除影子状态
若设备端已经是最新状态，设备端可以主动发送指令，删除设备影子中保存的某条属性状态  
设备发送以下内容到 topic `/fabric/shadow/update/${productKey}/${deviceName}`中。
其中，method 为 delete，属性的值为 null。  
* 删除影子中某一属性。
```json
{
  "method": "delete", 
  "state": {
    "reported": {
      "color": "null", 
      "temperature": "null"
    }
  }, 
  "version": 1
}
```
* 删除影子全部属性。
```json
{
  "method": "delete", 
  "state": {
    "reported": "null"
  }, 
  "version": 1
}
```
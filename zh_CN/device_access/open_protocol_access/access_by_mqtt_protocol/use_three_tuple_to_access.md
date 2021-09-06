# 使用设备三元组信息连接到 MQTT
本文主要介绍如何使用设备三元组通过 TCP 连接到 MQTT

## 设备三元组
| 组信息 | 描述  |
| ------------ | ------------ |
| product_key  | 产品 key  |
| device_name  | 设备名  |
| secret_key  | 设备密钥  |

## 三元组与 mqtt 连接参数的对应关系
| 参数 | 说明  |
| ------------ | ------------ |
| user_name  | 由 product_key 和 device_name 通过连接符”-”拼接而成:<p>`$product_key-$device_name`<p> 例如 product_key 为 `KQ0AWGR4`，device_name 为 `QCal29csf9`，则 user_name 为 `KQ0AWGR4-QCal29csf9`   |
| password  | 使用 secret_key 即可 |
| client_id  | user_name 的 md5 值，可在 linux 下使用命令转换: <code>echo -n $user_name &#124; md5sum</code>  |

## 使用 MQTTX 客户端接入 MQTT
1. [下载 MQTTX v1.6.0 客户端](https://packages.emqx.net/MQTTX/v1.6.0/MQTTX.Setup.1.6.0.exe)，完成安装。  
  更多的 MQTTX 信息，请访问[MQTTX 官网](https://mqttx.app/zh)

2. 连接 MQTT 请直接参考 [MQTTX官方文档](https://mqttx.app/zh/docs)
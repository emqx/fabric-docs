# 什么是 ssh/http 通道
平台提供的中继服务，用户可通过该中继服务从外部网络直接访问局域网内的设备的 ssh/http 服务。

## 使用说明
使用 ssh/http 通道需要在设备端安装我们的 Agent 客户端，由 Agent 负责与平台建立 tcp 通道

## 获取edge-agent
[edge-agent-bin-x86_64.tar.gz](https://static.emqx.net/fabric/edge-agent-0.6.0/edge-agent-bin-x86_64.tar.gz)

[edge-agent-bin-aarch64.tar.gz](https://static.emqx.net/fabric/edge-agent-0.6.0/edge-agent-bin-aarch64.tar.gz)

[edge-agent-bin-armv7.tar.gz](https://static.emqx.net/fabric/edge-agent-0.6.0/edge-agent-bin-armv7.tar.gz)

## 前置要求
首先用户需要[创建产品](../quick_start/create_product.md)和[创建设备](../quick_start/create_device)。

## 参数配置
将设备的三元组信息，更新到 edge-agent 的配置文件 config.yaml。
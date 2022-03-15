# emqx-platform-sdk 安装教程

## 项目地址

[emqx-platform-sdk](https://static.emqx.net/fabric/sdk/emqx-platform-sdk-0.7.2.tar.gz)

## 项目依赖

emqx-platform-sdk 需要以下依赖，编译项目前确保以下依赖已安装。

- libopenssl
- [json-c](https://static.emqx.net/fabric/sdk/dependency/json-c.tar.gz)
- [phao.mqtt.c](https://static.emqx.net/fabric/sdk/dependency/paho.tar.gz) 

## 依赖安装

- openssl

  ```shell
  $ sudo apt-get install libssl-dev
  ```

- json-c

  ```shell
  $ git clone https://github.com/json-c/json-c.git
  # 如果 github 连接不上，从文件服务器下载
  $ wget https://static.emqx.net/fabric/sdk/dependency/json-c.tar.gz
  $ mkdir json-c-build
  $ cd json-c-build
  $ cmake ../json-c   # See CMake section below for custom arguments
  $ make
  $ sudo make install
  ```

- paho.mqtt.c

  ```shell
  $ git clone https://github.com/eclipse/paho.mqtt.c.git
  $ # 如果 github 连接不上，从文件服务器下载
  $ wget https://static.emqx.net/fabric/sdk/dependency/paho.tar.gz
  $ cd paho.mqtt.c
  $ mkdir build
  $ cd build
  $ cmake ..
  $ make
  $ sudo make install
  ```

## emqx-platform-sdk 编译安装

```shell
$ wget https://static.emqx.net/fabric/sdk/emqx-platform-sdk-0.7.1.tar.gz
$ cd emqx-platform-sdk
$ mkdir build
$ cd build
$ cmake ..
$ make
$ sudo make install
```

## examples

项目的 examples 目录里有两个简单的 demo，一个是关于物模型的， 一个是设备影子的。生成的文件在 build 的 bin 目录下，此处可以参照 [设备端数据上报](../quick_start/device_data_upload.md) 更改 demo 的参数。

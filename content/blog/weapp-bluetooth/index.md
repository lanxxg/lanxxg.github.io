---
title: 微信小程序蓝牙开发总结
date: "2020-05-09"
description: Bluetooth Low Energy.
---

> 在做了小程序蓝牙连接项目之后，总结一下微信小程序蓝牙连接的流程和一些个人心得。

## 一些基本理论
1. 蓝牙：蓝牙是用于数字设备的短距离无线通信技术标准。从最初1.0版本，到现在已经是5.0版本了。简单的理解，前3个版本（1.0-3.0）包括基础速率Bluetooth Basic Rate（BR）、增强数据速率Enhanced Data Rate（EDR）、高速率High Speed（HS），蓝牙传输数据的速度越来越快了。在4.0版本，引入了低功耗蓝牙Bluetooth Low Energy（BLE）协议，把蓝牙分成低功耗蓝牙（BLE）、传统蓝牙（Classic Bluetooth）、高速蓝牙（Bluetooth high speed）三种模式。5.0版本的新功能主要集中在物联网技术上。

    | 蓝牙版本 | 发布时间 | 最大传输速度 | 传输距离 |
    | :-----   | :-----   | :-----   | :-----   |
    | 蓝牙5.1  | 2019 | 48 Mbit/s | 300m |
    | 蓝牙5.0  | 2016 | 48 Mbit/s | 300m |
    | 蓝牙4.2  | 2014 | 24 Mbit/s | 50m |
    | 蓝牙4.1  | 2013 | 24 Mbit/s | 50m |
    | 蓝牙4.0  | 2010 | 24 Mbit/s | 50m |
    | 蓝牙3.0+HS   | 2009 | 24 Mbit/s | 10m |
    | 蓝牙2.1+EDR  | 2007 | 3 Mbit/s | 10m |
    | 蓝牙2.0+EDR  | 2004 | 2.1 Mbit/s | 10m |
    | 蓝牙1.2  | 2003 | 1 Mbit/s | 10m |
    | 蓝牙1.1  | 2002 | 810 Kbit/s | 10m |
    | 蓝牙1.0  | 1999 | 723.1 Kbit/s | 10m |

2. 低功耗蓝牙（Bluetooth Low Energy）：低功耗蓝牙是蓝牙4.0版之后才支持的协议，主要优点有低功耗，低成本等。低功耗蓝牙与经典蓝牙协议不兼容，因此低功耗蓝牙设备不能和经典蓝牙设备连接。
3. GAP：Generic Access Profile (GAP) 协议定义了设备角色，主要有外围设备（Peripheral）和中心设备（Central），并控制设备连接和广播数据。外围设备通过不停地向外广播数据，让中心设备发现自己，中心设备扫描发现有外围设备存在后，可以与之建立连接，之后就可以使用外围设备提供的服务（Services）。
4. GATT：[Generic Attribute Profile (GATT)](https://www.bluetooth.com/specifications/gatt/) 协议定义了两个低功耗蓝牙设备使用服务（Services）和特征（Characteristics）的概念来回传输数据的方式。该协议建立在Attribute Protocol（ATT）协议之上。与 GAP 角色有个相似的概念，GATT 分服务端 Server 和客户端 Client。
5. Services：[Services](https://www.bluetooth.com/specifications/gatt/services/) 可以理解为蓝牙设备提供的服务，一个设备可以提供多个服务，每个服务都有一个唯一的UUID，以便可以与其它服务区分开来。一个服务又包含多个 Characteristic 特性值。
6. Characteristics：[Characteristics](https://www.bluetooth.com/specifications/gatt/characteristics/) 是 GATT 规范中最小的逻辑数据单元，每个特征总体上包括以下内容：

    1. Declaration：特征声明是特征的重要组成部分，因为它包含特征的UUID和Properties。Properties重点有以下几种
        - Read：表示允许客户端读取此特征值
        - Write：表示允许客户端写此特征值
        - Notify：每当服务端特征值变化时，服务端将异步通知客户端
        - Indicate：与Notify类似，区别是它需要客户端确认?（我不确定）
    2. Value: 特征值
    3. Descriptor：每个特征可以有一个或多个特征描述，包含有关特征值的额外信息。

7. UUID：Service 和 Characteristic 都通过一个 16bit 或 128bit 的 UUID 进行标识。

> 以上有些理论知识不太需要知道，写出来主要目的是为了对Services，Characteristics，UUID有个更好的了解，也表述的比较简单，有错误帮忙更正一下。

## 文档
目前微信小程序只支持低功耗蓝牙连接，经典蓝牙在社区里看到说在规划中，微信小程序官方文档把与蓝牙相关的API分为两部分 [低功耗蓝牙](https://developers.weixin.qq.com/miniprogram/dev/api/device/bluetooth-ble/wx.writeBLECharacteristicValue.html) 和 [蓝牙](https://developers.weixin.qq.com/miniprogram/dev/api/device/bluetooth/wx.stopBluetoothDevicesDiscovery.html)，我认为`蓝牙`这类的API用来小程序与系统（手机）蓝牙之间的交互，而`低功耗蓝牙`这类API是小程序通过系统蓝牙与低功耗蓝牙设备之间的交互。

* 低功耗蓝牙：包括创建蓝牙连接、向低功耗蓝牙设备读写数据、监听事件等
* 蓝牙：包括初始化蓝牙适配器、搜索蓝牙设备、获取蓝牙设备列表等

具体API请查看官方文档。

## 流程
流程以一个完整的蓝牙连接（开始到结束）进行描述，不考虑业务逻辑。
1. 调用 [wx.openBluetoothAdapter](https://developers.weixin.qq.com/miniprogram/dev/api/device/bluetooth/wx.openBluetoothAdapter.html) 初始化蓝牙模块，请确保手机蓝牙开启，微信拥有蓝牙权限。其他蓝牙相关 API 必须在 wx.openBluetoothAdapter 调用之后使用。
2. [可选] 通过 [wx.onBluetoothAdapterStateChange](https://developers.weixin.qq.com/miniprogram/dev/api/device/bluetooth/wx.onBluetoothAdapterStateChange.html) 监听手机蓝牙状态的改变。
3. 调用 [wx.startBluetoothDevicesDiscovery](https://developers.weixin.qq.com/miniprogram/dev/api/device/bluetooth/wx.startBluetoothDevicesDiscovery.html) 搜寻附近的蓝牙外围设备。设置services参数，则只会搜索 主服务的UUID 符合设定的蓝牙设备，从而提高搜索效率。一般同一类型设备主服务的UUID是一样的，以微信硬件平台的蓝牙智能灯为例，主服务的UUID是`FEE7`。

    ```
      // 只搜索主服务 UUID 为 FEE7 的设备
      wx.startBluetoothDevicesDiscovery({
        services: ['FEE7'],
        success (res) {
          console.log(res)
        }
      })
    ```

4. 调用 [wx.onBluetoothDeviceFound](https://developers.weixin.qq.com/miniprogram/dev/api/device/bluetooth/wx.onBluetoothDeviceFound.html) 监听搜索到的新设备。在回调函数里可通过advertisData或deviceId或name等匹配目标设备，具体用什么匹配要根据应用协议，需要注意的是，Android上获取到的deviceId为设备MAC地址，iOS上则为设备uuid。这里也可以在开启蓝牙搜索之后，延迟几秒使用 [wx.getBluetoothDevices](https://developers.weixin.qq.com/miniprogram/dev/api/device/bluetooth/wx.getBluetoothDevices.html) 获取搜索到的蓝牙设备然后在进行匹配，但在延迟的几秒钟里不一定能搜索到设备，前者可能更适用一些。还有一点需要注意的是部分安卓手机需要开启定位权限才能搜索到设备。

    ```
      // devices 设备列表结构
      [
        {
          RSSI: (Number),
          advertisData: (ArrayBuffer),
          advertisServiceUUIDs: (Array),
          deviceId: (String),
          localName: (String),
          name: (String),
        }
      ]
    
      // 匹配到device，取deviceId
      deviceId = device.deviceId
    ```

5. 在匹配到低功耗蓝牙设备之后，调用 [wx.stopBluetoothDevicesDiscovery](https://developers.weixin.qq.com/miniprogram/dev/api/device/bluetooth/wx.stopBluetoothDevicesDiscovery.html) 停止搜索。
6. 调用 [wx.createBLEConnection](https://developers.weixin.qq.com/miniprogram/dev/api/device/bluetooth-ble/wx.createBLEConnection.html) 连接低功耗蓝牙设备，参数deviceId传第4步获取到的deviceId。若小程序在之前已有搜索过某个蓝牙设备，并成功建立连接，可直接传入之前搜索获取的 deviceId 直接尝试连接该设备，无需进行搜索操作。
7. [可选] 通过 [wx.onBLEConnectionStateChange](https://developers.weixin.qq.com/miniprogram/dev/api/device/bluetooth-ble/wx.onBLEConnectionStateChange.html) 监听蓝牙连接状态变化。
8. 调用 [wx.getBLEDeviceServices](https://developers.weixin.qq.com/miniprogram/dev/api/device/bluetooth-ble/wx.getBLEDeviceServices.html) 获取低功耗蓝牙设备所有服务，参数deviceId传第4步获取到的deviceId。

    ```
      // services 结构
      [
        {
          isPrimary: (Boolean),
          uuid: (String)
        }
      ]
    
      // service 根据是否为主服务（isPrimary: true）或者uuid是否包含`FEE7`（第3步提到的）??
      serviceId = service.uuid
    ```

9. 调用 [wx.getBLEDeviceCharacteristics](https://developers.weixin.qq.com/miniprogram/dev/api/device/bluetooth-ble/wx.getBLEDeviceCharacteristics.html) 获取某个服务的所有特征值。特征值描述了服务是否可读，是否可写，是否支持通知等。参数deviceId传第4步获取到的deviceId，serviceId传从上一步获取的serviceId。

    ```
      // characteristics 结构
      [
        {
          uuid: (String),
          properties: (Object) {
            read: (Boolean)
            write: (Boolean)
            notify: (Boolean)
            indicate: (Boolean)
          }
        }
      ]
    ```

10. 如果设备的特征值支持 notify 或者 indicate，调用 [wx.notifyBLECharacteristicValueChange](https://developers.weixin.qq.com/miniprogram/dev/api/device/bluetooth-ble/wx.notifyBLECharacteristicValueChange.html) 启用低功耗蓝牙设备特征值变化时的 notify 功能，开启成功后再调用 [wx.onBLECharacteristicValueChange](https://developers.weixin.qq.com/miniprogram/dev/api/device/bluetooth-ble/wx.onBLECharacteristicValueChange.html) 监听低功耗蓝牙设备的特征值变化。
11. 上一步设置完后，如果设备的特征值支持write，通过 [wx.writeBLECharacteristicValue](https://developers.weixin.qq.com/miniprogram/dev/api/device/bluetooth-ble/wx.writeBLECharacteristicValue.html) 就可以向低功耗设备写二进制数了，如果数据包超过20个字节，分包发送。写成功会触发onBLECharacteristicValueChange监听事件。
12. 调用 [wx.closeBLEConnection](https://developers.weixin.qq.com/miniprogram/dev/api/device/bluetooth-ble/wx.closeBLEConnection.html) 关闭连接，调用 [wx.closeBluetoothAdapter](https://developers.weixin.qq.com/miniprogram/dev/api/device/bluetooth/wx.closeBluetoothAdapter.html) 关闭蓝牙模块。

    ![流程图](/flow.png)

## 编码
代码封装我觉得可以分为两块，一是对微信小程序蓝牙API的封装，二是业务逻辑（包括协议解析）的封装，第一块很好理解，比如把一些连贯调用的API整合成一个函数，或者把蓝牙API封装成Promise函数，方便在写第二块的时候使用，也可以在别的项目使用，但仔细想想，其实也没必要，看个人习惯吧。第二块就是把业务逻辑和协议解析封装一下，提供一些常用的API，如操作API（连接蓝牙、配对、发送指令等步骤），获取设备各种状态API（发送操作指令之后UI需要同步状态），监听蓝牙状态API（UI可能需要蓝牙连接状态），结束操作API（关闭连接，关闭蓝牙模块释放资源等）。

## 写在最后
一般蓝牙连接有个配对的过程，在上面所述第10步完成之后，我们需要根据协议发起配对，配对成功之后再发送控制指令。所以最好是配对成功我们才认为与低功耗设备蓝牙连接成功，而不是在第6步。


胡说八道~
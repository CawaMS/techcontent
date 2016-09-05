<properties
 pageTitle="远程监视解决方案中的设备信息元数据 | Azure"
 description="介绍 Azure IoT 预配置解决方案远程监视及其体系结构。"
 services=""
 suite="iot-suite"
 documentationCenter=""
 authors="dominicbetts"
 manager="timlt"
 editor=""/>  


<tags
 ms.service="iot-suite"
 ms.date="06/20/2016"
 wacn.date="08/22/2016"/>  


# 远程监视预配置解决方案中的设备信息元数据

Azure IoT 套件远程监视预配置解决方案演示了管理设备元数据的方法。本文将概述此解决方案为了帮助你理解内容而采用的方法：

- 解决方案存储哪些设备元数据。
- 解决方案如何管理设备元数据。

## 上下文

远程监视预配置解决方案使用 [Azure IoT 中心][lnk-iot-hub]，使设备能够将数据发送到云。IoT 中心包含一个[设备标识注册表][lnk-identity-registry]用于控制对 IoT 中心的访问。IoT 中心设备标识注册表与用于存储设备信息元数据的远程监视方案特定设备注册表不同。远程监视解决方案使用 [DocumentDB][lnk-docdb] 数据库来实现其用于存储设备信息元数据的设备注册表。[Microsoft Azure IoT 参考体系结构][lnk-ref-arch]描述了典型 IoT 解决方案中设备注册表的角色。

> [AZURE.NOTE] 远程监视预配置解决方案可使设备标识注册表与设备注册表保持同步。这两个注册表使用相同的设备 ID 来唯一标识连接到 IoT 中心的每个设备。

[IoT 中心设备管理预览][lnk-dm-preview]将类似于本文所述远程监视预配置解决方案中的设备信息管理功能的功能添加到 IoT 中心。但是，远程监视解决方案目前只使用 IoT 中心的正式版 (GA) 功能。

## 设备信息元数据

存储在设备注册表 DocumentDB 数据库中的设备信息元数据 JSON 文档具有以下结构：

```
{
  "DeviceProperties": {
    "DeviceID": "deviceid1",
    "HubEnabledState": null,
    "CreatedTime": "2016-04-25T23:54:01.313802Z",
    "DeviceState": "normal",
    "UpdatedTime": null
    },
  "SystemProperties": {
    "ICCID": null
  },
  "Commands": [],
  "CommandHistory": [],
  "IsSimulatedDevice": false,
  "id": "fe81a81c-bcbc-4970-81f4-7f12f2d8bda8"
}
```

- **DeviceProperties**：设备本身写入这些属性，并且设备是此数据的管理机构。其他示例设备属性包括制造商、型号和序号。
- **DeviceID**：唯一的设备 ID。此值在 IoT 中心设备标识注册表中相同。
- **HubEnabledState**：设备在 IoT 中心内的状态。在设备首次建立连接之前，此属性的初始值设置为 **null**。在解决方案门户中，此属性表示为“已注册但不存在”的设备。
- **CreatedTime**：设备创建时间。
- **DeviceState**：设备报告的状态。
- **UpdatedTime**：上次通过解决方案门户更新设备的时间。
- **SystemProperties**：解决方案门户写入系统属性，设备并不知道这些属性。如果解决方案已获授权并连接到负责管理启用 SIM 的设备的服务，**ICCID** 即为一个系统属性示例。
- **Commands**：设备支持的命令列表。设备向解决方案提供此信息。
- **CommandHistory**：远程监视解决方案发送给设备的命令列表，以及这些命令的状态。
- **IsSimulatedDevice**：将设备标识为模拟设备的标志。
- **id**：此设备文档的唯一 DocumentDB 标识符。

> [AZURE.NOTE] 设备信息还可能包含元数据，用于描述设备发送给 IoT 中心的遥测数据。远程监视解决方案使用此遥测元数据来自定义仪表板显示[动态遥测数据][lnk-dynamic-telemetry]的方式。

## 生命周期

当你首次在解决方案门户中创建设备时，解决方案将在其设备注册表中创建一个如上所示的条目。有许多信息最初是虚构的，并且 **HubEnabledState** 设置为 **null**。此时，解决方案还会在 IoT 中心设备标识注册表中为设备创建一个条目，以生成设备用来向 IoT 中心进行身份验证的密钥。

当设备首次连接到解决方案时，将发送设备信息消息。此设备信息消息包含设备属性，例如设备制造商、型号和序号。设备信息消息还包含设备支持的命令列表，其中包括有关任何命令参数的信息。当解决方案收到此消息时，将更新设备注册表中的设备信息元数据。

### 在解决方案门户中查看和编辑设备信息

解决方案门户中的设备列表以列的形式显示以下设备属性：“状态”、“设备 ID”、“制造商”、“型号”、“序号”、“固件”、“平台”、“处理器”和“已安装的 RAM”。设备属性“纬度”和“经度”驱动仪表板上必应地图中的位置。

![列出设备][img-device-list]  


如果在解决方案门户的“设备详细信息”窗格中单击“编辑”，可以编辑上述所有属性。编辑这些属性会更新 DocumentDB 数据库中设备的记录；但是，如果设备发送已更新的设备信息消息，则会覆盖在解决方案门户中进行的所有更改。你无法在解决方案门户中编辑 **DeviceId**、**Hostname**、**HubEnabledState**、**CreatedTime**、**DeviceState** 和 **UpdatedTime** 属性，因为只有设备具有这些属性的权限。

![编辑设备][img-device-edit]  


可以使用解决方案门户从解决方案中删除设备。当你删除设备时，解决方案将从解决方案设备注册表中删除设备信息元数据，并删除 IoT 中心设备标识注册表中的设备条目。在删除设备之前必须先将它禁用。

![删除设备][img-device-remove]  


## 设备信息消息处理

设备发送的设备信息消息与遥测消息不同，前者包含设备属性、设备可以响应的命令和任何命令历史记录等信息。IoT 中心本身不知道设备信息消息中包含的元数据，它以处理任何设备到云消息的相同方式处理消息。在远程监视解决方案中，[Azure 流分析][lnk-stream-analytics] (ASA) 作业读取来自 IoT 中心的消息。**DeviceInfo** 流分析作业筛选包含 **"ObjectType": "DeviceInfo"** 的消息，并将这些消息转发到 Web 作业中运行的 **EventProcessorHost** 主机实例。**EventProcessorHost** 实例中的逻辑使用设备 ID 来查找特定设备的 DocumentDB 记录并更新记录。设备注册表记录现在包含设备属性、命令和命令历史记录等信息。

> [AZURE.NOTE] 设备信息消息是标准的设备到云消息。解决方案使用 ASA 查询来区分设备信息消息与遥测消息。

## 示例设备信息记录

远程监视预配置解决方案使用两种设备信息记录：使用解决方案部署的模拟设备的记录，以及连接到解决方案的自定义设备的记录。

### 模拟设备

以下示例显示了模拟设备的 JSON 设备信息记录。此记录已设置 **UpdatedTime** 的值，表示设备已向 IoT 中心发送 **DeviceInfo** 消息。该记录包含一些通用设备属性、定义模拟设备支持的六个命令，并将 **IsSimulatedDevice** 标志设置为 **1**。

```
{
  "DeviceProperties": {
    "DeviceID": "SampleDevice001_455",
    "HubEnabledState": true,
    "CreatedTime": "2016-01-26T19:02:01.4550695Z",
    "DeviceState": "normal",
    "UpdatedTime": "2016-06-01T15:28:41.8105157Z",
    "Manufacturer": "Contoso Inc.",
    "ModelNumber": "MD-369",
    "SerialNumber": "SER9009",
    "FirmwareVersion": "1.39",
    "Platform": "Plat-34",
    "Processor": "i3-2191",
    "InstalledRAM": "3 MB",
    "Latitude": 47.583582,
    "Longitude": -122.130622
  },
  "Commands": [
    {
      "Name": "PingDevice",
      "Parameters": null
    },
    {
      "Name": "StartTelemetry",
      "Parameters": null
    },
    {
      "Name": "StopTelemetry",
      "Parameters": null
    },
    {
      "Name": "ChangeSetPointTemp",
      "Parameters": [
        {
          "Name": "SetPointTemp",
          "Type": "double"
        }
      ]
    },
    {
      "Name": "DiagnosticTelemetry",
      "Parameters": [
        {
          "Name": "Active",
          "Type": "boolean"
        }
      ]
    },
    {
      "Name": "ChangeDeviceState",
      "Parameters": [
        {
          "Name": "DeviceState",
          "Type": "string"
        }
      ]
    }
  ],
  "CommandHistory": [],
  "IsSimulatedDevice": 1,
  "Version": "1.0",
  "ObjectType": "DeviceInfo",
  "IoTHub": {
    "MessageId": null,
    "CorrelationId": null,
    "ConnectionDeviceId": "SampleDevice001_455",
    "ConnectionDeviceGenerationId": "635894317219942540",
    "EnqueuedTime": "0001-01-01T00:00:00",
    "StreamId": null
  },
  "SystemProperties": {
    "ICCID": null
  },
  "id": "7101c002-085f-4954-b9aa-7466980a2aaf"
}
```

### 自定义设备

以下示例显示了自定义设备的 JSON 设备信息记录，并将 **IsSimulatedDevice** 标志设置为 **0**。你可以看到，此自定义设备支持两个命令，并且解决方案门户已向设备发送 **SetTemperature** 命令：

```
{
  "DeviceProperties": {
    "DeviceID": "mydevice01",
    "HubEnabledState": true,
    "CreatedTime": "2016-03-28T21:05:06.6061104Z",
    "DeviceState": "normal",
    "UpdatedTime": "2016-06-07T22:05:34.2802549Z"
  },
  "SystemProperties": {
    "ICCID": null
  },
  "Commands": [
    {
      "Name": "SetHumidity",
      "Parameters": [
        {
          "Name": "humidity",
          "Type": "int"
        }
      ]
    },
    {
      "Name": "SetTemperature",
      "Parameters": [
        {
          "Name": "temperature",
          "Type": "int"
        }
      ]
    }
  ],
  "CommandHistory": [
    {
      "Name": "SetTemperature",
      "MessageId": "2a0cec61-5eca-4de7-92dc-9c0bc4211c46",
      "CreatedTime": "2016-06-07T21:05:18.140796Z",
      "Parameters": {
        "temperature": 20
      },
      "UpdatedTime": "2016-06-07T21:05:18.716076Z",
      "Result": "Expired"
    }
  ],
  "IsSimulatedDevice": 0,
  "id": "6184ae0f-2d94-4fbd-91cd-4b193aecc9d1",
  "ObjectType": "DeviceInfo",
  "Version": "1.0",
  "IoTHub": {
    "MessageId": null,
    "CorrelationId": null,
    "ConnectionDeviceId": "SampleCustom",
    "ConnectionDeviceGenerationId": "635947959068246845",
    "EnqueuedTime": "0001-01-01T00:00:00",
    "StreamId": null
  }
}
```

下图显示了设备为了更新设备信息元数据而发送的 JSON **DeviceInfo** 消息：

```
{ "ObjectType":"DeviceInfo",
  "Version":"1.0",
  "IsSimulatedDevice":false,
  "DeviceProperties": { "DeviceID":"mydevice01", "HubEnabledState":true },
  "Commands": [
    {"Name":"SetHumidity", "Parameters":[{"Name":"humidity","Type":"double"}]},
    {"Name":"SetTemperature", "Parameters":[{"Name":"temperature","Type":"double"}]}
  ]
}
```

## 后续步骤

可以在 [Connect your device to the remote monitoring preconfigured solution][lnk-connect-custom]（将设备连接到远程监视预配置解决方案）中详细了解如何从自定义设备发送设备信息消息。

可以在 [Customize a preconfigured solution][lnk-customize]（自定义预配置解决方案）详细了解如何自定义预配置解决方案。

<!-- Images and links -->

[img-device-list]: ./media/iot-suite-remote-monitoring-device-info/image1.png
[img-device-edit]: ./media/iot-suite-remote-monitoring-device-info/image2.png
[img-device-remove]: ./media/iot-suite-remote-monitoring-device-info/image3.png

[lnk-iot-hub]: /documentation/services/iot-hub/
[lnk-identity-registry]: /documentation/articles/iot-hub-devguide/#device-identity-registry
[lnk-docdb]: /documentation/services/documentdb/
[lnk-ref-arch]: http://download.microsoft.com/download/A/4/D/A4DAD253-BC21-41D3-B9D9-87D2AE6F0719/Microsoft_Azure_IoT_Reference_Architecture.pdf
[lnk-connect-custom]: /documentation/articles/iot-suite-connecting-devices/
[lnk-customize]: /documentation/articles/iot-suite-guidance-on-customizing-preconfigured-solutions/
[lnk-stream-analytics]: /documentation/services/stream-analytics/
[lnk-dm-preview]: /documentation/articles/iot-hub-device-management-overview/
[lnk-dynamic-telemetry]: /documentation/articles/iot-suite-dynamic-telemetry/

<!---HONumber=Mooncake_0815_2016-->
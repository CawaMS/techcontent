<properties
 pageTitle="IoT 中心 HA 和 DR | Azure"
 description="介绍可帮助你构建包含灾难恢复功能的高可用性 IoT 解决方案的功能。"
 services="iot-hub"
 documentationCenter=""
 authors="fsautomata"
 manager="timlt"
 editor=""/>

<tags
 ms.service="iot-hub"
 ms.date="02/03/2016"
 wacn.date="08/01/2016"/>

# IoT 中心高可用性和灾难恢复

作为一项 Azure 服务，IoT 中心在 Azure 区域级别使用冗余来提供高可用性 (HA)，而解决方案不需要执行任何额外的工作。此外，Azure 还可根据需要提供一些功能，使你能够构建包含灾难恢复 (DR) 功能或跨区域可用性的解决方案。若要提供全局跨区域的高可用性给设备或用户，必须妥善设计并准备好解决方案以利用这些 DR 功能。

## Azure IoT 中心 DR
除了区域内部的 HA，IoT 中心还实施了无需用户干预的灾难恢复故障转移机制。IoT 中心 DR 自行启动，其恢复时间目标 (RTO) 为 2 到 26 小时，恢复点目标 (RPO) 如下所示。

| 功能 | RPO |
| ------------- | --- |
| 注册表和通信操作的服务可用性 | 可能丢失 CName |
| 设备标识注册表中的标识数据 | 丢失 0-5 分钟的数据 |
| 设备到云的消息 | 所有未读的消息都丢失 |
| 操作监视消息 | 所有未读的消息都丢失 |
| 云到设备的消息 | 丢失 0-5 分钟的数据 |
| 云到设备的反馈队列 | 所有未读的消息都丢失 |

## IoT 中心区域故障转移

IoT 解决方案中部署拓扑的完整处理已超出本文的范围，但为了实现高可用性和灾难恢复，我们需要考虑区域故障转移部署模型。

在区域故障转移模型中，解决方案后端主要是在一个数据中心位置运行，但其他 IoT 中心和后端将部署在其他数据中心位置以进行故障转移，以防主要数据中心内的 IoT 中心发生中断，或者从设备到主要数据中心的网络连接因某种原因而中断。每当无法连接主要网关时，设备将使用辅助服务终结点。使用跨区域故障转移功能可改善解决方案的可用性，使其优于单个区域的高可用性。

从较高层面上讲，为了实现 IoT 中心的区域故障转移模型，需要做好以下准备。

* **辅助 IoT 中心和设备路由逻辑**：如果主要区域的服务中断，设备必须开始连接到次要区域。由于大多数服务状态感知的性质，解决方案管理员通常触发区域间的故障转移过程。要使新终结点与设备通信，同时保留过程控制权，最佳方式是让它们定期在监护服务中检查是否存在当前活动的终结点。该监护服务可以是简单的 Web 应用程序，可使用 DNS 重定向技术将它复制并使其可访问（例如，使用 [Azure 流量管理器][]）。
* **标识注册表复制** - 为了可用，辅助 IoT 中心必须包含能够连接到解决方案的所有设备标识。解决方案应该保留设备标识的异地复制备份，并在切换设备的活动终结点之前将其上载到辅助 IoT 中心。IoT 中心的设备标识导出功能在此上下文中非常有用。有关详细信息，请参阅 [IoT 中心开发人员指南 - 标识注册表][]。
* **合并逻辑** - 当主要区域再次可供使用时，所有在辅助站点中创建的状态和数据都必须迁移回到主要区域。这主要与设备标识和应用程序元数据相关，必须与主要 IoT 中心合并，也可能要与主要区域中的其他应用程序特定存储合并。为了简化此步骤，我们通常建议你使用幂等操作。这样可以将副作用降到最低，不只包括来自最终一致的事件分布的副作用，也包括来自事件的重复项目或失序传送的副作用。此外，应用程序逻辑应该设计为能够容许潜在的不一致或“稍微”过期的状态。这是因为系统需要额外的时间来根据恢复点目标 (RPO)“修复”自身。

## 后续步骤

若要了解有关 Azure IoT 中心的详细信息，请参阅以下链接：

- [IoT 中心入门（教程）][lnk-get-started]
- [Azure IoT 中心是什么？][]

[Azure 业务连续性技术指南]: /documentation/articles/resiliency-technical-guidance/
[Azure 应用程序的灾难恢复和高可用性]: /documentation/articles/resiliency-disaster-recovery-high-availability-azure-applications/
[防故障：弹性云体系结构指南]: https://msdn.microsoft.com/zh-cn/library/azure/jj853352.aspx
[Azure 流量管理器]: /documentation/services/traffic-manager/
[IoT 中心开发人员指南 - 标识注册表]: /documentation/articles/iot-hub-devguide/#identityregistry

[lnk-get-started]: /documentation/articles/iot-hub-csharp-csharp-getstarted/
[Azure IoT 中心是什么？]: /documentation/articles/iot-hub-what-is-iot-hub/

<!---HONumber=Mooncake_0307_2016-->
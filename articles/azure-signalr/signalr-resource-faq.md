---
title: Azure SignalR 服务常见问题解答
description: 获取有关 Azure SignalR 服务的常见问题的解答，包括故障排除和典型使用方案。
author: sffamily
ms.service: signalr
ms.topic: overview
ms.custom: devx-track-dotnet
ms.date: 11/13/2019
ms.author: zhshang
ms.openlocfilehash: d5dd765dd9b174ffbfec35b63ad5e55ce84193ad
ms.sourcegitcommit: de2750163a601aae0c28506ba32be067e0068c0c
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/04/2020
ms.locfileid: "89489555"
---
# <a name="azure-signalr-service-faq"></a>Azure SignalR 服务常见问题解答

## <a name="is-azure-signalr-service-ready-for-production-use"></a>Azure SignalR 服务是否随时可用于生产？

是的。
有关正式发布的公告，请参阅 [Azure SignalR 服务现已正式发布](https://azure.microsoft.com/blog/azure-signalr-service-now-generally-available/)。 

完全支持 [ASP.NET Core SignalR](https://docs.microsoft.com/aspnet/core/signalr/introduction)。

对 ASP.NET SignalR 的支持目前仍以公共预览版提供。 [此处提供了代码示例](https://github.com/aspnet/AzureSignalR-samples/tree/master/aspnet-samples/ChatRoom)。

## <a name="the-client-connection-closes-with-the-error-message-no-server-available-what-does-it-mean"></a>客户端连接关闭并显示错误消息“没有可用的服务器。” 它意味着什么？

仅当客户端正在向 Azure SignalR 服务发送消息时，才会发生此错误。

如果你没有任何应用程序服务器并且只使用 Azure SignalR 服务 REST API，则此行为是设计使然。
在无服务器体系结构中，客户端连接处于“侦听”模式，不会向 Azure SignalR 服务发送任何消息。
阅读[有关 REST API 的详细信息](./signalr-quickstart-rest-api.md)。

如果你有应用程序服务器，则此错误消息意味着没有任何应用程序服务器连接到 Azure SignalR 服务实例。

可能的原因包括：
- 没有任何应用程序服务器连接到 Azure SignalR 服务。 检查应用程序服务器日志，以查看可能的连接错误。 在具有多个应用程序服务器的高可用性设置中，这种情况非常罕见。
- Azure SignalR 服务实例存在连接问题。 此问题是暂时性的，实例将自动恢复。
如果此问题持续一个小时以上，请[在 GitHub 上提出问题](https://github.com/Azure/azure-signalr/issues/new)，或者[在 Azure 中创建支持请求](https://docs.microsoft.com/azure/azure-portal/supportability/how-to-create-azure-support-request)。

## <a name="when-there-are-multiple-application-servers-are-client-messages-sent-to-all-servers-or-just-one-of-them"></a>如果存在多个应用程序服务器，客户端消息是发送到所有服务器，还是只发送到其中的一个服务器？

客户端与应用程序服务器之间存在一对一映射。 来自一个客户端的消息始终会发送到同一个应用程序服务器。

在客户端或应用程序服务器断开连接之前，将保持映射。

## <a name="if-one-of-my-application-servers-is-down-how-can-i-find-it-and-get-notified"></a>如果我的某个应用程序服务器已关闭，如何发现这种情况并收到通知？

Azure SignalR 服务将监视应用程序服务器的检测信号。
如果在指定的一段时间内未收到检测信号，则认为应用程序服务器处于脱机状态。 映射到此应用程序服务器的所有客户端连接将会断开。

## <a name="why-does-my-custom-iuseridprovider-throw-an-exception-when-im-switching-from-aspnet-core-signalr-sdk-to-azure-signalr-service-sdk"></a>当我从 ASP.NET Core SignalR SDK 切换到 Azure SignalR 服务 SDK 时，自定义 `IUserIdProvider` 为何引发异常？

调用 `IUserIdProvider` 时，ASP.NET Core SignalR SDK 与 Azure SignalR 服务 SDK 的 `HubConnectionContext context` 参数不相同。

在 ASP.NET Core SignalR 中，`HubConnectionContext context` 是包含所有属性有效值的物理客户端连接的上下文。

在 Azure SignalR 服务 SDK 中，`HubConnectionContext context` 是逻辑客户端连接的上下文。 物理客户端将连接到 Azure SignalR 服务实例，因此，只提供有限数量的属性。

目前只能访问 `HubConnectionContext.GetHttpContext()` 和 `HubConnectionContext.User`。
你可以[查看源代码](https://github.com/Azure/azure-signalr/blob/dev/src/Microsoft.Azure.SignalR/HubHost/ServiceHubConnectionContext.cs)。

## <a name="can-i-configure-the-transports-available-in-azure-signalr-service-on-the-server-side-with-aspnet-core-signalr-for-example-can-i-disable-websocket-transport"></a>是否可以使用 ASP.NET Core SignalR 配置在服务器端上的 Azure SignalR 服务中可用的传输？ 例如，是否可以禁用 WebSocket 传输？

不是。

默认情况下，Azure SignalR 服务提供 ASP.NET Core SignalR 所支持的所有三种传输。 无法对其进行配置。 Azure SignalR 服务将处理连接以及所有客户端连接的传输。

可以按照 [ASP.NET Core SignalR 配置](https://docs.microsoft.com/aspnet/core/signalr/configuration?view=aspnetcore-2.1&tabs=dotnet#configure-allowed-transports-2)中所述配置客户端传输。

## <a name="what-is-the-meaning-of-metrics-like-message-count-or-connection-count-shown-in-the-azure-portal-which-kind-of-aggregation-type-should-i-choose"></a>Azure 门户中显示的指标（例如消息计数或连接计数）的含义是什么？ 我应该选择哪种聚合类型？

可以在 [Azure SignalR 服务中的消息和连接](signalr-concept-messages-and-connections.md)中找到有关计算这些指标的详细信息。

在 Azure SignalR 服务资源的“概述”窗格上，我们已经为你选择了合适的聚合类型。 如果转到指标窗格，则可以将 [Azure SignalR 服务中的消息和连接](../azure-monitor/platform/metrics-supported.md#microsoftsignalrservicesignalr)聚合类型作为引用。

## <a name="what-is-the-meaning-of-the-default-serverless-and-classic-service-modes-how-can-i-choose"></a>`Default`、`Serverless` 和 `Classic` 服务模式的含义是什么？ 如何选择？

下面是有关模式的信息：
* `Default` 模式需要中心服务器。 在此模式下，Azure SignalR 服务会将客户端流量路由到其连接的中心服务器连接。 Azure SignalR 服务将检查连接的中心服务器。 如果服务找不到连接的中心服务器，它将拒绝传入的客户端连接。 在此模式下，你还可使用管理 API 直接通过 Azure SignalR 服务管理已连接的客户端。
* `Serverless` 模式不允许任何服务器连接。 也就是说，它将拒绝所有服务器连接。 所有客户端必须处于无服务器模式。 客户端连接到 Azure SignalR 服务时，用户通常使用 Azure Functions 之类的无服务器技术来处理中心逻辑。 请参阅在 Azure SignalR 服务中使用无服务器模式的[简单示例](https://docs.microsoft.com/azure/azure-signalr/signalr-quickstart-azure-functions-javascript?WT.mc_id=signalrquickstart-github-antchu)。
* `Classic` 模式为混合状态。 当中心具有服务器连接时，新客户端将路由到中心服务器。 如果中心不具有服务器连接，则客户端将进入无服务器模式。 

  这可能会导致问题。 例如，如果所有服务器连接暂时丢失，则某些客户端将进入无服务器模式，而不是路由到中心服务器。

以下是关于选择模式的一些准则：
- 如果没有中心服务器，请选择 `Serverless`。
- 如果所有中心都有中心服务器，请选择 `Default`。
- 如果只有一部分中心具有中心服务器，你可以选择 `Classic`，但这可能会导致问题。 更好的方法是创建两个实例：一个为 `Serverless`，另一个为 `Default`。

## <a name="are-there-any-feature-differences-in-using-azure-signalr-service-with-aspnet-signalr"></a>将 Azure SignalR 服务与 ASP.NET SignalR 结合使用时是否有功能差异？
使用 Azure SignalR 服务时，ASP.NET SignalR 的某些 API 和功能不受支持：
- 不支持在客户端和中心（通常称为 `HubState`）之间传递任意状态的功能。
- 不支持 `PersistentConnection` 类。
- 不支持永久帧传输。
- 客户端脱机时，Azure SignalR 服务不再重播发送给客户端的消息。
- 使用 Azure SignalR 服务时，在连接期间，一个客户端连接的流量始终路由（也称为“粘滞”）到一个应用服务器实例。

对 ASP.NET SignalR 的支持侧重于兼容性，因此并非 ASP.NET Core SignalR 的所有新功能都受到支持。 例如，MessagePack 和流式处理仅适用于 ASP.NET Core SignalR 应用程序 。

你可以为不同的服务模式（`Classic`、`Default` 和 `Serverless`）配置 Azure SignalR 服务。 ASP.NET 不支持 `Serverless` 模式。 也不支持数据平面 REST API。

## <a name="where-does-my-data-reside"></a>我的数据存放在什么位置？

Azure SignalR 服务以数据处理器服务的形式运行。 它不会存储任何客户内容，且设计包含数据驻留。 如果将 Azure SignalR 服务与其他 Azure 服务（例如 Azure 存储）结合使用以进行诊断，请参阅[此白皮书](https://azure.microsoft.com/resources/achieving-compliant-data-residency-and-security-with-azure/)，以获取有关如何将数据驻留在 Azure 区域中的指南。

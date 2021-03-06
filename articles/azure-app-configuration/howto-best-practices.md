---
title: Azure 应用配置最佳做法 |Microsoft Docs
description: 了解使用 Azure 应用配置时的最佳做法。 涉及的主题包括密钥分组、键值组合、应用配置启动，等等。
services: azure-app-configuration
documentationcenter: ''
author: lisaguthrie
manager: maiye
editor: ''
ms.assetid: ''
ms.service: azure-app-configuration
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: lcozzens
ms.custom: devx-track-csharp, mvc
ms.openlocfilehash: d532b8aab87840f4b6ad90daedba743597f4fe43
ms.sourcegitcommit: 02ca0f340a44b7e18acca1351c8e81f3cca4a370
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/19/2020
ms.locfileid: "88588052"
---
# <a name="azure-app-configuration-best-practices"></a>Azure 应用配置最佳实践

本文介绍使用 Azure 应用配置时的常见模式和最佳实践。

## <a name="key-groupings"></a>键分组

应用配置提供了两个用于组织密钥的选项：

* 密钥前缀
* 标签

可以使用一个或两个选项对密钥进行分组。

*键前缀* 是键的开始部分。 您可以通过在名称中使用相同的前缀对一组密钥进行逻辑分组。 前缀可以包含由分隔符连接的多个组件，如 `/` 类似于 URL 路径，以形成一个命名空间。 当你在一个应用配置存储中存储多个应用程序、组件服务和环境的密钥时，此类层次结构非常有用。

需要记住的重要一点是，应用程序代码会引用这些密钥来检索相应设置的值。 密钥不应更改，否则每次发生时都必须修改你的代码。

*标签* 是键上的一个属性。 它们用于创建密钥的变体。 例如，可以将标签分配给多个版本的密钥。 版本可以是迭代、环境或某些其他上下文信息。 您的应用程序可以通过指定其他标签来请求一组完全不同的键值。 因此，所有键引用在代码中保持不变。

## <a name="key-value-compositions"></a>键-值组合

应用配置将存储的所有密钥视为独立的实体。 应用配置不会尝试推断键之间的任何关系，也不会根据层次结构继承键值。 不过，你可以通过在应用程序代码中结合使用带有正确配置堆栈的标签来聚合多组密钥。

接下来举例说明。 假设你有一个名为 **Asset1**的设置，其值可能因开发环境而异。 创建名为 "Asset1" 的项，其标签为空，标签为 "开发"。 在第一个标签中，你为 **Asset1**设置了默认值，并在后者中为 "开发" 指定了一个特定值。

在代码中，首先检索没有任何标签的键值，然后使用 "开发" 标签第二次检索相同的一组键值。 当你第二次检索值时，将覆盖以前的键值。 .NET Core 配置系统允许您 "堆栈" 多组配置数据彼此之上。 如果一个键存在于多个集中，则使用包含它的最后一个集。 使用新式编程框架（如 .NET Core）时，如果使用本机配置提供程序来访问应用配置，则可免费获取此堆栈功能。 下面的代码片段演示如何在 .NET Core 应用程序中实现堆栈：

```csharp
// Augment the ConfigurationBuilder with Azure App Configuration
// Pull the connection string from an environment variable
configBuilder.AddAzureAppConfiguration(options => {
    options.Connect(configuration["connection_string"])
           .Select(KeyFilter.Any, LabelFilter.Null)
           .Select(KeyFilter.Any, "Development");
});
```

[使用标签为不同环境启用不同配置](./howto-labels-aspnet-core.md) 提供了一个完整的示例。

## <a name="app-configuration-bootstrap"></a>应用配置启动

若要访问应用配置存储，可以使用 Azure 门户中提供的连接字符串。 由于连接字符串包含凭据信息，因此它们被视为机密信息。 这些机密需要存储在 Azure Key Vault 中，你的代码必须通过身份验证才能 Key Vault 检索它们。

更好的选择是使用 Azure Active Directory 中的托管标识功能。 使用托管标识，只需使用 "应用配置终结点 URL" 即可启动对应用配置存储的访问。 可以在应用程序代码中嵌入 URL (例如，在appsettings.js文件) * 上* 。 有关详细信息，请参阅 [与 Azure 托管标识集成](howto-integrate-azure-managed-service-identity.md) 。

## <a name="app-or-function-access-to-app-configuration"></a>应用程序或函数对应用配置的访问

你可以使用以下任一方法为 web 应用或功能提供对应用配置的访问权限：

* 在 Azure 门户中，在应用服务的 "应用程序设置" 中输入应用配置存储的连接字符串。
* 将连接字符串存储到 Key Vault 中的应用配置存储，并 [从应用服务引用它](https://docs.microsoft.com/azure/app-service/app-service-key-vault-references)。
* 使用 Azure 托管标识访问应用配置存储。 有关详细信息，请参阅 [与 Azure 托管标识集成](howto-integrate-azure-managed-service-identity.md)。
* 将配置从应用配置推送到应用服务。 应用配置提供了 Azure 门户的导出功能 (和将数据直接发送到应用服务的 Azure CLI) 。 利用此方法，你无需更改应用程序代码。

## <a name="reduce-requests-made-to-app-configuration"></a>减少对应用配置发出的请求

对应用配置的请求过多可能会导致限制或超额收费。 减少发出的请求数：

* 提高刷新超时时间，尤其是在配置值不经常更改的情况下。 使用[ `SetCacheExpiration` 方法](/dotnet/api/microsoft.extensions.configuration.azureappconfiguration.azureappconfigurationrefreshoptions.setcacheexpiration)指定新的刷新超时值。

* 监视单个 *sentinel 密钥*，而不是监视单个密钥。 仅当 sentinel 密钥更改时才刷新所有配置。 有关示例，请参阅 [在 ASP.NET Core 应用中使用动态配置](enable-dynamic-configuration-aspnet-core.md) 。

* 使用 Azure 事件网格在配置更改时接收通知，而不是不断地轮询任何更改。 有关详细信息，请参阅 [将 Azure 应用配置事件路由到 web 终结点](./howto-app-configuration-event.md)

## <a name="importing-configuration-data-into-app-configuration"></a>将配置数据导入到应用配置

应用配置提供了使用 Azure 门户或 CLI 从当前配置文件中大容量 [导入](https://aka.ms/azconfig-importexport1) 配置设置的选项。 你还可以使用相同的选项，从应用配置导出值，例如在相关存储之间。 如果要设置与 GitHub 存储库的持续同步，可以使用 [Github 操作](https://aka.ms/azconfig-gha2) ，以便可以继续使用现有的源代码管理实践，同时获得应用配置的好处。

## <a name="multi-region-deployment-in-app-configuration"></a>应用配置中的多区域部署

应用配置为 "区域服务"。 对于每个区域具有不同配置的应用程序，将这些配置存储在一个实例中可能会导致单点故障。 对于跨多个区域的每个区域部署一个应用配置实例，可能是更好的选择。 它可帮助实现地区性灾难恢复、性能和安全竖井。 按区域配置还可以提高延迟并使用单独的限制配额，因为限制是按实例进行的。 若要应用灾难恢复缓解措施，可以使用 [多个配置存储](./concept-disaster-recovery.md)。 

## <a name="next-steps"></a>后续步骤

* [键和值](./concept-key-value.md)

---
title: Azure 中的警报和通知监视概述
description: Azure 中的警报概述。 警报、经典警报和警报界面。
ms.subservice: alerts
ms.topic: conceptual
ms.date: 01/28/2018
ms.openlocfilehash: f58175d105e1dd36d58fbe4d8b68109810797b2a
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91317134"
---
# <a name="overview-of-alerts-in-microsoft-azure"></a>Microsoft Azure 中的警报概述 

本文介绍什么是警报及其优点，以及如何开始使用警报。  

## <a name="what-are-alerts-in-microsoft-azure"></a>什么是 Microsoft Azure 中的警报？

当你的基础结构或应用程序使用 Azure Monitor 中的监视数据发现问题时，警报会主动通知你。 有了警报，你就可以在系统的用户注意到问题之前确定和解决这些问题。 

## <a name="overview"></a>概述

下图表示了警报流。 

![警报流示意图](media/alerts-overview/Azure-Monitor-Alerts.svg)

警报规则独立于警报，也独立于警报触发时采取的操作。 警报规则捕获警报的目标和条件。 警报规则可以是“已启用”或“已禁用”状态。 警报只有在启用后才会触发。 

下面是警报规则的关键属性：

**目标资源** - 定义适用于警报的范围和信号。 目标可以是任何 Azure 资源。 示例目标：

- 虚拟机。
- 存储帐户。
- Log Analytics 工作区。
- Application Insights。 

对于某些资源（例如虚拟机）来说，可以将多个资源指定为警报规则的目标。

由目标资源发出的**信号**。 信号可以是以下类型：指标、活动日志、Application Insights 和日志。

**条件** -应用于目标资源的信号和逻辑的组合。 示例: 

- CPU 百分比 > 70%
- 服务器响应时间 > 4 毫秒 
- 日志查询的结果计数 > 100

**警报名称** -由用户配置的警报规则的特定名称。

**警报描述** -用户配置的警报规则的说明。

**严重性** -满足警报规则中指定的条件后警报的严重性。 严重性的范围为 0 到 4。

- 严重性为 0 = 严重
- 严重性为 1 = 错误
- 严重性为 2 = 警告
- 严重性为 3 = 信息
- 严重性为 4 = 详细 

**操作** - 触发警报时执行的特定操作。 有关详细信息，请参阅[操作组](./action-groups.md)。

## <a name="what-you-can-alert-on"></a>可以报警的内容

可以按照[监视数据源](./data-sources.md)中的说明，针对指标和日志发出警报。 信号包括但不限于：

- 指标值
- 日志搜索查询
- 活动日志事件
- 基础 Azure 平台的运行状况
- 网站可用性测试

## <a name="manage-alerts"></a>管理警报

可以设置警报状态来指定它在解决过程中所处的阶段。 符合警报规则中指定的条件以后，就会创建或触发警报，其状态为“新”。 可以在确认警报和关闭警报时更改状态。 所有状态更改都存储在警报历史记录中。

支持以下警报状态。

| 状态 | 说明 |
|:---|:---|
| 新建 | 已检测到该问题，但尚未查看。 |
| 已确认 | 管理员已审查警报，并已开始进行处理。 |
| 已关闭 | 问题已解决。 关闭某个警报后，可通过将其更改为另一种状态来重新打开它。 |

*警报状态*不同于且独立于*监视条件*。 警报状态是由用户设置的。 监视条件是由系统设置的。 当警报触发时，警报的监视条件设置为 "已 *激发"*; 当导致触发警报的基础条件清除时，监视条件设置为 *"已解决"*。 

在用户更改警报状态之前，警报状态不会改变。 了解[如何更改警报和智能组的状态](https://aka.ms/managing-alert-smart-group-states)。

## <a name="alerts-experience"></a>警报体验 
默认的“警报”页提供特定时间范围内创建的警报的摘要。 该页显示每种严重性的警报总数，列中会标识处于每种状态的、具有每种严重性的警报总数。 选择任一严重性可打开按该严重性筛选的“[所有警报](#all-alerts-page)”页。

相反，可以 [使用 REST api 以编程方式枚举订阅上生成的警报实例](#manage-your-alert-instances-programmatically)。

> [!NOTE]
   >  只能访问过去 30 天内生成的警报。

它不显示或跟踪经典警报。 可以通过更改订阅或筛选器参数来更新页面。 

![“警报”页的屏幕截图](media/alerts-overview/alerts-page.png)

可以通过选择页面顶部的下拉菜单中的值，来对此视图进行筛选。

| 列 | 说明 |
|:---|:---|
| 订阅 | 选择要查看其警报的 Azure 订阅。 （可选）可以选择你的所有订阅。 视图中仅包含你在所选订阅中有权访问的警报。 |
| 资源组 | 选择单个资源组。 只有包含选定资源组中的目标的警报才会包含在视图中。 |
| 时间范围 | 只有在选定时间范围内触发的警报才会包含在该视图中。 支持的值为过去 1 小时、过去 24 小时、过去 7 天和过去 30 天。 |

选择“警报”页面顶部的以下值打开另一个页面：

| Value | 说明 |
|:---|:---|
| 警报总数 | 符合选定条件的警报总数。 选择此值会打开未经筛选的“所有警报”视图。 |
| 智能组 | 从符合选定条件的警报创建的智能组总数。 选择此值会在“所有警报”视图中打开智能组列表。
| 警报规则总数 | 选定订阅和资源组中的警报规则总数。 选择此值会打开根据选定订阅和资源组筛选的“规则”视图。


## <a name="manage-alert-rules"></a>管理警报规则
若要显示“规则”页，请选择“管理警报规则”。  “规则”页是用于跨 Azure 订阅管理所有警报规则的一个地方。 此页列出所有警报规则，这些规则可以根据目标资源、资源组、规则名称或状态排序。 还可以在此页中编辑、启用或禁用警报规则。  

 ![“规则”页的屏幕截图](./media/alerts-overview/alerts-preview-rules.png)


## <a name="create-an-alert-rule"></a>创建警报规则
您可以通过一致的方式创作警报规则，无论监视服务或信号类型是什么。

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4tflw]

 
下面介绍如何创建新警报规则：
1. 选取警报的目标。
1. 从目标的可用信号中选择信号。
1. 指定要应用到信号中数据的逻辑。

这个创作过程经过了简化，用户在选择 Azure 资源之前，不再需要知道受支持的监视源或信号。 可用信号列表会根据你选择的目标资源自动筛选。 此外，还基于该目标，指导您自动定义警报规则的逻辑。  

可以在[使用 Azure Monitor 创建、查看和管理警报](./alerts-metric.md)中详细了解如何创建警报规则。

警报可在多个 Azure 监视服务中使用。 有关如何以及何时使用其中每种服务的信息，请参阅[监视 Azure 应用程序和资源](../overview.md)。 


## <a name="all-alerts-page"></a>“所有警报”页 
若要查看“所有警报”页，请选择“警报总数”。  在这里，可以查看在选定时间内创建的警报列表。 可以查看各个警报的列表，或包含这些警报的智能组列表。 选择页面顶部的标题可在视图之间进行切换。

![“所有警报”页的屏幕截图](media/alerts-overview/all-alerts-page.png)

可以选择页面顶部下拉菜单中的以下值，对该视图进行筛选：

| 列 | 说明 |
|:---|:---|
| 订阅 | 选择要查看其警报的 Azure 订阅。 （可选）可以选择你的所有订阅。 视图中仅包含你在所选订阅中有权访问的警报。 |
| 资源组 | 选择单个资源组。 只有包含选定资源组中的目标的警报才会包含在视图中。 |
| 资源类型 | 选择一个或多个资源类型。 只有包含选定类型中的目标的警报才会包含在视图中。 仅在指定资源组后，才显示此列。 |
| 资源 | 选择资源。 只有包含该资源（作为目标）的警报才会包含在视图中。 仅在指定资源类型后，才显示此列。 |
| severity | 选择警报严重性，或选择“所有”以包含所有严重性的警报。 |
| 监视条件 | 选择一个监视条件，或选择“所有”以包括所有条件的警报。 |
| 警报状态 | 选择一个警报状态，或选择“所有”以包括所有状态的警报。 |
| 监视服务 | 选择一个服务，或选择“所有”以包含所有服务。 只会包含使用该服务（作为目标）的规则创建的警报。 |
| 时间范围 | 只有在选定时间范围内触发的警报才会包含在该视图中。 支持的值为过去 1 小时、过去 24 小时、过去 7 天和过去 30 天。 |

选择页面顶部的“列”即可选择要显示的列。 

## <a name="alert-details-page"></a>“警报详细信息”页
当你选择某个警报时，此页会提供该警报的详细信息，并允许你更改其状态。

![“警报详细信息”页的屏幕截图](media/alerts-overview/alert-detail2.png)

“警报详细信息”页包括以下部分：

| 部分 | 说明 |
|:---|:---|
| 摘要 | 显示警报的属性和其他重要信息。 |
| 历史记录 | 列出警报执行的每个操作，以及对警报进行的任何更改。 目前仅限状态更改。 |
| 诊断 | 有关包含警报的智能组的信息。 “警报计数”表示包含在智能组中的警报数量。 在过去30天内创建的同一智能组中包含其他警报，警报列表页中的任何时间筛选器都是如此。 选择某个警报以查看其详细信息。 |

## <a name="role-based-access-control-rbac-for-your-alert-instances"></a>警报实例的基于角色的访问控制 (RBAC)

使用和管理警报实例需要用户具有[监视参与者](../../role-based-access-control/built-in-roles.md#monitoring-contributor)或[监视读取者](../../role-based-access-control/built-in-roles.md#monitoring-reader)的 Azure 内置角色。 在任何 Azure 资源管理器范围（从订阅级别到资源级别的粒度分配）内都支持这些角色。 例如，如果用户只具有虚拟机 `ContosoVM1` 的“监视参与者”访问权限，则该用户只能使用和管理 `ContosoVM1` 上生成的警报。

## <a name="manage-your-alert-instances-programmatically"></a>以编程方式管理警报实例

你可能希望以编程方式查询针对订阅生成的警报。 查询可能是在 Azure 门户之外创建自定义视图，也可能是为了分析警报来确定模式和趋势。

可以使用[警报管理 REST API](https://aka.ms/alert-management-api) 或 [Azure Resource Graph](../../governance/resource-graph/overview.md) 和[用于资源的 REST API](/rest/api/azureresourcegraph/resourcegraph(2019-04-01)/resources/resources) 查询针对订阅生成的警报。

用于资源的 Resource Graph REST API 可用于大规模查询警报实例。 如果必须管理多个订阅中生成的警报，则建议使用资源关系图。 

以下对 Resource Graph REST API 的示例请求返回一个订阅中的警报计数：

```json
{
  "subscriptions": [
    <subscriptionId>
  ],
  "query": "AlertsManagementResources | where type =~ 'Microsoft.AlertsManagement/alerts' | summarize count()"
}
```

还可以通过 Azure 资源图资源管理器在门户中查看此资源图形查询的结果： [portal.azure.com](https://portal.azure.com/?feature.customportal=false#blade/HubsExtension/ArgQueryBlade/query/AlertsManagementResources%20%7C%20where%20type%20%3D~%20%27Microsoft.AlertsManagement%2Falerts%27%20%7C%20summarize%20count())

可以查询警报的[基本](alerts-common-schema-definitions.md#essentials)字段。

可以使用[警报管理 REST API](https://aka.ms/alert-management-api) 获取有关特定警报的详细信息，包括其[警报上下文](alerts-common-schema-definitions.md#alert-context)字段。

## <a name="smart-groups"></a>智能组

智能组是根据机器学习算法对警报进行的聚合。这些算法有助于降低警报噪音，对故障排除也有帮助。 [详细了解智能组](https://aka.ms/smart-groups)和[如何管理智能组](https://aka.ms/managing-smart-groups)。

## <a name="next-steps"></a>后续步骤

- [详细了解智能组](https://aka.ms/smart-groups)
- [了解操作组](./action-groups.md)
- [在 Azure 中管理警报实例](https://aka.ms/managing-alert-instances)
- [Managing Smart Groups](https://aka.ms/managing-smart-groups)（管理智能组）
- [详细了解 Azure 警报定价](https://azure.microsoft.com/pricing/details/monitor/)

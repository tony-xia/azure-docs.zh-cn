---
title: Azure Application Insights 中基于日志的指标和预先聚合的指标 | Microsoft Docs
description: 为何要在 Azure Application Insights 中使用基于日志的指标或预先聚合的指标
ms.topic: conceptual
author: vgorbenko
ms.author: vitalyg
ms.date: 09/18/2018
ms.reviewer: mbullwin
ms.openlocfilehash: f7bfa15b12618715bf0d911e4b4927a1fa327107
ms.sourcegitcommit: f5580dd1d1799de15646e195f0120b9f9255617b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/29/2020
ms.locfileid: "91539123"
---
# <a name="log-based-and-pre-aggregated-metrics-in-application-insights"></a>Application Insights 中基于日志的指标和预先聚合的指标

本文介绍了 "传统" Application Insights 基于日志的度量值和预先聚合的度量值之间的差异。 这两种类型的指标都可供 Application Insights 用户使用，每种指标在监视应用程序运行状况、诊断和分析方面发挥了独特的作用。 检测应用程序的开发人员可以根据应用程序的大小、预期遥测量以及指标精度和警报方面的业务要求，确定哪种类型的指标最适合特定的方案。

## <a name="log-based-metrics"></a>基于日志的指标

过去，Application Insights 中的应用程序监视遥测数据模型仅基于少量的预定义类型的事件，例如请求、异常、依赖项调用、页面视图等。开发人员可以使用 SDK 手动发出这些事件 (通过编写显式调用 SDK) 的代码，也可以依赖自动检测的事件的自动收集。 在任一情况下，Application Insights 后端都会将所有收集的事件存储为日志。可以使用 Azure 门户中的 Application Insights 边栏选项卡作为分析和诊断工具来可视化日志中基于事件的数据。

使用日志保留完整事件集能够为分析和诊断带来很大的帮助。 例如，可以获取对特定 URL 发出的确切请求计数，以及发出这些调用的非重复用户数。 或者，可以获取详细的诊断跟踪，包括任何用户会话的异常和依赖项调用。 获取此类信息能够明显提高应用程序运行状况和使用情况的可见性，从而可以缩短诊断应用问题所需的时间。

同时，对于生成大量遥测数据的应用程序而言，收集完整事件集可能不切实际（甚至不可能）。 如果事件量过高，Application Insights 会实施多种遥测数据量缩减技术（例如[采样](./sampling.md)和[筛选](./api-filtering-sampling.md)），以减少收集和存储的事件数量。 遗憾的是，降低存储事件数量也会降低指标的准确性，因此，在幕后必须对日志中存储的事件执行查询时聚合。

> [!NOTE]
> 在 Application Insights 中，基于日志中存储的事件和测量值查询时聚合的指标称为基于日志的指标。 这些指标通常在事件属性中具有许多维度，因此非常适合用于分析，但这些指标的准确性受到采样和筛选的负面影响。

## <a name="pre-aggregated-metrics"></a>预先聚合的指标

除了基于日志的指标以外，在 2018 年年底，Application Insights 团队交付了存储在专用存储库（已针对时序进行优化）中的指标的公共预览版。 新指标不再作为包含大量属性的单个事件进行保存。 它们存储为预先聚合的时序，并且仅包含键维度。 这使得新指标在查询时间方面非常出色：检索数据的速度要快得多，而且所需的计算能力更低。 因此，可以实现[针对指标维度发出近实时警报](../platform/alerts-metric-near-real-time.md)、响应能力更强的[仪表板](./overview-dashboard.md)等方案。

> [!IMPORTANT]
> 基于日志的指标和预先聚合的指标可在 Application Insights 中共存。 为了区分两者，在 Application Insights UX 中，预先聚合的指标现在称为“标准指标(预览版)”，而事件中的传统指标已重命名为“基于日志的指标”。

较新的 Sdk ([Application Insights 2.7](https://www.nuget.org/packages/Microsoft.ApplicationInsights/2.7.2) SDK 或更高版本，适用于 .net) 集合期间预先聚合的度量值。 这适用于  [默认情况下发送的标准指标](../platform/metrics-supported.md#microsoftinsightscomponents) ，因此准确性不受采样或筛选影响。 它也适用于使用 [GetMetric](./api-custom-events-metrics.md#getmetric) 发送的自定义指标，从而减少数据引入量，降低成本。

对于不实施预先聚合的 SDK（即，早期版本的 Application Insights SDK 或用于浏览器检测的 SDK），Application Insights 后端仍会通过聚合 Application Insights 事件收集终结点收到的事件来填充新指标。 这意味着，尽管不能减少通过网络传输的数据量，但仍可以使用预先聚合的指标并改善性能，同时，在收集期间，可以使用不预先聚合指标的 SDK 来支持近实时维度警报。

值得一提的是，收集终结点会在引入采样之前预先聚合事件，这意味着，[引入采样](./sampling.md)永远不会影响预先聚合的指标，不管对应用程序使用哪个 SDK 版本。  

## <a name="using-pre-aggregation-with-application-insights-custom-metrics"></a>对 Application Insights 自定义指标使用预先聚合

可对自定义指标使用预先聚合。 带来的两个主要好处是，能够配置自定义指标的维度并发出其警报，以及减少从 SDK 发送到 Application Insights 收集终结点的数据量。

可通过多种方法[从 Application Insights SDK 发送自定义指标](./api-custom-events-metrics.md)。 如果 SDK 版本提供 [GetMetric 和 TrackValue](./api-custom-events-metrics.md#getmetric) 方法，则最好是使用这些方法发送自定义指标，因为在这种情况下，预先聚合在 SDK 中发生，这不仅可以减少 Azure 中存储的数据量，而且还能减少从 SDK 传输到 Application Insights 的数据量。 否则请使用 [trackMetric](./api-custom-events-metrics.md#trackmetric) 方法。此方法在数据引入期间会预先聚合指标事件。

## <a name="custom-metrics-dimensions-and-pre-aggregation"></a>自定义指标维度和预先聚合

使用 [trackMetric](./api-custom-events-metrics.md#trackmetric) 或 [GetMetric 和 TrackValue](./api-custom-events-metrics.md#getmetric) API 调用发送的所有指标将自动存储在日志和指标存储中。 但是，自定义指标的基于日志的版本始终保留所有维度，而指标的预先聚合版本在存储时默认不包含任何维度。 通过选中 "对自定义指标维度启用警报"，可以在 " [使用情况和估计成本](./pricing.md) " 选项卡上启用自定义度量值的维度收集： 

![用量和预估成本](./media/pre-aggregated-metrics-log-metrics/001-cost.png)

## <a name="why-is-collection-of-custom-metrics-dimensions-turned-off-by-default"></a>为何默认禁用了自定义指标维度的收集？

之所以默认禁用自定义指标维度的收集，是因为存储包含维度的自定义指标将来会在 Application Insights 中单独计费，而存储无维度自定义指标会保持免费（但不能超过配额限制）。 可以在官方[定价页](https://azure.microsoft.com/pricing/details/monitor/)中了解我们即将做出的定价模式变化。

## <a name="creating-charts-and-exploring-log-based-and-standard-pre-aggregated-metrics"></a>创建图表和浏览基于日志的指标与预先聚合的标准指标

使用 [Azure Monitor 指标资源管理器](../platform/metrics-getting-started.md)可以绘制预先聚合的指标和基于日志的指标的图表，以及创作包含图表的仪表板。 选择所需的 Application Insights 资源后，使用命名空间选取器在标准指标（预览版）和基于日志的指标之间切换，或选择自定义指标命名空间：

![指标命名空间](./media/pre-aggregated-metrics-log-metrics/002-metric-namespace.png)

## <a name="pricing-models-for-application-insights-metrics"></a>Application Insights 指标的定价模型

如果将指标引入 Application Insights（无论是基于日志的还是预聚合的），将产生基于引入数据的大小的成本，如[此处](./pricing.md#pricing-model)所述。 自定义指标（包括其所有维度）始终存储在 Application Insights 日志存储中；此外，默认情况下，会将自定义指标（不包含维度）的预聚合版本转发到指标存储。

如果为了在指标存储中存储预聚合指标的所有维度而选择[在自定义指标维度上启用警报](#custom-metrics-dimensions-and-pre-aggregation)选项，会产生基于[自定义指标定价](https://azure.microsoft.com/pricing/details/monitor/)的额外成本。

## <a name="next-steps"></a>后续步骤

* [近实时警报](../platform/alerts-metric-near-real-time.md)
* [GetMetric 和 TrackValue](./api-custom-events-metrics.md#getmetric)

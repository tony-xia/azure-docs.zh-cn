---
title: 从必应语音迁移到语音服务
titleSuffix: Azure Cognitive Services
description: 了解如何从 Azure 认知服务中的现有必应语音订阅迁移到语音服务。
services: cognitive-services
author: wsturman
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 04/03/2020
ms.author: nitinme
ms.openlocfilehash: 81c4c26f252cdd9eb302a7f8f362c8bf52e48629
ms.sourcegitcommit: d2222681e14700bdd65baef97de223fa91c22c55
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/07/2020
ms.locfileid: "91825592"
---
# <a name="migrate-from-bing-speech-to-the-speech-service"></a>从必应语音迁移到语音服务

本文介绍如何将应用程序从必应语音 API 迁移到语音服务。

本文概述了必应语音 Api 与语音服务之间的差异，并建议了迁移应用程序的策略。 必应语音 API 订阅密钥不适用于语音服务;需要新的语音服务订阅。

单个语音服务订阅密钥授予对以下功能的访问权限。 每个功能单独计量，以便仅针对你使用的功能收费。

* [语音转文本](speech-to-text.md)
* [自定义语音到文本](https://cris.ai)
* [文本转语音](text-to-speech.md)
* [自定义文本转语音声音](how-to-customize-voice-font.md)
* [语音翻译](speech-translation.md)（不包括[文本翻译](../translator/translator-info-overview.md)）

[语音 SDK](speech-sdk.md) 是必应语音客户端库的功能替换，但使用了不同 API。

## <a name="comparison-of-features"></a>功能比较

语音服务很大程度上与必应语音相似，但有以下差异。

| 功能 | 必应语音 | 语音服务 | 详细信息 |
|--|--|--|--|
| C# SDK | :heavy_check_mark: | :heavy_check_mark: | Speech service 支持 Windows 10、通用 Windows 平台 (UWP) 和 .NET Standard 2.0。 |
| C++ SDK | :heavy_minus_sign: | :heavy_check_mark: | Speech service 支持 Windows 和 Linux。 |
| Java SDK | :heavy_check_mark: | :heavy_check_mark: | 语音服务支持 Android 和语音设备。 |
| 连续语音识别 | 10 分钟 | 无限制 | 语音 SDK 支持无限制的连续识别，并在超时或断开连接时自动重新连接。 |
| 部分或中期结果 | :heavy_check_mark: | :heavy_check_mark: | 通过语音 SDK 支持。 |
| 自定义语音模型 | :heavy_check_mark: | :heavy_check_mark: | 必应语音需要单独的自定义语音订阅。 |
| 自定义语音字体 | :heavy_check_mark: | :heavy_check_mark: | 必应语音需要单独的自定义语音订阅。 |
| 24 kHz 语音 | :heavy_minus_sign: | :heavy_check_mark: |
| 语音意向识别 | 需要单独的 LUIS API 调用 | 已集成（与 SDK） | 可以将 LUIS 密钥用于语音服务。 |
| 简单意向识别 | :heavy_minus_sign: | :heavy_check_mark: |
| 批量听录长音频文件 | :heavy_minus_sign: | :heavy_check_mark: |
| 识别模式 | 通过终结点 URI 手动 | 自动 | 语音服务中不提供识别模式。 |
| 终结点位置 | 全球 | 区域 | 区域终结点改善延迟。 |
| REST API | :heavy_check_mark: | :heavy_check_mark: | 语音服务 REST Api 与必应语音 (不同的终结点) 兼容。 REST API 支持文本转语音以及限制的语音转文本功能。 |
| Websocket 协议 | :heavy_check_mark: | :heavy_minus_sign: | 语音 SDK 针对需要与服务建立持续连接的功能对 web 套接字连接进行抽象，因此不再支持手动订阅。 |
| 服务到服务 API 调用 | :heavy_check_mark: | :heavy_minus_sign: | 通过 C# 服务库在必应语音中提供。 |
| 开源 SDK | :heavy_check_mark: | :heavy_minus_sign: |

语音服务使用基于时间的定价模型 (而不是基于事务的模型) 。 有关详细信息，请参阅 [语音服务定价](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/) 。

## <a name="migration-strategies"></a>迁移策略

如果你或你的组织在开发或生产环境中具有使用必应语音 API 的应用程序，则应将其更新为尽快使用语音服务。 请参阅 [语音服务文档](index.yml) ，了解可用的 sdk、代码示例和教程。

语音服务 [REST api](rest-apis.md) 与必应语音 api 兼容。 如果当前正在使用必应语音 REST Api，只需更改 REST 终结点，并切换到语音服务订阅密钥。

如果使用特定编程语言的必应语音客户端库，则迁移到[语音 SDK](speech-sdk.md) 需要更改应用程序，因为 API 不同。 语音 SDK 可以简化代码，同时使你可以访问新功能。 语音 SDK 提供多种编程语言。 所有平台上的 API 均类似，从而简化了多平台开发。

语音服务未提供全局终结点。 确定应用程序在使用一个区域终结点处理其所有流量的情况下，能否高效运行。 如果不能，则使用地理位置来确定最高效的终结点。 你需要在每个所使用的区域中使用单独的语音服务订阅。

语音 SDK 入门：

1. 下载[语音 SDK](speech-sdk.md)。
1. 完成语音服务 [快速入门指南](~/articles/cognitive-services/Speech-Service/quickstarts/speech-to-text-from-microphone.md?pivots=programming-language-csharp&tabs=dotnet) 和 [教程](how-to-recognize-intents-from-speech-csharp.md)。 另请查看[代码示例](samples.md)来体验新的 API。
1. 更新应用程序以使用语音服务。

## <a name="support"></a>支持

必应语音客户应通过打开[支持票证](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest)来联系客户支持。 如果你的支持需要一个[技术支持计划](https://azure.microsoft.com/support/plans/)，也可以与我们联系。

有关 Speech service、SDK 和 API 支持，请访问语音服务 [支持页](support.md)。

## <a name="next-steps"></a>后续步骤

* [免费试用语音服务](overview.md#try-the-speech-service-for-free)
* [语音转文本入门](get-started-speech-to-text.md)
* [开始使用文本转语音](get-started-text-to-speech.md)

## <a name="see-also"></a>请参阅

* [语音服务发行说明](releasenotes.md)
* [什么是语音服务](overview.md)
* [语音服务和语音 SDK 文档](speech-sdk.md#get-the-speech-sdk)

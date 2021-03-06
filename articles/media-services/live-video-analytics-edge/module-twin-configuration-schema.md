---
title: 模块克隆 JSON 架构-Azure
description: 本主题介绍 IoT Edge 上的实时视频分析的模块克隆 JSON 架构。
ms.topic: conceptual
ms.date: 04/27/2020
ms.openlocfilehash: 8bd86bdc2c8de9ee586e785db2074fa772100420
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/23/2020
ms.locfileid: "87053073"
---
# <a name="module-twin-json-schema"></a>模块克隆 JSON 架构

设备孪生是存储设备状态信息（例如元数据、配置和条件）的 JSON 文档。 Azure IoT 中心为连接到 IoT 中心的每台设备保留一个设备孪生。 有关详细说明，请参阅[了解和使用 IoT 中心的模块孪生](../../iot-hub/iot-hub-devguide-module-twins.md)

本主题介绍 IoT Edge 上的实时视频分析的模块克隆 JSON 架构。

> [!NOTE]
> 若要有权访问媒体服务资源和媒体服务 API，必须先进行身份验证。 有关详细信息，请参阅[访问 Azure 媒体服务 API](../latest/media-services-apis-overview.md#accessing-the-azure-media-services-api)。

## <a name="module-twin-properties"></a>模块克隆属性

IoT Edge 上的实时视频分析公开以下模块克隆属性。 

|properties |必须 |动态 |描述 |
|---|---|---|---|
|applicationDataDirectory |是 |No |用于保留配置的已装入卷的路径。 |
|azureMediaServicesArmId |是 |No |媒体服务帐户的唯一 Azure 资源管理标识符。|
|aadTenantId |是 |No |客户 Azure AD 租户 ID。|
|aadServicePrincipalAppId |是 |是 |Azure AD AppId 创建的客户。|
|aadServicePrincipalCertificate |是的<sup>*</sup>  |是 |客户已创建 Azure AD AppId 证书。|
|aadServicePrincipalPassword |是的<sup>*</sup>  |是 |Azure AD AppId 密码创建的客户。|
|aadEndpoint |否 |否 |特定于云的 Azure AD 终结点。 <br/>默认值：`https://login.microsoftonline.com` |
|aadResourceId |否 |否 |特定于云的 Azure AD 受众/资源 ID <br/>默认值：`https://management.core.windows.net/` |
|armEndpoint |否 |否 |特定于云的 Azure 资源管理终结点。 <br/>默认值：`https://management.azure.com/` |
|diagnosticsLevel |否 |是 |事件详细级别： <br/>信息 & # x02758;警告 & # x02758;错误 & # x02758;严重 & # x02758;内容 |
|diagnosticsEventsOutputName |否 |是 |诊断事件的集线器输出。 <br/>（空表示不发布诊断）|
|operationalEventsOutputName|否|是|操作事件的中心输出。<br/>（空表示不发布操作事件）
|logLevel|否|是|下列类型作之一： <br/>& # x000B7;详细<br/>& # x000B7;信息（默认值）<br/>& # x000B7;出现<br/>& # x000B7;条<br/>& # x000B7;内容|
|logCategories|否|是|以下项的逗号分隔列表： Application、MediaPipeline、Events <br/>默认：应用程序，事件|
|debugLogsDirectory|否|是|调试日志的目录。 如果生成了现有日志，则禁用 "调试日志"。

<sup>*</sup>必须提供服务主体证书或密码。 

无需重新启动模块即可更新动态属性。 可以按照[获取媒体服务 API 访问权限](../latest/access-api-howto.md)一文中的说明，获取其中几个属性的值。 

有关可选诊断设置的角色的详细信息，请参阅有关[监视和日志记录](monitoring-logging.md)的文章。

```
{ 
    "properties.desired": { 
        // Required 
        "applicationDataDirectory": "/var/lib/azuremediaservices", 
        "azureMediaServicesArmId": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/<rgname>/providers/microsoft.media/mediaservices/<ams_account>", 
        "aadTenantId": "00000000-0000-0000-0000-000000000000", 
        "aadServicePrincipalAppId": "00000000-0000-0000-0000-000000000000", 
        "aadServicePrincipalPassword": "{Service principal password}", 

        // Optional API Access 
        "aadEndpoint": "https://<aad-endpoint>", 
        "aadResourceId": "https://management.core.windows.net/", 
        "armEndpoint": "https://management.azure.com/", 
        
        // Optional Diagnostics 
        "diagnosticsEventsOutputName": "lvaEdgeDiagnostics",
        "operationalEventsOutputName": "lvaEdgeOperational",
        "logLevel": "Information",
        "logCategories": "Application,Events"
    } 
} 
```

## <a name="next-steps"></a>后续步骤

[直接方法](direct-methods.md)

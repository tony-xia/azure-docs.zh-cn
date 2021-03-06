---
title: Azure AD Connect：自动升级 | Microsoft Docs
description: 本主题介绍 Azure AD Connect 同步中的内置自动升级功能。
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: 6b395e8f-fa3c-4e55-be54-392dd303c472
ms.service: active-directory
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 06/09/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 9c8dcc8766b21551f3cd62289805fe735ef0f333
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91317610"
---
# <a name="azure-ad-connect-automatic-upgrade"></a>Azure AD Connect：自动升级
此功能是随内部版本 [1.1.105.0（于 2016 年 2 月发布）](reference-connect-version-history.md)一起推出的。  此功能已在[内部版本 1.1.561](reference-connect-version-history.md) 中更新，现在支持以前不支持的其他方案。

## <a name="overview"></a>概述
使用 **自动升级** 功能是确保 Azure AD Connect 安装始终保持最新状态的最简单方法。 系统默认启用此功能，以便进行快速安装和 DirSync 升级。 发布新版本时，安装会自动升级。
默认情况下，针对以下方案启用自动升级：

* 快速设置安装和 DirSync 升级。
* 使用 SQL Express LocalDB，这是快速设置始终使用的选项。 使用 SQL Express 的 DirSync 也会使用 LocalDB。
* AD 帐户是快速设置和 DirSync 创建的默认 MSOL_ 帐户。
* Metaverse 中的对象少于 100,000 个。

可以使用 PowerShell cmdlet `Get-ADSyncAutoUpgrade`来查看当前的自动升级状态。 状态包括：

| 状态 | 注释 |
| --- | --- |
| Enabled |自动升级已启用。 |
| 已挂起 |只能由系统设置。 系统**目前没有**资格接收自动升级。 |
| 已禁用 |自动升级已禁用。 |

可以使用 `Set-ADSyncAutoUpgrade` 在“已启用”与“已禁用”之间切换。  只有系统才能设置“暂停”状态。  在 1.1.750.0 之前，如果自动升级状态设置为“已暂停”，则 Set-ADSyncAutoUpgrade cmdlet 会阻止自动升级。 此功能现已更改，不阻止自动升级。

自动升级使用 Azure AD Connect Health 作为升级基础结构。 为使自动升级正常工作，请确保根据 [Office 365 URL 和 IP 地址范围](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2)中所述，在代理服务器中打开 **Azure AD Connect Health** 的 URL。


如果服务器上正在运行**同步服务管理器** UI，则会暂停升级，直到 UI 关闭为止。

## <a name="troubleshooting"></a>疑难解答
如果 Connect 安装未按预期自动升级，请遵循以下步骤来找出可能的错误。

首先，不建议在新版本发行的第一天就自动升级。 由于升级前有刻意设计的随机性，因此，不用担心安装没有立即升级。

如果认为有问题，请先运行 `Get-ADSyncAutoUpgrade` 确保已启用自动升级。

如果状态为“已暂停”，则可以使用 `Get-ADSyncAutoUpgrade -Detail` 查看原因。  暂停原因可能包含任何字符串值，但通常包含 UpgradeResult 的字符串值，即 `UpgradeNotSupportedNonLocalDbInstall` 或 `UpgradeAbortedAdSyncExeInUse`。  也可能返回一个复合值，例如 `UpgradeFailedRollbackSuccess-GetPasswordHashSyncStateFailed`。

还有可能得到不是 UpgradeResult 的结果，即“AADHealthEndpointNotDefined”或“DirSyncInPlaceUpgradeNonLocalDb”。

然后，确保已在代理或防火墙中打开所需的 URL。 自动更新正在使用 Azure AD Connect Health，情形如[概述](#overview)中介绍的那样。 如果使用代理，请确保 Health 已配置为使用[代理服务器](how-to-connect-health-agent-install.md#configure-azure-ad-connect-health-agents-to-use-http-proxy)。 另请测试 [Health 的连接](how-to-connect-health-agent-install.md#test-connectivity-to-azure-ad-connect-health-service)（连接到 Azure AD）。

确认与 Azure AD 建立了连接后，可以深入了解事件日志。 启动事件查看器，并查看 **应用程序** 事件日志。 为源 **Azure AD Connect 升级** 和事件 ID 范围 **300-399**添加 eventlog 筛选器。  
![屏幕截图显示 "筛选当前日志" 窗口，其中包含 "事件源"，突出显示 "包括/排除" 事件 Id 框。](./media/how-to-connect-install-automatic-upgrade/eventlogfilter.png)  

此时可以看到与自动升级状态关联的事件日志。  
![用于自动升级的事件日志筛选器](./media/how-to-connect-install-automatic-upgrade/eventlogresult.png)  

结果代码前面会有包含状态概述的前缀。

| 结果代码前缀 | 说明 |
| --- | --- |
| Success |安装已成功升级。 |
| UpgradeAborted |某种临时状态停止了升级。 升级会重试，预期稍后会成功。 |
| UpgradeNotSupported |系统中的某个配置阻止系统自动升级。 升级会重试，以查看状态是否已变化，但预期只能手动升级系统。 |

下面是最常见的消息列表。 该列表并不完整，但结果消息应会明确说明问题所在。

| 结果消息 | 说明 |
| --- | --- |
| **UpgradeAborted** | |
| UpgradeAbortedCouldNotSetUpgradeMarker |无法写入注册表。 |
| UpgradeAbortedInsufficientDatabasePermissions |内置管理员组对数据库没有相应权限。 请手动升级到最新版的 Azure AD Connect 以解决此问题。 |
| UpgradeAbortedInsufficientDiskSpace |没有足够的磁盘空间用于支持升级。 |
| UpgradeAbortedSecurityGroupsNotPresent |找不到且无法解析同步引擎使用的所有安全组。 |
| UpgradeAbortedServiceCanNotBeStarted |NT 服务 **Microsoft Azure AD Sync** 未能启动。 |
| UpgradeAbortedServiceCanNotBeStopped |NT 服务 **Microsoft Azure AD Sync** 未能停止。 |
| UpgradeAbortedServiceIsNotRunning |NT 服务 **Microsoft Azure AD Sync** 未运行。 |
| UpgradeAbortedSyncCycleDisabled |[计划程序](how-to-connect-sync-feature-scheduler.md)中的 SyncCycle 选项已禁用。 |
| UpgradeAbortedSyncExeInUse |服务器上打开了 [Synchronization Service Manager UI](how-to-connect-sync-service-manager-ui.md)。 |
| UpgradeAbortedSyncOrConfigurationInProgress |安装向导正在运行，或者在计划程序外部计划了同步。 |
| **UpgradeNotSupported** | |
| UpgradeNotSupportedCustomizedSyncRules |已将自己的自定义规则添加到配置中。 |
| UpgradeNotSupportedInvalidPersistedState |安装不是快速设置或 DirSync 升级。 |
| UpgradeNotSupportedNonLocalDbInstall |使用的不是 SQL Server Express LocalDB 数据库。 |
|UpgradeNotSupportedLocalDbSizeExceeded|本地 DB 大小大于或等于 8 GB|
|UpgradeNotSupportedAADHealthUploadDisabled|已从门户中禁用运行状况数据上传|

## <a name="next-steps"></a>后续步骤
了解有关[将本地标识与 Azure Active Directory 集成](whatis-hybrid-identity.md)的详细信息。

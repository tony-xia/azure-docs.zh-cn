---
title: 跨 Microsoft 365 services 的管理角色文档-Azure AD |Microsoft Docs
description: 在 Azure Active Directory 中查找针对 Microsoft 365 服务的管理员角色的内容和 API 参考
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: users-groups-roles
ms.topic: reference
ms.date: 11/08/2019
ms.author: curtand
ms.reviewer: vincesm
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6060bc286024183661024abd56567eeefed76430
ms.sourcegitcommit: 814778c54b59169c5899199aeaa59158ab67cf44
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2020
ms.locfileid: "90053542"
---
# <a name="administrator-roles-for-microsoft-365-services"></a>Microsoft 365 服务的管理员角色

在 Azure AD 中，可以使用管理角色管理 Microsoft 365 中的所有产品。 某些产品还提供特定于该产品的其他角色。 有关每个产品所支持角色的信息，请参阅下表。 有关委派问题的常规讨论，请参阅 [Azure Active Directory 中的角色委派计划](roles-concept-delegation.md)。

## <a name="where-to-find-content"></a>查找内容位置

Microsoft 365 服务 | 角色内容 | API 内容
---------------------- | ------------------ | -----------------
Office 365 和 Microsoft 365 业务计划中的管理员角色 | [Microsoft 365 管理员角色](/office365/admin/add-users/about-admin-roles?view=o365-worldwide) | 不可用
Azure Active Directory (Azure AD) 和 Azure AD 标识保护| [Azure AD 管理员角色](directory-assign-admin-roles.md) | [图形 API](/graph/api/overview?view=graph-rest-1.0)<br>[提取角色分配](/graph/api/directoryrole-list?view=graph-rest-1.0)
Exchange Online| [Exchange 基于角色的访问控制](/exchange/understanding-role-based-access-control-exchange-2013-help) |  [PowerShell for Exchange](/powershell/module/exchange/role-based-access-control/add-managementroleentry?view=exchange-ps)<br>[提取角色分配](/powershell/module/exchange/role-based-access-control/get-rolegroup?view=exchange-ps)
SharePoint Online | [Azure AD 管理员角色](directory-assign-admin-roles.md)<br>还 [介绍了中的 SharePoint 管理员角色 Microsoft 365](/sharepoint/sharepoint-admin-role) | [图形 API](/graph/api/overview?view=graph-rest-1.0)<br>[提取角色分配](/graph/api/directoryrole-list?view=graph-rest-1.0)
Teams/Skype for Business | [Azure AD 管理员角色](directory-assign-admin-roles.md) | [图形 API](/graph/api/overview?view=graph-rest-1.0)<br>[提取角色分配](/graph/api/directoryrole-list?view=graph-rest-1.0)
安全与合规中心（Office 365 高级威胁防护、Exchange Online Protection、信息保护） | [Office 365 管理员角色](/office365/SecurityCompliance/permissions-in-the-security-and-compliance-center) | [Exchange PowerShell](/powershell/module/exchange/role-based-access-control/add-managementroleentry?view=exchange-ps)<br>[提取角色分配](/powershell/module/exchange/role-based-access-control/get-rolegroup?view=exchange-ps)
安全评分 | [Azure AD 管理员角色](directory-assign-admin-roles.md) | [图形 API](/graph/api/overview?view=graph-rest-1.0)<br>[提取角色分配](/graph/api/directoryrole-list?view=graph-rest-1.0)
合规性管理器 | [合规性管理器角色](/office365/securitycompliance/meet-data-protection-and-regulatory-reqs-using-microsoft-cloud#permissions-and-role-based-access-control) | 不可用
Azure 信息保护 | [Azure AD 管理员角色](directory-assign-admin-roles.md) | [图形 API](/graph/api/overview?view=graph-rest-1.0)<br>[提取角色分配](/graph/api/directoryrole-list?view=graph-rest-1.0)
Microsoft Cloud App Security | [基于角色的访问控制](/cloud-app-security/manage-admins) | [API 参考](/cloud-app-security/api-tokens) 
Azure 高级威胁防护 | [Azure ATP 角色组](/azure-advanced-threat-protection/atp-role-groups) | 不可用
Windows Defender 高级威胁防护 | [Windows Defender ATP 基于角色的访问控制](/windows/security/threat-protection/windows-defender-atp/rbac-windows-defender-advanced-threat-protection) | 不可用
Privileged Identity Management | [Azure AD 管理员角色](directory-assign-admin-roles.md) | [图形 API](/graph/api/overview?view=graph-rest-1.0)<br>[提取角色分配](/graph/api/directoryrole-list?view=graph-rest-1.0)
Intune | [Intune 基于角色的访问控制](/intune/role-based-access-control) | [图形 API](/graph/api/resources/intune-rbac-conceptual?view=graph-rest-beta)<br>[提取角色分配](/graph/api/intune-rbac-roledefinition-list?view=graph-rest-beta)
托管桌面 | [Azure AD 管理员角色](directory-assign-admin-roles.md) | [图形 API](/graph/api/overview?view=graph-rest-1.0)<br>[提取角色分配](/graph/api/directoryrole-list?view=graph-rest-1.0)

## <a name="next-steps"></a>后续步骤

* [如何分配或删除管理员角色 Azure AD](directory-manage-roles-portal.md)
* [Azure AD 管理员角色参考](directory-assign-admin-roles.md)
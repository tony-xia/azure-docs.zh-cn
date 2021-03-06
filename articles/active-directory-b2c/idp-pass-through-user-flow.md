---
title: 通过用户流将访问令牌传递给应用
titleSuffix: Azure AD B2C
description: 了解如何在 Azure Active Directory B2C 中以用户流中声明的方式传递 OAuth2.0 标识提供者的访问令牌。
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 08/17/2019
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 5b834dda926b7da1241a325e1453143eccafaf30
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/31/2020
ms.locfileid: "87488765"
---
# <a name="pass-an-access-token-through-a-user-flow-to-your-application-in-azure-active-directory-b2c"></a>在 Azure Active Directory B2C 中使用用户流将访问令牌传递给应用程序

Azure Active Directory B2C (Azure AD B2C) 中的[用户流](user-flow-overview.md)允许应用程序的用户通过标识提供者进行注册或登录。 此过程开始时，Azure AD B2C 会从标识提供者处收到一个[访问令牌](tokens-overview.md)。 Azure AD B2C 使用该令牌来检索有关用户的信息。 在用户流中启用声明即可将该令牌传递给你在 Azure AD B2C 中注册的应用程序。

Azure AD B2C 当前仅支持传递[OAuth 2.0](authorization-code-flow.md)标识提供程序（包括[Facebook](identity-provider-facebook.md)和[Google](identity-provider-google.md)）的访问令牌。 对于所有其他标识提供者，声明将返回空白。

## <a name="prerequisites"></a>必备条件

* 应用程序必须使用推荐的[用户流](user-flow-versions.md)。
* 用户流是使用 OAuth 2.0 标识提供者配置的。

## <a name="enable-the-claim"></a>启用声明

1. 以 Azure AD B2C 租户的全局管理员身份登录 [Azure 门户](https://portal.azure.com/)。
2. 请确保使用的是包含 Azure AD B2C 租户的目录。 选择顶部菜单中的“目录 + 订阅”筛选器，然后选择包含租户的目录  。
3. 选择 Azure 门户左上角的“所有服务”  ，搜索并选择 **Azure AD B2C**。
4. 选择“用户流(策略)”  ，然后选择用户流。 例如 B2C_1_signupsignin1。 
5. 选择“应用程序声明”  。
6. 启用“标识提供者访问令牌”  声明。

    ![启用“标识提供者访问令牌”声明](./media/idp-pass-through-user-flow/idp-pass-through-user-flow-app-claim.png)

7. 单击“保存”，保存用户流  。

## <a name="test-the-user-flow"></a>测试用户流

在 Azure AD B2C 中测试应用程序时，可以使 Azure AD B2C 令牌返回到 `https://jwt.ms`，以便在其中查看声明，这可能很有用处。

1. 在用户流的“概览”页上，选择“运行用户流”  。
2. 对于“应用程序”  ，选择你之前注册的应用程序。 “回复 URL”  应当显示 `https://jwt.ms` 才能看到以下示例中的令牌。
3. 单击“运行用户流”，然后使用帐户凭据登录。  应该会在 **idp_access_token** 声明中看到标识提供者的访问令牌。

    应会看到类似于以下示例的内容：

    ![jwt.ms 中突出显示了 idp_access_token 块的已解码令牌](./media/idp-pass-through-user-flow/idp-pass-through-user-flow-token.PNG)

## <a name="next-steps"></a>后续步骤

在 [Azure AD B2C 令牌概述](tokens-overview.md)中了解详细信息。

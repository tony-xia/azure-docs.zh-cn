---
title: 注册调用 Web API 的 Web 应用 - Microsoft 标识平台 | Azure
description: 了解如何注册调用 Web API 的 Web 应用
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/07/2019
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: f94a3da96243e30faa90277ce86efec037f54672
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/03/2020
ms.locfileid: "89436454"
---
# <a name="a-web-app-that-calls-web-apis-app-registration"></a>调用 Web API 的 Web 应用：应用注册

调用 Web API 的 Web 应用与登录用户的 Web 应用具有相同的注册。 因此，请按照[登录用户的 Web 应用：应用注册](scenario-web-app-sign-user-app-registration.md)中的说明操作。

但是，由于 Web 应用现在也调用 Web API，因此它将成为一个机密客户端应用程序。 这就是需要额外注册的原因。 此应用必须与 Microsoft 标识平台共享客户端凭据或机密  。

[!INCLUDE [Registration of client secrets](../../../includes/active-directory-develop-scenarios-registration-client-secrets.md)]

## <a name="api-permissions"></a>API 权限

Web 应用代表已登录用户调用 API。 为此，它们必须请求委托的权限  。 有关详细信息，请参阅 [添加访问 WEB API 的权限](quickstart-configure-app-access-web-apis.md#add-permissions-to-access-your-web-api)。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [调用 Web API 的 Web 应用：代码配置](scenario-web-app-call-api-app-configuration.md)

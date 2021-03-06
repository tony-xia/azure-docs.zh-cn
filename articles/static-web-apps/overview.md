---
title: 什么是 Azure 静态 Web 应用？
description: Azure 静态 Web 应用的主要特性和功能。
services: static-web-apps
author: craigshoemaker
ms.service: static-web-apps
ms.topic: overview
ms.date: 05/08/2020
ms.author: cshoe
ms.openlocfilehash: 086feb3aa7c9a38756fded58538eb22e094a5960
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/05/2020
ms.locfileid: "90906796"
---
# <a name="what-is-azure-static-web-apps-preview"></a>什么是 Azure 静态 Web 应用预览版？

Azure 静态 Web 应用是一种服务，可从 GitHub 存储库自动构建完整的堆栈 Web 应用，并将其部署到 Azure。

:::image type="content" source="media/overview/static-apps-overview.png" alt-text="静态 Web 应用概述":::

Azure 静态 Web 应用的工作流适用于开发人员的日常工作流。 基于 GitHub 交互构建并部署应用。

创建 Azure 静态 Web 应用资源时，Azure 会在应用的源代码存储库中设置 GitHub Actions 工作流，用于监视所选的分支。 每次向受监视的分支推送提交或接受拉取请求时，GitHub Action 都会自动构建应用，并将应用及其 API 部署到 Azure。

静态 Web 应用通常使用诸如 Angular、React、Svelte、Vue 或 Blazor 之类的库和框架进行构建。 这些应用包括构成应用程序的 HTML、CSS、JavaScript 和映像资产。 对于传统 Web 服务器，这些资产与任何所需的 API 终结点一起由单个服务器提供。

使用静态 Web 应用时，静态资产与传统 Web 服务器分离，由分布在世界各地的服务器端提供。 由于文件在物理上离最终用户更近，这种分布使文件提供的速度更快。 此外，API 终结点使用[无服务器体系结构](../azure-functions/functions-overview.md)，无需将完整的后端服务器组合在一起。

## <a name="key-features"></a>主要功能

- 适用于 HTML、CSS、JavaScript 和映像等静态内容的 Web 托管。
- 由 Azure Functions 提供的集成 API 支持。
- 一流的 GitHub 集成，其中存储库更改将触发生成和部署。
- 全球分布的静态内容，使内容更接近你的用户。
- 可自动续订的免费 SSL 证书。
- 自定义域为应用提供品牌自定义。
- 调用 API 时使用反向代理的无缝安全模型，这不需要配置 CORS。
- 身份验证提供程序与 Azure Active Directory、Facebook、Google、GitHub 和 Twitter 集成。
- 可自定义的授权角色定义和分配。
- 后端路由规则，使你能够完全控制所提供的内容和路由。
- 生成的临时版本由拉取请求提供支持，在发布前提供站点的预览版本。

## <a name="what-you-can-do-with-static-web-apps"></a>可对静态 Web 应用执行的操作

- 使用 [Angular](getting-started.md?tabs=angular)、[React](getting-started.md?tabs=react)、[Svelte](https://docs.microsoft.com/learn/modules/publish-app-service-static-web-app-api/)、[Vue](getting-started.md?tabs=react) 等 JavaScript 框架和库构建新式 Web 应用程序，或使用带 [Azure Functions](apis.md) 后端的 [Blazor](https://dotnet.microsoft.com/apps/aspnet/web-apps/blazor) 创建 WebAssembly 应用程序。
- 使用 [Gatsby](publish-gatsby.md)、[Hugo](publish-hugo.md)、[VuePress](publish-vuepress.md) 等框架发布静态站点。
- 使用 [Next.js](deploy-nextjs.md) 和 [Nuxt.js](deploy-nuxtjs.md) 等框架部署 Web 应用程序。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [构建第一个静态应用](getting-started.md)

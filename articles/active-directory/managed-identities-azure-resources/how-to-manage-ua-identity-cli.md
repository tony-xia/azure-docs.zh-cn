---
title: 管理用户分配的托管标识 - Azure CLI - Azure AD
description: 分步说明如何使用 Azure CLI 创建、列出和删除用户分配的托管标识。
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/17/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.custom: devx-track-azurecli
ms.openlocfilehash: 29a1a991ab79c38dad1a89533091d80406615d35
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/22/2020
ms.locfileid: "90969471"
---
# <a name="create-list-or-delete-a-user-assigned-managed-identity-using-the-azure-cli"></a>使用 Azure CLI 创建、列出或删除用户分配的托管标识


Azure 资源的托管标识在 Azure Active Directory 中为 Azure 服务提供了一个托管标识。 此标识可用于向支持 Azure AD 身份验证的服务进行身份验证，这样就无需在代码中输入凭据了。 

在本文中，将了解如何使用 Azure CLI 创建、列出和删除用户分配的托管标识。

## <a name="prerequisites"></a>先决条件

- 如果不熟悉 Azure 资源的托管标识，请查阅[概述部分](overview.md)。 请务必了解[系统分配的托管标识与用户分配的托管标识之间的差异](overview.md#managed-identity-types)。
- 如果没有 Azure 帐户，请在继续前[注册免费帐户](https://azure.microsoft.com/free/)。
- 若要运行示例脚本，有两个选项：
    - 使用 " [Azure Cloud Shell](../../cloud-shell/overview.md)"，可以使用代码块右上角的 " **试用** " 按钮打开。
    - 通过安装最新版本的 [Azure CLI](/cli/azure/install-azure-cli)在本地运行脚本，并使用 [az Login](/cli/azure/reference-index#az-login)登录到 Azure。 使用与要在其中创建资源的 Azure 订阅关联的帐户。   

> [!NOTE]
> 为了在使用 CLI 时修改用户权限，你必须在 Azure AD 图形 API 中提供服务主体的其他权限，因为 CLI 的某些部分对图形 API 执行 GET 请求。 否则，可能会收到 "权限不足，无法完成操作" 消息。 为此，你需要进入 Azure Active Directory 中的“应用注册”，选择你的应用，单击“API 权限”，向下滚动并选择“Azure Active Directory Graph”。 从那里选择“应用程序权限”，然后添加适当的权限。 

## <a name="create-a-user-assigned-managed-identity"></a>创建用户分配的托管标识 

若要创建用户分配的托管标识，你的帐户需要[托管标识参与者](../../role-based-access-control/built-in-roles.md#managed-identity-contributor)角色分配。

使用 [az identity create](/cli/azure/identity#az-identity-create) 命令创建用户分配的托管标识。 `-g` 参数指定要从中创建用户分配的托管标识的资源组，`-n` 参数指定其名称。 将 `<RESOURCE GROUP>` 和 `<USER ASSIGNED IDENTITY NAME>` 参数值替换为自己的值：

[!INCLUDE [ua-character-limit](~/includes/managed-identity-ua-character-limits.md)]

```azurecli-interactive
az identity create -g <RESOURCE GROUP> -n <USER ASSIGNED IDENTITY NAME>
```
## <a name="list-user-assigned-managed-identities"></a>列出用户分配的托管标识

若要列出/读取用户分配的托管标识，你的帐户需要[托管标识操作员](../../role-based-access-control/built-in-roles.md#managed-identity-operator)或[托管标识参与者](../../role-based-access-control/built-in-roles.md#managed-identity-contributor)角色分配。

若要列出用户分配的托管标识，请使用 [az identity list](/cli/azure/identity#az-identity-list) 命令。 将 `<RESOURCE GROUP>` 替换为自己的值：

```azurecli-interactive
az identity list -g <RESOURCE GROUP>
```

在 json 响应中，用户分配的托管标识为 `type` 键返回 `"Microsoft.ManagedIdentity/userAssignedIdentities"` 值。

`"type": "Microsoft.ManagedIdentity/userAssignedIdentities"`

## <a name="delete-a-user-assigned-managed-identity"></a>删除用户分配的托管标识

若要删除用户分配的托管标识，你的帐户需要[托管标识参与者](../../role-based-access-control/built-in-roles.md#managed-identity-contributor)角色分配。

若要删除用户分配的托管标识，请使用 [az identity delete](/cli/azure/identity#az-identity-delete) 命令。  -n 参数指定其名称，-g 参数指定创建了用户分配的托管标识的资源组。 将 `<USER ASSIGNED IDENTITY NAME>` 和 `<RESOURCE GROUP>` 参数值替换为自己的值：

```azurecli-interactive
az identity delete -n <USER ASSIGNED IDENTITY NAME> -g <RESOURCE GROUP>
```
> [!NOTE]
> 删除用户分配的托管标识不会从将其分配到的任何资源中删除引用。 请使用 `az vm/vmss identity remove` 命令从 VM/VMSS 中移除这些引用

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 标识命令的完整列表，请参阅 [az identity](/cli/azure/identity)。

有关如何向 Azure VM 分配用户分配的托管标识的信息，请参阅[使用 Azure CLI 在 Azure VM 上配置 Azure 资源的托管标识](qs-configure-cli-windows-vm.md#user-assigned-managed-identity)

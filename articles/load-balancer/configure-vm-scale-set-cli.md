---
title: 配置包含现有 Azure 负载均衡器的虚拟机规模集 - Azure CLI
description: 了解如何使用 Azure CLI 通过现有的 Azure 负载均衡器来配置虚拟机规模集。
author: asudbring
ms.author: allensu
ms.service: load-balancer
ms.topic: how-to
ms.date: 03/25/2020
ms.openlocfilehash: 990380d553cc12d1a87b2e1c7ca9b09864801294
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91333971"
---
# <a name="configure-a-virtual-machine-scale-set-with-an-existing-azure-load-balancer-using-the-azure-cli"></a>使用 Azure CLI 配置包含现有 Azure 负载均衡器的虚拟机规模集

本文介绍如何配置包含现有 Azure 负载均衡器的虚拟机规模集。 

## <a name="prerequisites"></a>必备条件

- Azure 订阅。
- 要将虚拟机规模集部署到的订阅中的现有标准 SKU 负载均衡器。
- 虚拟机规模集的 Azure 虚拟网络。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)] 

如果选择在本地使用 CLI，则本文要求安装 Azure CLI 版本的2.0.28 或更高版本。 要查找版本，请运行 `az --version`。 如果需要进行安装或升级，请参阅[安装 Azure CLI 2.0]( /cli/azure/install-azure-cli)。

## <a name="sign-in-to-azure-cli"></a>登录 Azure CLI

登录到 Azure。

```azurecli-interactive
az login
```

## <a name="deploy-a-virtual-machine-scale-set-with-existing-load-balancer"></a>部署包含现有负载均衡器的虚拟机规模集

请将括号中的值替换为配置中的资源名称。

```azurecli-interactive
az vmss create \
    --resource-group <resource-group> \
    --name <vmss-name>\
    --image <your-image> \
    --admin-username <admin-username> \
    --generate-ssh-keys  \
    --upgrade-policy-mode Automatic \
    --instance-count 3 \
    --vnet-name <virtual-network-name> \
    --subnet <subnet-name> \
    --lb <load-balancer-name> \
    --backend-pool-name <backend-pool-name>
```

以下示例部署一个虚拟机规模集：

- 该虚拟机规模集名为 myVMSS 
- 其中包含名为 myLoadBalancer 的 Azure 负载均衡器 
- 其中包含名为 myBackendPool 的负载均衡器后端池 
- 其中包含名为 myVnet 的 Azure 虚拟网络 
- 其中包含名为 mySubnet 的子网 
- 其中包含名为 myResourceGroup 的资源组 
- 虚拟机规模集的 Ubuntu 服务器映像

```azurecli-interactive
az vmss create \
    --resource-group myResourceGroup \
    --name myVMSS \
    --image Canonical:UbuntuServer:18.04-LTS:latest \
    --admin-username adminuser \
    --generate-ssh-keys \
    --upgrade-policy-mode Automatic \
    --instance-count 3 \
    --vnet-name myVnet\
    --subnet mySubnet \
    --lb myLoadBalancer \
    --backend-pool-name myBackendPool
```
> [!NOTE]
> 创建规模集后，无法为负载均衡器的运行状况探测所用的负载均衡规则修改后端端口。 为了更改端口，可以通过更新 Azure 虚拟机规模集来删除运行状况探测，更新端口，然后重新配置运行状况探测。

## <a name="next-steps"></a>后续步骤

在本文中，你已部署一个包含现有 Azure 负载均衡器的虚拟机规模集。  若要详细了解虚拟机规模集和负载均衡器，请参阅：

- [什么是 Azure 负载均衡器？](load-balancer-overview.md)
- [什么是虚拟机规模集？](../virtual-machine-scale-sets/overview.md)
                                
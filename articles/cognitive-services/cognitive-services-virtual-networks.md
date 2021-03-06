---
title: 虚拟网络
titleSuffix: Azure Cognitive Services
description: 为认知服务资源配置分层网络安全。
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: conceptual
ms.date: 10/07/2020
ms.author: aahi
ms.openlocfilehash: d320fcd0b7f9666da39dd1208efd9cdec04ad6b5
ms.sourcegitcommit: b87c7796c66ded500df42f707bdccf468519943c
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/08/2020
ms.locfileid: "91843135"
---
# <a name="configure-azure-cognitive-services-virtual-networks"></a>配置 Azure 认知服务虚拟网络

Azure 认知服务提供分层的安全模型。 借助此模型，可保护认知服务帐户，使其可供网络的特定子集访问。 配置网络规则后，仅通过指定网络集请求数据的应用程序才能访问帐户。 可以使用请求筛选来限制对资源的访问。 仅允许来自特定 IP 地址、IP 范围或 [Azure 虚拟网络](../virtual-network/virtual-networks-overview.md)中的子网列表的请求。

当网络规则生效时访问认知服务资源的应用程序需要授权。 [Azure Active Directory](../active-directory/fundamentals/active-directory-whatis.md) (Azure AD) 凭据或使用有效的 API 密钥，则支持授权。

> [!IMPORTANT]
> 默认情况下，打开认知服务帐户的防火墙规则会阻止传入的数据请求。 为了允许请求，需要满足以下条件之一：

> * 该请求应源自在目标认知服务帐户的 "允许的子网" 列表中 (VNet) 在 Azure 虚拟网络中运行的服务。 请求中源自 VNet 的终结点需要设置为认知服务帐户的 [自定义子域](cognitive-services-custom-subdomains.md) 。
> * 或请求应源自允许的 IP 地址列表。
>
> 被阻止的请求包括来自其他 Azure 服务、来自 Azure 门户、来自日志记录和指标服务等的请求。

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="scenarios"></a>方案

若要保护认知服务资源，你应该首先配置一个规则，拒绝从所有网络访问流量 (包括默认情况下) internet 流量。 然后，应配置允许访问特定 vnet 流量的规则。 借助此配置，可为应用程序生成安全网络边界。 还可以配置规则以授予对来自所选公共 internet IP 地址范围的流量的访问权限，从而支持来自特定 internet 或本地客户端的连接。

在所有网络协议中对 Azure 认知服务（包括 REST 和 WebSocket）强制实施网络规则。 若要使用 Azure 测试控制台等工具访问数据，必须配置显式网络规则。 您可以将网络规则应用于现有认知服务资源，或者在创建新认知服务资源时应用。 一旦应用网络规则，就会对所有请求强制实施这些规则。

## <a name="supported-regions-and-service-offerings"></a>支持的区域和服务产品

支持 [认知服务的区域](https://azure.microsoft.com/global-infrastructure/services/)支持虚拟网络 (vnet) 。 认知服务支持网络规则配置服务标记。 下面列出的服务包含在 **CognitiveServicesManagement** 服务标记中。

> [!div class="checklist"]
> * 异常检测器
> * 计算机视觉
> * 内容审查器
> * 自定义视觉
> * 人脸
> * 表单识别器
> * 语言理解 (LUIS)
> * 个性化体验创建服务
> * 文本分析
> * QnA Maker
> * 文本翻译
> * 沉浸式阅读器

> [!NOTE]
> 如果使用的是 LUIS，则 **CognitiveServicesManagement** 标记只允许使用 SDK 或 REST API 来使用该服务。 若要从虚拟网络访问和使用 LUIS 门户，你将需要使用以下标记：  
> * **AzureResourceManager** 
> * **CognitiveServicesManagement**
> * **AzureActiveDirectory**
> * **AzureFrontDoor.Frontend**


## <a name="change-the-default-network-access-rule"></a>更改默认网络访问规则

默认情况下，认知服务资源接受来自任何网络上的客户端的连接。 若要限制为仅允许选定网络访问，必须先更改默认操作。

> [!WARNING]
> 对网络规则进行更改可能会影响你的应用程序连接到 Azure 认知服务的能力。 除非还应用了**授予**访问权限的特定网络规则，否则将默认网络规则设置为“拒绝”会阻止对数据的所有访问。 在将默认规则更改为拒绝访问之前，务必先使用网络规则对所有许可网络授予访问权限。 如果允许列出本地网络的 IP 地址，请确保从本地网络添加所有可能的传出公共 IP 地址。

### <a name="managing-default-network-access-rules"></a>管理默认网络访问规则

可以通过 Azure 门户、PowerShell 或 Azure CLI 管理认知服务资源的默认网络访问规则。

# <a name="azure-portal"></a>[Azure 门户](#tab/portal)

1. 请参阅要保护的认知服务资源。

1. 选择名为 "**虚拟网络**" 的 "**资源管理**" 菜单。

   ![虚拟网络选项](media/vnet/virtual-network-blade.png)

1. 若要默认拒绝访问，请选择允许从“所选网络”进行访问。 如果选择 "仅 **选择的网络** " 设置，则通过配置的 **虚拟网络** 或 **地址范围** 进行 unaccompanied-所有访问都被拒绝。 拒绝所有访问时，不允许尝试使用认知服务资源的请求。 仍可以使用 Azure 门户、Azure PowerShell 或 Azure CLI 来配置认知服务资源。
1. 若要允许来自所有网络的流量，请选择允许从“所有网络”进行访问。

   ![虚拟网络拒绝](media/vnet/virtual-network-deny.png)

1. 单击“保存”应用所做的更改。

# <a name="powershell"></a>[PowerShell](#tab/powershell)

1. 安装 [Azure PowerShell](/powershell/azure/install-az-ps) 并 [登录](/powershell/azure/authenticate-azureps)，或选择 " **试用**"。

1. 显示认知服务资源的默认规则状态。

    ```azurepowershell-interactive
    $parameters = @{
        -ResourceGroupName "myresourcegroup"
        -Name "myaccount"
    }
    (Get-AzCognitiveServicesAccountNetworkRuleSet @parameters).DefaultAction
    ```

1. 将默认规则设置为默认拒绝网络访问。

    ```azurepowershell-interactive
    $parameters = @{
        -ResourceGroupName "myresourcegroup"
        -Name "myaccount"
        -DefaultAction Deny
    }
    Update-AzCognitiveServicesAccountNetworkRuleSet @parameters
    ```

1. 将默认规则设置为默认允许网络访问。

    ```azurepowershell-interactive
    $parameters = @{
        -ResourceGroupName "myresourcegroup"
        -Name "myaccount"
        -DefaultAction Allow
    }
    Update-AzCognitiveServicesAccountNetworkRuleSet @parameters
    ```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. 安装 [Azure CLI](/cli/azure/install-azure-cli) 并 [登录](/cli/azure/authenticate-azure-cli)，或选择 " **试用**"。

1. 显示认知服务资源的默认规则状态。

    ```azurecli-interactive
    az cognitiveservices account show \
        -g "myresourcegroup" -n "myaccount" \
        --query networkRuleSet.defaultAction
    ```

1. 将默认规则设置为默认拒绝网络访问。

    ```azurecli-interactive
    az cognitiveservices account update \
        -g "myresourcegroup" -n "myaccount" \
        --default-action Deny
    ```

1. 将默认规则设置为默认允许网络访问。

    ```azurecli-interactive
    az cognitiveservices account update \
        -g "myresourcegroup" -n "myaccount" \
        --default-action Allow
    ```

***

## <a name="grant-access-from-a-virtual-network"></a>允许从虚拟网络进行访问

你可以将认知服务资源配置为仅允许来自特定子网的访问。 允许的子网可能属于同一订阅或不同订阅中的 VNet，包括属于不同 Azure Active Directory 租户的订阅。

为 VNet 中的 Azure 认知服务启用 [服务终结点](../virtual-network/virtual-network-service-endpoints-overview.md) 。 服务终结点将流量从 VNet 路由到 Azure 认知服务服务的最佳路径。 子网和虚拟网络的标识也随每个请求进行传输。 然后，管理员可以为认知服务资源配置网络规则，以允许从 VNet 中的特定子网接收请求。 通过这些网络规则授予访问权限的客户端必须继续满足认知服务资源访问数据的授权要求。

每个认知服务资源最多支持100个虚拟网络规则，这些规则可以与 [IP 网络规则](#grant-access-from-an-internet-ip-range)结合。

### <a name="required-permissions"></a>所需的权限

若要将虚拟网络规则应用到认知服务资源，用户必须对所添加的子网具有适当的权限。 所需的权限是默认 *参与者* 角色或 *认知服务参与者* 角色。 还可以将所需权限添加到自定义角色定义。

认知服务资源和授予访问权限的虚拟网络可能位于不同的订阅中，其中包括属于不同 Azure AD 租户的订阅。

> [!NOTE]
> 目前，仅支持通过 Powershell、CLI 和 REST API 来配置对属于不同 Azure Active Directory 租户的虚拟网络中的子网授予访问权限的规则。 无法通过 Azure 门户配置此类规则，但可以在门户中查看此类规则。

### <a name="managing-virtual-network-rules"></a>管理虚拟网络规则

可以通过 Azure 门户、PowerShell 或 Azure CLI 管理认知服务资源的虚拟网络规则。

# <a name="azure-portal"></a>[Azure 门户](#tab/portal)

1. 请参阅要保护的认知服务资源。

1. 选择名为 "**虚拟网络**" 的 "**资源管理**" 菜单。

1. 检查并确保已选择允许从“所选网络”进行访问。

1. 若要使用现有网络规则授予对虚拟网络的访问权限，请在 " **虚拟网络**" 下选择 " **添加现有虚拟网络**"。

   ![添加现有 vNet](media/vnet/virtual-network-add-existing.png)

1. 选择 " **虚拟网络** " 和 " **子网** " 选项，然后选择 " **启用**"。

   ![添加现有 vNet 详细信息](media/vnet/virtual-network-add-existing-details.png)

1. 若要创建新的虚拟网络并授予其访问权限，请选择 " **添加新的虚拟网络**"。

   ![添加新 vNet](media/vnet/virtual-network-add-new.png)

1. 提供创建新虚拟网络所需的信息，然后选择 " **创建**"。

   ![创建 vNet](media/vnet/virtual-network-create.png)

    > [!NOTE]
    > 如果先前未为所选的虚拟网络和子网配置 Azure 认知服务的服务终结点，则可以将其配置为此操作的一部分。
    >
    > 目前，在创建规则期间，只会显示属于同一 Azure Active Directory 租户的虚拟网络供用户选择。 若要对属于其他租户的虚拟网络中的子网授予访问权限，请使用 Powershell、CLI 或 REST API。

1. 若要删除虚拟网络或子网规则，请选择 " **...** " 以打开虚拟网络或子网的上下文菜单，然后选择 " **删除**"。

   ![删除 vNet](media/vnet/virtual-network-remove.png)

1. 单击“保存”应用所做的更改。

# <a name="powershell"></a>[PowerShell](#tab/powershell)

1. 安装 [Azure PowerShell](/powershell/azure/install-az-ps) 并 [登录](/powershell/azure/authenticate-azureps)，或选择 " **试用**"。

1. 列出虚拟网络规则。

    ```azurepowershell-interactive
    $parameters = @{
        -ResourceGroupName "myresourcegroup"
        -Name "myaccount"
    }
    (Get-AzCognitiveServicesAccountNetworkRuleSet @parameters).VirtualNetworkRules
    ```

1. 为现有虚拟网络和子网上的 Azure 认知服务启用服务终结点。

    ```azurepowershell-interactive
    Get-AzVirtualNetwork -ResourceGroupName "myresourcegroup" `
        -Name "myvnet" | Set-AzVirtualNetworkSubnetConfig -Name "mysubnet" `
        -AddressPrefix "10.0.0.0/24" `
        -ServiceEndpoint "Microsoft.CognitiveServices" | Set-AzVirtualNetwork
    ```

1. 为虚拟网络和子网添加网络规则。

    ```azurepowershell-interactive
    $subParameters = @{
        -ResourceGroupName "myresourcegroup"
        -Name "myvnet"
    }
    $subnet = Get-AzVirtualNetwork @subParameters | Get-AzVirtualNetworkSubnetConfig -Name "mysubnet"

    $parameters = @{
        -ResourceGroupName "myresourcegroup"
        -Name "myaccount"
        -VirtualNetworkResourceId $subnet.Id
    }
    Add-AzCognitiveServicesAccountNetworkRule @parameters
    ```

    > [!TIP]
    > 若要为属于其他 Azure AD 租户的 VNet 中的子网添加网络规则，请使用“/subscriptions/subscription-ID/resourceGroups/resourceGroup-Name/providers/Microsoft.Network/virtualNetworks/vNet-name/subnets/subnet-name”格式的完全限定的 **VirtualNetworkResourceId** 参数。

1. 为虚拟网络和子网删除网络规则。

    ```azurepowershell-interactive
    $subParameters = @{
        -ResourceGroupName "myresourcegroup"
        -Name "myvnet"
    }
    $subnet = Get-AzVirtualNetwork @subParameters | Get-AzVirtualNetworkSubnetConfig -Name "mysubnet"

    $parameters = @{
        -ResourceGroupName "myresourcegroup"
        -Name "myaccount"
        -VirtualNetworkResourceId $subnet.Id
    }
    Remove-AzCognitiveServicesAccountNetworkRule @parameters
    ```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. 安装 [Azure CLI](/cli/azure/install-azure-cli) 并 [登录](/cli/azure/authenticate-azure-cli)，或选择 " **试用**"。

1. 列出虚拟网络规则。

    ```azurecli-interactive
    az cognitiveservices account network-rule list \
        -g "myresourcegroup" -n "myaccount" \
        --query virtualNetworkRules
    ```

1. 为现有虚拟网络和子网上的 Azure 认知服务启用服务终结点。

    ```azurecli-interactive
    az network vnet subnet update -g "myresourcegroup" -n "mysubnet" \
    --vnet-name "myvnet" --service-endpoints "Microsoft.CognitiveServices"
    ```

1. 为虚拟网络和子网添加网络规则。

    ```azurecli-interactive
    $subnetid=(az network vnet subnet show \
        -g "myresourcegroup" -n "mysubnet" --vnet-name "myvnet" \
        --query id --output tsv)

    # Use the captured subnet identifier as an argument to the network rule addition
    az cognitiveservices account network-rule add \
        -g "myresourcegroup" -n "myaccount" \
        --subnet $subnetid
    ```

    > [!TIP]
    > 若要为属于其他 Azure AD 租户的 VNet 中的子网添加规则，请使用“/subscriptions/subscription-ID/resourceGroups/resourceGroup-Name/providers/Microsoft.Network/virtualNetworks/vNet-name/subnets/subnet-name”格式的完全限定的子网 ID。
    > 
    > 可以使用 **subscription** 参数检索属于其他 Azure AD 租户的 VNet 的子网 ID。

1. 为虚拟网络和子网删除网络规则。

    ```azurecli-interactive
    $subnetid=(az network vnet subnet show \
        -g "myresourcegroup" -n "mysubnet" --vnet-name "myvnet" \
        --query id --output tsv)

    # Use the captured subnet identifier as an argument to the network rule removal
    az cognitiveservices account network-rule remove \
        -g "myresourcegroup" -n "myaccount" \
        --subnet $subnetid
    ```

***

> [!IMPORTANT]
> 请务必[将默认规则设置](#change-the-default-network-access-rule)为“拒绝”，否则网络规则不会有任何效果。

## <a name="grant-access-from-an-internet-ip-range"></a>允许从 Internet IP 范围进行访问

你可以将认知服务资源配置为允许来自特定公共 internet IP 地址范围的访问。 此配置授予对特定服务和本地网络的访问权限，从而有效地阻止一般的 internet 流量。

使用 [CIDR 表示法](https://tools.ietf.org/html/rfc4632) `16.17.18.0/24` 或以单个 IP 地址（如）提供允许的 internet 地址范围 `16.17.18.19` 。

   > [!Tip]
   > 不支持使用“/31”或“/32”前缀大小的小型地址范围。 这些范围应使用单独的 IP 地址规则配置。

IP 网络规则仅适用于**公共 Internet** IP 地址。 IP 规则不允许使用为专用网络保留的 IP 地址范围（如 [RFC 1918](https://tools.ietf.org/html/rfc1918#section-3) 中所定义）。 专用网络包括以、和开头的地址 `10.*` `172.16.*`  -  `172.31.*` `192.168.*` 。

   > [!NOTE]
   > IP 网络规则不会影响来自与认知服务资源位于同一 Azure 区域的请求。 请使用[虚拟网络规则](#grant-access-from-a-virtual-network)来允许相同区域的请求。

目前仅支持 IPV4 地址。 每个认知服务资源最多支持100个 IP 网络规则，这些规则可与 [虚拟网络规则](#grant-access-from-a-virtual-network)结合。

### <a name="configuring-access-from-on-premises-networks"></a>配置从本地网络的访问

若要使用 IP 网络规则将本地网络中的访问权限授予认知服务资源，必须确定网络使用的面向 internet 的 IP 地址。 若要获得帮助，请联系网络管理员。

如果要将 [ExpressRoute](../expressroute/expressroute-introduction.md) 本地用于公共对等互连或 Microsoft 对等互连，则需要确定 NAT IP 地址。 对于公共对等互连，默认情况下，每条 ExpressRoute 线路都使用两个 NAT IP 地址。 当流量进入 Microsoft Azure 网络主干时，每个都应用于 Azure 服务流量。 对于 Microsoft 对等互连，使用的 NAT IP 地址是客户提供的或由服务提供商提供的。 若要允许访问服务资源，必须在资源 IP 防火墙设置中允许这些公共 IP 地址。 若要查找公共对等互连 ExpressRoute 线路 IP 地址，请通过 Azure 门户[开具 ExpressRoute 支持票证](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview)。 详细了解[适用于 ExpressRoute 公共对等互连和 Microsoft 对等互连的 NAT](../expressroute/expressroute-nat.md#nat-requirements-for-azure-public-peering)。

### <a name="managing-ip-network-rules"></a>管理 IP 网络规则

可以通过 Azure 门户、PowerShell 或 Azure CLI 管理认知服务资源的 IP 网络规则。

# <a name="azure-portal"></a>[Azure 门户](#tab/portal)

1. 请参阅要保护的认知服务资源。

1. 选择名为 "**虚拟网络**" 的 "**资源管理**" 菜单。

1. 检查并确保已选择允许从“所选网络”进行访问。

1. 若要授予对 internet IP 范围的访问权限，请在 "**防火墙**地址范围" 下输入[CIDR) 格式](https://tools.ietf.org/html/rfc4632)的 IP 地址或地址范围 ( >  **Address Range**。 只接受有效的公共 IP (非保留的) 地址。

   ![添加 IP 范围](media/vnet/virtual-network-add-ip-range.png)

1. 若要删除 IP 网络规则，请选择地址范围旁的 "垃圾桶" <span class="docon docon-delete x-hidden-focus"></span> 图标。

   ![删除 IP 范围](media/vnet/virtual-network-delete-ip-range.png)

1. 单击“保存”应用所做的更改。

# <a name="powershell"></a>[PowerShell](#tab/powershell)

1. 安装 [Azure PowerShell](/powershell/azure/install-az-ps) 并 [登录](/powershell/azure/authenticate-azureps)，或选择 " **试用**"。

1. 列出 IP 网络规则。

    ```azurepowershell-interactive
    $parameters = @{
        -ResourceGroupName "myresourcegroup"
        -Name "myaccount"
    }
    (Get-AzCognitiveServicesAccountNetworkRuleSet @parameters).IPRules
    ```

1. 为单个 IP 地址添加网络规则。

    ```azurepowershell-interactive
    $parameters = @{
        -ResourceGroupName "myresourcegroup"
        -Name "myaccount"
        -IPAddressOrRange "16.17.18.19"
    }
    Add-AzCognitiveServicesAccountNetworkRule @parameters
    ```

1. 为 IP 地址范围添加网络规则。

    ```azurepowershell-interactive
    $parameters = @{
        -ResourceGroupName "myresourcegroup"
        -Name "myaccount"
        -IPAddressOrRange "16.17.18.0/24"
    }
    Add-AzCognitiveServicesAccountNetworkRule @parameters
    ```

1. 为单个 IP 地址删除网络规则。

    ```azurepowershell-interactive
    $parameters = @{
        -ResourceGroupName "myresourcegroup"
        -Name "myaccount"
        -IPAddressOrRange "16.17.18.19"
    }
    Remove-AzCognitiveServicesAccountNetworkRule @parameters
    ```

1. 为 IP 地址范围删除网络规则。

    ```azurepowershell-interactive
    $parameters = @{
        -ResourceGroupName "myresourcegroup"
        -Name "myaccount"
        -IPAddressOrRange "16.17.18.0/24"
    }
    Remove-AzCognitiveServicesAccountNetworkRule @parameters
    ```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. 安装 [Azure CLI](/cli/azure/install-azure-cli) 并 [登录](/cli/azure/authenticate-azure-cli)，或选择 " **试用**"。

1. 列出 IP 网络规则。

    ```azurecli-interactive
    az cognitiveservices account network-rule list \
        -g "myresourcegroup" -n "myaccount" --query ipRules
    ```

1. 为单个 IP 地址添加网络规则。

    ```azurecli-interactive
    az cognitiveservices account network-rule add \
        -g "myresourcegroup" -n "myaccount" \
        --ip-address "16.17.18.19"
    ```

1. 为 IP 地址范围添加网络规则。

    ```azurecli-interactive
    az cognitiveservices account network-rule add \
        -g "myresourcegroup" -n "myaccount" \
        --ip-address "16.17.18.0/24"
    ```

1. 为单个 IP 地址删除网络规则。

    ```azurecli-interactive
    az cognitiveservices account network-rule remove \
        -g "myresourcegroup" -n "myaccount" \
        --ip-address "16.17.18.19"
    ```

1. 为 IP 地址范围删除网络规则。

    ```azurecli-interactive
    az cognitiveservices account network-rule remove \
        -g "myresourcegroup" -n "myaccount" \
        --ip-address "16.17.18.0/24"
    ```

***

> [!IMPORTANT]
> 请务必[将默认规则设置](#change-the-default-network-access-rule)为“拒绝”，否则网络规则不会有任何效果。

## <a name="use-private-endpoints"></a>使用专用终结点

你可以将 [专用终结点](../private-link/private-endpoint-overview.md) 用于认知服务资源，以允许虚拟网络上的客户端 (VNet) 通过 [专用链接](../private-link/private-link-overview.md)安全地访问数据。 专用终结点使用 VNet 地址空间中的 IP 地址来处理认知服务资源。 VNet 和资源上的客户端之间的网络流量通过 Microsoft 主干网络上的 VNet 和专用链接，消除了公共 internet 的泄露。

认知服务资源的专用终结点可让你：

* 将防火墙配置为阻止认知服务服务公共终结点上的所有连接，从而保护认知服务资源。
* 通过使你能够阻止渗透 VNet 中的数据，提高 VNet 的安全性。
* 使用 [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md) 或 [ExpressRoutes](../expressroute/expressroute-locations.md) 与专用对等互连，从连接到 VNet 的本地网络安全连接到认知服务资源。

### <a name="conceptual-overview"></a>概念概述

专用终结点是 [VNet](../virtual-network/virtual-networks-overview.md)中 Azure 资源的特殊网络接口。 为认知服务资源创建专用终结点可在 VNet 中的客户端和资源之间提供安全连接。 从 VNet 的 IP 地址范围为专用终结点分配 IP 地址。 专用终结点和认知服务服务之间的连接使用安全的专用链接。

VNet 中的应用程序可以使用相同的连接字符串和它们将使用的授权机制，以无缝方式连接到专用终结点上的服务。 语音服务例外，需要单独的终结点。 请参阅 [语音服务专用终结点](#private-endpoints-with-the-speech-service)上的部分。 专用终结点可用于认知服务资源支持的所有协议，包括 REST。

可以在使用 [服务终结点](../virtual-network/virtual-network-service-endpoints-overview.md)的子网中创建专用终结点。 子网中的客户端可以使用专用终结点连接到一个认知服务资源，同时使用服务终结点访问其他资源。

在 VNet 中创建认知服务资源的专用终结点时，会将同意请求发送到认知服务资源所有者。 如果请求创建专用终结点的用户也是资源的所有者，则此许可请求会自动获得批准。

认知服务资源所有者可以通过[Azure 门户](https://portal.azure.com)中认知服务资源的 "*专用终结点*" 选项卡来管理同意请求和专用终结点。

### <a name="private-endpoints"></a>专用终结点

创建专用终结点时，必须指定它所连接的认知服务资源。 有关创建专用终结点的详细信息，请参阅：

* [使用 Azure 门户中的“专用链接中心”创建专用终结点](../private-link/create-private-endpoint-portal.md)
* [使用 Azure CLI 创建专用终结点](../private-link/create-private-endpoint-cli.md)
* [使用 Azure PowerShell 创建专用终结点](../private-link/create-private-endpoint-powershell.md)

### <a name="connecting-to-private-endpoints"></a>连接到专用终结点

当客户端连接到公共终结点时，使用专用终结点的 VNet 中的客户端应使用与认知服务资源相同的连接字符串。 语音服务例外，需要单独的终结点。 请参阅 [语音服务专用终结点](#private-endpoints-with-the-speech-service)上的部分。 我们依赖 DNS 解析，通过专用链接自动将来自 VNet 的连接路由到认知服务资源。 语音服务 

默认情况下，我们创建附加到 VNet 的 [专用 DNS 区域](../dns/private-dns-overview.md) ，其中包含专用终结点的必要更新。 但是，如果使用自己的 DNS 服务器，则可能需要对 DNS 配置进行其他更改。 以下 [DNS 更改](#dns-changes-for-private-endpoints) 部分介绍了专用终结点所需的更新。

### <a name="private-endpoints-with-the-speech-service"></a>带有语音服务的专用终结点

使用带有语音服务的专用终结点时，必须使用自定义终结点来调用语音服务。 不能使用全局终结点。 终结点必须遵循此模式： `{account}.{stt|tts|voice|dls}.speech.microsoft.com` 。

### <a name="dns-changes-for-private-endpoints"></a>专用终结点的 DNS 更改

创建专用终结点时，会将认知服务资源的 DNS CNAME 资源记录更新为前缀为 "*privatelink*" 的子域中的别名。 默认情况下，我们还会创建一个 [专用 dns 区域](../dns/private-dns-overview.md)，该区域与 "*privatelink*" 子域相对应，其中 dns a 用于专用终结点的资源记录。

当你通过专用终结点将终结点 URL 从 VNet 外部解析时，它将解析为认知服务资源的公共终结点。 当从承载专用终结点的 VNet 解析时，终结点 URL 解析为专用终结点的 IP 地址。

此方法使用同一连接字符串为在 VNet 外部宿主专用终结点和客户端的 VNet 中的客户端启用对认知服务资源的访问。

如果在网络上使用自定义 DNS 服务器，则客户端必须能够将认知服务资源终结点的完全限定域名 (FQDN) 解析为专用终结点 IP 地址。 将 DNS 服务器配置为将专用链接子域委托给 VNet 的专用 DNS 区域。

> [!TIP]
> 使用自定义或本地 DNS 服务器时，应将 DNS 服务器配置为将 "privatelink" 子域中的认知服务资源名称解析到专用终结点 IP 地址。 为此，可以将 "privatelink" 子域委托给 VNet 的专用 DNS 区域，或在 DNS 服务器上配置 DNS 区域并添加 DNS A 记录。

有关配置自己的 DNS 服务器以支持专用终结点的详细信息，请参阅以下文章：

* [Azure 虚拟网络中资源的名称解析](https://docs.microsoft.com/azure/virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances#name-resolution-that-uses-your-own-dns-server)
* [专用终结点的 DNS 配置](https://docs.microsoft.com/azure/private-link/private-endpoint-overview#dns-configuration)

### <a name="pricing"></a>定价

有关定价详细信息，请参阅 [Azure 专用链接定价](https://azure.microsoft.com/pricing/details/private-link)。

## <a name="next-steps"></a>后续步骤

* 探索各种 [Azure 认知服务](welcome.md)
* 了解有关[Azure 虚拟网络服务终结点](../virtual-network/virtual-network-service-endpoints-overview.md)的详细信息

---
title: 通过 PowerShell 加入 Azure 安全中心
description: 本文档将指导你完成通过 PowerShell cmdlet 开始使用 Azure 安全中心的过程。
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: e400fcbf-f0a8-4e10-b571-5a0d0c3d0c67
ms.service: security-center
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/02/2018
ms.author: memildin
ms.openlocfilehash: c805b35b2c05600f96983da42ea7206a09e2e3e8
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/05/2020
ms.locfileid: "91447392"
---
# <a name="automate-onboarding-of-azure-security-center-using-powershell"></a>使用 PowerShell 自动载入 Azure 安全中心

通过 Azure 安全中心 PowerShell 模块，能够以编程方式保护 Azure 工作负载的安全。
借助 PowerShell，可以自动执行任务，并可避免手动任务中固有的人为错误。 对于涉及几十个资源且具有数百乃至数千种资源的大规模部署，这一方法尤为有用 – 所有这些必须从一开始就受到保护。

通过 PowerShell 使用 Azure 安全中心，使你能够以编程方式自动开始使用 Azure 资源并对其进行管理，以及添加必要的安全控件。

本文提供了一个示例 PowerShell 脚本，可在你的环境中进行修改和使用，以跨订阅推出安全中心。 

在此示例中，通过启用 Azure Defender，提供高级威胁防护和检测功能，我们将对 ID 为 d07c0080-170c-4c24-861d-9c817742786c 的订阅启用安全中心，并应用提供高级别保护的建议设置：

1. 启用 [Azure Defender](azure-defender.md)。 
 
2. 将 Log Analytics 工作区设置为 Log Analytics 代理将发送其在与订阅关联的 VM 上收集的数据位置，在此示例中，是现有用户定义的工作区 (myWorkspace)。

3. 激活[部署 Log Analytics 代理](security-center-enable-data-collection.md#auto-provision-mma)的安全中心的自动代理预配。

5. 将组织的 [CISO 设置为安全中心警报和重要事件的安全联系人](security-center-provide-security-contact-details.md)。

6. 分配安全中心的[默认安全策略](tutorial-security-policy.md)。

## <a name="prerequisites"></a>先决条件

这些步骤应在运行安全中心 cmdlet 前执行：

1. 以管理员身份运行 PowerShell。

1. 在 PowerShell 中运行以下命令：
      
    ```Set-ExecutionPolicy -ExecutionPolicy AllSigned```

    ```Install-Module -Name Az.Security -Force```

## <a name="onboard-security-center-using-powershell"></a>通过 PowerShell 开始使用安全中心

1. 将你的订阅注册到安全中心资源提供程序：

    ```Set-AzContext -Subscription "d07c0080-170c-4c24-861d-9c817742786c"```

    ```Register-AzResourceProvider -ProviderNamespace 'Microsoft.Security'```

1. 可选：设置订阅的覆盖级别（启用/关闭 Azure Defender）。 如果未定义，Defender 将处于关闭状态：

    ```Set-AzContext -Subscription "d07c0080-170c-4c24-861d-9c817742786c"```

    ```Set-AzSecurityPricing -Name "default" -PricingTier "Standard"```

1. 配置代理将报告的 Log Analytics 工作区。 必须具有一个已创建的 Log Analytics 工作区，订阅的虚拟机将向其报告。 你可以定义向同一工作区报告的多个订阅。 如未定义，则将使用默认工作区。

    ```Set-AzSecurityWorkspaceSetting -Name "default" -Scope "/subscriptions/d07c0080-170c-4c24-861d-9c817742786c" -WorkspaceId"/subscriptions/d07c0080-170c-4c24-861d-9c817742786c/resourceGroups/myRg/providers/Microsoft.OperationalInsights/workspaces/myWorkspace"```

1. 在 Azure VM 上自动预配安装 Log Analytics 代理：
    
    ```Set-AzContext -Subscription "d07c0080-170c-4c24-861d-9c817742786c"```
    
    ```Set-AzSecurityAutoProvisioningSetting -Name "default" -EnableAutoProvision```

    > [!NOTE]
    > 建议启用自动预配，以确保 Azure 虚拟机自动受到 Azure 安全中心的保护。
    >

1. 可选：强烈建议为你加入的订阅定义安全联系人详细信息，该信息将被用作接收安全中心所生成警报和通知的收件人：

    ```Set-AzSecurityContact -Name "default1" -Email "CISO@my-org.com" -Phone "2142754038" -AlertAdmin -NotifyOnAlert```

1. 分配默认安全中心策略计划：

    ```Register-AzResourceProvider -ProviderNamespace 'Microsoft.PolicyInsights'```

    ```$Policy = Get-AzPolicySetDefinition | where {$_.Properties.displayName -EQ 'Enable Monitoring in Azure Security Center'} New-AzPolicyAssignment -Name 'ASC Default <d07c0080-170c-4c24-861d-9c817742786c>' -DisplayName 'Security Center Default <subscription ID>' -PolicySetDefinition $Policy -Scope '/subscriptions/d07c0080-170c-4c24-861d-9c817742786c'```

你已使用 PowerShell 成功加入 Azure 安全中心。

现在可以将这些 PowerShell cmdlet 与自动化脚本结合使用，从而以编程方式循环访问订阅和资源。 这可节省时间并减少人为错误的可能性。 你可以使用此[示例脚本](https://github.com/Microsoft/Azure-Security-Center/blob/master/quickstarts/ASC-Samples.ps1)作为参考。




## <a name="see-also"></a>另请参阅
若要详细了解如何通过 PowerShell 来自动开始使用安全中心，请参阅以下文章：

* [Az.Security](https://docs.microsoft.com/powershell/module/az.security)

若要详细了解安全中心，请参阅以下文章：

* [在 Azure 安全中心中设置安全策略](tutorial-security-policy.md) - 了解如何配置 Azure 订阅和资源组的安全策略。
* [管理和响应 Azure 安全中心的安全警报](security-center-managing-and-responding-alerts.md) -- 了解如何管理和响应安全警报。
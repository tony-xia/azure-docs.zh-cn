---
title: 使用 Chocolatey 设置 Azure 自动化持续部署
description: 本文介绍如何使用 State Configuration 和 Chocolatey 包管理器设置持续部署。
services: automation
ms.subservice: dsc
ms.date: 08/08/2018
ms.topic: conceptual
ms.custom: references_regions
ms.openlocfilehash: 87504625c298c4fb858ff90430d707081e87cd5a
ms.sourcegitcommit: ec682dcc0a67eabe4bfe242fce4a7019f0a8c405
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/09/2020
ms.locfileid: "86186565"
---
# <a name="set-up-continuous-deployment-with-chocolatey"></a>使用 Chocolatey 设置持续部署

DevOps 领域中有许多工具可帮助你处理持续集成管道中的各个点。 Azure Automation [State Configuration](automation-dsc-overview.md) 是 DevOps 团队可以采用的新选项。 

Azure 自动化是 Microsoft Azure 中的托管服务，可让你使用 Runbook、节点和共享资源（例如凭据、计划和全局变量），将各种任务自动化。 Azure Automation State Configuration 扩展了此自动化功能，以包含 PowerShell Desired State Configuration (DSC) 工具。 下面是全面的[概述](automation-dsc-overview.md)。

本文演示如何为 Windows 计算机设置持续部署 (CD)。 可以轻松扩展该技术，在角色（例如网站）中按需要添加更多 Windows 计算机，并从该角色扩展到其他角色。

![IaaS VM 的持续部署](./media/automation-dsc-cd-chocolatey/cdforiaasvm.png)

## <a name="at-a-high-level"></a>概括而言

此处要处理的事项很多，幸好这些事项可划分成两个主要过程：

- 编写并测试代码，针对系统的主要和次要版本创建并发布安装包。
- 创建和管理用于安装和执行包中代码的 VM。  

完成这两个核心过程后，随着创建和部署新版本，自动更新 VM 上的包很简单。

## <a name="component-overview"></a>组件概述

[apt-get](https://en.wikipedia.org/wiki/Advanced_Packaging_Tool) 之类的包管理器在 Linux 领域耳熟能详，但在 Windows 领域并不被大家所熟悉。
[Chocolatey](https://chocolatey.org/) 就是这样一个工具，Scott Hanselman 有关该工具主题的[博客](https://www.hanselman.com/blog/IsTheWindowsUserReadyForAptget.aspx)对此工具进行了深入介绍。 简单地说，Chocolatey 可让你使用命令行从中央存储库将包安装到 Windows 操作系统。 可以创建和管理自己的存储库，Chocolatey 可以从指定的任何数量的存储库来安装包。

[PowerShell DSC](/powershell/scripting/dsc/overview/overview) 是一个 PowerShell 工具，可使用它为计算机声明所需的配置。 例如，如果你想要安装 Chocolatey、安装 IIS、打开端口 80 并安装网站版本 1.0.0，DSC 本地配置管理器 (LCM) 将实现该配置。 DSC 拉取服务器有一个存储库用于保存计算机的配置。 每台计算机上的 LCM 定期检查计算机的配置是否与存储的配置匹配。 它可以报告状态，也可以尝试让计算机恢复到与存储的配置匹配。 可以编辑“拉”服务器上存储的配置，使一台计算机或一组计算机与更改的配置匹配。

DSC 资源是具有特定功能的代码模块，例如管理网络、Active Directory 或 SQL Server。 Chocolatey DSC 资源知道如何访问 NuGet 服务器（以及其他组件）、下载包、安装包，等等。 [PowerShell 库](https://www.powershellgallery.com/packages?q=dsc+resources&prerelease=&sortOrder=package-title)中有许多其他 DSC 资源。 在 Azure Automation State Configuration 拉取服务器上安装这些模块以供配置使用。

Resource Manager 模板以声明方式生成基础结构，例如网络、子网、网络安全性和路由、负载均衡器、NIC、VM，等等。 这篇[文章](../azure-resource-manager/management/deployment-models.md)比较了资源管理器部署模型（声明性）和 Azure 服务管理（ASM 或经典）部署模型（命令性）。 本文讨论了核心资源提供程序：计算、存储和网络。

资源管理器模板的一项主要功能是在预配时将 VM 扩展安装到 VM 中。 VM 扩展模块具有特定功能，例如运行自定义脚本、安装防病毒软件或运行 DSC 配置脚本。 有许多其他类型的 VM 扩展。

## <a name="quick-trip-around-the-diagram"></a>示意图速览

首先，需要编写、生成和测试代码，然后创建安装包。 Chocolatey 可以处理各种类型的安装包，例如 MSI、MSU、ZIP。 如果 Chocolatey 的本机功能不足以满足需要，还有 PowerShell 的完整功能可执行实际安装。 将包放入可访问的位置 - 包存储库。 本用例使用 Azure Blob 存储帐户中的公共文件夹，但它可以位于任何位置。 Chocolatey 原生可配合 NuGet 服务器和其他某些工具一起管理包元数据。 [本文](https://github.com/chocolatey/choco/wiki/How-To-Host-Feed)介绍了相应的选项。 本用例使用 NuGet。 Nuspec 是包的元数据。 Nuspec 信息编译到 NuPkg 中，并存储在 NuGet 服务器上。 当配置按名称请求某个包并引用 NuGet 服务器时，VM 中的 Chocolatey DSC 资源将获取并安装该包。 也可以请求特定版本的包。

示意图左下方有一个 Azure 资源管理器模板。 在本用例中，VM 扩展将 VM 注册到 Azure Automation State Configuration 拉取服务器成为一个节点。 此配置存储在拉取服务器中两次：一次以纯文本存储，一次编译为 MOF 文件。 在 Azure 门户中，MOF 代表节点配置，而不只是简单配置。 它是与节点关联的项目，因而节点知道它的配置。 以下详细信息演示如何将节点配置分配给节点。

创建和编译 Nuspec 并将其存储在 NuGet 服务器非常简单。 此外，你已在管理 VM。 

持续部署的下一步需要设置拉取服务器一次、向它注册节点一次，然后创建初始配置并存储到服务器中。 当包升级并部署到存储库时，仅需根据需要刷新拉取服务器中的配置和节点配置。

如果不是从资源管理器模板开始，也没关系。 有一些 PowerShell 命令可帮助你向拉取服务器注册 VM。 有关详细信息，请参阅[登记由 Azure Automation State Configuration 管理的计算机](automation-dsc-onboarding.md)。

## <a name="about-the-usage-example"></a>关于使用示例

本文中的用例是从一个来自于 Azure 库的通用 Windows Server 2012 R2 映像的 VM 开始的。 可以从任何存储的映像开始，并使用 DSC 配置对其进行调整。
不过，更改已刻入映像的配置要比使用 DSC 动态更新配置难得多。

将此技巧运用于 VM 时，不需要使用资源管理器模板和 VM 扩展。 即使 VM 不在 Azure 上，也能由 CD 管理。 只需在 VM 上安装 Chocolatey 并配置 LCM，以使其知道“拉”服务器的所在位置即可。

在生产环境中的 VM 上更新包时，在安装更新的过程中，需要将 VM 从轮转列表中排除。 具体的操作根据情况而有很大的差异。 例如，如果 VM 在 Azure 负载均衡器后面，则可以添加自定义探测。 更新 VM 时，让探测终结点返回 400。 可在配置中进行所需的调整来造成这种更改，但更新完成时，调整将切换为返回 200。

GitHub 上的[此 Visual Studio 项目](https://github.com/sebastus/ARM/tree/master/CDIaaSVM)中提供了本用例的完整源代码。

## <a name="step-1-set-up-the-pull-server-and-automation-account"></a>步骤 1：设置拉取服务器和自动化帐户

在经过身份验证的 (`Connect-AzAccount`) PowerShell 命令行中：（如果设置请求服务器，则可能需要几分钟时间）

```azurepowershell-interactive
New-AzResourceGroup –Name MY-AUTOMATION-RG –Location MY-RG-LOCATION-IN-QUOTES
New-AzAutomationAccount –ResourceGroupName MY-AUTOMATION-RG –Location MY-RG-LOCATION-IN-QUOTES –Name MY-AUTOMATION-ACCOUNT
```

可以将自动化帐户放入以下任何区域（也称为位置）：美国东部 2、美国中南部、US Gov 弗吉尼亚州、西欧、东南亚、日本东部、印度中部和澳大利亚东南部、加拿大中部、北欧。

## <a name="step-2-make-vm-extension-tweaks-to-the-resource-manager-template"></a>步骤 2：使 VM 扩展根据资源管理器模板进行调整

此 [Azure 快速入门模板](https://github.com/Azure/azure-quickstart-templates/tree/master/dsc-extension-azure-automation-pullserver)提供了 VM 注册（使用 PowerShell DSC VM 扩展）的详细信息。
此步骤将新的 VM 注册到“拉”服务器的 State Configuration 节点列表中。 此注册的一部分指定要应用到节点的节点配置。 此节点配置尚无需存在于拉取服务器中，因此第一次可以在步骤 4 中执行该操作。 但在步骤 2 中，需要确定节点名称和配置名称。 在本用例中，节点名称为“isvbox”，配置名称为“ISVBoxConfig”。 因此，节点配置名称（会在 DeploymentTemplate.json 中指定）为“ISVBoxConfig.isvbox”。

## <a name="step-3-add-required-dsc-resources-to-the-pull-server"></a>步骤 3：将所需的 DSC 资源添加到拉取服务器

PowerShell 库自动将 DSC 资源安装到 Azure 自动化帐户。
导航到所需的资源，并单击“部署到 Azure 自动化”按钮。

![PowerShell 库示例](./media/automation-dsc-cd-chocolatey/xNetworking.PNG)

Azure 门户最近添加的另一种技术允许提取新模块或更新现有模块。 依次单击“自动化帐户资源”、“资产”磁贴和“模块”磁贴。 通过“浏览库”图标可以查看库中的模块列表，向下钻取详细信息，并最终导入自动化帐户。 这是让模块随时保持最新状态的绝佳方法。 而且，导入功能会检查与其他模块的依赖性，以确保所有模块都保持同步。

另外还有一种手动方法，只对每个资源使用一次（除非以后再升级）。 有关创作 PowerShell 集成模块的详细信息，请参阅[为Azure自动化创作集成模块](https://azure.microsoft.com/blog/authoring-integration-modules-for-azure-automation/)。

>[!NOTE]
>适用于 Windows 计算机的 PowerShell 集成模块的文件夹结构与 Azure 自动化所需的文件夹结构稍有不同。 

1. 安装 [Windows Management Framework v5](https://aka.ms/wmf5latest)（对于 Windows 10 不需要安装）。

2. 安装集成模块。

    ```azurepowershell-interactive
    Install-Module –Name MODULE-NAME`    <—grabs the module from the PowerShell Gallery
    ```

3. 将模块文件夹从 c:\Program Files\WindowsPowerShell\Modules\MODULE-NAME 复制到临时文件夹中。

4. 删除主文件夹中的示例和文档。

5. 压缩主文件夹，ZIP 文件应使用该文件夹的名称命名。

6. 将 ZIP 文件放到可访问的 HTTP 位置，例如 Azure 存储帐户中的 Blob 存储。

7. 运行以下命令。

    ```azurepowershell-interactive
    New-AzAutomationModule `
      -ResourceGroupName MY-AUTOMATION-RG -AutomationAccountName MY-AUTOMATION-ACCOUNT `
      -Name MODULE-NAME –ContentLinkUri 'https://STORAGE-URI/CONTAINERNAME/MODULE-NAME.zip'
    ```

随附的示例针对 cChoco 和 xNetworking 实施了这些步骤。 

## <a name="step-4-add-the-node-configuration-to-the-pull-server"></a>步骤 4：将节点配置添加到拉取服务器

首次将配置导入到“拉”服务器并进行编译并没有什么特别之处。 后续导入或编译相同的配置时，过程完全相同。 每次更新包且需要向外推送到生产环境时，在确保配置文件（包括包的新版本）正确之后，就可以执行此步骤。 配置文件 ISVBoxConfig.ps1 如下所示：

```powershell
Configuration ISVBoxConfig
{
    Import-DscResource -ModuleName cChoco
    Import-DscResource -ModuleName xNetworking

    Node 'isvbox' {

        cChocoInstaller installChoco
        {
            InstallDir = 'C:\choco'
        }

        WindowsFeature installIIS
        {
            Ensure = 'Present'
            Name   = 'Web-Server'
        }

        xFirewall WebFirewallRule
        {
            Direction    = 'Inbound'
            Name         = 'Web-Server-TCP-In'
            DisplayName  = 'Web Server (TCP-In)'
            Description  = 'IIS allow incoming web site traffic.'
            Enabled       = 'True'
            Action       = 'Allow'
            Protocol     = 'TCP'
            LocalPort    = '80'
            Ensure       = 'Present'
        }

        cChocoPackageInstaller trivialWeb
        {
            Name      = 'trivialweb'
            Version   = '1.0.0'
            Source    = 'MY-NUGET-V2-SERVER-ADDRESS'
            DependsOn = '[cChocoInstaller]installChoco','[WindowsFeature]installIIS'
        }
    }
}
```

下面是 New-ConfigurationScript.ps1 脚本（已修改为使用 Az 模块）：

```powershell
Import-AzAutomationDscConfiguration `
    -ResourceGroupName MY-AUTOMATION-RG –AutomationAccountName MY-AUTOMATION-ACCOUNT `
    -SourcePath C:\temp\AzureAutomationDsc\ISVBoxConfig.ps1 `
    -Published –Force

$jobData = Start-AzAutomationDscCompilationJob `
    -ResourceGroupName MY-AUTOMATION-RG –AutomationAccountName MY-AUTOMATION-ACCOUNT `
    -ConfigurationName ISVBoxConfig

$compilationJobId = $jobData.Id

Get-AzAutomationDscCompilationJob `
    -ResourceGroupName MY-AUTOMATION-RG –AutomationAccountName MY-AUTOMATION-ACCOUNT `
    -Id $compilationJobId
```

这些步骤会生成要放在拉取服务器上的名为“ISVBoxConfig.isvbox”的新节点配置。 生成的节点配置名称为 `configurationName.nodeName`。

## <a name="step-5-create-and-maintain-package-metadata"></a>步骤 5：创建和维护包元数据

对于放入包存储库中的每一个包，需要使用 Nuspec 来描述它。 其必须经过编译并存储在 NuGet 服务器中。 [此处](https://docs.nuget.org/create/creating-and-publishing-a-package)对该过程进行了说明。 

可以使用 MyGet.org 作为 NuGet 服务器。 你可以购买此服务，但这是免费的入门 SKU。 在 [NuGet](https://www.nuget.org/) 中，可以找到为专用包安装你自己的 NuGet 服务器的说明。

## <a name="step-6-tie-it-all-together"></a>步骤 6：将其捆绑在一起

每当有某个版本通过 QA 和部署批准时，即会创建包，nuspec 和 nupkg 会更新并部署到 NuGet 服务器。 还必须更新配置（步骤 4）以便与新版本号匹配。 随后配置必须发送到拉取服务器并进行编译。

然后，依赖于该配置的 VM 将提取并安装更新。 其中的每项更新都很简单 - 只需一两行的 PowerShell 即可完成。 对于 Azure DevOps，有些更新封装在可一起链接到内部版本内的生成任务中。 [本文](https://www.visualstudio.com/docs/alm-devops-feature-index#continuous-delivery)提供更多详细信息。 此 [GitHub 存储库](https://github.com/Microsoft/vso-agent-tasks)详细介绍了各种可用的生成任务。

## <a name="related-articles"></a>相关文章
* [Azure 自动化 DSC 概述](automation-dsc-overview.md)
* [载入计算机以便通过 Azure 自动化 DSC 进行管理](automation-dsc-onboarding.md)

## <a name="next-steps"></a>后续步骤

- 有关概述，请参阅 [Azure Automation State Configuration 概述](automation-dsc-overview.md)。
- 若要开始使用此功能，请参阅 [Azure Automation State Configuration 入门](automation-dsc-getting-started.md)。
- 若要了解如何编译 DSC 配置，以便将它们分配给目标节点，请参阅[在 Azure Automation State Configuration 中编译 DSC 配置](automation-dsc-compile.md)。
- 有关 PowerShell cmdlet 参考，请参阅 [Az.Automation](/powershell/module/az.automation/?view=azps-3.7.0#automation)。
- 有关定价信息，请参阅 [Azure Automation State Configuration 定价](https://azure.microsoft.com/pricing/details/automation/)。

---
title: 使用 Azure 备份服务器备份工作负荷
description: 本文介绍了如何准备环境，以使用 Microsoft Azure 备份服务器 (MABS) 来保护和备份工作负荷。
ms.topic: conceptual
ms.date: 11/13/2018
ms.openlocfilehash: 6fe03260cc1759929e7ff9886b1b232a37056866
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/22/2020
ms.locfileid: "90975530"
---
# <a name="install-and-upgrade-azure-backup-server"></a>安装和升级 Azure 备份服务器

> [!div class="op_single_selector"]
>
> * [Azure 备份服务器](backup-azure-microsoft-azure-backup.md)
> * [SCDPM](backup-azure-dpm-introduction.md)
>
>

> 适用于：MABS v3。 （不再支持 MABS v2。 如果你使用的版本早于 MABS v3，请升级到最新版本。 ) 

本文介绍如何准备环境，以使用 Microsoft Azure 备份服务器 (MABS) 来备份工作负荷。 使用 Azure 备份服务器，可以通过一个控制台保护应用程序工作负载，如 Hyper-V VM、Microsoft SQL Server、SharePoint Server、Microsoft Exchange 和 Windows 客户端。

> [!NOTE]
> Azure 备份服务器现在可以保护 VMware VM 并提供改进的安全功能。 请按以下各部分所述安装该产品以及最新的 Azure 备份代理。 若要详细了解如何使用 Azure 备份服务器备份 VMware 服务器，请参阅[使用 Azure 备份服务器备份 VMware 服务器](backup-azure-backup-server-vmware.md)一文。 若要了解安全功能，请参阅 [Azure 备份安全功能文档](backup-azure-security-feature.md)。
>
>

部署在 Azure VM 中的 MABS 可以备份 Azure 中的 VM，但它们应位于同一域中以启用备份操作。 备份 Azure VM 的过程与在本地备份 VM 的过程相同，但在 Azure 中部署 MABS 有一些限制。 有关限制的详细信息，请参阅 [DPM 作为 Azure 虚拟机](/system-center/dpm/install-dpm#setup-prerequisites)

> [!NOTE]
> Azure 有两种用于创建和使用资源的部署模型：[资源管理器部署模型和经典部署模型](../azure-resource-manager/management/deployment-models.md)。 本文提供有关还原使用 Resource Manager 模型部署的 VM 的信息和过程。
>
>

Azure 备份服务器从 Data Protection Manager (DPM) 继承了大量工作负荷备份功能。 本文链接到 DPM 文档，介绍一些共享功能。 尽管 Azure 备份服务器与 DPM 共享许多相同的功能，Azure 备份服务器不会备份到磁带，也不与 System Center 集成。

## <a name="choose-an-installation-platform"></a>选择安装平台

若要启动并运行 Azure 备份服务器，首先要设置 Windows Server。 服务器可位于 Azure 中，也可位于本地。

* 若要保护本地工作负荷，MABS 服务器必须位于本地。
* 若要保护在 Azure VM 上运行的工作负荷，MABS 服务器必须位于 Azure 中，并作为 Azure VM 运行。

### <a name="using-a-server-in-azure"></a>使用 Azure 中的服务器

选择用于运行 Azure 备份服务器的服务器时，建议从 Windows Server 2016 Datacenter 或 Windows Server 2019 Datacenter 的库映像开始。 [在 Azure 门户中创建第一个 Windows 虚拟机](../virtual-machines/windows/quick-create-portal.md?toc=/azure/virtual-machines/windows/toc.json)一文提供了如何在 Azure 中开始使用建议的虚拟机的教程，即使以前从未使用过 Azure 也没关系。 建议服务器虚拟机 (VM) 最低要求应为：Standard_A4_v2 具有四个核心和 8 GB RAM。

使用 Azure 备份服务器保护工作负荷有许多细微差异需要注意。 [MABS 的保护矩阵](./backup-mabs-protection-matrix.md)帮助解释了这些细微差异。 部署计算机前，请先阅读完本文。

### <a name="using-an-on-premises-server"></a>使用本地服务器

如果你不想在 Azure 中运行基本服务器，则可以在 Hyper-v VM、VMware VM 或物理主机上运行服务器。 建议服务器硬件至少必须符合以下要求：双核和 8 GB RAM。 下表列出了支持的操作系统：

| 操作系统 | 平台 | SKU |
|:--- | --- |:--- |
| Windows Server 2019 |64 位 |Standard、Datacenter、Essentials |
| Windows Server 2016 和最新的 SP |64 位 |Standard、Datacenter、Essentials  |

可以使用 Windows Server 重复数据删除来删除 DPM 存储中的重复数据。 了解有关在 Hyper-V VM 中部署时 [DPM 和重复数据删除](/system-center/dpm/deduplicate-dpm-storage)如何配合工作的详细信息。

> [!NOTE]
> Azure 备份服务器设计为在专用的单一用途服务器上运行。 无法在上安装 Azure 备份服务器：
>
> * 作为域控制器运行的计算机
> * 安装了应用程序服务器角色的计算机
> * System Center Operations Manager 管理服务器的计算机
> * 运行 Exchange Server 的计算机
> * 作为群集节点的计算机
>
> Windows Server Core 或 Microsoft Hyper-V 服务器不支持安装 Azure 备份服务器。

请始终将 Azure 备份服务器加入域。 如果计划将服务器移到其他域，请先安装 Azure 备份服务器，然后将服务器加入到新域。 部署之后，*不支持*将现有 Azure 备份服务器计算机移到新域中。

无论是将备份数据发送到 Azure 还是在本地保留，都必须将 Azure 备份服务器注册到恢复服务保管库。

[!INCLUDE [backup-create-rs-vault.md](../../includes/backup-create-rs-vault.md)]

### <a name="set-storage-replication"></a>设置存储复制

存储复制选项可让你在异地冗余存储与本地冗余存储之间进行选择。 默认情况下，恢复服务保管库使用异地冗余存储。 如果此保管库是主保管库，请保留异地冗余存储这一存储选项。 如果想要一个更便宜、但持久性不太高的选项，请选择本地冗余存储。 有关详细信息，请参阅[Azure 存储复制概述](../storage/common/storage-redundancy.md)中[异地冗余](../storage/common/storage-redundancy.md#geo-redundant-storage)的[本地冗余](../storage/common/storage-redundancy.md#locally-redundant-storage)和[区域冗余](../storage/common/storage-redundancy.md#zone-redundant-storage)存储选项。

若要编辑存储复制设置，请执行以下操作：

1. 从 " **恢复服务保管库** " 窗格中选择新的保管库。 在 " **设置** " 部分下，选择 "  **属性**"。
2. 在 " **属性**" 中的 " **备份配置**" 下，选择 " **更新**"。

3. 选择存储复制类型，然后选择“保存”。

     ![设置新保管库的存储配置](./media/backup-create-rs-vault/recovery-services-vault-backup-configuration.png)

## <a name="software-package"></a>软件包

### <a name="downloading-the-software-package"></a>下载软件包

1. 登录 [Azure 门户](https://portal.azure.com/)。
2. 如果已打开恢复服务保管库，请转到步骤3。 如果未打开恢复服务保管库，但在 Azure 门户中，请在主菜单上选择 " **浏览**"。

   * 在资源列表中，键入“恢复服务”。
   * 开始键入时，会根据输入内容筛选该列表。 出现“**恢复服务保管库**”时，请选择它。

     ![创建恢复服务保管库步骤1](./media/backup-azure-microsoft-azure-backup/open-recovery-services-vault.png)

     此时显示恢复服务保管库列表。
   * 在恢复服务保管库列表中选择一个保管库。

     此时会打开选定的保管库仪表板。

     ![保管库仪表板](./media/backup-azure-microsoft-azure-backup/vault-dashboard.png)
3. 默认情况下，" **设置** " 窗格打开。 如果已关闭，请选择 " **设置** " 以打开 "设置" 窗格。

    ![设置窗格](./media/backup-azure-microsoft-azure-backup/vault-setting.png)
4. 选择 " **备份** " 打开入门向导。

    ![备份入门](./media/backup-azure-microsoft-azure-backup/getting-started-backup.png)

    在打开的 " **入门备份** " 窗格中，将自动选择 " **备份目标** "。

    ![Backup-goals-default-opened](./media/backup-azure-microsoft-azure-backup/getting-started.png)

5. 在 " **备份目标** " 窗格中，从 " **工作负荷的运行位置** " 菜单中选择 **"本地**"。

    ![用作目标的“本地”和“工作负荷”](./media/backup-azure-microsoft-azure-backup/backup-goals-azure-backup-server.png)

    从 " **要备份的内容？"** 下拉菜单中，选择要使用 Azure 备份服务器保护的工作负荷，然后选择 **"确定"**。

    “开始备份”向导可切换“准备基础结构”选项以将工作负荷备份到 Azure。

   > [!NOTE]
   > 如果只想备份文件和文件夹，建议使用 Azure 备份代理，并遵循[初步了解：备份文件和文件夹](./backup-windows-with-mars-agent.md)一文中的指南。 如果要保护的文件和文件夹超过文件和文件夹，或者计划在将来扩大保护需求，请选择这些工作负荷。
   >
   >

    ![快速启动向导更改](./media/backup-azure-microsoft-azure-backup/getting-started-prep-infra.png)

6. 在打开的 " **准备基础结构** " 窗格中，选择 "安装 Azure 备份服务器 **下载** 链接" 和 "下载保管库凭据"。 在向恢复服务保管库注册 Azure 备份服务器期间使用保管库凭据。 使用此链接转到“下载中心”，可从中下载软件包。

    ![为 Azure 备份服务器准备基础结构](./media/backup-azure-microsoft-azure-backup/azure-backup-server-prep-infra.png)

7. 选择所有文件，然后选择 " **下一步**"。 下载 Microsoft Azure 备份下载页中的所有文件，并将所有文件放在同一个文件夹中。

    ![下载中心 1](./media/backup-azure-microsoft-azure-backup/downloadcenter.png)

    由于所有文件的下载大小均为 > 3 GB，因此，在 10 Mbps 下载链接上，下载完成最多可能需要60分钟。

### <a name="extracting-the-software-package"></a>解压缩软件包

下载所有文件之后，选择 " **MicrosoftAzureBackupInstaller.exe**"。 这会启动“**Microsoft Azure 备份安装向导**”，并将安装程序文件解压缩到指定的位置。 继续执行向导，然后选择 " **提取** " 按钮开始解压缩过程。

> [!WARNING]
> 至少需要有 4GB 的可用空间才能解压缩安装程序文件。
>
>

![安装程序正在提取要安装的文件](./media/backup-azure-microsoft-azure-backup/extract/03.png)

提取过程完成后，请选中相应的复选框以启动刚刚解压缩的 *setup.exe* ，开始安装 Microsoft Azure 备份服务器并选择 " **完成** " 按钮。

### <a name="installing-the-software-package"></a>安装软件包

1. 选择 " **Microsoft Azure 备份** " 以启动安装向导。

    ![Microsoft Azure 备份安装向导](./media/backup-azure-microsoft-azure-backup/launch-screen2.png)
2. 在欢迎屏幕上，选择 " **下一步** " 按钮。 随即会转到“先决条件检查”部分。 在此屏幕上，选择 " **检查** " 以确定是否满足 Azure 备份服务器的硬件和软件先决条件。 如果成功满足所有先决条件，将看到一条消息，指示计算机满足要求。 选择“下一步”按钮。

    ![Azure 备份服务器 - 欢迎页和先决条件检查](./media/backup-azure-microsoft-azure-backup/prereq/prereq-screen2.png)
3. Azure 备份服务器安装包与所需的相应 SQL Server 二进制文件捆绑在一起。 启动新的 Azure 备份服务器安装时，选择 " **使用此安装程序安装 SQL Server 的新实例** " 选项，然后选择 " **检查并安装** " 按钮。 成功安装必备组件后，选择 " **下一步**"。

    >[!NOTE]
    >如果你要使用自己的 SQL Server，受支持的 SQL Server 版本包括 SQL Server 2014 SP1 或更高版本、2016 和 2017。  所有 SQL Server 版本都应当是 Standard 或 Enterprise 64 位。
    >Azure 备份服务器不能与远程 SQL Server 实例一起使用。 Azure 备份服务器使用的实例需在本地。 如果使用现有的 SQL server 进行 MABS，则 MABS 安装程序仅支持使用 SQL server 的 *命名实例* 。

    ![Azure 备份服务器 - SQL 检查](./media/backup-azure-microsoft-azure-backup/sql/01.png)

    如果出现故障，并建议重新启动计算机，请执行此操作，然后选择 " **再次检查**"。 如果遇到任何 SQL 配置问题，请根据 SQL 准则重新配置 SQL，然后使用现有的 SQL 实例重试安装/升级 MABS。

   **手动配置**

   使用自己的 SQL 实例时，请务必将 builtin\Administrators 添加到 master 数据库的 sysadmin 角色。

    **使用 SQL 2017 时的 SSRS 配置**

    使用自己的 SQL 2017 实例时，需要手动配置 SSRS。 配置 SSRS 后，请确保 SSRS 的 *IsInitialized* 属性设置为 *True*。 如果此属性设置为 True，MABS 将假设已配置 SSRS，因此会跳过 SSRS 配置。

    对 SSRS 配置使用以下值：
    * 服务帐户：“使用内置帐户”应为“网络服务”
    * Web 服务 URL：“虚拟目录”应为“ReportServer_\<SQLInstanceName>”
    * 数据库：DatabaseName 应为 ReportServer$\<SQLInstanceName>
    * Web 门户 URL：“虚拟目录”应为“Reports_\<SQLInstanceName>”

    [详细了解](/sql/reporting-services/report-server/configure-and-administer-a-report-server-ssrs-native-mode) SSRS 配置。

    > [!NOTE]
    > 用作 MABS 数据库的 SQL Server 的许可受 [Microsoft Online Services 条款](https://www.microsoft.com/licensing/product-licensing/products) (OST) 的约束。 根据 OST，与 MABS 捆绑的 SQL Server 只能用作 MABS 数据库。

4. 提供安装 Microsoft Azure 备份 server 文件的位置，然后选择 " **下一步**"。

    ![提供文件安装位置](./media/backup-azure-microsoft-azure-backup/space-screen.png)

    备份到 Azure 需要有暂存位置。 请确保暂存位置的空间至少为要备份到云的数据的 5%。 在磁盘保护方面，安装完成之后需要配置独立的磁盘。 有关存储池的详细信息，请参阅 [准备数据存储](/system-center/dpm/plan-long-and-short-term-data-storage)。
5. 为受限制的本地用户帐户提供强密码，然后选择 " **下一步**"。

    ![提供强密码](./media/backup-azure-microsoft-azure-backup/security-screen.png)
6. 选择是否要使用 *Microsoft 更新* 来检查更新，然后选择 " **下一步**"。

   > [!NOTE]
   > 我们建议让 Windows 更新重定向到 Microsoft 更新，此网站为 Windows 和 Microsoft Azure 备份服务器等其他产品提供了安全更新与重要更新。
   >
   >

    ![Microsoft 更新选择加入](./media/backup-azure-microsoft-azure-backup/update-opt-screen2.png)
7. 查看 *设置摘要* ，然后选择 " **安装**"。

    ![设置摘要](./media/backup-azure-microsoft-azure-backup/summary-screen.png)
8. 安装会分阶段进行。 第一个阶段，会在服务器上安装 Microsoft Azure 恢复服务代理。 向导还会检查 Internet 连接。 如果 Internet 连接可用，则可以继续安装。 如果没有，则需要提供代理详细信息以连接到 Internet。

    下一个步骤是配置 Microsoft Azure 恢复服务代理。 作为配置的一部分，你必须提供保管库凭据以将计算机注册到恢复服务保管库。 你还将提供密码，用于加密/解密在 Azure 和本地之间发送的数据。 可以自动生成通行短语，或提供自己的通行短语（最少包含 16 个字符）。 请继续运行向导，直到代理已完成配置。

    ![注册服务器向导](./media/backup-azure-microsoft-azure-backup/mars/04.png)
9. Microsoft Azure 备份服务器注册成功完成后，整个安装向导将继续安装和配置 SQL Server 及 Azure 备份服务器的组件。 SQL Server 组件安装完成后，会安装 Azure 备份服务器组件。

    ![Azure 备份服务器设置进度](./media/backup-azure-microsoft-azure-backup/final-install/venus-installation-screen.png)

安装步骤完成后，会一同创建产品的桌面图标。 双击该图标以启动产品。

### <a name="add-backup-storage"></a>添加备份存储

第一个备份副本保存在已附加到 Azure 备份服务器计算机的存储中。 有关添加磁盘的详细信息，请参阅[配置存储池和磁盘存储](./backup-mabs-add-storage.md)。

> [!NOTE]
> 即使你打算将数据发送到 Azure，也需要添加备份存储。 在当前的 Azure 备份服务器体系结构中，Azure 备份保管库将保存数据的*第二个*副本，而本地存储将保存第一个（必需的）备份副本。
>
>

### <a name="install-and-update-the-data-protection-manager-protection-agent"></a>安装和更新 Data Protection Manager 保护代理

MABS 使用 System Center Data Protection Manager 保护代理。 [此处](/system-center/dpm/deploy-dpm-protection-agent)介绍了在保护服务器上安装保护代理的步骤。

以下部分介绍如何更新客户端计算机的保护代理。

1. 在备份服务器管理员控制台中，选择“管理” > “代理”。

2. 在显示窗格中，选择要为其更新保护代理的客户端计算机。

   > [!NOTE]
   > “代理更新”列指示每个受保护计算机何时有保护代理更新可用。 在“操作”窗格中，仅当选择了受保护计算机并且有可用更新时，“更新”操作才可用。
   >
   >

3. 若要在所选计算机上安装更新的保护代理，请在“操作”窗格中选择“更新”。

4. 对于未连接到网络的客户端计算机，在计算机连接到网络之前，“代理状态”**** 列会显示“挂起更新”**** 状态。

   在客户端计算机连接到网络之后，客户端计算机的“代理更新”列会显示“正在更新”状态。

## <a name="move-mabs-to-a-new-server"></a>将 MABS 移到新服务器

如果需要将 MABS 移到新服务器，同时保留存储，请执行以下步骤。 仅当所有数据都在新式备份存储中时，才能执行此操作。

  > [!IMPORTANT]
  >
  > * 新的服务器名称必须与原始 Azure 备份服务器实例的名称相同。 若要使用以前的存储池和 MABS 数据库(DPMDB)保留恢复点，则不能更改新 Azure 备份服务器实例的名称。
  > * 必须有 MABS 数据库 (DPMDB) 的备份。 你将需要它来还原数据库。

1. 在显示窗格中，选择要为其更新保护代理的客户端计算机。
2. 关闭原始 Azure 备份服务器或使其脱机。
3. 重置 Active Directory 中的计算机帐户。
4. 在新计算机上安装服务器2016，并为其分配与原始 Azure 备份服务器相同的计算机名称。
5. 加入域。
6. 安装 Azure 备份服务器 V3 或更高版本 (从旧服务器移 MABS 存储池磁盘，并将) 导入。
7. 还原步骤 1 中创建的 DPMDB。
8. 将存储从原始备份服务器连接到新服务器。
9. 在 SQL 中，还原 DPMDB。
10. 在新服务器上以管理员) 身份运行 CMD (。 中转到 Microsoft Azure 备份安装位置和 bin 文件夹

    路径示例： `C:\windows\system32>cd "c:\Program Files\Microsoft Azure Backup\DPM\DPM\bin\"`

11. 若要连接到 Azure 备份，请运行 `DPMSYNC -SYNC`

    如果已将 **新** 磁盘添加到 DPM 存储池而不是移动旧磁盘，则运行 `DPMSYNC -Reallocatereplica` 。

## <a name="network-connectivity"></a>网络连接

Azure 备份服务器需要连接到 Azure 备份服务才能成功运行。 若要验证计算机是否已连接到 Azure，请在 Azure 备份服务器 PowerShell 控制台中使用 ```Get-DPMCloudConnection``` cmdlet。 如果 cmdlet 的输出为 TRUE，则连接存在，否则没有连接。

同时，Azure 订阅必须处于正常运行状态。 若要了解订阅的状态并对其进行管理，请登录到[订阅门户](https://account.windowsazure.com/Subscriptions)。

了解 Azure 连接和 Azure 订阅的状态后，可以使用下表来确定提供的备份/还原功能受到了哪些影响。

| 连接状态 | Azure 订阅 | 备份到 Azure | 备份到磁盘 | 从 Azure 还原 | 从磁盘还原 |
| --- | --- | --- | --- | --- | --- |
| 连续 |活动 |允许 |允许 |允许 |允许 |
| 连续 |Expired |已停止 |已停止 |允许 |允许 |
| 连续 |已取消预配 |已停止 |已停止 |已停止且已删除 Azure 恢复点 |已停止 |
| 连接断开超过 15 天 |活动 |已停止 |已停止 |允许 |允许 |
| 连接断开超过 15 天 |Expired |已停止 |已停止 |允许 |允许 |
| 连接断开超过 15 天 |已取消预配 |已停止 |已停止 |已停止且已删除 Azure 恢复点 |已停止 |

### <a name="recovering-from-loss-of-connectivity"></a>连接断开后进行恢复

如果你的防火墙或代理阻止访问 Azure，则需要在防火墙/代理配置文件中允许下列域地址：

* `http://www.msftncsi.com/ncsi.txt`
* \*.Microsoft.com
* \*.WindowsAzure.com
* \*.microsoftonline.com
* \*.windows.net

如果使用的是 ExpressRoute Microsoft 对等互连，请选择下列服务/区域：

* Azure Active Directory (12076:5060)
* Microsoft Azure 区域（根据恢复服务保管库的位置）
* Azure 存储（根据恢复服务保管库的位置）

有关更多详细信息，请访问 [ExpressRoute 路由要求](../expressroute/expressroute-routing.md)。

在 Azure 备份服务器计算机上还原与 Azure 的连接之后，可执行的操作取决于 Azure 订阅状态。 上表详细列出了有关计算机在“连接”之后允许的操作的信息。

### <a name="handling-subscription-states"></a>处理订阅状态

可以将 Azure 订阅从 *过期* 或 *取消预配* 状态转到 *活动* 状态。 但是，当状态 *处于非活动*状态时，这会对产品行为造成一些影响：

* *取消预配*订阅在取消预配的时间段内丢失功能。 切换为“活动”后，将恢复产品的备份/还原功能。 此外，只要以够长的保留期来保存本地磁盘上的备份数据，则还可以检索这些数据。 但是，一旦订阅进入“*已取消预配*”状态，Azure 中的备份数据便会丢失且不可检索。
* “*已过期*”的订阅只会在恢复“*活动*”状态之前失去功能。 在订阅 *过期* 期间计划的任何备份都不会运行。

## <a name="upgrade-mabs"></a>升级 MABS

使用以下过程升级 MABS。

### <a name="upgrade-from-mabs-v2-to-v3"></a>从 MABS V2 升级到 V3

> [!NOTE]
>
> MABS V2 并非安装 MABS V3 的必备组件。 但是，只能从 MABS V2 升级到 MABS V3。

使用以下步骤升级 MABS：

1. 若要从 MABS V2 升级到 MABS V3，请根据需要将 OS 升级到 Windows Server 2016 或 Windows Server 2019。

2. 升级服务器。 这些步骤类似于[安装](#install-and-upgrade-azure-backup-server)。 但是，对于 SQL 设置，你将获得将 SQL 实例升级到 SQL 2017 的选项，或使用你自己的 SQL server 2017 实例。

   > [!NOTE]
   >
   > 升级 SQL 实例时，请勿退出。 退出将卸载 SQL reporting 实例，因此尝试重新升级 MABS 将会失败。

   > [!IMPORTANT]
   >
   >  在升级到 SQL 2017 的过程中，我们会备份 SQL 加密密钥并卸载报告服务。 升级 SQL Server 后，将安装报告服务 (14.0.6827.4788) 并还原加密密钥。
   >
   > 手动配置 SQL 2017 时，请参阅“安装说明”下的“使用 SQL 2017 时的 SSRS 配置”部分。

3. 在受保护的服务器上更新保护代理。
4. 备份应会继续，而无需重启生产服务器。
5. 现在，可以开始保护数据。 如果要升级到新式备份存储，同时保护，你还可以选择要在其中存储备份的卷，并在预配空间下检查。 [了解详细信息](backup-mabs-add-storage.md)。

## <a name="troubleshooting"></a>疑难解答

如果 Microsoft Azure 备份服务器在安装阶段（或者备份或还原时）失败并出现错误，请参阅此[错误代码文档](https://support.microsoft.com/kb/3041338)以获取详细信息。
此外，还可以参考 [Azure 备份相关的常见问题](backup-azure-backup-faq.md)

## <a name="next-steps"></a>后续步骤

可以在此处获取有关[为 DPM 准备环境](/system-center/dpm/prepare-environment-for-dpm)的详细信息。 其中还包含有关可在其上部署和使用 Azure 备份服务器的受支持配置的信息。 可以使用一系列 [PowerShell cmdlet](/powershell/module/dataprotectionmanager/) 来执行各种操作。

请参阅这些文章，以深入了解如何使用 Microsoft Azure 备份服务器来保护工作负荷。

* [SQL Server 备份](backup-azure-backup-sql.md)
* [SharePoint Server 备份](backup-azure-backup-sharepoint.md)
* [备用服务器备份](backup-azure-alternate-dpm-server.md)

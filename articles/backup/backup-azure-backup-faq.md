---
title: 常见问题的解答
description: '针对以下常见问题的解答：包括恢复服务保管库在内的 Azure 备份功能、能够备份的内容、原理、加密和限制。 '
ms.topic: conceptual
ms.date: 07/07/2019
ms.openlocfilehash: 26d833dfea214c5f29b77c2ac34a5f8863c7fbc0
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/22/2020
ms.locfileid: "90986418"
---
# <a name="azure-backup---frequently-asked-questions"></a>Azure 备份 - 常见问题

本文回答有关 Azure 备份服务的常见问题。

## <a name="recovery-services-vault"></a>恢复服务保管库

### <a name="is-there-any-limit-on-the-number-of-vaults-that-can-be-created-in-each-azure-subscription"></a>在每个 Azure 订阅中可以创建的保管库数量是否有任何限制？

是的。 在 Azure 备份支持的区域中，可以为每个订阅最多创建 500 个恢复服务保管库。 如果需要更多保管库，请创建另一订阅。

### <a name="are-there-limits-on-the-number-of-serversmachines-that-can-be-registered-against-each-vault"></a>可针对每个保管库注册的服务器/计算机数量是否有限制？

每个保管库最多可以注册 1000 个 Azure 虚拟机。 如果你使用的是 Microsoft Azure 备份代理，则每个保管库最多可以注册 50 MARS 代理。 并且，可以将 50 个 MABS 服务器/DPM 服务器注册到一个保管库。

### <a name="how-many-datasourcesitems-can-be-protected-in-a-vault"></a>在保管库中可以保护多少个数据源/项？

可以在保管库中的所有工作负荷 (例如 IaaS VM、SQL、AFS) 上保护最多2000数据源/项。
例如，如果已在保管库中保护了 500 个 VM 和 400 个 Azure 文件存储共享，则最多只能保护其中的 1100 个 SQL 数据库。

### <a name="how-many-policies-can-i-create-per-vault"></a>每个保管库可以创建多少个策略？

每个保管库最多只能有 200 个策略。

### <a name="if-my-organization-has-one-vault-how-can-i-isolate-data-from-different-servers-in-the-vault-when-restoring-data"></a>如果本组织有一个保管库，如何在还原数据时将数据与保管库中的其他服务器隔离？

设置备份时，你想要一起恢复的服务器数据应使用相同密码。 如果要将恢复隔离到一个特定服务器或多个服务器，仅使用该服务器的密码。 例如，人力资源服务器可能使用一个加密通行短语，会计结算服务器使用另一个通行短语，而存储服务器使用第三个通行短语。

### <a name="can-i-move-my-vault-between-subscriptions"></a>是否可以在订阅之间移动我的保管库？

是的。 若要移动恢复服务保管库，请参阅此[文章](backup-azure-move-recovery-services-vault.md)

### <a name="can-i-move-backup-data-to-another-vault"></a>是否可以将备份数据移动到另一个保管库？

否。 保管库中存储的备份数据无法移动到不同的保管库。

### <a name="can-i-change-the-storage-redundancy-setting-after-a-backup"></a>能否在备份后更改存储冗余设置？

存储复制类型默认设置为异地冗余存储 (GRS)。 配置备份后，将禁用修改选项且不可更改。

![存储复制类型](./media/backup-azure-backup-faq/storage-replication-type.png)

如果已配置备份，并且必须从 GRS 移动到 LRS，请参阅 [配置备份后如何从 GRS 更改为 LRS](backup-create-rs-vault.md#how-to-change-from-grs-to-lrs-after-configuring-backup)。

### <a name="can-i-do-an-item-level-restore-ilr-for-vms-backed-up-to-a-recovery-services-vault"></a>是否可以对备份到恢复服务保管库的 VM 执行项级别还原 (ILR)？

- 由 Azure VM 备份支持的 Azure VM 支持 ILR。 有关详细信息，请参阅相关[文章](backup-azure-restore-files-from-vm.md)
- Azure 备份服务器 (MABS) 或 System Center DPM 备份本地 Vm 的在线恢复点，则不支持 ILR。

## <a name="azure-backup-agent"></a>Azure 备份代理

### <a name="where-can-i-find-common-questions-about-the-azure-backup-agent-for-azure-vm-backup"></a>在哪里可以找到有关 Azure VM 备份的 Azure 备份代理的常见问题？

- 有关在 Azure VM 上运行的代理，请阅读此[常见问题解答](backup-azure-vm-backup-faq.md)。
- 如需用于备份 Azure 文件夹的代理，请阅读此[常见问题解答](backup-azure-file-folder-backup-faq.md)。

## <a name="general-backup"></a>常规备份

### <a name="are-there-limits-on-backup-scheduling"></a>备份计划是否有限制？

是的。

- 可以备份 Windows Server 或 Windows 计算机，最多一天三次。 可以将计划策略设置为每日或每周计划。
- 可以备份 DPM，最多一天两次。 可以将计划策略设置为每日、每周、每月或每年。
- Azure VM 可每日备份一次。

### <a name="what-operating-systems-are-supported-for-backup"></a>支持哪些操作系统进行备份？

Azure 备份支持操作系统对文件和文件夹以及使用 Azure 备份服务器和 DPM 保护的工作负载应用程序进行备份。

**OS** | **SKU** | **详细信息**
--- | --- | ---
工作站 | |
Windows 10 64 位 | Enterprise、Pro、Home | 计算机应运行最新服务包和更新。
Windows 8.1 64 位 | Enterprise、Pro | 计算机应运行最新服务包和更新。
Windows 8 64 位 | Enterprise、Pro | 计算机应运行最新服务包和更新。
Windows 7 64 位 | Ultimate、Enterprise、Professional、Home Premium、Home Basic、Starter | 计算机应运行最新服务包和更新。
服务器 | |
Windows Server 2019 64 位 | Standard、Datacenter、Essentials | 使用最新服务包/更新。
Windows Server 2016 64 位 | Standard、Datacenter、Essentials | 使用最新服务包/更新。
Windows Server 2012 R2 64 位 | Standard、Datacenter、Foundation | 使用最新服务包/更新。
Windows Server 2012 64 位 | Datacenter、Foundation、Standard | 使用最新服务包/更新。
Windows Storage Server 2016 64 位 | Standard、Workgroup | 使用最新服务包/更新。
Windows Storage Server 2012 R2 64 位 | Standard、Workgroup、Essential | 使用最新服务包/更新。
Windows Storage Server 2012 64 位 | Standard、Workgroup | 使用最新服务包/更新。
Windows Server 2008 R2 SP1 64 位 | Standard、Enterprise、Datacenter、Foundation | 使用最新更新。
Windows Server 2008 64 位 | Standard、Enterprise、Datacenter | 使用最新更新。

Azure 备份不支持 32 位操作系统。

对于 Azure VM Linux 备份，Azure 备份支持 [Azure 认可的分发版列表](../virtual-machines/linux/endorsed-distros.md)，但 Core OS Linux 和 32 位操作系统除外。 只要 VM 上装有 VM 代理且支持 Python，其他自带 Linux 发行版应该也能正常运行。

### <a name="are-there-size-limits-for-data-backup"></a>是否存在数据备份的大小限制？

大小限制如下所示：

OS/计算机 | 数据源的大小限制
--- | ---
Windows 8 或更高版本 | 54,400 GB
Windows 7 |1700 GB
Windows Server 2012 或更高版本 | 54,400 GB
Windows Server 2008、Windows Server 2008 R2 | 1700 GB
Azure VM | 请参阅 [Azure VM 备份的支持矩阵](./backup-support-matrix-iaas.md#vm-storage-support)

### <a name="how-is-the-data-source-size-determined"></a>如何确定数据源大小？

下表说明了如何确定每个数据源大小。

**数据源** | **详细信息**
--- | ---
数据量(Volume) |从正在备份的单个卷 VM 备份的数据量。
SQL Server 数据库 |所备份的单个数据库的大小。
SharePoint | 正在备份的 SharePoint 场中内容和配置数据库的总和。
Exchange |正在备份 Exchange 服务器中所有 Exchange 数据库的总和。
BMR/系统状态 |正在备份计算机的 BMR 或系统状态的每个副本。

### <a name="is-there-a-limit-on-the-amount-of-data-backed-up-using-a-recovery-services-vault"></a>使用恢复服务保管库备份的数据量是否有限制？

可以使用恢复服务保管库备份的总数据量没有限制。 单个数据源（Azure VM 除外）大小最大可为 54,400 GB。 有关限制的详细信息，请参阅[支持矩阵中的保管库限制部分](./backup-support-matrix.md#vault-support)。

### <a name="why-is-the-size-of-the-data-transferred-to-the-recovery-services-vault-smaller-than-the-data-selected-for-backup"></a>为什么传输到恢复服务保管库的数据的大小小于选择进行备份的数据？

从 Azure 备份代理、DPM 和 Azure 备份服务器备份的数据都会在传输之前进行压缩和加密。 应用压缩和加密后，保管库中的数据将减少 30-40%。

### <a name="can-i-delete-individual-files-from-a-recovery-point-in-the-vault"></a>可以从保管库中的恢复点删除单个文件吗？

不可以，Azure 备份不支持从存储的备份中删除或清除单个项。

### <a name="if-i-cancel-a-backup-job-after-it-starts-is-the-transferred-backup-data-deleted"></a>如果在备份作业开始后取消，是否会删除已传输的备份数据？

否。 在备份作业取消之前传输到保管库中的所有数据将保留在保管库中。

- Azure 备份使用检查点机制，在备份过程中偶尔要对备份数据添加检查点。
- 由于备份数据中有检查点，下次备份过程可以验证文件的完整性。
- 下一备份作业会在已备份数据的基础上进行增量备份。 增量备份仅传输新的或已更改的数据，这相当于更好地利用带宽。

如果取消 Azure VM 的备份作业，则会忽略任何已传输的数据。 下次备份作业将传输上次成功的备份作业之后的增量数据。

## <a name="retention-and-recovery"></a>保留和恢复

### <a name="are-the-retention-policies-for-dpm-and-windows-machines-without-dpm-the-same"></a>DPM 和不带 DPM 的 Windows 计算机的保留策略是否相同？

是的，它们都有每日、每周、每月和每年保留策略。

### <a name="can-i-customize-retention-policies"></a>能否自定义保留策略？

是的，可以自定义策略。 例如，可以配置每周和每日保留期要求，但不能配置每年和每月保留期要求。

### <a name="can-i-use-different-times-for-backup-scheduling-and-retention-policies"></a>是否可以对备份计划和保留策略使用不同时间？

否。 只能在备份时间点应用保留策略。 例如，此图显示了中午 12 点和下午 6 点创建的备份保留策略。

![计划备份和保持](./media/backup-azure-backup-faq/Schedule.png)

### <a name="if-a-backup-is-kept-for-a-long-time-does-it-take-more-time-to-recover-an-older-data-point"></a>如果备份保留了很长一段时间，是否需要更多时间才能恢复较旧的数据点？

否。 恢复最旧或最新时间点所需的时间相同。 每个恢复点的行为类似一个完整的点。

### <a name="if-each-recovery-point-is-like-a-full-point-does-it-impact-the-total-billable-backup-storage"></a>如果每个恢复点相当于完整的点，它会影响总体可计费备份存储吗？

典型的长期保留点产品将备份数据存储为完整的点。

- 完整点的存储 *效率不高* ，但能使还原变得更方便和快速。
- 增量复制为高效存储，但要求还原数据链，这会影响恢复时间

Azure 备份存储体系结构在这两方面都能提供最佳性能，它以最佳方式存储数据，以便快速还原，且产生的存储成本低。 这种方法可确保提高（入口和出口）带宽使用效率。 数据存储量和恢复数据所需的时间都会尽量减少。 了解有关[增量备份](https://azure.microsoft.com/blog/microsoft-azure-backup-save-on-long-term-storage/)的更多信息。

### <a name="is-there-a-limit-on-the-number-of-recovery-points-that-can-be-created"></a>可创建的恢复点数量是否有限制？

最多可为单个受保护实例创建 9999 个恢复点。 受保护的实例包括计算机、服务器（物理或虚拟）或备份到 Azure 的工作负载。

- 了解有关[备份和保留](./backup-support-matrix.md)的更多信息。

### <a name="how-many-times-can-i-recover-data-thats-backed-up-to-azure"></a>我可以恢复多少次备份到 Azure 的数据？

Azure 备份的恢复数量没有限制。

### <a name="when-restoring-data-do-i-pay-for-the-egress-traffic-from-azure"></a>还原数据时，Azure 的出口流量是否需要付费？

否。 恢复是免费的，不收取传出流量费。

### <a name="what-happens-when-i-change-my-backup-policy"></a>如果更改备份策略，会发生什么情况？

应用新策略时，将遵循新策略的计划和保留期。

- 如果延长保留期，则会对现有的恢复点进行标记，按新策略要求来保留它们。
- 如果缩短保留期，则会将其标记为在下一清理作业中删除，随后会将其删除。

### <a name="how-long-is-data-retained-when-stopping-backups-but-selecting-the-option-to-retain-backup-data"></a>停止备份时数据保留多长时间，但选择保留备份数据的选项？

当停止备份并保留数据时，数据删除的现有策略规则将停止，并且数据将无限期保留，直到管理员要删除。

## <a name="encryption"></a>Encryption

### <a name="is-the-data-sent-to-azure-encrypted"></a>发送到 Azure 的数据会加密吗？

是的。 使用 AES256 在本地计算机上对数据进行加密。 数据通过安全的 HTTPS 链接发送。 在云中传输的数据仅在存储和恢复服务之间通过 HTTPS 链接进行保护。 iSCSI 协议可保护恢复服务和用户计算机之间传输的数据。 安全隧道用于保护 iSCSI 通道。

### <a name="is-the-backup-data-on-azure-encrypted-as-well"></a>Azure 中的备份数据也会加密吗？

是的。 Azure 中的数据为静态加密。

- 对于本地备份，使用在备份到 Azure 时提供的密码提供静态加密。
- 对于 Azure VM，使用存储服务加密 (SSE) 对数据进行静态加密。

Microsoft 在任何时候都不会解密备份数据。

### <a name="what-is-the-minimum-length-of-the-encryption-key-used-to-encrypt-backup-data"></a>用于加密备份数据的加密密钥的最小长度是多少？

Microsoft Azure 恢复服务 (MARS) 代理使用的加密密钥派生自密码长度至少为16个字符的密码。 对于 Azure Vm，Azure KeyVault 使用的密钥长度没有限制。

### <a name="what-happens-if-i-misplace-the-encryption-key-can-i-recover-the-data-can-microsoft-recover-the-data"></a>如果我丢失了加密密钥，会发生什么情况？ 能否恢复数据？ Microsoft 能否恢复数据？

用于加密备份数据的密钥仅出现在你的站点上。 Microsoft 不会在 Azure 中保留副本，也不能访问密钥。 如果错放密钥，Microsoft 将无法恢复备份数据。

## <a name="next-steps"></a>后续步骤

阅读其他常见问题：

- 有关 Azure VM 备份的[常见问题](backup-azure-vm-backup-faq.md)。
- 有关 Azure 备份代理的[常见问题](backup-azure-file-folder-backup-faq.md)

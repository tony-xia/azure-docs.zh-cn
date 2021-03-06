---
title: 虚拟机的软删除
description: 了解虚拟机的软删除如何使备份更安全。
ms.topic: conceptual
ms.date: 04/30/2020
ms.custom: references_regions
ms.openlocfilehash: a8b70d4c8240d096c19e5a8d7449921557b8896c
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2020
ms.locfileid: "89022235"
---
# <a name="soft-delete-for-virtual-machines"></a>虚拟机的软删除

VM 的软删除可保护 VM 的备份免遭意外删除。 即使删除了备份，这些备份也会以软删除状态再保留 14 天。

> [!NOTE]
> 软删除只是保护已删除的备份数据。 如果在没有备份的情况下删除 VM，软删除功能不会保留数据。 应使用 Azure 备份保护所有资源，以确保能够完全复原。
>

## <a name="supported-regions"></a>支持的区域

目前，美国西部当前支持软删除。东亚、加拿大中部、加拿大东部、法国中部、法国南部、韩国中部、韩国南部、英国南部、英国西部、澳大利亚东部、澳大利亚东南部、北欧、美国西部、2、美国中部、美国西部、美国中北部、美国中南部东亚、日本东部、日本西部、印度南部、印度中部、日本西部、印度南部、印度中部、印度西部、美国东部2、瑞士北部、瑞士西部、挪威西部、挪威东部和所有国家地区。

## <a name="soft-delete-for-vms-using-azure-portal"></a>使用 Azure 门户对 VM 进行软删除

1. 若要删除 VM 的备份数据，必须停止备份。 在 Azure 门户中，请参阅恢复服务保管库，右键单击备份项，然后选择 " **停止备份**"。

   ![Azure 门户中备份项的屏幕截图](./media/backup-azure-security-feature-cloud/backup-stopped.png)

2. 在接下来的窗口中，会提供用于删除或保留备份数据的选项。 如果依次选择“删除备份数据”、“停止备份”，则不会永久删除 VM 备份，**** **** 而是将备份数据以软删除状态保留 14 天。 如果选择“删除备份数据”，则会将一封“删除”警报电子邮件发送到配置的电子邮件 ID，告知用户备份数据将延期保留 14 天。**** 在第 12 天时也会发送一封警报电子邮件，告知还有两天时间可以恢复已删除的数据。 删除会推迟到第 15 天，到时数据将永久删除，同时系统会发送最后一封警报电子邮件，告知数据已永久删除。

   ![Azure 门户中“停止备份”屏幕的屏幕截图](./media/backup-azure-security-feature-cloud/delete-backup-data.png)

3. 在14天内，在恢复服务保管库中，软删除的 VM 旁边会显示一个红色的 "软删除" 图标。

   ![Azure 门户中处于软删除状态的 VM 的屏幕截图](./media/backup-azure-security-feature-cloud/vm-soft-delete.png)

   > [!NOTE]
   > 如果保管库中存在任何软删除的备份项，则无法在该时间删除该保管库。 在永久删除备份项后，尝试删除保管库，保管库中没有剩余的软删除状态项。

4. 若要还原软删除的 VM，必须先将其取消删除。 若要取消删除，请选择软删除的 VM，然后选择“取消删除”选项。****

   ![Azure 门户中“取消删除 VM”的屏幕截图](./media/backup-azure-security-feature-cloud/choose-undelete.png)

   窗口中会显示一条警告，指出如果选择取消删除，将取消删除 VM 的所有还原点，这些还原点可用于执行还原操作。 VM 将保留为“停止保护并保留数据”状态，备份将会暂停，备份数据将永久保留，且没有任何生效的备份策略。

   ![Azure 门户中“确认取消删除 VM”的屏幕截图](./media/backup-azure-security-feature-cloud/undelete-vm.png)

   此时，还可以通过从选定还原点中选择“还原 VM”来还原 VM。****  

   ![Azure 门户中“还原 VM”选项的屏幕截图](./media/backup-azure-security-feature-cloud/restore-vm.png)

   > [!NOTE]
   > 只有在用户执行“恢复备份”操作后，垃圾回收器才会运行并清理已过期的恢复点。****

5. 取消删除操作完成后，状态将恢复为“停止备份并保留数据”，然后，你可以选择“恢复备份”。**** “恢复备份”操作会恢复处于活动状态的、与定义备份和保留计划的用户所选的备份策略相关联的备份项。****

   ![Azure 门户中“恢复备份”选项的屏幕截图](./media/backup-azure-security-feature-cloud/resume-backup.png)

## <a name="soft-delete-for-vms-using-azure-powershell"></a>使用 Azure PowerShell 对 VM 进行软删除

> [!IMPORTANT]
> 使用 Azure PowerShell 进行软删除所需的 Az.RecoveryServices 版本最低为 2.2.0。 可使用 ```Install-Module -Name Az.RecoveryServices -Force``` 获取最新版本。

如上所述 Azure 门户，使用 Azure PowerShell 时，步骤顺序是相同的。

### <a name="delete-the-backup-item-using-azure-powershell"></a>使用 Azure PowerShell 删除备份项

使用 [AzRecoveryServicesBackupProtection](/powershell/module/az.recoveryservices/disable-azrecoveryservicesbackupprotection) PowerShell cmdlet 删除备份项。

```powershell
Disable-AzRecoveryServicesBackupProtection -Item $myBkpItem -RemoveRecoveryPoints -VaultId $myVaultID -Force

WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   -----
AppVM1           DeleteBackupData     Completed            12/5/2019 12:44:15 PM     12/5/2019 12:44:50 PM     0488c3c2-accc-4a91-a1e0-fba09a67d2fb
```

备份项的“DeleteState”将从“NotDeleted”更改为“ToBeDeleted”。 备份数据会保留 14 天。 如果要还原删除操作，则应执行撤消-删除操作。

### <a name="undoing-the-deletion-operation-using-azure-powershell"></a>使用 Azure PowerShell 撤销删除操作

首先，提取处于软删除状态的相关备份项， (即，将) 删除。

```powershell

Get-AzRecoveryServicesBackupItem -BackupManagementType AzureVM -WorkloadType AzureVM -VaultId $myVaultID | Where-Object {$_.DeleteState -eq "ToBeDeleted"}

Name                                     ContainerType        ContainerUniqueName                      WorkloadType         ProtectionStatus     HealthStatus         DeleteState
----                                     -------------        -------------------                      ------------         ----------------     ------------         -----------
VM;iaasvmcontainerv2;selfhostrg;AppVM1    AzureVM             iaasvmcontainerv2;selfhostrg;AppVM1       AzureVM              Healthy              Passed               ToBeDeleted

$myBkpItem = Get-AzRecoveryServicesBackupItem -BackupManagementType AzureVM -WorkloadType AzureVM -VaultId $myVaultID -Name AppVM1
```

然后，使用 [AzRecoveryServicesBackupItemDeletion](/powershell/module/az.recoveryservices/undo-azrecoveryservicesbackupitemdeletion) PowerShell cmdlet 执行撤消删除操作。

```powershell
Undo-AzRecoveryServicesBackupItemDeletion -Item $myBKpItem -VaultId $myVaultID -Force

WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   -----
AppVM1           Undelete             Completed            12/5/2019 12:47:28 PM     12/5/2019 12:47:40 PM     65311982-3755-46b5-8e53-c82ea4f0d2a2
```

备份项的“DeleteState”将还原为“NotDeleted”。 但保护仍处于停止状态。 [恢复备份](./backup-azure-vms-automation.md#change-policy-for-backup-items)以重新启用保护。

## <a name="soft-delete-for-vms-using-rest-api"></a>使用 REST API 对 VM 进行软删除

- 如[此处](backup-azure-arm-userestapi-backupazurevms.md#stop-protection-and-delete-data)所述，使用 REST API 删除备份。
- 如果要撤消这些删除操作，请参阅 [此处](backup-azure-arm-userestapi-backupazurevms.md#undo-the-deletion)所述的步骤。

## <a name="how-to-disable-soft-delete"></a>如何禁用软删除

建议不要禁用此功能。 唯一应该考虑禁用软删除的情况是，你打算将受保护的项移到新保管库，需要在删除后重新进行保护，因此等不及要求的 14 天（例如在测试环境中）。有关如何禁用软删除的说明，请参阅[启用和禁用软删除](backup-azure-security-feature-cloud.md#enabling-and-disabling-soft-delete)。

## <a name="next-steps"></a>后续步骤

- 阅读有关软删除的[常见问题解答](backup-azure-security-feature-cloud.md#frequently-asked-questions)
- 了解 [Azure 备份中的所有安全功能](security-overview.md)

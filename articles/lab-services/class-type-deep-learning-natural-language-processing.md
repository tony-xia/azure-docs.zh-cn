---
title: 使用 Azure 实验室服务设置专注于深度学习的实验室 | Microsoft Docs
description: 了解如何使用 Azure 实验室服务在自然语言处理 (NLP) 中设置以深度学习为中心的实验室。
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 01199e76b7b9cb1a6360b5aba010d0cfd8936c23
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91251450"
---
# <a name="set-up-a-lab-focused-on-deep-learning-in-natural-language-processing-using-azure-lab-services"></a>使用 Azure 实验室服务设置专注于自然语言处理中的深度学习的实验室
本文介绍如何使用 Azure 实验室服务来设置一个专注于自然语言处理 (NLP) 中的深度学习的实验室。 自然语言处理 (NLP) 是某种形式的人工智能 (AI)，可在计算机中实现翻译、语音识别和其他语言理解功能。  

使用 NLP 类的学生可以通过 Linux 虚拟机 (VM) 了解如何应用神经网络算法，以开发深度学习模型用于分析人类手写语言。 

## <a name="lab-configuration"></a>实验室配置
若要设置此实验室，你需要具有一个 Azure 订阅才能开始。 如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/)。 拥有 Azure 订阅后，可以在 Azure 实验室服务中创建新的实验室帐户，也可以使用现有的实验室帐户。 请参阅以下教程，了解如何创建新的实验室帐户：[有关设置实验室帐户的教程](tutorial-setup-lab-account.md)。
 
创建实验室帐户后，在实验室帐户中启用以下设置： 

| 实验室帐户设置 | Instructions |
| ----------- | ------------ |  
| 市场映像 | 启用用于 Linux (Ubuntu) 的 Data Science Virtual Machine 映像，以便在实验室帐户中使用。  有关说明，请参阅以下文章：[指定可供实验室创建者使用的市场映像](specify-marketplace-images.md)。 | 

按照[此教程](tutorial-setup-classroom-lab.md)创建新的实验室并应用以下设置：

| 实验室设置 | 值/说明 | 
| ------------ | ------------------ |
| 虚拟机 (VM) 大小 | **小型 GPU (计算) **。 此大小最适用于计算密集型和网络密集型应用程序（如人工智能和深度学习）。 |
| VM 映像 | [用于 Linux (Ubuntu)的 Data Science Virtual Machine](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-dsvm.ubuntu-1804)。 此映像提供机器学习和数据科学深度学习框架和工具。 若要查看此映像上安装的工具的完整列表，请参阅以下文章：[DSVM 中包含哪些组件？](../machine-learning/data-science-virtual-machine/overview.md#whats-included-on-the-dsvm)。 |
| 启用远程桌面连接 | <p>数据科学映像已配置为使用 X2Go，使教师和学生能够使用 GUI 远程桌面进行连接。  X2Go 不要求启用“启用远程桌面连接”设置。  仅当你要改用 RDP 时，才需要启用此设置。

>**重要说明**：尽管我们建议对数据科学映像使用 X2Go，但如果你要改用 RDP，则首次连接到 Linux VM 时需要使用 SSH，并安装 RDP 和 GUI 包。  以后，你/学生可以使用 RDP 连接到 Linux VM。  有关详细信息，请参阅[为 Linux VM 启用图形远程桌面](how-to-enable-remote-desktop-linux.md)。

用于 Linux 的 Data Science Virtual Machine 映像提供此类型课程所需的深度学习框架和工具。 因此，在创建模板虚拟机后，无需进一步自定义。 可以将其发布供学生使用。 选择模板页上的“发布”按钮，将模板发布到实验室。  

## <a name="cost"></a>成本
如果要估算此实验室的成本，可以使用以下示例： 

如果一个班级有 25 个学生，计划的课堂时间为 20 小时且家庭作业或任务时数配额为 10 小时，则实验室的价格为 - 25 个学生 * (20 + 10) 个小时 * 139 个实验室单位 * 0.01 美元/小时 = 1042.5 美元

有关定价的更多详细信息，请参阅 [Azure 实验室服务定价](https://azure.microsoft.com/pricing/details/lab-services/)。

## <a name="conclusion"></a>结束语
本文已指导你完成为自然语言处理课程创建实验室的步骤。 对于其他深度学习课程，可以使用类似的设置。

## <a name="next-steps"></a>后续步骤
接下来的步骤是设置任何实验室时通用的：

- [添加用户](tutorial-setup-classroom-lab.md#add-users-to-the-lab)
- [设置配额](how-to-configure-student-usage.md#set-quotas-for-users)
- [设置日程安排](tutorial-setup-classroom-lab.md#set-a-schedule-for-the-lab) 
- [通过电子邮件将注册链接发送给学生](how-to-configure-student-usage.md#send-invitations-to-users)。 


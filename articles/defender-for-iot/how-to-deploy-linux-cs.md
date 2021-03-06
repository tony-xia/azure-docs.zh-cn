---
title: '安装 & 部署 Linux c # 代理'
description: '了解如何在 Linux 上安装和部署基于 IoT c # 的基于 IoT 的安全代理的 Defender'
services: defender-for-iot
ms.service: defender-for-iot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.devlang: na
ms.custom: devx-track-csharp
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/09/2020
ms.author: mlottner
ms.openlocfilehash: 48737831440a1402b6974955b4da61a4216b011f
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/22/2020
ms.locfileid: "90934468"
---
# <a name="deploy-defender-for-iot-c-based-security-agent-for-linux"></a>为适用于 Linux 的基于 IoT c # 的安全代理部署 Defender

本指南介绍了如何在 Linux 上安装和部署基于 IoT c # 的基于 IoT 的安全代理的 Defender。

本指南介绍如何：

> [!div class="checklist"]
> * 安装
> * 验证部署
> * 卸载代理
> * 疑难解答

## <a name="prerequisites"></a>先决条件

有关其他平台和代理风格，请参阅 [选择正确的安全代理](how-to-deploy-agent.md)。

1. 若要部署安全代理，需要在安装计算机上拥有本地管理员权限。

1. 为设备[创建安全模块](quickstart-create-security-twin.md)。

## <a name="installation"></a>安装

若要部署安全代理，请使用以下步骤：

1. 从 [GitHub](https://aka.ms/iot-security-github-cs)下载最新版本到您的计算机。

1. 提取包的内容，然后导航到 _/Install_ 文件夹。

1. 运行 `chmod +x InstallSecurityAgent.sh`，将运行权限添加到 **InstallSecurityAgent 脚本**

1. 接下来，请在 **根权限**下运行以下命令：

   ```
   ./InstallSecurityAgent.sh -i -aui <authentication identity>  -aum <authentication method> -f <file path> -hn <host name>  -di <device id> -cl <certificate location kind>
   ```

   有关身份验证参数的详细信息，请参阅 [如何配置身份验证](concept-security-agent-authentication-methods.md)。

此脚本可执行以下操作：

- 安装必备组件。

- 添加已禁用交互式登录) 的服务用户 (。

- 将代理安装为 **后台** 程序-假定设备使用 **systemd** 进行经典部署模型。

- 将 **sudoers** 配置为允许代理以 root 身份执行特定任务。

- 使用提供的身份验证参数配置代理。

如需更多帮助，请结合 -help 参数运行该脚本：`./InstallSecurityAgent.sh --help`

### <a name="uninstall-the-agent"></a>卸载代理

若要卸载代理，请结合 -u 参数运行该脚本：`./InstallSecurityAgent.sh -u`。

> [!NOTE]
> 卸载时，不会删除在安装过程中安装的任何缺失的必备组件。

## <a name="troubleshooting"></a>疑难解答

1. 运行以下命令来检查部署状态：

    `systemctl status ASCIoTAgent.service`

1. 启用日志记录。
   如果代理无法启动，请启用日志记录以获取更多信息。

   通过以下方式启用日志记录：

   1. 在任意 Linux 编辑器中打开要编辑的配置文件：

        `vi /var/ASCIoTAgent/General.config`

   1. 编辑以下值：

      ```
      <add key="logLevel" value="Debug"/>
      <add key="fileLogLevel" value="Debug"/>
      <add key="diagnosticVerbosityLevel" value="Some" />
      <add key="logFilePath" value="IotAgentLog.log"/>
      ```

       **logFilePath** 值是可以配置的。

       > [!NOTE]
       > 建议在故障排除操作完成后关闭日志记录功能。**** 让日志记录保持启用状态会增加日志文件大小和数据使用量。****

   1. 通过运行以下命令重启代理：

       `systemctl restart ASCIoTAgent.service`

   1. 查看日志文件，了解有关故障的详细信息。

       日志文件位置为：`/var/ASCIoTAgent/IotAgentLog.log`

       根据你在步骤 2 中为 **logFilePath** 选择的名称，更改文件位置路径。

## <a name="next-steps"></a>后续步骤

- 阅读用于 IoT 服务的 Defender [概述](overview.md)
- 了解有关用于 IoT[体系结构](architecture.md)的 Defender 的详细信息
- 启用该[服务](quickstart-onboard-iot-hub.md)
- 阅读[常见问题解答](resources-frequently-asked-questions.md)
- 了解[警报](concept-security-alerts.md)

---
title: 为 Windows 虚拟桌面配置 GPU - Azure
description: 如何在 Windows 虚拟桌面中启用 GPU 加速的渲染和编码。
author: gundarev
ms.topic: how-to
ms.date: 05/06/2019
ms.author: denisgun
ms.openlocfilehash: 33b8d3f62ef45c6078f10535c6376f611472f5a2
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/03/2020
ms.locfileid: "89441742"
---
# <a name="configure-graphics-processing-unit-gpu-acceleration-for-windows-virtual-desktop"></a>为 Windows 虚拟桌面配置图形处理单元 (GPU) 加速

>[!IMPORTANT]
>本教程的内容适用于包含 Azure 资源管理器 Windows 虚拟桌面对象的 Windows 虚拟桌面。 如果你使用的是不包含 Azure 资源管理器对象的 Windows 虚拟桌面（经典），请参阅[此文](./virtual-desktop-fall-2019/configure-vm-gpu-2019.md)。

Windows 虚拟桌面支持用于提高应用性能和可伸缩性的 GPU 加速渲染和编码。 GPU 加速对于图形密集型应用尤其重要。

按照本文中的说明创建 GPU 优化的 Azure 虚拟机，将其添加到你的主机池，并将其配置为使用 GPU 加速进行渲染和编码。 本文假设你已配置 Windows 虚拟桌面租户。

## <a name="select-a-gpu-optimized-azure-virtual-machine-size"></a>选择 GPU 优化的 Azure 虚拟机大小

Azure 提供了许多 [GPU 优化的虚拟机大小](/azure/virtual-machines/windows/sizes-gpu)。 主机池的正确选择取决于多种因素，包括特定的应用工作负荷、所需的用户体验质量和成本。 通常，在给定的用户密度下，GPU 的大小越大、功能越强，提供的用户体验就越好。

## <a name="create-a-host-pool-provision-your-virtual-machine-and-configure-an-app-group"></a>创建主机池、预配虚拟机并配置应用组

使用你选择的大小的 VM 创建新的主机池。 有关说明，请参阅[教程：使用 Azure 门户创建主机池](/azure/virtual-desktop/create-host-pools-azure-marketplace)。

Windows 虚拟桌面在以下操作系统中支持 GPU 加速的渲染和编码：

* Windows 10 版本 1511 或更高版本
* Windows Server 2016 或更高版本

你还必须配置应用组，或使用在创建新主机池时自动创建的默认桌面应用组（名为“桌面应用程序组”）。 有关说明，请参阅[教程：管理 Windows 虚拟桌面的应用组](/azure/virtual-desktop/manage-app-groups)。

## <a name="install-supported-graphics-drivers-in-your-virtual-machine"></a>在虚拟机中安装受支持的图形驱动程序

若要在 Windows 虚拟桌面中利用 Azure N 系列 VM 的 GPU 功能，必须安装相应的图形驱动程序。 按照[支持的操作系统和驱动程序](/azure/virtual-machines/windows/sizes-gpu#supported-operating-systems-and-drivers)中的说明，以手动方式或使用 Azure VM 扩展安装来自相应显卡供应商的驱动程序。

Windows 虚拟桌面仅支持 Azure 分发的驱动程序。 此外，对于具有 NVIDIA GPU 的 Azure VM，Windows 虚拟桌面仅支持 [NVIDIA GRID 驱动程序](/azure/virtual-machines/windows/n-series-driver-setup#nvidia-grid-drivers)。

安装驱动程序后，需要重启 VM。 使用上述说明中的验证步骤确认图形驱动程序已成功安装。

## <a name="configure-gpu-accelerated-app-rendering"></a>配置 GPU 加速的应用渲染

默认情况下，在多会话配置中运行的应用和桌面是使用 CPU 呈现的，不会利用可用 GPU 进行渲染。 为会话主机配置组策略以启用 GPU 加速的渲染：

1. 使用具有本地管理员特权的帐户连接到 VM 的桌面。
2. 打开“开始”菜单，键入“gpedit.msc”以打开组策略编辑器。
3. 在树中导航到“计算机配置” > “管理模板” > “Windows 组件” > “远程桌面服务” > “远程桌面会话主机” > “远程会话环境”。
4. 选择 "策略" **将硬件图形适配器用于所有远程桌面服务会话** ，并将此策略设置为 " **启用** "，以在远程会话中启用 GPU 呈现。

## <a name="configure-gpu-accelerated-frame-encoding"></a>配置 GPU 加速的帧编码

远程桌面对应用和桌面（无论是使用 GPU 还是 CPU 渲染的）呈现的所有图形进行编码，以便将其传输到远程桌面客户端。 经常更新屏幕的一部分时，屏幕的此部分使用视频编解码器进行编码， (AVC/) 。 默认情况下，远程桌面不会将可用 GPU 用于此编码。 为会话主机配置组策略以启用 GPU 加速的帧编码。 继续上面的步骤：

>[!NOTE]
>GPU 加速帧编码在 NVv4 系列虚拟机中不可用。

1. 选择“为远程桌面连接配置 H.264/AVC 硬件编码”策略，并将此策略设置为“已启用”，以便在远程会话中针对 AVC/H.264 启用硬件编码。

    >[!NOTE]
    >在 Windows Server 2016 中，将“首选 AVC 硬件编码”选项设置为“始终尝试”。

2. 在编辑了组策略后，请强制执行组策略更新。 打开“命令提示”并键入：

    ```cmd
    gpupdate.exe /force
    ```

3. 从远程桌面会话中注销。

## <a name="configure-fullscreen-video-encoding"></a>配置全屏视频编码

如果经常使用生成高帧速率内容（如3D 建模、CAD/CAM 和视频应用程序）的应用程序，则可以选择为远程会话启用全屏视频编码。 全屏视频配置文件为此类应用程序提供了更高的帧速率和更好的用户体验。 建议为全屏视频编码使用 GPU 加速帧编码。 为会话主机配置组策略以启用全屏视频编码。 继续上面的步骤：

1. 选择“为远程桌面连接优先使用 H.264/AVC 444 图形模式”策略，并将此策略设置为“已启用”，以便在远程会话中强制使用 H.264/AVC 444 编解码器。
2. 在编辑了组策略后，请强制执行组策略更新。 打开“命令提示”并键入：

    ```cmd
    gpupdate.exe /force
    ```

3. 从远程桌面会话中注销。
## <a name="verify-gpu-accelerated-app-rendering"></a>配置 GPU 加速的应用渲染

若要验证应用是否正在使用 GPU 进行渲染，请尝试执行以下任一操作：

* 对于具有 NVIDIA GPU 的 Azure Vm，请使用 `nvidia-smi` 在运行应用时 [验证驱动程序安装](/azure/virtual-machines/windows/n-series-driver-setup#verify-driver-installation) 中所述的实用程序来检查 GPU 使用率。
* 在受支持的操作系统版本上，可以使用任务管理器来检查 GPU 利用率。 在“性能”选项卡中选择 GPU，查看应用是否正在使用 GPU。

## <a name="verify-gpu-accelerated-frame-encoding"></a>验证 GPU 加速的帧编码

若要验证远程桌面是否正在使用 GPU 加速的编码，请执行以下操作：

1. 使用 Windows 虚拟桌面客户端连接到 VM 的桌面。
2. 启动事件查看器并导航到以下节点：“应用程序和服务日志” > “Microsoft” > “Windows” > “RemoteDesktopServices-RdpCoreCDV” > “操作”
3. 若要确定是否正在使用 GPU 加速的编码，请查找事件 ID 170。 如果看到“AVC 硬件编码器已启用:1”，这表示已使用了 GPU 编码。

## <a name="verify-fullscreen-video-encoding"></a>验证全屏视频编码

验证远程桌面是否正在使用全屏视频编码：

1. 使用 Windows 虚拟桌面客户端连接到 VM 的桌面。
2. 启动事件查看器并导航到以下节点：“应用程序和服务日志” > “Microsoft” > “Windows” > “RemoteDesktopServices-RdpCoreCDV” > “操作”
3. 若要确定是否使用了全屏视频编码，请查找事件 ID 162。 如果看到“可用 AVC:1 初始配置文件:2048”，这表示已使用了 AVC 444。

## <a name="next-steps"></a>后续步骤

按照这些说明操作，应当可以让 GPU 加速在一个会话主机（一个 VM）上正常运行。 有关在较大的主机池中启用 GPU 加速的其他一些注意事项如下：

* 请考虑使用 [VM 扩展](/azure/virtual-machines/extensions/overview)来简化多个 VM 上的驱动程序安装和更新。 将 [NVIDIA GPU 驱动程序扩展](/azure/virtual-machines/extensions/hpccompute-gpu-windows)用于具有 NVIDIA GPU 的 VM，并将 [AMD GPU 驱动程序扩展](/azure/virtual-machines/extensions/hpccompute-amd-gpu-windows)用于具有 AMD GPU 的 VM。
* 请考虑使用 Active Directory 组策略来简化在多个 VM 上配置组策略的过程。 有关在 Active Directory 域中部署组策略的信息，请参阅[使用组策略对象](https://go.microsoft.com/fwlink/p/?LinkId=620889)。

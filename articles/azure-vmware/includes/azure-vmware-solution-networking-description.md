---
title: Azure VMWare 解决方案网络和连接
description: Azure VMWare 解决方案网络和连接说明。
ms.topic: include
ms.date: 09/28/2020
ms.openlocfilehash: 1d43a8a49fc56a6a5b534a89fd1dfc2f5ae2d478
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/05/2020
ms.locfileid: "91574444"
---
<!-- Used in introduction.md and concepts-networking.md -->

Azure VMware 解决方案提供一个私有云环境，可通过本地和基于 Azure 的环境或资源进行访问。 Azure ExpressRoute 和 VPN 连接等服务可提供连接。 这些服务需要特定的网络地址范围和防火墙端口才能启用。

部署私有云时，会创建用于管理、预配和 vMotion 的专用网络。 可使用这些专用网络来访问 vCenter 和 NSX-T 管理器，还可访问虚拟机 vMotion 或部署。  ExpressRoute Global Reach 用于将私有云连接到本地环境。 要实现该连接，你的订阅中必须存在带有 ExpressRoute 线路的虚拟网络。



>[!NOTE]
>会预配和提供对 Internet 和 Azure 服务的访问，从而在部署私有云时在生产网络中使用虚拟机 (VM)。  默认情况下，会对新的私有云禁用 Internet 访问权限，且随时都可启用或禁用此权限。
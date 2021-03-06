---
title: Azure 服务中断影响 Azure Key Vault 时该怎么办 - Azure Key Vault | Microsoft Docs
description: 了解 Azure 服务中断影响 Azure Key Vault 时该怎么办。
services: key-vault
author: ShaneBala-keyvault
manager: ravijan
ms.service: key-vault
ms.subservice: general
ms.topic: tutorial
ms.date: 08/28/2020
ms.author: sudbalas
ms.openlocfilehash: 27d8d4de308fe7cf6e6f36dd33f33bb73c495073
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/22/2020
ms.locfileid: "90983229"
---
# <a name="azure-key-vault-availability-and-redundancy"></a>Azure 密钥保管库可用性和冗余

Azure 密钥保管库具有多层冗余功能，确保密钥和机密持续可供应用程序使用，即使服务的单个组件发生故障也是如此。

> [!NOTE]
> 本指南适用于保管库。 托管 HSM 池使用其他高可用性和灾难恢复模型。 要了解详细信息，请参阅[有关托管 HSM 灾难恢复的指南](../managed-hsm/disaster-recovery-guide.md)。

密钥保管库的内容会在区域中复制，并且会复制到至少 150 英里以外（但位于同一个地理位置）的次要区域，以保持密钥和机密的高持续性。 有关特定区域对的详细信息，请参阅 [Azure 配对区域](../../best-practices-availability-paired-regions.md)一文。


如果密钥保管库服务中的单独组件发生故障，则区域内的替代组件将继续处理请求，确保不会导致功能损失。 要开始此过程，你无需执行任何操作；此过程以透明的方式自动发生。

在整个 Azure 区域不可用的情况下（这很少见），对该区域中的 Azure 密钥保管库发出的请求会自动路由（“故障转移”  ）到次要区域。 当主要区域再次可用时，请求将路由回（“故障回复”  ）到主要区域。 同样，不需要采取任何措施，因为这会自动发生。

通过这种高可用性设计，Azure 密钥保管库不需要停机进行维护活动。

应注意以下几个事项：

* 发生区域故障转移时，可能需要等待几分钟让服务故障转移。 在故障转移之前的这段时间内发出的请求可能会失败。
* 在故障转移期间，密钥保管库处于只读模式。 在此模式下支持的请求包括：
  * 列出证书
  * 获取证书
  * 列出机密
  * 获取机密
  * 列出密钥
  * 获取密钥（属性）
  * 加密
  * 解密
  * 包装
  * 解包
  * 验证
  * 签名
  * 备份

* 在故障转移期间，无法更改密钥保管库属性。 不能更改访问策略或防火墙配置和设置。

* 故障回复之后，所有请求类型（包括读取*和*写入请求）都将可用。

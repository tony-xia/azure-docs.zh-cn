---
title: Azure AD 中的 Azure AD Connect 云预配先决条件
description: 本文介绍云预配所需的先决条件和硬件要求。
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 12/06/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 7cf072ae9544cd479aeca02d9b9fcd670b8eb5fe
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/01/2020
ms.locfileid: "89226890"
---
# <a name="prerequisites-for-azure-ad-connect-cloud-provisioning"></a>Azure AD Connect 云预配先决条件
本文指导如何选择 Azure Active Directory (Azure AD) Connect 云预配并将其作为标识解决方案。



## <a name="cloud-provisioning-agent-requirements"></a>云预配代理要求
需要以下内容才能使用 Azure AD Connect 云预配：
    
- 不是来宾用户的 Azure AD 租户的混合标识管理员帐户。
- 使用 Windows 2012 R2 或更高版本的预配代理的本地服务器。  此服务器应该是基于 [Active Directory 管理层模型](/windows-server/identity/securing-privileged-access/securing-privileged-access-reference-material)的第0层服务器。
- 本地防火墙配置。

>[!NOTE]
>当前只能在英文版服务器上安装预配代理。 在非英文版服务器上安装英文语言包不是一个有效的解决方法，并且会导致代理安装失败。 

本文档的其余部分提供了有关这些先决条件的分步说明。

### <a name="in-the-azure-active-directory-admin-center"></a>在 Azure Active Directory 管理中心中

1. 在 Azure AD 租户上创建仅限云的混合标识管理员帐户。 这样一来，就可以在本地服务出现故障或不可用时管理租户的配置。 了解如何 [添加仅限云的混合标识管理员帐户](../fundamentals/add-users-azure-active-directory.md)。 完成此步骤至关重要，可确保自己不被锁定在租户外部。
1. 在 Azure AD 租户中添加一个或多个[自定义域名](../fundamentals/add-custom-domain.md)。 用户可以使用其中一个域名登录。

### <a name="in-your-directory-in-active-directory"></a>在 Active Directory 的目录中

运行 [IdFix 工具](/office365/enterprise/prepare-directory-attributes-for-synch-with-idfix)，为同步准备目录属性。

### <a name="in-your-on-premises-environment"></a>在本地环境中

1. 指定一台已加入域的、运行 Windows Server 2012 R2 或更高版本、至少有 4-GB RAM 且装有 .NET 4.7.1+ 运行时的主机服务器。

1. 本地服务器上的 PowerShell 执行策略必须设置为 Undefined 或 RemoteSigned。

1. 如果服务器和 Azure AD 之间存在防火墙，请配置以下项：
   - 确保代理可以通过以下端口向 Azure AD 发出出站请求：

        | 端口号 | 用途 |
        | --- | --- |
        | **80** | 下载证书吊销列表 (CRL) 的同时验证 TLS/SSL 证书。  |
        | **443** | 处理与服务的所有出站通信。 |
        | **8080**（可选） | 如果端口 443 不可用，代理将每隔 10 分钟通过端口 8080 报告其状态。 此状态显示在 Azure AD 门户上。 |
     
   - 如果防火墙根据原始用户强制实施规则，请打开这些端口以允许来自作为网络服务运行的 Windows 服务的流量。
   - 如果防火墙或代理允许指定安全后缀，请将连接添加到 \*.msappproxy.net 和 \*.servicebus.windows.net。 否则，请允许访问每周更新的 [Azure 数据中心 IP 范围](https://www.microsoft.com/download/details.aspx?id=41653)。
   - 代理需要访问 login.windows.net 和 login.microsoftonline.com 来完成初始注册。 另外，还请为这些 URL 打开防火墙。
   - 为了进行证书验证，请取消阻止以下 URL：mscrl.microsoft.com:80、crl.microsoft.com:80、ocsp.msocsp.com:80 和 www\.microsoft.com:80。 这些 URL 与其他 Microsoft 产品一起用于证书验证，因此可能已取消阻止这些 URL。

>[!NOTE]
> 不支持在 Windows Server Core 上安装云预配代理。


### <a name="additional-requirements"></a>其他需求
- [Microsoft .NET Framework 4.7.1](https://www.microsoft.com/download/details.aspx?id=56116) 

#### <a name="tls-requirements"></a>TLS 要求

>[!NOTE]
>传输层安全性 (TLS) 是为进行安全通信提供的协议。 更改 TLS 设置会影响整个林。 有关详细信息，请参阅[启用 TLS 1.1 和 TLS 1.2 作为 Windows 的 WinHTTP 中的默认安全协议的更新](https://support.microsoft.com/help/3140245/update-to-enable-tls-1-1-and-tls-1-2-as-default-secure-protocols-in-wi)。

托管 Azure AD Connect 云预配代理的 Windows 服务器必须先启用 TLS 1.2，然后才能安装它。

若要启用 TLS 1.2，请执行下列步骤。

1. 设置以下注册表项：
    
    ```
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2]
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client] "DisabledByDefault"=dword:00000000 "Enabled"=dword:00000001
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server] "DisabledByDefault"=dword:00000000 "Enabled"=dword:00000001
    [HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\v4.0.30319] "SchUseStrongCrypto"=dword:00000001
    ```

1. 重新启动服务器。


## <a name="next-steps"></a>后续步骤 

- [什么是预配？](what-is-provisioning.md)
- [什么是 Azure AD Connect 云预配？](what-is-cloud-provisioning.md)
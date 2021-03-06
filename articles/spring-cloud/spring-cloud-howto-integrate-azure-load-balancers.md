---
title: 教程-将 Azure 春季 Cloud 与 Azure 负载均衡解决方案集成
description: 如何将 Azure 春季 Cloud 与 Azure 负载均衡解决方案集成
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: how-to
ms.date: 04/20/2020
ms.custom: devx-track-java
ms.openlocfilehash: 02772f153cdda7e3f3c866c727d589e755e19033
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/22/2020
ms.locfileid: "90906928"
---
# <a name="integrate-azure-spring-cloud-with-azure-load-balance-solutions"></a>将 Azure Spring Cloud 与 Azure 负载均衡解决方案集成

**本文适用于：** ✔️ Java ✔️ C#

Azure 春季云在 Azure 上支持微服务。  提高企业可能需要多个数据中心，管理多个 Azure 春季云实例。

Azure 已提供不同的负载均衡解决方案。 有三个选项可用于将 Azure 春季云与 Azure 负载均衡解决方案集成：

1.  将 Azure 春季云与 Azure 流量管理器集成
2.  将 Azure 春季 Cloud 与 Azure 应用网关集成
3.  将 Azure 春季云与 Azure 前门集成

## <a name="prerequisites"></a>必备知识

* Azure 春季云： [如何创建 azure 春季云服务](https://docs.microsoft.com/azure/spring-cloud/spring-cloud-quickstart)
* Azure 流量管理器： [如何创建流量管理器](https://docs.microsoft.com/azure/traffic-manager/quickstart-create-traffic-manager-profile/)
* Azure 应用网关： [如何创建应用程序网关](https://docs.microsoft.com/azure/application-gateway/quick-create-portal)
* Azure 前门： [如何创建前门](https://docs.microsoft.com/azure/frontdoor/quickstart-create-front-door)

## <a name="integrate-azure-spring-cloud-with-azure-traffic-manager"></a>将 Azure 春季云与 Azure 流量管理器集成

若要将 Azure 春季云与流量管理器集成，请将其公共终结点添加为流量管理器的终结点，然后为流量管理器和 Azure 春季云配置自定义域。

### <a name="add-endpoint-in-traffic-manager"></a>在流量管理器中添加终结点
在流量管理器中添加终结点：
1.  指定要为*外部终结点*的**类型**。
1.  输入每个 Azure 春季 cloud 公共端点 (FQDN) 的完全限定的域名。
1. 单击“确定”。 

    ![流量管理器 1 ](media/spring-cloud-load-balancers/traffic-manager-1.png) ![ 流量管理器2](media/spring-cloud-load-balancers/traffic-manager-2.png)

### <a name="configure-custom-domain"></a>配置自定义域
完成配置：
1.  登录到域提供商的网站，并创建从自定义域到流量管理器的 Azure 默认域名的 CNAME 记录映射。
1.  按照说明 [如何将自定义域添加到 Azure 春季 Cloud](spring-cloud-tutorial-custom-domain.md)。
1. 将上面的自定义域绑定添加到流量管理器，将其添加到 Azure 弹簧 cloud 相应的应用服务，并在此处上传 SSL 证书

    ![流量管理器3](media/spring-cloud-load-balancers/traffic-manager-3.png)

## <a name="integrate-azure-spring-cloud-with-azure-app-gateway"></a>将 Azure 春季 Cloud 与 Azure 应用网关集成

要与 Azure 春季云服务集成，请完成以下配置：

### <a name="configure-backend-pool"></a>配置后端池
1. 将 **目标类型** 指定为 *IP 地址* 或 *FQDN*。
1. 输入 Azure 春季 cloud 公共终结点。

    ![应用程序网关1](media/spring-cloud-load-balancers/app-gateway-1.png)

### <a name="add-custom-probe"></a>添加自定义探测
1. 选择 " **运行状况探测** "，然后单击 " **添加** " 以打开自定义 **探测** 对话框。 
1. 关键点是**从后端 HTTP 设置选项中**选择 *"是" 作为 "* 选取主机名"。

    ![应用网关2](media/spring-cloud-load-balancers/app-gateway-2.png)

### <a name="configure-http-setting"></a>配置 Http 设置
1.  选择 " **Http 设置** "，然后单击 " **添加** " 以添加 Http 设置。
1.  替换**为新的主机名：** 选择 *"是"*。
1.  **主机名替代**： **从后端目标选择 "选取主机名**"。
1.  **使用自定义探测**：选择 *"是"* ，并选择上面创建的自定义探测。

    ![应用网关3](media/spring-cloud-load-balancers/app-gateway-3.png)

## <a name="integrate-azure-spring-cloud-with-azure-front-door"></a>将 Azure 春季云与 Azure 前门集成

与 Azure 春季云服务集成并配置后端池。 
1. **添加后端池**。
1. 通过添加主机指定后端终结点。

    ![前门1](media/spring-cloud-load-balancers/front-door-1.png)

1.  将 **后端主机类型** 指定为 *自定义主机*。
1.  在 **后端主机名称**中输入 Azure 春季 Cloud 公共终结点的 FQDN。
1.  接受 **后端主机标头** ，默认值与 **后端主机名**相同。

    ![前门2](media/spring-cloud-load-balancers/front-door-2.png)

## <a name="next-steps"></a>后续步骤
* [如何创建流量管理器](https://docs.microsoft.com/azure/traffic-manager/quickstart-create-traffic-manager-profile/)
* [如何创建应用程序网关](https://docs.microsoft.com/azure/application-gateway/quick-create-portal)
* [如何创建前门](https://docs.microsoft.com/azure/frontdoor/quickstart-create-front-door)

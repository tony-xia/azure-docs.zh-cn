---
title: Azure Service Fabric-配置容器存储凭据
description: 配置存储库凭据以从容器注册表下载映像
ms.topic: conceptual
ms.date: 12/09/2019
ms.custom: sfrev
ms.openlocfilehash: 142ede6fcc59063d83854712a966a90c7472923b
ms.sourcegitcommit: 9c262672c388440810464bb7f8bcc9a5c48fa326
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/03/2020
ms.locfileid: "89421418"
---
# <a name="configure-repository-credentials-for-your-application-to-download-container-images"></a>配置应用程序的存储库凭据以下载容器映像

通过将添加 `RepositoryCredentials` 到应用程序清单的部分来配置容器注册表身份验证 `ContainerHostPolicies` 。 在以下示例中，为容器注册表 (*myregistry.azurecr.io* 添加帐户和密码) ，这允许服务从存储库下载容器映像。

```xml
<ServiceManifestImport>
    ...
    <Policies>
        <ContainerHostPolicies CodePackageRef="Code">
            <RepositoryCredentials AccountName="myregistry" Password="=P==/==/=8=/=+u4lyOB=+=nWzEeRfF=" PasswordEncrypted="false"/>
            <PortBinding ContainerPort="80" EndpointRef="Guest1TypeEndpoint"/>
        </ContainerHostPolicies>
    </Policies>
    ...
</ServiceManifestImport>
```

建议使用部署到群集所有节点的加密证书来加密存储库密码。 当 Service Fabric 将服务包部署到群集时，即可使用加密证书解密密码文本。 Invoke-ServiceFabricEncryptText cmdlet 用于为密码创建密码文本，后者将添加到 ApplicationManifest.xml 文件中。
有关证书和加密语义的详细信息，请参阅 [机密管理](service-fabric-application-secret-management.md) 。

## <a name="configure-cluster-wide-credentials"></a>配置群集级凭据

Service Fabric 允许你配置群集范围的凭据，这些凭据可由应用程序用作默认存储库凭据。

可以通过将 `UseDefaultRepositoryCredentials` 属性添加到 `ContainerHostPolicies` ApplicationManifest.xml 中的 `true` 或值来启用或禁用此功能 `false` 。

```xml
<ServiceManifestImport>
    ...
    <Policies>
        <ContainerHostPolicies CodePackageRef="Code" UseDefaultRepositoryCredentials="true">
            <PortBinding ContainerPort="80" EndpointRef="Guest1TypeEndpoint"/>
        </ContainerHostPolicies>
    </Policies>
    ...
</ServiceManifestImport>
```

然后 Service Fabric 使用默认存储库凭据，这些凭据可在部分下的 Clustermanifest.xml 中指定 `Hosting` 。  如果 `UseDefaultRepositoryCredentials` 为 `true`，则 Service Fabric 将从 ClusterManifest 中读取以下值：

* DefaultContainerRepositoryAccountName (string)
* DefaultContainerRepositoryPassword (string)
* IsDefaultContainerRepositoryPasswordEncrypted (bool)
* DefaultContainerRepositoryPasswordType (字符串) 

下面是可以在文件的 ClusterManifestTemplate.js中的部分内添加内容的示例 `Hosting` 。 可以在群集创建时或配置升级后期添加 `Hosting` 节。 有关详细信息，请参阅[更改 Azure Service Fabric 群集设置](service-fabric-cluster-fabric-settings.md)和[管理 Azure Service Fabric 应用程序机密](service-fabric-application-secret-management.md)

```json
"fabricSettings": [
    ...,
    {
        "name": "Hosting",
        "parameters": [
          {
            "name": "EndpointProviderEnabled",
            "value": "true"
          },
          {
            "name": "DefaultContainerRepositoryAccountName",
            "value": "someusername"
          },
          {
            "name": "DefaultContainerRepositoryPassword",
            "value": "somepassword"
          },
          {
            "name": "IsDefaultContainerRepositoryPasswordEncrypted",
            "value": "false"
          },
          {
            "name": "DefaultContainerRepositoryPasswordType",
            "value": "PlainText"
          },
          {
        "name": "DefaultMSIEndpointForTokenAuthentication",
        "value": "URI"
          }
        ]
      },
]
```

## <a name="use-tokens-as-registry-credentials"></a>使用令牌作为注册表凭据

Service Fabric 支持使用令牌作为凭据为容器下载映像。  此功能利用底层虚拟机规模集的 *托管标识* 对注册表进行身份验证，从而无需管理用户凭据。  有关详细信息，请参阅 [Azure 资源的托管标识](../active-directory/managed-identities-azure-resources/overview.md) 。  使用此功能需要执行以下步骤：

1. 确保为 VM 启用 *系统分配的托管标识* 。

    ![Azure 门户：创建虚拟机规模集标识选项](./media/configure-container-repository-credentials/configure-container-repository-credentials-acr-iam.png)

2. 向虚拟机规模集授予对注册表的请求/读取映像的权限。 从 Azure 门户中 Azure 容器注册表的 "访问控制" ("IAM) " 边栏选项卡中，为虚拟机添加 *角色分配* ：

    ![将 VM 主体添加到 ACR](./media/configure-container-repository-credentials/configure-container-repository-credentials-vmss-identity.png)

3. 接下来，修改应用程序清单。 在 `ContainerHostPolicies` 部分中，添加属性 `‘UseTokenAuthenticationCredentials=”true”` 。

    ```xml
      <ServiceManifestImport>
          <ServiceManifestRef ServiceManifestName="NodeServicePackage" ServiceManifestVersion="1.0"/>
      <Policies>
        <ContainerHostPolicies CodePackageRef="NodeService.Code" Isolation="process" UseTokenAuthenticationCredentials="true">
          <PortBinding ContainerPort="8905" EndpointRef="Endpoint1"/>
        </ContainerHostPolicies>
        <ResourceGovernancePolicy CodePackageRef="NodeService.Code" MemoryInMB="256"/>
      </Policies>
      </ServiceManifestImport>
    ```

    > [!NOTE]
    > 如果为 true，则将标志 `UseDefaultRepositoryCredentials` 设置为 true `UseTokenAuthenticationCredentials` 会在部署过程中导致错误。

### <a name="using-token-credentials-outside-of-azure-global-cloud"></a>使用 Azure Global Cloud 之外的令牌凭据

使用基于令牌的注册表凭据时，Service Fabric 会代表虚拟机获取一个令牌，并将其提供给 ACR。 默认情况下，Service Fabric 会请求一个令牌，该令牌的受众是全局 Azure 云终结点。 如果要部署到另一个云实例，如 Azure 德国或 Azure 政府版，则需要替代参数的默认值 `DefaultMSIEndpointForTokenAuthentication` 。 如果不部署到特殊环境，请不要重写此参数。 如果是，则将替换默认值，该默认值为

```
http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.core.windows.net/
```

适用于环境的相应资源终结点。 例如，对于 [Azure 德国](https://docs.microsoft.com/azure/germany/germany-developer-guide#endpoint-mapping)，替代将是 

```json
{
    "name": "DefaultMSIEndpointForTokenAuthentication",
    "value": "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.core.cloudapi.de/"
}
```

[详细了解如何获取虚拟机规模集令牌](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/how-to-use-vm-token)。

## <a name="next-steps"></a>后续步骤

* 查看有关 [容器注册表身份验证](../container-registry/container-registry-authentication.md)的详细信息。

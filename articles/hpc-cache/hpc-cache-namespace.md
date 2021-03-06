---
title: 了解 Azure HPC 缓存聚合命名空间
description: 如何为 Azure HPC 缓存规划虚拟命名空间
author: ekpgh
ms.service: hpc-cache
ms.topic: how-to
ms.date: 09/30/2020
ms.author: v-erkel
ms.openlocfilehash: 1c28f549cf93d77f6aef6bcde6a2225345a79cc9
ms.sourcegitcommit: 06ba80dae4f4be9fdf86eb02b7bc71927d5671d3
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/01/2020
ms.locfileid: "91612942"
---
# <a name="plan-the-aggregated-namespace"></a>规划聚合命名空间

Azure HPC 缓存允许客户端通过虚拟命名空间访问各种存储系统，该命名空间隐藏后端存储系统的详细信息。

添加存储目标后，为存储目标设置一个或多个面向客户端的命名空间路径。 客户端计算机安装此文件路径，并可以将文件读取请求发送到缓存，而不是直接装载存储系统。

由于 Azure HPC 缓存管理此虚拟文件系统，因此你可以更改存储目标，而无需更改面向客户端的路径。 例如，你可以将硬件存储系统替换为云存储，而无需重写客户端过程。

## <a name="aggregated-namespace-example"></a>聚合命名空间示例

规划聚合命名空间，以便客户端计算机可以方便地访问所需的信息，以便管理员和工作流工程师可以轻松地区分路径。

例如，假设有一个系统使用 Azure HPC 缓存实例来处理存储在 Azure Blob 中的数据。 此分析需要存储在本地数据中心的模板文件。

模板数据存储在数据中心中，此作业所需的信息存储在这些子目录中：

* */goldline/templates/acme2017/sku798*
* */goldline/templates/acme2017/sku980*

数据中心存储系统将公开以下导出：

* */*
* */goldline*
* */goldline/templates*

使用 [CLFSLoad 实用程序](hpc-cache-ingest.md#pre-load-data-in-blob-storage-with-clfsload)将要分析的数据复制到名为 "sourcecollection 进行" 的 Azure Blob 存储容器。

若要允许通过缓存轻松访问，请考虑使用以下虚拟命名空间路径创建存储目标：

| 后端存储系统 <br/>  (NFS 文件路径或 Blob 容器)  | 虚拟命名空间路径 |
|-----------------------------------------|------------------------|
| /goldline/templates/acme2017/sku798     | /templates/sku798      |
| /goldline/templates/acme2017/sku980     | /templates/sku980      |
| sourcecollection 进行                        | /source               |

NFS 存储目标可以有多个虚拟命名空间路径，只要每个路径引用一个唯一的导出路径。  (读取 [nfs 命名空间路径](add-namespace-paths.md#nfs-namespace-paths) ，以了解建议的每个 NFS 存储目标最大命名空间路径数。 ) 

由于 NFS 源路径是相同导出的子目录，因此需要从同一存储目标定义多个命名空间路径。

| 存储目标主机名  | NFS 导出路径     | 子目录路径 | 命名空间路径    |
|--------------------------|---------------------|-------------------|-------------------|
| *IP 地址或主机名* | /goldline/templates | acme2017/sku798   | /templates/sku798 |
| *IP 地址或主机名* | /goldline/templates | acme2017/sku980   | /templates/sku980 |

客户端应用程序可以装载缓存并轻松访问聚合的命名空间文件路径 ``/source`` 、 ``/templates/sku798`` 和 ``/templates/sku980`` 。

另一种方法是创建类似 `/templates` 于父目录的虚拟路径， `acme2017` 然后在装入缓存后让客户端导航到各个 `sku798` 和 `sku980` 目录。 但是，不能创建作为另一个命名空间路径的子目录的命名空间路径。 因此，如果创建目录的路径 `acme2017` ，也不能创建任何命名空间路径直接访问其子目录。

"Azure HPC 缓存 **命名空间** 设置" 页显示面向客户端的文件系统，并允许你添加或编辑路径。 有关详细信息，请参阅 [设置聚合命名空间](add-namespace-paths.md) 。

## <a name="next-steps"></a>后续步骤

确定如何设置虚拟文件系统后，请执行以下步骤来创建它：

* [创建存储目标](hpc-cache-add-storage.md) ，将后端存储系统添加到 Azure HPC 缓存
* [添加命名空间路径](add-namespace-paths.md) ，以创建客户端计算机用于访问文件的聚合命名空间

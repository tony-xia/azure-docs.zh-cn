---
title: 排查 Azure SQL Edge 部署问题
description: 了解部署 Azure SQL Edge 时可能出现的错误
keywords: SQL 边缘，故障排除，部署错误
services: sql-edge
ms.service: sql-edge
ms.topic: troubleshooting
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 09/22/2020
ms.openlocfilehash: 517fed0dd9eb1736344546bde9f79e52ee17182f
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/25/2020
ms.locfileid: "91333097"
---
# <a name="troubleshooting-azure-sql-edge-deployments"></a>排查 Azure SQL Edge 部署问题 

本文提供了有关部署和使用 Azure SQL Edge 容器时可能出现的错误的信息，并提供故障排除技巧以帮助解决这些问题。 

Azure SQL Edge 支持两种部署模型： 
- 通过 Azure IoT Edge 连接的部署： azure SQL Edge 在 Azure Marketplace 中提供，可以部署为 [Azure IoT Edge](../iot-edge/about-iot-edge.md)的模块。 有关详细信息，请参阅[部署 Azure SQL Edge](deploy-portal.md)。<br>

- 断开连接的部署：可从 docker 中心拉取 Azure SQL Edge 容器映像，并将其部署为独立的 docker 容器或 kubernetes 群集。 有关详细信息，请参阅 [使用 Docker 部署 AZURE Sql edge](disconnected-deployment.md) 和 [在 Kubernetes 中部署 azure sql edge 容器](deploy-kubernetes.md)。

## <a name="troubleshooting-iot-edge-device-and-deployments"></a>IoT Edge 设备和部署的疑难解答

如果通过 Azure IoT Edge 部署 SQL Edge 时出现错误，请确保 `iotedge` 已正确配置并运行该服务。 以下文档可帮助解决与 Azure IoT Edge 相关的问题：
- [Azure IoT Edge 的常见问题和解决方法](../iot-edge/troubleshoot-common-errors.md)。
- [排除 IoT Edge 设备故障](../iot-edge/troubleshoot.md)

## <a name="docker-command-errors"></a>Docker 命令错误

如果遇到任何 `docker` 命令错误，请确保 Docker 服务正在运行，并尝试通过提升的权限运行。

例如，在 Linux 上运行 `docker` 命令时可能会遇到以下错误：

```output
Cannot connect to the Docker daemon. Is the docker daemon running on this host?
```

如果在 Linux 上遇到此错误，请尝试对相同的命令加上 `sudo` 前缀，然后再运行。 如果失败，请验证 Docker 服务是否正在运行；如果没有，请启动它。

```bash
sudo systemctl status docker
sudo systemctl start docker
```

在 Windows 上，验证并确保以管理员身份启动 PowerShell 或命令提示符。

## <a name="azure-sql-edge-container-startup-errors"></a>Azure SQL Edge 容器启动错误

如果 SQL Edge 容器无法运行，请尝试以下测试：

- 如果使用 Azure IoT Edge，请确保模块映像已成功下载，并且在模块清单中正确指定了环境变量和容器创建选项。

- 如果使用的是基于 docker 或 kubernetes 的部署，请确保 `docker run` 命令格式正确。 有关详细信息，请参阅 [使用 Docker 部署 AZURE Sql edge](disconnected-deployment.md) 和 [在 Kubernetes 中部署 azure sql edge 容器](deploy-kubernetes.md)。

- 如果收到 `failed to create endpoint CONTAINER_NAME on network bridge. Error starting proxy: listen tcp 0.0.0.0:1433 bind: address already in use.` 之类的错误，则尝试将容器端口 1433 映射到已在使用的端口。 如果在主计算机上本地运行 SQL Edge，就会发生这种情况。 如果启动两个 SQL 边缘容器，并尝试将它们映射到同一个主机端口，也可能会发生这种情况。 如果发生此错误，请使用 `-p` 参数将容器端口 1433 映射到不同的主机端口。 例如： 

    ```bash
    sudo docker run --cap-add SYS_PTRACE -e 'ACCEPT_EULA=1' -e 'MSSQL_SA_PASSWORD=yourStrong(!)Password' -p 1433:1433 --name azuresqledge -d mcr.microsoft.com/azure-sql-edge-developer.
    ```

- 如果在尝试启动容器时收到 `Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.30tdout=1&tail=all: dial unix /var/run/docker.sock: connect: permission denied` 之类的错误，请将用户添加到 Ubuntu 中的 Docker 组。 然后注销并重新登录，因为此更改将影响新会话。 

   ```bash
    usermod -aG docker $USER
   ```

- 检查是否有任何来自容器的错误消息。

   ```bash
   docker logs e69e056c702d
   ```

- 如果使用任何容器管理软件，请确保它支持以根身份运行的容器进程。 容器中的 sqlservr 进程以根身份运行。

- 默认情况下，Azure SQL Edge 容器以名为的非根用户身份运行 `mssql` 。 如果使用装入点或数据卷来持久保存数据，请确保 `mssql` 用户对卷具有适当的权限。 有关详细信息，请参阅以 [非超级用户身份运行](configure.md#run-azure-sql-edge-as-non-root-user) 和 [保留数据](configure.md#persist-your-data)。

- 如果 SQL Edge Docker 容器在启动后立即退出，请检查 Docker 日志。 如果通过 `docker run` 命令使用 Windows 上的 PowerShell，请使用双引号而不是单引号。 对于 PowerShell Core 时，请使用单引号。

- 查看 [SQL 边缘错误日志](#errorlogs)。

## <a name="sql-edge-connection-failures"></a>SQL 边缘连接失败

如果无法连接到在容器中运行的 SQL Edge 实例，请尝试以下测试：

- 查看输出的 " **状态** " 列，确保 SQL Edge 容器正在运行 `docker ps -a` 。 如果未在运行，则使用 `docker start <Container ID>` 启动它。

- 如果映射到非默认主机端口（非 1433），请确保在连接字符串中指定端口。 可在 `docker ps -a` 输出的“端口”列中查看端口映射。 有关连接到 Azure SQL Edge 的详细信息，请参阅 [连接和查询 AZURE Sql edge](connect.md)

- 如果以前使用映射的数据卷或数据卷容器部署了 SQL 边缘，并且现在使用现有的映射数据卷或数据卷容器，则 SQL Edge 将忽略 `MSSQL_SA_PASSWORD` 环境变量的值。 而是使用以前配置的 SA 用户密码。 出现这种情况的原因是： SQL Edge 重用映射的卷或数据卷容器中的现有主数据库文件。 如果遇到此问题，可以使用以下选项：

    - 如果仍可用，请使用以前使用过的密码进行连接。
    - 将 SQL Edge 配置为使用不同的映射卷或数据卷容器。
    - 从映射的卷或数据卷容器 (master .mdf 和 mastlog.ldf) 中删除现有的 master 数据库文件。

- 查看 [SQL 边缘错误日志](#errorlogs)。

## <a name="sql-edge-setup-and-error-logs"></a><a id="errorlogs"></a> SQL Edge 设置和错误日志

默认情况下，SQL Edge 错误日志存在于容器中的 **/var/opt/mssql/log** 目录中，可以使用以下任一方式进行访问：

- 如果在创建容器时将主机目录装载到了 **/var/opt/mssql**，可改为在主机上映射路径的 **log** 子目录中进行查找。
- 使用交互式命令提示符连接到容器。 如果容器未在运行，请先启动它。 然后使用交互式命令提示符来检查日志。 可以通过运行 `docker ps` 命令来获取容器 ID。

    ```bash
    docker start <ContainerID>
    docker exec -it <ContainerID> "/bin/bash"
    ```

    从容器内的 bash 会话运行以下命令：

    ```bash
    cd /var/opt/mssql/log
    cat errorlog
    ```
- 如果 SQL 边缘容器已启动并正在运行，并且您可以使用客户端工具连接到该实例，则可以使用该存储过程 `sp_readerrorlog` 来读取 SQL 边缘错误日志的内容。

## <a name="execute-commands-in-a-container"></a>在容器中执行命令

如果正在运行容器，则可从主机终端在容器内部执行命令。

若要获取容器 ID，请运行：

```bash
docker ps -a
```

若要在容器中启动 Bash 终端，请运行：

```bash
docker exec -it <Container ID> /bin/bash
```

现在，可以像在容器内的终端上那样运行这些命令。 完成后，键入 `exit`。 这将退出交互式命令会话，但容器会继续运行。

### <a name="enabling-verbose-logging"></a>启用详细日志记录

如果流式处理引擎的默认日志级别没有提供足够的信息，则可以在 SQL Edge 中启用流式处理引擎的调试日志记录。 若要启用调试日志记录，请将 `RuntimeLogLevel=debug` 环境变量添加到 SQL Edge 部署。 启用调试日志记录后，尝试重现该问题，并检查日志中是否有任何相关的消息或异常。 

> [!NOTE]
> "详细日志记录" 选项应仅用于故障排除，不适用于常规生产工作负荷。 


## <a name="next-steps"></a>后续步骤

- [SQL Edge 中的 ONNX 与的智能机器学习](onnx-overview.md)
- [Azure SQL Edge 中的数据流式处理](stream-data.md)
- [数据保留和清理](data-retention-overview.md)
- [填充时间间隔和输入缺失值](imputing-missing-values.md)








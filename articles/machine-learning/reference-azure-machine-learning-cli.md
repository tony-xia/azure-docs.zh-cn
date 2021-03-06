---
title: 安装和使用 Azure 机器学习 CLI
description: 了解如何安装和使用 Azure 机器学习 CLI 扩展来创建和管理资源，例如工作区、数据存储、数据集、管道、模型和部署。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
ms.reviewer: jmartens
ms.author: jordane
author: jpe316
ms.date: 06/22/2020
ms.custom: seodec18
ms.openlocfilehash: 50ac8ede9f8272d6956d1ac29e57a9c26d2c0bd9
ms.sourcegitcommit: 19dce034650c654b656f44aab44de0c7a8bd7efe
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/04/2020
ms.locfileid: "91708265"
---
# <a name="install--use-the-cli-extension-for-azure-machine-learning"></a>安装和使用 Azure 机器学习的 CLI 扩展


Azure 机器学习 CLI 是 [Azure CLI](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest&preserve-view=true)（适用于 Azure 平台的跨平台命令行接口）的一个扩展。 此扩展提供用于操作 Azure 机器学习的命令。 通过此扩展可将机器学习活动自动化。 以下列表提供了可以使用 CLI 扩展执行的一些示例操作：

+ 运行试验以创建机器学习模型

+ 注册机器学习模型以供客户使用

+ 打包、部署和跟踪机器学习模型的生命周期

CLI 不能取代 Azure 机器学习 SDK。 它是一个经过优化的补充工具，可以处理高度参数化的、非常适合自动化的任务。

## <a name="prerequisites"></a>先决条件

* 若要使用 CLI，必须拥有 Azure 订阅。 如果没有 Azure 订阅，请在开始操作前先创建一个免费帐户。 立即试用[免费版或付费版 Azure 机器学习](https://aka.ms/AMLFree)。

* 若要在**本地环境**中使用本文档所述的 CLI 命令，需要安装 [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest&preserve-view=true)。

    如果使用的是 [Azure Cloud Shell](https://azure.microsoft.com//features/cloud-shell/)，则可通过浏览器访问 CLI（它位于云端）。

## <a name="full-reference-docs"></a>完整参考文档

查找 [Azure CLI 的 azure-cli-ml 扩展的完整参考文档](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/?view=azure-cli-latest&preserve-view=true)。

## <a name="connect-the-cli-to-your-azure-subscription"></a>将 CLI 连接到 Azure 订阅

> [!IMPORTANT]
> 如果使用的是 Azure Cloud Shell，可跳过此部分。 Cloud Shell 会使用你登录 Azure 订阅所用的帐户自动对你进行身份验证。

可使用多种方法通过 CLI 对 Azure 订阅进行身份验证。 最基本的方法是使用浏览器进行交互式身份验证。 若要以交互方式进行身份验证，请打开命令行或终端，然后使用以下命令：

```azurecli-interactive
az login
```

如果 CLI 可以打开默认的浏览器，则它会打开该浏览器并加载登录页。 否则，需要打开浏览器并按照命令行中的说明操作。 按照说明操作时，需要浏览到 [https://aka.ms/devicelogin](https://aka.ms/devicelogin) 并输入授权代码。

[!INCLUDE [select-subscription](../../includes/machine-learning-cli-subscription.md)]

有关其他身份验证方法，请参阅[使用 Azure CLI 登录](https://docs.microsoft.com/cli/azure/authenticate-azure-cli?view=azure-cli-latest&preserve-view=true)。

## <a name="install-the-extension"></a>安装扩展

若要安装机器学习 CLI 扩展，请使用以下命令：

```azurecli-interactive
az extension add -n azure-cli-ml
```

> [!TIP]
> 可以在[此处](https://aka.ms/azml-deploy-cloud)找到可与以下命令配合使用的示例文件。

出现提示时，选择 `y` 安装该扩展。

若要验证是否已安装该扩展，请使用以下命令显示特定于机器学习的子命令列表：

```azurecli-interactive
az ml -h
```

## <a name="update-the-extension"></a>更新扩展

若要更新机器学习 CLI 扩展，请使用以下命令：

```azurecli-interactive
az extension update -n azure-cli-ml
```


## <a name="remove-the-extension"></a>删除扩展

若要删除 CLI 扩展，请使用以下命令：

```azurecli-interactive
az extension remove -n azure-cli-ml
```

## <a name="resource-management"></a>资源管理

以下命令演示如何使用 CLI 来管理 Azure 机器学习所用的资源。

+ 如果没有资源组，请创建一个资源组：

    ```azurecli-interactive
    az group create -n myresourcegroup -l westus2
    ```

+ 创建 Azure 机器学习工作区：

    ```azurecli-interactive
    az ml workspace create -w myworkspace -g myresourcegroup
    ```

    有关详细信息，请参阅 [az ml workspace create](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/workspace?view=azure-cli-latest&preserve-view=true#ext-azure-cli-ml-az-ml-workspace-create)。

+ 将工作区配置附加到某个文件夹，以启用 CLI 上下文感知。

    ```azurecli-interactive
    az ml folder attach -w myworkspace -g myresourcegroup
    ```

    此命令创建包含示例 runconfig 和 conda 环境文件的 `.azureml` 子目录。 此子目录还包含用来与 Azure 机器学习工作区通信的 `config.json` 文件。

    有关详细信息，请参阅 [az ml folder attach](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/folder?view=azure-cli-latest&preserve-view=true#ext-azure-cli-ml-az-ml-folder-attach)。

+ 将某个 Azure Blob 容器附加为数据存储。

    ```azurecli-interactive
    az ml datastore attach-blob  -n datastorename -a accountname -c containername
    ```

    有关详细信息，请参阅 [az ml datastore attach-blob](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/datastore?view=azure-cli-latest&preserve-view=true#ext-azure-cli-ml-az-ml-datastore-attach-blob)。

+ 将文件上传到数据存储。

    ```azurecli-interactive
    az ml datastore upload  -n datastorename -p sourcepath
    ```

    有关详细信息，请参阅 [az ml datastore upload](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/datastore?view=azure-cli-latest&preserve-view=true#ext-azure-cli-ml-az-ml-datastore-upload)。

+ 将某个 AKS 群集附加为计算目标。

    ```azurecli-interactive
    az ml computetarget attach aks -n myaks -i myaksresourceid -g myresourcegroup -w myworkspace
    ```

    有关详细信息，请参阅 [az ml computetarget attach aks](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/computetarget/attach?view=azure-cli-latest&preserve-view=true#ext-azure-cli-ml-az-ml-computetarget-attach-aks)

### <a name="compute-clusters"></a>计算群集

+ 创建新的托管计算群集。

    ```azurecli-interactive
    az ml computetarget create amlcompute -n cpu --min-nodes 1 --max-nodes 1 -s STANDARD_D3_V2
    ```



+ 创建具有托管标识的新托管计算群集

  + 用户分配的托管标识

    ```azurecli
    az ml computetarget create amlcompute --name cpu-cluster --vm-size Standard_NC6 --max-nodes 5 --assign-identity '/subscriptions/<subcription_id>/resourcegroups/<resource_group>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<user_assigned_identity>'
    ```

  + 系统分配的托管标识

    ```azurecli
    az ml computetarget create amlcompute --name cpu-cluster --vm-size Standard_NC6 --max-nodes 5 --assign-identity '[system]'
    ```
+ 将托管标识添加到现有群集：

    + 用户分配的托管标识
        ```azurecli
        az ml computetarget amlcompute identity assign --name cpu-cluster '/subscriptions/<subcription_id>/resourcegroups/<resource_group>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<user_assigned_identity>'
        ```
    + 系统分配的托管标识

        ```azurecli
        az ml computetarget amlcompute identity assign --name cpu-cluster '[system]'
        ```

有关详细信息，请参阅 [az ml computetarget create amlcompute](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/computetarget/create?view=azure-cli-latest&preserve-view=true#ext-azure-cli-ml-az-ml-computetarget-create-amlcompute)。

[!INCLUDE [aml-clone-in-azure-notebook](../../includes/aml-managed-identity-note.md)]

<a id="computeinstance"></a>

### <a name="compute-instance"></a>计算实例
管理计算实例。  在下面的所有示例中，计算实例的名称均为“cpu”

+ 创建新的 computeinstance。

    ```azurecli-interactive
    az ml computetarget create computeinstance  -n cpu -s "STANDARD_D3_V2" -v
    ```

    有关详细信息，请参阅 [az ml computetarget create computeinstance](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/computetarget/create?view=azure-cli-latest&preserve-view=true#ext-azure-cli-ml-az-ml-computetarget-create-computeinstance)。

+ 停止 computeinstance。

    ```azurecli-interactive
    az ml computetarget stop computeinstance -n cpu -v
    ```

    有关详细信息，请参阅 [az ml computetarget stop computeinstance](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/computetarget/computeinstance?view=azure-cli-latest&preserve-view=true#ext-azure-cli-ml-az-ml-computetarget-computeinstance-stop)。

+ 启动 computeinstance。

    ```azurecli-interactive
    az ml computetarget start computeinstance -n cpu -v
    ```

    有关详细信息，请参阅 [az ml computetarget start computeinstance](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/computetarget/computeinstance?view=azure-cli-latest&preserve-view=true#ext-azure-cli-ml-az-ml-computetarget-computeinstance-start)。

+ 重启 computeinstance。

    ```azurecli-interactive
    az ml computetarget restart computeinstance -n cpu -v
    ```

    有关详细信息，请参阅 [az ml computetarget restart computeinstance](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/computetarget/computeinstance?view=azure-cli-latest&preserve-view=true#ext-azure-cli-ml-az-ml-computetarget-computeinstance-restart)。

+ 删除 computeinstance。

    ```azurecli-interactive
    az ml computetarget delete -n cpu -v
    ```

    有关详细信息，请参阅 [az ml computetarget delete computeinstance](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/computetarget?view=azure-cli-latest&preserve-view=true#ext-azure-cli-ml-az-ml-computetarget-delete)。


## <a name="run-experiments"></a><a id="experiments"></a>运行试验

* 开始运行试验。 使用此命令时，请为 -c 参数指定 runconfig 文件的名称（如果查看的是文件系统，此名称为 \*.runconfig 前面的文本）。

    ```azurecli-interactive
    az ml run submit-script -c sklearn -e testexperiment train.py
    ```

    > [!TIP]
    > `az ml folder attach` 命令创建一个 `.azureml` 子目录，其中包含两个示例 runconfig 文件。 
    >
    > 如果你的某个 Python 脚本以编程方式创建运行配置对象，则你可以使用 [RunConfig.save()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.runconfiguration?view=azure-ml-py&preserve-view=true#&preserve-view=truesave-path-none--name-none--separate-environment-yaml-false-) 将此对象另存为 runconfig 文件。
    >
    > 可在此 [JSON 文件](https://github.com/microsoft/MLOps/blob/b4bdcf8c369d188e83f40be8b748b49821f71cf2/infra-as-code/runconfigschema.json)中找到完整的 runconfig 架构。 该架构通过每个对象的 `description` 键自行记录。 此外，还有可能值的枚举，在末尾还有模板代码片段。

    有关详细信息，请参阅 [az ml run submit-script](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/run?view=azure-cli-latest&preserve-view=true#ext-azure-cli-ml-az-ml-run-submit-script)。

* 查看试验列表：

    ```azurecli-interactive
    az ml experiment list
    ```

    有关详细信息，请参阅 [az ml experiment list](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/experiment?view=azure-cli-latest&preserve-view=true#ext-azure-cli-ml-az-ml-experiment-list)。

### <a name="hyperdrive-run"></a>HyperDrive 运行

可以结合使用 HyperDrive 与 Azure CLI 以执行参数优化运行。 首先，创建采用以下格式的 HyperDrive 配置文件。 有关超参数优化参数的详细信息，请参阅[优化模型的超参数](how-to-tune-hyperparameters.md)一文。

```yml
# hdconfig.yml
sampling: 
    type: random # Supported options: Random, Grid, Bayesian
    parameter_space: # specify a name|expression|values tuple for each parameter.
    - name: --penalty # The name of a script parameter to generate values for.
      expression: choice # supported options: choice, randint, uniform, quniform, loguniform, qloguniform, normal, qnormal, lognormal, qlognormal
      values: [0.5, 1, 1.5] # The list of values, the number of values is dependent on the expression specified.
policy: 
    type: BanditPolicy # Supported options: BanditPolicy, MedianStoppingPolicy, TruncationSelectionPolicy, NoTerminationPolicy
    evaluation_interval: 1 # Policy properties are policy specific. See the above link for policy specific parameter details.
    slack_factor: 0.2
primary_metric_name: Accuracy # The metric used when evaluating the policy
primary_metric_goal: Maximize # Maximize|Minimize
max_total_runs: 8 # The maximum number of runs to generate
max_concurrent_runs: 2 # The number of runs that can run concurrently.
max_duration_minutes: 100 # The maximum length of time to run the experiment before cancelling.
```

将此文件与运行配置文件一起添加。 然后使用以下命令提交 HyperDrive 运行：
```azurecli
az ml run submit-hyperdrive -e <experiment> -c <runconfig> --hyperdrive-configuration-name <hdconfig> my_train.py
```

请注意运行配置中的参数部分和 HyperDrive 配置中的参数空间 。这些参数包含要传递给训练脚本的命令行参数。 在每次迭代中，运行配置中的值保持不变，而 HyperDrive 配置中的范围将会更迭。 请不要在这两个文件中指定相同的参数。

## <a name="dataset-management"></a>数据集管理

以下命令演示如何在 Azure 机器学习中处理数据集：

+ 注册数据集：

    ```azurecli-interactive
    az ml dataset register -f mydataset.json
    ```

    如需用于定义数据集的 JSON 文件的格式信息，请使用 `az ml dataset register --show-template`。

    有关详细信息，请参阅 [az ml dataset register](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/dataset?view=azure-cli-latest&preserve-view=true#ext-azure-cli-ml-az-ml-dataset-register)。

+ 列出工作区中的所有数据集：

    ```azurecli-interactive
    az ml dataset list
    ```

    有关详细信息，请参阅 [az ml dataset list](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/dataset?view=azure-cli-latest&preserve-view=true#ext-azure-cli-ml-az-ml-dataset-list)。

+ 获取数据集的详细信息：

    ```azurecli-interactive
    az ml dataset show -n dataset-name
    ```

    有关详细信息，请参阅 [az ml dataset show](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/dataset?view=azure-cli-latest&preserve-view=true#ext-azure-cli-ml-az-ml-dataset-show)。

+ 取消注册数据集：

    ```azurecli-interactive
    az ml dataset unregister -n dataset-name
    ```

    有关详细信息，请参阅 [az ml dataset unregister](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/dataset?view=azure-cli-latest&preserve-view=true#ext-azure-cli-ml-az-ml-dataset-archive)。

## <a name="environment-management"></a>环境管理

以下命令演示如何为工作区创建、注册和列出 Azure 机器学习[环境](how-to-configure-environment.md)：

+ 为环境创建基架文件：

    ```azurecli-interactive
    az ml environment scaffold -n myenv -d myenvdirectory
    ```

    有关详细信息，请参阅 [az ml environment scaffold](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/environment?view=azure-cli-latest&preserve-view=true#ext-azure-cli-ml-az-ml-environment-scaffold)。

+ 注册环境：

    ```azurecli-interactive
    az ml environment register -d myenvdirectory
    ```

    有关详细信息，请参阅 [az ml environment register](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/environment?view=azure-cli-latest&preserve-view=true#ext-azure-cli-ml-az-ml-environment-register)。

+ 列出已注册的环境：

    ```azurecli-interactive
    az ml environment list
    ```

    有关详细信息，请参阅 [az ml environment list](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/environment?view=azure-cli-latest&preserve-view=true#ext-azure-cli-ml-az-ml-environment-list)。

+ 下载已注册的环境：

    ```azurecli-interactive
    az ml environment download -n myenv -d downloaddirectory
    ```

    有关详细信息，请参阅 [az ml environment download](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/environment?view=azure-cli-latest&preserve-view=true#ext-azure-cli-ml-az-ml-environment-download)。

### <a name="environment-configuration-schema"></a>环境配置架构

如果使用 `az ml environment scaffold` 命令，它将生成一个模板 `azureml_environment.json` 文件，可以通过 CLI 修改该文件并使用它来创建自定义环境配置。 顶层对象松散地映射到 Python SDK 中的 [`Environment`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.environment%28class%29?view=azure-ml-py&preserve-view=true) 类。 

```json
{
    "name": "testenv",
    "version": null,
    "environmentVariables": {
        "EXAMPLE_ENV_VAR": "EXAMPLE_VALUE"
    },
    "python": {
        "userManagedDependencies": false,
        "interpreterPath": "python",
        "condaDependenciesFile": null,
        "baseCondaEnvironment": null
    },
    "docker": {
        "enabled": false,
        "baseImage": "mcr.microsoft.com/azureml/base:intelmpi2018.3-ubuntu16.04",
        "baseDockerfile": null,
        "sharedVolumes": true,
        "shmSize": "2g",
        "arguments": [],
        "baseImageRegistry": {
            "address": null,
            "username": null,
            "password": null
        }
    },
    "spark": {
        "repositories": [],
        "packages": [],
        "precachePackages": true
    },
    "databricks": {
        "mavenLibraries": [],
        "pypiLibraries": [],
        "rcranLibraries": [],
        "jarLibraries": [],
        "eggLibraries": []
    },
    "inferencingStackVersion": null
}
```

下表详细介绍了 JSON 文件中的每个顶层字段、其类型和说明。 如果某个对象类型已链接到 Python SDK 中的类，则在每个 JSON 字段与 Python 类中的公共变量名称之间存在松散的 1:1 匹配。 在某些情况下，字段可能会映射到构造函数参数，而不是类变量。 例如，`environmentVariables` 字段映射到 [`Environment`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.environment%28class%29?view=azure-ml-py&preserve-view=true) 类中的 `environment_variables` 变量。

| JSON 字段 | 类型 | 说明 |
|---|---|---|
| `name` | `string` | 环境的名称。 名称不能以 **Microsoft** 或 **AzureML** 开头。 |
| `version` | `string` | 环境的版本。 |
| `environmentVariables` | `{string: string}` | 环境变量名称和值的哈希映射。 |
| `python` | [`PythonSection`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.environment.pythonsection?view=azure-ml-py&preserve-view=true)hat 定义要在目标计算资源上使用的 Python 环境和解释器。 |
| `docker` | [`DockerSection`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.environment.dockersection?view=azure-ml-py&preserve-view=true) | 定义根据环境规范对生成的 Docker 映像进行自定义时使用的设置。 |
| `spark` | [`SparkSection`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.environment.sparksection?view=azure-ml-py&preserve-view=true) | 此节配置 Spark 设置。 仅当 framework 设置为 PySpark 时才使用它。 |
| `databricks` | [`DatabricksSection`](https://docs.microsoft.com/python/api/azureml-core/azureml.core.databricks.databrickssection?view=azure-ml-py&preserve-view=true) | 配置 Databricks 库依赖项。 |
| `inferencingStackVersion` | `string` | 指定添加到映像的推理堆栈版本。 若要避免添加推理堆栈，请将此字段保留为 `null`。 有效值：latest。 |

## <a name="ml-pipeline-management"></a>ML 管道管理

以下命令演示如何使用机器学习管道：

+ 创建机器学习管道：

    ```azurecli-interactive
    az ml pipeline create -n mypipeline -y mypipeline.yml
    ```

    有关详细信息，请参阅 [az ml pipeline create](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/pipeline?view=azure-cli-latest&preserve-view=true#ext-azure-cli-ml-az-ml-pipeline-create)。

    有关管道 YAML 文件的详细信息，请参阅[在 YAML 中定义机器学习管道](reference-pipeline-yaml.md)。

+ 运行管道：

    ```azurecli-interactive
    az ml run submit-pipeline -n myexperiment -y mypipeline.yml
    ```

    有关详细信息，请参阅 [az ml run submit-pipeline](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/run?view=azure-cli-latest&preserve-view=true#ext-azure-cli-ml-az-ml-run-submit-pipeline)。

    有关管道 YAML 文件的详细信息，请参阅[在 YAML 中定义机器学习管道](reference-pipeline-yaml.md)。

+ 计划管道：

    ```azurecli-interactive
    az ml pipeline create-schedule -n myschedule -e myexpereiment -i mypipelineid -y myschedule.yml
    ```

    有关详细信息，请参阅 [az ml pipeline create-schedule](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/pipeline?view=azure-cli-latest&preserve-view=true#ext-azure-cli-ml-az-ml-pipeline-create-schedule)。

    有关管道计划 YAML 文件的详细信息，请参阅[在 YAML 中定义机器学习管道](reference-pipeline-yaml.md#schedules)。

## <a name="model-registration-profiling-deployment"></a>模型注册、分析和部署

以下命令演示如何注册已训练的模型，然后将其部署为生产服务：

+ 将模型注册到 Azure 机器学习：

    ```azurecli-interactive
    az ml model register -n mymodel -p sklearn_regression_model.pkl
    ```

    有关详细信息，请参阅 [az ml model register](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/model?view=azure-cli-latest&preserve-view=true#ext-azure-cli-ml-az-ml-model-register)。

+ **（可选）** 分析模型以获取部署的最佳 CPU 和内存值。
    ```azurecli-interactive
    az ml model profile -n myprofile -m mymodel:1 --ic inferenceconfig.json -d "{\"data\": [[1,2,3,4,5,6,7,8,9,10],[10,9,8,7,6,5,4,3,2,1]]}" -t myprofileresult.json
    ```

    有关详细信息，请参阅 [az ml model profile](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/model?view=azure-cli-latest&preserve-view=true#ext-azure-cli-ml-az-ml-model-profile)。

+ 将模型部署到 AKS
    ```azurecli-interactive
    az ml model deploy -n myservice -m mymodel:1 --ic inferenceconfig.json --dc deploymentconfig.json --ct akscomputetarget
    ```
    
    有关推理配置文件架构的详细信息，请参阅[推理配置架构](#inferenceconfig)。
    
    有关部署配置文件架构的详细信息，请参阅[部署配置架构](#deploymentconfig)。

    有关详细信息，请参阅 [az ml model deploy](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml/model?view=azure-cli-latest&preserve-view=true#ext-azure-cli-ml-az-ml-model-deploy)。

<a id="inferenceconfig"></a>

## <a name="inference-configuration-schema"></a>推理配置架构

[!INCLUDE [inferenceconfig](../../includes/machine-learning-service-inference-config.md)]

<a id="deploymentconfig"></a>

## <a name="deployment-configuration-schema"></a>部署配置架构

### <a name="local-deployment-configuration-schema"></a>本地部署配置架构

[!INCLUDE [deploymentconfig](../../includes/machine-learning-service-local-deploy-config.md)]

### <a name="azure-container-instance-deployment-configuration-schema"></a>Azure 容器实例部署配置架构 

[!INCLUDE [deploymentconfig](../../includes/machine-learning-service-aci-deploy-config.md)]

### <a name="azure-kubernetes-service-deployment-configuration-schema"></a>Azure Kubernetes 服务部署配置架构

[!INCLUDE [deploymentconfig](../../includes/machine-learning-service-aks-deploy-config.md)]

## <a name="next-steps"></a>后续步骤

* [机器学习 CLI 扩展的命令参考](https://docs.microsoft.com/cli/azure/ext/azure-cli-ml/ml?view=azure-cli-latest&preserve-view=true)。

* [使用 Azure Pipelines 训练和部署机器学习模型](/azure/devops/pipelines/targets/azure-machine-learning)

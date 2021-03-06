---
title: 快速入门：适用于 Node.js 的计算机视觉客户端库
description: 通过本快速入门开始使用适用于 Node.js 的计算机视觉客户端库
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: include
ms.date: 01/22/2020
ms.author: pafarley
ms.custom: devx-track-js
ms.openlocfilehash: d0e94066c6e8ab60a52761898e49bdf11997a062
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/05/2020
ms.locfileid: "91309807"
---
<a name="HOLTop"></a>

[参考文档](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-computervision/?view=azure-node-latest) | [库源代码](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/cognitiveservices/cognitiveservices-computervision) | [包 (npm)](https://www.npmjs.com/package/@azure/cognitiveservices-computervision) | [示例](https://azure.microsoft.com/resources/samples/?service=cognitive-services&term=vision&sort=0)

## <a name="prerequisites"></a>先决条件

* Azure 订阅 - [免费创建订阅](https://azure.microsoft.com/free/cognitive-services/)
* 最新版本的 [Node.js](https://nodejs.org/)
* 拥有 Azure 订阅后，在 Azure 门户中<a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesComputerVision"  title="创建计算机视觉资源"  target="_blank">创建计算机视觉资源 <span class="docon docon-navigate-external x-hidden-focus"></span></a>，获取密钥和终结点。 部署后，单击“转到资源”。
    * 需要从创建的资源获取密钥和终结点，以便将应用程序连接到计算机视觉服务。 你稍后会在快速入门中将密钥和终结点粘贴到下方的代码中。
    * 可以使用免费定价层 (`F0`) 试用该服务，然后再升级到付费层进行生产。
* 为密钥和终结点 URL [创建环境变量](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account#configure-an-environment-variable-for-authentication)，分别将其命名为 `COMPUTER_VISION_SUBSCRIPTION_KEY` 和 `COMPUTER_VISION_ENDPOINT`。

## <a name="setting-up"></a>设置

### <a name="create-a-new-nodejs-application"></a>创建新的 Node.js 应用程序

在控制台窗口（例如 cmd、PowerShell 或 Bash）中，为应用创建一个新目录并导航到该目录。

```console
mkdir myapp && cd myapp
```

运行 `npm init` 命令以使用 `package.json` 文件创建一个 node 应用程序。

```console
npm init
```

### <a name="install-the-client-library"></a>安装客户端库

安装 `ms-rest-azure` 和 `@azure/cognitiveservices-computervision` NPM 包:

```console
npm install @azure/cognitiveservices-computervision
```

应用的 `package.json` 文件将使用依赖项进行更新。

### <a name="prepare-the-nodejs-script"></a>准备 Node.js 脚本

创建新文件 *index.js*，将其在文本编辑器中打开。 添加以下 import 语句。

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_imports)]

然后定义函数 `computerVision`，并声明一个包含主函数和回调函数的异步系列。 我们会将快速入门代码添加到主函数中，并调用脚本底部的 `computerVision`。

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_functiondef_begin)]

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_functiondef_end)]

## <a name="object-model"></a>对象模型

以下类和接口将处理计算机视觉 Node.js SDK 的某些主要功能。

|名称|说明|
|---|---|
| [ComputerVisionClient](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-computervision/computervisionclient?view=azure-node-latest) | 所有计算机视觉功能都需要此类。 可以使用订阅信息实例化此类，然后使用它来执行大多数图像操作。|
|[VisualFeatureTypes](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-computervision/visualfeaturetypes?view=azure-node-latest)| 此枚举定义可在标准分析操作中执行的不同类型的图像分析。 请根据需求指定一组 **VisualFeatureTypes** 值。 |

## <a name="code-examples"></a>代码示例

这些代码片段演示如何使用适用于 Node.js 的计算机视觉客户端库执行以下任务：

* [对客户端进行身份验证](#authenticate-the-client)
* [分析图像](#analyze-an-image)
* [读取印刷体文本和手写文本](#read-printed-and-handwritten-text)

## <a name="authenticate-the-client"></a>验证客户端

为资源的 Azure 终结点和密钥创建变量。 如果在启动应用程序后创建了环境变量，则需要关闭再重新打开运行该应用程序的编辑器、IDE 或 shell 才能访问该变量。

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_vars)]

使用终结点和密钥实例化某个客户端。 使用密钥和终结点创建 [ApiKeyCredentials](https://docs.microsoft.com/python/api/msrest/msrest.authentication.apikeycredentials?view=azure-python) 对象，然后使用它创建 [ComputerVisionClient](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-computervision/computervisionclient?view=azure-node-latest) 对象。

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_client)]

## <a name="analyze-an-image"></a>分析图像

此部分的代码通过分析远程图像来提取各种视觉特征。 可以在客户端对象的 **analyzeImage** 方法中执行这些操作，也可以使用单个方法来调用它们。 有关详细信息，请参阅[参考文档](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-computervision/?view=azure-node-latest)。

> [!NOTE]
> 还可以分析本地图像。 请参阅 [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/ComputerVision/ComputerVisionQuickstart.js) 上的示例代码以了解涉及本地图像的方案。

### <a name="get-image-description"></a>获取图像说明

下面的代码获取为图像生成的描述文字列表。 有关更多详细信息，请参阅[描述图像](../../concept-describing-images.md)。

首先，定义要分析的图像的 URL：

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_describe_image)]

然后添加以下代码，用于获取图像详细信息并将其输出到控制台。

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_describe)]

### <a name="get-image-category"></a>获取图像类别

下面的代码获取所检测到的图像类别。 有关更多详细信息，请参阅[对图像进行分类](../../concept-categorizing-images.md)。

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_categories)]

定义帮助程序函数 `formatCategories`：

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_categories_format)]

### <a name="get-image-tags"></a>获取图像标记

以下代码获取图像中检测到的标记集。 有关更多详细信息，请参阅[内容标记](../../concept-tagging-images.md)。

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_tags)]

定义帮助程序函数 `formatTags`：

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_tagsformat)]

### <a name="detect-objects"></a>检测物体

以下代码检测图像中的常见物体并将其输出到控制台。 有关更多详细信息，请参阅[物体检测](../../concept-object-detection.md)。

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_objects)]

定义帮助程序函数 `formatRectObjects` 以返回上、左、下、右坐标以及宽度和高度。

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_objectformat)]

### <a name="detect-brands"></a>检测品牌

以下代码检测图像中的公司品牌和徽标，并将其输出到控制台。 有关更多详细信息，请参阅[品牌检测](../../concept-brand-detection.md)。

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_brands)]

### <a name="detect-faces"></a>检测人脸

下面的代码返回图像中检测到的人脸及其矩形坐标，以及选择面属性。 有关更多详细信息，请参阅[人脸检测](../../concept-detecting-faces.md)。

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_faces)]

定义帮助程序函数 `formatRectFaces`：

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_formatfaces)]

### <a name="detect-adult-racy-or-gory-content"></a>检测成人、色情或血腥内容

以下代码输出图像中检测到的成人内容。 有关更多详细信息，请参阅[成人、色情或血腥内容](../../concept-detecting-adult-content.md)。

定义要使用的图像的 URL：

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_adult_image)]

然后添加以下代码来检测成人内容，并将结果输出到控制台。

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_adult)]

### <a name="get-image-color-scheme"></a>获取图像配色方案

以下代码输出图像中检测到的颜色属性，如主色和主题色。 有关更多详细信息，请参阅[配色方案](../../concept-detecting-color-schemes.md)。

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_colors)]

定义帮助程序函数 `printColorScheme`，将颜色方案的详细信息输出到控制台。

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_colors_print)]

### <a name="get-domain-specific-content"></a>获取特定于域的内容

计算机视觉可以使用专用模型对图像进行进一步分析。 有关更多详细信息，请参阅[特定于域的内容](../../concept-detecting-domain-content.md)。

首先，定义要分析的图像的 URL：

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_domain_image)]

以下代码分析了图像中检测到的地标的相关数据。

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_landmarks)]

定义帮助程序函数 `formatRectDomain`，分析有关检测到的地标的位置数据。

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_landmarks_rect)]

### <a name="get-the-image-type"></a>获取图像类型

以下代码输出有关图像类型的信息&mdash;无论它是剪贴画还是线条图。

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_imagetype)]

定义帮助程序函数 `describeType`：

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_imagetype_describe)]

## <a name="extract-text-ocr-with-read"></a>通过 Read 提取文本 (OCR)

计算机视觉可以提取图像中的可见文本，并将其转换为字符流。 此示例使用读取操作。

> [!NOTE]
> 还可以从本地图像读取文本。 请参阅 [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/ComputerVision/ComputerVisionQuickstart.js) 上的示例代码以了解涉及本地图像的方案。

### <a name="set-up-test-images"></a>设置测试图像

保存要从中提取文本的图像的 URL 的引用。

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_read_images)]

### <a name="call-the-read-api"></a>调用读取 API

添加以下代码，该代码针对给定图像调用 `readTextFromURL` 和 `readTextFromFile` 函数。

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_read_call)]

定义 `readTextFromURL` 和 `readTextFromFile` 函数。 它们调用客户端对象上的 read 和 readInStream 方法，这些方法返回操作 ID 并启动异步进程来读取图像的内容 。 然后它们使用操作 ID 来检查操作状态，直到返回结果。 然后它们会返回提取的结果。

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_read_helper)]

然后定义帮助程序函数 `printRecText`，该函数将读取操作的结果输出到控制台。

[!code-javascript[](~/cognitive-services-quickstart-code/javascript/ComputerVision/ComputerVisionQuickstart.js?name=snippet_read_print)]

## <a name="run-the-application"></a>运行应用程序

在快速入门文件中使用 `node` 命令运行应用程序。

```console
node index.js
```

## <a name="clean-up-resources"></a>清理资源

如果想要清理并删除认知服务订阅，可以删除资源或资源组。 删除资源组同时也会删除与之相关联的任何其他资源。

* [门户](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [Azure CLI](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
>[计算机视觉 API 参考 (Node.js)](https://docs.microsoft.com/javascript/api/@azure/cognitiveservices-computervision/?view=azure-node-latest)

* [什么是计算机视觉？](../../overview.md)
* 可以在 [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/ComputerVision/ComputerVisionQuickstart.js) 上找到此示例的源代码。

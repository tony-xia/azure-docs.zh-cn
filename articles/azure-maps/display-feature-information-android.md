---
title: 显示 Azure Maps Android SDK 中的功能信息 |Microsoft Azure 映射
description: 了解如何在用户与地图功能交互时显示信息。 使用 Azure Maps Android SDK 显示 toast 消息和其他类型的消息。
author: rbrundritt
ms.author: richbrun
ms.date: 08/08/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.openlocfilehash: 44c5f9f1a7c0d014d101ad45a80e1e53c42f69a3
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/07/2020
ms.locfileid: "88009029"
---
# <a name="display-feature-information"></a>显示功能信息

空间数据通常使用点、线条和多边形来表示。 此数据通常具有与之关联的元数据信息。 例如，点可能表示商店的位置，而有关该餐馆的元数据可能是其所服务的食品的名称、地址和类型。 可以使用将此元数据添加为这些功能的属性 `JsonObject` 。 下面的代码使用 `title` 值为 "Hello World！" 的属性创建一个简单的点功能。

```java
//Create a data source and add it to the map.
DataSource dataSource = new DataSource();
map.sources.add(dataSource);

//Create a point feature with some properties and add it to the data source.
JsonObject properties = new JsonObject();
properties.addProperty("title", "Hello World!");

//Create a point feature, pass in the metadata properties, and add it to the data source.
dataSource.add(Feature.fromGeometry(Point.fromLngLat(-122.33, 47.64), properties));
```

当用户与地图上的功能交互时，可以使用事件来响应这些操作。 常见的情况是显示一条消息，该消息由用户与之交互的功能的元数据属性组成。 `OnFeatureClick`事件是用于检测用户在地图上点击功能时所使用的主要事件。 还有一个 `OnLongFeatureClick` 事件。 向映射添加 `OnFeatureClick` 事件时，可以将该事件限制为单个层，方法是传入层的 ID 以将其限制为。 如果未传入任何层 ID，请在映射上点击任意功能，而不考虑它所在的层，将触发此事件。 下面的代码创建一个符号层，用于在地图上呈现点数据，然后添加一个 `OnFeatureClick` 事件并将其限制为此符号层。

```java
//Create a symbol and add it to the map.
SymbolLayer symbolLayer = new SymbolLayer(dataSource);
map.layers.add(symbolLayer);

//Add a feature click event to the map.
map.events.add((OnFeatureClick) (features) -> {
    //Retrieve the title property of the feature as a string.
    String msg = features.get(0).properties().get("title").getAsString();

    //Do something with the message.
}, symbolLayer.getId());    //Limit this event to the symbol layer.
```

## <a name="display-a-toast-message"></a>显示 toast 消息

Toast 消息是向用户显示信息的最简单方法之一，在 Android 的所有版本中都提供。 它不支持任何类型的用户输入，只会在短时间内显示。 如果你想要快速让用户了解他们点击的内容，则 toast 消息可能是一个不错的选择。 下面的代码演示如何在事件中使用 toast 消息 `OnFeatureClick` 。

```java
//Add a feature click event to the map.
map.events.add((OnFeatureClick) (features) -> {
    //Retrieve the title property of the feature as a string.
    String msg = features.get(0).getStringProperty("title");

    //Display a toast message with the title information.
    Toast.makeText(this, msg, Toast.LENGTH_SHORT).show();
}, symbolLayer.getId());    //Limit this event to the symbol layer.
```

<center>

![正在点击的功能的动画和正在显示的 toast 消息](./media/display-feature-information-android/symbol-layer-click-toast-message.gif)</center>

除了 toast 消息以外，还有很多其他方法可以提供功能的元数据属性，例如：

- [Snakbar 小组件](https://developer.android.com/training/snackbar/showing.html)-Snackbars 提供有关操作的轻型反馈。 它们会在移动设备的底部显示简短消息，在较大的设备上显示在左下方。 Snackbars 显示在屏幕上的所有其他元素之上，一次只能显示一个。
- [对话框-对话框](https://developer.android.com/guide/topics/ui/dialogs)是一个小窗口，它会提示用户做出决定或输入其他信息。 对话框并不填充屏幕，通常用于模式事件，这些事件要求用户执行操作，然后才能继续。
- 向当前活动添加一个[片段](https://developer.android.com/guide/components/fragments)。
- 导航到另一个活动或视图。

## <a name="next-steps"></a>后续步骤

将更多数据添加到地图：

> [!div class="nextstepaction"]
> [添加符号层](how-to-add-symbol-to-android-map.md)

> [!div class="nextstepaction"]
> [在 Android 地图中添加形状](how-to-add-shapes-to-android-map.md)

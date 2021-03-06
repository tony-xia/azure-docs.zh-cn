---
title: 组件
description: Azure 远程渲染范围内组件的定义
author: florianborn71
ms.author: flborn
ms.date: 02/04/2020
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.openlocfilehash: a488e2499b92b290ad2b55120c3c70a18d45d426
ms.sourcegitcommit: f845ca2f4b626ef9db73b88ca71279ac80538559
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/09/2020
ms.locfileid: "89613935"
---
# <a name="components"></a>组件

Azure 远程渲染使用[实体组件系统](https://en.wikipedia.org/wiki/Entity_component_system)模式。 [实体](entities.md)代表对象的位置和层次结构组合，而组件则负责实现行为。

最常使用的组件类型是 [:::no-loc text="mesh components":::](meshes.md) ，它将网格添加到呈现管道中。 同样，[光线组件](../overview/features/lights.md)用于添加光线，而[剖切面组件](../overview/features/cut-planes.md)用于切开网格。

所有这些组件都使用它们所附着的实体的转换（位置、旋转、缩放）作为它们的参考点。

## <a name="working-with-components"></a>使用组件

可以通过编程方式轻松地添加、删除和操作组件：

```cs
// create a point light component
AzureSession session = GetCurrentlyConnectedSession();
PointLightComponent lightComponent = session.Actions.CreateComponent(ObjectType.PointLightComponent, ownerEntity) as PointLightComponent;

lightComponent.Color = new Color4Ub(255, 150, 20, 255);
lightComponent.Intensity = 11;

// ...

// destroy the component
lightComponent.Destroy();
lightComponent = null;
```

```cpp
// create a point light component
ApiHandle<AzureSession> session = GetCurrentlyConnectedSession();

ApiHandle<PointLightComponent> lightComponent = session->Actions()->CreateComponent(ObjectType::PointLightComponent, ownerEntity)->as<PointLightComponent>();

// ...

// destroy the component
lightComponent->Destroy();
lightComponent = nullptr;
```

组件在创建时附加到实体。 之后不能将其迁移至其他实体。 使用 `Component.Destroy()` 显式删除组件，或者在组件的所有者实体被销毁时自动显式删除组件。

一次只能向一个实体添加每个组件类型的一个实例。

## <a name="unity-specific"></a>Unity 特定

Unity 集成具有其他扩展函数，用于与组件进行交互。 请参阅 [Unity 游戏对象和组件](../how-tos/unity/objects-components.md)。

## <a name="api-documentation"></a>API 文档

* [C # ComponentBase](https://docs.microsoft.com/dotnet/api/microsoft.azure.remoterendering.componentbase)
* [C # RemoteManager CreateComponent ( # B1 ](https://docs.microsoft.com/dotnet/api/microsoft.azure.remoterendering.remotemanager.createcomponent)
* [C # FindComponentOfType ( # B1 ](https://docs.microsoft.com/dotnet/api/microsoft.azure.remoterendering.entity.findcomponentoftype)
* [C + + ComponentBase](https://docs.microsoft.com/cpp/api/remote-rendering/componentbase)
* [C + + RemoteManager：： CreateComponent ( # B1 ](https://docs.microsoft.com/cpp/api/remote-rendering/remotemanager#createcomponent)
* [C + + Entity：： FindComponentOfType ( # B1 ](https://docs.microsoft.com/cpp/api/remote-rendering/entity#findcomponentoftype)

## <a name="next-steps"></a>后续步骤

* [对象边界](object-bounds.md)
* [网格](meshes.md)

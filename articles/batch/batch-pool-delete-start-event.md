---
title: Azure Batch 池删除开始事件
description: 批处理池删除开始事件参考。 当池删除操作开始时，会发出此事件。
ms.topic: reference
ms.date: 04/20/2017
ms.openlocfilehash: d4fac833b6c77258e48b38838f6b7b133738a6e8
ms.sourcegitcommit: 877491bd46921c11dd478bd25fc718ceee2dcc08
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/02/2020
ms.locfileid: "83723929"
---
# <a name="pool-delete-start-event"></a>池删除开始事件

 当池删除操作开始时，会发出此事件。 由于池删除是异步事件，预计在删除操作完成后会发出池删除完成事件。

 以下示例显示了池删除开始事件的正文。

```
{
    "id": "myPool1"
}
```

|元素|类型|说明|
|-------------|----------|-----------|
|`id`|String|池的 ID。|

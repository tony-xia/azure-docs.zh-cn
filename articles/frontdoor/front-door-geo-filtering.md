---
title: Azure Front Door 的域中的地区筛选 | Microsoft Docs
description: 本文介绍 Azure Front Door 的地区筛选策略
services: frontdoor
documentationcenter: ''
author: duongau
editor: ''
ms.service: frontdoor
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/28/2020
ms.author: duau
ms.reviewer: tyao
ms.openlocfilehash: 42697a57d39f4a34eee4866b67e2cde947db1ff5
ms.sourcegitcommit: 3792cf7efc12e357f0e3b65638ea7673651db6e1
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/29/2020
ms.locfileid: "91449257"
---
# <a name="geo-filtering-on-a-domain-for-azure-front-door"></a>Azure 前门域上的地理筛选

默认情况下，无论请求来自何处，Azure 前门都将对所有用户请求做出响应。 在某些情况下，你可能想要按国家/地区限制对你的 web 应用程序的访问。 使用 Web 应用程序防火墙 (WAF) service 在前门中，你可以使用自定义访问规则为终结点上的特定路径定义策略，以允许或阻止来自指定国家/地区的访问。 

WAF 策略包含一组自定义规则。 规则由匹配条件、操作和优先级组成。 在匹配条件中，定义匹配变量、运算符和匹配值。 对于地理筛选规则，REMOTE_ADDR 匹配变量，运算符为 GeoMatch，值为一个有两个字母的国家/地区代码。 可以结合使用 GeoMatch 条件和 REQUEST_URI 字符串匹配条件来创建基于路径的地理筛选规则。

你可以通过使用 [Azure PowerShell](front-door-tutorial-geo-filtering.md) 或使用 [快速入门模板](https://github.com/Azure/azure-quickstart-templates/tree/master/101-front-door-geo-filtering)为前门配置异地筛选策略。

## <a name="countryregion-code-reference"></a>国家/地区代码参考

|国家/地区代码 | 国家/地区名称 |
| ----- | ----- |
| AD | 安道尔 |
| AE | 阿拉伯联合酋长国|
| AF | 阿富汗|
| AG | 安提瓜和巴布达|
| AL | 阿尔巴尼亚|
| AM | 亚美尼亚|
| AO | 安哥拉|
| AR | 阿根廷|
| AS | 美属萨摩亚|
| AT | 奥地利|
| AU | 澳大利亚|
| AZ | 阿塞拜疆|
| BA | 波斯尼亚和黑塞哥维那|
| BB | 巴巴多斯|
| BD | 孟加拉|
| BE | 比利时|
| BF | 布基纳法索|
| BG | 保加利亚|
| BH | 巴林|
| BI | 布隆迪|
| BJ | 贝宁|
| BL | 圣巴泰勒米|
| BN | 文莱|
| BO | 玻利维亚|
| BR | 巴西|
| BS | 巴哈马|
| BT | 不丹|
| BW | 博茨瓦纳|
| BY | 白俄罗斯|
| BZ | 伯利兹|
| CA | Canada|
| CD | 刚果民主共和国|
| CF | 中非共和国|
| CH | 瑞士|
| CI | 科特迪瓦|
| CL | 智利|
| CM | 喀麦隆|
| CN | 中国|
| CO | 哥伦比亚|
| CR | 哥斯达黎加|
| CU | 古巴|
| CV | 佛得角|
| CY | 塞浦路斯|
| CZ | 捷克共和国|
| DE | 德国|
| DK | 丹麦|
| DO | 多米尼加共和国|
| DZ | 阿尔及利亚|
| EC | 厄瓜多尔|
| EE | 爱沙尼亚|
| EG | 埃及|
| ES | 西班牙|
| ET | 埃塞俄比亚|
| FI | 芬兰|
| FJ | 斐济|
| FM | 密克罗尼西亚联邦|
| FR | 法国|
| GB | United Kingdom|
| GE | 格鲁吉亚|
| GF | 法属圭亚那|
| GH | 加纳|
| GN | 几内亚|
| GP | 瓜德罗普岛|
| GR | 希腊|
| GT | 危地马拉|
| GY | 圭亚那|
| HK | 香港特别行政区|
| HN | 洪都拉斯|
| HR | 克罗地亚|
| HT | 海地|
| HU | 匈牙利|
| ID | 印度尼西亚|
| IE | 爱尔兰|
| IL | 以色列|
| IN | 印度|
| IQ | 伊拉克|
| IR | 伊朗伊斯兰共和国|
| IS | 冰岛|
| IT | 意大利|
| JM | 牙买加|
| JO | 约旦|
| JP | 日本|
| KE | 肯尼亚|
| KG | 吉尔吉斯斯坦|
| KH | 柬埔寨|
| KI | 基里巴斯|
| KN | 圣基茨和尼维斯|
| KP | 朝鲜民主主义人民共和国|
| KR | 韩国|
| KW | 科威特|
| KY | 开曼群岛|
| KZ | 哈萨克斯坦|
| LA | 老挝人民民主共和国|
| LB | 黎巴嫩|
| LI | 列支敦士登|
| LK | 斯里兰卡|
| LR | 利比里亚|
| LS | 莱索托|
| LT | 立陶宛|
| LU | 卢森堡|
| LV | 拉脱维亚|
| LY | 利比亚 |
| MA | 摩洛哥|
| MD | 摩尔多瓦共和国|
| MG | 马达加斯加岛|
| MK | 北马其顿|
| ML | 马里|
| MM | 缅甸|
| MN | 蒙古|
| MO | 澳门特别行政区|
| MQ | 马提尼克岛|
| MR | 毛利塔尼亚|
| MT | 马耳他|
| MV | 马尔代夫|
| MW | 马拉维|
| MX | 墨西哥|
| MY | 马来西亚|
| MZ | 莫桑比克|
| NA | 纳米比亚|
| NE | 尼日尔|
| NG | 尼日利亚|
| NI | 尼加拉瓜|
| NL | 荷兰|
| 是 | 挪威|
| NP | 尼泊尔|
| NR | 瑙鲁|
| NZ | 新西兰|
| OM | 阿曼|
| PA | 巴拿马|
| PE | 秘鲁|
| PH | 菲律宾|
| PK | 巴基斯坦|
| PL | 波兰|
| PR | 波多黎各|
| PT | 葡萄牙|
| PW | 帕劳群岛|
| PY | 巴拉圭|
| QA | 卡塔尔|
| RE | 留尼汪|
| RO | 罗马尼亚|
| RS | 塞尔维亚|
| RU | 俄罗斯联邦|
| RW | 卢旺达|
| SA | 沙特阿拉伯|
| SD | 苏丹|
| SE | 瑞典|
| SG | 新加坡|
| SI | 斯洛文尼亚|
| SK | 斯洛伐克|
| SN | 塞内加尔|
| SO | 索马里|
| SR | 苏里南|
| SS | 南苏丹|
| SV | 萨尔瓦多|
| SY | 阿拉伯叙利亚共和国|
| SZ | 斯威士兰|
| TC | 特克斯和凯科斯群岛|
| TG | 多哥|
| TH | 泰国|
| TN | 突尼斯|
| TR | 土耳其|
| TT | 特立尼达和多巴哥|
| TW | 中国台湾|
| TZ | 坦桑尼亚联合共和国|
| UA | 乌克兰|
| UG | 乌干达|
| 美国 | United States|
| UY | 乌拉圭|
| UZ | 乌兹别克斯坦|
| VC | 圣文森特和格林纳丁斯|
| VE | 委内瑞拉|
| VG | 英属维尔京群岛|
| VI | 美属维尔京群岛|
| VN | 越南|
| ZA | 南非|
| ZM | 赞比亚|
| ZW | 津巴布韦|

## <a name="next-steps"></a>后续步骤

- 了解如何[创建 Front Door](quickstart-create-front-door.md)。
- 了解如何 [设置异地筛选 WAF 策略](front-door-tutorial-geo-filtering.md)。

---
title: 語言支援 - 文字分析 API
titleSuffix: Azure Cognitive Services
description: 文字分析 API 支援的自然語言清單。 本文說明每項作業支援的語言。
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: conceptual
ms.date: 08/26/2020
ms.author: aahi
ms.openlocfilehash: e2c6fc739fa81e6eb7c98073e3575e4143d317b2
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/26/2020
ms.locfileid: "88932962"
---
# <a name="text-analytics-api-v3-language-support"></a>文字分析 API v3 語言支援 

> [!IMPORTANT]
> 文字分析 API 的3.x 版目前無法在下欄區域使用：印度中部、阿拉伯聯合大公國北部、中國北部2中國東部。


#### <a name="sentiment-analysis"></a>[情感分析](#tab/sentiment-analysis)

| Language              | 語言代碼 | v2 支援 | v3 支援 | 起始 v3 模型版本： |              備註 |
|:----------------------|:-------------:|:----------:|:----------:|:--------------------------:|-------------------:|
| 簡體中文    |   `zh-hans`   |     ✓      |     ✓      |         2019-10-01         | 也接受 `zh` |
| 繁體中文   |   `zh-hant`   |            |     ✓      |         2019-10-01         |                    |
| 丹麥文               |     `da`      |     ✓      |            |                            |                    |
| 荷蘭文                 |     `nl`      |     ✓      |            |                            |                    |
| 英文               |     `en`      |     ✓      |     ✓      |         2019-10-01         |                    |
| 芬蘭文               |     `fi`      |     ✓      |            |                            |                    |
| 法文                |     `fr`      |     ✓      |     ✓      |         2019-10-01         |                    |
| 德文                |     `de`      |     ✓      |     ✓      |         2019-10-01         |                    |
| 希臘文                 |     `el`      |     ✓      |            |                            |                    |
| 義大利文               |     `it`      |     ✓      |     ✓      |         2019-10-01         |                    |
| 日文              |     `ja`      |     ✓      |     ✓      |         2019-10-01         |                    |
| 韓文                |     `ko`      |            |     ✓      |         2019-10-01         |                    |
| 挪威文 (巴克摩)   |     `no`      |     ✓      |     ✓       |        2020-07-01         |                    |
| 波蘭文                |     `pl`      |     ✓      |            |                            |                    |
| 葡萄牙文 (葡萄牙) |    `pt-PT`    |     ✓      |     ✓      |         2019-10-01         | 也接受 `pt` |
| 俄文               |     `ru`      |     ✓      |            |                            |                    |
| 西班牙文               |     `es`      |     ✓      |     ✓      |         2019-10-01         |                    |
| 瑞典文               |     `sv`      |     ✓      |            |                            |                    |
| 土耳其文               |     `tr`      |     ✓      |     ✓       |         2020-07-01        |                    |

### <a name="opinion-mining-v31-preview-only"></a>意見挖掘 (3.1-僅限預覽) 

| Language              | 語言代碼 | 從 v3 模型版本開始： |              備註 |
|:----------------------|:-------------:|:------------------------------------:|-------------------:|
| 英文               |     `en`      |              2020-04-01              |                    |


#### <a name="named-entity-recognition-ner"></a>[命名實體辨識 (NER) ](#tab/named-entity-recognition)

> [!NOTE]
> * NER v3 目前僅支援英文和西班牙文語言。 如果您使用不同的語言來呼叫 NER v3，則 API 會傳回2.1 的結果，前提是版本2.1 中支援該語言。
> * 2.1 版只會針對英文、簡體中文、法文、德文和西班牙文等語言傳回一組完整的可用實體。  針對其他支援的語言，會傳回「人員」、「位置」和「組織」實體。

| Language               | 語言代碼 | 2.1 版支援 | v3 支援 | 從 v3 模型版本開始： |       備註        |
|:-----------------------|:-------------:|:----------:|:----------:|:-------------------------------:|:------------------:|
| 阿拉伯文                |     `ar`      |     ✓      |            |                                 |                    |
| 捷克文                 |     `cs`      |     ✓      |            |                                 |                    |
| 簡體中文     |   `zh-hans`   |     ✓      |            |                                 | 也接受 `zh` |
| 繁體中文   |   `zh-hant`   |     ✓      |            |                                 |                    |
| 丹麥文                |     `da`      |     ✓      |            |                                 |                    |
| 荷蘭文                 |     `nl`      |     ✓      |            |                                 |                    |
| 英文                |     `en`      |     ✓      |     ✓      |           2019-10-01            |                    |
| 芬蘭文               |     `fi`      |     ✓      |            |                                 |                    |
| 法文                 |     `fr`      |     ✓      |            |                                 |                    |
| 德文                 |     `de`      |     ✓      |            |                                 |                    |
| Hebrew                |     `he`      |     ✓      |            |                                 |                    |
| 匈牙利文             |     `hu`      |     ✓      |            |                                 |                    |
| 義大利文               |     `it`      |     ✓      |            |                                 |                    |
| 日文              |     `ja`      |     ✓      |            |                                 |                    |
| 韓文                |     `ko`      |     ✓      |            |                                 |                    |
| 挪威文 (巴克摩)   |     `no`      |     ✓      |            |                                 | 也接受 `nb` |
| 波蘭文                |     `pl`      |     ✓      |            |                                 |                    |
| 葡萄牙文 (葡萄牙) |    `pt-PT`    |     ✓      |            |                                 | 也接受 `pt` |
| 葡萄牙文 (巴西)   |    `pt-BR`    |     ✓      |            |                                 |                    |
| 俄文              |     `ru`      |     ✓      |            |                                 |                    |
| 西班牙文               |     `es`      |     ✓      |     ✓       |              2020-04-01                   |                    |
| 瑞典文               |     `sv`      |     ✓      |            |                                 |                    |
| 土耳其文               |     `tr`      |     ✓      |            |                                 |                    |

#### <a name="key-phrase-extraction"></a>[關鍵片語擷取](#tab/key-phrase-extraction)

| Language              | 語言代碼 | v2 支援 | v3 支援 | 從 v3 模型版本開始提供： |       備註        |
|:----------------------|:-------------:|:----------:|:----------:|:-----------------------------------------:|:------------------:|
| 荷蘭文                 |     `nl`      |     ✓      |     ✓      |                2019-10-01                 |                    |
| 英文               |     `en`      |     ✓      |     ✓      |                2019-10-01                 |                    |
| 芬蘭文               |     `fi`      |     ✓      |     ✓      |                2019-10-01                 |                    |
| 法文                |     `fr`      |     ✓      |     ✓      |                2019-10-01                 |                    |
| 德文                |     `de`      |     ✓      |     ✓      |                2019-10-01                 |                    |
| 義大利文               |     `it`      |     ✓      |     ✓      |                2019-10-01                 |                    |
| 日文              |     `ja`      |     ✓      |     ✓      |                2019-10-01                 |                    |
| 韓文                |     `ko`      |     ✓      |     ✓      |                2019-10-01                 |                    |
| 挪威文 (巴克摩)   |     `no`      |     ✓      |     ✓      |                2019-10-01                 | 也接受 `nb` |
| 波蘭文                |     `pl`      |     ✓      |     ✓      |                2019-10-01                 |                    |
| 葡萄牙文 (葡萄牙) |    `pt-PT`    |     ✓      |     ✓      |                2019-10-01                 | 也接受 `pt` |
| 葡萄牙文 (巴西)   |    `pt-BR`    |     ✓      |     ✓      |                2019-10-01                 |                    |
| 俄文               |     `ru`      |     ✓      |     ✓      |                2019-10-01                 |                    |
| 西班牙文               |     `es`      |     ✓      |     ✓      |                2019-10-01                 |                    |
| 瑞典文               |     `sv`      |     ✓      |     ✓      |                2019-10-01                 |                    |

#### <a name="entity-linking"></a>[實體連結](#tab/entity-linking)

| Language | 語言代碼 | v2 支援 | v3 支援 | 從 v3 模型版本開始提供： | 備註 |
|:---------|:-------------:|:----------:|:----------:|:-----------------------------------------:|:-----:|
| 英文  |     `en`      |     ✓      |     ✓      |                2019-10-01                 |       |
| 西班牙文  |     `es`      |     ✓      |     ✓      |                2019-10-01                 |       |

#### <a name="language-detection"></a>[語言偵測](#tab/language-detection)

文字分析 API 可以偵測各式各樣的語言、變體、方言，以及某些區域/文化語言。  語言偵測會傳回語言的「指令碼」。 例如，針對片語 "I have a dog"，它會傳回 `en` 而不是 `en-US`。 唯一的特殊案例是中文，若它可根據提供的文字決定指令碼，語言偵測功能會傳回 `zh_CHS` 或 `zh_CHT`。 若無法識別中文文件的特定指令碼，則只會傳回 `zh`。

我們未發佈這項功能確切的語言清單，但它可以偵測到多種不同的語言、變體、方言，以及某些區域性/文化語言。 

如果您有以較不常用的語言表示的內容，您可以嘗試使用「語言偵測」，看它是否會傳回代碼。 對於無法偵測到的語言，會產生 `unknown` 回應。

---

## <a name="see-also"></a>請參閱

* [什麼是文字分析 API？](overview.md)   

---
title: 包含檔案
description: 包含檔案
services: storage
author: roygara
ms.service: storage
ms.topic: include
ms.date: 12/27/2019
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: 7fd91e898c12a13e35ae8b9055ebb5a57de2a051
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/01/2020
ms.locfileid: "89272170"
---
Azure 檔案儲存體提供了四種不同的儲存體，分別是進階、交易最佳化、經常性存取和非經常性存取，使您可以根據案例的效能和價格需求來量身打造檔案共用：

- **進階**：進階檔案共用是由固態硬碟 (SSD) 所支援，且會以 **FileStorage 儲存體帳戶**類型進行部署。 進階檔案共用可提供持續高效能和低延遲，能在毫秒內處理大部分的 IO 作業，適用於 IO 密集型工作負載。 這使其適合各種不同的工作負載，例如資料庫、網站託管以及開發環境等等。 
- **交易最佳化**：交易最佳化檔案共用可讓交易繁重的工作負載不需要進階檔案共用所提供的延遲功能。 交易最佳化檔案共用會在硬碟 (HDD) 所支援的標準儲存硬體上提供，並且會以**一般用途版本 2 (GPv2) 儲存體帳戶**類型進行部署。 這一層的儲存體原本稱為「標準」層，不過這是指儲存體媒體類型，而不是存取層本身 (經常性存取和非經常性存取也是「標準」層，因其位於標準儲存硬體上)。
- **經常性**：經常性檔案共用可針對一般用途的檔案共用案例 (例如小組共用和 Azure 檔案同步) 提供儲存體最佳化。經常性檔案共用會在 HDD 所支援的標準儲存硬體上提供，並且會以**一般用途版本 2 (GPv2) 儲存體帳戶**類型進行部署。
- **非經常性**：非經常性檔案共用可針對線上封存儲存案例提供符合成本效益的儲存體最佳化。 Azure 檔案同步也可能適合變換較小的工作負載。 非經常性檔案共用會在 HDD 所支援的標準儲存硬體上提供，並且會以**一般用途版本 2 (GPv2) 儲存體帳戶**類型進行部署。

進階檔案共用僅適用於佈建計費模型。 如需進階檔案共用佈建計費模型的詳細資訊，請參閱[了解如何佈建進階檔案共用](../articles/storage/files/storage-files-planning.md#understanding-provisioning-for-premium-file-shares)。 標準檔案共用 (包括交易最佳化、經常性存取和非經常性存取檔案共用) 都可使用「隨用隨付」模型。

經常性存取和非經常性存取檔案共用目前可於下列公用區域子集中使用 (交易最佳化檔案共用可在所有 Azure 區域中使用)：

- 澳大利亞中部
- 澳大利亞中部 2
- 澳大利亞東部
- 澳大利亞東南部
- 巴西南部
- 加拿大東部
- 加拿大中部
- 法國中部
- 法國南部
- 德國北部 (公開)
- 德國中西部 (公開)
- 印度中部
- 印度南部
- 印度西部
- 日本東部
- 日本西部
- 南韓中部
- 南韓南部
- 挪威東部
- 挪威西部
- 南非北部
- 南非西部
- 瑞士北部
- 瑞士西部
- 阿拉伯聯合大公國中部
- 阿拉伯聯合大公國北部
- 英國南部
- 英國西部
- 美國中北部
- 美國中南部

若要部署經常性存取或非經常性存取檔案共用，請參閱[建立經常性存取或非經常性存取檔案共用](../articles/storage/files/storage-how-to-create-file-share.md#create-a-hot-or-cool-file-share)。 
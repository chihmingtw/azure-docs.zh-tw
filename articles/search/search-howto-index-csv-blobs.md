---
title: 搜尋 CSV blob
titleSuffix: Azure Cognitive Search
description: 使用 delimitedText 剖析模式，從 Azure Blob 儲存體解壓縮並匯入 CSV。
manager: nitinme
author: mgottein
ms.author: magottei
ms.devlang: rest-api
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 07/11/2020
ms.openlocfilehash: 14846761535a77f28adbd0147d244817cb799d86
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/26/2020
ms.locfileid: "88935835"
---
# <a name="how-to-index-csv-blobs-using-delimitedtext-parsing-mode-and-blob-indexers-in-azure-cognitive-search"></a>如何在 Azure 認知搜尋中使用 delimitedText 剖析模式和 Blob 索引子來編制 CSV blob 的索引

依預設， [Azure 認知搜尋 blob 索引子](search-howto-indexing-azure-blob-storage.md) 會將分隔的文字 blob 剖析為單一的文字區塊。 不過，使用包含 CSV 資料的 blob，您通常想要將 blob 中的每一行當做個別文件。 例如，在給訂下列分隔文字時，您可能想要將它剖析為 2 個文件，每個都包含 [識別碼]、[發佈日期] 和 [標籤] 欄位： 

```text
id, datePublished, tags
1, 2016-01-12, "azure-search,azure,cloud"
2, 2016-07-07, "cloud,mobile"
```

在本文中，您將瞭解如何藉由設定剖析模式，使用 Azure 認知搜尋 blob 索引子來剖析 CSV blob `delimitedText` 。 

> [!NOTE]
> 遵循 [一對多索引](search-howto-index-one-to-many-blobs.md) 中的索引子設定建議，從一個 Azure blob 輸出多個搜尋檔。

## <a name="setting-up-csv-indexing"></a>設定 CSV 編製索引
若要編制 CSV blob 的索引，請 `delimitedText` 在 [建立索引子](/rest/api/searchservice/create-indexer) 要求上使用剖析模式建立或更新索引子定義：

```http
    {
      "name" : "my-csv-indexer",
      ... other indexer properties
      "parameters" : { "configuration" : { "parsingMode" : "delimitedText", "firstLineContainsHeaders" : true } }
    }
```

`firstLineContainsHeaders` 指出每個 blob 的第一個 (非空白) 行包含標頭。
如果 blob 不包含初始的標頭行，應該在索引子組態中指定標頭︰ 

```http
"parameters" : { "configuration" : { "parsingMode" : "delimitedText", "delimitedTextHeaders" : "id,datePublished,tags" } } 
```

您可以使用 `delimitedTextDelimiter` 組態集來自訂分隔符號字元。 例如：

```http
"parameters" : { "configuration" : { "parsingMode" : "delimitedText", "delimitedTextDelimiter" : "|" } }
```

> [!NOTE]
> 目前，只支援 UTF-8 編碼。 如果您需要支援其他編碼，請在 [UserVoice](https://feedback.azure.com/forums/263029-azure-search) 票選該編碼。

> [!IMPORTANT]
> 當您使用帶分隔符號的文字剖析模式時，Azure 認知搜尋會假設資料來源中的所有 blob 都是 CSV。 如果您需要支援在相同的資料來源中混用 CSV 和非 CSV Blob，請在 [UserVoice](https://feedback.azure.com/forums/263029-azure-search) 票選該編碼。
> 
> 

## <a name="request-examples"></a>要求範例
總而言之，以下是完整的承載範例。 

資料來源： 

```http
    POST https://[service name].search.windows.net/datasources?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key]

    {
        "name" : "my-blob-datasource",
        "type" : "azureblob",
        "credentials" : { "connectionString" : "DefaultEndpointsProtocol=https;AccountName=<account name>;AccountKey=<account key>;" },
        "container" : { "name" : "my-container", "query" : "<optional, my-folder>" }
    }   
```

索引子：

```http
    POST https://[service name].search.windows.net/indexers?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key]

    {
      "name" : "my-csv-indexer",
      "dataSourceName" : "my-blob-datasource",
      "targetIndexName" : "my-target-index",
      "parameters" : { "configuration" : { "parsingMode" : "delimitedText", "delimitedTextHeaders" : "id,datePublished,tags" } }
    }
```

## <a name="help-us-make-azure-cognitive-search-better"></a>協助我們改善 Azure 認知搜尋
如果您有功能要求或改進的想法，請在 [UserVoice](https://feedback.azure.com/forums/263029-azure-search/) 提供。
---
title: 透過範例瞭解 Azure Data Factory 定價
description: 此文章透過詳細範例說明及示範 Azure Data Factory 的定價模型
documentationcenter: ''
author: djpmsft
ms.author: daperlov
manager: jroth
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 12/27/2019
ms.openlocfilehash: d679dbb7a14767b83d6508e4b1e637584f33210a
ms.sourcegitcommit: e69bb334ea7e81d49530ebd6c2d3a3a8fa9775c9
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2020
ms.locfileid: "88949946"
---
# <a name="understanding-data-factory-pricing-through-examples"></a>透過範例了解 Data Factory 定價

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

此文章透過詳細範例說明及示範 Azure Data Factory 的定價模型。

> [!NOTE]
> 下列範例中使用的價格為假設，並非表示實際定價。

## <a name="copy-data-from-aws-s3-to-azure-blob-storage-hourly"></a>每小時將資料從 AWS S3 複製到 Azure Blob 儲存體

在此案例中，您希望依每小時的排程將資料從 AWS S3 複製到 Azure Blob 儲存體。

若要完成案例，您需要使用下列項目建立管線：

1. 複製活動，包含要從 AWS S3 複製之資料的輸入資料集。

2. Azure 儲存體上之資料的輸出資料集。

3. 每小時執行管線的排程觸發程序。

   ![案例 1](media/pricing-concepts/scenario1.png)

| **作業** | **類型與單位** |
| --- | --- |
| 建立連結的服務 | 2 個讀取/寫入實體  |
| 建立資料集 | 4 個讀取/寫入實體 (2 個用於建立資料集，2 個用於連結的服務參考) |
| 建立管線 | 3 個讀取/寫入實體 (1 個用於建立管線，2 個用於資料集參考) |
| 取得管線 | 1 個讀取/寫入實體 |
| 執行管線 | 2 個活動執行 (1 個用於觸發程序執行，1 個用於活動執行) |
| 複製資料假設︰執行時間 = 10 分鐘 | 10 \* 4 Azure 整合執行階段 (預設 DIU 設定 = 4) 如需有關資料整合單位和最佳化複製效能的詳細資訊，請參閱[此文章](copy-activity-performance.md) |
| 監視管線假設：僅發生 1 次執行 | 2 個重試的監視執行記錄 (1 個用於管線執行，1 個用於活動執行) |

**總案例定價：$0.16811**

- Data Factory 作業 = **$0.0001**
  - 讀取/寫入 = 10\*00001 = $0.0001 [1 讀取/寫入 = $0.50/50000 = 0.00001]
  - 監視 = 2\*000005 = $0.00001 [1 監視 = $0.25/50000 = 0.000005]
- 管線協調流程 &amp; 執行 = **$0.168**
  - 活動執行 = 001\*2 = 0.002 [1 執行 = $1/1000 = 0.001]
  - 資料移動活動 = $0.166 (依比例分配 10 分鐘的執行時間。 Azure 整合執行階段上每小時 0.25 美元)

## <a name="copy-data-and-transform-with-azure-databricks-hourly"></a>每小時複製資料，並使用 Azure Databricks 轉換

在此案例中，您希望依每小時的排程將資料從 AWS S3 複製到 Azure Blob 儲存體，並使用 Azure Databricks 轉換。

若要完成案例，您需要使用下列項目建立管線：

1. 一個複製活動，包含要從 AWS S3 複製資料的輸入資料集，以及 Azure 儲存體上之資料的輸出資料集。
2. 一個用於資料轉換的 Azure Databricks 活動。
3. 一個每小時執行管線的排程觸發程序。

![案例 2](media/pricing-concepts/scenario2.png)

| **作業** | **類型與單位** |
| --- | --- |
| 建立連結的服務 | 3 個讀取/寫入實體  |
| 建立資料集 | 4 個讀取/寫入實體 (2 個用於建立資料集，2 個用於連結的服務參考) |
| 建立管線 | 3 個讀取/寫入實體 (1 個用於建立管線，2 個用於資料集參考) |
| 取得管線 | 1 個讀取/寫入實體 |
| 執行管線 | 3 個活動執行 (1 個用於觸發程序執行，2 個用於活動執行) |
| 複製資料假設︰執行時間 = 10 分鐘 | 10 \* 4 Azure 整合執行階段 (預設 DIU 設定 = 4) 如需有關資料整合單位和最佳化複製效能的詳細資訊，請參閱[此文章](copy-activity-performance.md) |
| 監視管線假設：僅發生 1 次執行 | 3 個重試的監視執行記錄 (1 個用於管線執行，2 個用於活動執行) |
| 執行 Databricks 活動假設：執行時間 = 10 分鐘 | 10 分鐘外部管線活動執行 |

**總案例定價：$0.16916**

- Data Factory 作業 = **$0.00012**
  - 讀取/寫入 = 11\*00001 = $0.00011 [1 R/W = $0.50/50000 = 0.00001]
  - 監視 = 3\*000005 = $0.00001 [1 監視 = $0.25/50000 = 0.000005]
- 管線協調流程 &amp; 執行 = **$0.16904**
  - 活動執行 = 001\*3 = 0.003 [1 執行 = $1/1000 = 0.001]
  - 資料移動活動 = $0.166 (依比例分配 10 分鐘的執行時間。 Azure 整合執行階段上每小時 0.25 美元)
  - 外部管線活動 = $0.000041 (依比例分配 10 分鐘的執行時間。 Azure 整合執行階段上每小時 0.00025 美元)

## <a name="copy-data-and-transform-with-dynamic-parameters-hourly"></a>每小時複製資料並使用動態參數進行轉換

在此案例中，您希望依每小時的排程將資料從 AWS S3 複製到 Azure Blob 儲存體，並使用 Azure Databricks (透過指令碼中的動態參數) 轉換。

若要完成案例，您需要使用下列項目建立管線：

1. 一個複製活動，包含要從 AWS S3 複製資料的輸入資料集、Azure 儲存體上之資料的輸出資料集。
2. 一個 Lookup 活動，用於以動態方式將參數傳遞到轉換指令碼。
3. 一個用於資料轉換的 Azure Databricks 活動。
4. 一個每小時執行管線的排程觸發程序。

![案例 3](media/pricing-concepts/scenario3.png)

| **作業** | **類型與單位** |
| --- | --- |
| 建立連結的服務 | 3 個讀取/寫入實體  |
| 建立資料集 | 4 個讀取/寫入實體 (2 個用於建立資料集，2 個用於連結的服務參考) |
| 建立管線 | 3 個讀取/寫入實體 (1 個用於建立管線，2 個用於資料集參考) |
| 取得管線 | 1 個讀取/寫入實體 |
| 執行管線 | 4 個活動執行 (1 個用於觸發程序執行，3 個用於活動執行) |
| 複製資料假設︰執行時間 = 10 分鐘 | 10 \* 4 Azure 整合執行階段 (預設 DIU 設定 = 4) 如需有關資料整合單位和最佳化複製效能的詳細資訊，請參閱[此文章](copy-activity-performance.md) |
| 監視管線假設：僅發生 1 次執行 | 4 個重試的監視執行記錄 (1 個用於管線執行，3 個用於活動執行) |
| 執行 Lookup 活動假設：執行時間 = 1 分鐘 | 1 分鐘管線活動執行 |
| 執行 Databricks 活動假設：執行時間 = 10 分鐘 | 10 分鐘外部管線活動執行 |

**總案例定價：$0.17020**

- Data Factory 作業 = **$0.00013**
  - 讀取/寫入 = 11\*00001 = $0.00011 [1 R/W = $0.50/50000 = 0.00001]
  - 監視 = 4\*000005 = $0.00002 [1 監視 = $0.25/50000 = 0.000005]
- 管線協調流程 &amp; 執行 = **$0.17007**
  - 活動執行 = 001\*4 = 0.004 [1 執行 = $1/1000 = 0.001]
  - 資料移動活動 = $0.166 (依比例分配 10 分鐘的執行時間。 Azure 整合執行階段上每小時 0.25 美元)
  - 管線活動 = $0.00003 (依比例分配 1 分鐘的執行時間。 Azure 整合執行階段上每小時 $0.002 美元)
  - 外部管線活動 = $0.000041 (依比例分配 10 分鐘的執行時間。 Azure 整合執行階段上每小時 0.00025 美元)

## <a name="using-mapping-data-flow-debug-for-a-normal-workday"></a>針對一般 workday 使用對應資料流程的資料流程

如果您是資料工程師，您必須負責設計、建立及測試每天的對應資料流程程。 您會在早上登入 ADF UI，並啟用資料流程的偵測模式。 Debug 會話的預設 TTL 為60分鐘。 您的工作會在一天內完成8小時，因此您的 Debug 會話永遠不會過期。 因此，您當天的費用將會是：

**8 (小時) x 8 (計算優化核心) x $0.193 = $12.35**

## <a name="transform-data-in-blob-store-with-mapping-data-flows"></a>使用對應資料流程轉換 blob 存放區中的資料

在此案例中，您想要以每小時的排程，在 ADF 對應資料流程中以視覺化方式轉換 Blob 存放區中的資料。

若要完成案例，您需要使用下列項目建立管線：

1. 具有轉換邏輯的資料流程活動。

2. Azure 儲存體上資料的輸入資料集。

3. Azure 儲存體上之資料的輸出資料集。

4. 每小時執行管線的排程觸發程序。

| **作業** | **類型與單位** |
| --- | --- |
| 建立連結的服務 | 2 個讀取/寫入實體  |
| 建立資料集 | 4 個讀取/寫入實體 (2 個用於建立資料集，2 個用於連結的服務參考) |
| 建立管線 | 3 個讀取/寫入實體 (1 個用於建立管線，2 個用於資料集參考) |
| 取得管線 | 1 個讀取/寫入實體 |
| 執行管線 | 2 個活動執行 (1 個用於觸發程序執行，1 個用於活動執行) |
| 資料流程假設：執行時間 = 10 分鐘 + 10 分鐘的 TTL | \*具有 TTL 10 的一般計算10個核心 |
| 監視管線假設：僅發生 1 次執行 | 2 個重試的監視執行記錄 (1 個用於管線執行，1 個用於活動執行) |

**總案例定價： $1.4631**

- Data Factory 作業 = **$0.0001**
  - 讀取/寫入 = 10\*00001 = $0.0001 [1 讀取/寫入 = $0.50/50000 = 0.00001]
  - 監視 = 2\*000005 = $0.00001 [1 監視 = $0.25/50000 = 0.000005]
- 管線協調流程 &amp; 執行 = **$1.463**
  - 活動執行 = 001\*2 = 0.002 [1 執行 = $1/1000 = 0.001]
  - 資料流程活動 = $1.461 的20分鐘比例 (10 分鐘的執行時間 + 10 分鐘的 TTL) 。 在 Azure Integration Runtime 上使用 $ 0.274/小時，一般計算為16個核心

## <a name="data-integration-in-azure-data-factory-managed-vnet"></a>Azure Data Factory 受控 VNET 中的資料整合
在此案例中，您想要刪除 Azure Blob 儲存體上的原始檔案，並將資料從 Azure SQL Database 複製到 Azure Blob 儲存體。 您將在不同的管線上執行兩次。 這兩個管線的執行時間重迭。
![Scenario4 ](media/pricing-concepts/scenario-4.png) 若要完成此案例，您必須使用下列專案建立兩個管線：
  - 管線活動–刪除活動。
  - 複製活動，其中包含要從 Azure Blob 儲存體複製之資料的輸入資料集。
  - Azure SQL Database 上資料的輸出資料集。
  - 執行管線的排程觸發程式。


| **作業** | **類型與單位** |
| --- | --- |
| 建立連結的服務 | 4個讀取/寫入實體 |
| 建立資料集 | 8個讀取/寫入實體 (4 用於建立資料集，4個用於連結的服務參考)  |
| 建立管線 | 6個讀取/寫入實體 (2 以建立管線，4個用於資料集參考)  |
| 取得管線 | 2 個讀取/寫入實體 |
| 執行管線 | 6個活動執行 (2 個用於觸發程式執行，4個用於活動執行)  |
| 執行刪除活動：每次執行時間 = 5 分鐘。第一個管線中的「刪除活動」執行是從 10:00 AM UTC 到 10:05 AM UTC。 第二個管線中的「刪除活動」執行是從 10:02 AM UTC 到 10:07 AM UTC。|受管理的 VNET 中的總7分鐘管線活動執行。 管線活動最多可支援50個受管理 VNET 的平行存取。 |
| 資料複製假設：每次執行時間 = 10 分鐘。第一個管線中的複製執行是從 10:06 AM UTC 到 10:15 AM UTC。 第二個管線中的「刪除活動」執行是從 10:08 AM UTC 到 10:17 AM UTC。 | 10 * 4 Azure Integration Runtime (預設 DIU 設定 = 4) 如需資料整合單位的詳細資訊和優化複製效能的詳細資訊，請參閱 [這篇文章](copy-activity-performance.md) |
| 監視管線假設：只發生2個執行 | 6個重新嘗試的監視執行記錄 (2 個用於管線執行，4個用於活動執行)  |


**總案例定價： $0.45523**

- Data Factory 作業 = $0.00023
  - 讀取/寫入 = 20 * 00001 = $0.0002 [1 R/W = $ 0.50/50000 = 0.00001]
  - 監視 = 6 * >000005 = $0.00003 [1 監視 = $ 0.25/50000 = 0.000005]
- 管線協調流程 & 執行 = $0.455
  - 活動執行 = 0.001 * 6 = 0.006 [1 執行 = $ 1/1000 = 0.001]
  - 資料移動活動 = $0.333 (按比例執行10分鐘的執行時間。 Azure 整合執行階段上每小時 0.25 美元)
  - 管線活動 = $0.116 (在執行時間的7分鐘按比例計費。 Azure Integration Runtime) 每小時 $ 1

> [!NOTE]
> 這些價格僅供範例之用。

**常見問題集**

問：如果我想要執行50以上的管線活動，這些活動可以同時執行嗎？

答：將允許最大的50並行管線活動。  51th 管線活動將會排入佇列，直到開啟「可用位置」為止。 適用于外部活動。 將允許最大800並行外部活動。

## <a name="next-steps"></a>接下來的步驟

既然您了解 Azure Data Factory 的定價，您可以立即開始！

- [使用 Azure Data Factory UI 來建立資料處理站](quickstart-create-data-factory-portal.md)

- [Azure Data Factory 簡介](introduction.md)

- [Azure Data Factory 中的視覺化撰寫](author-visually.md)

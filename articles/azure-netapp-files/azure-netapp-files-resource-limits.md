---
title: Azure NetApp Files 的資源限制 | Microsoft Docs
description: 說明 Azure NetApp Files 資源的限制，以及如何要求增加資源限制。
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 08/21/2020
ms.author: b-juche
ms.openlocfilehash: 9facbc1629b8e1330c6bbafb4444d5bfc237d16f
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/22/2020
ms.locfileid: "88752305"
---
# <a name="resource-limits-for-azure-netapp-files"></a>Azure NetApp Files 的資源限制

了解 Azure NetApp Files 的資源限制有助於管理磁碟區。

## <a name="resource-limits"></a>資源限制

下表說明 Azure NetApp Files 的資源限制：

|  資源  |  預設限制  |  可透過支援要求進行調整  |
|----------------|---------------------|--------------------------------------|
|  每個 Azure 區域的 NetApp 帳戶數目   |  10    |  是   |
|  每一 NetApp 帳戶的容量集區數目   |    25     |   是   |
|  每個訂用帳戶的磁片區數目   |    500     |   是   |
|  每個容量集區的磁片區數目     |    500   |    是     |
|  每個磁片區的快照集數目       |    255     |    否        |
|  委派至 Azure NetApp Files 的子網數目 (Microsoft) 每個 Azure 虛擬網路的 NetApp/磁片區    |   1   |    否    |
|  VNet 中使用的 Ip 數目 (包括立即對等互連 Vnet) 與 Azure NetApp Files   |    1000   |    否   |
|  單一容量集區的大小下限   |  4 TiB     |    否  |
|  單一容量集區的大小上限    |  500 TiB   |   否   |
|  單一磁片區的大小下限    |    100 GiB    |    否    |
|  單一磁片區的大小上限     |    100 TiB    |    否    |
|  單一檔案的大小上限     |    16 TiB    |    否    |    
|  單一目錄中目錄中繼資料的大小上限      |    320 MB    |    否    |    
|  每個磁片區 ([maxfiles](#maxfiles)) 的檔案數目上限     |    1 億    |    是    |    

如需詳細資訊，請參閱 [容量管理常見問題](azure-netapp-files-faqs.md#capacity-management-faqs)。

## <a name="maxfiles-limits"></a>Maxfiles 限制 <a name="maxfiles"></a> 

Azure NetApp Files 磁片區有一個稱為 *maxfiles*的限制。 Maxfiles 限制是磁片區可包含的檔案數目。 Azure NetApp Files 磁片區的 maxfiles 限制是根據磁片區 (配額) 的大小來編制索引。 磁片區的 maxfiles 限制會隨著每個 TiB 的布建磁片區大小，以20000000檔案的速率增加或減少。 

服務會根據布建的大小，動態調整磁片區的 maxfiles 限制。 例如，一開始設定大小為 1 TiB 的磁片區，其 maxfiles 限制為20000000。 磁片區大小的後續變更會根據下列規則，自動 readjustment maxfiles 限制： 

|    磁片區大小 (配額)      |  自動 readjustment maxfiles 限制    |
|----------------------------|-------------------|
|    < 1 TiB                 |    2 千萬     |
|    >= 1 TiB 但 < 2 TiB    |    40000000     |
|    >= 2 TiB 但 < 3 TiB    |    6000 萬     |
|    >= 3 TiB 但 < 4 TiB    |    80000000     |
|    >= 4 TiB                |    1 億    |

如果您已經為磁片區配置至少 4 TiB 配額，您可以起始 [支援要求](#limit_increase) ，將 maxfiles 限制增加到100000000以上。

## <a name="request-limit-increase"></a>要求限制增加 <a name="limit_increase"></a> 

您可以建立 Azure 支援要求，以增加上表中的可調整限制。 

從 Azure 入口網站導覽平面： 

1. 按一下 [說明 **+ 支援**]。
2. 按一下 [ **+ 新增支援要求**]。
3. 在 [基本概念] 索引標籤中提供下列資訊： 
    1. 問題類型：選取 **服務和訂用帳戶限制 (配額) **。
    2. 訂用帳戶：選取您需要增加配額的資源訂用帳戶。
    3. 配額類型：選取 **儲存體： Azure NetApp Files 的限制**。
    4. 按一下 **[下一步：方案]**。
4. 在 [詳細資料] 索引標籤上：
    1. 在 [描述] 方塊中，提供對應資源類型的下列資訊：

        |  資源  |    父資源      |    要求的新限制     |    增加配額的原因       |
        |----------------|------------------------------|---------------------------------|------------------------------------------|
        |  帳戶 |  *訂用帳戶識別碼*   |  *要求的新 **帳戶** 數目上限*    |  *提示要求的案例或使用案例為何？*  |
        |  集區    |  *訂用帳戶識別碼，帳戶 URI*  |  *要求的新 **集** 區數目上限*   |  *提示要求的案例或使用案例為何？*  |
        |  磁碟區  |  *訂用帳戶識別碼、帳戶 URI、集區 URI*   |  *要求的新 **磁片** 區數目上限*     |  *提示要求的案例或使用案例為何？*  |
        |  Maxfiles  |  *訂用帳戶識別碼、帳戶 URI、集區 URI、磁片區 URI*   |  *要求新的最大 **maxfiles** 數目*     |  *提示要求的案例或使用案例為何？*  |    

    2. 指定適當的支援方法，並提供您的合約資訊。

    3. 按一下 **[下一步]： [檢查 + 建立]** 以建立要求。 


## <a name="next-steps"></a>後續步驟  

- [了解 Azure NetApp Files 的儲存體階層](azure-netapp-files-understand-storage-hierarchy.md)
- [適用於Azure NetApp Files 的成本模型](azure-netapp-files-cost-model.md)

---
title: Azure 備份定價
description: 瞭解如何估計預算 Azure 備份定價的成本。
ms.topic: conceptual
ms.date: 06/16/2020
ms.openlocfilehash: 03ec0076d3089562ddaace6db413fb3f1ba949a6
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/20/2020
ms.locfileid: "88654526"
---
# <a name="azure-backup-pricing"></a>Azure 備份定價

若要瞭解 Azure 備份定價，請造訪 [Azure 備份定價頁面](https://azure.microsoft.com/pricing/details/backup/)。

## <a name="download-detailed-estimates-for-azure-backup-pricing"></a>下載 Azure 備份定價的詳細估計值

如果您想要估計預算或成本比較的成本，請下載詳細的 [Azure 備份定價估算器](https://aka.ms/AzureBackupCostEstimates)。  

### <a name="what-does-the-estimator-contain"></a>估算器包含什麼？

Azure 備份 cost 估算器工作表中的選項，可讓您使用 Azure 備份來估計您想要備份的所有可能工作負載。 這些工作負載包括：

- Azure VM
- 內部部署伺服器
- Azure Vm 中的 SQL
- Azure Vm 中的 SAP Hana
- Azure 檔案儲存體共用

## <a name="estimate-costs-for-backing-up-azure-vms-or-on-premises-servers"></a>估計備份 Azure Vm 或內部部署伺服器的成本

若要使用 Azure 備份估計備份 Azure Vm 或內部部署伺服器的成本，您需要下列參數：

- 您正在嘗試備份的 Vm 或內部部署伺服器的大小
  - 輸入需要備份的磁片或伺服器的「已使用大小」

- 具有該大小的伺服器數目

- 這些伺服器上預期的資料流程失量為何？<br>
  流失指的是資料中的變更量。 例如，如果您的 VM 有 200 GB 的資料要備份，且每天有 10 GB 的變更，每日變換率為5%。

  - 較高的變換表示您備份更多資料

  - **如果您**正在執行資料庫，請為檔案伺服器挑選 [**低**] 或 [**適中**]

  - 如果您知道 **流失的百分比**，可以使用 [ **輸入您自己的%** ] 選項

- 選擇備份原則

  - 您預期保留「每日」備份的時間長度為何？  (（天）) 

  - 您預期要保留「每週」備份的時間長度？  (周) 

  - 您預期要保留「每月」備份的時間長度？  (月) 

  - 您希望保留「每年」備份的時間長度？ 多年來 () 

  - 您希望保留「立即還原快照」的時間長度？  (1-5 天) 

    - 此選項可讓您使用儲存在磁片上的快照集，以快速的方式從最久還原到7天。

- **選用** –選擇性磁片備份

  - 如果您在備份 Azure Vm 時使用 **選擇性磁片備份** 選項，請選擇 [ **排除磁片** ] 選項，並根據大小來輸入從備份中排除的磁片百分比。 例如，如果您的 VM 連線到每個磁片使用 200 GB 的三個磁片，而且如果您想要從備份中排除兩個磁片，請輸入66.7%。

- **選用** –備份儲存體冗余

  - 這表示您的備份資料所送出之儲存體帳戶的冗余。 建議您使用 **GRS** 來獲得最高的可用性。 由於它可確保備份資料的複本會保留在不同的區域中，因此可協助您符合多個合規性標準。 如果您要備份不需要企業級備份的開發或測試環境，請將 [冗余] 變更為 [ **LRS** ]。 如果您想要在已啟用備份的[跨區域還原](backup-azure-arm-restore-vms.md#cross-region-restore)時瞭解成本，請在工作表中選取 [ **RAGRS** ] 選項。

- **選用** -修改區域定價或申請折扣費率

  - 如果您想要檢查您的估價是否有不同的區域或折扣費率，請針對 [**試用不同區域嗎？** ] 選項選取 [**是]** ，然後輸入您想要執行預估的費率。

## <a name="estimate-costs-for-backing-up-sql-servers-in-azure-vms"></a>估計在 Azure Vm 中備份 SQL server 的成本

若要估計使用 Azure 備份在 Azure Vm 中備份 SQL server 的成本，您需要下列參數：

- 您正在嘗試備份的 SQL server 大小

- 具有上述大小的 SQL server 數目

- SQL server 備份資料的預期壓縮為何？

  - 大部分的 Azure 備份客戶都會看到，在 **啟用**sql 壓縮時，備份資料會有80% 壓縮與 sql server 大小的比較。

  - 如果您預期會看到不同的壓縮，請在此欄位中輸入數位

- 記錄備份的預期大小為何？

  - % 表示每日記錄檔大小為 SQL server 大小的%

- 這些伺服器上預期的每日資料變換量為何？

  - 一般而言，資料庫會有「高」變換

  - 如果您知道 **流失的百分比**，可以使用 [ **輸入您自己的%** ] 選項

- 選擇備份原則

  - 備份類型

    - 您可以選擇的最有效原則是每 **日差異** ，每週/每月/每年完整備份。 Azure 備份也可以透過單鍵的方式從差異還原。

    - 您也可以選擇擁有每日/每週/每月/每年完整備份的原則。 此選項所耗用的儲存空間會比第一個選項稍微多一點。

  - 您希望保留「記錄」備份的時間長度？  (（天）) [7-35]

  - 您預期保留「每日」備份的時間長度為何？  (（天）) 

  - 您預期要保留「每週」備份的時間長度？  (周) 

  - 您預期要保留「每月」備份的時間長度？  (月) 

  - 您希望保留「每年」備份的時間長度？ 多年來 () 

- **選用** –備份儲存體冗余

  - 這表示您的備份資料所送出之儲存體帳戶的冗余。 建議您使用 **GRS** 來獲得最高的可用性。 由於它可確保備份資料的複本會保留在不同的區域中，因此可協助您符合多個合規性標準。 如果您要備份不需要企業級備份的開發或測試環境，請將 [冗余] 變更為 [ **LRS** ]。

- **選用** -修改區域定價或申請折扣費率

  - 如果您想要檢查您的估價是否有不同的區域或折扣費率，請針對 [**試用不同區域嗎？** ] 選項選取 [**是]** ，然後輸入您想要執行預估的費率。

## <a name="estimate-costs-for-backing-up-sap-hana-servers-in-azure-vms"></a>估計在 Azure Vm 中備份 SAP Hana 伺服器的成本

若要預估使用 Azure 備份在 Azure Vm 中執行 SAP Hana 伺服器的備份成本，您需要下列參數：

- 您正在嘗試備份之 SAP Hana 資料庫的大小總計。 這應該是每個資料庫的完整備份大小總和，如 SAP Hana 所報告。
- 具有上述大小的 SAP Hana 伺服器數目
- 記錄備份的預期大小為何？
  
  - % 表示在 SAP Hana 伺服器上備份的 SAP Hana 資料庫總大小的%，每日平均記錄檔大小為%
- 這些伺服器上預期的每日資料變換量為何？
  - [%] 表示平均每日變換大小，以您在 SAP Hana 伺服器上備份的 SAP Hana 資料庫總大小的% 表示
  - 一般而言，資料庫會有「高」變換
  - 如果您知道 **流失的百分比**，可以使用 [ **輸入您自己的%** ] 選項
- 選擇備份原則
  - 備份類型
    - 您可以選擇的最有效原則是每 **日差異** ，每 **周/每月/每年** 完整備份。 Azure 備份也可以透過單鍵的方式從差異還原。
    - 您也可以選擇擁有 **每日/每週/每月/每年** 完整備份的原則。 此選項所耗用的儲存空間會比第一個選項稍微多一點。
  - 您希望保留「記錄」備份的時間長度？  (（天）) [7-35]
  - 您預期保留「每日」備份的時間長度為何？  (（天）) 
  - 您預期要保留「每週」備份的時間長度？  (周) 
  - 您預期要保留「每月」備份的時間長度？  (月) 
  - 您希望保留「每年」備份的時間長度？ 多年來 () 
- **選用** –備份儲存體冗余
  
  - 這表示您的備份資料所送出之儲存體帳戶的冗余。 建議您使用 **GRS** 來獲得最高的可用性。 由於它可確保備份資料的複本會保留在不同的區域中，因此可協助您符合多個合規性標準。 如果您要備份不需要企業級備份的開發或測試環境，請將 [冗余] 變更為 [ **LRS** ]。
- **選用** -修改區域定價或申請折扣費率
  
  - 如果您想要檢查您的估價是否有不同的區域或折扣費率，請針對 [**試用不同區域嗎？** ] 選項選取 [**是]** ，然後輸入您想要執行預估的費率。
  
## <a name="estimate-costs-for-backing-up-azure-file-shares"></a>估計備份 Azure 檔案共用的成本

若要使用 Azure 備份所提供的 [快照式備份解決方案](azure-file-share-backup-overview.md) 來估計備份 Azure 檔案共用的成本，您需要下列參數：

- 您要備份之檔案共用的大小 (**GB**) 。

- 如果您想要備份分散到多個儲存體帳戶的檔案共用，請指定裝載具有上述大小之檔案共用的儲存體帳戶數目。

- 您要備份的檔案共用上預期的資料流程失量。 <br>變換指的是資料中的變更量，而它會直接影響快照儲存體大小。 例如，如果您有一個檔案共用，其中包含要備份的 200 GB 資料，而每日有 10 GB 的變更，則每日變換率為5%。
  - 較高的變換表示每日檔案共用內容中的資料變更數量很高，因此增量快照集 (只) 大小也會比較多。
  - 根據您的檔案共用特性和使用方式，選取低 (1% ) 、適中 (3% ) 或高 (5% ) 。
  - 如果您知道檔案共用的確切 **流失%** ，可以從下拉式清單中選取 [ **輸入您自己的%** ] 選項。 指定每日、每週、每月和每年變換的% )  (的值。

- 儲存體帳戶的類型 (standard 或 premium) ，以及裝載已備份檔案共用之儲存體帳戶的儲存體冗余設定。 <br>在 Azure 檔案共用的目前備份解決方案中，快照集會儲存在與備份檔案共用相同的儲存體帳戶中。 因此，與快照集相關聯的儲存體成本會以您的 Azure 檔案計費的一部分計費，並根據裝載備份檔案共用和快照集之儲存體帳戶的帳戶類型和冗余設定的快照定價。

- 不同備份的保留期
  - 您預期保留「每日」備份的時間長度為何？  (（天）) 
  - 您預期要保留「每週」備份的時間長度？  (周) 
  - 您預期要保留「每月」備份的時間長度？  (月) 
  - 您希望保留「每年」備份的時間長度？ 多年來 () 

  請參閱 [Azure 檔案共用支援矩陣](azure-file-share-support-matrix.md#retention-limits) ，以取得每個類別中支援的最大保留值。

- **選用** –修改區域定價或申請折扣費率。
  - 預設值是針對「美國東部」區域，在估算器中為「快照集儲存體成本」和「受保護的實例成本」設定的預設值。 如果您想要檢查您的估價是否有不同的區域或折扣費率，請針對 [**試用不同區域嗎？** ] 選項選取 **[是]** ，然後輸入您想要執行預估的費率。

## <a name="next-steps"></a>後續步驟

[什麼是 Azure 備份服務？](backup-overview.md)

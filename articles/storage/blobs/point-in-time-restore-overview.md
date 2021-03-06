---
title: '區塊 blob 的時間點還原 (預覽) '
titleSuffix: Azure Storage
description: 區塊 blob 的時間點還原可讓您將儲存體帳戶還原至特定時間點的先前狀態，以提供防止意外刪除或損毀的保護。
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 06/10/2020
ms.author: tamram
ms.subservice: blobs
ms.custom: references_regions, devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: 07a1e81f2fe63ea8d34f362627ad79ab6729f3f0
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/01/2020
ms.locfileid: "89229118"
---
# <a name="point-in-time-restore-for-block-blobs-preview"></a>區塊 blob 的時間點還原 (預覽) 

時間點還原可讓您將區塊 blob 資料還原至先前的狀態，以提供防止意外刪除或損毀的保護。 時間點還原適用于使用者或應用程式不小心刪除資料或應用程式錯誤損毀資料的案例。 時間點還原也可讓測試案例必須將資料集還原為已知狀態，再執行進一步的測試。

若要瞭解如何啟用儲存體帳戶的時間點還原，請參閱 [啟用和管理區塊 blob 的時間點還原 (預覽) ](point-in-time-restore-manage.md)。

## <a name="how-point-in-time-restore-works"></a>時間點還原的運作方式

若要啟用時間點還原，您必須建立儲存體帳戶的管理原則，並指定保留期限。 在保留期限內，您可以從目前狀態將區塊 blob 還原至先前時間點的狀態。

若要起始時間點還原，請呼叫 [還原 Blob 範圍](/rest/api/storagerp/storageaccounts/restoreblobranges) 作業，並指定 UTC 時間的還原點。 您可以指定要還原之容器和 blob 名稱的字典範圍，或略過範圍以還原儲存體帳戶中的所有容器。 每個還原作業最多可支援10個字典的範圍。

Azure 儲存體會分析在要求的還原點（以 UTC 時間指定）和目前的時間之間，對指定的 blob 進行的所有變更。 還原作業是不可部分完成的，因此它會完全成功還原所有變更，否則會失敗。 如果有任何無法還原的 blob，則作業會失敗，且受影響容器的讀取和寫入作業會繼續。

一次只能在一個儲存體帳戶上執行一個還原作業。 還原作業在進行中時無法取消，但可以執行第二個還原作業來復原第一項作業。

**還原 Blob 範圍**作業會傳回可唯一識別作業的還原識別碼。 若要檢查時間點還原的狀態，請使用「**還原 Blob 範圍**」作業所傳回的還原識別碼，呼叫「**取得還原狀態**」作業。

請記住下列關於還原作業的限制：

- 已透過 [Put 區塊](/rest/api/storageservices/put-block) 或 [PUT 區塊從 URL](/rest/api/storageservices/put-block-from-url)上傳，但未透過 [put block List](/rest/api/storageservices/put-block-list)認可的區塊不是 blob 的一部分，因此不會在還原作業中還原。
- 無法還原具有作用中租用的 blob。 如果包含作用中租用的 blob 包含在要還原的 blob 範圍內，則還原作業會以不可部分完成的方式失敗。
- 在還原作業中，不會建立或刪除快照集。 只有基底 blob 會還原為先前的狀態。
- 如果在目前的時間和還原點之間的期間內，將 blob 移至經常性存取層與非經常性存取層之間，則會將 blob 還原至先前的層級。 不過，已移至封存層的 blob 將不會還原。

> [!IMPORTANT]
> 當您執行還原作業時，Azure 儲存體會封鎖作業期間正在還原之範圍中 blob 的資料作業。 主要位置會封鎖讀取、寫入和刪除作業。 基於這個理由，在進行還原作業時，在 Azure 入口網站中列出容器的作業可能不會如預期般執行。
>
> 如果儲存體帳戶是異地複寫的，則在還原作業期間，次要位置的讀取作業可能會繼續進行。

> [!CAUTION]
> 時間點還原只支援在區塊 blob 上還原作業。 無法還原容器上的作業。 如果您在時間點還原預覽版期間呼叫「 [刪除容器](/rest/api/storageservices/delete-container) 」作業來刪除儲存體帳戶中的容器，則無法使用還原作業來還原該容器。 在預覽期間，如果您想要還原，請刪除個別的 blob，而不是刪除容器。

### <a name="prerequisites-for-point-in-time-restore"></a>時間點還原的必要條件

還原時間點需要啟用下列 Azure 儲存體功能：

- [虛刪除](soft-delete-overview.md)
- [變更摘要 (預覽) ](storage-blob-change-feed.md)
- [Blob 版本設定](versioning-overview.md)

啟用還原時間點之前，請先為儲存體帳戶啟用這些功能。 請務必先註冊變更摘要和 Blob 版本設定預覽，然後才能加以啟用。

### <a name="retention-period-for-point-in-time-restore"></a>時間點還原的保留期間

當您針對儲存體帳戶啟用時間點還原時，會指定保留期限。 您儲存體帳戶中的區塊 blob 可在保留期間內還原。

當您啟用時間點還原時，就會開始保留期限。 請記住，您無法將 blob 還原至保留期間開始之前的狀態。 例如，如果您在5月1日啟用時間點還原，保留30天，則在5月15日之前，您最多可以還原15天。 在6月1日，您可以從1到30天還原資料。

時間點還原的保留期限必須至少為一天，小於針對虛刪除指定的保留週期。 例如，如果虛刪除保留期限設定為7天，則時間點還原保留期限可能介於1到6天之間。

### <a name="permissions-for-point-in-time-restore"></a>時間點還原的許可權

若要起始還原作業，用戶端必須具有儲存體帳戶中所有容器的寫入權限。 若要使用 Azure Active Directory (Azure AD) 授與授權還原作業的許可權，請將 **儲存體帳戶參與者** 角色指派給儲存體帳戶、資源群組或訂用帳戶層級的安全性主體。

## <a name="about-the-preview"></a>關於預覽

僅限一般用途 v2 儲存體帳戶支援還原時間點。 只有經常性存取和非經常性存取層中的資料可以使用時間點還原來還原。

下欄區域支援預覽版中的時間點還原：

- 加拿大中部
- 加拿大東部
- 法國中部

預覽版包含下列限制：

- 不支援還原 premium 區塊 blob。
- 不支援還原封存層中的 Blob。 例如，如果經常性存取層的 Blob 在兩天前已移至封存層，且還原作業還原至三天前的某個時間點，則 Blob 不會還原至經常性存取層。
- 不支援還原 Azure Data Lake Storage Gen2 的平面和階層命名空間。
- 不支援使用客戶提供的金鑰來還原儲存體帳戶。

> [!IMPORTANT]
> 時間點還原預覽僅供非生產環境使用。 生產環境的服務等級協定 (SLA) 目前無法使用。

### <a name="register-for-the-preview"></a>註冊預覽

若要註冊預覽，請執行下列命令：

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```powershell
# Register for the point-in-time restore preview
Register-AzProviderFeature -FeatureName RestoreBlobRanges -ProviderNamespace Microsoft.Storage

# Register for change feed (preview)
Register-AzProviderFeature -FeatureName Changefeed -ProviderNamespace Microsoft.Storage

# Register for Blob versioning
Register-AzProviderFeature -FeatureName Versioning -ProviderNamespace Microsoft.Storage

# Refresh the Azure Storage provider namespace
Register-AzResourceProvider -ProviderNamespace Microsoft.Storage
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az feature register --namespace Microsoft.Storage --name RestoreBlobRanges
az feature register --namespace Microsoft.Storage --name Changefeed
az feature register --namespace Microsoft.Storage --name Versioning
az provider register --namespace 'Microsoft.Storage'
```

---

### <a name="check-registration-status"></a>檢查註冊狀態

時間點還原的註冊會自動進行，且應不到10分鐘。 若要檢查您的註冊狀態，請執行下列命令：

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```powershell
Get-AzProviderFeature -ProviderNamespace Microsoft.Storage `
    -FeatureName RestoreBlobRanges

Get-AzProviderFeature -ProviderNamespace Microsoft.Storage `
    -FeatureName Changefeed

Get-AzProviderFeature -ProviderNamespace Microsoft.Storage `
    -FeatureName Versioning
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az feature list -o table --query "[?contains(name, 'Microsoft.Storage/RestoreBlobRanges')].{Name:name,State:properties.state}"
az feature list -o table --query "[?contains(name, 'Microsoft.Storage/Changefeed')].{Name:name,State:properties.state}"
az feature list -o table --query "[?contains(name, 'Microsoft.Storage/Versioning')].{Name:name,State:properties.state}"
```

---

## <a name="pricing-and-billing"></a>價格和計費

時間點還原的計費取決於處理以執行還原作業的資料量。 處理的資料量取決於還原點與目前時間之間發生的變更數目。 例如，假設變更在儲存體帳戶中封鎖 blob 資料的變動率相當頻繁，則在1天內返回的還原作業會在10天內回復的還原成本 1/10。

若要預估還原作業的成本，請檢查變更摘要記錄檔，以估計還原期間修改過的資料量。 例如，如果變更摘要的保留期限為30天，而變更摘要的大小為 10 MB，則還原至較早10天的時間點，將會花費大約為該區域中 LRS 帳戶所列出的一到一第三個價格。 還原至較早27天的點，將會花費大約九-十分之一的價格。

如需時間點還原價格的詳細資訊，請參閱 [區塊 blob 定價](https://azure.microsoft.com/pricing/details/storage/blobs/)。

## <a name="ask-questions-or-provide-feedback"></a>提出問題或提供意見反應

若要詢問有關時間點還原預覽的問題，或是要提供意見反應，請洽詢 Microsoft pitrdiscussion@microsoft.com 。

## <a name="next-steps"></a>後續步驟

- [啟用和管理區塊 blob 的時間點還原 (預覽) ](point-in-time-restore-manage.md)
- [Azure Blob 儲存體中的變更摘要支援 (預覽)](storage-blob-change-feed.md)
- [啟用 Blob 的虛刪除](soft-delete-enable.md)
- [啟用和管理 Blob 版本設定](versioning-enable.md)

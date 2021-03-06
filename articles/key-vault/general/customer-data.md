---
title: Azure Key Vault 客戶資料功能 - Azure Key Vault | Microsoft Docs
description: 瞭解客戶資料，Azure Key Vault 在建立或更新保存庫、金鑰、秘密、憑證及受控儲存體帳戶期間接收。
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.topic: reference
ms.date: 01/07/2019
ms.author: mbaldwin
ms.openlocfilehash: e7cfc707aa4bccdcd72e45efa3693ebd8f88a211
ms.sourcegitcommit: 9ce0350a74a3d32f4a9459b414616ca1401b415a
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/13/2020
ms.locfileid: "88189925"
---
# <a name="azure-key-vault-customer-data-features"></a>Azure Key Vault 客戶資料功能

Azure Key Vault 會在建立或更新保存庫、金鑰、祕密、憑證及受控儲存體帳戶期間，接收客戶資料。 使用者可以直接在 Azure 入口網站，或透過 REST API 看見這些客戶資料。 客戶資料可以透過更新或刪除包含資料的物件，進行編輯或刪除。

使用者或應用程式存取 Key Vault 時會產生系統存取記錄。 詳細存取記錄可以使用 Azure Insights 提供給客戶。

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-intro-sentence.md)]

## <a name="identifying-customer-data"></a>識別客戶資料

下列資訊會識別 Azure Key Vault 內的客戶資料：

- Azure Key Vault 的存取原則包含代表使用者、群組或應用程式的物件識別碼
- 憑證主體可能包含電子郵件地址或其他使用者或組織識別碼
- 憑證連絡人可能包含使用者電子郵件地址、名稱或電話號碼
- 憑證簽發者可能包含電子郵件地址、名稱、電話號碼、帳戶認證以及組織詳細資料
- 任意標記可以套用至 Azure Key Vault 中的物件。 這些物件包含保存庫、金鑰、祕密、憑證和儲存體帳戶。 使用的標記可能包含個人資料
- Azure Key Vault 存取記錄包含每個 REST API 呼叫的物件識別碼、[UPN](../../active-directory/hybrid/plan-connect-userprincipalname.md) 及 IP 位址
- Azure Key Vault 診斷記錄可能包含 REST API 呼叫的物件識別碼和 IP 位址

## <a name="deleting-customer-data"></a>刪除客戶資料

用來建立保存庫、金鑰、祕密、憑證及受控儲存體帳戶的相同 REST API、入口網站體驗和 SDK，也可以用來更新和刪除這些物件。

虛刪除可讓您在刪除後的90天復原已刪除的資料。 使用虛刪除時，可以藉由執行清除作業，在90天的保留期限到期之前永久刪除資料。 如果已將保存庫或訂用帳戶設定為封鎖清除作業，那麼在已排程的保留期間結束之前，無法永久刪除資料。

## <a name="exporting-customer-data"></a>匯出客戶資料

用來建立保存庫、金鑰、祕密、憑證及受控儲存體帳戶的相同 REST API、入口網站體驗和 SDK，也可以讓您檢視及匯出這些物件。

Azure Key Vault 存取記錄是選擇性的功能，您可以針對每個 REST API 呼叫開啟此功能以產生記錄。 這些記錄會轉移到您的訂用帳戶中的儲存體帳戶，您可以在其中套用符合組織需求的保留原則。

包含個人資料的 Azure Key Vault 診斷記錄，可以藉由在使用者隱私權入口網站中進行匯出要求來擷取。 這項要求必須由租用戶管理員提出。

## <a name="next-steps"></a>後續步驟

- [Azure Key Vault 記錄](logging.md)) 

- [Azure Key Vault 虛刪除概觀](soft-delete-cli.md)

- [Azure Key Vault 金鑰作業](https://docs.microsoft.com/rest/api/keyvault/key-operations)

- [Azure Key Vault 祕密作業](https://docs.microsoft.com/rest/api/keyvault/secret-operations)

- [Azure Key Vault 憑證和原則](https://docs.microsoft.com/rest/api/keyvault/certificates-and-policies)

- [Azure Key Vault 儲存體帳戶作業](https://docs.microsoft.com/rest/api/keyvault/storage-account-key-operations)

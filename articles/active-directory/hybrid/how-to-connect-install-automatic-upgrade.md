---
title: Azure AD Connect：自動升級 | Microsoft Docs
description: 本主題說明 Azure AD Connect 同步處理中內建的自動升級功能。
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: 6b395e8f-fa3c-4e55-be54-392dd303c472
ms.service: active-directory
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 06/09/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: dcc6de1ce50e86f177023a0a66c436633c8d502c
ms.sourcegitcommit: 269da970ef8d6fab1e0a5c1a781e4e550ffd2c55
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/10/2020
ms.locfileid: "88053281"
---
# <a name="azure-ad-connect-automatic-upgrade"></a>Azure AD Connect：自動升級
此功能已隨組建 [1.1.105.0 (於 2016 年 2 月發行)](reference-connect-version-history.md) 一起推出。  這項功能已在[組建 1.1.561](reference-connect-version-history.md) 中更新，且現在支援先前未支援的其他案例。

## <a name="overview"></a>概觀
使用「自動升級」  功能是可確保您 Azure AD Connect 安裝永遠維持在最新狀態的最簡單方式。 此功能預設為啟用，以供進行快速安裝及 DirSync 升級。 新版本發行時，您的安裝會自動升級。
預設會針對下列情況啟用自動升級：

* 快速設定安裝和 DirSync 升級。
* 使用 SQL Express LocalDB (這是「快速設定」一律使用的)。 使用 SQL Express 的 DirSync 也會使用 LocalDB。
* AD 帳戶是「快速設定」和 DirSync 所建立的預設 MSOL_ 帳戶。
* Metaverse 中的物件少於 100,000 個

您可以使用 PowerShell Cmdlet `Get-ADSyncAutoUpgrade`來檢視目前的自動升級狀態。 其狀態如下：

| State | 註解 |
| --- | --- |
| 啟用 |已啟用自動升級。 |
| 暫止 |只有系統才能設定。 系統**目前沒有**資格再接收自動升級。 |
| 已停用 |已停用自動升級。 |

您可以使用 `Set-ADSyncAutoUpgrade` 在 [已啟用] 與 [已停用] 之間進行變更。 應該只有系統才能設定 [已暫止] 狀態。  在 1.1.750.0 之前，自動升級狀態如果設定為 [暫止]，則 Set-ADSyncAutoUpgrade Cmdlet 會封鎖自動升級。 此功能現在已變更，因此不會封鎖自動升級。

自動升級使用 Azure AD Connect Health 做為升級基礎結構。 為了讓自動升級能夠運作，請確定您已依照 **Office 365 URL 與 IP 位址範圍** 中的記載，在您 Proxy 伺服器中開啟 [Azure AD Connect Health](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2)的 URL。


如果伺服器上正在執行 **Synchronization Service Manager** UI，則會暫止升級，直到 UI 關閉為止。

## <a name="troubleshooting"></a>疑難排解
如果您的 Connect 安裝未如預期般自動升級，請遵循下列步驟來找出可能的錯誤。

首先，不建議您在新版本發行的第一天就自動升級。 由於升級前有刻意設計的隨機性，因此不用擔心您的安裝沒有立即升級。

如果您認為有問題，請先執行 `Get-ADSyncAutoUpgrade` 確保已啟用自動升級。

如果狀態是 [已暫停]，您可以使用 `Get-ADSyncAutoUpgrade -Detail` 來查看原因。  暫止原因可以包含任何字串值，但通常會包含 UpgradeResult 的字串值，也就是 `UpgradeNotSupportedNonLocalDbInstall` 或 `UpgradeAbortedAdSyncExeInUse` 。  可能也會傳回復合值，例如 `UpgradeFailedRollbackSuccess-GetPasswordHashSyncStateFailed` 。

也可以取得不是 UpgradeResult 的結果，例如 ' AADHealthEndpointNotDefined ' 或 ' DirSyncInPlaceUpgradeNonLocalDb '。

然後，確定您已在您的 Proxy 或防火牆中開啟所需的 URL。 自動更新會如 [概觀](#overview)所述使用 Azure AD Connect Health。 如果您使用 Proxy，請確定 Health 已設定為使用 [roxy 伺服器](how-to-connect-health-agent-install.md#configure-azure-ad-connect-health-agents-to-use-http-proxy)。 而且測試對 Azure AD 的 [Health 連線](how-to-connect-health-agent-install.md#test-connectivity-to-azure-ad-connect-health-service) 。

透過對已驗證 Azure AD 的連線，即可查看事件記錄檔。 啟動事件檢視器，並查看 **應用程式** 事件記錄。 新增來源 **Azure AD Connect 升級**的事件記錄篩選以及事件識別碼範圍 **300-399**。  
![自動升級的事件記錄篩選](./media/how-to-connect-install-automatic-upgrade/eventlogfilter.png)  

您現在可以看到與自動升級狀態關聯的事件記錄。  
![自動升級的事件記錄篩選](./media/how-to-connect-install-automatic-upgrade/eventlogresult.png)  

結果碼前面會有包含狀態概觀的前置詞。

| 結果碼前置詞 | 描述 |
| --- | --- |
| Success |安裝已順利升級。 |
| UpgradeAborted |發生暫時狀況導致升級停止。 它將會重試一次，而且預期稍後成功。 |
| UpgradeNotSupported |系統具有封鎖自動升級系統的組態。 它將會重試以查看狀態是否已變更，但預期情況是系統必須手動升級。 |

以下是最常見的訊息清單。 清單不完整，但結果訊息應該清楚顯示問題所在。

| 結果訊息 | 描述 |
| --- | --- |
| **UpgradeAborted** | |
| UpgradeAbortedCouldNotSetUpgradeMarker |無法寫入登錄。 |
| UpgradeAbortedInsufficientDatabasePermissions |內建的系統管理員群組沒有資料庫的權限。 請手動升級至最新版的 Azure AD Connect 來解決此問題。 |
| UpgradeAbortedInsufficientDiskSpace |沒有足夠的光碟空間，以支援升級。 |
| UpgradeAbortedSecurityGroupsNotPresent |找不到且無法解析同步處理引擎使用的所有安全性群組。 |
| UpgradeAbortedServiceCanNotBeStarted |NT 服務 **Microsoft Azure AD Sync** 無法啟動。 |
| UpgradeAbortedServiceCanNotBeStopped |NT 服務 **Microsoft Azure AD Sync** 無法停止。 |
| UpgradeAbortedServiceIsNotRunning |NT 服務 **Microsoft Azure AD Sync** 並未執行。 |
| UpgradeAbortedSyncCycleDisabled |[排程器](how-to-connect-sync-feature-scheduler.md) 中的 SyncCycle 選項已停用。 |
| UpgradeAbortedSyncExeInUse |伺服器上的 [Synchronization Service Manager UI](how-to-connect-sync-service-manager-ui.md) 為開啟。 |
| UpgradeAbortedSyncOrConfigurationInProgress |安裝精靈正在執行或排程器外部已排定同步處理。 |
| **UpgradeNotSupported** | |
| UpgradeNotSupportedCustomizedSyncRules |您已將自己的自訂規則加入組態。 |
| UpgradeNotSupportedInvalidPersistedState |安裝不是快速設定或 DirSync 升級。 |
| UpgradeNotSupportedNonLocalDbInstall |您不是使用 SQL Server Express LocalDB 資料庫。 |
|UpgradeNotSupportedLocalDbSizeExceeded|本機資料庫大小大於或等於 8 GB|
|UpgradeNotSupportedAADHealthUploadDisabled|已從入口網站停用健全狀況資料上傳|

## <a name="next-steps"></a>後續步驟
深入了解 [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)。

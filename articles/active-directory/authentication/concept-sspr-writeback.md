---
title: 內部部署密碼回寫與自助式密碼重設 - Azure Active Directory
description: 了解如何將 Azure Active Directory 中的密碼變更或重設事件回寫至內部部署目錄環境
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 07/14/2020
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.reviewer: rhicock
ms.collection: M365-identity-device-management
ms.openlocfilehash: 3959fc7df78a5c1f255f7551a018eec6b7279eb1
ms.sourcegitcommit: 6fc156ceedd0fbbb2eec1e9f5e3c6d0915f65b8e
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/21/2020
ms.locfileid: "88717433"
---
# <a name="how-does-self-service-password-reset-writeback-work-in-azure-active-directory"></a>自助式密碼重設回寫如何在 Azure Active Directory 中運作？

Azure Active Directory (Azure AD) 自助式密碼重設 (SSPR) 可讓使用者在雲端重設其密碼，但大部分公司也會在其使用者所在處具有內部部署 Active Directory Domain Services (AD DS) 環境。 密碼回寫是透過 [Azure AD Connect](../hybrid/whatis-hybrid-identity.md) 所實現的功能，可將雲端中的密碼變更即時回寫至現有內部部署目錄。 在此設定中，當使用者在雲端使用 SSPR 來變更或重設其密碼時，更新過的密碼也會寫回內部部署 AD DS 環境

> [!IMPORTANT]
> 此概念文章會說明自助密碼重設回寫運作方式的系統管理員。 如果您是已註冊自助式密碼重設的使用者，而且需要取回您的帳戶，請移至 https://aka.ms/sspr 。
>
> 如果您的 IT 小組尚未啟用重設您密碼的功能，請與您的技術服務人員聯繫以取得其他協助。

使用下列混合式身分識別模型的環境可支援密碼回寫：

* [密碼雜湊同步處理](../hybrid/how-to-connect-password-hash-synchronization.md)
* [傳遞驗證](../hybrid/how-to-connect-pta.md)
* [Active Directory 同盟服務](../hybrid/how-to-connect-fed-management.md) \(英文\)

密碼回寫提供下列功能：

* **強制執行內部部署 Active Directory Domain Services (AD DS) 密碼原則**：當使用者重設其密碼時，系統會進行檢查以確保此動作符合內部部署 AD DS 原則，然後才將其認可至該目錄。 這項檢閱包括檢查歷程記錄、複雜度、有效期、密碼篩選，以及在 AD DS 中定義的任何其他密碼限制。
* **零延遲的意見反應**： 密碼回寫是一項同步作業。 如果使用者的密碼不符合原則，或因為任何原因而無法重設或變更，則其會立即收到通知。
* **支援從存取面板和 Office 365 變更密碼**：當已同盟或已同步處理密碼雜湊的使用者變更其已過期或尚未過期密碼時，系統會將這些密碼回寫到 AD DS。
* **支援在管理員從 Azure 入口網站重設密碼時將密碼回寫**：當管理員在 [Azure 入口網站](https://portal.azure.com)中重設使用者密碼時，如果該使用者為已同盟或已同步處理密碼雜湊的使用者，系統就會將密碼回寫至內部部署環境。 Office 管理入口網站目前不支援此功能。
* **不需要任何輸入防火牆規則**：密碼回寫會使用「Azure 服務匯流排」轉送作為基礎通訊通道。 所有通訊都會透過連接埠 443 來輸出。

> [!NOTE]
> 內部部署 AD 中的受保護群組所包含的系統管理員帳戶無法用於密碼回寫。 系統管理員可在雲端變更其密碼，但不能使用密碼重設來重設忘記的密碼。 如需受保護群組的詳細資訊，請參閱 [AD DS 中受保護的帳戶和群組](/windows-server/identity/ad-ds/plan/security-best-practices/appendix-c--protected-accounts-and-groups-in-active-directory)。

若要開始使用 SSPR 回寫，請完成下列教學課程：

> [!div class="nextstepaction"]
> [教學課程：啟用自助式密碼重設 (SSPR) 回寫](./tutorial-enable-sspr-writeback.md)

## <a name="how-password-writeback-works"></a>密碼回寫的運作方式

當已同盟的或已同步處理密碼雜湊的使用者嘗試在雲端重設或變更其密碼時，會發生下列動作：

1. 系統會執行檢查，以了解使用者擁有哪一種密碼。 如果密碼是在內部部署環境中進行管理的：
   * 系統會執行檢查以了解回寫服務是否已啟動並在執行中。 如果是，使用者可以繼續進行。
   * 如果回寫服務關閉，系統會告知使用者其無法立即重設其密碼。
1. 接著，使用者會通過適當的驗證閘道，然後到達 [重設密碼] 畫面。
1. 使用者選取新的密碼，並加以確認。
1. 當使用者選取 [送出] 時，系統會以在回寫設定程序期間所建立的對稱金鑰加密純文字密碼。
1. 經過加密的密碼會放入承載中，透過 HTTPS 通道傳送到租用戶特定的服務匯流排轉送 (會在回寫設定程序期間為您設定此轉送)。 此轉送受到只有您的內部部署安裝才會知道的隨機產生密碼所保護。
1. 在訊息抵達服務匯流排之後，密碼重設端點會自動甦醒，並看到有擱置中的重設要求。
1. 服務會接著使用雲端錨點屬性來尋找使用者。 若要讓此查閱成功，則必須符合下列條件：

   * 使用者物件必須存在於 AD DS 連接器空間中。
   * 使用者物件必須連結至對應的 Metaverse (MV) 物件。
   * 使用者物件必須連結至對應的 Azure AD 連接器物件。
   * 從 AD DS 連接器物件到 MV 的連結必須在連結上有同步處理規則 `Microsoft.InfromADUserAccountEnabled.xxx` 。

   當呼叫來自雲端時，同步處理引擎會使用 **cloudAnchor** 屬性來查閱 Azure AD 連接器空間物件。 接著，它會接著連結回到 MV 物件，然後在連結之後，再回到 AD DS 物件。 因為同一個使用者可以有多個 AD DS 物件 (多樹系) ，所以同步處理引擎會依賴此 `Microsoft.InfromADUserAccountEnabled.xxx` 連結來挑選正確的物件。

1. 找到使用者帳戶之後，會嘗試在適當的 AD DS 樹系中直接重設密碼。
1. 如果密碼設定作業成功，系統會告訴使用者其密碼已變更。

   > [!NOTE]
   > 如果將使用者的密碼雜湊同步處理至 Azure AD 時使用密碼雜湊同步處理，則內部部署密碼原則就有可能比雲端密碼原則弱。 在此情況下，系統會強制執行內部部署原則。 此原則可確保不論您是使用密碼雜湊同步處理還是同盟來提供單一登入，都會在雲端強制執行內部部署原則。

1. 如果密碼設定作業失敗，系統會顯示錯誤以提示使用者再試一次。 作業可能會失敗，原因如下：
    * 服務已停止運作。
    * 他們所選的密碼不符合組織原則。
    * 在本機 AD DS 環境中找不到使用者。

   錯誤訊息會對使用者提供指引，讓他們可以嘗試自行解決而不需要系統管理員介入。

## <a name="password-writeback-security"></a>密碼回寫安全性

密碼回寫是一項極為安全的服務。 為了確保資訊會受到保護，系統會啟用四層式安全性模型，如下所示：

* **租用戶特定的服務匯流排轉送**
   * 當您設定服務時，便會設定一個以隨機產生強式密碼保護的租用戶特定服務匯流排轉送，且 Microsoft 永遠無法得知該密碼。
* **受到鎖定的密碼編譯強式密碼加密金鑰**
   * 建立服務匯流排轉送之後，即會建立強式對稱金鑰，以在透過線路傳送密碼時予以加密。 此金鑰只會存在於您公司的雲端密碼存放區中，並受到嚴密鎖定和稽核，就像目錄中的任何其他密碼一樣。
* **業界標準傳輸層安全性 (TLS)**
   1. 當雲端上發生密碼重設或變更作業時，系統便會以公開金鑰將純文字密碼加密。
   1. 加密密碼會放入 HTTPS 訊息中，使用 Microsoft TLS/SSL 憑證透過加密通道傳送到服務匯流排轉送。
   1. 在訊息抵達服務匯流排之後，您的內部部署代理程式會喚醒服務匯流排，並使用先前產生的強式密碼向其進行驗證。
   1. 內部部署代理程式會拾取加密訊息，然後使用私密金鑰進行解密。
   1. 內部部署代理程式會嘗試透過 AD DS SetPassword API 來設定密碼。 此步驟可讓您強制執行 AD DS 的內部部署密碼原則 (例如雲端中) 的複雜性、年齡、歷程記錄和篩選。
* **訊息到期原則**
   * 如果訊息因內部部署服務已停止運作而停留在服務匯流排中，在數分鐘之後，它就會逾時而被移除。 讓訊息逾時並予以移除可更進一步提升安全性。

### <a name="password-writeback-encryption-details"></a>密碼回寫的加密詳細資料

在使用者提交密碼重設之後，重設要求會先經過數個加密步驟，然後才抵達您的內部部署環境。 這些加密步驟可確保提供最高的服務可靠性和安全性。 這些步驟的說明如下：

1. **採用 2048 位元 RSA 金鑰的密碼加密**：在使用者提交要寫回到內部部署環境的密碼之後，所提交的密碼本身會以 2048 位元 RSA 金鑰加密。
1. **採用 AES-GCM 的套件層級加密**：整個套件 (密碼 + 必要的中繼資料) 會以 AES-GCM 加密。 此加密可防止任何人直接存取基礎服務匯流排通道，使其無法對內容進行觀看或篡改。
1. **所有通訊都會透過 TLS/SSL 進行**：所有與服務匯流排的通訊都發生在 SSL/TLS 通道中。 這個加密可保護內容，免於遭到未經授權的第三方存取。
1. **每六個月自動變換金鑰**：所有金鑰會每6個月變換一次，或每次停用密碼回寫，然後在 Azure AD Connect 上重新啟用，以確保最高的服務安全性和安全性。

### <a name="password-writeback-bandwidth-usage"></a>密碼回寫頻寬使用量

密碼回寫是一種低頻寬服務，只有在下列情況下，才會將要求傳回給內部部署代理程式︰

* 透過 Azure AD Connect 啟用或停用此功能時會傳送兩則訊息。
* 每 5 分鐘傳送一則訊息作為服務活動訊號 (只要服務正在執行)。
* 每次提交新密碼時會傳送兩則訊息：
   * 第一則訊息是一個執行作業的要求。
   * 第二則訊息包含該作業的結果，並且會在下列情況下傳送︰
      * 每次在使用者自助式密碼重設期間提交新密碼時。
      * 每次在使用者密碼變更作業期間提交新密碼時。
      * 每次在系統管理員所起始的使用者密碼重設 (僅限從 Azure 系統管理員入口網站) 期間提交新密碼時。

#### <a name="message-size-and-bandwidth-considerations"></a>訊息大小和頻寬考量

上述每則訊息的大小通常會小於 1 KB。 即使在負載極大的情況下，密碼回寫服務本身每秒也只會耗用幾千位元的頻寬。 因為每則訊息都以即時方式傳送 (僅限密碼更新作業要求時)，且因為訊息大小是如此的小，所以回寫功能的頻寬使用量會因太小，以致沒有可測得的影響。

## <a name="supported-writeback-operations"></a>支援的回寫作業

下列所有情況都會回寫密碼︰

* **支援的使用者作業**
   * 任何終端使用者自助式自願變更密碼作業。
   * 任何終端使用者自助式強制變更密碼作業 (例如密碼到期)。
   * 任何源自[密碼重設入口網站](https://passwordreset.microsoftonline.com)的終端使用者自助式密碼重設。

* **支援的系統管理員作業**
   * 任何系統管理員自助式自願變更密碼作業。
   * 任何系統管理員自助式強制變更密碼作業 (例如密碼到期)。
   * 任何源自[密碼重設入口網站](https://passwordreset.microsoftonline.com)的系統管理員自助式密碼重設。
   * 系統管理員從 [Azure 入口網站](https://portal.azure.com)起始的任何終端使用者密碼重設。
   * 系統管理員從 [Microsoft Graph API Beta](/graph/api/passwordauthenticationmethod-resetpassword?tabs=http&view=graph-rest-beta) 起始的任何終端使用者密碼重設。

## <a name="unsupported-writeback-operations"></a>不支援的回寫作業

下列任何情況均不會回寫密碼︰

* **不支援的使用者作業**
   * 任何由終端使用者使用 PowerShell 第 1 版、第 2 版或 Microsoft Graph API 來進行的自有密碼重設。
* **不支援的系統管理員作業**
   * 任何由系統管理員從 PowerShell 第 1 版、第 2 版或 Microsoft Graph API 起始的終端使用者密碼重設 (支援 [Microsoft Graph API 搶鮮版 (Beta)](/graph/api/passwordauthenticationmethod-resetpassword?tabs=http&view=graph-rest-beta))。
   * 系統管理員從 [Microsoft 365 系統管理中心](https://admin.microsoft.com)起始的任何終端使用者密碼重設。
   * 任何系統管理員都無法使用密碼重設工具來重設自己的密碼以進行密碼回寫。

> [!WARNING]
> 使用如 Active Directory 使用者和電腦或 Active Directory 系統管理中心的內部部署 AD DS 系統管理工具中 [使用者必須在下次登入時變更密碼] 核取方塊，為 Azure AD Connect 支援的一項預覽功能。 如需詳細資訊，請參閱[使用 Azure AD Connect 同步實作密碼雜湊同步處理](../hybrid/how-to-connect-password-hash-synchronization.md)。

## <a name="next-steps"></a>後續步驟

若要開始使用 SSPR 回寫，請完成下列教學課程：

> [!div class="nextstepaction"]
> [教學課程：啟用自助式密碼重設 (SSPR) 回寫](./tutorial-enable-sspr-writeback.md)
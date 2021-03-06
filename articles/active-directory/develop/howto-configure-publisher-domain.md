---
title: 設定應用程式的發行者網域 |Azure
titleSuffix: Microsoft identity platform
description: 瞭解如何設定應用程式的發行者網域，讓使用者知道其資訊的傳送位置。
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: how-to
ms.workload: identity
ms.date: 07/23/2020
ms.author: ryanwi
ms.reviewer: lenalepa, sureshja, zachowd
ms.custom: aaddev
ms.openlocfilehash: d62328534157a1e2d9e53830be9d6b87e08c9e40
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/24/2020
ms.locfileid: "87129265"
---
# <a name="how-to-configure-an-applications-publisher-domain"></a>如何：設定應用程式的發行者網域

應用程式的發行者網域會在[應用程式的同意提示](application-consent-experience.md)中顯示給使用者，讓使用者知道其資訊的傳送位置。 在 2019 5 月21日之後註冊的多租使用者應用程式，沒有發行者網域會顯示為未**驗證**。 多租使用者應用程式是支援單一組織目錄以外之帳戶的應用程式;例如，支援所有 Azure AD 帳戶，或支援所有 Azure AD 帳戶和個人 Microsoft 帳戶。

## <a name="new-applications"></a>新的應用程式

當您註冊新的應用程式時，您應用程式的發行者網域可能會設定為預設值。 此值取決於應用程式的註冊位置，特別是應用程式是否登錄在租使用者中，以及租使用者是否有租使用者驗證的網域。

如果有租使用者驗證的網域，應用程式的發行者網域會預設為租使用者的主要已驗證網域。 如果沒有租使用者驗證的網域（也就是未在租使用者中註冊應用程式的情況），應用程式的發行者網域將會設定為 null。

下表摘要說明「發行者」網域值的預設行為。  

| 租使用者驗證的網域 | 發行者網域的預設值 |
|-------------------------|----------------------------|
| null | null |
| *. onmicrosoft.com | *. onmicrosoft.com |
| -*. onmicrosoft.com<br/>-domain1.com<br/>-domain2.com （主要） | domain2.com |

如果未設定多租使用者應用程式的發行者網域，或其設定為 onmicrosoft.com 結尾的網域，則應用程式的同意提示會顯示 [未**驗證**] 以取代發行者網域。

## <a name="grandfathered-applications"></a>Grandfathered 應用程式

如果您的應用程式是在 2019 5 月21日之前註冊，則如果您尚未設定發行者網域，則應用程式的同意提示不會顯示 [未**驗證**]。 我們建議您設定發行者網域值，讓使用者可以在應用程式的同意提示中看到這則資訊。

## <a name="configure-publisher-domain-using-the-azure-portal"></a>使用 Azure 入口網站設定發行者網域

若要設定應用程式的發行者網域，請遵循下列步驟。

1. 使用公司或學校帳戶或個人的 Microsoft 帳戶登入 [Azure 入口網站](https://portal.azure.com)。

1. 如果您的帳戶出現在多個 Azure AD 租用戶中：
   1. 從頁面右上角的功能表中選取您的設定檔，然後**切換目錄**。
   1. 將工作階段變更為要在其中建立應用程式的 Azure AD 租用戶。

1. 流覽至[Azure Active Directory > 應用程式註冊](https://go.microsoft.com/fwlink/?linkid=2083908)，尋找並選取您要設定的應用程式。

   選取應用程式後，您會看到應用程式的 **[總覽**] 頁面。

1. 從應用程式的 **[總覽**] 頁面中，選取 [**商標**] 區段。

1. 尋找 [**發行者網域]** 欄位，然後選取下列其中一個選項：

   - 如果您尚未設定網域，請選取 [**設定網域**]。
   - 如果已設定網域，請選取 [**更新網域**]。

如果您的應用程式已在租使用者中註冊，您會看到兩個索引標籤可供您選取：**選取已驗證的網域**並**驗證新的網域**。

如果您的應用程式未在租使用者中註冊，您只會看到為您的應用程式驗證新網域的選項。

### <a name="to-verify-a-new-domain-for-your-app"></a>驗證應用程式的新網域

1. 建立名為的檔案 `microsoft-identity-association.json` ，並貼上下列 JSON 程式碼片段。

   ```json
   {
      "associatedApplications": [
         {
            "applicationId": "{YOUR-APP-ID-HERE}"
         },
         {
            "applicationId": "{YOUR-OTHER-APP-ID-HERE}"
         }
      ]
    }
   ```

1. 將預留位置 *{您的應用程式識別碼-此處}* 取代為對應至您應用程式的應用程式（用戶端）識別碼。

1. 將檔案裝載于： `https://{YOUR-DOMAIN-HERE}.com/.well-known/microsoft-identity-association.json` 。 取代預留位置 *{您的網域-此處}* ，以符合已驗證的網域。

1. 按一下 [**驗證並儲存網域**] 按鈕。

### <a name="to-select-a-verified-domain"></a>選取已驗證的網域

- 如果您的租使用者已驗證網域，請從 [**選取已驗證的網域**] 下拉式清單中選取其中一個網域。

>[!Note]
> 應傳回的預期 ' Content-type ' 標頭為 `application/json` 。 如果您使用任何其他專案，則可能會收到錯誤，如下所述`application/json; charset=utf-8` 
> 
>``` "Verification of publisher domain failed. Error getting JSON file from https:///.well-known/microsoft-identity-association. The server returned an unexpected content type header value. " ```
>

## <a name="implications-on-the-app-consent-prompt"></a>應用程式同意提示的含意

設定發行者網域會影響使用者在應用程式同意提示中看到的內容。 若要完全瞭解同意提示的元件，請參閱[瞭解應用程式同意體驗](application-consent-experience.md)。

下表描述在 2019 5 月21日之前建立之應用程式的行為。

![2019 5 月21日之前建立之應用程式的同意提示](./media/howto-configure-publisher-domain/old-app-behavior-table.png)

在2019以後建立的新應用程式的行為，將取決於發行者網域和應用程式的類型。 下表描述使用不同的設定組合時，應該會看到的變更。

![2019 5 月21日之後所建立應用程式的同意提示](./media/howto-configure-publisher-domain/new-app-behavior-table.png)

## <a name="implications-on-redirect-uris"></a>重新導向 Uri 的含意

在指定重新導向 Uri 時，使用任何工作或學校帳戶或個人 Microsoft 帳戶（[多租](single-and-multi-tenant-apps.md)使用者）登入使用者的應用程式會受到少數限制。

### <a name="single-root-domain-restriction"></a>單一根域限制

當多租使用者應用程式的「發行者」網域值設定為 null 時，應用程式會受限於共用單一根域以用於重新導向 Uri。 例如，由於根域 contoso.com 不符合 fabrikam.com，因此不允許下列值組合。

```
"https://contoso.com",
"https://fabrikam.com",
```

### <a name="subdomain-restrictions"></a>子域限制

允許子域，但您必須明確地註冊根域。 例如，當下列 Uri 共用單一根域時，不允許使用此組合。

```
"https://app1.contoso.com",
"https://app2.contoso.com",
```

不過，如果開發人員明確新增根域，則允許此組合。

```
"https://contoso.com",
"https://app1.contoso.com",
"https://app2.contoso.com",
```

### <a name="exceptions"></a>例外狀況

下列情況不受限於單一根域限制：

- 單一租使用者應用程式，或以單一目錄中的帳戶為目標的應用程式
- 使用 localhost 做為重新導向 Uri
- 以自訂配置（非 HTTP 或 HTTPS）重新導向 Uri

## <a name="configure-publisher-domain-programmatically"></a>以程式設計方式設定發行者網域

目前，沒有任何 REST API 或 PowerShell 支援以程式設計方式設定發行者網域。

---
title: MSAL 驗證流程 | Azure
titleSuffix: Microsoft identity platform
description: 了解 Microsoft 驗證程式庫 (MSAL) 所使用的驗證流程和授權。
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 07/08/2020
ms.author: marsma
ms.reviewer: saeeda
ms.openlocfilehash: 0dbd7eb9203f31b580f586a8a7d1fa216533628c
ms.sourcegitcommit: 1e6c13dc1917f85983772812a3c62c265150d1e7
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/09/2020
ms.locfileid: "86170508"
---
# <a name="authentication-flows"></a>驗證流程

Microsoft Authentication Library (MSAL) 支援數個驗證流程，以用於不同的應用程式案例。

| Flow | 描述 | 使用於 |
|--|--|--|
| [授權碼](#authorization-code) | 使用於裝置上所安裝的應用程式中，以存取受保護的資源，例如 Web API。 可讓您將登入和 API 存取權新增至您的行動和桌面應用程式。 | [傳統型應用程式](scenario-desktop-overview.md)、[行動應用程式](scenario-mobile-overview.md)、[Web 應用程式](scenario-web-app-call-api-overview.md) |
| [用戶端認證](#client-credentials) | 可讓您使用應用程式的身分識別來存取 Web 裝載資源。 通常用於必須在背景中執行 (不需與使用者直接互動) 的伺服器對伺服器互動。 | [精靈應用程式](scenario-daemon-overview.md) |
| [裝置程式碼](#device-code) | 可讓使用者登入輸入受限的裝置，例如智慧型電視、IoT 裝置或印表機。 | [傳統型/行動應用程式](scenario-desktop-acquire-token.md#command-line-tool-without-a-web-browser) |
| [隱含授與](#implicit-grant) | 允許應用程式取得權杖，而不需執行後端伺服器認證交換。 讓應用程式能夠在用戶端 JavaScript 程式碼內，登入使用者、維護會話，以及取得其他 web Api 的權杖。 | [單頁應用程式 (SPA)](scenario-spa-overview.md) |
| [代理者](#on-behalf-of) | 應用程式會叫用服務或 Web API，而這又需要呼叫另一個服務或 Web API。 其概念是透過要求鏈傳播委派的使用者身分識別和權限。 | [Web API](scenario-web-api-call-api-overview.md) |
| [使用者名稱/密碼](#usernamepassword) | 可讓應用程式直接處理其密碼來登入使用者。 不建議使用此流程。 | [傳統型/行動應用程式](scenario-desktop-acquire-token.md#username-and-password) |
| [整合式 Windows 驗證](#integrated-windows-authentication) | 允許已加入網域或 Azure Active Directory (Azure AD) 電腦上的應用程式以無訊息方式取得權杖 (使用者不需要進行任何 UI 互動)。 | [傳統型/行動應用程式](scenario-desktop-acquire-token.md#integrated-windows-authentication) |

## <a name="how-each-flow-emits-tokens-and-codes"></a>每個流程如何發出權杖和授權碼

視用戶端應用程式的建立方式而定，它可以使用 Microsoft 身分識別平臺所支援的一或多個驗證流程。 這些流程可能會產生數種類型的權杖和授權碼，而且需要不同的權杖才能使其運作。

| Flow                                                                               | 需要            | id_token | 存取權杖 | 重新整理權杖 | 授權碼 |
|------------------------------------------------------------------------------------|:-------------------:|:--------:|:------------:|:-------------:|:------------------:|
| [授權碼流程](v2-oauth2-auth-code-flow.md)                             |                     | x        | x            | x             | x                  |
| [用戶端認證](v2-oauth2-client-creds-grant-flow.md)                         |                     |          | x (僅限應用程式) |               |                    |
| [裝置碼流程](v2-oauth2-device-code.md)                                       |                     | x        | x            | x             |                    |
| [隱含流程](v2-oauth2-implicit-grant-flow.md)                                  |                     | x        | x            |               |                    |
| [代理者流程](v2-oauth2-on-behalf-of-flow.md)                                | 存取權杖        | x        | x            | x             |                    |
| 使用者[名稱/密碼](v2-oauth-ropc.md) (ROPC)                                        | 使用者名稱 & 密碼 | x        | x            | x             |                    |
| [混合式 OIDC 流程](v2-protocols-oidc.md#protocol-diagram-access-token-acquisition) |                     | x        |              |               | x                  |
| [重新整理權杖兌換](v2-oauth2-auth-code-flow.md#refresh-the-access-token)   | 重新整理權杖       | x        | x            | x             |                    |

### <a name="interactive-and-non-interactive-authentication"></a>互動式和非互動式驗證

其中有幾個流程同時支援互動式和非互動式權杖取得。

  - [**互動式**] 表示系統會提示使用者輸入。 例如，提示使用者登入、 (MFA) 執行多重要素驗證，或授與其他資源的同意。
  - **非互動式**或*無*訊息驗證會嘗試以登入伺服器*無法*提示使用者提供其他資訊的方式來取得權杖。

以 MSAL 為基礎的應用程式應該先嘗試以*無*訊息方式取得權杖，然後只在非互動式方法失敗時，才互動。 如需此模式的詳細資訊，請參閱[使用 Microsoft 驗證程式庫取得和快取權杖 (MSAL) ](msal-acquire-cache-tokens.md)。

## <a name="authorization-code"></a>授權碼

[OAuth 2 授權碼授](v2-oauth2-auth-code-flow.md)與可以在安裝于裝置上的應用程式中使用，以存取受保護的資源，例如 web api。 這可讓您將登入和 API 存取權新增至您的行動和傳統型應用程式。

當使用者登入 Web 應用程式 (網站) 時，Web 應用程式會收到授權碼。 兌換授權碼可取得權杖來呼叫 Web API。

![授權碼流程的圖表](media/msal-authentication-flows/authorization-code.png)

在上圖中，應用程式：

1. 要求授權碼，其可兌換為存取權杖。
2. 使用存取權杖來呼叫 Web API。

### <a name="considerations"></a>考量

- 您只能使用授權碼一次來兌換權杖。 請勿嘗試使用相同的授權碼多次取得權杖，因為通訊協定標準規格已明確禁止。 如果您將程式碼兌換數次，不論是刻意或因為您不知道架構也會為您執行，您會收到下列錯誤：

    `AADSTS70002: Error validating credentials. AADSTS54005: OAuth2 Authorization code was already redeemed, please retry with a new valid code or use an existing refresh token.`

## <a name="client-credentials"></a>用戶端認證

[OAuth 2 用戶端認證流程](v2-oauth2-client-creds-grant-flow.md)可讓您使用應用程式的身分識別來存取 web 主控的資源。 這類型的授與通常用於必須在背景中執行 (不需與使用者直接互動) 的伺服器對伺服器互動。 這些類型的應用程式通常稱為精靈或服務帳戶。

用戶端認證授與流程可允許 Web 服務 (機密用戶端) 在呼叫另一個 Web 服務時，使用自己的認證來進行驗證，而不是模擬使用者。 在此案例中，用戶端通常是中介層 Web 服務、精靈服務或網站。 對於較高層級的保證，Microsoft 身分識別平台也可讓呼叫服務使用憑證 (而非共用密碼) 做為認證。

> [!NOTE]
> 機密用戶端流程無法在 UWP、Xamarin 和 Xamarin 等行動裝置平臺上使用，因為它們僅支援公用用戶端應用程式。 公用用戶端應用程式不知道如何向識別提供者證明應用程式的身分識別。 您可以藉由部署憑證，在 web 應用程式或 Web API 後端上達成安全連線。

### <a name="application-secrets"></a>應用程式祕密

![具有密碼的機密用戶端圖表](media/msal-authentication-flows/confidential-client-password.png)

在上圖中，應用程式：

1. 使用應用程式秘密或密碼認證來取得權杖。
2. 使用權杖來提出資源的要求。

### <a name="certificates"></a>憑證

![具有憑證的機密用戶端圖表](media/msal-authentication-flows/confidential-client-certificate.png)

在上圖中，應用程式：

1. 使用憑證認證來取得權杖。
2. 使用權杖來提出資源的要求。

這些用戶端認證必須：

- 已向 Azure AD 註冊。
- 在您的程式碼中建立機密用戶端應用程式物件時傳入。

## <a name="device-code"></a>裝置碼

[OAuth 2 裝置程式碼流程](v2-oauth2-device-code.md)可讓使用者登入受輸入限制的裝置，例如智慧型電視、IoT 裝置和印表機。 使用 Azure AD 的互動式驗證需要網頁瀏覽器。 當裝置或作業系統未提供網頁瀏覽器時，裝置程式碼流程可讓使用者使用其他裝置（例如電腦或行動電話）以互動方式登入。

藉由使用裝置程式碼流程，應用程式會透過專為這些裝置和作業系統設計的雙步驟流程來取得權杖。 這類應用程式的範例包括在 IoT 裝置上執行的 (CLI) 工具的命令列介面。

![裝置碼流程的圖表](media/msal-authentication-flows/device-code.png)

在上圖中︰

1. 每當需要使用者驗證時，應用程式會提供代碼，並要求使用者使用另一部裝置（例如網際網路連線的 smartphone）來流覽 URL (例如 `https://microsoft.com/devicelogin`) 。 然後，系統會提示使用者輸入程式碼，並繼續進行一般驗證體驗，包括同意提示和[多重要素驗證](../authentication/concept-mfa-howitworks.md)（如有需要）。
1. 成功驗證之後，命令列應用程式會透過後端通道接收所需的權杖，並使用這些權杖來執行所需的 Web API 呼叫。

### <a name="constraints"></a>條件約束

- 裝置程式碼流程僅適用于公用用戶端應用程式。
- 在建構公用用戶端應用程式時傳入的授權單位，必須是下列其中一項：
  - 租使用者，格式為， `https://login.microsoftonline.com/{tenant}/,` 其中 `{tenant}` 是代表租使用者識別碼的 GUID，或與租使用者相關聯的功能變數名稱。
  - 適用于表單中的公司和學校帳戶 `https://login.microsoftonline.com/organizations/` 。

## <a name="implicit-grant"></a>隱含授與

[OAuth 2 隱含授](v2-oauth2-implicit-grant-flow.md)與流程可讓應用程式從 Microsoft 身分識別平臺取得權杖，而不需要執行後端伺服器認證交換。 此流程可讓應用程式在用戶端 JavaScript 程式碼內，登入使用者、維護會話，以及取得其他 web Api 的權杖。

![隱含授與流程的圖表](media/msal-authentication-flows/implicit-grant.svg)

許多新式 web 應用程式都是以用戶端、單一頁面應用程式的形式建立 (SPA) 以 JavaScript 或 SPA 架構（例如，角度、Vue.js 和 React.js）撰寫。 這些應用程式會在網頁瀏覽器中執行，且具有與傳統伺服器端 Web 應用程式不同的驗證特性。 Microsoft 身分識別平台可讓單頁應用程式登入使用者，並使用隱含授與流程來取得權杖以存取後端服務或 Web API。 隱含流程可讓應用程式取得識別碼權杖以代表經過驗證的使用者，也可以存取呼叫受保護的 API 所需的權杖。

此驗證流程不包括使用跨平臺 JavaScript 架構（例如 Electron 或回應原生）的應用程式案例，因為它們需要進一步的功能來與原生平臺互動。

透過隱含流程模式發行的權杖有**長度限制**，因為它們會透過 URL 傳回至瀏覽器 (其中 `response_mode` 是 `query` 或 `fragment`) 。 有些瀏覽器會限制瀏覽器列中的 URL 長度，並在太長時失敗。 因此，這些隱含流程權杖不會包含 `groups` 或 `wids` 宣告。

## <a name="on-behalf-of"></a>代理者

當應用程式叫用服務或 Web API 而又需要呼叫另一個服務或 Web API 時，會使用[OAuth 2 代理者驗證流程](v2-oauth2-on-behalf-of-flow.md)流程。 其概念是透過要求鏈傳播委派的使用者身分識別和權限。 若要讓仲介層服務向下游服務提出已驗證的要求，它需要*代表*使用者保護來自 Microsoft 身分識別平臺的存取權杖。

![代理者流程的圖表](media/msal-authentication-flows/on-behalf-of.png)

在上圖中︰

1. 應用程式會取得 Web API 的存取權杖。
2. 用戶端 (Web、傳統型、行動或單頁應用程式) 會呼叫受保護的 Web API，並在 HTTP 要求的驗證標頭中新增存取權杖作為持有人權杖。 Web API 會驗證使用者。
3. 當用戶端呼叫 Web API 時，Web API 會代表使用者要求另一個權杖。
4. 受保護的 Web API 會使用此權杖，代表使用者呼叫下游 Web API。 Web API 稍後也可以要求其他下游 API (但仍代表相同的使用者) 的權杖。

## <a name="usernamepassword"></a>使用者名稱/密碼

 (ROPC) 授與的[OAuth 2 資源擁有者密碼認證](v2-oauth-ropc.md)，可讓應用程式藉由直接處理其密碼來登入使用者。 在傳統型應用程式中，您可使用使用者名稱/密碼流程，以無訊息方式取得權杖。 使用此應用程式時不需要 UI。

![使用者名稱/密碼流程的圖表](media/msal-authentication-flows/username-password.png)

在上圖中，應用程式：

1. 藉由將使用者名稱和密碼傳送給識別提供者來取得權杖。
2. 使用權杖來呼叫 Web API。

> [!WARNING]
> 不建議使用此流程。 它需要高程度的信任和認證暴露。 只有在無法使用更安全的流程時，您*才*應該使用此流程。 如需詳細資訊，請參閱[持續增加的密碼問題有何解決方案？](https://news.microsoft.com/features/whats-solution-growing-problem-passwords-says-microsoft/)。

在已加入網域的 Windows 電腦上以無訊息方式取得權杖的慣用流程是[整合式 Windows 驗證](#integrated-windows-authentication)。 在其他情況下，請使用[裝置程式碼流程](#device-code)。

雖然在某些案例（例如 DevOps）中，使用者名稱/密碼流程可能很有用，但如果您想要在提供自己 UI 的互動式案例中使用使用者名稱/密碼，請避免此情況。

藉由使用使用者名稱/密碼：

- 需要執行多重要素驗證的使用者將無法登入，因為沒有互動。
- 使用者將無法執行單一登入。

### <a name="constraints"></a>條件約束

除了[整合式 Windows 驗證條件約束](#integrated-windows-authentication)之外，也適用下列條件約束：

- 使用者名稱/密碼流程與條件式存取和多重要素驗證不相容。 因此，如果您的應用程式在租用戶系統管理員需要多重要素驗證的 Azure AD 租用戶中執行，您就無法使用此流程。 許多組織都會這麼做。
- ROPC 僅適用于工作和學校帳戶。 您無法將 ROPC 用於 Microsoft 帳戶 (MSA) 。
- 此流程適用於 .NET 傳統型和 .NET Core，但不適用於通用 Windows 平台。
- 在 Azure AD B2C 中，ROPC 流程只適用于本機帳戶。 如需有關 MSAL.NET 和 Azure AD B2C 中 ROPC 的詳細資訊，請參閱搭配[使用 ROPC 與 Azure AD B2C](msal-net-aad-b2c-considerations.md#resource-owner-password-credentials-ropc)。

## <a name="integrated-windows-authentication"></a>整合式 Windows 驗證

MSAL 支援整合式 Windows 驗證 (IWA) 適用于在已加入網域或已加入 Azure AD 的 Windows 電腦上執行的桌面和行動應用程式。 這些應用程式可以使用 IWA，以無訊息方式取得權杖，而不需要使用者進行 UI 互動。

![整合式 Windows 驗證的圖表](media/msal-authentication-flows/integrated-windows-authentication.png)

在上圖中，應用程式：

1. 使用整合式 Windows 驗證取得權杖。
2. 使用權杖來提出資源的要求。

### <a name="constraints"></a>條件約束

整合式 Windows 驗證 (IWA) *僅*支援同盟使用者-在 Active Directory 中建立的使用者和 Azure AD 的支援。 直接在 Azure AD 中建立的使用者若未 Active Directory 支援 (受管理使用者，) 就無法使用此驗證流程。 這項限制並不會影響[使用者名稱/密碼流程](#usernamepassword)。

IWA 適用于 .NET Framework、.NET Core 和通用 Windows 平臺應用程式。

IWA 不會略過多重要素驗證。 若已設定多重要素驗證，如果需要多重要素驗證挑戰，IWA 可能會失敗。 多重要素驗證需要使用者互動。

您無法控制何時要執行識別提供者要求雙重要素驗證。 租用戶系統管理員可以辦到。 通常，當您從不同的國家/地區登入、未透過 VPN 連線到公司網路，有時甚至是透過 VPN 連線時，都需要雙重要素驗證。 Azure AD 會使用 AI 來持續了解是否需要雙重要素驗證。 如果 IWA 失敗，您應該回到[互動式使用者提示](#interactive-and-non-interactive-authentication)。

在建立公用用戶端應用程式時傳入的授權單位必須是下列其中一項：

- 租使用者，格式為， `https://login.microsoftonline.com/{tenant}/,` 其中 `{tenant}` 是代表租使用者識別碼的 GUID，或與租使用者相關聯的功能變數名稱。
- 適用於任何公司和學校帳戶 (`https://login.microsoftonline.com/organizations/`)。 不支援 (MSA) 的 Microsoft 個人帳戶;您不能使用或租使用者 `/common` `/consumers` 。

因為 IWA 是無訊息流程，所以必須符合下列其中一個條件：

- 您應用程式的使用者先前必須已同意才能使用應用程式。
- 租用戶系統管理員先前必須已同意租用戶中的所有使用者，才能使用該應用程式。

這表示下列其中一個條件成立：

- 身為開發人員，您已在自己的 Azure 入口網站中選取 **[授**與]。
- 租使用者系統管理員已在 Azure 入口網站的應用程式註冊的 [ **API 許可權**] 索引標籤中，選取了 **{tenant domain} 的 [授與/撤銷管理員同意**] (請參閱[新增存取 web api](quickstart-configure-app-access-web-apis.md#add-permissions-to-access-web-apis)) 的許可權。
- 您已提供使用者同意應用程式的方式;請參閱[要求個別使用者同意](v2-permissions-and-consent.md#requesting-individual-user-consent)。
- 您已提供一種方式，讓租使用者系統管理員同意應用程式;請參閱系統[管理員同意](v2-permissions-and-consent.md#requesting-consent-for-an-entire-tenant)。

已針對 .NET 傳統型、.NET Core 和 Windows 通用平台應用程式啟用 IWA 流程。 在 .NET Core 上，您必須將使用者名稱提供給 IWA，因為 .NET Core 無法從作業系統取得使用者名稱。

如需同意的詳細資訊，請參閱 [v2.0 權限和同意](v2-permissions-and-consent.md)。

## <a name="next-steps"></a>後續步驟

既然您已複習過 Microsoft 驗證程式庫支援的驗證流程 (MSAL) ，請瞭解如何取得和快取這些流程中使用的權杖：

[使用 Microsoft 驗證程式庫取得和快取權杖 (MSAL) ](msal-acquire-cache-tokens.md)

---
title: 教學課程：Azure Active Directory 單一登入 (SSO) 與 Raketa 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Raketa 之間的單一登入。
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 07/28/2020
ms.author: jeedes
ms.openlocfilehash: f5a01d74d22d6ea010d80ef1ef8ce1a53fbd4422
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2020
ms.locfileid: "88548808"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-raketa"></a>教學課程：Azure Active Directory 單一登入 (SSO) 與 Raketa 整合

在本教學課程中，您會了解如何將 Raketa 與 Azure Active Directory (Azure AD) 進行整合。 在整合 Raketa 與 Azure AD 時，您可以︰

* 在 Azure AD 中控制可存取 Raketa 的人員。
* 讓使用者使用其 Azure AD 帳戶自動登入 Raketa。
* 在 Azure 入口網站集中管理您的帳戶。

若要深入了解 SaaS 應用程式與 Azure AD 整合，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on)。

## <a name="prerequisites"></a>必要條件

若要開始，您需要下列項目：

* Azure AD 訂用帳戶。 如果沒有訂用帳戶，您可以取得[免費帳戶](https://azure.microsoft.com/free/)。
* 已啟用 Raketa 單一登入 (SSO) 的訂用帳戶。

## <a name="scenario-description"></a>案例描述

在本教學課程中，您會在測試環境中設定和測試 Azure AD SSO。

* Raketa 支援 **SP** 起始的 SSO。
* 設定 Raketa 後，您可以強制執行工作階段控制項，以即時防止組織的敏感資料遭到外洩和滲透。 工作階段控制項會從條件式存取延伸。 [了解如何使用 Microsoft Cloud App Security 來強制執行工作階段控制項](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app)。

## <a name="adding-raketa-from-the-gallery"></a>從資源庫新增 Raketa

若要設定將 Raketa 整合到 Azure AD 中，您需要從資源庫將 Raketa 新增到受控 SaaS 應用程式清單。

1. 使用公司或學校帳戶或個人的 Microsoft 帳戶登入 [Azure 入口網站](https://portal.azure.com)。
1. 在左方瀏覽窗格上，選取 [Azure Active Directory] 服務 [1]。

    ![rkt_1](./media/raketa-tutorial/azure-active-directory.png)

1. 瀏覽至**企業應用程式** [2]，然後選取 [所有應用程式] [3]。

1. 若要新增應用程式，請選取 [新增應用程式] [4]。 

    ![rkt_2](./media/raketa-tutorial/new-app.png)

1. 在 [從資源庫新增] [5] 區段的搜尋方塊 [6] 中輸入 **Raketa**。

1. 從結果面板 [7] 選取 **Raketa**，然後按一下 [新增] 按鈕 [8]。 

    ![rkt_3](./media/raketa-tutorial/add-btn.png)


## <a name="configure-and-test-azure-ad-single-sign-on-for-raketa"></a>設定及測試適用於 Raketa 的 Azure AD 單一登入

以名為 **B.Simon** 的測試使用者，設定及測試與 Raketa 搭配運作的 Azure AD SSO。 若要讓 SSO 能夠運作，您必須建立 Azure AD 使用者與 Raketa 中相關使用者之間的連結關聯性。

若要設定及測試與 Raketa 搭配運作的 Azure AD SSO，請完成下列建置組塊：

1. **[設定 Azure AD SSO](#configure-azure-ad-sso)** - 讓您的使用者能夠使用此功能。
    1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 B.Simon 測試 Azure AD 單一登入。
    1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 B.Simon 能夠使用 Azure AD 單一登入。
1. **[設定 Raketa SSO](#configure-raketa-sso)** - 在應用程式端設定單一登入設定。
    1. **[建立 Raketa 測試使用者](#create-raketa-test-user)** - 使 Raketa 中對應的 B.Simon 連結到該使用者在 Azure AD 中的代表項目。
1. **[測試 SSO](#test-sso)** - 驗證組態是否能運作。

## <a name="configure-azure-ad-sso"></a>設定 Azure AD SSO

依照下列步驟在 Azure 入口網站中啟用 Azure AD SSO。

1. 在 [Azure 入口網站](https://portal.azure.com/)的 [Raketa] 應用程式整合頁面上，尋找 [管理] 區段並選取 [單一登入] [9]。

    ![rkt_4](./media/raketa-tutorial/manage-sso.png)

1. 在 [**選取單一登入方法**] 頁面 [9] 上，選取 **SAML** [10]。

    ![rkt_5](./media/raketa-tutorial/saml.png)

1. 在 [以 SAML 設定單一登入] 頁面上，按一下 [基本 SAML 設定] [11] 的編輯/畫筆圖示，以編輯設定。

1. 在 [基本 SAML 組態] 區段上，輸入下列欄位的值：

    1. 在 **識別碼 (實體識別碼)** [12] 和**登入 URL** [14] 文字方塊中，輸入 URL：`https://raketa.travel/`。

    1. 在**回覆 URL** [13] 文字方塊中，使用下列模式來輸入 URL：`https://raketa.travel/sso/acs?clientId=<CLIENT_ID>`。  

    ![rkt_6](./media/raketa-tutorial/enter-urls.png)

    > [!NOTE]
    > [回覆 URL] 不是真實的值。 請使用實際的「回覆 URL」來更新此值。 請連絡 [Raketa 用戶端支援小組](mailto:help@raketa.travel)以取得此值。 您也可以參考 Azure 入口網站中**基本 SAML 組態**區段所示的模式。

1. 在 [以 SAML 設定單一登入] 頁面的 [SAML 簽署憑證] 區段中，尋找**憑證 (Base64)** 並選取 [下載] [15]，以下載憑證並將其儲存在電腦上。

1. 在 [設定 Raketa] 區段上，根據您的需求複製適當的 URL。

    1. 登入 URL [16] - 用來將使用者重新導向至驗證系統的授權網頁 URL。

    1. Azure AD 識別碼 [17] - Azure AD 識別碼。

    1. 登出 URL [18] - 用來在登出後重新導向使用者的網頁 URL。

    ![rkt_7](./media/raketa-tutorial/copy-urls.png)


### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

在本節中，您將在 Azure 入口網站中建立名為 B.Simon 的測試使用者。

1. 在 Azure 入口網站的左窗格中，依序選取 [Azure Active Directory] [1]、[使用者] [19] 和 [所有使用者] [20]。

1. 在畫面頂端選取 [新增使用者] [21]。

    ![rkt_8](./media/raketa-tutorial/new-user.png)

1. 在 [使用者] 屬性中，執行下列步驟：

   1. 在 [使用者名稱] [22] 欄位中，輸入 username@companydomain.extension。 例如： `B.Simon@contoso.com` 。

   1. 在 [名稱] 欄位 [23]中，輸入 `B.Simon`。

   1. 選取 [顯示密碼] 核取方塊 [25]，然後記下 [密碼] 方塊 [24] 中顯示的值。

   1. 按一下 [建立] [26]。 

    ![rkt_9](./media/raketa-tutorial/create-user.png)


### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Raketa 的存取權授與 B.Simon，讓其能夠使用 Azure 單一登入。

1. 在 Azure 入口網站中，選取 [企業應用程式] [2]，然後選取 [所有應用程式] [3]。

1. 在應用程式清單中，選取 [Raketa] [27]。  

    ![rkt_10](./media/raketa-tutorial/add-raketa.png)

1. 在應用程式的概觀頁面中尋找 [管理] 區段，然後選取 [使用者和群組] [28]。 

    ![rkt_11](./media/raketa-tutorial/users-groups.png)

1. 選取 [新增使用者] [29]，然後在 [新增指派] 對話方塊中選取 [使用者和群組] [30]。

    ![rkt_12](./media/raketa-tutorial/add-user-raketa.png)

1. 在 [使用者和群組] 對話方塊的 [使用者] 清單中選取 [B.Simon] [31]，然後按一下畫面底部的 [選取] [32] 按鈕。

1. 如果您在 SAML 判斷提示中需要任何角色值，請在 [選取角色] 對話方塊的清單中為使用者選取適當的角色，然後按一下畫面底部的 [選取] 按鈕。

1. 在 [新增指派] 對話方塊中，按一下 [指派] 按鈕 [33]。 

    ![rkt_13](./media/raketa-tutorial/assign-user.png)

## <a name="configure-raketa-sso"></a>設定 Raketa SSO

若要設定 **Raketa** 端的單一登入，您必須將從 Azure 入口網站下載的 [憑證 (Base64)] 和複製的適當 URL 傳送給 [Raketa 支援小組](mailto:help@raketa.travel)。 他們會進行此設定，讓兩端的 SAML SSO 連線都設定正確。

### <a name="create-raketa-test-user"></a>建立 Raketa 測試使用者

在本節中，您會在 Raketa 中建立名為 B.Simon 的使用者。 請與 [Raketa 支援小組](mailto:help@raketa.travel)合作，在 Raketa 平台中新增使用者。 您必須先建立和啟動使用者，然後才能使用單一登入。

## <a name="test-sso"></a>測試 SSO

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在存取面板中按一下 [Raketa] 圖格時，應該會自動登入您已設定 SSO 的 Raketa。 如需「存取面板」的詳細資訊，請參閱[存取面板簡介](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction)。

## <a name="additional-resources"></a>其他資源

- [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [什麼是 Azure Active Directory 中的條件式存取？](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [嘗試搭配 Azure AD 使用 Raketa](https://aad.portal.azure.com/)

- [什麼是 Microsoft Cloud App Security 中的工作階段控制項？](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

- [如何使用進階可見性和控制項保護 Raketa](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)
---
title: 教學課程：使用 Azure Active Directory 設定 StarLeaf 來自動布建使用者 |Microsoft Docs
description: 瞭解如何設定 Azure Active Directory，以將使用者帳戶自動布建和取消布建至 StarLeaf。
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: article
ms.date: 07/19/2019
ms.author: zhchia
ms.openlocfilehash: 68484d2047e09dc82d44e15e86b13c4c54c49b03
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2020
ms.locfileid: "88533788"
---
# <a name="tutorial-configure-starleaf-for-automatic-user-provisioning"></a>教學課程：設定 StarLeaf 來自動布建使用者

本教學課程的目的是要示範在 StarLeaf 中執行的步驟，以及 Azure Active Directory (Azure AD) 將 Azure AD 設定為自動布建和解除布建使用者和/或群組至 StarLeaf。

> [!NOTE]
>  本教學課程會說明建置在 Azure AD 使用者佈建服務之上的連接器。 如需此服務的用途、運作方式和常見問題等重要詳細資訊，請參閱[使用 Azure Active Directory 對 SaaS 應用程式自動佈建和取消佈建使用者](../app-provisioning/user-provisioning.md)。
>
> 此連接器目前為預覽版。 如需有關預覽功能的一般 Microsoft Azure 使用規定詳細資訊，請參閱 [Microsoft Azure 預覽版增補使用規定](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

## <a name="prerequisites"></a>Prerequisites

本教學課程中概述的案例假設您已經具有下列必要條件：

* Azure AD 租用戶。
* [StarLeaf 租](https://starleaf.com/)使用者。
* StarLeaf 中具有系統管理員許可權的使用者帳戶。

## <a name="assign-users-to-starleaf"></a>將使用者指派給 StarLeaf
Azure Active Directory 使用所謂「指派」的概念，決定應該授權哪些使用者存取選取的應用程式。 在自動使用者布建的內容中，只會同步處理在 Azure AD 中指派給應用程式的使用者和/或群組。

設定並啟用自動使用者布建之前，您應該決定 Azure AD 中的哪些使用者和群組需要存取 StarLeaf。 然後，您可以依照下列 [指示](../manage-apps/assign-user-or-group-access-portal.md)，將使用者和群組指派給 StarLeaf。

## <a name="important-tips-for-assigning-users-to-starleaf"></a>將使用者指派給 StarLeaf 的重要秘訣

* 建議將單一 Azure AD 使用者指派給 StarLeaf，以測試自動使用者布建設定。 您可以稍後再指派其他使用者和群組。

* 當您將使用者指派給 StarLeaf 時，您必須在 [指派] 對話方塊中選取任何有效的應用程式特定角色 (如果有的話) 。 具有預設存取角色的使用者會從佈建中排除。

## <a name="set-up-starleaf-for-provisioning"></a>設定布建的 StarLeaf

使用 Azure AD 設定 StarLeaf 來自動布建使用者之前，您必須在 StarLeaf 中設定 SCIM 布建：

1. 登入您的 [StarLeaf 管理主控台](https://portal.starleaf.com/#page=login)。 流覽至 **[**  >  **整合新增整合**]。

    ![StarLeaf 新增 SCIM](media/starleaf-provisioning-tutorial/image00.png)

2. 選取要 Microsoft Azure Active Directory 的 **類型** 。 在 [ **名稱**] 中輸入適當的名稱。 按一下 [套用]。

    ![StarLeaf 新增 SCIM](media/starleaf-provisioning-tutorial/image01.png)

3.  接著會顯示 **SCIM 基底 URL** 和 **存取權杖** 值。 這些值將會在 Azure 入口網站中您 StarLeaf 應用程式的 [布建] 索引標籤中輸入至 [ **租使用者 URL** ] 和 [ **秘密權杖** ] 欄位中。 

    ![StarLeaf 建立權杖](media/starleaf-provisioning-tutorial/image02.png)

## <a name="add-starleaf-from-the-gallery"></a>從資源庫新增 StarLeaf

若要使用 Azure AD 設定 StarLeaf 來自動布建使用者，您需要從 Azure AD 應用程式資源庫將 StarLeaf 新增至受控 SaaS 應用程式清單。

**若要從 Azure AD 應用程式庫新增 StarLeaf，請執行下列步驟：**

1. 在 [ **[Azure 入口網站](https://portal.azure.com)** 的左側導覽面板中，選取 [ **Azure Active Directory**]。

    ![Azure Active Directory 按鈕](common/select-azuread.png)

2. 移至 [企業應用程式]，然後選取 [所有應用程式]。

    ![企業應用程式刀鋒視窗](common/enterprise-applications.png)

3. 若要加入新的應用程式，請選取窗格頂端的 [ **新增應用程式** ] 按鈕。

    ![新增應用程式按鈕](common/add-new-app.png)

4. 在搜尋方塊中，輸入 **StarLeaf**，在 [結果] 面板中選取 [ **StarLeaf** ]。
    ![結果清單中的 StarLeaf](common/search-new-app.png)

## <a name="configure-automatic-user-provisioning-to-starleaf"></a>設定自動使用者布建至 StarLeaf

本節將引導您逐步設定 Azure AD 布建服務，以根據 Azure AD 中的使用者和/或群組指派，在 StarLeaf 中建立、更新及停用使用者和/或群組。

1. 登入 [Azure 入口網站](https://portal.azure.com)。 選取 [企業應用程式]，然後選取 [所有應用程式]。

    ![企業應用程式刀鋒視窗](common/enterprise-applications.png)

2. 在應用程式清單中，選取 [ **StarLeaf**]。

    ![應用程式清單中的 StarLeaf 連結](common/all-applications.png)

3. 選取 [佈建] 索引標籤。

    ![佈建索引標籤](common/provisioning.png)

4. 將 [佈建模式] 設定為 [自動]。

    ![佈建索引標籤](common/provisioning-automatic.png)

5. 在 [管理員認證] 區段底下，分別輸入**租使用者 URL**和**秘密權杖**中先前抓取的**SCIM 基底 url**和**存取權杖**值。 按一下 [ **測試連接** ] 以確保 Azure AD 可以連線至 StarLeaf。 如果連接失敗，請確定您的 StarLeaf 帳戶具有系統管理員許可權，然後再試一次。

    ![租用戶 URL + 權杖](common/provisioning-testconnection-tenanturltoken.png)

6. 在 [ **通知電子郵件** ] 欄位中，輸入應收到布建錯誤通知之個人或群組的電子郵件地址，然後勾選 [ **發生失敗時傳送電子郵件通知** ] 方塊。

    ![通知電子郵件](common/provisioning-notification-email.png)

7. 按一下 [檔案] 。

8. **在 [對應**] 區段下，選取 [**同步處理 Azure Active Directory 使用者至 StarLeaf**]。

    ![StarLeaf 建立權杖](media/starleaf-provisioning-tutorial/usermapping.png)

9. 在 [ **屬性對應** ] 區段中，檢查從 Azure AD 同步處理到 StarLeaf 的使用者屬性。 選取為 [比對 **] 屬性的屬性會** 用來比對 StarLeaf 中的使用者帳戶以進行更新作業。 選取 [儲存] 按鈕以認可所有變更。

    ![StarLeaf 建立權杖](media/starleaf-provisioning-tutorial/userattribute.png)


10. 若要設定範圍篩選，請參閱[範圍篩選教學課程](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md)中提供的下列指示。


11. 若要啟用 StarLeaf Azure AD 的布建服務，請在 [**設定**] 區段中，將 [布建**狀態**] 變更為 [**開啟**]。

    ![佈建狀態已切換為開啟](common/provisioning-toggle-on.png)

12. 在 [**設定**] 區段的 [**範圍**] 中選擇所需的值，以定義您想要布建到 StarLeaf 的使用者和/或群組。

    ![佈建範圍](common/provisioning-scope.png)

13. 當您準備好要佈建時，按一下 [儲存]。

    ![儲存雲端佈建設定](common/provisioning-configuration-save.png)

此作業會對在 [設定]**** 區段的 [範圍]**** 中定義的所有使用者和/或群組，啟動首次同步處理。 初始同步處理會比後續同步處理花費更多時間執行，只要 Azure AD 佈建服務正在執行，這大約每 40 分鐘便會發生一次。 您可以使用 [ **同步處理詳細資料** ] 區段來監視進度，並遵循連結來布建活動報告，該報告描述 StarLeaf 上的 Azure AD 布建服務所執行的所有動作。

如需如何讀取 Azure AD 布建記錄的詳細資訊，請參閱自動布建 [使用者帳戶的報告](../app-provisioning/check-status-user-account-provisioning.md)

## <a name="connector-limitations"></a>連接器限制

* StarLeaf 目前不支援群組布建。 
* StarLeaf 需要 **電子郵件** 和使用者 **名稱** 值的來源值相同。

## <a name="additional-resources"></a>其他資源

* [管理企業應用程式的使用者帳戶佈建](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>後續步驟

* 瞭解如何 [查看記錄並取得布建活動的報告](../app-provisioning/check-status-user-account-provisioning.md)。

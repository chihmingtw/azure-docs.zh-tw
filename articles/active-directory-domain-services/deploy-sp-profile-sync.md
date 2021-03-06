---
title: 使用 Azure AD DS 啟用 SharePoint 使用者設定檔服務 |Microsoft Docs
description: 瞭解如何設定 Azure Active Directory Domain Services 受控網域以支援 SharePoint Server 的設定檔同步處理
services: active-directory-ds
author: iainfoulds
manager: daveba
ms.assetid: 938a5fbc-2dd1-4759-bcce-628a6e19ab9d
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: how-to
ms.date: 07/09/2020
ms.author: iainfou
ms.openlocfilehash: aa66f7150e5c3ac4560d7f97cb73b1effb8eec89
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/22/2020
ms.locfileid: "88749629"
---
# <a name="configure-azure-active-directory-domain-services-to-support-user-profile-synchronization-for-sharepoint-server"></a>設定 Azure Active Directory Domain Services 以支援 SharePoint Server 的使用者設定檔同步處理

SharePoint Server 包含用來同步處理使用者設定檔的服務。 這項功能可讓使用者設定檔儲存在中央位置，並可跨多個 SharePoint 網站和伺服器陣列存取。 若要設定 SharePoint Server 使用者設定檔服務，必須在 Azure Active Directory Domain Services (Azure AD DS) 受控網域中授與適當的許可權。 如需詳細資訊，請參閱 [SharePoint Server 中的使用者設定檔同步](/SharePoint/administration/user-profile-service-administration)處理。

本文說明如何設定 Azure AD DS 以允許 SharePoint Server 使用者設定檔同步處理服務。

## <a name="before-you-begin"></a>開始之前

若要完成本文章，您需要下列資源和權限：

* 有效的 Azure 訂用帳戶。
    * 如果您沒有 Azure 訂用帳戶，請先[建立帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。
* 與您的訂用帳戶相關聯的 Azure Active Directory 租用戶，可與內部部署目錄或僅限雲端的目錄同步。
    * 如果需要，請[建立 Azure Active Directory 租用戶][create-azure-ad-tenant]或[將 Azure 訂用帳戶與您的帳戶建立關聯][associate-azure-ad-tenant]。
* 已在您的 Azure AD 租用戶中啟用並設定 Azure Active Directory Domain Services 受控網域。
    * 如有需要，請完成教學課程，以 [建立並設定 Azure Active Directory Domain Services 受控網域][create-azure-ad-ds-instance]。
* 已加入 Azure AD DS 受控網域的 Windows Server 管理 VM。
    * 如有需要，請完成教學課程以 [建立管理 VM][tutorial-create-management-vm]。
* 屬於您 Azure AD 租用戶中 Azure AD DC 系統管理員群組成員的使用者帳戶。
* 使用者設定檔同步處理服務的 SharePoint 服務帳戶。
    * 如有需要，請參閱 [規劃 SharePoint Server 中的系統管理和服務帳戶][sharepoint-service-account]。

## <a name="service-accounts-overview"></a>服務帳戶總覽

在受控網域中，名為「 *AAD DC 服務帳戶* 」的安全性群組會在「 *使用者* 組織單位」 (OU) 中存在。 下列權限會委派給此安全性群組的成員：

- 在根 DSE 上複寫**目錄變更**許可權。
- *在 (* 容器) 的設定命名內容上複寫**目錄變更**許可權 `cn=configuration` 。

*AAD DC 服務帳戶*安全性群組也是內建組*Windows 之前的 Windows 2000 相容存取權*的成員。

當加入至此安全性群組時，會授與 SharePoint Server 使用者設定檔同步處理服務的服務帳戶正確運作所需的許可權。

## <a name="enable-support-for-sharepoint-server-user-profile-sync"></a>啟用 SharePoint Server 使用者設定檔同步處理的支援

SharePoint Server 的服務帳戶需要有足夠的許可權，才能將變更複寫至目錄，並讓 SharePoint Server 使用者設定檔同步運作正常。 若要提供這些許可權，請將用於 SharePoint 使用者設定檔同步處理的服務帳戶新增至 *AAD DC 服務帳戶* 群組。

從您的 Azure AD DS 管理 VM 中，完成下列步驟：

> [!NOTE]
> 若要編輯受控網域中的群組成員資格，您必須登入屬於 *AAD DC 系統管理員* 群組成員的使用者帳戶。

1. 從開始畫面選取 [系統 **管理工具**]。 您可以在教學課程中所安裝的可用管理工具清單中， [建立管理 VM][tutorial-create-management-vm]。
1. 若要管理群組成員資格，請從 [系統管理工具] 清單中選取 [ **Active Directory 管理中心** ]。
1. 在左窗格中，選擇您的受控網域，例如 *aaddscontoso.com*。 現有的 Ou 和資源清單會顯示出來。
1. 選取 [ **使用者** ] OU，然後選擇 [ *AAD DC 服務帳戶* ] 安全性群組。
1. 選取 [ **成員**]，然後選擇 [ **新增 ...**]。
1. 輸入 SharePoint 服務帳戶的名稱，然後選取 **[確定]**。 在下列範例中，SharePoint 服務帳戶的名稱為 *spadmin*：

    ![將 SharePoint 服務帳戶新增至 AAD DC 服務帳戶安全性群組](./media/deploy-sp-profile-sync/add-member-to-aad-dc-service-accounts-group.png)

## <a name="next-steps"></a>後續步驟

如需詳細資訊，請參閱 [管理 SharePoint Server 中的使用者設定檔同步](/SharePoint/administration/manage-profile-synchronization)處理。

<!-- INTERNAL LINKS -->
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
[create-azure-ad-ds-instance]: tutorial-create-instance.md
[tutorial-create-management-vm]: tutorial-create-management-vm.md

<!-- EXTERNAL LINKS -->
[sharepoint-service-account]: /sharepoint/security-for-sharepoint-server/plan-for-administrative-and-service-accounts
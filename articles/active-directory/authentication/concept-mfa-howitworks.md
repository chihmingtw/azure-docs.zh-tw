---
title: Azure Multi-Factor Authentication 總覽
description: 瞭解 Azure Multi-Factor Authentication 如何協助保護資料和應用程式的存取，同時滿足使用者對簡單登入程式的需求。
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 07/14/2020
ms.author: iainfou
author: iainfoulds
manager: daveba
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.openlocfilehash: b5f8d08f0814ec69719c002ea9efd39bb38b7d16
ms.sourcegitcommit: 6fc156ceedd0fbbb2eec1e9f5e3c6d0915f65b8e
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/21/2020
ms.locfileid: "88718011"
---
# <a name="how-it-works-azure-multi-factor-authentication"></a>運作方式：Azure Multi-Factor Authentication

多重要素驗證是使用者在登入過程中經提示而提供其他形式的識別 (例如，在其行動電話上輸入代碼或提供指紋掃描) 的程序。

如果僅使用密碼來驗證使用者，將會有不安全的媒介可能遭到攻擊。 如果密碼不嚴謹或已於他處公開，那麼以使用者名稱和密碼登入的究竟是使用者，還是攻擊者？ 當您要求第二種形式的驗證時，安全性將會增加，因為這項額外的因素並不容易讓攻擊者取得或複製。

![不同多重要素驗證形式的概念影像](./media/concept-mfa-howitworks/methods.png)

Azure Multi-Factor Authentication 的運作需要下列二種或更多的驗證方法：

* 您知道的某些資訊 (通常是密碼)。
* 您擁有的某些資訊，例如不容易複製的信任裝置 (如電話或硬體金鑰)。
* 代表您個人身分的某些資訊 - 指紋或臉部掃描之類的生物識別特徵。

使用者可以在單一步驟中將其身分註冊至自助式密碼重設和 Azure Multi-Factor Authentication，以簡化上線體驗。 系統管理員可定義可以使用的次要驗證形式。 在使用者執行自助式密碼重設以進一步保護該程序時，也可以要求 Azure Multi-Factor Authentication。

![在 [登入] 畫面使用中的驗證方法](media/concept-authentication-methods/overview-login.png)

Azure Multi-Factor Authentication 可協助保護資料和應用程式的存取，同時為使用者提供簡單的簡化。 它藉由要求第二種形式的驗證來提供額外的安全性，並透過一系列易於使用的[驗證方法](concept-authentication-methods.md)來提供增強式驗證。 因管理員所做的設定決定不同，使用者可能必須也可能無須通過 MFA。

您的應用程式或服務不需要進行任何變更，即可使用 Azure Multi-Factor Authentication。 驗證提示是 Azure AD 登入事件的一部分，可在必要時自動要求和處理 MFA 挑戰。

## <a name="available-verification-methods"></a>可用的驗證方法

當使用者登入應用程式或服務並收到 MFA 提示時，他們可以選擇其中一個已註冊的額外驗證形式。 系統管理員可能需要註冊這些 Azure Multi-Factor Authentication 驗證方法，或者使用者可以存取自己的 [ [我的設定檔](https://myprofile.microsoft.com) ] 來編輯或新增驗證方法。

下列額外形式的驗證可與 Azure Multi-Factor Authentication 搭配使用：

* Microsoft Authenticator 應用程式
* OATH 硬體權杖
* SMS
* 語音通話

## <a name="how-to-enable-and-use-azure-multi-factor-authentication"></a>如何啟用及使用 Azure Multi-Factor Authentication

您可以啟用 Azure Multi-Factor Authentication 的使用者和群組，在登入事件期間提示進行額外的驗證。 所有 Azure AD 租使用者都可以使用[安全性預設值](../fundamentals/concept-fundamentals-security-defaults.md)，以快速啟用所有使用者的 Microsoft Authenticator 應用程式。

如需更細微的控制，可以使用 [條件式存取](../conditional-access/overview.md) 原則來定義需要 MFA 的事件或應用程式。 當使用者在公司網路或已註冊的裝置上時，這些原則可以允許一般登入事件，但會在遠端或個人裝置上提示輸入其他驗證因素。

![條件式存取如何運作以保護登入程序的概觀圖表](media/tutorial-enable-azure-mfa/conditional-access-overview.png)

## <a name="next-steps"></a>後續步驟

若要瞭解授權，請參閱 [Azure Multi-Factor Authentication 的功能和](concept-mfa-licensing.md)授權。

若要查看作用中的 MFA，請在下列教學課程中為一組測試使用者啟用 Azure Multi-Factor Authentication：

> [!div class="nextstepaction"]
> [啟用 Azure Multi-Factor Authentication](./tutorial-enable-azure-mfa.md)
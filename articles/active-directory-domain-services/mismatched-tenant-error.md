---
title: 修正 Azure AD Domain Services 中不相符的目錄錯誤 |Microsoft Docs
description: 瞭解不相符的目錄錯誤的意義，以及如何在 Azure AD Domain Services 中解決此問題
services: active-directory-ds
author: iainfoulds
manager: daveba
ms.assetid: 40eb75b7-827e-4d30-af6c-ca3c2af915c7
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: troubleshooting
ms.date: 07/09/2020
ms.author: iainfou
ms.openlocfilehash: 0e21009341857cc6de3cb7aa411445bc10e6827e
ms.sourcegitcommit: f844603f2f7900a64291c2253f79b6d65fcbbb0c
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/10/2020
ms.locfileid: "86223477"
---
# <a name="resolve-mismatched-directory-errors-for-existing-azure-active-directory-domain-services-managed-domains"></a>解決現有 Azure Active Directory Domain Services 受控網域的不相符目錄錯誤

如果 Azure Active Directory Domain Services (Azure AD DS) 受控網域顯示不相符的租使用者錯誤，在解決之前，您將無法管理受控網域。 如果基礎 Azure 虛擬網路移至不同的 Azure AD 目錄，就會發生此錯誤。

本文說明發生錯誤的原因，以及解決方法。

## <a name="what-causes-this-error"></a>造成此錯誤的原因為何？

當 Azure AD DS 受控網域和虛擬網路屬於兩個不同的 Azure AD 租使用者時，就會發生不相符的目錄錯誤。 例如，您可能會有一個名為*aaddscontoso.com*的受控網域，它會在 Contoso 的 Azure AD 租使用者中執行。 不過，適用于受控網域的 Azure 虛擬網路是 Fabrikam Azure AD 租使用者的一部分。

Azure 會使用角色型存取控制 (RBAC) 來限制對資源的存取。 當您在 Azure AD 租使用者中啟用 Azure AD DS 時，認證雜湊會同步處理至受控網域。 此作業需要您是 Azure AD 目錄的租使用者系統管理員，而且必須控制認證的存取權。

若要將資源部署到 Azure 虛擬網路並控制流量，您必須在部署受控網域的虛擬網路上具有系統管理許可權。

為了讓 RBAC 能夠一致且安全地存取 Azure AD DS 使用的所有資源，受控網域和虛擬網路必須屬於相同的 Azure AD 租使用者。

下列規則適用于部署：

- Azure AD 目錄有多個 Azure 訂用帳戶。
- Azure 訂用帳戶有多個資源，例如虛擬網路。
- 已針對 Azure AD 目錄啟用單一受控網域。
- 受控網域可以在屬於相同 Azure AD 租使用者內任何 Azure 訂用帳戶的虛擬網路上啟用。

### <a name="valid-configuration"></a>有效的設定

在下列範例部署案例中，contoso 受控網域已在 Contoso Azure AD 租使用者中啟用。 受控網域會部署在屬於 Contoso Azure AD 租使用者所擁有之 Azure 訂用帳戶的虛擬網路中。

受控網域和虛擬網路都屬於相同的 Azure AD 租使用者。 此範例設定有效且完全受支援。

![具有相同 Azure AD 租使用者之受控網域和虛擬網路部分的有效 Azure AD DS 租使用者設定](./media/getting-started/valid-tenant-config.png)

### <a name="mismatched-tenant-configuration"></a>不相符租用戶設定

在此範例部署案例中，contoso Azure AD 租使用者中會啟用 Contoso 受控網域。 不過，受控網域會部署在屬於 Fabrikam Azure AD 租使用者所擁有之 Azure 訂用帳戶的虛擬網路中。

受控網域和虛擬網路屬於兩個不同的 Azure AD 租使用者。 此範例設定是不相符的租使用者，且不受支援。 虛擬網路必須移至與受控網域相同的 Azure AD 租使用者。

![不相符租用戶設定](./media/getting-started/mismatched-tenant-config.png)

## <a name="resolve-mismatched-tenant-error"></a>解決不相符的租使用者錯誤

下列兩個選項會解決不相符的目錄錯誤：

* 首先，從現有的 Azure AD 目錄中[刪除受控網域](delete-aadds.md)。 然後，在與您想要使用的虛擬網路相同的 Azure AD 目錄中，[建立替代的受控網域](tutorial-create-instance.md)。 準備好時，請將先前加入已刪除網域的所有機器加入重新建立的受控網域。
* 將包含虛擬網路的[Azure 訂](../cost-management-billing/manage/billing-subscription-transfer.md)用帳戶移至與受控網域相同的 Azure AD 目錄。

## <a name="next-steps"></a>後續步驟

如需有關 Azure AD DS 的問題疑難排解的詳細資訊，請參閱[疑難排解指南](troubleshoot.md)。

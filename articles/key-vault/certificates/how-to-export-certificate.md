---
title: 從 Azure Key Vault 匯出憑證
description: 了解如何從 Azure Key Vault 匯出憑證。
services: key-vault
author: sebansal
tags: azure-key-vault
ms.service: key-vault
ms.subservice: certificates
ms.topic: how-to
ms.custom: mvc, devx-track-azurecli
ms.date: 08/11/2020
ms.author: sebansal
ms.openlocfilehash: 408b507ff23a2a20c5ed6d2fc32fd5dbf5dbab5d
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/26/2020
ms.locfileid: "88930888"
---
# <a name="export-certificates-from-azure-key-vault"></a>從 Azure Key Vault 匯出憑證

了解如何從 Azure Key Vault 匯出憑證。 您可以使用 Azure CLI、Azure PowerShell 或 Azure 入口網站來匯出憑證。 也可以使用 Azure 入口網站來匯出 Azure App Service 憑證。

## <a name="about-azure-key-vault-certificates"></a>關於 Azure Key Vault 憑證

Azure Key Vault 可讓您輕鬆地為您的網路佈建、管理及部署數位憑證。 其也會啟用應用程式的安全通訊。 如需詳細資訊，請參閱 [Azure Key Vault 憑證](https://docs.microsoft.com/azure/key-vault/certificates/about-certificates)。

### <a name="composition-of-a-certificate"></a>憑證的構成要素

建立 Key Vault 憑證時，會建立相同名稱的可定址「金鑰」和「秘密」。 Azure Key Vault 允許金鑰作業。 Key Vault 祕密允許擷取憑證值作為秘密。 Key Vault 憑證也會包含公用 x509 憑證的中繼資料。 如需詳細資訊，請移至[憑證的構成要素](https://docs.microsoft.com/azure/key-vault/certificates/about-certificates#composition-of-a-certificate)。

### <a name="exportable-and-non-exportable-keys"></a>可匯出或不可匯出的金鑰

在建立 Key Vault 憑證之後，您可以使用私密金鑰，從可定址秘密中擷取該憑證。 擷取 PFX 或 PEM 格式的憑證。

- **可匯出**：用來建立憑證的原則指出金鑰是可匯出的。
- **不可匯出**：用來建立憑證的原則指出金鑰是不可匯出的。 在此情況下，當私密金鑰以秘密形式擷取時，不會將其視為值的一部分。

支援的金鑰類型：RSA、RSA-HSM、EC、EC-HSM、oct (列於[此處](https://docs.microsoft.com/rest/api/keyvault/createcertificate/createcertificate#jsonwebkeytype)) 僅允許匯出 RSA、EC。 HSM 金鑰不可匯出。

如需詳細資訊，請參閱[關於 Azure Key Vault 憑證](https://docs.microsoft.com/azure/key-vault/certificates/about-certificates#exportable-or-non-exportable-key)。

## <a name="export-stored-certificates"></a>匯出儲存的憑證

您可以使用 Azure CLI、Azure PowerShell 或 Azure 入口網站，匯出 Azure Key Vault 中儲存的憑證。

> [!NOTE]
> 只有當您在金鑰保存庫中匯入憑證時，才需要憑證密碼。 Key Vault 不會儲存相關聯的密碼。 當您匯出憑證時，密碼為空白。

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

在 Azure CLI 中使用下列命令，下載 Key Vault 憑證的**公開部分**。

```azurecli
az keyvault certificate download --file
                                 [--encoding {DER, PEM}]
                                 [--id]
                                 [--name]
                                 [--subscription]
                                 [--vault-name]
                                 [--version]
```

如需詳細資訊，請檢視[範例和參數定義](https://docs.microsoft.com/cli/azure/keyvault/certificate?view=azure-cli-latest#az-keyvault-certificate-download)。

如果您想要下載整個憑證 (即其構成要素的公開和私人部分)，請下載憑證作為祕密。

```azurecli
az keyvault secret download –file {nameofcert.pfx}
                            [--encoding {ascii, base64, hex, utf-16be, utf-16le, utf-8}]
                            [--id]
                            [--name]
                            [--subscription]
                            [--vault-name]
                            [--version]
```

如需詳細資訊，請參閱[參數定義](https://docs.microsoft.com/cli/azure/keyvault/secret?view=azure-cli-latest#az-keyvault-secret-download)。

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

在 Azure PowerShell 中使用此命令，從名為 **ContosoKV01** 的金鑰保存庫取得名為 **TestCert01** 的憑證。 若要將憑證下載為 PFX 檔案，請執行下列命令。 這些命令會存取 **SecretId**，然後將內容儲存為 PFX 檔案。

```azurepowershell
$cert = Get-AzKeyVaultCertificate -VaultName "ContosoKV01" -Name "TestCert01"
$kvSecret = Get-AzKeyVaultSecret -VaultName "ContosoKV01" -Name $Cert.Name
$kvSecretBytes = [System.Convert]::FromBase64String($kvSecret.SecretValueText)
$certCollection = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2Collection
$certCollection.Import($kvSecretBytes,$null,[System.Security.Cryptography.X509Certificates.X509KeyStorageFlags]::Exportable)
$password = '******'
$protectedCertificateBytes = $certCollection.Export([System.Security.Cryptography.X509Certificates.X509ContentType]::Pkcs12, $password)
$pfxPath = [Environment]::GetFolderPath("Desktop") + "\MyCert.pfx"
[System.IO.File]::WriteAllBytes($pfxPath, $protectedCertificateBytes)
```

此命令會利用私密金鑰匯出整個憑證鏈。 憑證使用密碼保護。
如需 **Get-AzKeyVaultCertificate** 命令和參數的詳細資訊，請參閱 [Get-AzKeyVaultCertificate - 範例 2](https://docs.microsoft.com/powershell/module/az.keyvault/Get-AzKeyVaultCertificate?view=azps-4.4.0)。

# <a name="portal"></a>[入口網站](#tab/azure-portal)

在 Azure 入口網站上，於您在 [憑證] 刀鋒視窗上建立/匯入憑證之後，會收到已成功建立憑證的通知。 選取憑證和目前版本來查看要下載的選項。

若要下載憑證，請選取 [以 CER 格式下載] 或 [以 PFX/PEM 格式下載]。

![憑證下載](../media/certificates/quick-create-portal/current-version-shown.png)

**匯出 Azure App Service 憑證**

Azure App Service 憑證是方便您購買 SSL 憑證的方式。 您可以從入口網站內將其指派給 Azure 應用程式。 也可以從入口網站將這些憑證匯出為 PFX 檔案，以便在其他地方使用。 在您匯入 PFX 檔案之後，App Service 憑證位於**祕密**之下。

如需詳細資訊，請參閱[匯出 Azure App Service 憑證](https://social.technet.microsoft.com/wiki/contents/articles/37431.exporting-azure-app-service-certificates.aspx)的步驟。

---

## <a name="read-more"></a>閱讀更多資訊
* [各種憑證檔案類型和定義](https://docs.microsoft.com/archive/blogs/kaushal/various-ssltls-certificate-file-typesextensions)

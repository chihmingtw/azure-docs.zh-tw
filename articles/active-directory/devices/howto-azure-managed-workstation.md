---
title: 部署 Azure 受控工作站-Azure Active Directory
description: 瞭解如何部署安全且受 Azure 管理的工作站，以降低因設定錯誤或洩漏而發生缺口的風險。
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: how-to
ms.date: 11/18/2019
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: frasim
ms.collection: M365-identity-device-management
ms.openlocfilehash: 841bc3ae4fbddb376ea4da8141bf4df3f895c4dc
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/01/2020
ms.locfileid: "89269551"
---
# <a name="deploy-a-secure-azure-managed-workstation"></a>部署安全的 Azure 管理工作站

現在您已 [瞭解安全工作站](concept-azure-managed-workstation.md)，現在可以開始進行部署程式。 透過本指南，您可以使用已定義的設定檔，從一開始就建立更安全的工作站。

![安全工作站的部署](./media/howto-azure-managed-workstation/deploying-secure-workstations.png)

在部署解決方案之前，請先選取設定檔。 您可以在部署中同時使用多個設定檔，並為其指派標籤或群組。

> [!NOTE]
> 視您的需求而需要套用任何設定檔。 您可以藉由在 Microsoft Intune 中指派，移至另一個設定檔。

| 設定檔 | 低 | 增強 | 高 | 特製化 | 受保護 | 隔離 |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| Azure AD 中的使用者 | 是 | 是 | 是 | 是 | 是 | 是 |
| 受 Intune 管理 | 是 | 是 | 是 | 是 | 是 | 是 |
| 裝置 Azure AD 已註冊 | 是 |  |  |  |  | |   |
| 已加入裝置 Azure AD |   | 是 | 是 | 是 | 是 | 是 |
| 已套用 Intune 安全性基準 |   | 是 <br>  (增強)  | 是 <br>  (HighSecurity)  | 是 <br>  (NCSC)  | 是 <br>  (安全)  | NA |
| 硬體符合安全的 Windows 10 標準 |   | 是 | 是 | 是 | 是 | 是 |
| 已啟用 Microsoft Defender ATP |   | 是  | 是 | 是 | 是 | 是 |
| 移除系統管理員許可權 |   |   | 是  | 是 | 是 | 是 |
| 使用 Microsoft Autopilot 進行部署 |   |   | 是  | 是 | 是 | 是 |
| 僅由 Intune 安裝的應用程式 |   |   |   | 是 | 是 |是 |
| Url 僅限核准清單 |   |   |   | 是 | 是 |是 |
| 網際網路封鎖 (輸入/輸出)  |   |   |   |  |  |是 |

> [!NOTE]
> 在安全工作站指引中，會將設定檔和原則指派給 **裝置** 。 使用者不會直接套用原則，讓裝置共用 (共用裝置) 生效。 如果您的部署中未共用安全的工作站，或需要個別的使用者原則，則可以將使用者原則設定檔指派指派給使用者和裝置。 

## <a name="license-requirements"></a>授權需求

本指南涵蓋的概念假設您有 Microsoft 365 企業版 E5 或對等的 SKU。 本指南中的某些建議可使用較低的 Sku 來執行。 如需詳細資訊，請參閱 [Microsoft 365 企業版授權](https://www.microsoft.com/licensing/product-licensing/microsoft-365-enterprise)。

若要將授權布建自動化，請考慮以 [群組為基礎的授權](../users-groups-roles/licensing-groups-assign.md) 給您的使用者。

## <a name="azure-active-directory-configuration"></a>Azure Active Directory 組態

Azure Active Directory (Azure AD) 管理系統管理員工作站的使用者、群組和裝置。 使用 [系統管理員帳戶](../users-groups-roles/directory-assign-admin-roles.md)啟用身分識別服務和功能。

當您建立安全的工作站系統管理員帳戶時，您會將帳戶公開給目前的工作站。 請務必使用已知的安全裝置來執行此初始設定和所有全域設定。 若要減少首次體驗的攻擊風險，請考慮遵循指導方針 [來防止惡意](/windows/security/threat-protection/intelligence/prevent-malware-infection)代碼的感染。

至少需要您的系統管理員進行多重要素驗證。 請參閱 [部署以雲端為基礎的 MFA](../authentication/howto-mfa-getstarted.md) 以取得實施指引。

### <a name="azure-ad-users-and-groups"></a>Azure AD 使用者和群組

1. 在 Azure 入口網站中，流覽至**Azure Active Directory**  >  **使用者**的  >  **新使用者**。
1. 遵循 [建立使用者教學](/Intune/quickstart-create-user)課程中的步驟來建立您的裝置系統管理員。
1. 輸入：

   * **名稱** -安全的工作站系統管理員
   * **使用者名稱** - `secure-ws-admin@identityitpro.com`
   * **目錄角色**  - **受限的系統管理員**，然後選取**Intune 系統管理員**角色。

1. 選取 [建立]  。

接下來，您會建立兩個群組：工作站使用者和工作站裝置。

在 Azure 入口網站中，流覽至**Azure Active Directory**  >  **群組**  >  **新增群組**。

1. 針對工作站使用者群組，您可能會想要設定以 [群組為基礎的授權](../users-groups-roles/licensing-groups-assign.md) ，將授權的布建自動化至使用者。
1. 針對工作站使用者群組，請輸入：

   * **群組類型** -安全性
   * **組名** -安全工作站使用者
   * **成員資格類型** -已指派

1. 新增您的安全工作站系統管理員使用者： `secure-ws-admin@identityitpro.com`
1. 您可以新增任何其他將管理安全工作站的使用者。
1. 選取 [建立]  。
1. 針對工作站裝置群組，請輸入：

   * **群組類型** -安全性
   * **組名** -安全工作站
   * **成員資格類型** -已指派

1. 選取 [建立]  。

### <a name="azure-ad-device-configuration"></a>Azure AD 裝置設定

#### <a name="specify-who-can-join-devices-to-azure-ad"></a>指定可以將裝置加入 Azure AD 的人員

在 Active Directory 中設定您的裝置設定，以允許您的系統管理安全性群組將裝置加入您的網域。 若要從 Azure 入口網站設定此設定：

1. 移至 [Azure Active Directory] > [裝置] > [裝置設定]。
1. 選擇 [**使用者可以將裝置加入至 Azure AD**] 底下選取 [已**選取**]，然後選取 [安全工作站使用者] 群組。

#### <a name="removal-of-local-admin-rights"></a>移除本機系統管理員許可權

此方法要求 VIP、DevOps 和安全層級工作站的使用者沒有其電腦的系統管理員許可權。 若要從 Azure 入口網站設定此設定：

1. 移至 [Azure Active Directory] > [裝置] > [裝置設定]。
1. 在**加入 Azure AD 的裝置上的其他本機系統管理員**下，選取 [**無**]。

#### <a name="require-multi-factor-authentication-to-join-devices"></a>需要多重要素驗證才能加入裝置

進一步加強將裝置加入 Azure AD 的程式：

1. 移至 [Azure Active Directory] > [裝置] > [裝置設定]。
1. 在 [**需要多重要素驗證來加入裝置**] 下方選取 **[是]** 。
1. 選取 \[儲存\]。

#### <a name="configure-mobile-device-management"></a>設定行動裝置管理

從 Azure 入口網站：

1. 流覽至**Azure Active Directory**  >  **行動 (MDM 和 MAM) **  >  **Microsoft Intune**。
1. 將 [ **MDM 使用者範圍** ] 設定變更為 [ **全部**]。
1. 選取 \[儲存\]。

這些步驟可讓您使用 Intune 管理任何裝置。 如需詳細資訊，請參閱 [Intune 快速入門：設定 Windows 10 裝置的自動註冊](/Intune/quickstart-setup-auto-enrollment)。 您在未來的步驟中會建立 Intune 設定和合規性原則。

#### <a name="azure-ad-conditional-access"></a>Azure AD 條件式存取

Azure AD 條件式存取有助於將特殊許可權的系統管理工作限制為符合規範的裝置。 登入雲端應用程式時，需要 **安全工作站使用者** 群組的預先定義成員，才能執行多重要素驗證。 最佳做法是從原則中排除緊急存取帳戶。 如需詳細資訊，請參閱 [Azure AD 中的管理緊急存取帳戶](../users-groups-roles/directory-emergency-access.md)。

## <a name="intune-configuration"></a>Intune 設定

### <a name="configure-enrollment-status"></a>設定註冊狀態

請務必確定您的安全工作站是受信任的乾淨裝置。 當您購買新的裝置時，您可以堅持將它們設定為 [在 S 模式中 Windows 10 專業版](/Windows/deployment/Windows-10-pro-in-s-mode)，以在供應鏈管理期間限制暴露于弱點。 從供應商收到裝置之後，您可以使用 Autopilot 從 S 模式變更。 下列指導方針提供有關套用轉換流程的詳細資料。

為了確保在使用之前完全設定裝置，Intune 提供了在 **安裝所有應用程式和設定檔之前，封鎖裝置使用**的方法。

從 **Azure 入口網站**：

1. 移至**Microsoft Intune**  >  **裝置註冊**  >  **Windows 註冊**  >  **註冊狀態頁面**  >  的**預設**  >  **設定**。
1. 將 [ **顯示應用程式佈建檔安裝進度** ] 設定為 **[是]**。
1. 設定 [ **封鎖裝置使用]，直到所有應用程式和設定檔都安裝** 為 **[是]**。

### <a name="create-an-autopilot-deployment-profile"></a>建立 Autopilot 部署設定檔

建立裝置群組之後，您必須建立部署設定檔來設定 Autopilot 裝置。

在 Intune 的 Azure 入口網站：

1. 選取**裝置註冊**  >  **Windows 註冊**  >  **部署配置**檔  >  **建立設定檔**。
1. 輸入：

   * 名稱- **安全的工作站部署設定檔**。
   * 描述- **安全工作站的部署**。
   * 把 [將所有目標裝置轉換為 Autopilot]  設為 [是]  。 此設定可確保清單中的所有裝置都會向 AutoPilot 部署服務註冊。 等候 48 小時讓註冊處理完畢。

1. 選取 [下一步]  。

   * 針對 [ **部署模式]**，選擇 [ **自我部署 (預覽]) **。 具有此設定檔的裝置會與註冊裝置的使用者相關聯。 需有使用者認證，才能註冊裝置。 請務必注意，在 **自我部署** 模式中部署裝置可讓您在共用模型中部署膝上型電腦。 在裝置第一次指派給使用者之前，不會進行任何使用者指派。 因此，在使用者指派完成之前，將不會啟用任何使用者原則（例如 BitLocker）。 如需如何登入受保護裝置的詳細資訊，請參閱 [選取的設定檔](/intune/device-profile-assign)。
   * [ **加入 Azure AD 為** ] 方塊應該會顯示 **Azure AD 已加入** 並呈現灰色。
   * 選取您的語言 (區域) 、使用者帳戶類型 **標準**。 

1. 選取 [下一步]  。

   * 如果您已預先設定範圍標籤，請加以選取。

1. 選取 [下一步]  。
1. 選擇**Assignments**  >  **指派給**  >  **所選群組**的指派。 在 [ **選取要包含的群組**] 中，選擇 [ **安全工作站**]。
1. 選取 [下一步]  。
1. 選取 [建立] 以建立設定檔。 現在可以指派 Autopilot 部署設定檔給裝置。

Autopilot 中的裝置註冊會根據裝置類型和角色提供不同的使用者體驗。 在我們的部署範例中，我們會說明一個模型，其中受保護的裝置會大量部署並可共用，但在第一次使用時，會將裝置指派給使用者。 如需詳細資訊，請參閱 [Intune Autopilot 裝置註冊](/intune/device-enrollment)。

### <a name="configure-windows-update"></a>設定 Windows Update

將 Windows 10 保持在最新狀態是最重要的一件事。 若要維護處於安全狀態的 Windows，您可以部署更新通道來管理更新套用至工作站的步調。 

本指引建議您建立新的更新通道，並變更下列預設設定：

在 Azure 入口網站中：

1. 移至**Microsoft Intune**的  >  **軟體更新**  >  **Windows 10 更新**通道。
1. 輸入：

   * 名稱- **Azure 管理的工作站更新**
   * 維護通道- **Windows 測試人員-快速**
   * 品質更新延遲 (天) - **3**
   * 功能更新延遲期間 (天) - **3**
   * 自動更新行為-在 **沒有使用者控制的情況下自動安裝並重新啟動**
   * 防止使用者暫停 Windows 更新- **封鎖**
   * 需要使用者核准才能在工作時間以外重新開機- **必要**
   * 允許使用者重新開機 (預約重新開機) - **必要**
   * 自動重新開機後將使用者轉換成預約重新開機 (天) - **3**
   * 延遲預約重新開機提醒 (天) - **3**
   * 設定暫止重新開機的期限 (天) - **3**

1. 選取 [建立]  。
1. 在 [ **指派** ] 索引標籤上，新增 **安全工作站** 群組。

如需 Windows Update 原則的詳細資訊，請參閱 [原則 CSP-更新](/windows/client-management/mdm/policy-csp-update)。

### <a name="windows-defender-atp-intune-integration"></a>Windows Defender ATP Intune 整合

Windows Defender ATP 與 Microsoft Intune 合作，以協助防止安全性缺口。 它們也可以限制缺口的影響。 ATP 功能提供即時威脅偵測，以及啟用端點裝置的廣泛審核和記錄。

若要設定 Windows Defender ATP 與 Intune 的整合，請移至 Azure 入口網站。

1. 流覽至**Microsoft Intune**  >  的**裝置合規性**  >  **Windows Defender ATP**。
1. 在 [設定 **Windows Defender ATP**] 下的步驟1中，選取 **[連線 Windows Defender ATP 以 Microsoft Intune 到 Windows Defender 的安全性中心**。
1. 在 [Windows Defender 資訊安全中心] 中：

   1. 選取 [設定] > [進階功能]。
   1. 針對 **Microsoft Intune 連接**，請選擇 [ **開啟**]。
   1. 選取 [儲存喜好設定]。

1. 建立連線之後，請返回 Intune， **然後選取頂端的 [** 重新整理]。
1. 將 [將 Windows 裝置 10.0.15063 版和更高版本連線至 Windows Defender ATP]**** 設定為 [開啟]****。
1. 選取 \[儲存\]。

如需詳細資訊，請參閱 [Windows Defender 進階威脅防護](/Windows/security/threat-protection/windows-defender-atp/windows-defender-advanced-threat-protection)。

### <a name="finish-workstation-profile-hardening"></a>完成工作站設定檔強化

若要成功完成解決方案的強化，請下載並執行適當的腳本。 尋找您所需 **設定檔層級**的下載連結：

| 設定檔 | 下載位置 | 檔案名稱 |
| --- | --- | --- |
| 低安全性 | N/A | N/A |
| 加強的安全性 | https://aka.ms/securedworkstationgit | 增強-工作站-Windows10- (1809) ps1 |
| 高安全性 | https://aka.ms/securedworkstationgit | HighSecurityWorkstation-Windows10- (1809) ps1 |
| 特製化 | https://github.com/pelarsen/IntunePowerShellAutomation | DeviceConfiguration_NCSC-Windows10 (1803) SecurityBaseline.ps1 |
| 特殊化合規性 * | https://aka.ms/securedworkstationgit | DeviceCompliance_NCSC-Windows10 (1803) ps1 |
| 受保護 | https://aka.ms/securedworkstationgit | 安全-工作站-Windows10- (1809) # A0 |

\* 特殊化合規性是一種腳本，會強制執行 NCSC Windows10 SecurityBaseline 中提供的特殊設定。

順利執行腳本之後，您可以在 Intune 中更新設定檔和原則。 增強和安全設定檔的腳本會為您建立原則和設定檔，但您必須將原則指派給您的 **安全工作站** 裝置群組。

* 您可以在這裡找到腳本所建立的 Intune 裝置設定檔： **Azure 入口網站**  >  **Microsoft Intune**  >  **裝置配置**  >  **檔**。
* 您可以在這裡找到腳本所建立的 Intune 裝置合規性原則： **Azure 入口網站**  >  **Microsoft Intune**  >  **裝置合規性**  >  **原則**。

若要檢查腳本所做的變更，您可以匯出設定檔。 如此一來，您可以依照 [SECCON 檔](/windows/security/threat-protection/windows-security-configuration-framework/windows-security-configuration-framework)中所述，判斷可能需要的其他強化措施。

`DeviceConfiguration_Export.ps1`從[DeviceConfiguration GiuHub 存放庫](https://github.com/microsoftgraph/powershell-intune-samples/tree/master/DeviceConfiguration)執行 intune 資料匯出腳本，以匯出所有目前的 intune 設定檔。

## <a name="additional-configurations-and-hardening-to-consider"></a>要考慮的其他設定和強化

遵循此處的指引，您已部署安全的工作站。 不過，您也應該考慮其他控制項。 例如：

* 限制存取替代瀏覽器
* 允許輸出 HTTP
* 封鎖選取網站
* 設定執行自訂 PowerShell 腳本的許可權

### <a name="set-rules-in-the-firewall-configuration-service-provider-csp"></a>在防火牆設定服務提供者 (CSP) 中設定規則

您可以視需要變更輸入和輸出規則，以針對您的允許和封鎖端點進行管理。 當您繼續強化安全工作站時，可以放寬拒絕所有輸入和輸出流量的限制。 您可以新增允許的輸出網站，以包含常見和信任的網站。 如需詳細資訊，請參閱 [防火牆設定服務](/Windows/client-management/mdm/firewall-csp)。

限制的 URL 流量管理包括：

* 拒絕在安全工作站設定檔腳本中設定的所有輸入流量。
* 拒絕選取的 Azure 和 Microsoft 服務以外的所有輸出，包括 Azure Cloud Shell 和允許 Azure 密碼重設的能力。

```
[HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings]
"ProxyEnable"=dword:00000001
"ProxyServer"="127.0.0.1:80"
"ProxyOverride"="*.azure.com;*.azure.net;*.microsoft.com;*.windowsupdate.com;*.microsoftonline.com;*.microsoftonline.cn;*.windows.net;*.windowsazure.com;*.windowsazure.cn;*.azure.cn;*.loganalytics.io;*.applicationinsights.io;*.vsassets.io;*.azure-automation.net;*.visualstudio.com,portal.office.com;*.aspnetcdn.com;*.sharepointonline.com;*.msecnd.net;*.msocdn.com;*.webtrends.com"
"AutoDetect"=dword:00000000
```

如果您想要提供更多細微性給您的封鎖規則，您可以參考維護 [網域封鎖清單的 Spamhaus 專案 (雙) ](https://www.spamhaus.org/dbl/)：這是適合用來做為封鎖網站之先進規則集的良好資源。

### <a name="manage-local-applications"></a>管理本機應用程式

移除本機應用程式時，安全工作站會移至真正強化的狀態，包括生產力應用程式。 在這裡，您會新增 Chrome 作為預設瀏覽器，並使用 Intune 來限制使用者修改瀏覽器和其外掛程式的能力。

#### <a name="deploy-applications-using-intune"></a>使用 Intune 部署應用程式

在某些情況下，受保護的工作站上需要有應用程式，例如 Google Chrome 瀏覽器。 下列範例提供將 Chrome 安裝至安全性群組安全 **工作站**中之裝置的指示。

1. 下載 [適用于 Windows 64](https://cloud.google.com/chrome-enterprise/browser/download/)位的離線安裝程式 Chrome 套件組合。
1. 解壓縮檔案，並記下檔案的位置 `GoogleChromeStandaloneEnterprise64.msi` 。
1. 在**Azure 入口網站**流覽至**Microsoft Intune**  >  **用戶端應用**  >  **程式**  >  **新增**]。
1. 在 [ **應用程式類型**] 下，選擇 [ **企業**營運]。
1. 在 [ **應用程式套件**檔案] 下，從解壓縮的位置選取檔案， `GoogleChromeStandaloneEnterprise64.msi` 然後選取 **[確定]**。
1. 在 [ **應用程式資訊**] 下，提供描述和發行者。 選取 [確定]。
1. 選取 [新增]。
1. 在 [**指派**] 索引標籤上，選取 [**指派類型**] 下的 [已**註冊裝置可用**]
1. 在 [ **包含的群組**] 底下，新增 **安全工作站** 群組。
1. 選取 [確定]，然後選取 [儲存]。

如需有關設定 Chrome 設定的詳細資訊，請參閱 [使用 Microsoft Intune 管理 Chrome 瀏覽器](https://support.google.com/chrome/a/answer/9102677)。

#### <a name="configuring-the-company-portal-for-custom-apps"></a>設定自訂應用程式的公司入口網站

在安全模式中，應用程式安裝僅限於 Intune 公司入口網站。 不過，安裝入口網站需要 Microsoft Store 的存取權。 在安全的解決方案中，您可以透過離線模式，讓公司入口網站可供所有裝置使用。

受 Intune 管理的 [公司入口網站](/Intune/store-apps-company-portal-app) 複本可讓您依需求存取可向下推送至受保護工作站使用者的其他工具。

您可能需要安裝 Windows 32 位應用程式或部署需要特殊準備的其他應用程式。 在這種情況下， [Microsoft win32 內容準備工具](https://github.com/Microsoft/Microsoft-Win32-Content-Prep-Tool) 可以提供立即可用的格式檔案 `.intunewin` 來進行安裝。

### <a name="conditional-access-only-allowing-secured-workstation-ability-to-access-azure-portal"></a>條件式存取僅允許安全的工作站存取 Azure 入口網站

Azure AD 提供管理和限制的能力，以及誰可以存取您的 Azure 雲端管理入口網站。 啟用 [條件式存取](../conditional-access/overview.md) 可確保只有您的安全工作站可以管理或變更資源。 當您部署這項功能時，請務必在部署這項功能時，如果 [緊急存取](../users-groups-roles/directory-emergency-access.md) 功能可以或應該僅用於極端案例和透過原則管理的帳戶。

> [!NOTE]
> 您將需要建立使用者群組，並包含可以略過條件式存取原則的緊急使用者。 在我們的範例中，我們有一個稱為**緊急 BreakGlass**的安全性群組

1. 流覽至**Azure 入口網站**  >  **Microsoft Intune**  >  **條件式存取-原則的**  >  **新原則**。
1. 提供原則的 **名稱** 。
1. 選取**使用者和群組**  >  **選取使用者和群組** 
1. 選取 [**包含**  >  **目錄角色**] > 選擇 > 全域系統管理員、特殊許可權角色系統管理員、特殊許可權驗證管理員、安全性系統管理員、規範管理員、條件式存取系統管理員、應用程式系統管理員、雲端應用程式系統管理員、Intune 服務管理員等角色
1. 選取 [ **排除** ] > 選擇 [ **使用者和群組** ] > 選取 [ **選取排除的使用者** ] > 選取 **緊急 BreakGlass** 群組。
1. 選取 **雲端應用程式或動作** > 選取 **所有雲端應用程式**
1. 選取 **條件** > 選取 **裝置平臺** > 選擇設定 **[是]** > 選取 [ **選取裝置平臺** ] 選擇 **視窗**
1. 選取 [ **存取控制** ] > 選取 **[授與存取權** **]** > 選擇 [ **要求裝置必須標示為符合規範**]。 
1. 選取 [**啟用原則**  >  **On** ]
 
此原則設定可確保您的系統管理員必須使用受 Intune 設定的相容 Windows 裝置，以及 WDATP。 

組織也應該考慮封鎖其環境中的舊版驗證通訊協定。 有多種方式可以完成這項工作，如需有關封鎖舊版驗證通訊協定的詳細資訊，請參閱文章 [：如何：封鎖舊版驗證以使用條件式存取 Azure AD](../conditional-access/block-legacy-authentication.md)。

### <a name="use-powershell-to-create-custom-settings"></a>使用 PowerShell 建立自訂設定

您也可以使用 PowerShell 來擴充主機管理功能。 此範例腳本會在主機上設定預設背景。 它也是透過 Azure 入口網站和設定檔提供的功能。 範例腳本僅用來說明 PowerShell 功能。

您可能需要在安全工作站上設定一些自訂控制項和設定。 此範例會使用 Powershell 輕鬆將裝置識別為立即可用、安全的工作站，來變更工作站的背景。

Microsoft 腳本中心提供的 [SetDesktopBackground.ps1](https://gallery.technet.microsoft.com/scriptcenter/Set-Desktop-Image-using-5430c9fb/) 腳本，可讓 Windows 在啟動時載入這個 [免費的一般背景影像](https://i.imgur.com/OAJ28zO.png) 。

1. 將腳本下載到本機裝置。
1. 更新背景影像的 customerXXXX 和下載位置。 在我們的範例中，我們會將 customerXXXX 取代為背景。
1. 流覽至**Azure 入口網站**  >  **Microsoft Intune**  >  **裝置**設定  >  **PowerShell 腳本**  >  **新增**。
1. 提供腳本的 **名稱** ，並指定 **腳本位置**。
1. 選取 [設定] 。
   1. 將 **[使用登入認證執行此腳本** ] 設定為 **[是]**。
   1. 選取 [確定]  。
1. 選取 [建立]  。
1. 選取**指派**  >  **選取群組**。
   1. 新增安全性群組安全 **工作站**。
   1. 選取 \[儲存\]。

## <a name="enroll-and-validate-your-first-device"></a>註冊並驗證您的第一部裝置

1. 若要註冊您的裝置，您需要下列資訊：
   * **序號** -在裝置底座上找到。
   * **Windows 產品識別碼**-在**System**  >  **About** [windows 設定] 功能表的 [系統] 底下找到。
   * 您可以執行 [WindowsAutoPilotInfo](https://aka.ms/Autopilotshell) 來取得 CSV 雜湊檔，其中包含裝置註冊所需的所有資訊。
   
     執行 `Get-WindowsAutoPilotInfo – outputfile device1.csv` 以將資訊輸出為 CSV 檔案，您可以將其匯入至 Intune。

     > [!NOTE]
     > 腳本需要較高的許可權。 它會以遠端簽署的方式執行。 `Set-ExecutionPolicy -ExecutionPolicy RemoteSigned`命令可讓腳本正確執行。

   * 您可以藉由登入 Windows 10 1809 版或更高版本的裝置來收集這項資訊。 您的硬體轉銷商也可以提供此資訊。
1. 在**Azure 入口網站**中，移至**Microsoft Intune**  >  **裝置註冊**  >  **Windows 註冊**  >  **裝置-管理 Windows Autopilot 裝置**。
1. 選取 [匯 **入** ]，然後選擇您的 CSV 檔案。
1. 將裝置新增至 **安全工作站** 安全性群組。
1. 在您想要設定的 Windows 10 裝置上，移至 [ **Windows 設定**  >  **更新 & 安全性**  >  **修復**]。
   1. 在 [**重設這部電腦**] 下選擇 [**開始**使用]。
   1. 遵循提示來重設並重新設定已設定設定檔和合規性原則的裝置。

設定裝置之後，請完成審核並檢查設定。 先確認第一個裝置已正確設定，再繼續進行部署。

## <a name="assign-devices"></a>指派裝置

若要指派裝置和使用者，您必須將 [選取的設定檔](/intune/device-profile-assign) 對應至您的安全性群組。 所有需要服務許可權的新使用者也必須新增至安全性群組。

## <a name="using-sentinel-and-windows-defender-atp-to-monitor-and-respond-to-security-incidents"></a>使用 Sentinel 和 Windows Defender ATP 來監視和回應安全性事件

您可以藉由啟用 [Sentinel] 並利用 [威脅和弱點管理](/windows/security/threat-protection/microsoft-defender-atp/next-gen-threat-and-vuln-mgt) 來監視安全的工作站部署，本指南不會提供徹底的威脅搜尋，而是為了監視和回應潛在的安全性事件，提供良好的一般意義。

我們會使用 **Azure Sentinel** 來： 

* 從 Intune、Azure 入口網站和 Azure AD 收集資料，以進行使用者和裝置監視
* 協助調查使用者和裝置設定的可疑活動
* 並使用 WDATP 推動探索安全性調查的能力

Sentinel 監視要求您的資料來源（例如 Azure AD）必須設定連接器。

1. 在**Azure 入口網站**中，移至**Azure Sentinel (Preview) ** > 選取 [**新增**]
1. 在 [**選擇要加入的工作區] Azure Sentinel**選取 [**建立新的工作區**]
1. 輸入：
   * **Log Analytics 工作區** -「安全工作站監視」
   * **訂** 用帳戶-選取您的有效訂用帳戶
   * **資源群組** -選取 [建立新的] > 安全工作站 RG > **確定]**
   * **位置** -選取地理位置最適合您部署的位置
   * **定價層** -選取 **每 GB (2018) **
1. 選取 [確定]  。

接下來，我們會將可用的安全工作站資料來源連接到監視。

1. 在 **Azure 入口網站**中，移至 **Azure Sentinel 工作** 區 > 選取 **安全工作站監視** 工作區
1. 選取 **資料連線器**
1. 查看必要條件之後，選擇 **Azure Active Directory** > 開啟連接器] 頁面 >。 繼續進行設定，然後選取 **[連線]** Azure AD 登入記錄，以及 Azure AD Audit 記錄檔。
1. 查看必要條件之後，選擇 [ **Azure 活動** > 開啟連接器] 頁面 >。 繼續設定 Azure 活動記錄 > 選取您的訂用帳戶 > 選取 **[連線]**

當 Sentinel 收集資料時，您將能夠藉由選取 **Azure 入口網站**來觀察活動，請移至 **Azure Sentinel 總覽** 

我們將使用 **Windows Defender ATP (WDATP) ** ：

* 持續觀察並監視弱點和錯誤設定
* 利用 WDATP 來優先處理動態威脅
* 使用端點偵測和回應來驅動弱點的相互關聯 (EDR) 警示
* 使用儀表板來識別調查期間的電腦層級弱點
* 將補救推送至 Intune

設定您的 [DEFENDER ATP 儀表板](https://securitycenter.windows.com/machines)。 使用威脅的指引 [& 弱點管理儀表板總覽](/windows/security/threat-protection/microsoft-defender-atp/tvm-dashboard-insights)。

## <a name="monitoring-application-activity-using-microsoft-monitoring-agent-mma"></a>使用 Microsoft Monitoring Agent (MMA) 監視應用程式活動
從特製化的工作站開始，會啟用應用程式保險箱來監視工作站上的應用程式活動。 若要將監視整合到您的 Log Analytics 工作區，必須遵循 MMA 代理程式和設定。 

> [!NOTE]
> 特製化的工作站設定檔包含 AppLocker 監視原則。 需要部署原則，才能監視用戶端上的應用程式活動

使用 Intune PowerShell 腳本部署 MMA 代理程式

1. 將安裝 [腳本下載到本機裝置](https://aka.ms/securedworkstationgit)。
1. 更新參數、 **$WorkSpaceID** 和 **$WorkSpaceKey**
1. 流覽至**Azure 入口網站**  >  **Microsoft Intune**  >  **裝置**設定  >  **PowerShell 腳本**  >  **新增**。
1. 提供腳本的 **名稱** ，並指定 **腳本位置**。
1. 選取 [設定] 。
   1. 將 **[使用登入認證執行此腳本** ] 設定為 **[是]**。
   1. 選取 [確定]  。
1. 選取 [建立]  。
1. 選取**指派**  >  **選取群組**。
   1. 新增安全性群組安全 **工作站**。
   1. 選取 \[儲存\]。

接下來，您必須設定 Log Analytics 以接收新的記錄
1. 在 **Azure 入口網站**中，移至 **Log Analytics 工作區** > 選取-「安全工作站監視」
1. 選取**Advanced settings**  >  **Data**  >  **Windows 事件記錄**檔
1. 在 **[從下列事件記錄檔收集事件**] 中 
1. 輸入：
   * ' Microsoft-Windows-AppLocker/EXE 和 DLL ' > 取消選取 **資訊**
   * 「Microsoft-Windows-AppLocker/MSI 和腳本」 > 取消選取 **資訊**
   * ' Microsoft-Windows-AppLocker/已封裝應用程式部署 ' > 取消選取 **資訊**
   * ' Microsoft-Windows-AppLocker/已封裝應用程式-執行 ' > 取消選取 **資訊**
1. 選取 [儲存]

應用程式記錄將可在您選取的 Log Analytics 工作區中使用。

## <a name="monitoring"></a>監視

* 瞭解如何[使用 Azure Sentinel](../../sentinel/tutorial-detect-threats-built-in.md)來偵測威脅
* [使用 Azure Sentinel 調查事件](../../sentinel/tutorial-investigate-cases.md)
* [在 Azure Sentinel 中設定自動化威脅回應](../../sentinel/tutorial-respond-threats-playbook.md)
* 瞭解如何審查您的 [曝光分數](/windows/security/threat-protection/microsoft-defender-atp/tvm-exposure-score)
* 檢查 [安全性建議](/windows/security/threat-protection/microsoft-defender-atp/tvm-security-recommendation)
* 管理安全性 [補救](/windows/security/threat-protection/microsoft-defender-atp/tvm-remediation)
* 管理 [端點偵測和回應](/windows/security/threat-protection/microsoft-defender-atp/overview-endpoint-detection-response)
* 使用 [Intune 設定檔監視](/intune/device-profile-monitor)來監視設定檔。

## <a name="next-steps"></a>後續步驟

* 深入瞭解 [Microsoft Intune](/intune/index)。
* 瞭解 [Azure AD](../index.yml)。
* 使用 [Microsoft Defender 進階威脅防護](/windows/security/threat-protection/microsoft-defender-atp/microsoft-defender-advanced-threat-protection)
* 探索 [Azure Sentinel](../../sentinel/index.yml)
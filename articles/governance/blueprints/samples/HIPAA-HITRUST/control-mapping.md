---
title: HIPAA HITRUST 藍圖範例控制項
description: HIPAA HITRUST 藍圖範例的控制項對應。 每個控制項都會對應至一或多個可協助評量的 Azure 原則。
ms.date: 08/03/2020
ms.topic: sample
ms.openlocfilehash: 10b771e3cfb18a28bd720332a26e13bb1d1f6022
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/14/2020
ms.locfileid: "88209418"
---
# <a name="control-mapping-of-the-hipaa-hitrust-blueprint-sample"></a>HIPAA HITRUST 藍圖範例的控制項對應

下列文章將詳細說明 Azure 藍圖 HIPAA HITRUST 藍圖範例與 HIPAA HITRUST 控制項的對應情形。 如需控制項的詳細資訊，請參閱 [HIPAA HITRUST](https://www.hhs.gov/hipaa/for-professionals/security/laws-regulations/index.html)。

以下是 **HIPAA HITRUST** 控制項的對應。 使用右側的導覽區可直接跳到特定的控制項對應。 許多對應的控制項都是以 [Azure 原則](../../../policy/overview.md)方案進行實作的。 若要檢閱完整方案，請在 Azure 入口網站中開啟 [原則]，然後選取 [定義] 頁面。 然後，找出並選取 **\[預覽\]：稽核 HIPAA HITRUST 控制項**內建原則計畫。

> [!IMPORTANT]
> 下列每個控制措施都與一或多個 [Azure 原則](../../../policy/overview.md)定義相關聯。 這些原則可協助您使用工具[存取合規性](../../../policy/how-to/get-compliance-data.md)；不過，控制措施和一或多個原則之間，通常不是 1：1 或完整對應。 因此，Azure 原則中的**符合規範**只是指原則本身，這不保證您符合控制措施所有需求的規範。 此外，合規性標準包含目前未由任何 Azure 原則定義解決的控制措施。 因此，Azure 原則中的合規性只是整體合規性狀態的部分觀點。 此合規性藍圖範例的控制措施與 Azure 原則定義之間的關聯，可能會隨著時間而改變。 若要檢視變更歷程記錄，請參閱 [GitHub 認可歷程記錄](https://github.com/MicrosoftDocs/azure-docs/commits/master/articles/governance/blueprints/samples/HIPAA-HITRUST/control-mapping.md) \(英文\)。

## <a name="control-against-malicious-code"></a>控制惡意程式碼

此藍圖指派了 [Azure 原則](../../../policy/overview.md)定義，以在「Azure 資訊安全中心」監視虛擬機器缺少的 Endpoint Protection，並在 Windows 虛擬機器上強制執行 Microsoft 反惡意程式碼軟體解決方案，協助您管理 Endpoint Protection，包括惡意程式碼防護。

- Azure 的 Microsoft Antimalware 應設定為自動更新保護簽章
- 在 Azure 資訊安全中心中監視缺少的 Endpoint Protection
- 應在虛擬機器擴展集上安裝端點保護解決方案
- 應在虛擬機器上啟用自適性應用程式控制


## <a name="management-of-removable-media"></a>卸載式媒體的管理

根據資料分類層級，組織會在使用媒體前進行註冊 (包括膝上型電腦)、對這類媒體的使用方式制定合理的限制，以及對包含涵蓋資訊的媒體提供適當的實體和邏輯保護層級 (包括加密)，直到正確地終結或清理為止。

- 應在 SQL 資料庫上啟用透明資料加密
- 應在虛擬機器上套用磁碟加密
- 未連結的磁碟應加密
- Data Lake Store 帳戶需要加密
- SQL 伺服器的 TDE 保護裝置應以您自己的金鑰加密
- SQL 受控執行個體的 TDE 保護裝置應以您自己的金鑰加密

## <a name="control-of-operational-software"></a>作業軟體的控制 

組織會在資訊系統 (括伺服器、工作站和膝上型電腦) 上識別未經授權的軟體、採用全部允許和拒絕例外狀況的原則來禁止在資訊系統上執行已知的未經授權軟體，以及定期檢閱和更新未經授權的軟體清單 (至少一年一次)。

- 您應在機器上修復安全性組態的弱點
- 應補救容器安全性設定中的弱點
- 應修復虛擬機器擴展集上安全性組態的弱點
- 應在虛擬機器上啟用自適性應用程式控制

## <a name="change-control-procedures"></a>變更控制程序

對虛擬機器所做的任何變更都會加以記錄並引發警示，因此能確保所有虛擬機器映像在任何時候都是完整的，並可透過電子方法 (例如入口網站或警示)，將變更或移動的結果及後續的映像完整性驗證，提供給企業擁有者和/或客戶。

- \[預覽\] 在 [系統稽核原則 - 詳細追蹤] 中顯示來自 Windows VM 設定的稽核結果
- \[預覽\]：顯示「系統稽核原則 - 詳細追蹤」中的 Windows VM 設定稽核結果

## <a name="control-of-technical-vulnerabilities"></a>技術弱點的控制 

所有系統和網路元件都有強化的設定標準。

- 弱點評定應於您的 SQL 伺服器上啟用
- SQL 受控執行個體上應啟用弱點評定
- \[預覽\] 應在虛擬機器上啟用弱點評定
- 弱點評量解決方案應修復弱點
- 您應在機器上修復安全性組態的弱點
- 應修復虛擬機器擴展集上安全性組態的弱點
- 應補救容器安全性設定中的弱點
- 應修復 SQL 資料庫的弱點
- \[預覽\]：Kubernetes Services 上應定義 Pod 安全性原則

## <a name="segregation-in-networks"></a>網路中的隔離

組織的安全性閘道 (例如防火牆) 會強制執行安全性原則，並且會設定為可篩選網域之間的流量、封鎖未經授權的存取，以及用來維護內部有線、內部無線和外部網路部分 (例如網際網路) 之間的隔離 (包括 DMZ)，並且強制執行每個網域的存取控制原則。

- 子網路應該與網路安全性群組建立關聯
- 虛擬機器應該連線到已核准的虛擬網路
- 虛擬機器應該與網路安全性群組建立關聯
- 服務匯流排應該使用虛擬網路服務端點
- App Service 應該使用虛擬網路服務端點
- SQL Server 應該使用虛擬網路服務端點
- 事件中樞應該使用虛擬網路服務端點
- Cosmos DB 應該使用虛擬網路服務端點
- Key Vault 應該使用虛擬網路服務端點
- 閘道子網路不應設定網路安全性群組
- 儲存體帳戶應該使用虛擬網路服務端點
- \[預覽\]：Container Registry 應該使用虛擬網路服務端點
- 應在內部對應虛擬機器中套用自適性網路強化建議

## <a name="network-connection-control"></a>網路連線控制

網路流量會根據組織的存取控制原則，透過每個網路存取點的防火牆和其他網路相關限制，或外部電信服務的受控介面來加以控制。

- 應啟用儲存體帳戶的安全傳輸
- 您的 API 應用程式應使用最新的 TLS 版本
- 您的 Web 應用程式應使用最新的 TLS 版本
- 您的函式應用程式應使用最新的 TLS 版本
- 函式應用程式應只可經由 HTTPS 存取
- Web 應用程式應只可經由 HTTPS 存取
- API 應用程式應只可經由 HTTPS 存取
- 應為 MySQL 資料庫伺服器啟用 [強制執行 SSL 連線]
- 應為 PostgreSQL 資料庫伺服器啟用 [強制執行 SSL 連線]
- 應該只允許對 Redis Cache 的安全連線
- 子網路應該與網路安全性群組建立關聯
- 應在 IaaS 上強化 Web 應用程式的 NSG 規則
- 應強化與虛擬機器互動的網際網路網路安全性群組規則
- 虛擬機器應該連線到已核准的虛擬網路
- 虛擬機器應該與網路安全性群組建立關聯

## <a name="network-controls"></a>網路控制

組將實體伺服器、應用程式或資料遷移至虛擬化伺服器時，組織會使用安全且加密的通道。

- Just-In-Time 網路存取控制應套用在虛擬機器上
- 應在內部對應虛擬機器中套用自適性網路強化建議
- 服務匯流排應該使用虛擬網路服務端點
- App Service 應該使用虛擬網路服務端點
- SQL Server 應該使用虛擬網路服務端點
- 事件中樞應該使用虛擬網路服務端點
- Cosmos DB 應該使用虛擬網路服務端點
- Key Vault 應該使用虛擬網路服務端點
- 稽核不受限制的儲存體帳戶網路存取
- 儲存體帳戶應該使用虛擬網路服務端點
- \[預覽\]：Container Registry 應該使用虛擬網路服務端點

## <a name="security-of-network-services"></a>網絡服務的安全性

網路服務提供者/管理員所提供的已同意服務會正式地受到管理及監視，以確保能安全地供人使用。

- \[預覽\]：應在 Windows 虛擬機器上安裝網路流量資料收集代理程式
- \[預覽\]：應在 Linux 虛擬機器上安裝網路流量資料收集代理程式且應該啟用網路監看員

## <a name="information-exchange-policies-and-procedures"></a>資訊交換原則和程序

組織會透過授權外部資訊系統上的個人，來限制如何使用組織控制的可攜式存放媒體。

- 確定 Web 應用程式已將 [用戶端憑證 (傳入用戶端憑證)] 設定為 [開啟]
- CORS 不應允許每項資源存取您的 Web 應用程式
- CORS 不應允許每項資源存取函式應用程式
- CORS 不應允許每項資源存取您的 API 應用程式
- 應關閉 Web 應用程式的遠端偵錯
- 函式應用程式的遠端偵錯應關閉
- 應關閉 API 應用程式的遠端偵錯

## <a name="on-line-transactions"></a>線上交易

組織會需要在交易中牽涉到的每一方之間使用加密，以及使用電子簽章。組織會確保交易詳細資料的儲存位置位於任何可公開存取的環境之外 (例如組織內部網路上現有的儲存平台上)，而且不會在可直接從網際網路存取的儲存媒體上保留及公開。使用受信任的授權單位時 (例如，基於發行和維護數位簽章及/或數位憑證的用途)，安全性會整合並內嵌於整個端對端憑證/簽章管理流程。

- 應啟用儲存體帳戶的安全傳輸
- 您的 API 應用程式應使用最新的 TLS 版本
- 您的 Web 應用程式應使用最新的 TLS 版本
- 您的函式應用程式應使用最新的 TLS 版本
- 函式應用程式應只可經由 HTTPS 存取
- Web 應用程式應只可經由 HTTPS 存取
- API 應用程式應只可經由 HTTPS 存取
- 應為 MySQL 資料庫伺服器啟用 [強制執行 SSL 連線]
- 應為 PostgreSQL 資料庫伺服器啟用 [強制執行 SSL 連線]
- 應該只允許對 Redis Cache 的安全連線
- \[預覽\]：部署必要條件，以稽核信任根未包含指定憑證的 Windows VM
- \[預覽\]：顯示信任根未包含指定憑證的 Windows VM 稽核結果

## <a name="user-password-management"></a>使用者密碼管理

在所有系統元件上進行傳輸和儲存時都會將密碼加密。

- \[預覽\]稽核密碼安全性設定不安全的 VM

## <a name="user-authentication-for-external-connections"></a>用於外部連線的使用者驗證

在組織網路的所有外部連線上都會實作強式驗證方法，例如多重要素、Radius 或 Kerberos (適用於特殊權限存取) 和 CHAP (用於加密撥號方法所用的認證)。

- 應在您訂用帳戶上具有擁有者權限的帳戶上啟用 MFA
- 應在您訂用帳戶上具有寫入權限的帳戶上啟用 MFA
- 應在您訂用帳戶上具有讀取權限的帳戶上啟用 MFA
- Just-In-Time 網路存取控制應套用在虛擬機器上

## <a name="user-identification-and-authentication"></a>使用者識別與授權

執行特殊權限功能 (例如系統管理) 的使用者會在執行這些特殊權限功能時使用不同的帳戶。多重要素驗證方法會根據組織原則 (例如遠端網路存取) 來使用。

- 應在您訂用帳戶上具有擁有者權限的帳戶上啟用 MFA
- 應在您訂用帳戶上具有寫入權限的帳戶上啟用 MFA
- 應在您訂用帳戶上具有讀取權限的帳戶上啟用 MFA
- 應針對您的訂用帳戶指定最多 3 位擁有者
- 應將一個以上的擁有者指派給您的訂用帳戶
- 部署必要條件，以稽核其中 Administrator 群組包含任何指定成員的 Windows VM
- 顯示其中 Administrators 群組包含了任一指定成員的 Windows VM 稽核結果
- 部署必要條件以稽核其中 Administrators 群組不包含任何指定成員的 Windows VM
- 從其中 Administrators 群組不包含任何指定成員的 Windows VM 顯示稽核結果
- 部署必要條件以稽核 Administrators 群組不只包含指定成員的 Windows VM
- 顯示其中 Administrators 群組不只包含指定成員的 Windows VM 稽核結果

## <a name="privilege-management"></a>權限管理

針對裝載虛擬化系統的系統，其管理功能或管理主控台的存取權會依據最低權限原則來有限制地提供給個人，並透過技術控制項提供支援。

- Just-In-Time 網路存取控制應套用在虛擬機器上
- 應關閉虛擬機器上的管理連接埠
- 應針對您的訂用帳戶指定最多 3 位擁有者
- 應將一個以上的擁有者指派給您的訂用帳戶
- 具有擁有者權限的外部帳戶應該從您的訂用帳戶中移除
- 具有擁有者權限的已取代帳戶應該從您的訂用帳戶中移除
- 稽核自訂 RBAC 規則的使用方式
- 應在 Kubernetes 服務上使用角色型存取控制 (RBAC)

## <a name="review-of-user-access-rights"></a>檢閱使用者存取權限

組織會維護一份記錄在案的資訊資產授權使用者清單。

- 稽核自訂 RBAC 規則的使用方式

## <a name="remote-diagnostic-and-configuration-port-protection"></a>遠端診斷和設定連接埠保護

安裝在電腦或網路系統上的連接埠、服務和類似的應用程式若不是商務功能的特定需求，則會停用或移除。

- Just-In-Time 網路存取控制應套用在虛擬機器上
- 應關閉虛擬機器上的管理連接埠
- 應關閉 Web 應用程式的遠端偵錯
- 函式應用程式的遠端偵錯應關閉
- 應關閉 API 應用程式的遠端偵錯
- 應在虛擬機器上啟用自適性應用程式控制

## <a name="audit-logging"></a>稽核記錄

已傳送和接收的訊息記錄都會加以維護，包括訊息的日期、時間、來源和目的地，但不包含其內容。當系統處於作用中狀態時，一律會提供稽核功能，並追蹤關鍵事件、成功/失敗的資料存取、系統安全性設定變更、特殊權限或公用程式的使用、產生的任何警示、保護系統 (例如 A/V 和 IDS) 的啟用和停用、識別和驗證機制的啟用和停用，以及系統層級物件的建立和刪除。

- 應在 Azure Data Lake Store 中啟用診斷記錄
- 應在 Logic Apps 中啟用診斷記錄 
- 應在 IoT 中樞內啟用診斷記錄 
- 應啟用 Batch 帳戶中的診斷記錄 
- 應在虛擬機器擴展集中啟用診斷記錄 
- 應啟用事件中樞內的診斷記錄 
- 應在搜尋服務中啟用診斷記錄 
- 應在 App Service 中啟用診斷記錄 
- 應在 Data Lake Analytics 中啟用診斷記錄 
- 應啟用 Key Vault 中的診斷記錄 
- 應在服務匯流排中啟用診斷記錄
- 應啟用 Azure 串流分析中的診斷記錄
- 應啟用 SQL 伺服器上的稽核
- 稽核診斷設定
- Azure 監視器應從所有區域收集活動記錄

## <a name="monitoring-system-use"></a>監視系統使用

在整個組織環境中部署的自動化系統會用來監視關鍵事件和異常活動，以及分析系統記錄，並且定期檢閱其結果。監視項目包含特殊權限的作業、授權的存取或未經授權的存取嘗試，包括嘗試存取已停用的帳戶，以及系統警示或失敗。

- Azure 監視器應從所有區域收集活動記錄
- Log Analytics 代理程式應該安裝在虛擬機器上
- Log Analytics 代理程式應該安裝在虛擬機器擴展集上
- \[預覽\]：部署必要條件，以稽核 Log Analytics 代理程式未如預期連線的 Windows VM
- \[預覽\]：顯示 Log Analytics 代理程式未如預期連線的 Windows VM 稽核結果
- Azure 監視器記錄設定檔應收集 [寫入]、[刪除] 和 [動作] 分類的記錄
- 您的訂用帳戶上應啟用 Log Analytics 監視代理程式的自動化佈建

## <a name="segregation-of-duties"></a>權責區分

權責區分是用來限制未經授權或無意間修改資訊和系統的風險。 若未經過授權或偵測，任何人都不可以存取、修改或使用資訊系統。 對於負責系統管理和存取控制的個人，其存取權會限制在每個使用者角色和責任所需的最低權限，而這些人員無法存取與這些控制項相關的稽核功能。

- 應在 Kubernetes 服務上使用角色型存取控制 (RBAC)
- 稽核自訂 RBAC 規則的使用方式
- \[預覽\]：部署必要條件，以稽核 [使用者權限指派] 中的 Windows VM 組態
- \[預覽\]：顯示「使用者權限指派」中的 Windows VM 設定稽核結果
- \[預覽\]：部署必要條件，以稽核「安全性選項 - 使用者帳戶控制」中的 Windows VM 設定
- \[預覽\]：顯示「安全性選項 - 使用者帳戶控制」中的 Windows VM 設定稽核結果
- 自訂訂用帳戶擁有者角色不應存在

## <a name="administrator-and-operator-logs"></a>系統管理員與操作員的記錄

組織會確保已啟用適當的記錄功能，以檢閱系統管理員活動；並且會定期檢閱系統管理員和操作員記錄。

- 特定系統管理作業應有活動記錄警示

## <a name="identification-of-risks-related-to-external-parties"></a>識別與外部合作對象相關的風險

加密組織與外部合作對象之間的遠端存取連線

- 應啟用儲存體帳戶的安全傳輸
- 函式應用程式應只可經由 HTTPS 存取
- Web 應用程式應只可經由 HTTPS 存取
- API 應用程式應只可經由 HTTPS 存取
- 應為 MySQL 資料庫伺服器啟用 [強制執行 SSL 連線]
- 應為 PostgreSQL 資料庫伺服器啟用 [強制執行 SSL 連線]
- 應該只允許對 Redis Cache 的安全連線

## <a name="business-continuity-and-risk-assessment"></a>商務持續性和風險評量

組織會識別其重要的商務程序，並將商務持續性的資訊安全管理需求與其他作業、人員配置、材料、傳輸及設備等的相關持續性需求整合。

- 稽核未設定災害復原的虛擬機器
- Key Vault 物件應該要可復原
- \[預覽\]：部署必要條件，以稽核「安全性選項 - 修復主控台」中的 Windows VM 設定
- \[預覽\] 在 [安全性選項 - 修復主控台] 中顯示來自 Windows VM 設定的稽核結果

## <a name="back-up"></a>備份

此藍圖會指派 Azure 原則定義，以便透過電子方式將組織的系統備份資訊轉移至替代儲存地點。 如需進行儲存體中繼資料的實體出貨，請考慮使用 Azure 資料箱。

- 應為 Azure SQL Database 啟用長期異地備援備份
- 應為適用於 MySQL 的 Azure 資料庫啟用異地備援備份
- 應為適用於 PostgreSQL 的 Azure 資料庫啟用異地備援備份
- 應為適用於 MariaDB 的 Azure 資料庫啟用異地備援備份
- 應該為虛擬機器啟用 Azure 備份

> [!NOTE]
> 特定 Azure 原則定義的可用性可能因 Azure Government 和其他國家雲端而異。 

## <a name="next-steps"></a>後續步驟

您已檢閱了 HIPAA HITRUST 藍圖範例的控制項對應。 接下來，請瀏覽下列文章，以深入了解概觀及部署此範例的方法：

> [!div class="next step action"]
> [HIPAA HITRUST 藍圖 - 概觀](./control-mapping.md)
> [HIPAA HITRUST 藍圖 - 部署步驟](./deploy.md)

有關藍圖及其使用方式的其他文件：

- 了解[藍圖生命週期](../../concepts/lifecycle.md)。
- 了解如何使用[靜態與動態參數](../../concepts/parameters.md)。
- 了解如何自訂[藍圖排序順序](../../concepts/sequencing-order.md)。
- 了解如何使用[藍圖資源鎖定](../../concepts/resource-locking.md)。
- 了解如何[更新現有的指派](../../how-to/update-existing-assignments.md)。

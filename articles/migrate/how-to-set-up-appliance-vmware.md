---
title: 設定適用于 VMware 的 Azure Migrate 設備
description: 瞭解如何設定 Azure Migrate 設備來評定和遷移 VMware Vm。
ms.topic: article
ms.date: 04/16/2020
ms.openlocfilehash: c9a9f1567f984fc5770b47d3998610cb69643360
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/26/2020
ms.locfileid: "88923601"
---
# <a name="set-up-an-appliance-for-vmware-vms"></a>為 VMware Vm 設定設備

遵循這篇文章來設定 Azure Migrate 設備以使用 [Azure Migrate：伺服器評估](migrate-services-overview.md#azure-migrate-server-assessment-tool) 工具進行評量，以及使用 [Azure Migrate：伺服器遷移](migrate-services-overview.md#azure-migrate-server-migration-tool) 工具進行無代理程式的遷移。

[Azure Migrate 設備](migrate-appliance.md)是 Azure Migrate 所使用的輕量設備：伺服器評量和伺服器遷移以探索內部部署 VMware vm、將 VM 中繼資料/效能資料傳送至 Azure，以及在無代理程式遷移期間複寫 VMware vm。

您可以使用幾種方法來部署設備：

- 使用下載的 OVA 範本，在 VMware VM 上進行設定。 這是本文中所述的方法。
- 使用 PowerShell 安裝程式指令碼在 VMware VM 或實體機器上進行設定。 如果您無法使用 OVA 範本設定 VM，或如果您是在 Azure Government 中，則應該使用[此方法](deploy-appliance-script.md)。

建立設備之後，您會檢查其是否可以連線到 Azure Migrate：伺服器評量、進行第一次設，以及向 Azure Migrate 專案註冊設備。


## <a name="appliance-deployment-ova"></a>設備部署 (OVA) 

若要使用 OVA 範本來設定設備，您可以：
- 在入口網站中提供設備名稱並產生 Azure Migrate 專案金鑰
- 下載 OVA 範本檔案，並將其匯入 vCenter Server。
- 建立設備，並確認其可以連線至 Azure Migrate 伺服器評估。
- 第一次設定設備，並使用 Azure Migrate 專案金鑰向 Azure Migrate 專案註冊。

### <a name="generate-the-azure-migrate-project-key"></a>產生 Azure Migrate 專案金鑰

1. 在 [移轉目標] > [伺服器] >  **[Azure Migrate：伺服器評估]** 中，選取 [探索]。
2. 在 [探索機器] > [機器是否已虛擬化?] 中，選取 [是，使用 VMware vSphere Hypervisor]。
3. 在 **1：產生 Azure Migrate 專案金鑰**中，為您將設定的 Azure Migrate 設備提供名稱，以便探索 VMware Vm。此名稱應為英數位元，且長度不超過14個字元。
1. 按一下 [ **產生金鑰** ] 以開始建立必要的 Azure 資源。 在資源建立期間，請勿關閉 [探索電腦] 頁面。
1. 成功建立 Azure 資源之後，就會產生 **Azure Migrate 的專案金鑰** 。
1. 複製金鑰，因為您需要它才能在其設定期間完成設備註冊。

### <a name="download-the-ova-template"></a>下載 OVA 範本
在 **2：下載 Azure Migrate 設備**上，選取。OVA 檔案，然後按一下 [ **下載**]。 


   ![探索電腦的選項](./media/tutorial-assess-vmware/servers-discover.png)


   ![產生金鑰的選取專案](./media/tutorial-assess-vmware/generate-key-vmware.png)

### <a name="verify-security"></a>確認安全性

請先確認 OVA 檔案是安全的，再進行部署。

1. 在存放下載檔案的目標電腦上，開啟系統管理員命令視窗。
2. 執行下列命令以產生 OVA 的雜湊：
    - ```C:\>CertUtil -HashFile <file_location> [Hashing Algorithm]```
    - 使用方式範例：```C:\>CertUtil -HashFile C:\AzureMigrate\AzureMigrate.ova SHA256```
3. 針對最新的設備版本，所產生的雜湊應符合這些 [設定](./tutorial-assess-vmware.md#verify-security)。



## <a name="create-the-appliance-vm"></a>建立設備 VM

匯入所下載的檔案，並建立 VM。

1. 在 vSphere 用戶端主控台中，**按一下 [** 檔案  >  **部署 OVF 範本**]。
![用於部署 OVF 範本的功能表命令](./media/tutorial-assess-vmware/deploy-ovf.png)

2. 在 [部署 OVF 範本精靈] > [來源] 中，指定 OVA 檔案的位置。
3. 在 [名稱]  和 [位置] 中，指定 VM 的易記名稱。 選取將裝載 VM 的庫存物件。
5. 在 [主機/叢集] 中，指定 VM 的執行所在主機或叢集。
6. 在**儲存體**中，指定 VM 的儲存目的地。
7. 在 [磁碟格式] 中，指定磁碟類型和大小。
8. 在 [ **網路對應**] 中，指定 VM 將連線的網路。 此網路必須能夠連線到網際網路，以將中繼資料傳送至 Azure Migrate 伺服器評量。
9. 檢閱並確認設定，然後按一下 [完成]****。


## <a name="verify-appliance-access-to-azure"></a>確認設備是否能存取 Azure

確定設備 VM 可以連線至[公用](migrate-appliance.md#public-cloud-urls)和[政府](migrate-appliance.md#government-cloud-urls)雲端的 Azure URL。


### <a name="configure-the-appliance"></a>設定設備

第一次設定設備。

> [!NOTE]
> 如果您使用 [PowerShell 指令碼](deploy-appliance-script.md)來設定設備 (而非使用下載的 OVA)，則此程序中的前兩個步驟將與之無關。

1. 在 vSphere 用戶端主控台中，以滑鼠右鍵按一下 VM，然後選取 [開啟主控台]。
2. 提供設備的語言、時區和密碼。
3. 在任何可連線至 VM 的機器上開啟瀏覽器，並開啟設備 Web 應用程式的 URL：**https://設備名稱或 IP 位址:44368**。

   或者，您也可以選取應用程式捷徑，從設備桌面開啟應用程式。
1. 接受 **授權條款**，並閱讀協力廠商資訊。
1. 在 [Web 應用程式] > [設定必要條件] 中，執行下列動作：
   - **連線能力**：應用程式會確認 VM 是否能夠存取網際網路。 如果 VM 使用 Proxy：
     - 按一下 [ **設定 proxy** ]，在表單 http://ProxyIPAddress 或 http://ProxyFQDN) 和接聽埠中指定 proxy 位址 (。
     - 如果 Proxy 需要驗證，請指定認證。
     - 僅支援 HTTP Proxy。
     - 如果您已新增 proxy 詳細資料或停用 proxy 及/或驗證，請按一下 [ **儲存** ] 以觸發連線檢查。
   - **時間同步**：設備上的時間應該與網際網路時間同步，探索才能正常運作。
   - **安裝更新**：設備會確保已安裝最新的更新。 檢查完成後，您可以按一下 [ **View 設備服務** ] 以查看設備上執行的元件狀態和版本。
   - **安裝 VDDK**：設備會檢查是否已安裝 VMware vSphere 虛擬磁碟開發套件 (VDDK)。 如果未安裝，請從 VMware 下載 VDDK 6.7，並將下載的 zip 內容解壓縮到設備上的指定位置（如 **安裝指示**中所述）。

     在遷移至 Azure 期間，Azure Migrate 伺服器移轉會使用 VDDK 來複寫機器。 
1. 如果您想要的話，您可以在設備設定期間隨時 **重新執行必要條件** ，以檢查設備是否符合所有必要條件。

### <a name="register-the-appliance-with-azure-migrate"></a>向 Azure Migrate 註冊設備

1. 貼上從入口網站複製的 **Azure Migrate 專案金鑰** 。 如果您沒有金鑰，請移至 [ **伺服器評量]> 探索> 管理現有的設備**]，選取您在產生金鑰時所提供的設備名稱，並複製對應的金鑰。
1. 按一下 [ **登入**]。 它會在新的瀏覽器索引標籤中開啟 Azure 登入提示。如果未出現，請確定您已在瀏覽器中停用快顯封鎖程式。
1. 在新的索引標籤上，使用您的 Azure 使用者名稱和密碼登入。
   
   不支援使用 PIN 登入。
3. 成功登入後，返回 web 應用程式。 
4. 如果用於記錄的 Azure 使用者帳戶具有在金鑰產生期間所建立之 Azure 資源的正確 [許可權](tutorial-prepare-vmware.md#prepare-azure) ，則會起始設備註冊。
1. 成功註冊設備之後，您可以按一下 [ **查看詳細資料**] 來查看註冊詳細資料。


## <a name="start-continuous-discovery"></a>開始連續探索

設備必須連線到 vCenter Server，才能探索 VM 的設定與效能資料。

1. 在**步驟1：提供 vCenter Server 認證**中，按一下 [**新增認證**] 以指定認證的易記名稱、為設備將用來在 VCenter Server 實例上探索 vm 的 vCenter Server**帳戶新增使用者名稱和****密碼**。
    - 您應該已在[上一個教學課程](tutorial-prepare-vmware.md#set-up-permissions-for-assessment)中設定具有必要權限的帳戶。
    - 如果您想要將探索範圍限定成特定的 VMware 物件 (vCenter Server 資料中心、叢集、叢集的資料夾、主機、主機的資料夾或個別的 VM)，請參閱本文的[指示](set-discovery-scope.md)來限制 Azure Migrate 所使用的帳戶。
1. 在 [ **步驟2：提供 vCenter Server 詳細資料**] 中，按一下 [ **新增探索來源** ]，從下拉式清單中選取認證的易記名稱，並指定 vCenter Server 實例的 **IP 位址/FQDN** 。 您可以保留預設的 **埠** (443) 或指定 vCenter Server 接聽的自訂埠，然後按一下 [ **儲存**]。
1. 當您按一下 [儲存] 時，應用裝置會嘗試使用提供的認證驗證 vCenter Server 的連線，並針對 vCenter Server 的 IP 位址/FQDN 顯示資料表中的 **驗證狀態** 。
1. 在開始探索之前，您可以重新 **驗證** 連線以 vCenter Server 任何時間。
1. 在 **步驟3：提供 VM 認證以探索已安裝的應用程式，以及執行無代理程式**相依性對應，按一下 [ **新增認證**]，並指定提供認證的作業系統、 **認證的易記** 名稱以及使用者名稱和 **密碼**。 然後按一下 [ **儲存**]。

    - 如果您已建立可用於[應用程式探索功能](how-to-discover-applications.md)，或[無代理程式相依性分析功能](how-to-create-group-machine-dependencies-agentless.md)的認證，您可以選擇性地在此新增認證。
    - 如果您不想要使用這些功能，您可以按一下滑杆來略過該步驟。 您可以稍後隨時反轉意圖。
    - 檢查 [應用程式探索](migrate-support-matrix-vmware.md#application-discovery-requirements)所需的認證，或 [無代理](migrate-support-matrix-vmware.md#dependency-analysis-requirements-agentless)程式相依性分析所需的認證。

5. 按一下 [ **開始探索**] 以啟動 VM 探索。 成功起始探索之後，您可以針對資料表中的 vCenter Server IP 位址/FQDN 檢查探索狀態。

探索的運作方式如下：
- 所探索到的 VM 中繼資料會在大約 15 分鐘後出現在入口網站中。
- 探索已安裝的應用程式、角色和功能需要花一點時間。 時間長短取決於所探索的 VM 數目。 針對 500 部 VM，應用程式清查大約需要一小時才會出現在 Azure Migrate 入口網站中。

## <a name="next-steps"></a>後續步驟

請參閱 [VMware 評定](tutorial-assess-vmware.md) 和 [無代理程式遷移](tutorial-migrate-vmware.md)的教學課程。

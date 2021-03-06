---
title: 使用 PowerShell 將 Windows Server 備份至 Azure
description: 在本文中，您將瞭解如何使用 PowerShell 來設定 Windows Server 或 Windows 用戶端上的 Azure 備份，以及管理備份和復原。
ms.topic: conceptual
ms.date: 12/2/2019
ms.openlocfilehash: 47c8fc39626d3bca3355c1d1e46f1634327748a8
ms.sourcegitcommit: c6b9a46404120ae44c9f3468df14403bcd6686c1
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/26/2020
ms.locfileid: "88892366"
---
# <a name="deploy-and-manage-backup-to-azure-for-windows-serverwindows-client-using-powershell"></a>使用 PowerShell 部署和管理 Windows Server/Windows 用戶端的 Azure 備份

本文說明如何使用 PowerShell，在 Windows Server 或 Windows 用戶端上設定 Azure 備份，以及管理備份和復原。

## <a name="install-azure-powershell"></a>安裝 Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

若要開始使用，請[安裝最新的 PowerShell 版本](/powershell/azure/install-az-ps)。

## <a name="create-a-recovery-services-vault"></a>建立復原服務保存庫

下列步驟將引導您完成建立復原服務保存庫。 復原服務保存庫不同於備份保存庫。

1. 如果您是第一次使用 Azure 備份，就必須使用 **>register-azresourceprovider** 指令程式，向您的訂用帳戶註冊 Azure 復原服務提供者。

    ```powershell
    Register-AzResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"
    ```

2. 復原服務保存庫是 Azure Resource Manager 資源，因此您必須將它放在資源群組內。 您可以使用現有的資源群組，或建立一個新的群組。 建立新的資源群組時，請指定資源群組的名稱和位置。  

    ```powershell
    New-AzResourceGroup –Name "test-rg" –Location "WestUS"
    ```

3. 使用 **New-AzRecoveryServicesVault** Cmdlet 建立新的保存庫。 請務必為保存庫指定與用於資源群組相同的位置。

    ```powershell
    New-AzRecoveryServicesVault -Name "testvault" -ResourceGroupName " test-rg" -Location "WestUS"
    ```

4. 指定要使用的儲存空間備援類型。 您可以使用 [本機冗余儲存體 (LRS) ](../storage/common/storage-redundancy.md) 或 [地理位置多餘的儲存體 (GRS) ](../storage/common/storage-redundancy.md)。 下列範例顯示*testVault*的 **-BackupStorageRedundancy**選項設定為**異地備援**。

   > [!TIP]
   > 許多 Azure 備份 Cmdlet 都需要將復原服務保存庫物件當做輸入。 基於這個理由，將備份復原服務保存庫物件儲存在變數中是很方便的。
   >
   >

    ```powershell
    $Vault1 = Get-AzRecoveryServicesVault –Name "testVault"
    Set-AzRecoveryServicesBackupProperties -Vault $Vault1 -BackupStorageRedundancy GeoRedundant
    ```

## <a name="view-the-vaults-in-a-subscription"></a>在訂用帳戶中檢視保存庫

使用 **Get-AzRecoveryServicesVault** 來檢視目前訂用帳戶中所有保存庫的清單。 您可以使用此命令來檢查是否已建立新的保存庫，或查看訂用帳戶中有哪些保存庫可用。

執行命令 **Get-AzRecoveryServicesVault**，就會列出訂用帳戶中的所有保存庫。

```powershell
Get-AzRecoveryServicesVault
```

```Output
Name              : Contoso-vault
ID                : /subscriptions/1234
Type              : Microsoft.RecoveryServices/vaults
Location          : WestUS
ResourceGroupName : Contoso-docs-rg
SubscriptionId    : 1234-567f-8910-abc
Properties        : Microsoft.Azure.Commands.RecoveryServices.ARSVaultProperties
```

[!INCLUDE [backup-upgrade-mars-agent.md](../../includes/backup-upgrade-mars-agent.md)]

## <a name="installing-the-azure-backup-agent"></a>安裝 Azure 備份代理程式

在安裝 Azure 備份代理程式之前，您必須在 Windows Server 上下載並提供安裝程式。 您可以從 [Microsoft 下載中心](https://aka.ms/azurebackup_agent) 或從復原服務保存庫的 [儀表板] 頁面取得最新版的安裝程式。 請將安裝程式儲存至容易存取的位置，例如 `C:\Downloads\*`。

或者，使用 PowerShell 來取得下載程式：

 ```powershell
 $MarsAURL = 'https://aka.ms/Azurebackup_Agent'
 $WC = New-Object System.Net.WebClient
 $WC.DownloadFile($MarsAURL,'C:\downloads\MARSAgentInstaller.EXE')
 C:\Downloads\MARSAgentInstaller.EXE /q
 ```

若要安裝代理程式，請在已提升權限的 PowerShell 主控台中執行下列命令：

```powershell
MARSAgentInstaller.exe /q
```

這會以所有預設選項安裝代理程式。 安裝作業會在背景中進行幾分鐘。 如果您未指定 */nu* 選項，則會在安裝結束時開啟 **Windows Update** 視窗，以檢查是否有任何更新。 安裝之後，代理程式會顯示在已安裝的程式清單中。

若要查看已安裝的程式清單，請移至 [控制台] > [程式] > [程式和功能]。

![已安裝代理程式](./media/backup-client-automation/installed-agent-listing.png)

### <a name="installation-options"></a>安裝選項

若要查看所有可透過命令列執行的選項，請使用下列命令：

```powershell
MARSAgentInstaller.exe /?
```

可用的選項包括：

| 選項 | 詳細資料 | 預設 |
| --- | --- | --- |
| /q |無訊息安裝 |- |
| /p:"location" |Azure 備份代理程式的安裝資料夾路徑。 |C:\Program Files\Microsoft Azure Recovery Services Agent |
| /s:"location" |Azure 備份代理程式的快取資料夾路徑。 |C:\Program Files\Microsoft Azure Recovery Services Agent\Scratch |
| /m |選擇加入 Microsoft Update |- |
| /nu |安裝完成之後，請勿檢查更新 |- |
| /d |解除安裝 Microsoft Azure 復原服務代理程式 |- |
| /ph |Proxy 主機位址 |- |
| /po |Proxy 主機連接埠號碼 |- |
| /pu |Proxy 主機使用者名稱 |- |
| /pw |Proxy 密碼 |- |

## <a name="registering-windows-server-or-windows-client-machine-to-a-recovery-services-vault"></a>將 Windows Server 或 Windows 用戶端電腦註冊到復原服務保存庫

建立復原服務保存庫之後，請下載最新版本的代理程式和保存庫認證，並將它們儲存在方便的位置 (如 C:\Downloads)。

```powershell
$CredsPath = "C:\downloads"
$CredsFilename = Get-AzRecoveryServicesVaultSettingsFile -Backup -Vault $Vault1 -Path $CredsPath
```

### <a name="registering-using-the-ps-az-module"></a>使用 PS Az 模組註冊

> [!NOTE]
> 在 Az 3.5.0 版中，已修正產生保存庫憑證的錯誤。 使用 Az 3.5.0 版或更高版本下載保存庫憑證。

在最新的 PowerShell Az 模組中，因為基礎平臺的限制，所以下載保存庫認證需要自我簽署憑證。 下列範例示範如何提供自我簽署憑證，並下載保存庫認證。

```powershell
$dt = $(Get-Date).ToString("M-d-yyyy")
$cert = New-SelfSignedCertificate -CertStoreLocation Cert:\CurrentUser\My -FriendlyName 'test-vaultcredentials' -subject "Windows Azure Tools" -KeyExportPolicy Exportable -NotAfter $(Get-Date).AddHours(48) -NotBefore $(Get-Date).AddHours(-24) -KeyProtection None -KeyUsage None -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.2") -Provider "Microsoft Enhanced Cryptographic Provider v1.0"
$certficate = [convert]::ToBase64String($cert.Export([System.Security.Cryptography.X509Certificates.X509ContentType]::Pfx))
$CredsFilename = Get-AzRecoveryServicesVaultSettingsFile -Backup -Vault $Vault -Path $CredsPath -Certificate $certficate
```

在 Windows Server 或 Windows 用戶端電腦上，執行 [Start-OBRegistration](/powershell/module/msonlinebackup/start-obregistration) Cmdlet 向保存庫註冊電腦。
此程式和其他用於備份的 Cmdlet 來自于 MSONLINE 模組，這是 MARS AgentInstaller 在安裝過程中新增的模組。

代理程式安裝程式不會更新 $Env:P SModulePath 變數。 這表示模組自動載入會失敗。 若要解決此問題，您可以執行下列命令：

```powershell
$Env:PSModulePath += ';C:\Program Files\Microsoft Azure Recovery Services Agent\bin\Modules'
```

或者，您可以在指令碼中手動載入模組，如下所示：

```powershell
Import-Module -Name 'C:\Program Files\Microsoft Azure Recovery Services Agent\bin\Modules\MSOnlineBackup'
```

在載入 Online Backup Cmdlet 後，您必須註冊保存庫認證：

```powershell
Start-OBRegistration -VaultCredentials $CredsFilename.FilePath -Confirm:$false
```

```Output
CertThumbprint      : 7a2ef2caa2e74b6ed1222a5e89288ddad438df2
SubscriptionID      : ef4ab577-c2c0-43e4-af80-af49f485f3d1
ServiceResourceName : testvault
Region              : WestUS
Machine registration succeeded.
```

> [!IMPORTANT]
> 請勿使用相對路徑來指定保存庫認證檔。 您必須提供絕對路徑做為 Cmdlet 的輸入。
>
>

## <a name="networking-settings"></a>網路設定

若 Windows 電腦是透過 Proxy 伺服器連線到網際網路，您也可以提供 Proxy 設定給代理程式。 在此範例中，沒有 proxy 伺服器，因此會明確地清除任何 proxy 相關資訊。

您也可以針對給定的一組當週天數，使用 [`work hour bandwidth`] 和 [`non-work hour bandwidth`] 的選項來控制頻寬使用情形。

使用 [Set-OBMachineSetting](/powershell/module/msonlinebackup/set-obmachinesetting) Cmdlet 即可設定 Proxy 和頻寬的詳細資料：

```powershell
Set-OBMachineSetting -NoProxy
```

```Output
Server properties updated successfully.
```

```powershell
Set-OBMachineSetting -NoThrottle
```

```Output
Server properties updated successfully.
```

## <a name="encryption-settings"></a>加密設定

傳送至 Azure 備份的備份資料會進行加密來保護資料的機密性。 加密複雜密碼是在還原時用來解密資料的「密碼」。

您必須在 Azure 入口網站 [復原服務保存庫] 區段中，選取 [設定] > [內容] > [安全性 PIN 碼] 底下的 [產生]，以產生安全性 Pin 碼。

>[!NOTE]
> 安全性 PIN 碼只能透過 Azure 入口網站產生。

接著，在命令中使用此 PIN 碼作為 `generatedPIN`：

```powershell
$PassPhrase = ConvertTo-SecureString -String "Complex!123_STRING" -AsPlainText -Force
Set-OBMachineSetting -EncryptionPassPhrase $PassPhrase -SecurityPin "<generatedPIN>"
```

```Output
Server properties updated successfully
```

> [!IMPORTANT]
> 設定之後，請確保複雜密碼資訊安全且安全。 若沒有此複雜密碼，您將無法從 Azure 還原資料。
>
>

## <a name="back-up-files-and-folders"></a>備份檔案和資料夾

Windows Server 和用戶端的所有 Azure 備份都是由原則來掌管。 原則包含三個部分：

1. **備份排程** ，指定何時進行備份並與服務同步。
2. **保留排程** 可指定要在 Azure 中保留復原點多久時間。
3. **檔案包含/排除規格** ，指出要備份的項目。

本文件中要說明如何將備份自動化，因此我們假設還未設定任何選項。 一開始，請先使用 [New-OBPolicy](/powershell/module/msonlinebackup/new-obpolicy) Cmdlet 建立新的備份原則。

```powershell
$NewPolicy = New-OBPolicy
```

目前，原則是空的，而且需要其他 Cmdlet 來定義要包含或排除的專案、執行備份的時間，以及將儲存備份的位置。

### <a name="configuring-the-backup-schedule"></a>設定備份排程

原則三個部分的第一個部分是備份排程，這是使用 [New-OBSchedule](/powershell/module/msonlinebackup/new-obschedule) Cmdlet 建立的。 備份排程會定義何時需要進行備份。 建立排程時，您需要指定兩個輸入參數：

* **星期幾** 要執行備份。 您可以只選一天或選擇一週的每天都執行備份工作，或任意選取要一週的哪幾天。
* **時段** 。 您可以定義多達一天三個不同時段來觸發備份。

例如，您可以設定每週六和日下午 4 點執行備份原則。

```powershell
$Schedule = New-OBSchedule -DaysOfWeek Saturday, Sunday -TimesOfDay 16:00
```

備份排程需要與原則建立關聯，您可以使用 [Set-OBSchedule](/powershell/module/msonlinebackup/set-obschedule) Cmdlet 來達成。

```powershell
Set-OBSchedule -Policy $NewPolicy -Schedule $Schedule
```

```Output
BackupSchedule : 4:00 PM Saturday, Sunday, Every 1 week(s) DsList : PolicyName : RetentionPolicy : State : New PolicyState : Valid
```

### <a name="configuring-a-retention-policy"></a>設定保留原則

保留原則會定義所建立備份工作的復原點保留時間長度。 使用 [OBRetentionPolicy 指令程式](/powershell/module/msonlinebackup/new-obretentionpolicy) 建立新的保留原則時，您可以指定備份復原點將保留 Azure 備份的天數。 以下範例將保留原則設定為七天。

```powershell
$RetentionPolicy = New-OBRetentionPolicy -RetentionDays 7
```

保留原則必須與主要原則建立關聯，方法為使用 Cmdlet [Set-OBRetentionPolicy](/powershell/module/msonlinebackup/set-obretentionpolicy)：

```powershell
Set-OBRetentionPolicy -Policy $NewPolicy -RetentionPolicy $RetentionPolicy
```

```Output
BackupSchedule  : 4:00 PM
                  Saturday, Sunday,
                  Every 1 week(s)
DsList          :
PolicyName      :
RetentionPolicy : Retention Days : 7

                  WeeklyLTRSchedule :
                  Weekly schedule is not set

                  MonthlyLTRSchedule :
                  Monthly schedule is not set

                  YearlyLTRSchedule :
                  Yearly schedule is not set

State           : New
PolicyState     : Valid
```

### <a name="including-and-excluding-files-to-be-backed-up"></a>包含及排除要備份的檔案

`OBFileSpec` 物件會定義備份中要包含與排除的檔案。 此組規則可劃分出電腦上要保護的檔案和資料夾。 您可以設定所需數量的檔案包含或排除規則，並建立其與原則的關聯。 建立新的 OBFileSpec 物件時，您可以：

* 指定要包含的檔案和資料夾
* 指定要排除的檔案和資料夾
* 指定要遞迴備份資料夾中的資料，或是否僅備份指定資料夾中最上層的檔案。

後者可利用在 New-OBFileSpec 命令中使用 -NonRecursive 旗標來達成。

在下例中，我們會備份磁碟區 C: 和 D:，並排除 Windows 資料夾和任何暫存資料夾中的作業系統二進位檔。 若要這樣做，我們將使用 [New-OBFileSpec](/powershell/module/msonlinebackup/new-obfilespec) Cmdlet 建立二個檔案規格：一個用於包含，另一個用於排除。 一旦建立檔案規格之後，再使用 [Add-OBFileSpec](/powershell/module/msonlinebackup/add-obfilespec) Cmdlet 建立與原則的關聯。

```powershell
$Inclusions = New-OBFileSpec -FileSpec @("C:\", "D:\")
$Exclusions = New-OBFileSpec -FileSpec @("C:\windows", "C:\temp") -Exclude
Add-OBFileSpec -Policy $NewPolicy -FileSpec $Inclusions
```

```Output
BackupSchedule  : 4:00 PM
                  Saturday, Sunday,
                  Every 1 week(s)
DsList          : {DataSource
                  DatasourceId:0
                  Name:C:\
                  FileSpec:FileSpec
                  FileSpec:C:\
                  IsExclude:False
                  IsRecursive:True

                  , DataSource
                  DatasourceId:0
                  Name:D:\
                  FileSpec:FileSpec
                  FileSpec:D:\
                  IsExclude:False
                  IsRecursive:True

                  }
PolicyName      :
RetentionPolicy : Retention Days : 7

                  WeeklyLTRSchedule :
                  Weekly schedule is not set

                  MonthlyLTRSchedule :
                  Monthly schedule is not set

                  YearlyLTRSchedule :
                  Yearly schedule is not set

State           : New
PolicyState     : Valid
```

```powershell
Add-OBFileSpec -Policy $NewPolicy -FileSpec $Exclusions
```

```Output
BackupSchedule  : 4:00 PM
                  Saturday, Sunday,
                  Every 1 week(s)
DsList          : {DataSource
                  DatasourceId:0
                  Name:C:\
                  FileSpec:FileSpec
                  FileSpec:C:\
                  IsExclude:False
                  IsRecursive:True
                  ,FileSpec
                  FileSpec:C:\windows
                  IsExclude:True
                  IsRecursive:True
                  ,FileSpec
                  FileSpec:C:\temp
                  IsExclude:True
                  IsRecursive:True

                  , DataSource
                  DatasourceId:0
                  Name:D:\
                  FileSpec:FileSpec
                  FileSpec:D:\
                  IsExclude:False
                  IsRecursive:True

                  }
PolicyName      :
RetentionPolicy : Retention Days : 7

                  WeeklyLTRSchedule :
                  Weekly schedule is not set

                  MonthlyLTRSchedule :
                  Monthly schedule is not set

                  YearlyLTRSchedule :
                  Yearly schedule is not set

State           : New
PolicyState     : Valid
```

### <a name="applying-the-policy"></a>套用原則

現在原則物件已完成，且具有關聯的備份排程、保留原則及包含/排除的檔案清單。 此原則現在已經過認可，適合用於 Azure 備份。 套用新建立的原則之前，請使用 [Remove-OBPolicy](/powershell/module/msonlinebackup/remove-obpolicy) Cmdlet 確認沒有現有的備份原則與伺服器相關聯。 移除原則時，系統會提示確認。 若要略過確認，請使用 `-Confirm:$false` 旗標搭配 Cmdlet。

```powershell
Get-OBPolicy | Remove-OBPolicy
```

```Output
Microsoft Azure Backup Are you sure you want to remove this backup policy? This will delete all the backed up data. [Y] Yes [A] Yes to All [N] No [L] No to All [S] Suspend [?] Help (default is "Y"):
```

若要認可原則物件已完成，請使用 [Set-OBPolicy](/powershell/module/msonlinebackup/set-obpolicy) Cmdlet。 系統將提示您進行確認。 若要略過確認，請使用 `-Confirm:$false` 旗標搭配 Cmdlet。

```powershell
Set-OBPolicy -Policy $NewPolicy
```

```Output
Microsoft Azure Backup Do you want to save this backup policy ? [Y] Yes [A] Yes to All [N] No [L] No to All [S] Suspend [?] Help (default is "Y"):
BackupSchedule : 4:00 PM Saturday, Sunday, Every 1 week(s)
DsList : {DataSource
         DatasourceId:4508156004108672185
         Name:C:\
         FileSpec:FileSpec
         FileSpec:C:\
         IsExclude:False
         IsRecursive:True,

         FileSpec
         FileSpec:C:\windows
         IsExclude:True
         IsRecursive:True,

         FileSpec
         FileSpec:C:\temp
         IsExclude:True
         IsRecursive:True,

         DataSource
         DatasourceId:4508156005178868542
         Name:D:\
         FileSpec:FileSpec
         FileSpec:D:\
         IsExclude:False
         IsRecursive:True
    }
PolicyName : c2eb6568-8a06-49f4-a20e-3019ae411bac
RetentionPolicy : Retention Days : 7
              WeeklyLTRSchedule :
              Weekly schedule is not set

              MonthlyLTRSchedule :
              Monthly schedule is not set

              YearlyLTRSchedule :
              Yearly schedule is not set
State : Existing PolicyState : Valid
```

您可以使用 [Get-OBPolicy](/powershell/module/msonlinebackup/get-obpolicy) Cmdlet 來檢視現有備份原則的詳細資料。 您可以使用 [Get-OBSchedule](/powershell/module/msonlinebackup/get-obschedule) Cmdlet (適用於備份排程) 和 [Get-OBRetentionPolicy](/powershell/module/msonlinebackup/get-obretentionpolicy) Cmdlet (適用於保留原則) 進一步向下鑽研

```powershell
Get-OBPolicy | Get-OBSchedule
```

```Output
SchedulePolicyName : 71944081-9950-4f7e-841d-32f0a0a1359a
ScheduleRunDays : {Saturday, Sunday}
ScheduleRunTimes : {16:00:00}
State : Existing
```

```powershell
Get-OBPolicy | Get-OBRetentionPolicy
```

```Output
RetentionDays : 7
RetentionPolicyName : ca3574ec-8331-46fd-a605-c01743a5265e
State : Existing
```

```powershell
Get-OBPolicy | Get-OBFileSpec
```

```Output
FileName : *
FilePath : \?\Volume{b835d359-a1dd-11e2-be72-2016d8d89f0f}\
FileSpec : D:\
IsExclude : False
IsRecursive : True

FileName : *
FilePath : \?\Volume{cdd41007-a22f-11e2-be6c-806e6f6e6963}\
FileSpec : C:\
IsExclude : False
IsRecursive : True

FileName : *
FilePath : \?\Volume{cdd41007-a22f-11e2-be6c-806e6f6e6963}\windows
FileSpec : C:\windows
IsExclude : True
IsRecursive : True

FileName : *
FilePath : \?\Volume{cdd41007-a22f-11e2-be6c-806e6f6e6963}\temp
FileSpec : C:\temp
IsExclude : True
IsRecursive : True
```

### <a name="performing-an-on-demand-backup"></a>執行隨選備份

設定備份原則之後，即會根據排程進行備份。 您也可以使用 [Start-OBBackup](/powershell/module/msonlinebackup/start-obbackup) Cmdlet 來觸發隨選備份：

```powershell
Get-OBPolicy | Start-OBBackup
```

```Output
Initializing
Taking snapshot of volumes...
Preparing storage...
Generating backup metadata information and preparing the metadata VHD...
Data transfer is in progress. It might take longer since it is the first backup and all data needs to be transferred...
Data transfer completed and all backed up data is in the cloud. Verifying data integrity...
Data transfer completed
In progress...
Job completed.
The backup operation completed successfully.
```

## <a name="back-up-windows-server-system-state-in-mars-agent"></a>在 MARS 代理程式中備份 Windows Server 系統狀態

本節涵蓋在 MARS 代理程式中設定系統狀態的 PowerShell 命令

### <a name="schedule"></a>排程

```powershell
$sched = New-OBSchedule -DaysOfWeek Sunday,Monday,Tuesday,Wednesday,Thursday,Friday,Saturday -TimesOfDay 2:00
```

### <a name="retention"></a>保留

```powershell
$rtn = New-OBRetentionPolicy -RetentionDays 32 -RetentionWeeklyPolicy -RetentionWeeks 13 -WeekDaysOfWeek Sunday -WeekTimesOfDay 2:00  -RetentionMonthlyPolicy -RetentionMonths 13 -MonthDaysOfMonth 1 -MonthTimesOfDay 2:00
```

### <a name="configuring-schedule-and-retention"></a>設定排程和保留

```powershell
New-OBPolicy | Add-OBSystemState |  Set-OBRetentionPolicy -RetentionPolicy $rtn | Set-OBSchedule -Schedule $sched | Set-OBSystemStatePolicy
 ```

### <a name="verifying-the-policy"></a>驗證原則

```powershell
Get-OBSystemStatePolicy
 ```

## <a name="restore-data-from-azure-backup"></a>從 Azure 備份還原資料

本節將引導您逐步完成自動化從 Azure 備份復原資料。 此工作涉及下列步驟：

1. 挑選來源磁碟區
2. 選擇要還原的備份點
3. 指定要還原的項目
4. 觸發還原程序

### <a name="picking-the-source-volume"></a>挑選來源磁碟區

若要從 Azure 備份還原專案，您必須先識別專案的來源。 我們在 Windows Server 或 Windows 用戶端的內容中執行命令，則已經識別電腦。 找出來源的下一步是識別包含它的磁碟區。 執行 [Get-OBRecoverableSource](/powershell/module/msonlinebackup/get-obrecoverablesource) Cmdlet 可以擷取一份這部電腦所備份磁碟區或來源清單。 此命令會傳回這部伺服器/用戶端備份的所有來源陣列。

```powershell
$Source = Get-OBRecoverableSource
$Source
```

```Output
FriendlyName : C:\
RecoverySourceName : C:\
ServerName : myserver.microsoft.com

FriendlyName : D:\
RecoverySourceName : D:\
ServerName : myserver.microsoft.com
```

### <a name="choosing-a-backup-point-from-which-to-restore"></a>選擇要從中還原的備份點

您可以執行 [Get-OBRecoverableItem](/powershell/module/msonlinebackup/get-obrecoverableitem) Cmdlet 並搭配適當參數來擷取備份點清單。 在我們範例中，我們將選擇來源磁碟區 *C:* 的最新備份點，並將其用於還原特定檔案。

```powershell
$Rps = Get-OBRecoverableItem $Source[0]
$Rps
```

```Output

IsDir                : False
ItemNameFriendly     : C:\
ItemNameGuid         : \\?\Volume{297cbf7a-0000-0000-0000-401f00000000}\
LocalMountPoint      : C:\
MountPointName       : C:\
Name                 : C:\
PointInTime          : 10/17/2019 7:52:13 PM
ServerName           : myserver.microsoft.com
ItemSize             :
ItemLastModifiedTime :

IsDir                : False
ItemNameFriendly     : C:\
ItemNameGuid         : \\?\Volume{297cbf7a-0000-0000-0000-401f00000000}\
LocalMountPoint      : C:\
MountPointName       : C:\
Name                 : C:\
PointInTime          : 10/16/2019 7:00:19 PM
ServerName           : myserver.microsoft.com
ItemSize             :
ItemLastModifiedTime :
```

物件 `$Rps` 是備份點陣列。 第一個元素是最新備份點，且第 N 個元素是最舊的備份點。 為了選擇最新的時間點，我們將使用 `$Rps[0]` 。

### <a name="specifying-an-item-to-restore"></a>指定要還原的項目

若要還原特定檔案，請指定相對於根磁碟區的檔案名稱。 例如，若要擷取 C:\Test\Cat.job，請執行下列命令。

```powershell
$Item = New-OBRecoverableItem $Rps[0] "Test\cat.jpg" $FALSE
$Item
```

```Output
IsDir                : False
ItemNameFriendly     : C:\Test\cat.jpg
ItemNameGuid         :
LocalMountPoint      : C:\
MountPointName       : C:\
Name                 : cat.jpg
PointInTime          : 10/17/2019 7:52:13 PM
ServerName           : myserver.microsoft.com
ItemSize             :
ItemLastModifiedTime : 21-Jun-14 6:43:02 AM

```

### <a name="triggering-the-restore-process"></a>觸發還原程序

為了觸發還原程序，我們首先需要指定復原選項。 使用 [New-OBRecoveryOption](/powershell/module/msonlinebackup/new-obrecoveryoption) Cmdlet 可以完成這項工作。 在此範例中，我們假設我們想要將檔案還原至 *C：\temp*。我們也假設我們想要跳過已存在於目的資料夾 *C：\temp*上的檔案。若要建立這類復原選項，請使用下列命令：

```powershell
$RecoveryOption = New-OBRecoveryOption -DestinationPath "C:\temp" -OverwriteType Skip
```

現在，從 `Get-OBRecoverableItem` Cmdlet 的輸出，在選取的 `$Item` 上使用 [Start-OBRecovery](/powershell/module/msonlinebackup/start-obrecovery) 命令來觸發還原程序：

```powershell
Start-OBRecovery -RecoverableItem $Item -RecoveryOption $RecoveryOption
```

```Output
Estimating size of backup items...
Estimating size of backup items...
Estimating size of backup items...
Estimating size of backup items...
Job completed.
The recovery operation completed successfully.
```

## <a name="uninstalling-the-azure-backup-agent"></a>解除安裝 Azure 備份代理程式

使用下列命令即可解除安裝 Azure 備份代理程式：

```powershell
.\MARSAgentInstaller.exe /d /q
```

若要從電腦解除安裝代理程式二進位檔，請考量下列後果：

* 這會從電腦移除檔案篩選器，並停止追蹤變更。
* 會從電腦移除所有原則資訊，但服務中會繼續保存這些原則資訊。
* 會移除所有備份排程，且不再進行任何備份。

不過，儲存在 Azure 中的資料會保持不變，並會根據您所設定的保留原則進行保留。 較舊的時間點會自動過時。

## <a name="remote-management"></a>遠端管理

關於 Azure 備份代理程式、原則和資料來源的所有管理工作，皆可透過 PowerShell 遠端完成。 要遠端管理的電腦必須經過正確準備。

依預設，WinRM 服務會設定為手動啟動。 但您必須將啟動類型設定為 [ *自動* ]，並應該啟動該服務。 若要驗證 WinRM 服務有在執行，[狀態] 屬性的值應該是 [ *執行中*]。

```powershell
Get-Service -Name WinRM
```

```Output
Status   Name               DisplayName
------   ----               -----------
Running  winrm              Windows Remote Management (WS-Manag...
```

應該將 PowerShell 設定為可以遠端執行。

```powershell
Enable-PSRemoting -Force
```

```Output
WinRM is already set up to receive requests on this computer.
WinRM has been updated for remote management.
WinRM firewall exception enabled.
```

```powershell
Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Force
```

現在可以遠端管理電腦 - 從代理程式的安裝開始。 例如，下列指令碼會將代理程式複製到遠端電腦並進行安裝。

```powershell
$DLoc = "\\REMOTESERVER01\c$\Windows\Temp"
$Agent = "\\REMOTESERVER01\c$\Windows\Temp\MARSAgentInstaller.exe"
$Args = "/q"
Copy-Item "C:\Downloads\MARSAgentInstaller.exe" -Destination $DLoc -Force

$Session = New-PSSession -ComputerName REMOTESERVER01
Invoke-Command -Session $Session -Script { param($D, $A) Start-Process -FilePath $D $A -Wait } -ArgumentList $Agent, $Args
```

## <a name="next-steps"></a>後續步驟

如需有關 Windows Server/用戶端的 Azure 備份詳細資訊：

* [Azure 備份的簡介](./backup-overview.md)
* [備份 Windows 伺服器](backup-windows-with-mars-agent.md)

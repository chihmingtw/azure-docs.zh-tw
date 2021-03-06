---
title: 在加密的裝置上設定 LVM 和 RAID-Azure 磁碟加密
description: 本文提供在 Linux Vm 的加密裝置上設定 LVM 和 RAID 的指示。
author: jofrance
ms.service: security
ms.topic: how-to
ms.author: jofrance
ms.date: 03/17/2020
ms.custom: seodec18
ms.openlocfilehash: 746243336d74aefc55df48872fe9dd21e9cd99a5
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/28/2020
ms.locfileid: "87268215"
---
# <a name="configure-lvm-and-raid-on-encrypted-devices"></a>在加密的裝置上設定 LVM 和 RAID

本文逐步解說如何在加密的裝置上執行邏輯磁片區管理（LVM）和 RAID。 此程式適用于下列環境：

- Linux 散發套件
    - RHEL 7.6 +
    - Ubuntu 18.04 +
    - SUSE 12 +
- Azure 磁碟加密單一傳遞延伸模組
- Azure 磁碟加密雙重傳遞延伸模組


## <a name="scenarios"></a>案例

本文中的程式支援下列案例：  

- 在加密的裝置上設定 LVM （LVM-crypt）
- 在加密的裝置上設定 RAID （crypt）

在基礎裝置或裝置加密之後，您就可以在該加密層上建立 LVM 或 RAID 結構。 

實體磁片區（Pv）會建立在加密層的頂端。 實體磁片區是用來建立磁片區群組。 您會建立磁片區，並在/etc/fstab 上新增所需的專案 

![LVM 結構層的圖表](./media/disk-encryption/lvm-raid-on-crypt/000-lvm-raid-crypt-diagram.png)

以類似的方式，會在磁片上的加密層級上建立 RAID 裝置。 檔案系統是在 RAID 裝置上建立，並以一般裝置的形式新增至/etc/fstab。

## <a name="considerations"></a>考量

我們建議您使用 crypt 上的 LVM。 當 LVM 因特定應用程式或環境的限制而無法使用時，可選擇 RAID。

您將使用**EncryptFormatAll**選項。 如需此選項的詳細資訊，請參閱[在 Linux vm 上使用資料磁片的 EncryptFormatAll 功能](./disk-encryption-linux.md#use-encryptformatall-feature-for-data-disks-on-linux-vms)。

雖然您也可以在加密作業系統時使用這個方法，但我們只是在這裡加密資料磁片磁碟機。

這些程式假設您已在[Linux vm](./disk-encryption-linux.md)和[快速入門：使用 Azure CLI 建立和加密 linux vm](./disk-encryption-cli-quickstart.md)中的 Azure 磁碟加密案例中，審查了必要條件。

Azure 磁碟加密的雙通路版本是在取代路徑上，不應再用於新的加密。

## <a name="general-steps"></a>一般步驟

當您使用「內部 crypt」設定時，請使用下列程式中所述的進程。

>[!NOTE] 
>我們會在本文中使用到變數。 請視需要將值取代。

### <a name="deploy-a-vm"></a>部署 VM 
下列命令是選擇性的，但我們建議您將其套用至新部署的虛擬機器（VM）。

PowerShell：

```powershell
New-AzVm -ResourceGroupName ${RGNAME} `
-Name ${VMNAME} `
-Location ${LOCATION} `
-Size ${VMSIZE} `
-Image ${OSIMAGE} `
-Credential ${creds} `
-Verbose
```
Azure CLI：

```bash
az vm create \
-n ${VMNAME} \
-g ${RGNAME} \
--image ${OSIMAGE} \
--admin-username ${username} \
--admin-password ${password} \
-l ${LOCATION} \
--size ${VMSIZE} \
-o table
```
### <a name="attach-disks-to-the-vm"></a>將磁片連接至 VM
針對 `$N` 您想要附加至 VM 的新磁片數目，重複下列命令。

PowerShell：

```powershell
$storageType = 'Standard_LRS'
$dataDiskName = ${VMNAME} + '_datadisk0'
$diskConfig = New-AzDiskConfig -SkuName $storageType -Location $LOCATION -CreateOption Empty -DiskSizeGB 5
$dataDisk1 = New-AzDisk -DiskName $dataDiskName -Disk $diskConfig -ResourceGroupName ${RGNAME}
$vm = Get-AzVM -Name ${VMNAME} -ResourceGroupName ${RGNAME} 
$vm = Add-AzVMDataDisk -VM $vm -Name $dataDiskName -CreateOption Attach -ManagedDiskId $dataDisk1.Id -Lun 0
Update-AzVM -VM ${VM} -ResourceGroupName ${RGNAME}
```

Azure CLI：

```bash
az vm disk attach \
-g ${RGNAME} \
--vm-name ${VMNAME} \
--name ${VMNAME}datadisk1 \
--size-gb 5 \
--new \
-o table
```

### <a name="verify-that-the-disks-are-attached-to-the-vm"></a>確認磁片已連接至 VM
PowerShell：
```powershell
$VM = Get-AzVM -ResourceGroupName ${RGNAME} -Name ${VMNAME}
$VM.StorageProfile.DataDisks | Select-Object Lun,Name,DiskSizeGB
```
![PowerShell 中已連接的磁片清單](./media/disk-encryption/lvm-raid-on-crypt/001-lvm-raid-check-disks-powershell.png)

Azure CLI：

```bash
az vm show -g ${RGNAME} -n ${VMNAME} --query storageProfile.dataDisks -o table
```
![Azure CLI 中已連接的磁片清單](./media/disk-encryption/lvm-raid-on-crypt/002-lvm-raid-check-disks-cli.png)

入口網站：

![入口網站中已連接的磁片清單](./media/disk-encryption/lvm-raid-on-crypt/003-lvm-raid-check-disks-portal.png)

作業系統：

```bash
lsblk 
```
![作業系統中已連接的磁片清單](./media/disk-encryption/lvm-raid-on-crypt/004-lvm-raid-check-disks-os.png)

### <a name="configure-the-disks-to-be-encrypted"></a>設定要加密的磁片
此設定是在作業系統層級完成。 對應的磁片會透過 Azure 磁碟加密設定傳統加密：

- 檔案系統是在磁片上建立的。
- 系統會建立暫存掛接點來掛接檔案系統。
- /Etc/fstab 上的檔案系統會設定為在開機時掛接。

檢查指派給新磁片的裝置代號。 在此範例中，我們會使用四個數據磁片。

```bash
lsblk 
```
![連接至 OS 的資料磁片](./media/disk-encryption/lvm-raid-on-crypt/004-lvm-raid-check-disks-os.png)

### <a name="create-a-file-system-on-top-of-each-disk"></a>在每個磁片上建立檔案系統
此命令會在 "for" 迴圈的 "in" 部分定義的每個磁片上，逐一查看 ext4 檔案系統的建立。

```bash
for disk in c d e f; do echo mkfs.ext4 -F /dev/sd${disk}; done |bash
```
![建立 ext4 檔案系統](./media/disk-encryption/lvm-raid-on-crypt/005-lvm-raid-create-temp-fs.png)

尋找您最近建立之檔案系統的通用唯一識別碼（UUID）、建立暫存資料夾、在/etc/fstab 上新增對應的專案，以及掛接所有檔案系統。

此命令也會在 "for" 迴圈的 "in" 部分中，逐一查看每個定義的磁片：

```bash
for disk in c d e f; do diskuuid="$(blkid -s UUID -o value /dev/sd${disk})"; \
mkdir /tempdata${disk}; \
echo "UUID=${diskuuid} /tempdata${disk} ext4 defaults,nofail 0 0" >> /etc/fstab; \
mount -a; \
done
``` 

### <a name="verify-that-the-disks-are-mounted-properly"></a>確認磁片已正確掛接
```bash
lsblk
```
![已掛接的暫存檔案系統清單](./media/disk-encryption/lvm-raid-on-crypt/006-lvm-raid-verify-temp-fs.png)

也請確認磁片已設定：

```bash
cat /etc/fstab
```
![透過 fstab 設定資訊](./media/disk-encryption/lvm-raid-on-crypt/007-lvm-raid-verify-temp-fstab.png)

### <a name="encrypt-the-data-disks"></a>加密資料磁片
使用金鑰加密金鑰（KEK）的 PowerShell：

```powershell
$sequenceVersion = [Guid]::NewGuid() 
Set-AzVMDiskEncryptionExtension -ResourceGroupName $RGNAME `
-VMName ${VMNAME} `
-DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl `
-DiskEncryptionKeyVaultId $KeyVaultResourceId `
-KeyEncryptionKeyUrl $keyEncryptionKeyUrl `
-KeyEncryptionKeyVaultId $KeyVaultResourceId `
-VolumeType 'DATA' `
-EncryptFormatAll `
-SequenceVersion $sequenceVersion `
-skipVmBackup;
```

使用 KEK Azure CLI：

```bash
az vm encryption enable \
--resource-group ${RGNAME} \
--name ${VMNAME} \
--disk-encryption-keyvault ${KEYVAULTNAME} \
--key-encryption-key ${KEYNAME} \
--key-encryption-keyvault ${KEYVAULTNAME} \
--volume-type "DATA" \
--encrypt-format-all \
-o table
```
### <a name="verify-the-encryption-status"></a>確認加密狀態

只有在所有磁片都已加密時，才繼續進行下一個步驟。

PowerShell：

```powershell
Get-AzVmDiskEncryptionStatus -ResourceGroupName ${RGNAME} -VMName ${VMNAME}
```
![PowerShell 中的加密狀態](./media/disk-encryption/lvm-raid-on-crypt/008-lvm-raid-verify-encryption-status-ps.png)

Azure CLI：

```bash
az vm encryption show -n ${VMNAME} -g ${RGNAME} -o table
```
![Azure CLI 中的加密狀態](./media/disk-encryption/lvm-raid-on-crypt/009-lvm-raid-verify-encryption-status-cli.png)

入口網站：

![入口網站中的加密狀態](./media/disk-encryption/lvm-raid-on-crypt/010-lvm-raid-verify-encryption-status-portal.png)

OS 層級：

```bash
lsblk
```
![OS 中的加密狀態](./media/disk-encryption/lvm-raid-on-crypt/011-lvm-raid-verify-encryption-status-os.png)

延伸模組會將檔案系統新增至/var/lib/azure_disk_encryption_config/azure_crypt_mount （舊的加密）或/etc/crypttab （新的加密）。

>[!NOTE] 
>請勿修改這些檔案中的任何一個。

此檔案會在開機過程中負責啟動這些磁片，讓 LVM 或 RAID 稍後可以使用它們。 

不必擔心此檔案上的掛接點。 在我們于這些加密裝置上建立實體磁片區或 RAID 裝置之後，Azure 磁碟加密將無法取得以一般檔案系統形式掛接的磁片。 （這會移除準備程式期間所使用的檔案系統格式）。

### <a name="remove-the-temporary-folders-and-temporary-fstab-entries"></a>移除暫存資料夾和暫存 fstab 專案
您會將用來做為 LVM 一部分的磁片上的檔案系統卸載。

```bash
for disk in c d e f; do unmount /tempdata${disk}; done
```
並移除/etc/fstab 專案：

```bash
vi /etc/fstab
```
### <a name="verify-that-the-disks-are-not-mounted-and-that-the-entries-on-etcfstab-were-removed"></a>確認磁片未裝載，且已移除/etc/fstab 上的專案

```bash
lsblk
```
![確認暫存檔案系統已卸載](./media/disk-encryption/lvm-raid-on-crypt/012-lvm-raid-verify-disks-not-mounted.png)

並確認磁片已設定：
```bash
cat /etc/fstab
```
![確認已移除暫存 fstab 專案](./media/disk-encryption/lvm-raid-on-crypt/013-lvm-raid-verify-fstab-temp-removed.png)

## <a name="steps-for-lvm-on-crypt"></a>LVM 的步驟-crypt
現在，基礎磁片已加密，您可以建立 LVM 結構。

不使用裝置名稱，而是使用每個磁片的/dev/mapper 路徑來建立實體磁片區（在磁片上的 crypt 層，而不是磁片本身）。

### <a name="configure-lvm-on-top-of-the-encrypted-layers"></a>在加密的層級上設定 LVM
#### <a name="create-the-physical-volumes"></a>建立實體磁片區
您會收到一則警告，詢問是否可以清除檔案系統簽章。 繼續輸入**y**，或使用**echo "y"** ，如下所示：

```bash
echo "y" | pvcreate /dev/mapper/c49ff535-1df9-45ad-9dad-f0846509f052
echo "y" | pvcreate /dev/mapper/6712ad6f-65ce-487b-aa52-462f381611a1
echo "y" | pvcreate /dev/mapper/ea607dfd-c396-48d6-bc54-603cf741bc2a
echo "y" | pvcreate /dev/mapper/4159c60a-a546-455b-985f-92865d51158c
```
![確認已建立實體磁片區](./media/disk-encryption/lvm-raid-on-crypt/014-lvm-raid-pvcreate.png)

>[!NOTE] 
>此處的/dev/mapper/device 名稱必須根據**lsblk**的輸出來取代為實際值。

#### <a name="verify-the-information-for-physical-volumes"></a>確認實體磁片區的資訊
```bash
pvs
```

![實體磁片區的資訊](./media/disk-encryption/lvm-raid-on-crypt/015-lvm-raid-pvs.png)

#### <a name="create-the-volume-group"></a>建立磁片區群組
使用已初始化的相同裝置來建立磁片區群組：

```bash
vgcreate vgdata /dev/mapper/
```

### <a name="check-the-information-for-the-volume-group"></a>檢查磁片區群組的資訊

```bash
vgdisplay -v vgdata
```
```bash
pvs
```
![磁片區群組的資訊](./media/disk-encryption/lvm-raid-on-crypt/016-lvm-raid-pvs-on-vg.png)

#### <a name="create-logical-volumes"></a>建立邏輯磁片區

```bash
lvcreate -L 10G -n lvdata1 vgdata
lvcreate -L 7G -n lvdata2 vgdata
``` 

#### <a name="check-the-created-logical-volumes"></a>檢查已建立的邏輯磁片區

```bash
lvdisplay
lvdisplay vgdata/lvdata1
lvdisplay vgdata/lvdata2
```
![邏輯磁片區的資訊](./media/disk-encryption/lvm-raid-on-crypt/017-lvm-raid-lvs.png)

#### <a name="create-file-systems-on-top-of-the-structures-for-logical-volumes"></a>在邏輯磁片區的結構之上建立檔案系統

```bash
echo "yes" | mkfs.ext4 /dev/vgdata/lvdata1
echo "yes" | mkfs.ext4 /dev/vgdata/lvdata2
```

#### <a name="create-the-mount-points-for-the-new-file-systems"></a>建立新檔案系統的掛接點

```bash
mkdir /data0
mkdir /data1
```

#### <a name="add-the-new-file-systems-to-etcfstab-and-mount-them"></a>將新的檔案系統新增至/etc/fstab 並加以掛接

```bash
echo "/dev/mapper/vgdata-lvdata1 /data0 ext4 defaults,nofail 0 0" >>/etc/fstab
echo "/dev/mapper/vgdata-lvdata2 /data1 ext4 defaults,nofail 0 0" >>/etc/fstab
mount -a
```

#### <a name="verify-that-the-new-file-systems-are-mounted"></a>確認新的檔案系統已裝載

```bash
lsblk -fs
df -h
```
![掛載的檔案系統資訊](./media/disk-encryption/lvm-raid-on-crypt/018-lvm-raid-lsblk-after-lvm.png)

在這種**lsblk**變化中，我們會依反向順序列出顯示相依性的裝置。 此選項有助於識別依邏輯磁片區分組的裝置，而不是原始的/dev/sd [磁片] 裝置名稱。

請務必確定**nofail**選項已新增至在透過 Azure 磁碟加密加密的裝置上建立的 LVM 磁片區的掛接點選項。 它可防止作業系統在開機程式（或在維護模式）期間停滯。

如果您不使用**nofail**選項：

- 作業系統永遠不會進入 Azure 磁碟加密啟動的階段，而且會解除鎖定和掛接資料磁片。 
- 在開機程式結束時，加密的磁片將會解除鎖定。 LVM 磁片區和檔案系統將會自動掛接，直到 Azure 磁碟加密解除鎖定為止。 

您可以測試重新開機 VM，並驗證檔案系統也會在開機之後自動掛接。 視檔案系統的數量和大小而定，此程式可能需要幾分鐘的時間。

#### <a name="reboot-the-vm-and-verify-after-reboot"></a>重新開機 VM，並在重新開機後進行驗證

```bash
shutdown -r now
```
```bash
lsblk
df -h
```
## <a name="steps-for-raid-on-crypt"></a>Crypt 的步驟
現在，基礎磁片已加密，您可以繼續建立 RAID 結構。 該程式與 LVM 的程式相同，但不使用裝置名稱，而是針對每個磁片使用/dev/mapper 路徑。

#### <a name="configure-raid-on-top-of-the-encrypted-layer-of-the-disks"></a>在磁片的加密層級上設定 RAID
```bash
mdadm --create /dev/md10 \
--level 0 \
--raid-devices=4 \
/dev/mapper/c49ff535-1df9-45ad-9dad-f0846509f052 \
/dev/mapper/6712ad6f-65ce-487b-aa52-462f381611a1 \
/dev/mapper/ea607dfd-c396-48d6-bc54-603cf741bc2a \
/dev/mapper/4159c60a-a546-455b-985f-92865d51158c
```
![透過 mdadm 命令設定 RAID 的資訊](./media/disk-encryption/lvm-raid-on-crypt/019-lvm-raid-md-creation.png)

>[!NOTE] 
>此處的/dev/mapper/device 名稱必須根據**lsblk**的輸出，以實際值來取代。

### <a name="checkmonitor-raid-creation"></a>檢查/監視 RAID 建立
```bash
watch -n1 cat /proc/mdstat
mdadm --examine /dev/mapper/[]
mdadm --detail /dev/md10
```
![RAID 的狀態](./media/disk-encryption/lvm-raid-on-crypt/020-lvm-raid-md-details.png)

### <a name="create-a-file-system-on-top-of-the-new-raid-device"></a>在新的 RAID 裝置上建立檔案系統
```bash
mkfs.ext4 /dev/md10
```

為檔案系統建立新的掛接點，將新的檔案系統新增至/etc/fstab，然後將它掛接：

```bash
for device in md10; do diskuuid="$(blkid -s UUID -o value /dev/${device})"; \
mkdir /raiddata; \
echo "UUID=${diskuuid} /raiddata ext4 defaults,nofail 0 0" >> /etc/fstab; \
mount -a; \
done
```

確認已裝載新的檔案系統：

```bash
lsblk -fs
df -h
```
![掛載的檔案系統資訊](./media/disk-encryption/lvm-raid-on-crypt/021-lvm-raid-lsblk-md-details.png)

請務必確定**nofail**選項已新增至在透過 Azure 磁碟加密加密的裝置上建立之 RAID 磁片區的掛接點選項。 它可防止作業系統在開機程式（或在維護模式）期間停滯。

如果您不使用**nofail**選項：

- 作業系統永遠不會進入 Azure 磁碟加密啟動的階段，而且會解除鎖定和掛接資料磁片。
- 在開機程式結束時，加密的磁片將會解除鎖定。 RAID 磁片區和檔案系統將會自動掛接，直到 Azure 磁碟加密解除鎖定為止。

您可以測試重新開機 VM，並驗證檔案系統也會在開機之後自動掛接。 視檔案系統的數量和大小而定，此程式可能需要幾分鐘的時間。

```bash
shutdown -r now
```

而且當您可以登入時：

```bash
lsblk
df -h
```
## <a name="next-steps"></a>後續步驟

- [Azure 磁碟加密的疑難排解](disk-encryption-troubleshooting.md)

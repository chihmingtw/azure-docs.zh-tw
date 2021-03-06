---
title: 將安全性原則套用至系統時，Azure VM 沒有回應
description: 本文提供的步驟可解決當 VM 在 Azure VM 中將安全性原則套用至系統而沒有回應時，負載畫面停滯的問題。
services: virtual-machines-windows
documentationcenter: ''
author: TobyTu
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.assetid: a97393c3-351d-4324-867d-9329e31b5629
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.topic: troubleshooting
ms.date: 06/15/2020
ms.author: v-mibufo
ms.openlocfilehash: 6b50bffd1a44c0cf53f15650f5ff4d938f45df4d
ms.sourcegitcommit: 877491bd46921c11dd478bd25fc718ceee2dcc08
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/02/2020
ms.locfileid: "84907975"
---
# <a name="azure-vm-is-unresponsive-while-applying-security-policy-to-the-system"></a>將安全性原則套用至系統時，Azure VM 沒有回應

本文提供的步驟可解決當作業系統在 Azure VM 中套用安全性原則時，會停止回應並使其變得沒有回應的問題。

## <a name="symptoms"></a>徵狀

當您使用 [[開機診斷](boot-diagnostics.md)] 來觀看 VM 的螢幕擷取畫面時，您會看到螢幕擷取畫面在開機時顯示 OS 停滯，訊息如下：

> 「正在將安全性原則套用至系統」。

:::image type="content" source="media/unresponsive-vm-apply-security-policy/apply-policy.png" alt-text="Windows Server 2012 R2 啟動畫面的螢幕擷取畫面已停滯。":::

:::image type="content" source="media/unresponsive-vm-apply-security-policy/apply-policy-2.png" alt-text="作業系統啟動畫面的螢幕擷取畫面已停滯。":::

## <a name="cause"></a>原因

此問題的可能原因有眾多。 在執行記憶體傾印分析之前，您將無法得知來源。

## <a name="resolution"></a>解決方案

### <a name="process-overview"></a>程序概觀

1. [建立和存取修復 VM](#create-and-access-a-repair-vm)
2. [啟用序列主控台和記憶體傾印集合](#enable-serial-console-and-memory-dump-collection)
3. [重建 VM](#rebuild-the-vm)
4. [收集記憶體傾印檔案](#collect-the-memory-dump-file)

### <a name="create-and-access-a-repair-vm"></a>建立和存取修復 VM

1. 使用 [VM 修復命令的步驟 1-3](repair-windows-vm-using-azure-virtual-machine-repair-commands.md#repair-process-example) 準備修復 VM。
2. 使用遠端桌面連線連接到修復 VM。

### <a name="enable-serial-console-and-memory-dump-collection"></a>啟用序列主控台和記憶體傾印集合

若要啟用記憶體傾印集合和序列主控台，請執行下列指令碼：

1. 開啟提升權限的命令提示字元工作階段 (以系統管理員身分執行)。
2. 列出 BCD 存放區資料，並判斷您將在下一個步驟中使用的開機載入器識別碼。

     1. 針對第1代 VM，請輸入下列命令，並記下所列的識別碼：

        ```console
        bcdedit /store <BOOT PARTITON>:\boot\bcd /enum
        ```

        在命令中，將取代 \<BOOT PARTITON> 為連接的磁片中包含開機資料夾的磁碟分割字母。

        :::image type="content" source="media/unresponsive-vm-apply-security-policy/store-data.png" alt-text="圖表顯示在第1代 VM 中列出 BCD 存放區的輸出，其中列出 Windows 開機載入器的識別碼號碼。":::

     2. 針對第2代 VM，請輸入下列命令，並記下所列的識別碼：

        ```console
        bcdedit /store <LETTER OF THE EFI SYSTEM PARTITION>:EFI\Microsoft\boot\bcd /enum
        ```

        - 在命令中，將取代 \<LETTER OF THE EFI SYSTEM PARTITION> 為 EFI 系統磁碟分割的字母。
        - 啟動 [磁片管理] 主控台，以識別標示為「EFI 系統磁碟分割」的適當系統磁碟分割可能會很有説明。
        - 此識別碼可以是唯一的 GUID，也可以是預設的 "bootmgr"。
3. 執行下列命令以啟用序列主控台：

    ```console
    bcdedit /store <VOLUME LETTER WHERE THE BCD FOLDER IS>:\boot\bcd /ems {<BOOT LOADER IDENTIFIER>} ON
    ```

    ```console
    bcdedit /store <VOLUME LETTER WHERE THE BCD FOLDER IS>:\boot\bcd /emssettings EMSPORT:1 EMSBAUDRATE:115200
    ```

    - 在命令中，將取代 \<VOLUME LETTER WHERE THE BCD FOLDER IS> 為 BCD 資料夾的字母。
    - 在命令中，將取代 \<BOOT LOADER IDENTIFIER> 為您在上一個步驟中找到的識別碼。
4. 確認 OS 磁片上的可用空間大於 VM 上的記憶體大小（RAM）。

    1. 如果 OS 磁片上沒有足夠的空間，您應該變更將建立記憶體傾印檔案的位置。 除了在 OS 磁片上建立檔案，您也可以將它參照到任何其他已連接到具有足夠可用空間之 VM 的資料磁片。 若要變更位置，請在下列命令中，以資料磁片的磁碟機號（例如 "F："）取代 "% SystemRoot%"。
    2. 輸入下列命令（建議的傾印設定）：

        載入中斷的 OS 磁碟：

        ```console
        REG LOAD HKLM\BROKENSYSTEM <VOLUME LETTER OF BROKEN OS DISK>:\windows\system32\config\SYSTEM
        ```

        在 ControlSet001 上啟用：

        ```console
        REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 1 /f
        REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "%SystemRoot%\MEMORY.DMP" /f
        REG ADD "HKLM\BROKENSYSTEM\ControlSet001\Control\CrashControl" /v NMICrashDump /t REG_DWORD /d 1 /f
        ```

        在 ControlSet002 上啟用：

        ```console
        REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v CrashDumpEnabled /t REG_DWORD /d 1 /f
        REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v DumpFile /t REG_EXPAND_SZ /d "%SystemRoot%\MEMORY.DMP" /f
        REG ADD "HKLM\BROKENSYSTEM\ControlSet002\Control\CrashControl" /v NMICrashDump /t REG_DWORD /d 1 /f
        ```

        卸載中斷的 OS 磁碟：

        ```console
        REG UNLOAD HKLM\BROKENSYSTEM
        ```

### <a name="rebuild-the-vm"></a>重建 VM

使用 [VM 修復命令的步驟 5](repair-windows-vm-using-azure-virtual-machine-repair-commands.md#repair-process-example) 重新組裝 VM。

### <a name="collect-the-memory-dump-file"></a>收集記憶體傾印檔案

若要解決這個問題，您需要先使用記憶體傾印檔案來收集損毀和連線支援的記憶體傾印檔案。 若要收集傾印檔案，請遵循下列步驟：

1. 將 OS 磁片連結至新的修復 VM：

    - 使用[VM 修復命令的步驟 1-3](repair-windows-vm-using-azure-virtual-machine-repair-commands.md#repair-process-example)來準備新的修復 VM。
    - 使用遠端桌面連線連接到修復 VM。

2. 找出傾印檔案，並提交支援票證：

    - 在修復 VM 上，移至連接的 OS 磁片中的 windows 資料夾。 如果指派給已連結 OS 磁碟的磁碟機代號是 `F`，您必須移至 `F:\Windows`。
    - 找出記憶體 dmp 檔案，然後提交包含記憶體傾印檔案的[支援票證](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)。
    - 如果找不到記憶體 dmp 檔案，您可能會想要改為[在序列主控台中使用非遮罩式插斷（NMI）呼叫](serial-console-windows.md#use-the-serial-console-for-nmi-calls)。 您可以遵循本指南，[使用 NMI 呼叫來產生損毀](/windows/client-management/generate-kernel-or-complete-crash-dump)傾印檔案。

## <a name="next-steps"></a>後續步驟

如果您在套用本機使用者和群組原則時遇到問題，請參閱[在套用群組原則本機使用者和群組原則時，VM 沒有回應](unresponsive-vm-apply-group-policy.md)
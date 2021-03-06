---
title: 包含檔案
description: 包含檔案
services: virtual-machines-windows
author: cynthn
ms.service: virtual-machines-windows
ms.topic: include
ms.date: 02/11/2019
ms.author: cynthn
ms.custom: include file
ms.openlocfilehash: 00661043d1ec9769adbf4119a2c9c1925dcd29fa
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/13/2020
ms.locfileid: "88186310"
---
## <a name="supported-operating-systems-and-drivers"></a>支援的作業系統和驅動程式

### <a name="nvidia-tesla-cuda-drivers"></a>NVIDIA Tesla (CUDA) 驅動程式

適用于 NC、NCv2、NCv3、NCasT4_v3、ND 和 NDv2 系列 Vm 的 NVIDIA Tesla (CUDA) 驅動程式 (只有下表所列的作業系統才支援 NV 系列) 的選用功能。 以下是本文章發行時的最新驅動程式下載連結。 如需最新的驅動程式，請瀏覽 [NVIDIA](https://www.nvidia.com/) 網站。

> [!TIP]
> 在 Windows Server VM 上手動安裝 CUDA 驅動程式的替代方案，就是部署 Azure [資料科學虛擬機器](../articles/machine-learning/data-science-virtual-machine/overview.md)映像。 適用於 Windows Server 2016 的 DSVM 版本會預先安裝 NVIDIA CUDA 驅動程式、CUDA 深度類神經網路程式庫和其他工具。


| OS | 驅動程式 |
| -------- |------------- |
| Windows Server 2019 | [451.82](http://us.download.nvidia.com/tesla/451.82/451.82-tesla-desktop-winserver-2019-2016-international.exe) ( .exe)  |
| Windows Server 2016 | [451.82](http://us.download.nvidia.com/tesla/451.82/451.82-tesla-desktop-winserver-2019-2016-international.exe) ( .exe)  |

### <a name="nvidia-grid-drivers"></a>NVIDIA GRID 驅動程式

Microsoft 會將適用于 NV 和 NVv3 系列 Vm 的 NVIDIA GRID 驅動程式安裝程式重新散發為虛擬工作站或虛擬應用程式。 僅在 Azure NV 系列 Vm 上安裝這些方格驅動程式，僅適用于下表所列的作業系統。 這些驅動程式包含在 Azure 的 GRID 虛擬 GPU 軟體的授權中。 您不需要設定 NVIDIA vGPU 軟體授權服務器。

Azure 所轉散發的格線驅動程式不適用於非 NV 系列 Vm，例如 NC、NCv2、NCv3、ND 和 NDv2 系列 Vm。

請注意，Nvidia 延伸模組一律會安裝最新的驅動程式。 我們會為客戶提供舊版的連結，其相依于較舊的版本。

若為 Windows Server 2019、Windows Server 2016 和 Windows 10 (最新組建 2004) ：
- [方格 11 (451.48) ](https://go.microsoft.com/fwlink/?linkid=874181) ( .exe) 
- [方格 10.1 (442.06) ](https://download.microsoft.com/download/b/8/f/b8f5ecec-b8f9-47de-b007-ac40adc88dc8/442.06_grid_win10_64bit_international_whql.exe) ( .exe)  

若為 Windows Server 2012 R2： 
- [方格 11 (451.48) ](https://go.microsoft.com/fwlink/?linkid=874184) ( .exe) 
- [方格 10.1 (442.66) ](https://download.microsoft.com/download/4/3/3/4330fd5c-c685-4ca1-abca-3b2fb3c11d2e/442.06_grid_win8_win7_64bit_international_whql.exe) ( .exe)   

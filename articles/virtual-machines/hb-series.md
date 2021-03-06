---
title: HB 系列
description: HB 系列 Vm 的規格。
author: ju-shim
ms.service: virtual-machines
ms.subservice: sizes
ms.topic: conceptual
ms.date: 08/06/2020
ms.author: amverma
ms.reviewer: jushiman
ms.openlocfilehash: e735b389b5e223e558736f5eaa3c393f8bc29004
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/07/2020
ms.locfileid: "87926258"
---
# <a name="hb-series"></a>HB 系列

HB 系列 Vm 已針對記憶體頻寬所驅動的應用程式優化，例如流體 dynamics、明確有限元素分析和氣象模型。 HB Vm 功能 60 AMD EPYC 7551 處理器核心，每個 CPU 核心 4 GB RAM，且不會同時進行多執行緒處理。 HB VM 最多可提供每秒 260 GB 的記憶體頻寬。

ACU：199-216

進階儲存體：支援

進階儲存體快取：支援

即時移轉：不支援

記憶體保留更新：不支援

| 大小 | vCPU | 處理器 | 記憶體 (GB) | 記憶體頻寬 GB/秒 | 基底 CPU 頻率 (GHz)  | 所有核心頻率 (GHz、尖峰)  | 單核心頻率 (GHz，尖峰)  | RDMA 效能 (Gb/s)  | MPI 支援 | 暫存儲存體 (GB)  | 最大資料磁碟 | 最大乙太網路 Nic |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_HB60rs | 60 | AMD EPYC 7551 | 240 | 263 | 2.0 | 2.55 | 2.55 | 100 | 全部 | 700 | 4 | 1 |

[!INCLUDE [virtual-machines-common-sizes-table-defs](../../includes/virtual-machines-common-sizes-table-defs.md)]

## <a name="other-sizes"></a>其他大小

- [一般用途](sizes-general.md)
- [記憶體最佳化](sizes-memory.md)
- [儲存體最佳化](sizes-storage.md)
- [GPU 最佳化](sizes-gpu.md)
- [高效能計算](sizes-hpc.md)
- [前幾代](sizes-previous-gen.md)

## <a name="next-steps"></a>後續步驟
- 深入瞭解如何設定[您的 vm](./workloads/hpc/configure.md)、在[hpc 工作負載](./workloads/hpc/overview.md)中[啟用](./workloads/hpc/enable-infiniband.md)未充分運用、[設定 MPI](./workloads/hpc/setup-mpi.md)，以及優化 Azure 的 hpc 應用程式。
- 閱讀[Azure 計算技術小組的 blog](https://techcommunity.microsoft.com/t5/azure-compute/bg-p/AzureCompute)，瞭解最新的公告和一些 HPC 範例和結果。
- 如需執行 HPC 工作負載的更高層級架構視圖，請參閱[Azure 上的高效能運算 (HPC) ](/azure/architecture/topics/high-performance-computing/)。
- 深入了解 [Azure 計算單位 (ACU)](acu.md) 如何協助您比較各個 Azure SKU 的計算效能。

---
title: Azure Service Fabric 網格應用程式中的監視和診斷
description: 了解如何在 Azure 上監視和診斷 Service Fabric Mesh 應用程式。
author: srrengar
ms.topic: conceptual
ms.date: 03/19/2019
ms.author: srrengar
ms.custom: mvc, devcenter
ms.openlocfilehash: e940f0cf0d1547b317cd9e7bd15ac5486d5e70b2
ms.sourcegitcommit: dabd9eb9925308d3c2404c3957e5c921408089da
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2020
ms.locfileid: "86248402"
---
# <a name="monitoring-and-diagnostics"></a>監視和診斷
Azure Service Fabric Mesh 是一個受到完整管理的服務，讓開發人員能夠部署微服務應用程式，而不需管理虛擬機器、儲存體或網路功能。 Service Fabric Mesh 的監視和診斷可歸類為三種主要診斷資料類型：

- 應用程式記錄 - 根據您檢測應用程式的方式，這些會定義為來自您的容器化應用程式的記錄 (例如 Docker 記錄)
- 平台事件 - 來自 Mesh 平台並與您的容器作業相關的事件，目前包括容器啟用、停用及終止。
- 容器計量 - 您的容器適用的資源使用率和效能計量 (Docker 統計資料)

本文討論最新預覽版本可用的監視和診斷選項。

## <a name="application-logs"></a>應用程式記錄

您可以從您已部署的容器，依照容器檢視 Docker 記錄。 在 Service Fabric Mesh 應用程式模型中，每個容器都是您應用程式中的程式碼封裝。 若要查看相關聯的記錄與程式碼封裝，請使用下列命令：

```cli
az mesh code-package-log get --resource-group <nameOfRG> --app-name <nameOfApp> --service-name <nameOfService> --replica-name <nameOfReplica> --code-package-name <nameOfCodePackage>
```

> [!NOTE]
> 您可以使用 "az mesh service-replica" 命令來取得複本名稱。 複本名稱是從0遞增的整數。

從投票應用程式的 VotingWeb.Code 容器查看記錄時，其外觀如下所示：

```cli
az mesh code-package-log get --resource-group <nameOfRG> --application-name SbzVoting --service-name VotingWeb --replica-name 0 --code-package-name VotingWeb.Code
```

## <a name="container-metrics"></a>容器計量 

網格環境會公開一些計量，指出容器的執行方式。 您可以透過 Azure 入口網站和 Azure 監視器 CLI 取得下列計量：

| 計量 | 描述 | 單位|
|----|----|----|
| CpuUtilization | 以百分比表示的 ActualCpu/AllocatedCpu | % |
| MemoryUtilization | 以百分比表示的 ActualMem/AllocatedMem | % |
| AllocatedCpu | 根據 Azure Resource Manager 範本配置的 Cpu | Millicore |
| AllocatedMemory | 根據 Azure Resource Manager 範本配置的記憶體 | MB |
| ActualCpu | CPU 使用量 | Millicore |
| ActualMemory | 記憶體使用量 | MB |
| ContainerStatus | 0-無效：容器狀態不明 <br> 1-暫止：容器已排程啟動 <br> 2-正在啟動：容器正在啟動中 <br> 3-已啟動：容器已成功啟動 <br> 4-正在停止：正在停止容器 <br> 5-已停止：容器已成功停止 | N/A |
| ApplicationStatus | 0-未知：無法抓取狀態 <br> 1-就緒：應用程式已成功執行 <br> 2-升級：正在進行升級 <br> 3-建立：正在建立應用程式 <br> 4-正在刪除：正在刪除應用程式 <br> 5-失敗：無法部署應用程式 | N/A |
| ServiceStatus | 0-無效：服務目前沒有健全狀況狀態 <br> 1-確定：服務狀況良好  <br> 2-警告：可能有一些錯誤需要調查 <br> 3-錯誤：有問題需要調查 <br> 4-未知：無法抓取狀態 | N/A |
| ServiceReplicaStatus | 0-無效：複本目前沒有健全狀況狀態 <br> 1-確定：服務狀況良好  <br> 2-警告：可能有一些錯誤需要調查 <br> 3-錯誤：有問題需要調查 <br> 4-未知：無法抓取狀態 | N/A | 
| RestartCount | 容器重新開機次數 | N/A |

> [!NOTE]
> ServiceStatus 和 ServiceReplicaStatus 值與 Service Fabric 中的[HealthState](/dotnet/api/system.fabric.health.healthstate?view=azure-dotnet)相同。 

每個計量都可在不同的維度上使用，因此您可以在不同的層級上查看匯總。 目前的維度清單如下所示：

* ApplicationName
* ServiceName
* ServiceReplicaName
* CodePackageName

> [!NOTE]
> Linux 應用程式無法使用 [CodePackageName] 維度。 

每個維度都會對應至[Service Fabric 應用程式模型](service-fabric-mesh-service-fabric-resources.md#applications-and-services)的不同元件

### <a name="azure-monitor-cli"></a>Azure 監視器 CLI

[AZURE 監視器 CLI](/cli/azure/monitor/metrics?view=azure-cli-latest#az-monitor-metrics-list)檔中提供完整的命令清單，但我們已包含以下幾個實用的範例 

在每個範例中，資源識別碼會遵循此模式

`"/subscriptions/<your sub ID>/resourcegroups/<your RG>/providers/Microsoft.ServiceFabricMesh/applications/<your App name>"`


* 應用程式中容器的 CPU 使用率

```cli
    az monitor metrics list --resource <resourceId> --metric "CpuUtilization"
```
* 每個服務複本的記憶體使用率
```cli
    az monitor metrics list --resource <resourceId> --metric "MemoryUtilization" --dimension "ServiceReplicaName"
``` 

* 在1小時的時間範圍內重新開機每個容器 
```cli
    az monitor metrics list --resource <resourceId> --metric "RestartCount" --start-time 2019-02-01T00:00:00Z --end-time 2019-02-01T01:00:00Z
``` 

* 1小時時間範圍內名為 "VotingWeb" 之服務的平均 CPU 使用率
```cli
    az monitor metrics list --resource <resourceId> --metric "CpuUtilization" --start-time 2019-02-01T00:00:00Z --end-time 2019-02-01T01:00:00Z --aggregation "Average" --filter "ServiceName eq 'VotingWeb'"
``` 

### <a name="metrics-explorer"></a>計量瀏覽器

計量瀏覽器是入口網站中的一個分頁，您可以在其中視覺化網格應用程式的所有計量。 此分頁可以在入口網站的應用程式頁面和 Azure 監視器分頁中存取，後者可用來查看支援 Azure 監視器的所有 Azure 資源的計量。 

![計量瀏覽器](./media/service-fabric-mesh-monitoring-diagnostics/metricsexplorer.png)


<!--
### Container Insights

In addition to the metrics explorer, we also have a dashboard available out of the box that shows sample metrics over time under the Insights blade in the application's page in the portal. 

![Container Insights](./media/service-fabric-mesh-monitoring-diagnostics/containerinsights.png)
-->

## <a name="next-steps"></a>後續步驟
* 若要深入了解 Service Fabric Mesh，請閱讀 [Service Fabric Mesh 概觀](service-fabric-mesh-overview.md)。
* 若要深入瞭解 Azure 監視器計量命令，請參閱[AZURE 監視器 CLI](/cli/azure/monitor/metrics?view=azure-cli-latest#az-monitor-metrics-list)檔。

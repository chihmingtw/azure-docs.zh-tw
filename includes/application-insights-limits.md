---
title: 包含檔案
description: 包含檔案
services: application-insights
author: mrbullwinkle
ms.service: application-insights
ms.topic: include
ms.date: 08/06/2019
ms.author: mbullwin
ms.custom: include file
ms.openlocfilehash: 14e2bd4af2616e9dd33fe8267de132ab6c0f1cfe
ms.sourcegitcommit: 877491bd46921c11dd478bd25fc718ceee2dcc08
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/02/2020
ms.locfileid: "69982619"
---
每個應用程式的計量和事件數目有一些限制，也就是每個檢測金鑰。 限制取決於您選擇的[定價方案](https://azure.microsoft.com/pricing/details/application-insights/)。

| 資源 | 預設限制 | 注意
| --- | --- | --- |
| 每日資料總量 | 100 GB | 您可以設定上限來減少資料。 如果您需要更多資料，可以從入口網站將限制增加到最多 1,000 GB。 對於大於 1000 GB 的容量，傳送電子郵件至 AIDataCap@microsoft.com 。
| 節流 | 每秒32000個事件 | 此限制會測量超過一分鐘。
| 資料保留 | [30-730 天](https://docs.microsoft.com/azure/azure-monitor/app/pricing#change-the-data-retention-period)  | 此資源適用於[搜尋](../articles/azure-monitor/app/diagnostic-search.md)、[分析](../articles/azure-monitor/app/analytics.md)和[計量瀏覽器](../articles/azure-monitor/app/metrics-explorer.md)。
| [可用性多步驟測試](../articles/azure-monitor/app/availability-multistep.md)詳述的結果保留期 | 90 天 | 此資源會提供每個步驟的詳細結果。
| 遙測專案大小上限 | 64 kB |
| 每個批次的遙測專案上限 | 64 K |
| 屬性和度量名稱長度 | 150 | 請參閱[類型結構描述](https://github.com/microsoft/ApplicationInsights-Home/tree/master/EndpointSpecs/Schemas/Bond)。
| 屬性值字串長度 | 8,192  | 請參閱[類型結構描述](https://github.com/microsoft/ApplicationInsights-Home/tree/master/EndpointSpecs/Schemas/Bond)。
| 追蹤和例外狀況訊息長度 | 32,768  | 請參閱[類型結構描述](https://github.com/microsoft/ApplicationInsights-Home/tree/master/EndpointSpecs/Schemas/Bond)。
| 每個應用程式的[可用性測試](../articles/azure-monitor/app/monitor-web-app-availability.md)計數 | 100 |
| [分析工具](../articles/azure-monitor/app/profiler.md)資料保留期 | 5 天 |
| 每天傳送的[分析工具](../articles/azure-monitor/app/profiler.md)資料 | 10 GB |

如需詳細資訊，請參閱[關於 Application Insights 中的價格和配額](../articles/azure-monitor/app/pricing.md)。
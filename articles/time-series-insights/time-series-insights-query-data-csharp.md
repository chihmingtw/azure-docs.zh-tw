---
title: '使用 c # 程式碼從 Gen1 環境查詢資料-Azure 時間序列深入解析 Gen1 |Microsoft Docs'
description: '瞭解如何使用以 c # 撰寫的自訂應用程式來查詢 Azure 時間序列深入解析 Gen1 環境中的資料。'
ms.service: time-series-insights
services: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.devlang: csharp
ms.workload: big-data
ms.topic: conceptual
ms.date: 08/12/2020
ms.custom: seodec18
ms.openlocfilehash: e7f5750d89c661cad7616cf1c56157c6e92e1175
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/20/2020
ms.locfileid: "88654458"
---
# <a name="query-data-from-the-azure-time-series-insights-gen1-environment-using-c-sharp"></a>使用 C 銳利查詢 Azure 時間序列深入解析 Gen1 環境中的資料

此 c # 範例示範如何使用 [Gen1 查詢 api](https://docs.microsoft.com/rest/api/time-series-insights/gen1-query) 來查詢 Azure 時間序列深入解析 Gen1 環境中的資料。

> [!TIP]
> View Gen1 c # 程式碼範例位於 [https://github.com/Azure-Samples/Azure-Time-Series-Insights](https://github.com/Azure-Samples/Azure-Time-Series-Insights/tree/master/gen1-sample/csharp-tsi-gen1-sample) 。

## <a name="summary"></a>摘要

以下範例程式碼會示範下列功能：

* 如何使用 [Microsoft.IdentityModel.Clients.ActiveDirectory](https://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory/) 透過 Azure Active Directory 取得存取權杖。

* 如何在 `Authorization` 後續的查詢 API 要求標頭中傳遞取得的存取權杖。

* 此範例會呼叫每個 Gen1 查詢 Api，以示範如何對進行 HTTP 要求：
  * [取得環境 API](https://docs.microsoft.com/rest/api/time-series-insights/gen1-query-api#get-environments-api) ，以傳回使用者有權存取的環境
  * [取得環境可用性 API](https://docs.microsoft.com/rest/api/time-series-insights/gen1-query-api#get-environment-availability-api)
  * [取得環境中繼資料 API](https://docs.microsoft.com/rest/api/time-series-insights/gen1-query-api#get-environment-metadata-api) 以取出環境中繼資料
  * [取得環境事件 API](https://docs.microsoft.com/rest/api/time-series-insights/gen1-query-api#get-environment-events-api)
  * [取得環境匯總 API](https://docs.microsoft.com/rest/api/time-series-insights/gen1-query-api#get-environment-aggregates-api)

* 如何使用 WSS 與 Gen1 查詢 Api 互動以訊息：

  * [取得環境事件串流 API](https://docs.microsoft.com/rest/api/time-series-insights/gen1-query-api#get-environment-events-streamed-api)
  * [取得環境匯總串流 API](https://docs.microsoft.com/rest/api/time-series-insights/gen1-query-api#get-environment-aggregates-streamed-api)

## <a name="prerequisites-and-setup"></a>先決條件和設定

編譯及執行範例程式碼之前，您必須先完成下列步驟：

1. 布建[Gen1 Azure 時間序列深入解析](https://docs.microsoft.com/azure/time-series-insights/time-series-insights-get-started)環境。
1. 針對 Azure Active Directory 設定 Azure 時間序列見解環境，如[驗證和授權](time-series-insights-authentication-and-authorization.md)中所述。
1. 安裝必要的專案相依性。
1. 使用適當的環境識別碼來取代每個 **#DUMMY #** ，以編輯以下的範例程式碼。
1. 在 Visual Studio 內執行程式碼。

## <a name="project-dependencies"></a>專案相依性

建議使用最新版的 Visual Studio：

* [Visual Studio 2019](https://visualstudio.microsoft.com/vs/) - Version 16.4.2+

範例程式碼有兩個必要的相依性：

* [Microsoft.identitymodel。](https://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory/) 3.13.9 套件。
* [Newtonsoft.Js](https://www.nuget.org/packages/Newtonsoft.Json) 9.0.1 版套件。

選取 [建置] > [建置方案]，以下載 Visual Studio 2019 中的套件。

或者，使用 [NuGet 2.12 +](https://www.nuget.org/)新增套件：

* `dotnet add package Newtonsoft.Json --version 9.0.1`
* `dotnet add package Microsoft.IdentityModel.Clients.ActiveDirectory --version 3.13.9`

## <a name="c-sample-code"></a>C# 範例程式碼

請參閱 [Azure 時間序列深入解析](https://github.com/Azure-Samples/Azure-Time-Series-Insights/blob/master/gen1-sample/csharp-tsi-gen1-sample/Program.cs)] 存放庫，以存取 c # 範例程式碼。 

## <a name="next-steps"></a>後續步驟

* 若要深入了解查詢，請參閱[查詢 API 參考](https://docs.microsoft.com/rest/api/time-series-insights/gen1-query-api) (機器翻譯)。

* 請參閱如何[使用用戶端 SDK 將 JavaScript 應用程式連線到時間序列見解](https://github.com/microsoft/tsiclient) (英文)。
Azure-範例/Azure-時間序列-見解/gen1-sample/csharp-tsi-gen1-sample/Program .cs

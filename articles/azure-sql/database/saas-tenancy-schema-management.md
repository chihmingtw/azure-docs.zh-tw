---
title: 在單一租使用者應用程式中管理架構
description: 在使用 Azure SQL Database 的單一租用戶應用程式中管理多個租用戶的結構描述
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 09/19/2018
ms.openlocfilehash: 60c2330578ef4b8e3e40dc3e37a0c8b1eb291e2f
ms.sourcegitcommit: 877491bd46921c11dd478bd25fc718ceee2dcc08
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/02/2020
ms.locfileid: "85255546"
---
# <a name="manage-schema-in-a-saas-application-using-the-database-per-tenant-pattern-with-azure-sql-database"></a>使用每一租用戶一個資料庫的模式，透過 Azure SQL Database 管理 SaaS 應用程式中的結構描述
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]
 
隨著資料庫應用程式的進化，勢必要對資料庫結構描述或參考資料進行變更。  並且需要定期執行資料庫維護工作。 若要對採用每一租用戶一個資料庫模式的應用程式進行管理，您需要將這些變更和維護工作套用至多個租用戶資料庫。

本教學課程會探討兩種案例：為所有租用戶部署參考資料更新，以及針對包含參考資料的資料表重建索引。 在所有租用戶資料庫及建立新租用戶資料庫所用的範本資料庫上，[彈性作業](../../sql-database/elastic-jobs-overview.md)功能會用來執行這些動作。

在本教學課程中，您將了解如何：

> [!div class="checklist"]
> 
> * 建立作業代理程式
> * 使 T-SQL 作業在所有租用戶資料庫上執行
> * 更新所有租用戶資料庫中的參考資料
> * 針對所有租用戶資料庫中的資料表建立索引


若要完成本教學課程，請確定符合下列必要條件：

* 已部署 Wingtip Tickets SaaS Database Per Tenant 應用程式。 若要在五分鐘內完成部署，請參閱[部署及探索每一租用戶一個資料庫的 Wingtip Tickets SaaS 應用程式](../../sql-database/saas-dbpertenant-get-started-deploy.md)
* 已安裝 Azure PowerShell。 如需詳細資料，請參閱[開始使用 Azure PowerShell](https://docs.microsoft.com/powershell/azure/get-started-azureps)
* 已安裝最新版的 SQL Server Management Studio (SSMS)。 [下載並安裝 SSMS](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms)


## <a name="introduction-to-saas-schema-management-patterns"></a>SaaS 結構描述管理模式的簡介

每一租用戶一個資料庫的模式可有效地隔離租用戶資料，但是會增加要管理及維護的資料庫數目。 [彈性作業](../../sql-database/elastic-jobs-overview.md)可協助管理和管理多個資料庫。 這些作業可讓您安全且可靠地對一組資料庫執行工作 (Transact-SQL 指令碼)。 作業可在應用程式中跨所有租用戶資料庫部署結構描述和一般參考資料變更。 彈性作業也可以用來維護建立新租用戶所使用的「範本」** 資料庫，確保它隨時具有最新的結構描述和參考資料。

![畫面](./media/saas-tenancy-schema-management/schema-management-dpt.png)


## <a name="elastic-jobs-public-preview"></a>彈性作業公開預覽

新版本的「彈性作業」現在是 Azure SQL Database 的整合功能。 這個新版本的彈性作業目前處於公開預覽狀態。 此公開預覽目前支援使用 PowerShell 來建立作業代理程式，以及使用 T-sql 建立及管理作業。
如需詳細資訊，請參閱[彈性資料庫作業](https://docs.microsoft.com/azure/azure-sql/database/elastic-jobs-overview)的文章。

## <a name="get-the-wingtip-tickets-saas-database-per-tenant-application-scripts"></a>取得每一租用戶一個資料庫的 Wingtip Tickets SaaS 應用程式指令碼

在 [WingtipTicketsSaaS-DbPerTenant](https://github.com/Microsoft/WingtipTicketsSaaS-DbPerTenant) GitHub 存放庫可取得應用程式原始程式碼和管理指令碼。 關於下載和解除封鎖 Wingtip Tickets SaaS 指令碼的步驟，請參閱[一般指引](saas-tenancy-wingtip-app-guidance-tips.md)。

## <a name="create-a-job-agent-database-and-new-job-agent"></a>建立作業代理程式資料庫和新的作業代理程式

本教學課程要求您必須使用 PowerShell 建立作業代理程式和其支援作業代理程式資料庫。 作業代理程式資料庫會保存作業定義、作業狀態和記錄。 一旦建立作業代理程式和其資料庫之後，您就可以立即建立及監視作業。

1. 在 [PowerShell ISE]**** 中開啟 …\\Learning Modules\\Schema Management\\*Demo-SchemaManagement.ps1*。
1. 按 **F5** 以執行指令碼。

*Demo-SchemaManagement.ps1*腳本會呼叫*Deploy-SchemaManagement.ps1*腳本，在目錄伺服器上建立名為*osagent*的資料庫。 接著，該指令碼會建立作業代理程式，以參數的形式使用資料庫。

## <a name="create-a-job-to-deploy-new-reference-data-to-all-tenants"></a>建立作業以將新的參考資料部署到所有租用戶

在 Wingtip Tickets 應用程式中，每個租用戶資料庫都包含一組支援的場地類型。 每個場地都屬於特定場地類型，其中定義了可舉辦的事件種類，並決定應用程式中使用的背景影像。 若要讓應用程式支援新的事件類型，則必須更新此參考資料及新增場地類型。  在這個練習中，您要將更新部署到所有租用戶資料庫，以新增兩個額外的場地類型：*Motorcycle Racing* (機車賽) 和 *Swimming Club* (游泳俱樂部)。

首先，檢閱每個租用戶資料庫中的場地類型。 連線至 SQL Server Management Studio (SSMS) 中的其中一個租用戶資料庫，並檢查 VenueTypes 資料表。  您也可以在 Azure 入口網站的查詢編輯器中查詢此資料表 (從資料庫頁面進行存取)。 

1. 開啟 SSMS 並連線到租用戶伺服器：tenants1-dpt-&lt;user&gt;.database.windows.net**
1. 若要確認目前**未**包含*Motorcycle 比賽*和*Swimming 俱樂部*，請流覽至*tenants1-tenants1-dpt user- &lt; 使用者 &gt; *伺服器上的_contosoconcerthall_資料庫，並查詢*VenueTypes*資料表。

現在讓我們建立作業以更新所有租用戶資料庫中的 *VenueTypes* 資料表，以新增場地類型。

為了建立新的作業，您會使用作業代理程式建立時，在 jobagent__ 資料庫中建立的一組作業系統預存程序。

1. 在 SSMS 中，連線到目錄伺服器：catalog-dpt-&lt;user&gt;.database.windows.net** 伺服器 
1. 在 SSMS 中，開啟檔案 …\\Learning Modules\\Schema Management\\DeployReferenceData.sql
1. 修改陳述式：SET @wtpUser = &lt;user&gt; ，並使用您在部署 Wingtip Tickets SaaS 每一租用戶一個資料庫應用程式時使用的 User 值來取代
1. 確定您已連線到 jobagent__ 資料庫，然後按 **F5** 以執行指令碼

請觀察 *DeployReferenceData.sql* 指令碼中的下列元素：
* **sp\_add\_target\_group** 會建立目標群組名稱 DemoServerGroup。
* **sp\_add\_target\_group\_member** 會用來定義一組目標資料庫。  首先會新增 tenants1-dpt-&lt;User&gt;__ 伺服器。  將伺服器新增為目標，會使該伺服器中的資料庫在作業執行時納入作業中。 接著，basetenantdb__ 資料庫和 adhocreporting** 資料庫 (將在之後的教學課程中使用) 會新增為目標。
* **sp\_add\_job** 會建立稱為「參考資料部署」__ 的作業。
* **sp\_add\_jobstep** 會建立作業步驟，包含更新參考 VenueTypes 資料表的 T-SQL 命令文字。
* 指令碼中的其餘檢視會顯示物件是否存在，以及監視作業執行。 使用這些查詢來檢閱 **lifecycle** 資料行中的狀態值，以判斷作業在所有目標資料庫上完成的時間。

當指令碼完成之後，您可以驗證參考資料是否已更新。  在 SSMS 中，瀏覽至 tenants1-dpt-&lt;user&gt;** 伺服器上的 contosoconcerthall** 資料庫，並查詢 VenueTypes** 資料表。  請檢查*Motorcycle 的比賽*和*Swimming 俱樂部*現在**是否**存在。


## <a name="create-a-job-to-manage-the-reference-table-index"></a>建立作業以管理參考資料表索引

此練習會在參考資料表主索引鍵上使用作業重建索引。  這是典型的資料庫維護作業，可能會在載入大量資料後執行。

使用相同的作業「系統」預存程序建立作業。

1. 開啟 SSMS 並聯機至_目錄-tenants1-dpt user- &lt; user &gt; . database.windows.net_伺服器
1. 開啟檔案 _... \\學習模組 \\ 架構管理 \\ onlinereindex.sql_
1. 按一下滑鼠右鍵，選取 [連線]，然後連接到_tenants1-dpt user- &lt; user &gt; . database.windows.net_伺服器（如果尚未連線）
1. 確定您已連線到 jobagent__ 資料庫，然後按 **F5** 以執行指令碼

請觀察 OnlineReindex.sql__ 指令碼中的下列元素：
* **sp \_ add \_ job**會建立名為「線上重新編制 PK \_ \_ 索引重建 venuetyp \_ \_ 265E44FD7FD4C885 的新作業」
* **sp\_add\_jobstep** 會建立作業步驟，其中包含要更新索引的 T-SQL 命令文字
* 指令碼中的其餘檢視會監視作業執行。 使用這些查詢來檢閱 **lifecycle** 資料行中的狀態值，以判斷作業在所有目標群組成員上成功完成的時間。



## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> 
> * 建立可跨多個資料庫執行 T-SQL 作業的作業代理程式
> * 更新所有租用戶資料庫中的參考資料
> * 針對所有租用戶資料庫中的資料表建立索引

接下來，請嘗試[特定報表教學](../../sql-database/saas-tenancy-cross-tenant-reporting.md)課程，以探索跨租使用者資料庫執行分散式查詢。


## <a name="additional-resources"></a>其他資源

* [以 Wingtip 票證 SaaS Database Per Tenant 應用程式部署為基礎的其他教學課程](../../sql-database/saas-dbpertenant-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials)
* [管理相應放大的雲端資料庫](../../sql-database/elastic-jobs-overview.md)

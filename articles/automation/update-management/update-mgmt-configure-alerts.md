---
title: 如何建立 Azure 自動化更新管理的警示
description: 本文說明如何設定 Azure 警示，以通知有關更新評估或部署的狀態。
services: automation
ms.subservice: update-management
ms.date: 07/28/2020
ms.topic: conceptual
ms.openlocfilehash: 2c39a07ceac4d36bf3ef7394927589b53da7d789
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/30/2020
ms.locfileid: "87450054"
---
# <a name="how-to-create-alerts-for-update-management"></a>如何建立更新管理的警示

Azure 中的警示會主動通知您 runbook 作業、服務健康情況問題，或與您的自動化帳戶相關的其他案例的結果。 Azure 自動化不包含預先設定的警示規則，但您可以根據它所產生的資料來建立自己的警示規則。 本文提供使用更新管理隨附的計量來建立警示規則的指引。

## <a name="available-metrics"></a>可用的計量

Azure 自動化會建立兩個與收集並轉送至 Azure 監視器更新管理相關的不同平臺計量。 這些計量可供使用[計量瀏覽器](../../azure-monitor/platform/metrics-charts.md)進行分析，並使用[計量警示規則](../../azure-monitor/platform/alerts-metric.md)進行警示。

發出的兩個計量如下：

* 更新部署機器執行總計
* 更新部署執行總計

用於警示時，這兩個計量支援包含額外資訊的維度，以協助將警示範圍設定為特定的更新部署詳細資料。 下表顯示有關設定警示時可用之計量和維度的詳細資料。

|訊號名稱|維度|描述
|---|---|---|
|`Total Update Deployment Runs`|- 更新部署名稱<br>- 狀態 | 針對更新部署的整體狀態發出警示。|
|`Total Update Deployment Machine Runs`|- 更新部署名稱</br>- 狀態</br>- 目標電腦</br>- 更新部署執行識別碼    |針對鎖定特定機器的更新部署狀態發出警示。|

## <a name="create-alert"></a>建立警示

請遵循下列步驟來設定警示，讓您知道更新部署的狀態。 如果您不熟悉 Azure 警示，請參閱[Azure 警示總覽](../../azure-monitor/platform/alerts-overview.md)。

1. 在您的自動化帳戶中，選取 [**監視**] 下的 [**警示**]，然後選取 [**新增警示規則**]。

2. 在 [**建立警示規則**] 頁面上，已選取您的自動化帳戶作為資源。 如果您想要變更它，請選取 [**編輯資源**]。

3. 在 [選取資源] 頁面上，從 [**依資源類型篩選**] 下拉式清單中選擇 [**自動化帳戶**]。

4. 選取您想要使用的自動化帳戶，然後選取 [**完成**]。

5. 選取 [**新增條件**] 以選擇適合您需求的信號。

6. 針對維度，請從清單中選取有效的值。 如果您想要的值不在清單中，請選取 **\+** 維度旁的，然後輸入自訂名稱。 然後選取要尋找的值。 如果您想要選取維度的所有值，請選取 [**選取 \* ** ] 按鈕。 如果您沒有為維度選取值，則更新管理會忽略該維度。

    ![設定訊號邏輯](./media/update-mgmt-manage-updates-for-vm/signal-logic.png)

7. 在 [**警示邏輯**] 底下的 [**時間匯總**] 和 [**閾值**] 欄位中輸入值，然後選取 [**完成**]。

8. 在下一個頁面上，輸入警示的名稱和描述。

9. 將執行成功的 [嚴重性] 欄位設定為 [資訊 (嚴重性 2)]，或將執行失敗的嚴重性設定為 [資訊 (嚴重性 1)]。

    ![設定訊號邏輯](./media/update-mgmt-manage-updates-for-vm/define-alert-details.png)

10. 選取 **[是]** 以啟用警示規則。

## <a name="configure-action-groups-for-your-alerts"></a>設定警示的動作群組

設定好警示之後，您就可以設定動作群組，這是可跨多個警示使用的一組動作。 這些動作可能包括電子郵件通知、runbook、webhook 和更多功能。 若要深入了解動作群組，請參閱[建立及管理動作群組](../../azure-monitor/platform/action-groups.md)。

1. 選取警示，然後選取 [動作群組] 下的 [新建]。

2. 輸入動作群組的完整名稱和簡短名稱。 當使用指定的群組傳送通知時，更新管理會使用簡短名稱。

3. 在 [動作] 中，輸入指定動作的名稱，例如**電子郵件通知**。

4. 針對 [動作類型]，選取適當的類型，例如 [電子郵件/簡訊/推送/語音]。

5. 選取 [編輯詳細資料]。

6. 在窗格中填入動作類型。 例如，如果使用 [**電子郵件/SMS/推播/語音**]，請輸入動作名稱，選取 [**電子郵件**] 核取方塊，輸入有效的電子郵件地址，然後選取 **[確定]**。

    ![設定電子郵件動作群組](./media/update-mgmt-manage-updates-for-vm/configure-email-action-group.png)

7. 在 [新增動作群組] 窗格中，選取 **[確定]**。

8. 如需警示電子郵件，您可以自訂電子郵件主旨。 選取 [建立規則] 下的 [自訂動作]，然後選取 [電子郵件主旨]。

9. 當您完成時，選取 [建立警示規則]。

## <a name="next-steps"></a>後續步驟


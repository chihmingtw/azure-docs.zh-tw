---
title: 調整伺服器群組-超大規模資料庫 (Citus) -適用於 PostgreSQL 的 Azure 資料庫
description: 調整伺服器群組記憶體、磁片和 CPU 資源，以處理增加的負載
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: how-to
ms.date: 8/10/2020
ms.openlocfilehash: 85a1f0dcc2e778a09cf0d19b2a85d6faf371f032
ms.sourcegitcommit: 1aef4235aec3fd326ded18df7fdb750883809ae8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/12/2020
ms.locfileid: "88134510"
---
# <a name="server-group-size"></a>伺服器群組大小

超大規模資料庫 (Citus) 部署選項會使用協同合作的資料庫伺服器來平行處理查詢執行，並儲存更多資料。 伺服器群組「大小」指的是伺服器數目和各項的硬體資源。

## <a name="picking-initial-size"></a>挑選初始大小

伺服器群組的大小（以節點數目和其硬體容量而言）很容易變更 ([請參閱以下](#scale-a-hyperscale-citus-server-group)) 。 不過，您仍然需要為新的伺服器群組選擇初始大小。 以下是合理選擇的一些秘訣。

### <a name="multi-tenant-saas-use-case"></a>多租使用者 SaaS 使用案例

若要從現有的單一節點于 postgresql 資料庫實例遷移至超大規模資料庫 (Citus) ，建議您選擇一個叢集，其中的工作者虛擬核心和 RAM 數目總計等於原始實例的數量。 在這種情況下，我們看到了2倍的效能改進，因為分區化可改善資源使用率，並允許較小的索引等等。

協調器節點所需的虛擬核心數目取決於您現有的工作負載 (寫入/讀取輸送量) 。 協調器節點不需要像背景工作節點一樣多的 RAM，但是 RAM 配置是根據 vCore 計數來決定，如超大規模資料庫設定) [選項](concepts-hyperscale-configuration-options.md)中所述 (，因此 vCore 計數基本上是真正的決策。

### <a name="real-time-analytics-use-case"></a>即時分析使用案例

Total 虛擬核心：當工作資料符合 RAM 時，您可以預期超大規模資料庫 () Citus 的線性效能改進，與背景工作核心的數目成正比。 若要針對您的需求判斷正確的虛擬核心數目，請考慮單一節點資料庫中查詢的目前延遲，以及超大規模資料庫中的必要延遲 (Citus) 。 將目前的延遲除以所需的延遲，並將結果四捨五入。

背景工作角色 RAM：最好的情況是提供足夠的記憶體，讓大部分的工作中集合能夠容納到記憶體。 您的應用程式所使用的查詢類型會影響記憶體需求。 您可以執行說明查詢的分析，以判斷其所需的記憶體數量。 請記住，虛擬核心和 RAM 會相應縮小，如[超大規模資料庫設定選項](concepts-hyperscale-configuration-options.md)一文所述。

## <a name="scale-a-hyperscale-citus-server-group"></a>調整超大規模資料庫 (Citus) 伺服器群組

適用於 PostgreSQL 的 Azure 資料庫超大規模資料庫 (Citus) 提供自我服務調整，以處理增加的負載。 Azure 入口網站可讓您輕鬆地加入新的背景工作節點，以及增加現有節點的虛擬核心。 新增節點不會造成停機時間，甚至會將分區移至新節點 (稱為[分區重新平衡](#rebalance-shards)) 發生，而不會中斷查詢。

### <a name="add-worker-nodes"></a>新增背景工作節點

若要新增節點，請移至超大規模資料庫 (Citus) 伺服器群組中的 [**設定**] 索引標籤。  拖曳背景**工作節點計數**的滑杆會變更此值。

![資源滑杆](./media/howto-hyperscale-scaling/01-sliders-workers.png)

按一下 [**儲存**] 按鈕，讓變更的值生效。

> [!NOTE]
> 一旦增加並儲存之後，就無法使用滑杆來減少背景工作節點的數目。

#### <a name="rebalance-shards"></a>重新平衡分區

若要利用新加入的節點，您必須重新平衡分散式資料表[分區](concepts-hyperscale-distributed-data.md#shards)，這表示要將某些分區從現有的節點移至新的節點。 請先確認新的背景工作角色已成功完成布建。 然後開機磁碟分割 rebalancer，方法是使用 psql 連接到叢集協調器節點，然後執行：

```sql
SELECT rebalance_table_shards('distributed_table_name');
```

函數會重新平衡其引數中名為之資料表的[共置群組](concepts-hyperscale-colocation.md)中的所有資料表。`rebalance_table_shards` 因此，您不需要針對每個分散式資料表呼叫函數，只要在每個共置群組的代表性資料表上呼叫它即可。

### <a name="increase-or-decrease-vcores-on-nodes"></a>增加或減少節點上的虛擬核心

> [!NOTE]
> 此功能目前為預覽狀態。 若要針對伺服器群組中的節點要求變更虛擬核心，請[洽詢 Azure 支援](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)。

除了新增節點之外，您還可以增加現有節點的功能。 向上和向下調整計算容量，對於效能實驗以及對流量需求的短期或長期變更都很有用。

若要變更所有背景工作節點的虛擬核心，請調整 [**每個背景工作角色節點) 設定 (**] 底下的 [**虛擬核心**] 滑杆。 協調器節點的虛擬核心可以獨立調整。 調整 [設定** (協調器] 節點) **下的 [**虛擬核心**] 滑杆。

## <a name="next-steps"></a>後續步驟

- 深入瞭解伺服器群組[效能選項](concepts-hyperscale-configuration-options.md)。

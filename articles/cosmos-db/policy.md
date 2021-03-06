---
title: 使用 Azure 原則來實作 Azure Cosmos DB 資源的治理和控制
description: 了解如何使用 Azure 原則來實作 Azure Cosmos DB 資源的治理和控制。
author: plzm
ms.author: paelaz
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 05/20/2020
ms.openlocfilehash: a1b1c01f7cf720690decd9c7aac5fb14b92121ec
ms.sourcegitcommit: 877491bd46921c11dd478bd25fc718ceee2dcc08
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/02/2020
ms.locfileid: "84432014"
---
# <a name="use-azure-policy-to-implement-governance-and-controls-for-azure-cosmos-db-resources"></a>使用 Azure 原則來實作 Azure Cosmos DB 資源的治理和控制

[Azure 原則](../governance/policy/overview.md)有助於強制執行組織治理標準、評估資源合規性，以及實作自動補救。 常見的使用案例包括安全性、成本管理和設定一致性。

Azure 原則提供內建原則定義。 您可以針對內建原則定義未解決的案例，建立自訂原則定義。 如需詳細資訊，請參閱 [Azure 原則文件](../governance/policy/overview.md)。

## <a name="assign-a-built-in-policy-definition"></a>指派內建原則定義

原則定義會描述資源合規性條件，以及符合條件時所要採取的效果。 原則_指派_會從原則_定義_來建立。 您可以將內建或自訂原則定義用於您的 Azure Cosmos DB 資源。 原則指派的範圍限於 Azure 管理群組、Azure 訂用帳戶或資源群組，並且會套用至所選範圍內的資源。 您可以選擇性從範圍中排除特定資源。

您可以使用 [Azure 入口網站](../governance/policy/assign-policy-portal.md)、[Azure PowerShell](../governance/policy/assign-policy-powershell.md)、[Azure CLI](../governance/policy/assign-policy-azurecli.md) 或 [ARM 範本](../governance/policy/assign-policy-template.md)來建立原則指派。

若要針對 Azure Cosmos DB 從內建原則定義建立原則指派，請使用[使用 Azure 入口網站建立原則指派](../governance/policy/assign-policy-portal.md)一文中的步驟。

在選取原則定義的步驟中，於 [搜尋] 欄位中輸入 `Cosmos DB`，以篩選可用內建原則定義的清單。 選取其中一個可用內建原則定義，然後選擇 [選取] 以繼續建立原則指派。

> [!TIP]
> 您也可以搭配 Azure PowerShell、Azure CLI 或 ARM 範本使用 [可用定義] 窗格中顯示的內建原則定義名稱，來建立原則指派。

:::image type="content" source="./media/policy/available-definitions.png" alt-text="搜尋 Azure Cosmos DB 內建原則定義":::

## <a name="create-a-custom-policy-definition"></a>建立自訂原則定義

針對內建原則未解決的特定案例，您可以建立[自訂原則定義](../governance/policy/tutorials/create-custom-policy-definition.md)。 稍後您會從自訂原則_定義_建立原則_指派_。

### <a name="property-types-and-property-aliases-in-policy-rules"></a>原則規則中的屬性類型和屬性別名

使用[自訂原則定義步驟](../governance/policy/tutorials/create-custom-policy-definition.md)來識別建立原則規則所需的資源屬性和屬性別名。

若要識別 Azure Cosmos DB 特定屬性別名，請使用命名空間 `Microsoft.DocumentDB`，並搭配自訂原則定義步驟一文所示的其中一個方法。

#### <a name="use-the-azure-cli"></a>使用 Azure CLI：
```azurecli-interactive
# Login first with az login if not using Cloud Shell

# Get Azure Policy aliases for namespace Microsoft.DocumentDB
az provider show --namespace Microsoft.DocumentDB --expand "resourceTypes/aliases" --query "resourceTypes[].aliases[].name"
```

#### <a name="use-azure-powershell"></a>使用 Azure PowerShell：
```azurepowershell-interactive
# Login first with Connect-AzAccount if not using Cloud Shell

# Use Get-AzPolicyAlias to list aliases for Microsoft.DocumentDB namespace
(Get-AzPolicyAlias -NamespaceMatch 'Microsoft.DocumentDB').Aliases
```

這些命令會針對 Azure Cosmos DB 屬性輸出屬性別名名稱的清單。 以下是輸出的摘錄：

```json
[
  "Microsoft.DocumentDB/databaseAccounts/sku.name",
  "Microsoft.DocumentDB/databaseAccounts/virtualNetworkRules[*]",
  "Microsoft.DocumentDB/databaseAccounts/virtualNetworkRules[*].id",
  "Microsoft.DocumentDB/databaseAccounts/isVirtualNetworkFilterEnabled",
  "Microsoft.DocumentDB/databaseAccounts/consistencyPolicy.defaultConsistencyLevel",
  "Microsoft.DocumentDB/databaseAccounts/enableAutomaticFailover",
  "Microsoft.DocumentDB/databaseAccounts/Locations",
  "Microsoft.DocumentDB/databaseAccounts/Locations[*]",
  "Microsoft.DocumentDB/databaseAccounts/Locations[*].locationName",
  "..."
]
```

您可以使用[自訂原則定義規則](../governance/policy/tutorials/create-custom-policy-definition.md#policy-rule)中的任何屬性別名名稱。

以下是原則定義的範例，它會檢查是否已針對多個寫入位置設定 Azure Cosmos DB 帳戶。 自訂原則定義包含兩個規則：一個用來檢查特定類型的屬性別名，另一個則用於類型的特定屬性，在此案例中是儲存多個寫入位置設定的欄位。 這兩個規則都會使用別名名稱。

```json
"policyRule": {
  "if": {
    "allOf": [
      {
        "field": "type",
        "equals": "Microsoft.DocumentDB/databaseAccounts"
      },
      {
        "field": "Microsoft.DocumentDB/databaseAccounts/enableMultipleWriteLocations",
        "notEquals": true
      }
    ]
  },
  "then": {
    "effect": "Audit"
  }
}
```

自訂原則定義可以用來建立原則指派，就像使用內建原則定義一樣。

## <a name="policy-compliance"></a>原則相容性

建立原則指派之後，Azure 原則會評估指派範圍中的資源。 會評估每個資源對於原則的_合規性_。 原則中指定的_效果_接著會套用至不符合規範的資源。

您可以在 [Azure 入口網站](../governance/policy/how-to/get-compliance-data.md#portal)中，或透過 [Azure CLI](../governance/policy/how-to/get-compliance-data.md#command-line) 或 [Azure 監視器記錄](../governance/policy/how-to/get-compliance-data.md#azure-monitor-logs)，來檢閱合規性結果和補救詳細資料。

下列螢幕擷取畫面顯示兩個範例原則指派。

其中一個指派是以內建原則定義為基礎，會檢查 Azure Cosmos DB 資源是否只部署到允許的 Azure 區域。 資源合規性會針對範圍內的資源顯示原則評估結果（符合規範或不符合規範）。

另一個指派是以自訂原則定義為基礎。 此指派會檢查是否已為多個寫入位置設定 Cosmos DB 帳戶。

部署原則指派之後，合規性儀表板會顯示評估結果。 請注意，部署原則指派之後最多可能需要 30 分鐘的時間。 此外，在建立原則指派之後，[可以立即視需要啟動原則評估掃描](../governance/policy/how-to/get-compliance-data.md#on-demand-evaluation-scan)。

螢幕擷取畫面顯示下列範圍內 Azure Cosmos DB 帳戶的相容性評估結果：

- 兩個帳戶中有零個符合必須設定虛擬網路（VNet）篩選的原則。
- 兩個帳戶中有零個符合原則，需要為多個寫入位置設定帳戶
- 兩個帳戶中有零個符合原則，即資源已部署到允許的 Azure 區域。

:::image type="content" source="./media/policy/compliance.png" alt-text="列出的 Azure 原則指派的相容性結果":::

若要修復不符合規範的資源，請參閱[如何使用 Azure 原則補救資源](../governance/policy/how-to/remediate-resources.md)。

## <a name="next-steps"></a>後續步驟

- 請[參閱 Azure Cosmos DB 的自訂原則定義範例](https://github.com/Azure/azure-policy/tree/master/samples/CosmosDB)，包括以上所示的多個寫入位置和 VNet 篩選原則。
- [在 Azure 入口網站中建立原則指派](../governance/policy/assign-policy-portal.md)
- [檢閱適用於 Azure Cosmos DB 的 Azure 原則內建原則定義](./policy-samples.md)

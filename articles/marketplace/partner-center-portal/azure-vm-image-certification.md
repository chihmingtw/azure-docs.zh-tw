---
title: Azure 虛擬機器映射驗證-Azure Marketplace
description: 了解如何在商業 Marketplace 中測試並提交虛擬機器供應項目。
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
author: iqshahmicrosoft
ms.author: iqshah
ms.date: 08/14/2020
ms.openlocfilehash: fd8f41f88b6184eee15477c460dc9d2e521d25e6
ms.sourcegitcommit: d7352c07708180a9293e8a0e7020b9dd3dd153ce
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2020
ms.locfileid: "89144143"
---
# <a name="azure-virtual-machine-image-validation"></a>Azure 虛擬機器映射驗證

此文章描述如何在商業 Marketplace 中測試並提交虛擬機器 (VM) 映像，以確保其符合最新的 Azure Marketplace 發佈需求。

提交您的 VM 供應項目之前，請先完成下列步驟：

- 使用一般化映像部署 Azure VM。
- 執行驗證。

## <a name="deploy-an-azure-vm-using-your-generalized-image"></a>使用一般化映像部署 Azure VM

本節說明如何部署一般化虛擬硬碟 (VHD) 映射，以建立新的 Azure VM 資源。 在此程式中，我們將使用提供的 Azure Resource Manager 範本和 Azure PowerShell 腳本。

### <a name="prepare-an-azure-resource-manager-template"></a>準備 Azure Resource Manager 範本

本節描述如何建立及部署使用者提供的虛擬機器 (VM) 映像。 您可以從 Azure 部署的虛擬硬碟提供作業系統和資料磁片 VHD 映射來進行這項作業。 這些步驟會使用一般化 VHD 部署 VM。

1. 登入 [Azure 入口網站](https://portal.azure.com/)。
2. 將一般化作業系統 VHD 和資料磁片 Vhd 上傳至您的 Azure 儲存體帳戶。
3. 在首頁上，選取 [ **建立資源**]、搜尋 [範本部署]，然後選取 [ **建立**]。
4. 選擇 [在編輯器中建置自己的範本]。

    :::image type="content" source="media/vm/template-deployment.png" alt-text="顯示範本的選取專案。":::

5. 在編輯器中貼上下列 JSON 範本，然後選取 [ **儲存**]。

```JSON
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "userStorageAccountName": {
            "type": "String"
        },
        "userStorageContainerName": {
            "defaultValue": "vhds",
            "type": "String"
        },
        "dnsNameForPublicIP": {
            "type": "String"
        },
        "adminUserName": {
            "defaultValue": "isv",
            "type": "String"
        },
        "adminPassword": {
            "defaultValue": "",
            "type": "SecureString"
        },
        "osType": {
            "defaultValue": "windows",
            "allowedValues": [
                "windows",
                "linux"
            ],
            "type": "String"
        },
        "subscriptionId": {
            "type": "String"
        },
        "location": {
            "type": "String"
        },
        "vmSize": {
            "type": "String"
        },
        "publicIPAddressName": {
            "type": "String"
        },
        "vmName": {
            "type": "String"
        },
        "virtualNetworkName": {
            "type": "String"
        },
        "nicName": {
            "type": "String"
        },
        "vhdUrl": {
            "type": "String",
            "metadata": {
                "description": "VHD Url..."
            }
        }
    },
    "variables": {
        "addressPrefix": "10.0.0.0/16",
        "subnet1Name": "Subnet-1",
        "subnet2Name": "Subnet-2",
        "subnet1Prefix": "10.0.0.0/24",
        "subnet2Prefix": "10.0.1.0/24",
        "publicIPAddressType": "Dynamic",
        "vnetID": "[resourceId('Microsoft.Network/virtualNetworks',parameters('virtualNetworkName'))]",
        "subnet1Ref": "[concat(variables('vnetID'),'/subnets/',variables('subnet1Name'))]",
        "hostDNSNameScriptArgument": "[concat(parameters('dnsNameForPublicIP'),'.',parameters('location'),'.cloudapp.azure.com')]",
        "osDiskVhdName": "[concat('http://',parameters('userStorageAccountName'),'.blob.core.windows.net/',parameters('userStorageContainerName'),'/',parameters('vmName'),'osDisk.vhd')]"
    },
    "resources": [
        {
            "type": "Microsoft.Network/publicIPAddresses",
            "apiVersion": "2015-06-15",
            "name": "[parameters('publicIPAddressName')]",
            "location": "[parameters('location')]",
            "properties": {
                "publicIPAllocationMethod": "[variables('publicIPAddressType')]",
                "dnsSettings": {
                    "domainNameLabel": "[parameters('dnsNameForPublicIP')]"
                }
            }
        },
        {
            "type": "Microsoft.Network/virtualNetworks",
            "apiVersion": "2015-06-15",
            "name": "[parameters('virtualNetworkName')]",
            "location": "[parameters('location')]",
            "properties": {
                "addressSpace": {
                    "addressPrefixes": [
                        "[variables('addressPrefix')]"
                    ]
                },
                "subnets": [
                    {
                        "name": "[variables('subnet1Name')]",
                        "properties": {
                            "addressPrefix": "[variables('subnet1Prefix')]"
                        }
                    },
                    {
                        "name": "[variables('subnet2Name')]",
                        "properties": {
                            "addressPrefix": "[variables('subnet2Prefix')]"
                        }
                    }
                ]
            }
        },
        {
            "type": "Microsoft.Network/networkInterfaces",
            "apiVersion": "2015-06-15",
            "name": "[parameters('nicName')]",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[concat('Microsoft.Network/publicIPAddresses/', parameters('publicIPAddressName'))]",
                "[concat('Microsoft.Network/virtualNetworks/', parameters('virtualNetworkName'))]"
            ],
            "properties": {
                "ipConfigurations": [
                    {
                        "name": "ipconfig1",
                        "properties": {
                            "privateIPAllocationMethod": "Dynamic",
                            "publicIPAddress": {
                                "id": "[resourceId('Microsoft.Network/publicIPAddresses',parameters('publicIPAddressName'))]"
                            },
                            "subnet": {
                                "id": "[variables('subnet1Ref')]"
                            }
                        }
                    }
                ]
            }
        },
        {
            "type": "Microsoft.Compute/virtualMachines",
            "apiVersion": "2015-06-15",
            "name": "[parameters('vmName')]",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[concat('Microsoft.Network/networkInterfaces/', parameters('nicName'))]"
            ],
            "properties": {
                "hardwareProfile": {
                    "vmSize": "[parameters('vmSize')]"
                },
                "osProfile": {
                    "computername": "[parameters('vmName')]",
                    "adminUsername": "[parameters('adminUsername')]",
                    "adminPassword": "[parameters('adminPassword')]"
                },
                "storageProfile": {
                    "osDisk": {
                        "name": "[concat(parameters('vmName'),'-osDisk')]",
                        "osType": "[parameters('osType')]",
                        "caching": "ReadWrite",
                        "image": {
                            "uri": "[parameters('vhdUrl')]"
                        },
                        "vhd": {
                            "uri": "[variables('osDiskVhdName')]"
                        },
                        "createOption": "FromImage"
                    }
                },
                "networkProfile": {
                    "networkInterfaces": [
                        {
                            "id": "[resourceId('Microsoft.Network/networkInterfaces',parameters('nicName'))]"
                        }
                    ]
                }
            }
        },
        {
            "type": "Microsoft.Compute/virtualMachines/extensions",
            "apiVersion": "2015-06-15",
            "name": "[concat(parameters('vmName'),'/WinRMCustomScriptExtension')]",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[concat('Microsoft.Compute/virtualMachines/', parameters('vmName'))]"
            ],
            "properties": {
                "publisher": "Microsoft.Compute",
                "type": "CustomScriptExtension",
                "typeHandlerVersion": "1.4",
                "settings": {
                    "fileUris": [
                        "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-vm-winrm-windows/ConfigureWinRM.ps1",
                        "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-vm-winrm-windows/makecert.exe",
                        "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-vm-winrm-windows/winrmconf.cmd"
                    ],
                    "commandToExecute": "[concat('powershell -ExecutionPolicy Unrestricted -file ConfigureWinRM.ps1 ',variables('hostDNSNameScriptArgument'))]"
                }
            }
        }
    ]
}
```

<br>

6. 為顯示的自訂部署屬性頁面提供參數值。

| resourceGroupName | 現有的 Azure 資源群組名稱。 通常會使用與您金鑰保存庫相同的 RG。 |
| --- | --- |
| TemplateFile | 檔案 VHDtoImage.json 的完整路徑名稱。 |
| userStorageAccountName | 儲存體帳戶的名稱。 |
| dnsNameForPublicIP | 公用 IP 的 DNS 名稱；必須小寫。 |
| subscriptionId | Azure 訂用帳戶識別碼。 |
| Location | 資源群組的標準 Azure 地理位置。 |
| vmName | 虛擬機器的名稱。 |
| vhdUrl | 虛擬硬碟的網址。 |
| vmSize | 虛擬機器執行個體的大小。 |
| publicIPAddressName | 公用 IP 位址的名稱。 |
| virtualNetworkName | 虛擬網路的名稱。 |
| nicName | 適用於虛擬網路的網路介面卡名稱。 |
| adminUserName | 系統管理員帳戶的使用者名稱。 |
| adminPassword | 系統管理員密碼。 |
|

7. 在提供這些值之後，請選取 [購買]。

Azure 將會開始部署。 其會在指定的儲存體帳戶路徑中，使用指定的非受控 VHD 來建立新 VM。 您可在 Azure 入口網站中選取左側的 [虛擬機器] 來追蹤進度。 建立 VM 後，狀態會從 [正在啟動] 變更為 [正在執行]。

#### <a name="for-generation-2-vm-deployment-use-this-template"></a>針對第2代 VM 部署，請使用此範本：

```JSON
{
   "$schema":"https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
   "contentVersion":"1.0.0.0",
   "parameters":{
      "userStorageAccountName":{
         "type":"String"
      },
      "userStorageContainerName":{
         "type":"String",
         "defaultValue":"vhds-86350-720-f4efbbb2-611b-4cd7-ad1e-afdgfuadfluad"
      },
      "dnsNameForPublicIP":{
         "type":"String"
      },
      "adminUserName":{
         "defaultValue":"isv",
         "type":"String"
      },
      "adminPassword":{
         "type":"securestring",
         "defaultValue":"Certcare@86350"
      },
      "osType":{
         "type":"string",
         "defaultValue":"linux",
         "allowedValues":[
            "windows",
            "linux"
         ]
      },
      "subscriptionId":{
         "type":"string"
      },
      "location":{
         "type":"string"
      },
      "vmSize":{
         "type":"string"
      },
      "publicIPAddressName":{
         "type":"string"
      },
      "vmName":{
         "type":"string"
      },
      "vNetNewOrExisting":{
         "defaultValue":"existing",
         "allowedValues":[
            "new",
            "existing"
         ],
         "type":"String",
         "metadata":{
            "description":"Specify whether to create a new or existing virtual network for the VM."
         }
      },
      "virtualNetworkName":{
         "type":"String",
         "defaultValue":""
      },
      "SubnetName":{
         "defaultValue":"subnet-5",
         "type":"String"
      },
      "SubnetPrefix":{
         "defaultValue":"10.0.5.0/24",
         "type":"String"
      },
      "nicName":{
         "type":"string"
      },
      "vhdUrl":{
         "type":"string",
         "metadata":{
            "description":"VHD Url..."
         }
      },
      "Datadisk1":{
         "type":"string",
         "metadata":{
            "description":"datadisk1 Url..."
         }
      },
      "Datadisk2":{
         "type":"string",
         "metadata":{
            "description":"datadisk2 Url..."
         }
      },
      "Datadisk3":{
         "type":"string",
         "metadata":{
            "description":"datadisk2 Url..."
         }
      }
   },
   "variables":{
      "addressPrefix":"10.0.0.0/16",
      "subnet1Name":"[parameters('SubnetName')]",
      "subnet1Prefix":"[parameters('SubnetPrefix')]",
      "publicIPAddressType":"Static",
      "vnetID":"[resourceId('Microsoft.Network/virtualNetworks',parameters('virtualNetworkName'))]",
      "subnet1Ref":"[concat(variables('vnetID'),'/subnets/',variables('subnet1Name'))]",
      "osDiskVhdName":"[concat('http://',parameters('userStorageAccountName'),'.blob.core.windows.net/',parameters('userStorageContainerName'),'/',parameters('vmName'),'osDisk.vhd')]",
      "dataDiskVhdName":"[concat('http://',parameters('userStorageAccountName'),'.blob.core.windows.net/',parameters('userStorageContainerName'),'/',parameters('vmName'),'datadisk')]",
      "imageName":"[concat(parameters('vmName'), '-image')]"
   },
   "resources":[
      {
         "apiVersion":"2015-05-01-preview",
         "type":"Microsoft.Network/publicIPAddresses",
         "name":"[parameters('publicIPAddressName')]",
         "location":"[parameters('location')]",
         "properties":{
            "publicIPAllocationMethod":"[variables('publicIPAddressType')]",
            "dnsSettings":{
               "domainNameLabel":"[parameters('dnsNameForPublicIP')]"
            }
         }
      },
      {
         "type":"Microsoft.Compute/images",
         "apiVersion":"2019-12-01",
         "name":"[variables('imageName')]",
         "location":"[parameters('location')]",
         "properties":{
            "storageProfile":{
               "osDisk":{
                  "osType":"[parameters('osType')]",
                  "osState":"Generalized",
                  "blobUri":"[parameters('vhdUrl')]",
                  "storageAccountType":"Standard_LRS"
               },
               "dataDisks":[
                  {
                     "lun":0,
                     "blobUri":"[parameters('Datadisk1')]",
                     "storageAccountType":"Standard_LRS"
                  },
                  {
                     "lun":1,
                     "blobUri":"[parameters('Datadisk2')]",
                     "storageAccountType":"Standard_LRS"
                  },
                  {
                     "lun":2,
                     "blobUri":"[parameters('Datadisk3')]",
                     "storageAccountType":"Standard_LRS"
                  }
               ]
            },
            "hyperVGeneration":"V2"
         }
      },
      {
         "apiVersion":"2015-05-01-preview",
         "type":"Microsoft.Network/virtualNetworks",
         "name":"[parameters('virtualNetworkName')]",
         "location":"[parameters('location')]",
         "properties":{
            "addressSpace":{
               "addressPrefixes":[
                  "[variables('addressPrefix')]"
               ]
            },
            "subnets":[
               {
                  "name":"[variables('subnet1Name')]",
                  "properties":{
                     "addressPrefix":"[variables('subnet1Prefix')]"
                  }
               }
            ]
         },
         "condition":"[equals(parameters('vNetNewOrExisting'), 'new')]"
      },
      {
         "apiVersion":"2016-09-01",
         "type":"Microsoft.Network/networkInterfaces",
         "name":"[parameters('nicName')]",
         "location":"[parameters('location')]",
         "dependsOn":[
            "[concat('Microsoft.Network/publicIPAddresses/', parameters('publicIPAddressName'))]",
            "[concat('Microsoft.Network/virtualNetworks/', parameters('virtualNetworkName'))]"
         ],
         "properties":{
            "ipConfigurations":[
               {
                  "name":"ipconfig1",
                  "properties":{
                     "privateIPAllocationMethod":"Dynamic",
                     "publicIPAddress":{
                        "id":"[resourceId('Microsoft.Network/publicIPAddresses',parameters('publicIPAddressName'))]"
                     },
                     "subnet":{
                        "id":"[variables('subnet1Ref')]"
                     }
                  }
               }
            ]
         }
      },
      {
         "apiVersion":"2019-12-01",
         "type":"Microsoft.Compute/virtualMachines",
         "name":"[parameters('vmName')]",
         "location":"[parameters('location')]",
         "dependsOn":[
            "[concat('Microsoft.Network/networkInterfaces/', parameters('nicName'))]",
            "[variables('imageName')]"
         ],
         "properties":{
            "hardwareProfile":{
               "vmSize":"[parameters('vmSize')]"
            },
            "osProfile":{
               "computername":"[parameters('vmName')]",
               "adminUsername":"[parameters('adminUsername')]",
               "adminPassword":"[parameters('adminPassword')]"
            },
            "storageProfile":{
               "imageReference":{
                  "id":"[resourceId('Microsoft.Compute/images', variables('imageName'))]"
               },
               "dataDisks":[
                  {
                     "name":"[concat(parameters('vmName'),'_DataDisk0')]",
                     "lun":0,
                     "createOption":"FromImage",
                     "managedDisk":{
                        "storageAccountType":"Standard_LRS"
                     }
                  },
                  {
                     "name":"[concat(parameters('vmName'),'_DataDisk1')]",
                     "lun":1,
                     "createOption":"FromImage",
                     "managedDisk":{
                        "storageAccountType":"Standard_LRS"
                     }
                  },
                  {
                     "name":"[concat(parameters('vmName'),'_DataDisk2')]",
                     "lun":2,
                     "createOption":"FromImage",
                     "managedDisk":{
                        "storageAccountType":"Standard_LRS"
                     }
                  }
               ],
               "osDisk":{
                  "name":"[concat(parameters('vmName'),'-OSDisk')]",
                  "createOption":"FromImage",
                  "managedDisk":{
                     "storageAccountType":"Standard_LRS"
                  }
               }
            },
            "networkProfile":{
               "networkInterfaces":[
                  {
                     "id":"[resourceId('Microsoft.Network/networkInterfaces',parameters('nicName'))]"
                  }
               ]
            }
         }
      }
   ]
}
```

### <a name="deploy-an-azure-vm-using-powershell"></a>使用 PowerShell 部署 Azure VM

複製並編輯下列指令碼，以提供 `$storageaccount` 和 `$vhdUrl` 變數的值。 執行它，從您現有的通用 VHD 建立 Azure VM 資源。

```PowerShell
# storage account of existing generalized VHD

$storageaccount = "testwinrm11815" # generalized VHD URL
$vhdUrl = "https://testwinrm11815.blob.core.windows.net/vhds/testvm1234562016651857.vhd"

echo "New-AzResourceGroupDeployment -Name "dplisvvm$postfix" -ResourceGroupName "$rgName" -TemplateFile "C:\certLocation\VHDtoImage.json" -userStorageAccountName "$storageaccount" -dnsNameForPublicIP "$vmName" -subscriptionId "$mysubid" -location "$location" -vmName "$vmName" -vaultName "$kvname" -vaultResourceGroup "$rgName" -certificateUrl
$objAzureKeyVaultSecret.Id -vhdUrl "$vhdUrl" -vmSize "Standard\_A2" -publicIPAddressName "myPublicIP1" -virtualNetworkName "myVNET1" -nicName "myNIC1" -adminUserName "isv" -adminPassword $pwd"

# deploying VM with existing VHD

New-AzResourceGroupDeployment -Name"dplisvvm$postfix" -ResourceGroupName"$rgName" -TemplateFile"C:\certLocation\VHDtoImage.json" - userStorageAccountName"$storageaccount" -dnsNameForPublicIP"$vmName" -subscriptionId"$mysubid" -location"$location" - vmName"$vmName" -vaultName"$kvname" -vaultResourceGroup"$rgName" -certificateUrl$objAzureKeyVaultSecret.Id -vhdUrl"$vhdUrl" - vmSize"Standard\_A2" -publicIPAddressName"myPublicIP1" -virtualNetworkName"myVNET1" -nicName"myNIC1" -adminUserName"isv" - adminPassword$pwd
```

## <a name="run-validations"></a>執行驗證

有兩種方式可以在已部署的映射上執行驗證。

### <a name="use-certification-test-tool-for-azure-certified"></a>使用適用於 Azure 認證的認證測試工具

#### <a name="download-and-run-the-certification-test-tool"></a>下載並執行認證測試工具

Azure 認證的認證測試工具是在本機 Windows 電腦上執行，但可測試 Azure 型 Windows 或 Linux VM。 其會驗證您的使用者 VM 映像是否可以搭配 Microsoft Azure 使用，以及您 VHD 的準備工作是否有依循指導方針並達到要求。

1. 下載並安裝最新版本的 [Azure 認證的認證測試工具](https://www.microsoft.com/download/details.aspx?id=44299)。
2. 開啟認證工具，然後選取 [啟動新測試]。
3. 從 [測試資訊] 畫面，輸入測試回合的測試名稱。
4. 為您的 VM 選取平臺，也就是 **Windows Server** 或 **Linux**。 所選的平台會影響其餘的選項。
5. 如果您的 VM 使用此資料庫服務，請選取 [Azure SQL Database 測試] 核取方塊。

#### <a name="connect-the-certification-tool-to-a-vm-image"></a>將認證工具連線至 VM 映像

1. 選取 [SSH 驗證] 模式：密碼驗證或金鑰檔案驗證。
2. 如果使用密碼型驗證，請輸入 **VM DNS 名稱**、 **使用者名稱**和 **密碼**的值。 您也可以變更預設的 [SSH 連接埠] 號碼。

    :::image type="content" source="media/vm/azure-vm-cert-2.png" alt-text="顯示選取的 VM 測試資訊。":::

3. 如果使用以金鑰檔案為基礎的驗證，請輸入虛擬機器 DNS 名稱、使用者名稱和私密金鑰位置等值。 您也可以加上 [複雜密碼] 或是變更預設的 [SSH 連接埠] 號碼。
4. 輸入完整 VM DNS 名稱 (例如 MyVMName.Cloudapp.net)。
5. 輸入 **使用者名稱** 和 **密碼**。

    :::image type="content" source="media/vm/azure-vm-cert-4.png" alt-text="顯示選取的 VM 使用者名稱和密碼。":::

6. 選取 [下一步]。

#### <a name="run-a-certification-test"></a>執行認證測試

在認證工具中為您的 VM 映射提供參數值之後，請選取 [測試連線]，以建立與您的 VM 的有效連接。 驗證連線後，選取 [下一步] 啟動測試。 當測試完成時，結果會顯示在資料表中。 [狀態] 欄會針對每個測試顯示 (通過/失敗/警告)。 如果任何一項測試失敗，您的映像即未通過認證。 在此情況下，請檢閱需求與失敗訊息、進行建議的變更，然後再執行測試一次。

自動化測試完成之後，請在 [問卷] 畫面的兩個索引標籤 ([一般評估] 和 [核心自訂]) 上，提供 VM 映像的其他相關資訊，然後選取 [下一步]。

最後一個畫面可讓您提供更多的資訊，例如 Linux VM 映射的 SSH 存取訊號，以及如果您要尋找例外狀況的任何失敗評量的說明。

最後，選取 [產生報告]，以下載已執行測試案例的測試結果和記錄檔，以及問卷調查的答案。 將結果儲存在與您的 VHD 相同的容器中。

## <a name="how-to-use-powershell-to-consume-the-self-test-api"></a>如何使用 PowerShell 來使用自我測試 API

### <a name="on-linux-os"></a>在 Linux OS 上

在 PowerShell 中呼叫 API：

1. 使用 WebRequest 命令呼叫 API。
2. 方法為 Post，內容類型為 JSON，如下列程式碼範例和螢幕擷取畫面所示。
3. 指定 JSON 格式的主體參數。

下列範例顯示 PowerShell 對 API 的呼叫：

```POWERSHELL
$accesstoken = “token”
$headers = New-Object “System.Collections.Generic.Dictionary[[String],[String]]”
$headers.Add(“Authorization”, “Bearer $accesstoken”)
$DNSName = “\&lt;\&lt;Machine DNS Name\&gt;\&gt;”
$UserName = “\&lt;\&lt;User ID\&gt;\&gt;”
$Password = “\&lt;\&lt;Password\&gt;\&gt;”
$OS = “Linux”
$PortNo = “22”
$CompanyName = “ABCD”
$AppID = “\&lt;\&lt;Application ID\&gt;\&gt;”
$TenantId = “\&lt;\&lt;Tenant ID\&gt;\&gt;”

$body =
@{
DNSName = $DNSName
UserName = $UserName
Password = $Password
OS = $OS
PortNo = $PortNo
CompanyName = $CompanyName
AppID = $AppID
TenantId = $TenantId
}| ConvertTo-Json

$body

$uri = “URL”

$res = (Invoke-WebRequest -Method “Post” -Uri $uri -Body $body -ContentType “application/json” -Headers $headers).Content
```

<br>以下是在 PowerShell 中呼叫 API 的範例：

[![在 PowerShell 中呼叫 API 的螢幕範例。](media/vm/call-api-in-powershell.png)](media/vm/call-api-in-powershell.png#lightbox)

<br>使用上述範例後，您將可擷取 JSON 並加以剖析，以取得下列詳細資料：

```PowerShell
$resVar=$res|ConvertFrom-Json

$actualresult =$resVar.Response |ConvertFrom-Json

Write-Host”OSName: $($actualresult.OSName)”Write-Host”OSVersion: $($actualresult.OSVersion)”Write-Host”Overall Test Result: $($actualresult.TestResult)”For ($i=0; $i -lt$actualresult.Tests.Length; $i++){ Write-Host”TestID: $($actualresult.Tests[$i].TestID)”Write-Host”TestCaseName: $($actualresult.Tests[$i].TestCaseName)”Write-Host”Description: $($actualresult.Tests[$i].Description)”Write-Host”Result: $($actualresult.Tests[$i].Result)”Write-Host”ActualValue: $($actualresult.Tests[$i].ActualValue)”}
```

<br>此範例畫面會顯示 `$res.Content` 以 JSON 格式顯示測試結果的詳細資料：

[![使用測試結果的詳細資料，在 PowerShell 中呼叫 API 的畫面範例。](media/vm/call-api-in-powershell-details.png)](media/vm/call-api-in-powershell-details.png#lightbox)

<br>以下是在線上 JSON 檢視器中查看的 JSON 測試結果範例 (例如 [Code 美化](https://codebeautify.org/jsonviewer) 或 [JSON 檢視器](https://jsonformatter.org/json-viewer)) 。

![線上 JSON 檢視器中有更多的測試結果。](media/vm/test-results-json-viewer-1.png)

### <a name="on-windows-os"></a>在 Windows 作業系統上

在 PowerShell 中呼叫 API：

1. 使用 WebRequest 命令呼叫 API。
2. 方法為 Post，內容類型為 JSON，如下列程式碼範例和範例畫面所示。
3. 以 JSON 格式建立主體參數。

此程式碼範例會示範 PowerShell 對 API 的呼叫：

```PowerShell
$accesstoken = “Get token for your Client AAD App”$headers = New-Object”System.Collections.Generic.Dictionary[[String],[String]]”$headers.Add(“Authorization”, “Bearer $accesstoken”)$Body = @{ “DNSName” = “XXXX.westus.cloudapp.azure.com”“UserName” = “XXX”“Password” = “XXX@123456”“OS” = “Windows”“PortNo” = “5986”“CompanyName” = “ABCD” “AppID” = “XXXX-XXXX-XXXX” “TenantId” = “XXXX-XXXX-XXXX” } | ConvertTo-Json$res = Invoke-WebRequest -Method”Post” -Uri$uri -Body$Body -ContentType”application/json” –Headers $headers;$Content = $res | ConvertFrom-Json
```

這些範例畫面顯示在 PowerShell 中呼叫 API 的範例：

**使用 SSH 金鑰**：

 :::image type="content" source="media/vm/call-api-with-ssh-key.png" alt-text="使用 SSH 金鑰在 PowerShell 中呼叫 API。":::

**使用密碼**：

 :::image type="content" source="media/vm/call-api-with-password.png" alt-text="使用密碼在 PowerShell 中呼叫 API。":::

<br>使用上述範例後，您將可擷取 JSON 並加以剖析，以取得下列詳細資料：

```PowerShell
$resVar=$res|ConvertFrom-Json

$actualresult =$resVar.Response |ConvertFrom-Json

Write-Host”OSName: $($actualresult.OSName)”Write-Host”OSVersion: $($actualresult.OSVersion)”Write-Host”Overall Test Result: $($actualresult.TestResult)”For ($i=0; $i -lt$actualresult.Tests.Length; $i++){ Write-Host”TestID: $($actualresult.Tests[$i].TestID)”Write-Host”TestCaseName: $($actualresult.Tests[$i].TestCaseName)”Write-Host”Description: $($actualresult.Tests[$i].Description)”Write-Host”Result: $($actualresult.Tests[$i].Result)”Write-Host”ActualValue: $($actualresult.Tests[$i].ActualValue)”}
```

<br>此畫面會顯示 `$res.Content` 以 JSON 格式顯示測試結果的詳細資料：

 :::image type="content" source="media/vm/test-results-json-format.png" alt-text="JSON 格式的測試結果詳細資料。":::

<br>以下是在線上 JSON 檢視器中查看的測試結果範例 (例如 [Code 美化](https://codebeautify.org/jsonviewer) 或 [JSON 檢視器](https://jsonformatter.org/json-viewer)) 。

![在線上 JSON 檢視器中測試結果。](media/vm/test-results-json-viewer-2.png)

## <a name="how-to-use-curl-to-consume-the-self-test-api-on-linux-os"></a>如何使用捲曲在 Linux OS 上使用自我測試 API

以捲曲的方式呼叫 API：

1. 使用 curl 命令呼叫 API。
2. 方法為 Post，內容類型為 JSON，如下列程式碼片段所示。

```JSON
CURL POST -H “Content-Type:application/json”

-H “Authorization: Bearer XXXXXX-Token-XXXXXXXX”

[https://isvapp.azure-api.net/selftest-vm](https://isvapp.azure-api.net/selftest-vm)

-d ‘{ “DNSName”:”XXXX.westus.cloudapp.azure.com”, “UserName”:”XXX”, “Password”:”XXXX@123456”, “OS”:”Linux”, “PortNo”:”22”, “CompanyName”:”ABCD”, “AppId”:”XXXX-XXXX-XXXX”, “TenantId “XXXX-XXXX-XXXX”}’
```

<br>以下是使用捲曲來呼叫 API 的範例：

![使用捲曲來呼叫 API 的範例。](media/vm/use-curl-call-api.png)

<br>以下是來自捲曲呼叫的 JSON 結果：

![來自捲曲呼叫的 JSON 結果。](media/vm/test-results-json-viewer-3.png)

## <a name="next-step"></a>後續步驟

- 讀取 [常見的 SAS URI 問題和修正程式](common-sas-uri-issues.md)。

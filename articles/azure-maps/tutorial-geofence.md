---
title: 教學課程：建立地理柵欄並在Microsoft Azure 地圖服務上追蹤裝置
description: 了解如何設定地理柵欄。 請參閱如何使用 Azure 地圖服務空間服務來追蹤相對於地理柵欄的裝置。
author: anastasia-ms
ms.author: v-stharr
ms.date: 8/11/2020
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: mvc
ms.openlocfilehash: b88d9132ec1548c9d94fc418af35b55ac2836e96
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/11/2020
ms.locfileid: "88121233"
---
# <a name="tutorial-set-up-a-geofence-by-using-azure-maps"></a>教學課程：使用 Azure 地圖服務設定地理柵欄

本教學課程將逐步引導您了解在下列案例的內容中建立和使用 Azure 地圖服務地理柵欄服務的基本概念：

工地管理員必須在設備進入和離開建築區域的周邊時追蹤設備。每當設備離開或進入這些周邊區域時，都會傳送電子郵件通知給 Operations Manager。

Azure 地圖服務提供許多服務，以支援上述案例中在設備進入和離開建築區域時加以追蹤。 在本教學課程中，我們說明如何：

> [!div class="checklist"]
> * 上傳 [Geofencing GeoJSON 資料](geofence-geojson.md)，以定義我們想要監視的工地區域。 我們將使用[資料上傳 API](https://docs.microsoft.com/rest/api/maps/data/uploadpreview)，將地理柵欄當作多邊形座標上傳至您的 Azure 地圖服務帳戶。
> * 設定兩個[邏輯應用程式](https://docs.microsoft.com/azure/event-grid/handler-webhooks#logic-apps)；當觸發時，會在設備進入並離開地理柵欄區域時，傳送電子郵件通知給網站建構的 Operations Manager。
> * 使用 [Azure 事件方格](https://docs.microsoft.com/azure/event-grid/overview)來訂閱 Azure 地圖服務地理柵欄進入和退出事件。 我們將設定兩個 Web Hook 事件訂用帳戶，以呼叫您在兩個 Logic Apps 中定義的 HTTP 端點。 然後，Logic Apps 會在設備移動或進入地理柵欄時，傳送適當的電子郵件通知。
> * 使用 [Search Geofence Get API](https://docs.microsoft.com/rest/api/maps/spatial/getgeofence)，在設備離開並進入地理柵欄區域時接收通知。

## <a name="prerequisites"></a>Prerequisites

1. [建立 Azure 地圖服務帳戶](quick-demo-map-app.md#create-an-azure-maps-account)
2. [取得主要訂用帳戶金鑰](quick-demo-map-app.md#get-the-primary-key-for-your-account)，也稱為主要金鑰或訂用帳戶金鑰。

本教學課程使用 [Postman](https://www.postman.com/) 應用程式，但您可以選擇不同的 API 開發環境。

## <a name="upload-geofencing-geojson-data"></a>上傳 Geofencing GeoJSON 資料

在本教學課程中，我們將上傳包含 `FeatureCollection` 的 Geofencing GeoJSON 資料。 `FeatureCollection` 包含在工地內定義多邊形區域的兩個地理柵欄。 第一個地理柵欄沒有到期時間或限制。 第二個地理柵欄只能在上班時間查詢 (早上 9 點到下午 5 點， 太平洋時區)，且會在 2022 年 1 月 1 日之後失效。 如需關於GeoJSON 格式的詳細資訊，請參閱 [Geofencing GeoJSON 資料](geofence-geojson.md)。

>[!TIP]
>您可以隨時更新您的 Geofencing 資料。 如需如何更新資料的詳細資訊，請參閱[資料上傳 API](https://docs.microsoft.com/rest/api/maps/data/uploadpreview)

1. 開啟 Postman 應用程式。 在 Postman 應用程式頂端處，選取 [新增]。 在 [新建] 視窗中，選取 [集合]。  為集合命名，然後選取 [建立] 按鈕。

2. 若要建立要求，請再次選取 [新增]。 在 [新建] 視窗中，選取 [要求]。 輸入要求的 [要求名稱]。 選取您在先前的步驟中建立的集合，然後選取 [儲存]。

3. 在 [產生器] 索引標籤中選取 **POST** HTTP 方法，然後輸入下列 URL，以將地理柵欄資料上傳至 Azure 地圖服務。 對於此要求以及本文中提及的其他要求，請將 `{Azure-Maps-Primary-Subscription-key}` 取代為您的主要訂用帳戶金鑰。

    ```HTTP
    https://atlas.microsoft.com/mapData/upload?subscription-key={Azure-Maps-Primary-Subscription-key}&api-version=1.0&dataFormat=geojson
    ```

    URL 路徑中的 _geojson_ 參數代表上傳資料的資料格式。

4. 按一下 [本文] 索引標籤。選取 [原始]，然後選取 JSON 作為輸入格式。 將下列 GeoJSON 資料複製並貼到**主體** 文字區域中：

   ```JSON
   {
      "type": "FeatureCollection",
      "features": [
        {
          "type": "Feature",
          "geometry": {
            "type": "Polygon",
            "coordinates": [
              [
                [
                  -122.13393688201903,
                  47.63829579223815
                ],
                [
                  -122.13389128446579,
                  47.63782047131512
                ],
                [
                  -122.13240802288054,
                  47.63783312249837
                ],
                [
                  -122.13238388299942,
                  47.63829037035086
                ],
                [
                  -122.13393688201903,
                  47.63829579223815
                ]
              ]
            ]
          },
          "properties": {
            "geometryId": "1"
          }
        },
        {
          "type": "Feature",
          "geometry": {
            "type": "Polygon",
            "coordinates": [
              [
                [
                  -122.13374376296996,
                  47.63784758098976
                ],
                [
                  -122.13277012109755,
                  47.63784577367854
                ],
                [
                  -122.13314831256866,
                  47.6382813338708
                ],
                [
                  -122.1334782242775,
                  47.63827591198201
                ],
                [
                  -122.13374376296996,
                  47.63784758098976
                ]
              ]
            ]
          },
          "properties": {
            "geometryId": "2",
            "validityTime": {
            "expiredTime": "2022-01-01T00:00:00",
            "validityPeriod": [
                {
                  "startTime": "2020-07-15T16:00:00",
                  "endTime": "2020-07-15T24:00:00",
                  "recurrenceType": "Daily",
                  "recurrenceFrequency": 1,
                  "businessDayOnly": true
                }
              ]
            }
          }
        }
      ]
   }
   ```

5. 按一下藍色的 [傳送] 按鈕，然後等候系統處理要求。 在要求完成後，移至回應的 [標頭] 索引標籤。 複製 [位置] 索引鍵的值，也就是 `status URL`。

    ```http
    https://atlas.microsoft.com/mapData/operations/<operationId>?api-version=1.0
    ```

6. 若要檢查 API 呼叫的狀態，請在 `status URL` 建立 **GET** HTTP 要求。 您必須將主要訂用帳戶金鑰附加至 URL，以進行驗證。 **GET** 要求應會類似於下列 URL：

   ```HTTP
   https://atlas.microsoft.com/mapData/<operationId>/status?api-version=1.0&subscription-key={Subscription-key}
   ```

7. 當 **GET** HTTP 要求成功完成時，會傳回 `resourceLocation`。 `resourceLocation` 也包含已上傳內容的唯一 `udid`。 在本教學課程的最後一節中，您需要儲存此 `udid` 來查詢 Get Geofence API。 您也可以在下一個步驟中使用 `resourceLocation` URL 來擷取此資源中的中繼資料。

      ```json
      {
          "status": "Succeeded",
          "resourceLocation": "https://atlas.microsoft.com/mapData/metadata/{udid}?api-version=1.0"
      }
      ```

8. 若要擷取內容中繼資料，請對您在步驟 7 中擷取的 `resourceLocation` URL 建立 **GET** HTTP 要求。 務必將主要訂用帳戶金鑰附加至 URL，以進行驗證。 **GET** 要求應會類似於下列 URL：

    ```http
   https://atlas.microsoft.com/mapData/metadata/{udid}?api-version=1.0&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

9. 當 **GET** HTTP 要求成功完成時，回應主體會包含步驟 7 的 `resourceLocation` 中指定的 `udid`、未來存取/下載內容的位置，以及一些關於內容的其他中繼資料，例如建立/更新日期、大小等等。 整體回應的範例如下：

    ```json
    {
        "udid": "{udid}",
        "location": "https://atlas.microsoft.com/mapData/{udid}?api-version=1.0",
        "created": "7/15/2020 6:11:43 PM +00:00",
        "updated": "7/15/2020 6:11:45 PM +00:00",
        "sizeInBytes": 1962,
        "uploadStatus": "Completed"
    }
    ```

## <a name="create-logic-app-workflows"></a>建立邏輯應用程式工作流程

在本節中，我們將建立兩個[邏輯應用程式](https://docs.microsoft.com/azure/event-grid/handler-webhooks#logic-apps)端點，以觸發電子郵件通知。 我們將示範如何建立第一個觸發程式，在每次呼叫其端點時，都會傳送電子郵件通知。

1. 登入 [Azure 入口網站](https://portal.azure.com)

2. 按一下 [Azure 入口網站](https://portal.azure.com)左上角的 [建立資源]。

3. 在 [搜尋 Marketplace] 方塊中，輸入 **Logic App**。

4. 從「結果」中選取**邏輯應用程式**。 按一下 [建立] 按鈕。

5. 在 [邏輯應用程式] 頁面上，輸入下列值：
    * 您想要使用於此邏輯應用程式的「訂用帳戶」。
    * 此邏輯應用程式的「資源群組」名稱。 您可以選擇 [建立新的] 或 [使用現有的] 資源群組。
    * 邏輯應用程式的「邏輯應用程式名稱」。 在此情況下，我們會使用 `Equipment-Enter` 做為名稱。

    基於本教學課程的目的，請將其餘的值保留為預設設定。

    :::image type="content" source="./media/tutorial-geofence/logic-app-create.png" alt-text="建立邏輯應用程式":::

6. 按一下 [檢閱 + 建立] 按鈕。 檢閱設定，然後按一下 [建立] 以提交部署。 當部署成功完成時，按一下 [移至資源]。 您將會進入**邏輯應用程式設計工具**

7. 現在，我們將選取觸發程式類型。 向下捲動到「從通用觸發程式開始」*區段。 按一下 [收到 HTTP 要求時]。

     :::image type="content" source="./media/tutorial-geofence/logic-app-trigger.png" alt-text="建立邏輯應用程式 Http 觸發程序":::

8. 按一下位於設計工具右上角的 [儲存]。 系統會自動產生 **HTTP POST URL**。 儲存 URL，您在下一節中需要使用此資訊來建立事件端點。

    :::image type="content" source="./media/tutorial-geofence/logic-app-httprequest.png" alt-text="邏輯應用程式 HTTP 要求 URL 和 JSON":::

9. 選取 [+ 新增步驟] 。 現在請選擇動作。 在搜尋方塊中鍵入 `outlook.com email`。 在 [動作] 清單中向下捲動，然後按一下 [傳送電子郵件 (V2)]。 
  
    :::image type="content" source="./media/tutorial-geofence/logic-app-designer.png" alt-text="建立邏輯應用程式設計工具":::

10. 登入 Outlook.com 帳戶。 請務必按一下 [是] 允許邏輯應用程式存取帳戶。 填寫傳送電子郵件的欄位。

    :::image type="content" source="./media/tutorial-geofence/logic-app-email.png" alt-text="建立邏輯應用程式傳送電子郵件步驟":::

    >[!TIP]
    > 您可以設定邏輯應用程式來讀取事件方格所傳送的資料，來擷取 GeoJSON 回應資料，例如電子郵件通知中的 `geometryId` 或 `deviceId`。 如需如何設定邏輯應用程式以取用並將事件資料傳遞至電子郵件通知的詳細資訊，請參閱[教學課程：使用事件方格和 Logic Apps 來傳送 Azure IoT 中樞事件的相關電子郵件通知](https://docs.microsoft.com/azure/event-grid/publish-iot-hub-events-to-logic-apps)。

11. 按一下「Logic Apps 設計工具」左上角的 [儲存]。

12. 重複步驟 3-11 建立第二個邏輯應用程式，以在設備離開工地時通知管理員。 為邏輯應用程式 `Equipment-Exit` 命名。

## <a name="create-azure-maps-events-subscriptions"></a>建立 Azure 地圖服務事件訂用帳戶

Azure 地圖服務支援三種事件類型。 您可以在[這裡](https://docs.microsoft.com/azure/event-grid/event-schema-azure-maps)查看 Azure 地圖服務支援的事件類型。  我們需要建立兩個不同的事件訂用帳戶：一個用於地理柵欄進入事件，另一個用於地理柵欄退出事件。

請依照下列步驟，建立地理柵欄進入事件的事件訂用帳戶。 您可以用類似的方式重複這些步驟，以訂閱地理柵欄退出事件。

1. 瀏覽至您的 Azure 地圖服務帳戶。 在儀表板中，選取 [訂用帳戶]。 按一下您的訂用帳戶名稱，然後從設定功能表中選取 [事件]。

    :::image type="content" source="./media/tutorial-geofence/events-tab.png" alt-text="瀏覽至 Azure 地圖服務帳戶事件":::

2. 若要建立事件訂用帳戶，請從事件頁面中選取 [+事件訂用帳戶]。

    :::image type="content" source="./media/tutorial-geofence/create-event-subscription.png" alt-text="建立 Azure 地圖服務事件訂用帳戶":::

3. 在 [建立事件訂用帳戶] 頁面上，輸入下列值：
    * 事件訂用帳戶的「名稱」。
    * 「事件結構描述」應該是「事件方格結構描述」。
    * 此事件訂用帳戶的「系統主題名稱」。 在此案例中，我們會使用 `Contoso-Construction`。
    * 在「事件種類篩選條件」中，選擇 `Geofence Entered` 作為事件種類。
    * 在「端點類型」上，選擇 `Web Hook`。
    * 針對「端點」，請複製您在上一節中建立之邏輯應用程式進入端點的 HTTP POST URL。 如果您忘記儲存此資訊，可以直接回到邏輯應用程式設計工具，並從 HTTP 觸發程序步驟複製。

    :::image type="content" source="./media/tutorial-geofence/events-subscription.png" alt-text="Azure 地圖服務事件訂用帳戶的詳細資料":::

4. 按一下 [建立]。

5. 針對您在上一節中建立的邏輯應用程式退出端點，重複步驟 1-4。 在步驟 3 中，請務必選擇 `Geofence Exited` 作為事件種類。

## <a name="use-search-geofence-get-api"></a>使用 Search Geofence Get API

現在，我們將使用 [Search Geofence Get API](https://docs.microsoft.com/rest/api/maps/spatial/getgeofence)，在設備進入或離開地理柵欄時，傳送電子郵件通知給 Operations Manager。

每個設備都有一個 `deviceId`。 在本教學課程中，我們將追蹤單一設備，其唯一識別碼為 `device_1`。

為了清楚起見，下圖顯示裝置在一段時間內的五個位置，從「開始」位置開始，也就是地理柵欄以外的地方。 基於本教學課程的目的，「開始」 位置並未定義，因為我們不會在該位置查詢裝置。

當我們查詢 [Search Geofence Get API](https://docs.microsoft.com/rest/api/maps/spatial/getgeofence) 了解指出初始地理柵欄進入或結束的設備位置時，事件方格會呼叫適當的邏輯應用程式端點，將電子郵件通知傳送至 Operations Manager。

下列各節會使用五個不同的設備位置座標，讓 HTTP GET Geofencing API 發出要求。

![Azure 地圖服務中的地理柵欄地圖](./media/tutorial-geofence/geofence.png)

### <a name="equipment-location-1-47638237-122132483"></a>設備位置 1(47.638237，-122.132483)

1. 在 Postman 應用程式頂端處，選取 [新增]。 在 [新建] 視窗中，選取 [要求]。  輸入要求的 [要求名稱]。 我們將使用「位置 1」的名稱。 選取您在 [上傳Geofencing GeoJSON 資料區段](#upload-geofencing-geojson-data)中建立的集合，然後選取 [儲存]。

2. 在 [建立器] 索引標籤中選取 **GET** HTTP 方法，然後輸入下列 URL，並確定以您的主要訂用帳戶金鑰取代 `{Azure-Maps-Primary-Subscription-key}`，並使用您儲存在[上傳 Geofencing GeoJSON 資料區段](#upload-geofencing-geojson-data)中的 `udid` 取代 `{udid}`。

   ```HTTP
   https://atlas.microsoft.com/spatial/geofence/json?subscription-key={subscription-key}&api-version=1.0&deviceId=device_01&udid={udid}&lat=47.638237&lon=-122.1324831&searchBuffer=5&isAsync=True&mode=EnterAndExit
   ```

3. 按一下 [傳送] 按鈕。 下列 GeoJSON 將會出現在回應視窗中。

    ```json
    {
      "geometries": [
        {
          "deviceId": "device_1",
          "udId": "64f71aa5-bbee-942d-e351-651a6679a7da",
          "geometryId": "1",
          "distance": -999.0,
          "nearestLat": 47.638291,
          "nearestLon": -122.132483
        },
        {
          "deviceId": "device_1",
          "udId": "64f71aa5-bbee-942d-e351-651a6679a7da",
          "geometryId": "2",
          "distance": 999.0,
          "nearestLat": 47.638053,
          "nearestLon": -122.13295
        }
      ],
      "expiredGeofenceGeometryId": [],
      "invalidPeriodGeofenceGeometryId": [],
      "isEventPublished": true
    }
    ```

4. 在上述 GeoJSON 回應中，與主要位置地理柵欄的距離顯示為負數，表示設備位於地理柵欄內。 與子工地地理柵欄的距離顯示為正數，表示設備位於子工地地理柵欄外。 因為這是您第一次將此裝置放在主要網站地理柵欄內，因此 `isEventPublished` 參數會設定為 `true` 而且 Operations Manager 會收到電子郵件通知，表示設備已進入地理柵欄。

### <a name="location-2-4763800-122132531"></a>位置 2 (47.63800，-122.132531)

1. 在 Postman 應用程式頂端處，選取 [新增]。 在 [新建] 視窗中，選取 [要求]。  輸入要求的 [要求名稱]。 我們將使用「位置 2」的名稱。 選取您在 [上傳Geofencing GeoJSON 資料區段](#upload-geofencing-geojson-data)中建立的集合，然後選取 [儲存]。

2. 在 [建立器] 索引標籤中選取 **GET** HTTP 方法，然後輸入下列 URL，並確定以您的主要訂用帳戶金鑰取代 `{Azure-Maps-Primary-Subscription-key}`，並使用您儲存在[上傳 Geofencing GeoJSON 資料區段](#upload-geofencing-geojson-data)中的 `udid` 取代 `{udid}`。

   ```HTTP
   https://atlas.microsoft.com/spatial/geofence/json?subscription-key={subscription-key}&api-version=1.0&deviceId=device_01&udId={udId}&lat=47.63800&lon=-122.132531&searchBuffer=5&isAsync=True&mode=EnterAndExit
   ```

3. 按一下 [傳送] 按鈕。 下列 GeoJSON 將會出現在回應視窗中：

    ```json
    {
      "geometries": [
        {
          "deviceId": "device_01",
          "udId": "64f71aa5-bbee-942d-e351-651a6679a7da",
          "geometryId": "1",
          "distance": -999.0,
          "nearestLat": 47.637997,
          "nearestLon": -122.132399
        },
        {
          "deviceId": "device_01",
          "udId": "64f71aa5-bbee-942d-e351-651a6679a7da",
          "geometryId": "2",
          "distance": 999.0,
          "nearestLat": 47.63789,
          "nearestLon": -122.132809
        }
      ],
      "expiredGeofenceGeometryId": [],
      "invalidPeriodGeofenceGeometryId": [],
      "isEventPublished": false
    }
    ````

4. 在上述的 GeoJSON 回應中，設備仍然在主要位置地理柵欄中，而且尚未進入子位置地理柵欄。 因此，`isEventPublished` 參數會設定為 `false` 而 Operations Manager 不會收到任何電子郵件通知。

### <a name="location-3-4763810783315048-12213336020708084"></a>位置 3 (47.63810783315048，-122.13336020708084)

1. 在 Postman 應用程式頂端處，選取 [新增]。 在 [新建] 視窗中，選取 [要求]。  輸入要求的 [要求名稱]。 我們將使用「位置 3」的名稱。 選取您在 [上傳Geofencing GeoJSON 資料區段](#upload-geofencing-geojson-data)中建立的集合，然後選取 [儲存]。

2. 在 [建立器] 索引標籤中選取 **GET** HTTP 方法，然後輸入下列 URL，並確定以您的主要訂用帳戶金鑰取代 `{Azure-Maps-Primary-Subscription-key}`，並使用您儲存在[上傳 Geofencing GeoJSON 資料區段](#upload-geofencing-geojson-data)中的 `udid` 取代 `{udid}`。

    ```HTTP
      https://atlas.microsoft.com/spatial/geofence/json?subscription-key={subscription-key}&api-version=1.0&deviceId=device_01&udid={udid}&lat=47.63810783315048&lon=-122.13336020708084&searchBuffer=5&isAsync=True&mode=EnterAndExit
      ```

3. 按一下 [傳送] 按鈕。 下列 GeoJSON 將會出現在回應視窗中：

    ```json
    {
      "geometries": [
        {
          "deviceId": "device_01",
          "udId": "64f71aa5-bbee-942d-e351-651a6679a7da",
          "geometryId": "1",
          "distance": -999.0,
          "nearestLat": 47.638294,
          "nearestLon": -122.133359
        },
        {
          "deviceId": "device_01",
          "udId": "64f71aa5-bbee-942d-e351-651a6679a7da",
          "geometryId": "2",
          "distance": -999.0,
          "nearestLat": 47.638161,
          "nearestLon": -122.133549
        }
      ],
      "expiredGeofenceGeometryId": [],
      "invalidPeriodGeofenceGeometryId": [],
      "isEventPublished": true
    }
    ````

4. 在上述的 GeoJSON 回應中，設備仍然在主要位置地理柵欄中，但已進入子位置地理柵欄。 因此，`isEventPublished` 參數會設定為 `true`，而 Operations Manager 會收到電子郵件通知，指出設備已進入地理柵欄。

    >[!NOTE]
    >如果設備在上班時間後已移至子位置，則不會發佈任何事件，且 Operations Manager 不會收到任何通知。  

### <a name="location-4-47637988-1221338344"></a>位置 4 (47.637988，-122.1338344)

1. 在 Postman 應用程式頂端處，選取 [新增]。 在 [新建] 視窗中，選取 [要求]。  輸入要求的 [要求名稱]。 我們將使用「位置 4」的名稱。 選取您在 [上傳Geofencing GeoJSON 資料區段](#upload-geofencing-geojson-data)中建立的集合，然後選取 [儲存]。

2. 在 [建立器] 索引標籤中選取 **GET** HTTP 方法，然後輸入下列 URL，並確定以您的主要訂用帳戶金鑰取代 `{Azure-Maps-Primary-Subscription-key}`，並使用您儲存在[上傳 Geofencing GeoJSON 資料區段](#upload-geofencing-geojson-data)中的 `udid` 取代 `{udid}`。

    ```HTTP
    https://atlas.microsoft.com/spatial/geofence/json?subscription-key={subscription-key}&api-version=1.0&deviceId=device_01&udid={udid}&lat=47.637988&userTime=2023-01-16&lon=-122.1338344&searchBuffer=5&isAsync=True&mode=EnterAndExit
    ```

3. 按一下 [傳送] 按鈕。 下列 GeoJSON 將會出現在回應視窗中：

    ```json
    {
      "geometries": [
        {
          "deviceId": "device_01",
          "udId": "64f71aa5-bbee-942d-e351-651a6679a7da",
          "geometryId": "1",
          "distance": -999.0,
          "nearestLat": 47.637985,
          "nearestLon": -122.133907
        }
      ],
      "expiredGeofenceGeometryId": [
        "2"
      ],
      "invalidPeriodGeofenceGeometryId": [],
      "isEventPublished": false
    }
    ````

4. 在上述的 GeoJSON 回應中，設備仍然在主要位置地理柵欄中，但已離開子位置地理柵欄。 不過，如果您有注意到，`userTime` 值會在 `expiredTime` (如地理柵欄資料中所定義) 之後。 因此，`isEventPublished` 參數會設定為 `false` 而 Operations Manager 不會收到電子郵件通知。

### <a name="location-547637988-1221338344"></a>位置 5 (47.637988，-122.1338344)

1. 在 Postman 應用程式頂端處，選取 [新增]。 在 [新建] 視窗中，選取 [要求]。  輸入要求的 [要求名稱]。 我們將使用「位置 4」的名稱。 選取您在 [上傳Geofencing GeoJSON 資料區段](#upload-geofencing-geojson-data)中建立的集合，然後選取 [儲存]。

2. 在 [建立器] 索引標籤中選取 **GET** HTTP 方法，然後輸入下列 URL，並確定以您的主要訂用帳戶金鑰取代 `{Azure-Maps-Primary-Subscription-key}`，並使用您儲存在[上傳 Geofencing GeoJSON 資料區段](#upload-geofencing-geojson-data)中的 `udid` 取代 `{udid}`。

    ```HTTP
    https://atlas.microsoft.com/spatial/geofence/json?subscription-key={subscription-key}&api-version=1.0&deviceId=device_01&udid={udid}&lat=47.637988&lon=-122.1338344&searchBuffer=5&isAsync=True&mode=EnterAndExit
    ```

3. 按一下 [傳送] 按鈕。 下列 GeoJSON 將會出現在回應視窗中：

    ```json
    {
      "geometries": [
      {
        "deviceId": "device_01",
        "udId": "64f71aa5-bbee-942d-e351-651a6679a7da",
        "geometryId": "1",
        "distance": -999.0,
        "nearestLat": 47.637985,
        "nearestLon": -122.133907
      },
      {
        "deviceId": "device_01",
        "udId": "64f71aa5-bbee-942d-e351-651a6679a7da",
        "geometryId": "2",
        "distance": 999.0,
        "nearestLat": 47.637945,
        "nearestLon": -122.133683
      }
      ],
      "expiredGeofenceGeometryId": [],
      "invalidPeriodGeofenceGeometryId": [],
      "isEventPublished": true
    }
    ````

4. 在上述的 GeoJSON 回應中，設備已離開主要位置地理柵欄。 因此，`isEventPublished` 參數會設定為 `true`，而 Operations Manager 會收到電子郵件通知，指出設備已離開地理柵欄。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [在 Azure 邏輯應用程式中處理內容類型](https://docs.microsoft.com/azure/logic-apps/logic-apps-content-type)

> [!div class="nextstepaction"]
> [使用事件方格和 Logic Apps 來傳送電子郵件通知](https://docs.microsoft.com/azure/event-grid/publish-iot-hub-events-to-logic-apps)

> [!div class="nextstepaction"]
> [事件方格中支援的事件處理常式](https://docs.microsoft.com/azure/event-grid/event-handlers)。

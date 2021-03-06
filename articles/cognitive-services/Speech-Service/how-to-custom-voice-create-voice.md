---
title: 建立自訂語音語音服務
titleSuffix: Azure Cognitive Services
description: 當您準備好上傳資料時，請移至自訂語音入口網站。 建立或選取自訂語音專案。 專案必須與您要用於語音訓練的資料共用正確的語言/地區設定和性別屬性。
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 11/04/2019
ms.author: erhopf
ms.openlocfilehash: 5f087a2880c16218905a4410a2f591511a155ffd
ms.sourcegitcommit: d7fba095266e2fb5ad8776bffe97921a57832e23
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/09/2020
ms.locfileid: "84629003"
---
# <a name="create-a-custom-voice"></a>建立自訂語音

在[準備自訂語音的資料](how-to-custom-voice-prepare-data.md)中，我們說明了您可以用來定型自訂語音和不同格式需求的不同資料類型。 準備好您的資料之後，您可以開始將它們上傳到[自訂語音入口網站](https://aka.ms/custom-voice-portal)，或透過自訂語音訓練 API。 我們將在此說明透過入口網站訓練自訂語音的步驟。

> [!NOTE]
> 此頁面假設您已閱讀[開始使用自訂語音](how-to-custom-voice.md)和[準備自訂語音的資料](how-to-custom-voice-prepare-data.md)，並已建立自訂語音專案。

請檢查自訂語音：語言所支援的語言[以進行自訂](language-support.md#customization)。

## <a name="upload-your-datasets"></a>上傳您的資料集

當您準備好上傳資料時，請移至[自訂語音入口網站](https://aka.ms/custom-voice-portal)。 建立或選取自訂語音專案。 專案必須與您要用於語音訓練的資料共用正確的語言/地區設定和性別屬性。 例如， `en-GB` 如果您的音訊錄音是以英國輔的英文來完成，請選取。

移至 [**資料**] 索引標籤，然後按一下 [**上傳資料**]。 在嚮導中，選取符合您已備妥之專案的正確資料類型。

您上傳的每個資料集都必須符合所選資料類型的需求。 在您的資料上傳之前，請務必正確地設定其格式。 這可確保自訂語音服務會正確地處理資料。 移至 [[準備資料以進行自訂語音](how-to-custom-voice-prepare-data.md)]，並確定您的資料已然而格式化。

> [!NOTE]
> 免費訂用帳戶（F0）使用者可以同時上傳兩個資料集。 標準訂用帳戶（S0）使用者可以同時上傳五個資料集。 若達到限制，請等候直到至少其中一個資料集完成匯入。 然後再試一次。

> [!NOTE]
> 「免費訂閱」（F0）使用者和500適用于「標準訂用帳戶」（S0）使用者，每個訂用帳戶允許匯入的資料集數目上限為10個 .zip 檔案。

一旦您按了 [上傳] 按鈕，就會自動驗證資料集。 資料驗證封裝含音訊檔案的一系列檢查，以確認其檔案格式、大小和取樣率。 修正錯誤（如果有的話），然後再次提交。 成功起始資料匯入要求時，您應該會在資料表中看到對應于您剛才上傳之資料集的專案。

下表顯示所匯入資料集的處理狀態：

| 州 | 意義 |
| ----- | ------- |
| Processing | 已收到您的資料集，而且正在處理中。 |
| 成功 | 您的資料集已經過驗證，現在可能用來建立語音模型。 |
| Failed | 您的資料集在處理期間因許多原因而失敗，例如檔案錯誤、資料問題或網路問題。 |

驗證完成之後，您可以在 [**語句**] 資料行中看到每個資料集的相符語句總數。 如果您選取的資料類型需要長音訊分割，則此欄只會根據您的文字記錄或透過語音轉譯服務，來反映我們為您分割的語句。 您可以進一步下載已驗證的資料集，以查看成功匯入語句的詳細結果及其對應文字記錄。 提示：長音訊分割可能需要超過一小時的時間才能完成資料處理。

針對 en-us 和 zh-CN 資料集，您可以進一步下載報表來檢查每個錄製的發音分數和雜訊程度。 發音分數介於 0 到 100 的範圍間。 70 以下的分數通常表示語音錯誤或腳本不符。 大量腔調字可能會降低您的發音分數，並影響產生的數位語音。

信噪比 (SNR) 愈高表示您音訊中的雜音愈低。 在專業錄音室中，通常可達到 50+ SNR。 SNR 低於 20 的音訊可能會在您產生的語音中導致明顯的雜音。

請考慮以較低的發音分數或較差的信噪比來重新錄製任何語句。 若無法重新錄製，您可以從資料集中排除這些語句。

## <a name="build-your-custom-voice-model"></a>建立您的自訂語音模型

驗證資料集之後，您就可以使用它來建立自訂語音模型。

1.  流覽至**文字轉換語音 > 自訂語音 > [專案名稱] > 訓練**。

2.  按一下 [**定型模型**]。

3.  接下來，輸入 [**名稱**] 和 [**描述**]，以協助您識別此模型。

    請謹慎選擇名稱。 您在這裡輸入的名稱，會是您在語音合成要求中指定語音作為 SSML 輸入一部分所使用的名稱。 只允許字母、數位和一些標點符號字元，例如-、 \_ 和（'，'）。 針對不同的語音模型使用不同的名稱。

    [**描述**] 欄位的常見用法是記錄用來建立模型的資料集名稱。

4.  從 [**選取定型資料**] 頁面中，選擇您想要用於定型的一個或多個資料集。 在提交之前，請先檢查語句的數目。 您可以從任何數目的語句開始，以進行 en-us 和 zh-CN 語音模型。 針對其他地區設定，您必須選取2000以上的語句，才能訓練語音。

    > [!NOTE]
    > 重複的音訊名稱將從訓練中移除。 請確定您選取的資料集不會在多個 .zip 檔案中包含相同的音訊名稱。

    > [!TIP]
    > 品質結果需要使用相同說話者的資料集。 當您提交用於定型的資料集包含的總數小於6000的相異語句時，您會透過統計參數合成技術來訓練您的語音模型。 如果您的定型資料超過6000個相異語句的總數，您將會透過串連合成技術開始進行定型程式。 通常，串連技術可能會導致更自然且更精確的語音結果。 如果您想要使用最新的類神經 TTS 技術來定型模型，以產生相當於可公開使用之[神經語音](language-support.md#neural-voices)的數位語音，[請洽詢自訂語音小組](https://go.microsoft.com/fwlink/?linkid=2108737)。

5.  按一下 [**定型**] 開始建立您的語音模型。

定型資料表會顯示新的專案，並對應到這個新建立的模型。 資料表也會顯示狀態： [處理中]、[成功]、[失敗]。

所顯示的狀態會反映將您的資料集轉換成語音模型的程式，如下所示。

| 州 | 意義 |
| ----- | ------- |
| Processing | 正在建立您的語音模型。 |
| 成功 | 您的語音模型已建立且可以部署。 |
| Failed | 您的語音模型因許多原因而在定型中失敗，例如未察覺的資料問題或網路問題。 |

訓練時間會因所處理的音訊資料量而有所不同。 一般時間範圍為數百個語句 30 分鐘到 20,000 個語句 40 小時。 一旦您的模型定型成功之後，您就可以開始進行測試。

> [!NOTE]
> 免費訂用帳戶（F0）使用者可以同時訓練一個語音字型。 標準訂用帳戶（S0）使用者可以同時訓練三個語音。 若達到限制，請等候直到至少其中一個聲音音調完成訓練，然後再試一次。

> [!NOTE]
> 每個訂用帳戶所允許的語音模型數目上限為10個免費訂閱（F0）使用者的模型，以及標準訂用帳戶（S0）使用者100。

如果您使用的是類神經語音訓練功能，您可以選擇訓練針對即時串流案例優化的模型，或針對非同步[長音訊合成](long-audio-api.md)優化的 HD 類神經模型。  

## <a name="test-your-voice-model"></a>測試您的語音模型

在成功建立聲音音調後，您就可以進行測試，再將它部署以供使用。

1.  流覽至**文字轉換語音 > 自訂語音 > [專案名稱] > 測試**。

2.  按一下 [**加入測試**]。

3.  選取一或多個您想要測試的模型。

4.  提供您想要語音說話的文字。 如果您已選取一次測試多個模型，則會使用相同的文字來測試不同的模型。

    > [!NOTE]
    > 您的文字語言必須與聲音音調語言相同。 只有成功定型的模型可以進行測試。 此步驟僅支援純文字。

5.  按一下 [建立]。

提交測試要求之後，您會回到 [測試] 頁面。 資料表中現在包含與新的要求對應的項目，以及狀態資料行。 合成語音可能需要幾分鐘的時間。 當 [狀態] 欄顯示 [**成功**] 時，您可以播放音訊，或下載文字輸入（.txt 檔案）和音訊輸出（.wav 檔案），並進一步 audition 後者的品質。

您也可以在每個已選取要測試之模型的 [詳細資料] 頁面中找到測試結果。 移至 [**定型**] 索引標籤，然後按一下模型名稱，以輸入模型詳細資料頁面。

## <a name="create-and-use-a-custom-voice-endpoint"></a>建立和使用自訂語音端點

成功建立並測試語言模型之後，您可以將它部署至自訂文字轉換語音端點。 然後，您可以使用此端點取代透過 REST API 提出文字轉換語音要求時的慣用端點。 您的自訂端點只能由您用來部署字型的訂用帳戶呼叫。

若要建立新的自訂語音端點，請移至**文字轉換語音 > 自訂語音 > 部署**。 選取 [**新增端點**]，然後輸入自訂端點的**名稱**和**描述**。 然後選取您想要與此端點產生關聯的自訂語音模型。

當您**按一下 [新增] 按鈕之後**，您會在 [端點] 資料表中看到新端點的專案。 具現化新的端點可能需要幾分鐘的時間。 當部署的狀態為 [**成功**] 時，端點就可供使用。

> [!NOTE]
> 免費訂用帳戶（F0）使用者只能部署一個模型。 標準訂用帳戶（S0）使用者最多可以建立50個端點，每個端點都有自己的自訂語音。

> [!NOTE]
> 若要使用您的自訂語音，您必須指定語音模型名稱、直接在 HTTP 要求中使用自訂 URI，然後使用相同的訂用帳戶來通過 TTS 服務的驗證。

部署端點之後，端點名稱會顯示為連結。 按一下連結以顯示端點的特定資訊，例如端點金鑰、端點 URL 和範例程式碼。

端點的線上測試也可透過自訂語音入口網站使用。 若要測試您的端點，請從**端點詳細資料**頁面選擇 [**檢查端點**]。 [Endpoint Testing] \(端點測試\) 頁面隨即出現。 在文字方塊中輸入要說出的文字（純文字或[SSML 格式](speech-synthesis-markup.md)）。 若要聆聽以您自訂聲音音調說出的文字，請選取 [播放]****。 這項測試功能將根據您的自訂語音合成使用量收費。

自訂端點的功能與用於文字轉換語音要求的標準端點完全相同。 如需詳細資訊，請參閱 [REST API](rest-text-to-speech.md)。

## <a name="next-steps"></a>後續步驟

* [指南：錄製您的語音範例](record-custom-voice-samples.md)
* [文字轉換語音 API 參考](rest-text-to-speech.md)
* [長音訊 API](long-audio-api.md)

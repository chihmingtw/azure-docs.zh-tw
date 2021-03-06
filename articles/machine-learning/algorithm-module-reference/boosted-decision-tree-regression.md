---
title: 促進式決策樹回歸：模組參考
titleSuffix: Azure Machine Learning
description: 瞭解如何使用 Azure Machine Learning 中的「促進式決策樹回歸」模組，利用提升來建立迴歸樹狀結構的集團。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 08/24/2020
ms.openlocfilehash: 171c53aa36ecc854cf32a64f9f2639a7086c24e5
ms.sourcegitcommit: 9c3cfbe2bee467d0e6966c2bfdeddbe039cad029
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2020
ms.locfileid: "88782578"
---
# <a name="boosted-decision-tree-regression-module"></a>促進式決策樹回歸模組

本文說明 Azure Machine Learning 設計工具 (預覽) 中的模組。

您可以使用此模組來建立使用提升的迴歸樹狀結構集團。 *提升* 表示每個樹狀結構都相依于先前的樹狀結構。 演算法的學習方式是在剩餘的樹狀結構上進行調整。 因此，決策樹集團中的促進有助於提高精確度，但稍微有涵蓋範圍不足的風險。  

此模組是以 LightGBM 演算法為基礎。
  
此回歸方法是受監督的學習方法，因此需要已加上 *標籤的資料集*。 標籤資料行必須包含數值。  

> [!NOTE]
> 本單元只適用於使用數值變數的資料集。  

定義模型之後，使用 [定型模型](./train-model.md)來定型模型。

  
## <a name="more-about-boosted-regression-trees"></a>深入瞭解提升迴歸樹  

除了封袋、隨機樹系等等，「促進」也是建立集團模型的傳統方法之一。  在 Azure Machine Learning 中，促進式決策樹使用的是有效的超市漸層提升演算法執行。 梯度提升是一種適用於迴歸問題的機器學習技術。 它會逐步建置迴歸樹，利用預先定義的損失函數來測量每個步驟中的錯誤，並於下一個步驟中更正。 因此，預測模型實際上是一整團的較弱預測模型。  
  
在回歸問題中，以逐步方式提升組建一連串的樹狀結構，然後使用任意 differentiable 損失函數來選取最佳樹狀結構。  
  
如需詳細資訊，請參閱以下文章：  
  
+ [https://wikipedia.org/wiki/Gradient_boosting#Gradient_tree_boosting](https://wikipedia.org/wiki/Gradient_boosting)

    這段有關漸層提升的維琪百科文章會在促進式樹狀結構上提供一些背景。 
  
-  [https://research.microsoft.com/apps/pubs/default.aspx?id=132652](https://research.microsoft.com/apps/pubs/default.aspx?id=132652)  

    Microsoft Research：從 RankNet 到 LambdaRank 到 LambdaMART：概述。 依 J.C。 Burges.

梯度促進方法也可用於分類問題，以適當的遺失函數將問題簡化成迴歸。 如需分類工作之促進式樹狀結構的詳細資訊，請參閱 [二級促進式決策樹](./two-class-boosted-decision-tree.md)。  

## <a name="how-to-configure-boosted-decision-tree-regression"></a>如何設定促進式決策樹回歸

1.  將促進式 **決策樹** 模組新增至您的管線。 您可以在 [**回歸**] 類別之下的**Machine Learning**、 **Initialize**底下找到此模組。 
  
2.  藉由設定 [ **建立定型模式]** 選項，指定您要如何定型模型。  
  
    -   **單一參數**：如果您知道要如何設定模型，請選取此選項，並提供一組特定值做為引數。 
     
    -   **參數範圍**：如果您不確定最佳參數，且想要執行參數清除，請選取此選項。 選取要逐一查看的值範圍，而 [ [微調模型超參數](tune-model-hyperparameters.md) ] 會逐一查看您提供的所有可能設定組合，以判斷產生最佳結果的超參數。    
   
  
3. **每個樹狀結構的分葉數目上限**：指出終端節點的數目上限 (離開可在任何樹狀結構中建立的) 。  

    增加此值，您可能會增加樹狀結構的大小，並取得較佳的精確度，風險是過度配適和較長的定型時間。  

4. **每個分葉節點的樣本數下限**：指出在樹狀結構中建立任何終端節點 (分葉) 所需的最小案例數目。

    藉由增加此值，您會增加建立新規則的臨界值。 例如，若預設值是 1，即使單一案例可能會造成新規則的建立。 如果您將此值增加至 5，則定型資料至少要包含 5 個案例，才會符合相同的條件。

5. **學習速率**：輸入介於0和1之間的數位，以定義學習時的步驟大小。 學習率決定了學習模組在最佳解決方案上的聚合速度或緩慢。 如果步驟的大小太大，您可能會超過最佳的解決方案。 如果步驟的大小太小，則定型需要較長的時間來融合到最佳解決方案。

6. 已建立**的樹狀結構數目**：指出要在集團中建立的決策樹總數目。 藉由建立更多決策樹，您可能會獲得更好的涵蓋範圍，但是定型時間會增加。

    此值也會控制視覺化定型模型時所顯示的樹狀結構數目。 如果您想要查看或列印單一樹狀結構，您可以將此值設定為1。不過， (樹狀結構中只會產生一個樹狀結構，其中包含一組初始的參數) ，且不會執行任何進一步的反覆運算。

7. **亂數字種子**：輸入選擇性的非負整數，以作為隨機的種子值。 指定種子可確保跨回合的重現性具有相同的資料和參數。

    根據預設，隨機種子會設定為0，這表示會從系統時鐘取得初始種子值。
  

9. 定型模型：

    + 如果您將 [ **建立定型模式** ] 設定為 [ **單一參數**]，請連接已標記的資料集和「 [定型模型](train-model.md) 」模組。  
  
    + 如果您將 [ **建立定型模式** ] 設定為 [ **參數範圍**]，請連接已加上標籤的資料集，並使用 [微調模型超參數](tune-model-hyperparameters.md)來定型模型。  
  
    > [!NOTE]
    > 
    > 如果您將參數範圍傳遞給 [定型模型](train-model.md)，它只會使用單一參數清單中的預設值。  
    > 
    > 如果您將一組參數值傳遞至 [微調模型超參數](tune-model-hyperparameters.md) 模組，當它預期每個參數的設定範圍時，會忽略這些值，並使用學習模組的預設值。  
    > 
    > 如果您選取 [ **參數範圍** ] 選項，並輸入任何參數的單一值，則整個清除期間都會使用您所指定的單一值，即使其他參數會在某個範圍的值之間變更。
    

10. 提交管線。  
  
## <a name="results"></a>結果

定型完成後：

+ 若要使用模型進行評分，請將 [定型模型](train-model.md) 連接到 [評分模型](./score-model.md)，以預測新輸入範例的值。

+ 若要儲存定型模型的快照集，請在**定型模型**的右面板中選取 [**輸出**] 索引標籤，然後按一下 [**註冊資料集**] 圖示。 定型模型的複本將會儲存為模組樹狀結構中的模組，而且不會在管線的後續執行期間更新。

## <a name="next-steps"></a>後續步驟

請參閱 Azure Machine Learning 的[可用模組集](module-reference.md)。 

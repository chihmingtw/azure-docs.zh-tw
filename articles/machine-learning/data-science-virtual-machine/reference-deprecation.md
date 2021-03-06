---
title: 參考：資料科學虛擬機器映射 Retirements
titleSuffix: Azure Data Science Virtual Machine
description: 影響 Azure 資料科學虛擬機器的 retirements 詳細資料
author: lobrien
ms.service: machine-learning
ms.subservice: data-science-vm
ms.author: laobri
ms.date: 07/17/2020
ms.topic: reference
ms.openlocfilehash: d5f541dec14eebc944e4eac11dbe569b38cb277e
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2020
ms.locfileid: "89001615"
---
# <a name="reference-retirements-of-dsvm-images"></a>參考： DSVM 影像的 Retirements

以下我們會討論淘汰 (淘汰映射) ，以及處理 Azure 資料科學虛擬機器上即將推出的 retirements 的建議。

## <a name="why-we-retire-dsvm-images"></a>為何要淘汰 DSVM 映射

資料科學虛擬機器目前有兩個版本：

* Ubuntu
* Windows Server

這些版本的 DSVM 映射是以特定的作業系統版本為基礎，例如 Ubuntu 18.04 LTS。 經過一段時間之後，作業系統 _版本_ 的維護期間將會結束，而且/或資料科學工具將不再支援較舊的作業系統版本。 為了讓 DSVM 映射保持最新的作業系統和資料科學工具的最新狀態，我們會根據較新的作業系統版本發行新的 DSVM 映射。

## <a name="how-we-retire-dsvm-images"></a>如何淘汰 DSVM 映射

我們會發出 _淘汰公告_ ，其中現有的 DSVM 使用者會透過電子郵件) 即將淘汰的即將淘汰的電子郵件通知 (。 淘汰公告會詳細說明 _淘汰日期_ (在此日期之後，映射無法使用) 和緩和建議 (例如，升級至較新的 DSVM 映射) 。

在 _公告_ 日期中，我們將隱藏 marketplace 中的映射，以便：

1. 新使用者無法不慎布建已淘汰的 DSVM 映射。
2. 現有的使用者可以使用 ARM 部署，直到淘汰日期為止。

隱藏的映射將會在 _淘汰日期_ 之前接收作業系統修補程式，但 __不__ 會收到資料科學工具和架構的更新。 在 _停用日期_時，將無法使用 ARM 部署的映射。

您訂用帳戶中任何已布建的 DSVM 映射在淘汰日期之後仍會繼續運作。 不過，我們建議您升級至最新的 DSVM 映射，讓您擁有最新的作業系統和資料科學工具。

## <a name="impact"></a>影響

您訂用帳戶中現有的 DSVM 布建映射將會在停用日期之後繼續運作。 不過，我們建議使用者使用 Azure 入口網站或 ARM 範本，將其 DSVM 映射升級至較新的版本。

> [!WARNING]
> 使用虛擬機器擴展集布建的已淘汰 DSVM 映射將無法在停用日期之後擴大。
>
> 未以新 DSVM 映射詳細資料更新的 ARM 範本將無法在停用日期之後進行部署。


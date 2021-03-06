---
title: 自動化服務登錄和探索
description: 瞭解如何使用春季雲端服務登錄來自動化服務探索和註冊
author: bmitchell287
ms.service: spring-cloud
ms.topic: conceptual
ms.date: 10/05/2019
ms.author: brendm
ms.custom: devx-track-java
ms.openlocfilehash: c3e26b157630df6004292c93a0a0a47307d5949a
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/23/2020
ms.locfileid: "87071010"
---
# <a name="discover-and-register-your-spring-cloud-services"></a>探索和註冊您的春天雲端服務

服務探索是以微服務為基礎的架構的關鍵需求。  手動設定每個用戶端需要一些時間，並引進人為錯誤的可能性。  Azure 春季雲端服務登錄可解決此問題。  一旦設定之後，服務登錄伺服器就會控制應用程式微服務的服務註冊和探索。 服務登錄伺服器會維護已部署之微服務的登錄、啟用用戶端負載平衡，以及從用戶端分離服務提供者，而不需要依賴 DNS。

## <a name="register-your-application-using-spring-cloud-service-registry"></a>使用春季雲端服務登錄來註冊您的應用程式

在您的應用程式可以使用春季雲端服務登錄來管理服務註冊和探索之前，應用程式的*pom.xml*檔中必須包含數個相依性。
將*netflix-eureka-用戶端*和春季-雲端*入門*的相依性納入您的*pom.xml*的雲端

```xml
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>spring-cloud-starter-azure-spring-cloud-client</artifactId>
        <version>2.1.0</version>
    </dependency>
```

## <a name="update-the-top-level-class"></a>更新最上層類別

最後，我們會將批註新增至您應用程式的最上層類別

 ```java
    package foo.bar;

    import org.springframework.boot.SpringApplication;
    import org.springframework.cloud.client.SpringCloudApplication;

    @SpringCloudApplication
    public class DemoApplication {
        public static void main(String... args) {
            SpringApplication.run(DemoApplication.class, args);
        }
    }
 ```

春季雲端服務登錄伺服器端點將會插入為您應用程式中的環境變數。  微服務現在可以向服務登錄伺服器註冊自己，並探索其他相依的微服務。

請注意，可能需要幾分鐘的時間，變更才會從伺服器傳播至所有微服務。

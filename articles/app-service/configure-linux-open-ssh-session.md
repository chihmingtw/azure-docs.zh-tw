---
title: 適用于 Linux 容器的 SSH 存取
description: 您可以在 Azure App Service 中開啟 Linux 容器的 SSH 會話。 自訂 Linux 容器可支援自訂映射的一些修改。
keywords: azure app service, web 應用程式, linux, oss
author: msangapu-msft
ms.assetid: 66f9988f-8ffa-414a-9137-3a9b15a5573c
ms.topic: article
ms.date: 02/25/2019
ms.author: msangapu
ms.custom: seodec18
ms.openlocfilehash: d46aacc441b412a117d906e0201a9bed6046a338
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2020
ms.locfileid: "88961600"
---
# <a name="open-an-ssh-session-to-a-linux-container-in-azure-app-service"></a>在 Azure App Service 中開啟 Linux 容器的 SSH 會話

[安全殼層 (SSH)](https://wikipedia.org/wiki/Secure_Shell) 通常用於從命令列終端機，遠端執行系統管理命令。 Linux 上的 App Service 提供應用程式容器的 SSH 支援。 

![Linux App Service SSH](./media/configure-linux-open-ssh-session/app-service-linux-ssh.png)

您也可以使用 SSH 和 SFTP，直接從本機開發電腦連線到容器。

## <a name="open-ssh-session-in-browser"></a>在瀏覽器中開啟 SSH 工作階段

[!INCLUDE [Open SSH session in browser](../../includes/app-service-web-ssh-connect-no-h.md)]

## <a name="use-ssh-support-with-custom-docker-images"></a>使用 SSH 支援搭配自訂 Docker 映像

請參閱 [在自訂容器中設定 SSH](configure-custom-container.md#enable-ssh)。

## <a name="open-ssh-session-from-remote-shell"></a>從遠端殼層開啟 SSH 工作階段

> [!NOTE]
> 此功能目前為預覽狀態。
>

使用 TCP 通道，您可以透過已驗證的 WebSocket 連線，在您的開發電腦與「用於容器的 Web App」之間建立網路連線。 它可讓您從所選的用戶端，開啟您的容器在 App Service 中執行的 SSH 工作階段。

若要開始，您必須安裝 [Azure CLI](/cli/azure/install-azure-cli?view=azure-cli-latest)。 若要查看未安裝 Azure CLI 時的運作方式，請開啟 [Azure Cloud Shell](../cloud-shell/overview.md)。 

使用 [az webapp remote-connection create](/cli/azure/ext/webapp/webapp/remote-connection?view=azure-cli-latest#ext-webapp-az-webapp-remote-connection-create) 命令，開啟您應用程式的遠端連線。 _\<subscription-id>_ _\<group-name>_ \_ \<app-name> 為您的應用程式指定和 _。

```azurecli-interactive
az webapp create-remote-connection --subscription <subscription-id> --resource-group <resource-group-name> -n <app-name> &
```

> [!TIP]
> 命令尾端的 `&` 只是為了方便您使用 Cloud Shell。 它會在背景執行程序，以便您在相同的殼層中執行下一個命令。

命令輸出會為您提供開啟 SSH 工作階段所需的資訊。

```output
Port 21382 is open
SSH is available { username: root, password: Docker! }
Start your favorite client and connect to port 21382
```

使用本機連接埠，以您所選的用戶端開啟包含您的容器的 SSH 工作階段。 下列範例會使用預設 [ssh](https://ss64.com/bash/ssh.html) 命令：

```bash
ssh root@127.0.0.1 -p <port>
```

在出現提示時，輸入 `yes` 繼續連線。 系統會接著提示您輸入密碼。 使用稍早所示的 `Docker!`。

<pre>
Warning: Permanently added '[127.0.0.1]:21382' (ECDSA) to the list of known hosts.
root@127.0.0.1's password:
</pre>

驗證後，您應會看到工作階段歡迎畫面。

<pre>
  _____
  /  _  \ __________ _________   ____
 /  /_\  \___   /  |  \_  __ \_/ __ \
/    |    \/    /|  |  /|  | \/\  ___/
\____|__  /_____ \____/ |__|    \___  &gt;
        \/      \/                  \/
A P P   S E R V I C E   O N   L I N U X

0e690efa93e2:~#
</pre>

您現在已連線到您的連接器。  

嘗試執行 [top](https://ss64.com/bash/top.html) 命令。 您應能夠在程序清單中看到您應用程式的程序。 在下列範例輸出中，它是具有 `PID 263` 的程序。

<pre>
Mem: 1578756K used, 127032K free, 8744K shrd, 201592K buff, 341348K cached
CPU:   3% usr   3% sys   0% nic  92% idle   0% io   0% irq   0% sirq
Load average: 0.07 0.04 0.08 4/765 45738
  PID  PPID USER     STAT   VSZ %VSZ CPU %CPU COMMAND
    1     0 root     S     1528   0%   0   0% /sbin/init
  235     1 root     S     632m  38%   0   0% PM2 v2.10.3: God Daemon (/root/.pm2)
  263   235 root     S     630m  38%   0   0% node /home/site/wwwroot/app.js
  482   291 root     S     7368   0%   0   0% sshd: root@pts/0
45513   291 root     S     7356   0%   0   0% sshd: root@pts/1
  291     1 root     S     7324   0%   0   0% /usr/sbin/sshd
  490   482 root     S     1540   0%   0   0% -ash
45539 45513 root     S     1540   0%   0   0% -ash
45678 45539 root     R     1536   0%   0   0% top
45733     1 root     Z        0   0%   0   0% [init]
45734     1 root     Z        0   0%   0   0% [init]
45735     1 root     Z        0   0%   0   0% [init]
45736     1 root     Z        0   0%   0   0% [init]
45737     1 root     Z        0   0%   0   0% [init]
45738     1 root     Z        0   0%   0   0% [init]
</pre>

## <a name="next-steps"></a>後續步驟

您可以在 [Azure 論壇](/answers/topics/azure-webapps.html)張貼問題和疑難。

如需「用於容器的 Web App」的詳細資訊，請參閱：

* [簡介從 VS Code 在 Azure App Service 上進行 Node.js 應用程式的遠端偵錯](https://medium.com/@auchenberg/introducing-remote-debugging-of-node-js-apps-on-azure-app-service-from-vs-code-in-public-preview-9b8d83a6e1f0)
* [快速入門：在 App Service 上執行自訂容器](quickstart-custom-container.md?pivots=container-linux)
* [在 Linux 上的 Azure App Service 中使用 Ruby](quickstart-ruby.md)
* [Azure App Service 用於容器的 Web App 常見問題](faq-app-service-linux.md)
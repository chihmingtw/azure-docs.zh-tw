---
title: 大量匯出和匯入 Azure 通知中樞註冊 |Microsoft Docs
description: 瞭解如何使用通知中樞大量支援，在通知中樞上執行大量作業，或匯出所有註冊。
services: notification-hubs
author: sethmanheim
manager: femila
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: ''
ms.devlang: ''
ms.topic: article
ms.date: 08/04/2020
ms.author: sethm
ms.reviewer: thsomasu
ms.lastreviewed: 03/18/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: c0771864229c8a3918da076de48fb6e033d2cf5a
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2020
ms.locfileid: "89018173"
---
# <a name="export-and-import-azure-notification-hubs-registrations-in-bulk"></a>大量匯出和匯入 Azure 通知中樞註冊

在某些情況下，您必須在通知中心裡建立或修改大量的註冊。 其中一些案例是在批次計算之後進行標記更新，或是將現有的推播實作為使用 Azure 通知中樞。

本文說明如何在通知中樞上執行大量作業，或大量匯出所有註冊。

## <a name="high-level-flow"></a>高層級流程

批次支援的用途，是要支援牽涉到數百萬項註冊的長時間執行工作。 為了達到此規模，批次支援會使用 Azure 儲存體來儲存工作詳細資料和輸出。 進行大量更新作業時，使用者必須在 Blob 容器中建立一個內容為註冊更新作業清單的檔案。 工作開始時，使用者會先提供輸入 Blob 的 URL，以及輸出目錄的 URL (也是在 Blob 容器中)。 作業啟動之後，使用者可以查詢工作開始時所提供的 URL 位置，以檢查狀態。 特定作業只能執行特定種類的作業， (建立、更新或刪除) 。 匯出作業會以類似的方式執行。

## <a name="import"></a>匯入

### <a name="set-up"></a>設定

本節假設您有下列實體：

- 已佈建的通知中心。
- Azure 儲存體 Blob 容器。
- [Azure 儲存體 NuGet 套件](https://www.nuget.org/packages/windowsazure.storage/)和[通知中樞 nuget 套件](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/)的參考。

### <a name="create-input-file-and-store-it-in-a-blob"></a>建立輸入檔案並將其儲存在 Blob 中

輸入檔案包含以 XML 序列化的註冊清單，每列一個。 下列程式碼範例會使用 Azure SDK 來示範如何序列化註冊，並將其上傳至 blob 容器：

```csharp
private static void SerializeToBlob(CloudBlobContainer container, RegistrationDescription[] descriptions)
{
    StringBuilder builder = new StringBuilder();
    foreach (var registrationDescription in descriptions)
    {
        builder.AppendLine(registrationDescription.Serialize());
    }

    var inputBlob = container.GetBlockBlobReference(INPUT_FILE_NAME);
    using (MemoryStream stream = new MemoryStream(Encoding.UTF8.GetBytes(builder.ToString())))
    {
        inputBlob.UploadFromStream(stream);
    }
}
```

> [!IMPORTANT]
> 前述程式碼會序列化記憶體中的註冊，並將整個資料流上傳至 Blob 中。 如果您上傳的檔案超過幾 mb，請參閱 Azure blob 指引，以瞭解如何執行這些步驟;例如， [區塊 blob](/rest/api/storageservices/Understanding-Block-Blobs--Append-Blobs--and-Page-Blobs)。

### <a name="create-url-tokens"></a>建立 URL 權杖

上傳輸入檔之後，請產生 Url，以針對輸入檔和輸出目錄提供給您的通知中樞。 您可以使用兩個不同的 blob 容器來輸入和輸出。

```csharp
static Uri GetOutputDirectoryUrl(CloudBlobContainer container)
{
    SharedAccessBlobPolicy sasConstraints = new SharedAccessBlobPolicy
    {
        SharedAccessExpiryTime = DateTime.UtcNow.AddDays(1),
        Permissions = SharedAccessBlobPermissions.Write | SharedAccessBlobPermissions.List | SharedAccessBlobPermissions.Read
    };

    string sasContainerToken = container.GetSharedAccessSignature(sasConstraints);
    return new Uri(container.Uri + sasContainerToken);
}

static Uri GetInputFileUrl(CloudBlobContainer container, string filePath)
{
    SharedAccessBlobPolicy sasConstraints = new SharedAccessBlobPolicy
    {
        SharedAccessExpiryTime = DateTime.UtcNow.AddDays(1),
        Permissions = SharedAccessBlobPermissions.Read
    };
    string sasToken = container.GetBlockBlobReference(filePath).GetSharedAccessSignature(sasConstraints);
    return new Uri(container.Uri + "/" + filePath + sasToken);
}
```

### <a name="submit-the-job"></a>提交工作

您現在可以使用兩個輸入和輸出 Url 來啟動批次作業。

```csharp
NotificationHubClient client = NotificationHubClient.CreateClientFromConnectionString(CONNECTION_STRING, HUB_NAME);
var createTask = client.SubmitNotificationHubJobAsync(
new NotificationHubJob {
        JobType = NotificationHubJobType.ImportCreateRegistrations,
        OutputContainerUri = outputContainerSasUri,
        ImportFileUri = inputFileSasUri
    }
);
createTask.Wait();

var job = createTask.Result;
long i = 10;
while (i > 0 && job.Status != NotificationHubJobStatus.Completed)
{
    var getJobTask = client.GetNotificationHubJobAsync(job.JobId);
    getJobTask.Wait();
    job = getJobTask.Result;
    Thread.Sleep(1000);
    i--;
}
```

除了輸入和輸出 Url 以外，此範例也會建立 `NotificationHubJob` 包含物件的物件 `JobType` ，此物件可以是下列其中一種類型：

- `ImportCreateRegistrations`
- `ImportUpdateRegistrations`
- `ImportDeleteRegistrations`

呼叫完成後，通知中樞會繼續作業，而且您可以透過呼叫 [GetNotificationHubJobAsync](/dotnet/api/microsoft.azure.notificationhubs.notificationhubclient.getnotificationhubjobasync?view=azure-dotnet)來檢查其狀態。

工作完成時，您可以檢視輸出目錄中的下列檔案，以檢查結果：

- `/<hub>/<jobid>/Failed.txt`
- `/<hub>/<jobid>/Output.txt`

這些檔案會列出批次作業中的成功和失敗作業。 檔案格式為 `.cvs` ，其中每個資料列都有原始輸入檔的行號，而作業的輸出通常 (建立或更新的註冊描述) 。

### <a name="full-sample-code"></a>完整的範例程式碼

下列範例程式碼會將註冊匯入至通知中樞。

```csharp
using Microsoft.Azure.NotificationHubs;
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.Blob;
using System;
using System.Collections.Generic;
using System.Globalization;
using System.IO;
using System.Linq;
using System.Runtime.Serialization;
using System.Text;
using System.Threading;
using System.Threading.Tasks;
using System.Xml;

namespace ConsoleApplication1
{
    class Program
    {
        private static string CONNECTION_STRING = "---";
        private static string HUB_NAME = "---";
        private static string INPUT_FILE_NAME = "CreateFile.txt";
        private static string STORAGE_ACCOUNT = "---";
        private static string STORAGE_PASSWORD = "---";
        private static StorageUri STORAGE_ENDPOINT = new StorageUri(new Uri("---"));

        static void Main(string[] args)
        {
            var descriptions = new[]
            {
                new MpnsRegistrationDescription(@"http://dm2.notify.live.net/throttledthirdparty/01.00/12G9Ed13dLb5RbCii5fWzpFpAgAAAAADAQAAAAQUZm52OkJCMjg1QTg1QkZDMkUxREQFBlVTTkMwMQ"),
                new MpnsRegistrationDescription(@"http://dm2.notify.live.net/throttledthirdparty/01.00/12G9Ed13dLb5RbCii5fWzpFpAgAAAAADAQAAAAQUZm52OkJCMjg1QTg1QkZDMjUxREQFBlVTTkMwMQ"),
                new MpnsRegistrationDescription(@"http://dm2.notify.live.net/throttledthirdparty/01.00/12G9Ed13dLb5RbCii5fWzpFpAgAAAAADAQAAAAQUZm52OkJCMjg1QTg1QkZDMhUxREQFBlVTTkMwMQ"),
                new MpnsRegistrationDescription(@"http://dm2.notify.live.net/throttledthirdparty/01.00/12G9Ed13dLb5RbCii5fWzpFpAgAAAAADAQAAAAQUZm52OkJCMjg1QTg1QkZDMdUxREQFBlVTTkMwMQ"),
            };

            // Write to blob store to create an input file
            var blobClient = new CloudBlobClient(STORAGE_ENDPOINT, new Microsoft.WindowsAzure.Storage.Auth.StorageCredentials(STORAGE_ACCOUNT, STORAGE_PASSWORD));
            var container = blobClient.GetContainerReference("testjobs");
            container.CreateIfNotExists();

            SerializeToBlob(container, descriptions);

            // TODO then create Sas
            var outputContainerSasUri = GetOutputDirectoryUrl(container);
            var inputFileSasUri = GetInputFileUrl(container, INPUT_FILE_NAME);


            // Import this file
            NotificationHubClient client = NotificationHubClient.CreateClientFromConnectionString(CONNECTION_STRING, HUB_NAME);
            var createTask = client.SubmitNotificationHubJobAsync(
                new NotificationHubJob {
                    JobType = NotificationHubJobType.ImportCreateRegistrations,
                    OutputContainerUri = outputContainerSasUri,
                    ImportFileUri = inputFileSasUri
                }
            );
            createTask.Wait();

            var job = createTask.Result;
            long i = 10;
            while (i > 0 && job.Status != NotificationHubJobStatus.Completed)
            {
                var getJobTask = client.GetNotificationHubJobAsync(job.JobId);
                getJobTask.Wait();
                job = getJobTask.Result;
                Thread.Sleep(1000);
                i--;
            }
        }

        private static void SerializeToBlob(CloudBlobContainer container, RegistrationDescription[] descriptions)
        {
            StringBuilder builder = new StringBuilder();
            foreach (var registrationDescription in descriptions)
            {
                builder.AppendLine(registrationDescription.Serialize());
            }

            var inputBlob = container.GetBlockBlobReference(INPUT_FILE_NAME);
            using (MemoryStream stream = new MemoryStream(Encoding.UTF8.GetBytes(builder.ToString())))
            {
                inputBlob.UploadFromStream(stream);
            }
        }

        static Uri GetOutputDirectoryUrl(CloudBlobContainer container)
        {
            // Set the expiry time and permissions for the container.
            // In this case no start time is specified, so the shared access signature becomes valid immediately.
            SharedAccessBlobPolicy sasConstraints = new SharedAccessBlobPolicy
            {
                SharedAccessExpiryTime = DateTime.UtcNow.AddHours(4),
                Permissions = SharedAccessBlobPermissions.Write | SharedAccessBlobPermissions.List | SharedAccessBlobPermissions.Read
            };

            // Generate the shared access signature on the container, setting the constraints directly on the signature.
            string sasContainerToken = container.GetSharedAccessSignature(sasConstraints);

            // Return the URI string for the container, including the SAS token.
            return new Uri(container.Uri + sasContainerToken);
        }

        static Uri GetInputFileUrl(CloudBlobContainer container, string filePath)
        {
            // Set the expiry time and permissions for the container.
            // In this case no start time is specified, so the shared access signature becomes valid immediately.
            SharedAccessBlobPolicy sasConstraints = new SharedAccessBlobPolicy
            {
                SharedAccessExpiryTime = DateTime.UtcNow.AddHours(4),
                Permissions = SharedAccessBlobPermissions.Read
            };

            // Generate the shared access signature on the container, setting the constraints directly on the signature.
            string sasToken = container.GetBlockBlobReference(filePath).GetSharedAccessSignature(sasConstraints);

            // Return the URI string for the container, including the SAS token.
            return new Uri(container.Uri + "/" + filePath + sasToken);
        }

        static string GetJobPath(string namespaceName, string notificationHubPath, string jobId)
        {
            return string.Format(CultureInfo.InvariantCulture, @"{0}//{1}/{2}/", namespaceName, notificationHubPath, jobId);
        }
    }
}
```

## <a name="export"></a>匯出

註冊的匯出與匯入類似，但有以下差異：

- 您只需要輸出 URL。
- 您可以建立 ExportRegistrations 類型的 NotificationHubJob。

### <a name="sample-code-snippet"></a>範例程式碼片段

以下是在 JAVA 中匯出註冊的範例程式碼片段：

```java
// Submit an export job
NotificationHubJob job = new NotificationHubJob();
job.setJobType(NotificationHubJobType.ExportRegistrations);
job.setOutputContainerUri("container uri with SAS signature");
job = hub.submitNotificationHubJob(job);

// Wait until the job is done
while(true){
    Thread.sleep(1000);
    job = hub.getNotificationHubJob(job.getJobId());
    if(job.getJobStatus() == NotificationHubJobStatus.Completed)
        break;
}

```

## <a name="next-steps"></a>後續步驟

若要深入瞭解註冊，請參閱下列文章：

- [註冊管理](notification-hubs-push-notification-registration-management.md)
- [要進行註冊的標記](notification-hubs-tags-segment-push-message.md)
- [範本註冊](notification-hubs-templates-cross-platform-push-messages.md)

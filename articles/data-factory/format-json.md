---
title: Azure Data Factory 中的 JSON 格式
description: 本主題描述如何在 Azure Data Factory 中處理 JSON 格式。
author: linda33wj
manager: shwang
ms.reviewer: craigg
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 08/10/2020
ms.author: jingwang
ms.openlocfilehash: 2fc2ccdc7a0520bd156bde8c1da36e19a9e2af1e
ms.sourcegitcommit: 1a0dfa54116aa036af86bd95dcf322307cfb3f83
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/10/2020
ms.locfileid: "88042185"
---
# <a name="json-format-in-azure-data-factory"></a>Azure Data Factory 中的 JSON 格式

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

當您想要**剖析 json 檔案，或將資料寫入 json 格式**時，請遵循這篇文章。 

下列連接器支援 JSON 格式： [Amazon S3](connector-amazon-simple-storage-service.md)、 [azure Blob](connector-azure-blob-storage.md)、 [Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md)、 [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md)、 [Azure 檔案儲存體](connector-azure-file-storage.md)、[檔案系統](connector-file-system.md)、 [FTP](connector-ftp.md)、 [Google Cloud Storage](connector-google-cloud-storage.md)、 [HDFS](connector-hdfs.md)、 [HTTP](connector-http.md)和[SFTP](connector-sftp.md)。

## <a name="dataset-properties"></a>資料集屬性

如需可用來定義資料集的區段和屬性完整清單，請參閱[資料集](concepts-datasets-linked-services.md)一文。 本節提供 JSON 資料集所支援的屬性清單。

| 屬性         | 描述                                                  | 必要 |
| ---------------- | ------------------------------------------------------------ | -------- |
| type             | 資料集的類型屬性必須設定為**Json**。 | 是      |
| location         | 檔案 (s) 的位置設定。 每個以檔案為基礎的連接器在下都有自己的位置類型和支援的屬性 `location` 。 **請參閱連接器文章-> 資料集屬性一節中的詳細資訊**。 | 是      |
| encodingName     | 用來讀取/寫入測試檔案的編碼類型。 <br>允許的值如下： "UTF-8"、"UTF-16"、"UTF-UTF-16BE"、"UTF-32"、"32BE"、"US-ASCII"、"UTF-7"、"BIG5"、"EUC-JP"、"EUC-KR"、"GB2312"，"GB18030"，"JOHAB"，"SHIFT-JIS"，"CP875"，"CP866"，"IBM00858"，"IBM037"，"IBM273"，"IBM437"，"IBM500"，"IBM737"，"IBM775"，"IBM850"，"IBM852"，"IBM855"，"IBM857"，"IBM860"，"IBM861"，"IBM863"，"IBM864"，"IBM865"，"IBM869"，"IBM870"，"IBM01140"，"IBM01141"，"IBM01142"，"IBM01143"，"IBM01144"，"IBM01145"，"IBM01146"，"IBM01147"，"IBM01148"，"IBM01149"，"ISO-2022-JP"，"ISO-2022-KR"，"ISO-8859-1"，"ISO-8859-2"，"ISO-8859-3"，"ISO-8859-4"，"ISO-8859-5"，"ISO-8859-6"，"ISO-8859-7"，"iso-8859-8"，"iso-8859-9"，"ISO-8859-13"、「ISO-8859-15」、「WINDOWS-874」、「WINDOWS-1250」、「WINDOWS-1251」、「WINDOWS-1252」、「WINDOWS-1253」、「WINDOWS-1254」、「WINDOWS-1255」、「WINDOWS-1256」、「WINDOWS-1257」、「WINDOWS-1258」。| 否       |
| compression | 用來設定檔案壓縮的屬性群組。 當您想要在活動執行期間執行壓縮/解壓縮時，請設定此區段。 | 否 |
| type<br/> (* `compression`) * | 用來讀取/寫入 JSON 檔案的壓縮編解碼器。 <br>允許的值為**bzip2**、 **gzip**、 **deflate**、 **ZipDeflate**、 **snappy**或**lz4**。 表示在儲存檔案時使用。 預設值為 [未壓縮]。<br>**注意：** 目前的複製活動不支援 "snappy" & "lz4"，而且對應資料流程不支援 "ZipDeflate"。<br>**請注意**，使用複製活動將**ZipDeflate**檔案解壓縮 (s) 並寫入以檔案為基礎的接收資料存放區時，預設會將檔案解壓縮至資料夾： `<path specified in dataset>/<folder named as source zip file>/` ， `preserveZipFileNameAsFolder` 在[複製活動來源](#json-as-source)上使用，以控制是否要將 ZIP 檔案名保留為資料夾結構。 | 否。  |
| 等級<br/> (* `compression`) * | 壓縮比例。 <br>允許的值為**最佳**或**最快**。<br>- **最快速：** 即使產生的檔案未以最佳方式壓縮，壓縮作業也應該儘快完成。<br>- **最佳**：即使作業需要較長的時間才能完成，壓縮作業也應以最佳方式壓縮。 如需詳細資訊，請參閱 [壓縮層級](https://msdn.microsoft.com/library/system.io.compression.compressionlevel.aspx) 主題。 | 否       |

以下是 Azure Blob 儲存體上的 JSON 資料集範例：

```json
{
    "name": "JSONDataset",
    "properties": {
        "type": "Json",
        "linkedServiceName": {
            "referenceName": "<Azure Blob Storage linked service name>",
            "type": "LinkedServiceReference"
        },
        "schema": [ < physical schema, optional, retrievable during authoring > ],
        "typeProperties": {
            "location": {
                "type": "AzureBlobStorageLocation",
                "container": "containername",
                "folderPath": "folder/subfolder",
            },
            "compression": {
                "type": "gzip"
            }
        }
    }
}
```

## <a name="copy-activity-properties"></a>複製活動屬性

如需可用來定義活動的區段和屬性完整清單，請參閱[管線](concepts-pipelines-activities.md)一文。 本節提供 JSON 來源和接收所支援的屬性清單。

瞭解如何從 JSON 檔案解壓縮資料，並對應至接收資料存放區/格式，或從[架構對應](copy-activity-schema-and-type-mapping.md)中反向。

### <a name="json-as-source"></a>JSON 做為來源

複製活動的 [ *** \* 來源 \* *** ] 區段支援下列屬性。

| 屬性      | 描述                                                  | 必要 |
| ------------- | ------------------------------------------------------------ | -------- |
| type          | 複製活動來源的類型屬性必須設定為**JSONSource**。 | 是      |
| formatSettings | 屬性的群組。 請參閱下面的**JSON 讀取設定**表格。 | 否       |
| storeSettings | 一組屬性，說明如何從資料存放區讀取資料。 每個以檔案為基礎的連接器在下都有自己支援的讀取設定 `storeSettings` 。 **請參閱連接器中的詳細資料文章-> 複製活動屬性一節**。 | 否       |

支援的**JSON 讀取設定** `formatSettings` 如下：

| 屬性      | 描述                                                  | 必要 |
| ------------- | ------------------------------------------------------------ | -------- |
| type          | FormatSettings 的類型必須設定為**JsonReadSettings**。 | 是      |
| compressionProperties | 一組屬性，說明如何將指定壓縮編解碼器的資料解壓縮。 | 否       |
| preserveZipFileNameAsFolder<br> (* `compressionProperties`) * | 適用于使用**ZipDeflate**壓縮設定輸入資料集時。 指出是否要在複製期間保留來源 ZIP 檔案名做為資料夾結構。<br>-設定為**true 時 (預設) **，Data Factory 會將解壓縮的檔案寫入 `<path specified in dataset>/<folder named as source zip file>/` 。<br>-設定為**false**時，Data Factory 會將解壓縮的檔案直接寫入至 `<path specified in dataset>` 。 請確定您在不同的來源 zip 檔案中沒有重複的檔案名，以避免發生賽車或非預期的行為。  | 否 |

### <a name="json-as-sink"></a>JSON 做為接收

複製活動的 [ *** \* 接收 \* *** ] 區段支援下列屬性。

| 屬性      | 描述                                                  | 必要 |
| ------------- | ------------------------------------------------------------ | -------- |
| type          | 複製活動來源的類型屬性必須設定為**JSONSink**。 | 是      |
| formatSettings | 屬性的群組。 請參閱下面的**JSON 寫入設定**表格。 | 否       |
| storeSettings | 一組屬性，說明如何將資料寫入資料存放區。 每個以檔案為基礎的連接器在下都有它自己的支援寫入設定 `storeSettings` 。 **請參閱連接器中的詳細資料文章-> 複製活動屬性一節**。 | 否       |

支援的**JSON 寫入設定** `formatSettings` 如下：

| 屬性      | 描述                                                  | 必要                                              |
| ------------- | ------------------------------------------------------------ | ----------------------------------------------------- |
| type          | FormatSettings 的類型必須設定為**JsonWriteSettings**。 | 是                                                   |
| filePattern |表示每個 JSON 檔案中儲存的資料模式。 允許的值為： **setOfObjects** (JSON 行) 和**arrayOfObjects**。 **預設**值為**setOfObjects**。 關於這些模式的詳細資訊，請參閱 [JSON 檔案模式](#json-file-patterns)一節。 |否 |

### <a name="json-file-patterns"></a>JSON 檔案模式

從 JSON 檔案複製資料時，複製活動會自動偵測並剖析下列 JSON 檔案模式。 將資料寫入 JSON 檔案時，您可以在複製活動接收上設定檔案模式。

- **類型 I：setOfObjects**

    每個檔案都包含單一物件、JSON 行或串連的物件。

    * **單一物件 JSON 範例**

        ```json
        {
            "time": "2015-04-29T07:12:20.9100000Z",
            "callingimsi": "466920403025604",
            "callingnum1": "678948008",
            "callingnum2": "567834760",
            "switch1": "China",
            "switch2": "Germany"
        }
        ```

    * **接收) 的 JSON 行 (預設值**

        ```json
        {"time":"2015-04-29T07:12:20.9100000Z","callingimsi":"466920403025604","callingnum1":"678948008","callingnum2":"567834760","switch1":"China","switch2":"Germany"}
        {"time":"2015-04-29T07:13:21.0220000Z","callingimsi":"466922202613463","callingnum1":"123436380","callingnum2":"789037573","switch1":"US","switch2":"UK"}
        {"time":"2015-04-29T07:13:21.4370000Z","callingimsi":"466923101048691","callingnum1":"678901578","callingnum2":"345626404","switch1":"Germany","switch2":"UK"}
        ```

    * **串連的 JSON 範例**

        ```json
        {
            "time": "2015-04-29T07:12:20.9100000Z",
            "callingimsi": "466920403025604",
            "callingnum1": "678948008",
            "callingnum2": "567834760",
            "switch1": "China",
            "switch2": "Germany"
        }
        {
            "time": "2015-04-29T07:13:21.0220000Z",
            "callingimsi": "466922202613463",
            "callingnum1": "123436380",
            "callingnum2": "789037573",
            "switch1": "US",
            "switch2": "UK"
        }
        {
            "time": "2015-04-29T07:13:21.4370000Z",
            "callingimsi": "466923101048691",
            "callingnum1": "678901578",
            "callingnum2": "345626404",
            "switch1": "Germany",
            "switch2": "UK"
        }
        ```

- **類型 II：arrayOfObjects**

    每個檔案都會包含物件的陣列。

    ```json
    [
        {
            "time": "2015-04-29T07:12:20.9100000Z",
            "callingimsi": "466920403025604",
            "callingnum1": "678948008",
            "callingnum2": "567834760",
            "switch1": "China",
            "switch2": "Germany"
        },
        {
            "time": "2015-04-29T07:13:21.0220000Z",
            "callingimsi": "466922202613463",
            "callingnum1": "123436380",
            "callingnum2": "789037573",
            "switch1": "US",
            "switch2": "UK"
        },
        {
            "time": "2015-04-29T07:13:21.4370000Z",
            "callingimsi": "466923101048691",
            "callingnum1": "678901578",
            "callingnum2": "345626404",
            "switch1": "Germany",
            "switch2": "UK"
        }
    ]
    ```

## <a name="mapping-data-flow-properties"></a>對應資料流程屬性

在對應資料流程中，您可以在下列資料存放區中讀取和寫入 JSON 格式： [Azure Blob 儲存體](connector-azure-blob-storage.md#mapping-data-flow-properties)、 [Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md#mapping-data-flow-properties)和[Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md#mapping-data-flow-properties)。

### <a name="source-properties"></a>來源屬性

下表列出 json 來源所支援的屬性。 您可以在 [**來源選項**] 索引標籤中編輯這些屬性。

| 名稱 | 描述 | 必要 | 允許的值 | 資料流程腳本屬性 |
| ---- | ----------- | -------- | -------------- | ---------------- |
| 萬用字元路徑 | 所有符合萬用字元路徑的檔案都會被處理。 覆寫資料集中所設定的資料夾和檔案路徑。 | 否 | String[] | wildcardPaths |
| 分割區根路徑 | 對於已分割的檔案資料，您可以輸入資料分割根路徑，以便將分割的資料夾當做資料行來讀取 | 否 | String | partitionRootPath |
| 檔案清單 | 您的來源是否指向列出要處理之檔案的文字檔 | 否 | `true` 或 `false` | fileList |
| 要儲存檔案名稱的資料行 | 建立具有來原始檔案名和路徑的新資料行 | 否 | String | rowUrlColumn |
| 完成後 | 在處理之後刪除或移動檔案。 檔案路徑從容器根目錄開始 | 否 | Delete： `true` 或`false` <br> 進入`['<from>', '<to>']` | purgeFiles <br> moveFiles |
| 依上次修改時間篩選 | 選擇根據上次修改檔案的時間來篩選檔案 | 否 | 時間戳記 | modifiedAfter <br> modifiedBefore |
| 單一檔 | 對應資料流程從每個檔案讀取一個 JSON 檔 | 否 | `true` 或 `false` | singleDocument |
| 不具引號的資料行名稱 | 如果選取了不具引號的資料**行名稱**，則對應資料流程會讀取未以引號括住的 JSON 資料行。 | 否 | `true` 或 `false` |  unquotedColumnNames |
| 有批註 | 如果 JSON 資料具有 C 或 c + + 樣式批註，請選取 [**具有批註**] | 否 | `true` 或 `false` | asComments |
| 單引號 | 讀取未以引號括住的 JSON 資料行 | 否 | `true` 或 `false` | singleQuoted |
| 反斜線已轉義 | 如果使用反斜線來轉義 JSON 資料中的字元，請選取 [**反斜線**（已轉義）] | 否 | `true` 或 `false` | backslashEscape |

### <a name="source-format-options"></a>來源格式選項

使用 JSON 資料集做為資料流程中的來源，可讓您設定五個額外的設定。 這些設定可以在 [**來源選項**] 索引標籤的 [ **JSON 設定**] [可折疊] 底下找到。  

![JSON 設定](media/data-flow/json-settings.png "JSON 設定")

#### <a name="default"></a>Default

根據預設，JSON 資料會以下列格式讀取。

```
{ "json": "record 1" }
{ "json": "record 2" }
{ "json": "record 3" }
```

#### <a name="single-document"></a>單一檔

如果選取了 [**單一**檔]，則對應的資料流程會從每個檔案讀取一個 JSON 檔。 

``` json
File1.json
{
    "json": "record 1"
}
File2.json
{
    "json": "record 2"
}
File3.json
{
    "json": "record 3"
}
```
> [!NOTE]
> 如果資料流程在預覽 JSON 資料時擲回錯誤指出「corrupt_record」，則您的資料可能包含 JSON 檔案中的單一檔。 設定「單一檔」應該會清除該錯誤。

#### <a name="unquoted-column-names"></a>不具引號的資料行名稱

如果選取了不具引號的資料**行名稱**，則對應資料流程會讀取未以引號括住的 JSON 資料行。 

```
{ json: "record 1" }
{ json: "record 2" }
{ json: "record 3" }
```

#### <a name="has-comments"></a>有批註

如果 JSON 資料具有 C 或 c + + 樣式批註，請選取 [**具有批註**]。

``` json
{ "json": /** comment **/ "record 1" }
{ "json": "record 2" }
{ /** comment **/ "json": "record 3" }
```

#### <a name="single-quoted"></a>單引號

如果 JSON 欄位和值使用單引號，而不是雙引號，請選取 [**單引號**]。

```
{ 'json': 'record 1' }
{ 'json': 'record 2' }
{ 'json': 'record 3' }
```

#### <a name="backslash-escaped"></a>反斜線已轉義

如果使用反斜線來轉義 JSON 資料中的字元，請選取 [**反斜線**]。

```
{ "json": "record 1" }
{ "json": "\} \" \' \\ \n \\n record 2" }
{ "json": "record 3" }
```

### <a name="sink-properties"></a>接收屬性

下表列出 json 接收所支援的屬性。 您可以在 [**設定**] 索引標籤中編輯這些屬性。

| 名稱 | 描述 | 必要 | 允許的值 | 資料流程腳本屬性 |
| ---- | ----------- | -------- | -------------- | ---------------- |
| 清除資料夾 | 如果在寫入之前清除目的資料夾 | 否 | `true` 或 `false` | truncate |
| 檔案名選項 | 寫入之資料的命名格式。 根據預設，每個資料分割的一個檔案格式`part-#####-tid-<guid>` | 否 | 模式：字串 <br> 每個資料分割： String [] <br> 當做資料行中的資料：字串 <br> 輸出至單一檔案：`['<fileName>']`  | filePattern <br> partitionFileNames <br> rowUrlColumn <br> partitionFileNames |

### <a name="creating-json-structures-in-a-derived-column"></a>在衍生的資料行中建立 JSON 結構

您可以透過衍生的資料行運算式產生器，將複雜的資料行加入至您的資料流程。 在 [衍生的資料行] 轉換中，加入新的資料行，然後按一下藍色方塊來開啟 [運算式產生器]。 若要讓資料行變得複雜，您可以手動輸入 JSON 結構，或使用 UX 以互動方式加入個子。

#### <a name="using-the-expression-builder-ux"></a>使用運算式產生器 UX

在輸出架構側邊窗格中，將滑鼠停留在資料行上，然後按一下加號圖示。 選取 [**加入 subcolumn** ]，將資料行設為複雜類型。

![新增子資料行](media/data-flow/addsubcolumn.png "新增子資料行")

您可以用相同的方式加入其他資料行和個子。 針對每個非複雜欄位，可以在右邊的運算式編輯器中加入運算式。

![複雜資料行](media/data-flow/complexcolumn.png "複雜資料行")

#### <a name="entering-the-json-structure-manually"></a>手動輸入 JSON 結構

若要手動加入 JSON 結構，請加入新的資料行，並在編輯器中輸入運算式。 運算式會遵循下列一般格式：

```
@(
    field1=0,
    field2=@(
        field1=0
    )
)
```

如果已針對名為 "complexColumn" 的資料行輸入此運算式，則會將它寫入至接收，做為下列 JSON：

```
{
    "complexColumn": {
        "field1": 0,
        "field2": {
            "field1": 0
        }
    }
}
```

#### <a name="sample-manual-script-for-complete-hierarchical-definition"></a>完整階層式定義的範例手動腳本
```
@(
    title=Title,
    firstName=FirstName,
    middleName=MiddleName,
    lastName=LastName,
    suffix=Suffix,
    contactDetails=@(
        email=EmailAddress,
        phone=Phone
    ),
    address=@(
        line1=AddressLine1,
        line2=AddressLine2,
        city=City,
        state=StateProvince,
        country=CountryRegion,
        postCode=PostalCode
    ),
    ids=[
        toString(CustomerID), toString(AddressID), rowguid
    ]
)
```

## <a name="next-steps"></a>後續步驟

- [複製活動概觀](copy-activity-overview.md)
- [對應資料流程](concepts-data-flow-overview.md)
- [查閱活動](control-flow-lookup-activity.md)
- [GetMetadata 活動](control-flow-get-metadata-activity.md)

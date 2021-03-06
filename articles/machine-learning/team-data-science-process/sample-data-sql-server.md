---
title: 對 Azure 上 SQL Server 中的資料進行取樣 - Team Data Science Process
description: 使用 SQL 或 Python 程式設計語言，對儲存在 Azure 上 SQL Server 中的資料進行取樣，然後將它移到 Azure Machine Learning。
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: e43c343b27dfe2dc0c364e58ed7305bdcec37215
ms.sourcegitcommit: 0100d26b1cac3e55016724c30d59408ee052a9ab
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/07/2020
ms.locfileid: "86026061"
---
# <a name="sample-data-in-sql-server-on-azure"></a><a name="heading"></a>Azure 上的 SQL Server 範例資料

本文說明如何使用 SQL 或 Python 程式設計語言，對儲存在 Azure 上之 SQL Server 中的資料進行取樣。 也示範如何透過將取樣的資料儲存到檔案，讓取樣資料移動到 Azure Machine Learning、將取樣的資料上傳至 Azure blob，然後將其讀入 Azure Machine Learning Studio。

Python 取樣使用 [pyodbc](https://code.google.com/p/pyodbc/) ODBC 程式庫來連接到 Azure 上的 SQL Server 以及 [Pandas](https://pandas.pydata.org/) 程式庫來進行取樣。

> [!NOTE]
> 本文件中的 SQL 程式碼範例假設資料位於 Azure 上的 SQL Server 中。 如果資料不在其中，請參閱[移動資料至 Azure 虛擬機器上的 SQL Server](move-sql-server-virtual-machine.md) 一文，以取得如何將資料移至 Azure 上 SQL Server 的指示。
> 
> 

**為何要對您的資料進行取樣？**
如果您規劃分析的資料集很龐大，通常最好是對資料進行向下取樣，將資料縮減為更小但具代表性且更容易管理的大小。 取樣有助於資料的瞭解、探索和特徵工程設計。 它在 [Team Data Science Process (TDSP)](https://docs.microsoft.com/azure/machine-learning/team-data-science-process/) 中扮演的角色是，能夠快速建立資料處理函式與機器學習服務模型的原型。

這個取樣工作是 [Team Data Science Process (TDSP)](https://docs.microsoft.com/azure/machine-learning/team-data-science-process/)中的一個步驟。

## <a name="using-sql"></a><a name="SQL"></a>使用 SQL
本節將說明使用 SQL，對資料庫中的資料執行簡單隨機取樣的數個方法。 根據資料大小及其分佈來選擇方法。

以下兩個項目示範如何在 SQL Server 中使用 `newid` 進行取樣。 您選擇的方法取決於您想要樣本的隨機程度（在下列範例程式碼中 pk_id 會假設為自動產生的主要金鑰）。

1. 較不嚴格的隨機取樣

    ```sql
    select  * from <table_name> where <primary_key> in 
    (select top 10 percent <primary_key> from <table_name> order by newid())
    ```

2. 更隨機範例 

    ```sql
    SELECT * FROM <table_name>
    WHERE 0.1 >= CAST(CHECKSUM(NEWID(), <primary_key>) & 0x7fffffff AS float)/ CAST (0x7fffffff AS int)
    ```

Tablesample 也可用來對資料進行取樣。 如果您的資料大小很大（假設不同頁面上的資料不會相互關聯），而且查詢在合理的時間內完成，則此選項可能是較好的方法。

```sql
SELECT *
FROM <table_name> 
TABLESAMPLE (10 PERCENT)
```

> [!NOTE]
> 您可以藉由將這個取樣的資料儲存於新的資料表中，從該資料中探索並產生功能
> 
> 

### <a name="connecting-to-azure-machine-learning"></a><a name="sql-aml"></a>連接到 Azure Machine Learning
您可以在 Azure Machine Learning [匯入資料][import-data]模組中直接使用上述取樣查詢，來進行即時資料縮小取樣，然後帶入 Azure Machine Learning 實驗中。 使用讀取器模組讀取取樣資料的螢幕擷取畫面如下所示：

![讀取器 SQL][1]

## <a name="using-the-python-programming-language"></a><a name="python"></a>使用 Python 程式設計語言
本節示範如何使用 [pyodbc 程式庫](https://code.google.com/p/pyodbc/) 來建立連線至 Python 中 SQL Server 資料庫的 ODBC。 資料庫連接字串如下所示：（以您的設定取代 servername、dbname、username 和 password）：

```python
#Set up the SQL Azure connection
import pyodbc    
conn = pyodbc.connect('DRIVER={SQL Server};SERVER=<servername>;DATABASE=<dbname>;UID=<username>;PWD=<password>')
```

Python 中的 [Pandas](https://pandas.pydata.org/) 程式庫提供一組豐富的資料結構和資料分析工具，可用來對 Python 程式設計進行資料操作。 下列程式碼會將 Azure SQL Database 中資料表的資料0.1% 樣本讀取至 Pandas 資料：

```python
import pandas as pd

# Query database and load the returned results in pandas data frame
data_frame = pd.read_sql('''select column1, column2... from <table_name> tablesample (0.1 percent)''', conn)
```

您現在可以在 Pandas 資料框架中使用取樣資料。 

### <a name="connecting-to-azure-machine-learning"></a><a name="python-aml"></a>連接到 Azure Machine Learning
您可以使用下列程式碼範例，將向下取樣的資料儲存至檔案，並將它上傳至 Azure Blob。 使用[匯入資料][import-data]模組即可將 Blob 中的資料直接讀取到「Azure Machine Learning 實驗」中。 步驟如下： 

1. 將 Pandas 資料框架寫入本機檔案

    ```python
    dataframe.to_csv(os.path.join(os.getcwd(),LOCALFILENAME), sep='\t', encoding='utf-8', index=False)
    ```

2. 將本機檔案上傳至 Azure Blob

    ```python
    from azure.storage import BlobService
    import tables

    STORAGEACCOUNTNAME= <storage_account_name>
    LOCALFILENAME= <local_file_name>
    STORAGEACCOUNTKEY= <storage_account_key>
    CONTAINERNAME= <container_name>
    BLOBNAME= <blob_name>

    output_blob_service=BlobService(account_name=STORAGEACCOUNTNAME,account_key=STORAGEACCOUNTKEY)    
    localfileprocessed = os.path.join(os.getcwd(),LOCALFILENAME) #assuming file is in current working directory

    try:

    #perform upload
    output_blob_service.put_block_blob_from_path(CONTAINERNAME,BLOBNAME,localfileprocessed)

    except:            
        print ("Something went wrong with uploading blob:"+BLOBNAME)
    ```

3. 使用 Azure Machine Learning [匯入資料][import-data]模組從 Azure Blob 讀取資料，如以下螢幕擷取畫面所示：

![讀取器 Blob][2]

## <a name="the-team-data-science-process-in-action-example"></a>Team Data Science Process 實務範例
如需使用公用資料集進行小組資料科學程式的範例，請參閱[Team 資料科學程式實務：使用 SQL Server](sql-walkthrough.md)。

[1]: ./media/sample-sql-server-virtual-machine/reader_database.png
[2]: ./media/sample-sql-server-virtual-machine/reader_blob.png

[import-data]: https://msdn.microsoft.com/library/azure/4e1b0fe6-aded-4b3f-a36f-39b8862b9004/

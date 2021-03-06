<properties
	pageTitle="コンピューティングのリンクされたサービス | Microsoft Azure"
	description="Azure Data Factory パイプラインでデータの変換/処理に使用できるコンピューティング環境について学習します。"
	services="data-factory"
	documentationCenter=""
	authors="spelluru"
	manager="jhubbard"
	editor="monicar"/>

<tags
	ms.service="data-factory"
	ms.workload="data-services"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="01/19/2016"
	ms.author="spelluru"/>

# コンピューティングのリンクされたサービス

この記事では、データの処理または変換に利用できるさまざまなコンピューティング環境について説明します。これらのコンピューティング環境を Azure Data Factory にリンクする「リンクされたサービス」の構成時に Data Factory でサポートされるさまざまな構成 (オンデマンドと Bring Your Own の比較) に関する詳細も提供します。

## オンデマンドのコンピューティング環境

この種類の構成では、コンピューティング環境は Azure Data Factory サービスにより完全管理されます。データを処理するためのジョブが送信される前に Data Factory サービスにより自動的に作成され、ジョブの完了時に削除されます。オンデマンドのコンピューティング環境のために「リンクされたサービス」を作成し、構成し、ジョブ実行、クラスター管理、ブートストラップ アクションの詳細設定を制御できます。

> [AZURE.NOTE] オンデマンド構成は現在のところ、Azure HDInsight クラスターにのみ対応しています。

## Azure HDInsight のオンデマンドのリンクされたサービス
Azure Data Factory サービスは、データを処理するための Windows/Linux ベースのオンデマンド HDInsight クラスターを自動的に作成します。このクラスターはクラスターに関連付けられているストレージ アカウント (JSON の linkedServiceName プロパティ) と同じリージョンで作成されます。

オンデマンド HDInsight のリンクされたサービスに関する次の**重要な**点に注意してください。

- 作成したオンデマンド HDInsight クラスターは Azure サブスクリプションに表示されません。Azure Data Factory サービスがあなたの代わりにオンデマンド HDInsight クラスターを管理します。
- オンデマンド HDInsight クラスターで実行されるジョブのログは HDInsight クラスターに関連付けられているストレージ アカウントにコピーされます。これらのログには、Azure クラシック ポータルの **[アクティビティの実行の詳細]** ブレードからアクセスできます。詳細については、「[パイプラインの監視と管理](data-factory-monitor-manage-pipelines.md)」という記事を参照してください。
- HDInsight クラスターが稼動し、ジョブを実行している時間に対してのみ課金されます。

> [AZURE.IMPORTANT] オンデマンドで Azure HDInsight クラスターをプロビジョニングするには一般的に **15 分**以上かかります。

### 例
次の JSON は、オンデマンド HDInsight のリンクされたサービスを定義します。Data Factory は、データ スライスを処理するときに、**Windows ベースの** HDInsight クラスターを自動的に作成します。このサンプル JSON では **osType** が指定されていないことに注意してください。このプロパティの既定値は **Windows** です。

	{
	  "name": "HDInsightOnDemandLinkedService",
	  "properties": {
	    "type": "HDInsightOnDemand",
	    "typeProperties": {
	      "version": "3.2",
	      "clusterSize": 1,
	      "timeToLive": "00:30:00",
	      "linkedServiceName": "StorageLinkedService"
	    }
	  }
	}


次の JSON は、Linux ベースのオンデマンド HDInsight のリンクされたサービスを定義します。Data Factory サービスは、データ スライスを処理するときに、**Linux ベースの** HDInsight クラスターを自動的に作成します。**sshUserName** と **sshPassword** の値を指定する必要があります。


	{
	    "name": "HDInsightOnDemandLinkedService",
	    "properties": {
	        "hubName": "getstarteddf0121_hub",
	        "type": "HDInsightOnDemand",
	        "typeProperties": {
	            "version": "3.2",
	            "clusterSize": 4,
	            "timeToLive": "00:05:00",
	            "osType": "linux",
	            "sshPassword": "MyPassword!",
	            "sshUserName": "myuser",
	            "linkedServiceName": "StorageLinkedService",
	        }
	    }
	}

> [AZURE.IMPORTANT] 
HDInsight クラスターは、JSON (**linkedServiceName**) で指定した BLOB ストレージに**既定のコンテナー**を作成します。クラスターを削除しても、HDInsight はこのコンテナーを削除しません。これは設計によるものです。オンデマンド HDInsight のリンクされたサービスでは、既存のライブ クラスター (**timeToLive**) がある場合を除き、スライスを処理する必要があるたびに HDInsight クラスターが作成され、処理が終了すると削除されます。
> 
> 処理されるスライスが多いほど、Azure Blob Storage 内のコンテナーも増えます。ジョブのトラブルシューティングのためにコンテナーが必要ない場合、コンテナーを削除してストレージ コストを削減できます。コンテナーの名前は、"adf**Data Factory 名**-**リンクされたサービス名**-日時スタンプ" というパターンになります。Azure BLOB ストレージ内のコンテナーを削除するには、[Microsoft ストレージ エクスプローラー](http://storageexplorer.com/)などのツールを使用します。

### プロパティ

プロパティ | 説明 | 必須
-------- | ----------- | --------
type | type プロパティは **HDInsightOnDemand** に設定されます。 | はい
clusterSize | オンデマンド クラスターのサイズです。このオンデマンド クラスターに配置するノードの数を指定します。 | はい
timetolive | <p>オンデマンド HDInsight クラスターに許可されるアイドル時間です。他のアクティブなジョブがクラスターにない場合、アクティビティ実行の完了後にオンデマンド HDInsight クラスターが起動状態を維持する時間を指定します。</p><p>たとえば、アクティビティ実行に 6 分かかるときに timetolive が 5 分に設定されている場合、アクティビティ実行の 6 分間の処理の後、クラスターが起動状態を 5 分間維持します。別のアクティビティ実行が 6 分の時間枠で実行される場合、それは同じクラスターで処理されます。</p><p>オンデマンド HDInsight クラスターの作成は高額な作業であり (時間もかかることがあります)、オンデマンド HDInsight クラスターを再利用し、Data Factory のパフォーマンスを改善する必要がある場合にこの設定を利用します。</p><p>timetolive 値を 0 に設定した場合、アクティビティ実行の処理直後にクラスターが削除されます。その一方で、高い値を設定した場合、クラスターは不必要にアイドル状態を維持し、コストの上昇を招きます。そのため、ニーズに合わせて適切な値を設定することが重要です。</p><p>timetolive プロパティ値が適切に設定されている場合、複数のパイプラインでオンデマンド HDInsight クラスターの同じインスタンスを共有できます。</p> | はい
version | HDInsight クラスターのバージョン。既定値は、Windows クラスターでは 3.1、Linux クラスターでは 3.2 です。 | いいえ
linkedServiceName | データの保存し、処理するためのオンデマンド クラスターで使用される BLOB ストアです。 | はい
additionalLinkedServiceNames | Data Factory サービスがあなたの代わりに登録できるように、HDInsight の「リンクされたサービス」の追加ストレージ アカウントを指定します。 | いいえ
osType | オペレーティング システムの種類。使用可能な値: Windows (既定値) および Linux | いいえ
hcatalogLinkedServiceName | HCatalog データベースを指す Azure SQL のリンクされたサービスの名前。オンデマンド HDInsight クラスターは、Azure SQL データベースをメタストアとして使用して作成されます。 | いいえ
sshUser | Linux ベースの HDInsight クラスターの SSH ユーザー。 | はい (Linux のみ)
sshPassword | Linux ベースの HDInsight クラスターの SSH パスワード。 | はい (Linux のみ)


#### additionalLinkedServiceNames JSON の例

    "additionalLinkedServiceNames": [
        "otherLinkedServiceName1",
		"otherLinkedServiceName2"
  	]
 
### 高度なプロパティ

次のプロパティを指定し、オンデマンド HDInsight クラスターを詳細に設定することもできます。

プロパティ | 説明 | 必須
:-------- | :----------- | :--------
coreConfiguration | 作成する HDInsight クラスターに core 構成パラメーター (core-site.xml と同じ) を指定します。 | いいえ
hBaseConfiguration | HDInsight クラスターに HBase 構成パラメーター (hbase-site.xml) を指定します。 | いいえ
hdfsConfiguration | HDInsight クラスターに HDFS 構成パラメーター (hdfs-site.xml) を指定します。 | いいえ
hiveConfiguration | HDInsight クラスターに hive 構成パラメーター (hive-site.xml) を指定します。 | いいえ
mapReduceConfiguration | HDInsight クラスターに MapReduce 構成パラメーター (mapred-site.xml) を指定します。 | いいえ
oozieConfiguration | HDInsight クラスターに Oozie 構成パラメーター (oozie-site.xml) を指定します。 | いいえ
stormConfiguration | HDInsight クラスターに Storm 構成パラメーター (storm-site.xml) を指定します。 | いいえ
yarnConfiguration | HDInsight クラスターに Yarn 構成パラメーター (yarn-site.xml) を指定します。 | いいえ

#### 例 – オンデマンド HDInsight クラスターと詳細なプロパティ

	{
	  "name": " HDInsightOnDemandLinkedService",
	  "properties": {
	    "type": "HDInsightOnDemand",
	    "typeProperties": {
	      "clusterSize": 16,
	      "timeToLive": "01:30:00",
	      "version": "3.2",
	      "linkedServiceName": "adfods1",
	      "coreConfiguration": {
	        "templeton.mapper.memory.mb": "5000"
	      },
	      "hiveConfiguration": {
	        "templeton.mapper.memory.mb": "5000"
	      },
	      "mapReduceConfiguration": {
	        "mapreduce.reduce.java.opts": "-Xmx4000m",
	        "mapreduce.map.java.opts": "-Xmx4000m",
	        "mapreduce.map.memory.mb": "5000",
	        "mapreduce.reduce.memory.mb": "5000",
	        "mapreduce.job.reduce.slowstart.completedmaps": "0.8"
	      },
	      "yarnConfiguration": {
	        "yarn.app.mapreduce.am.resource.mb": "5000",
	        "mapreduce.map.memory.mb": "5000"
	      },
	      "additionalLinkedServiceNames": [
	        "datafeeds",
	        "adobedatafeed"
	      ]
	    }
	  }
	}

### ノードのサイズ
次のプロパティを使用して、ヘッド ノード、データ ノード、Zookeeper ノードのサイズを指定できます。

プロパティ | 説明 | 必須
:-------- | :----------- | :--------
headNodeSize | ヘッド ノードのサイズを指定します。既定値は "Large" です。詳細については、下記の「**ノードのサイズの指定**」をご覧ください。 | いいえ
dataNodeSize | データ ノードのサイズを指定します。既定値は "Large" です。 | いいえ
zookeeperNodeSize | Zookeeper ノードのサイズを指定します。既定値は "Small" です。 | いいえ
 
#### ノードのサイズの指定
上記のプロパティで指定する必要がある文字列値については、「[仮想マシンのサイズ](../virtual-machines/virtual-machines-size-specs.md#size-tables)」をご覧ください。値は、この記事に記載されている**コマンドレットと API** に準拠する必要があります。この記事に示すように、Large (既定値) サイズのデータ ノードのメモリ容量は 7 GB ですが、シナリオによってはこれでは不十分な場合があります。

D4 サイズのヘッド ノードと worker ノードを作成する場合は、headNodeSize プロパティと dataNodeSize プロパティの値として **Standard\_D4** を指定する必要があります。

	"headNodeSize": "Standard_D4",	
	"dataNodeSize": "Standard_D4",

これらのプロパティに間違った値を指定すると、次のエラーが発生します。**エラー:** クラスターを作成できませんでした。例外: クラスター作成操作を完了できません。コード '400' で操作が失敗しました。取り残されたクラスターの状態: 'Error'。メッセージ: 'PreClusterCreationValidationFailure'。" というエラー メッセージが表示される場合があります。このエラーが発生した場合は、前述の記事の表に記載されている**コマンドレットと API** の名前を使用していることを確認してください。



## Bring Your Own のコンピューティング環境

この種類の構成では、既存のコンピューティング環境を Data Factory の「リンクされたサービス」として登録できます。このコンピューティング環境はユーザーにより管理され、Data Factory サービスをこの環境を利用し、アクティビティを実行します。

この種類の構成は次のコンピューティング環境でサポートされています。

- Azure HDInsight
- Azure Batch
- Azure Machine Learning

## Azure HDInsight のリンクされたサービス

Azure HDInsight の「リンクされたサービス」を作成し、独自の HDInsight クラスターを Data Factory に登録できます。

### 例

	{
	  "name": "HDInsightLinkedService",
	  "properties": {
	    "type": "HDInsight",
	    "typeProperties": {
	      "clusterUri": " https://<hdinsightclustername>.azurehdinsight.net/",
	      "userName": "admin",
	      "password": "<password>",
	      "location": "WestUS",
	      "linkedServiceName": "MyHDInsightStoragelinkedService"
	    }
	  }
	}

### プロパティ

プロパティ | 説明 | 必須
-------- | ----------- | --------
type | type プロパティは **HDInsight** に設定されます。 | はい
clusterUri | HDInsight クラスターの URI です。 | はい
username | 既存の HDInsight クラスターに接続するために使用されるユーザーの名前を指定します。 | はい
パスワード | ユーザー アカウントのパスワードを指定します。 | あり
location | HDInsight クラスターの場所を指定します (例: WestUS)。 | はい
linkedServiceName | HDInsight クラスターで使用される BLOB ストレージの「リンクされたサービス」の名前です。 | はい

## Azure Batch の「リンクされたサービス」

Azure Batch の「リンクされたサービス」を作成し、仮想マシン (VM) の Batch プールを Data Factory に登録できます。Azure Batch と Azure HDInsight のいずれかを利用し、.NET カスタム アクティビティを実行できます。

Azure Batch サービスの利用が初めての場合、次のトピックを参照してください。


- Azure Batch サービスの概要については、「[Azure Batch の基本](../batch/batch-technical-overview.md)」をご覧ください。
- Azure Batch アカウントについては、[New-AzureBatchAccount](https://msdn.microsoft.com/library/mt125880.aspx) コマンドレットを使用して作成する方法または [Azure クラシック ポータル](../batch/batch-account-create-portal.md)を使用して作成する方法をご覧ください。コマンドレットの使用方法の詳細については、「[PowerShell を使用した Azure Batch アカウントの管理](http://blogs.technet.com/b/windowshpc/archive/2014/10/28/using-azure-powershell-to-manage-azure-batch-account.aspx)」トピックをご覧ください。
- Azure Batch プールを作成するための [New-AzureBatchPool](https://msdn.microsoft.com/library/mt125936.aspx) コマンドレット。

### 例

	{
	  "name": "AzureBatchLinkedService",
	  "properties": {
	    "type": "AzureBatch",
	    "typeProperties": {
	      "accountName": "<Azure Batch account name>",
	      "accessKey": "<Azure Batch account key>",
	      "poolName": "<Azure Batch pool name>",
	      "linkedServiceName": "<Specify associated storage linked service reference here>"
	    }
	  }
	}

**accountName** プロパティのバッチ アカウントの名前に「**.<リージョン名**」を追加します。例:

			"accountName": "mybatchaccount.eastus"

もう 1 つの選択肢は下のように batchUri エンドポイントを指定することです。

			"accountName": "adfteam",
			"batchUri": "https://eastus.batch.azure.com",

### プロパティ

プロパティ | 説明 | 必須
-------- | ----------- | --------
type | type プロパティは **AzureBatch** に設定されます。 | はい
accountName | Azure Batch アカウントの名前です。 | あり
accessKey | Azure Batch アカウントのアクセス キーです。 | はい
poolName | 仮想マシンのプールの名前です。 | はい
linkedServiceName | この Azure Batch の「リンクされたサービス」に関連付けられている Azure Storage の「リンクされたサービス」の名前です。この「リンクされたサービス」はアクティビティの実行に必要なファイルのステージングとアクティビティ実行ログの保存に利用されます。 | はい


## Azure Machine Learning のリンクされたサービス

Azure Machine Learning の「リンクされたサービス」を作成し、Machine Learning のバッチ スコアリング エンドポイントを Data Factory に登録します。

### 例

	{
	  "name": "AzureMLLinkedService",
	  "properties": {
	    "type": "AzureML",
	    "typeProperties": {
	      "mlEndpoint": "https://[batch scoring endpoint]/jobs",
	      "apiKey": "<apikey>"
	    }
	  }
	}

### プロパティ

プロパティ | 説明 | 必須
-------- | ----------- | --------
Type | type プロパティは **AzureML** に設定されます。 | はい
mlEndpoint | バッチ スコアリング URL です。 | はい
apiKey | 公開されたワークスペース モデルの API です。 | はい


## Azure Data Lake Analytics リンク サービス
**Azure Data Lake Analytics** リンク サービスを作成して、Azure Data Lake Analytics コンピューティング サービスを Azure Data Factory にリンクしてから、パイプラインで [Data Lake Analytics U-SQL アクティビティ](data-factory-usql-activity.md)を使用します。

次の例は、Azure Data Lake Analytics リンク サービスの JSON 定義です。

	{
	    "name": "AzureDataLakeAnalyticsLinkedService",
	    "properties": {
	        "type": "AzureDataLakeAnalytics",
	        "typeProperties": {
	            "accountName": "adftestaccount",
	            "dataLakeAnalyticsUri": "datalakeanalyticscompute.net",
	            "authorization": "<authcode>",
				"sessionId": "<session ID>",
	            "subscriptionId": "<subscription id>",
	            "resourceGroupName": "<resource group name>"
	        }
	    }
	}


次の表は、JSON 定義で使用されるプロパティの説明です。

プロパティ | 説明 | 必須
-------- | ----------- | --------
型 | type プロパティは **AzureDataLakeAnalytics** に設定する必要があります。 | あり
accountName | Azure Data Lake Analytics アカウント名。 | あり
dataLakeAnalyticsUri | Azure Data Lake Analytics URI。 | いいえ
authorization | Data Factory Editor で **[承認]** ボタンをクリックし、OAuth ログインを完了すると、承認コードが自動的に取得されます。 | あり
subscriptionId | Azure サブスクリプション ID | いいえ (指定されていない場合は Data Factory のサブスクリプションが使用されます)。
resourceGroupName | Azure リソース グループ名 | いいえ (指定されていない場合は Data Factory のリソース グループが使用されます)。
sessionId | OAuth 承認セッションのセッション ID です。各セッション ID は一意であり、1 回のみ使用できます。セッション ID は、Data Factory Editor で自動生成されます。 | はい

**[認証]** ボタンを使用して生成した認証コードは、いずれ有効期限が切れます。さまざまな種類のユーザー アカウントの有効期限については、次の表を参照してください。認証**トークンの有効期限が切れる**と、次のエラー メッセージが表示される場合があります: "資格情報の操作エラー: invalid\_grant - AADSTS70002: 資格情報の検証中にエラーが発生しました。AADSTS70008: 指定されたアクセス権の付与は期限が切れているか、失効しています。トレース ID: d18629e8-af88-43c5-88e3-d8419eb1fca1 相関 ID: fac30a0c-6be6-4e02-8d69-a776d2ffefd7 タイムスタンプ: 2015-12-15 21:09:31Z"

| ユーザー タイプ | 有効期限 |
| :-------- | :----------- | 
| Azure Active Directory (@hotmail.com、@live.com など) によって管理されていないユーザー | 12 時間 |
| Azure Active Directory (AAD) によって管理されているユーザー | | OAuth ベースのリンクされたサービスに基づいて、最後のスライス実行からスライスが 14 日間実行しない場合、最後のスライス実行から 14 日です。<p>OAuth ベースのリンクされたサービスに基づいて、少なくとも 14 日間に 1 回スライスが実行する場合、90 日です。</p> |

 
このエラーを回避または解決するには、**トークンの有効期限が切れた**ときに、**[認証]** ボタンを使用して再認証し、リンクされたサービスを再デプロイする必要があります。次のセクションのコードを使用して、sessionId と authorization プロパティの値をプログラムで生成することもできます。

### プログラムを使用して sessionId と authorization の値を生成するには 
次のコードは、**sessionId** と **authorization** の値を生成します。

    if (linkedService.Properties.TypeProperties is AzureDataLakeStoreLinkedService ||
        linkedService.Properties.TypeProperties is AzureDataLakeAnalyticsLinkedService)
    {
        AuthorizationSessionGetResponse authorizationSession = this.Client.OAuth.Get(this.ResourceGroupName, this.DataFactoryName, linkedService.Properties.Type);

        WindowsFormsWebAuthenticationDialog authenticationDialog = new WindowsFormsWebAuthenticationDialog(null);
        string authorization = authenticationDialog.AuthenticateAAD(authorizationSession.AuthorizationSession.Endpoint, new Uri("urn:ietf:wg:oauth:2.0:oob"));

        AzureDataLakeStoreLinkedService azureDataLakeStoreProperties = linkedService.Properties.TypeProperties as AzureDataLakeStoreLinkedService;
        if (azureDataLakeStoreProperties != null)
        {
            azureDataLakeStoreProperties.SessionId = authorizationSession.AuthorizationSession.SessionId;
            azureDataLakeStoreProperties.Authorization = authorization;
        }

        AzureDataLakeAnalyticsLinkedService azureDataLakeAnalyticsProperties = linkedService.Properties.TypeProperties as AzureDataLakeAnalyticsLinkedService;
        if (azureDataLakeAnalyticsProperties != null)
        {
            azureDataLakeAnalyticsProperties.SessionId = authorizationSession.AuthorizationSession.SessionId;
            azureDataLakeAnalyticsProperties.Authorization = authorization;
        }
    }

コードで使用する Data Factory クラスに関する詳細は、「[AzureDataLakeStoreLinkedService クラス](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.azuredatalakestorelinkedservice.aspx)」、「[AzureDataLakeAnalyticsLinkedService クラス](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.azuredatalakeanalyticslinkedservice.aspx)」、「[AuthorizationSessionGetResponse クラス](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.authorizationsessiongetresponse.aspx)」のトピックを参照してください。WindowsFormsWebAuthenticationDialog クラスの Microsoft.IdentityModel.Clients.ActiveDirectory.WindowsForms.dll に参照を追加する必要があります。
 

## Azure SQL のリンクされたサービス

Azure SQL のリンクされたサービスを作成し、[ストアド プロシージャ アクティビティ](data-factory-stored-proc-activity.md)で使用して、Data Factory パイプラインからストアド プロシージャを起動します。このリンクされたサービスの詳細については、[Azure SQL コネクタ](data-factory-azure-sql-connector.md#azure-sql-linked-service-properties)に関する記事を参照してください。

<!---HONumber=AcomDC_0224_2016-->
<properties
	pageTitle="HDInsight での Hadoop Sqoop の使用 | Microsoft Azure"
	description="コンピューターから Azure PowerShell を使用して、Hadoop クラスターと Azure SQL データベース間で Sqoop インポートとエクスポートを実行する方法について説明します。"
	editor="cgronlun"
	manager="paulettm"
	services="hdinsight"
	documentationCenter=""
	tags="azure-portal"
	authors="mumian"/>

<tags
	ms.service="hdinsight"
	ms.workload="big-data"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="12/01/2015"
	ms.author="jgao"/>

#HDInsight の Hadoop での Sqoop の使用 (Windows)

[AZURE.INCLUDE [sqoop-selector](../../includes/hdinsight-selector-use-sqoop.md)]

HDInsight で Sqoop を使用して、HDInsight クラスターと Azure SQL Database または SQL Server データベース間でインポートとエクスポートを実行する方法について説明します。

> [AZURE.NOTE] この記事の手順は、Windows ベースまたは Linux ベースの HDInsight クラスターで使用できます。ただし、これらの手順は Windows クライアントでのみ機能します。
>
> Linux、OS X、または Unix クライアントと Linux ベースの HDInsight サーバーを使用している場合は、「[HDInsight の Hadoop での Sqoop の使用 (SSH)](hdinsight-use-sqoop-mac-linux.md)」を参照してください。

Hadoop はログやファイルなどの非構造化データおよび半構造化データを処理する場合に自然な選択ですが、リレーショナル データベースに格納された構造化データを処理する必要が生じることもあります。

[Sqoop][sqoop-user-guide-1.4.4] は、Hadoop クラスターとリレーショナル データベース間でデータを転送するためのツールです。このツールを使用して、SQL Server、MySQL、Oracle などのリレーショナル データベース管理システム (RDBMS) から Hadoop 分散ファイル システム (HDFS) へデータをインポートしたり、MapReduce または Hive を使用して Hadoop のデータを変換してから、そのデータを RDBMS にエクスポートして戻したりできます。このチュートリアルでは、リレーショナル データベースとして SQL Server データベースを使用します。

HDInsight クラスターでサポートされている Sqoop のバージョンについては、「[HDInsight で提供されるクラスター バージョンの新機能][hdinsight-versions]」を参照してください。

###前提条件

このチュートリアルを読み始める前に、次の項目を用意する必要があります。

- **Azure PowerShell を実行できるワークステーション**。「[Install Azure PowerShell 1.0 and greater (Azure PowerShell 1.0 以上のインストール)](hdinsight-administer-use-powershell.md#install-azure-powershell-10-and-greater)」をご覧ください。

既存の Azure SQL Database または Microsoft SQL Server を使用する場合

- **Azure SQL データベース**: ワークステーションから Azure SQL データベース サーバーに対するアクセスを許可するようにファイアウォール ルールを構成する必要があります。Azure SQL データベースを作成して、ファイアウォールを構成する手順については、「[Azure SQL データベースの概要][sqldatabase-get-started]」を参照してください。 

	> [AZURE.NOTE] 既定では、Azure SQL データベースは Azure HDinsight などの Azure サービスからの接続を許可します。このファイアウォール設定が無効になっている場合は、Azure プレビュー ポータルから有効にする必要があります。Azure SQL Database の作成方法とファイアウォール ルールの構成方法については、「[SQL Database の作成と構成][sqldatabase-create-configue]」を参照してください。

- **SQL Server**: HDInsight クラスターが SQL Server と同じ Azure の仮想ネットワーク上にある場合は、この記事の手順を使用して、SQL Server データベースとの間でデータをインポートおよびエクスポートできます。

	> [AZURE.NOTE] HDInsight は場所ベースの仮想ネットワークのみをサポートし、アフィニティ グループ ベースの仮想ネットワークは現在扱っていません。

	* 仮想ネットワークを作成および構成する場合は、「[Virtual Network の構成タスク](../services/virtual-machines/)」を参照してください。

		* SQL Server をデータセンター内で使用している場合は、仮想ネットワークを*サイト間*または*ポイント対サイト*として構成する必要があります。

			> [AZURE.NOTE] **ポイント対サイト**仮想ネットワークの場合、SQL Server が VPN クライアント構成アプリケーションを実行している必要があります。このアプリケーションは、Azure 仮想ネットワーク構成の**ダッシュボード**から入手できます。

		* SQL Server を Azure 仮想マシンで使用する際に、SQL Server をホストする仮想マシンが HDInsight と同じ仮想ネットワークに属している場合は、任意の仮想ネットワーク構成を使用できます。

	* 仮想ネットワークに HDInsight クラスターをプロビジョニングする場合は、「[カスタム オプションを使用した HDInsight での Hadoop クラスターのプロビジョニング](hdinsight-provision-clusters.md)」を参照してください。

	> [AZURE.NOTE] SQL Server では認証を許可する必要もあります。この記事の手順を実行するには、SQL Server ログインを使用する必要があります。
	
##シナリオの理解

HDInsight クラスターにはサンプル データがいくつか付属しています。そのうちの以下の 2 つを使用します。

- log4j ログ ファイル (*/example/data/sample.log*)。次のようなログがファイルから抽出されます。

		2012-02-03 18:35:34 SampleClass6 [INFO] everything normal for id 577725851
		2012-02-03 18:35:34 SampleClass4 [FATAL] system problem at id 1991281254
		2012-02-03 18:35:34 SampleClass3 [DEBUG] detail for id 1304807656
		...

- *hivesampletable* という名前の Hive テーブル。*/hive/warehouse/hivesampletable* にあるデータ ファイルを参照しています。テーブルはモバイル デバイスのデータを含んでいます。

まず、*sample.log* と *hivesampletable* を Azure SQL データベースまたは SQL Server にエクスポートしてから、次のパスを使用して、モバイル デバイスのデータを含むテーブルを HDInsight にインポートして戻します。

	/tutorials/usesqoop/importeddata

## PowerShell を使用した Sqoop の実行

このセクションの PowerShell サンプルでは、次の手順を実行します。

1. Azure に接続します。
2. Azure リソース グループを作成します。詳細については、[Azure リソース マネージャーでの Windows PowerShell の使用](../powershell-azure-resource-manager.md)に関するページを参照してください。
3. Azure SQL Database サーバー、Azure SQL Database、および 2 つのテーブルを作成します。 

	SQL Server を使用する場合は、次のステートメントを使用してテーブルを作成します。
	
		CREATE TABLE [dbo].[log4jlogs](
		 [t1] [nvarchar](50),
		 [t2] [nvarchar](50),
		 [t3] [nvarchar](50),
		 [t4] [nvarchar](50),
		 [t5] [nvarchar](50),
		 [t6] [nvarchar](50),
		 [t7] [nvarchar](50))

		CREATE TABLE [dbo].[mobiledata](
		 [clientid] [nvarchar](50),
		 [querytime] [nvarchar](50),
		 [market] [nvarchar](50),
		 [deviceplatform] [nvarchar](50),
		 [devicemake] [nvarchar](50),
		 [devicemodel] [nvarchar](50),
		 [state] [nvarchar](50),
		 [country] [nvarchar](50),
		 [querydwelltime] [float],
		 [sessionid] [bigint],
		 [sessionpagevieworder][bigint])

	データベースとテーブルを確認する最も簡単な方法は、Visual Studio を使用することです。データベース サーバーとデータベースは、Azure ポータルを使用して確認できます。

4. HDInsight クラスターを作成します。

	クラスターを確認するには、Azure ポータルまたは Azure PowerShell を使用します。

5. ソース データ ファイルを前処理します。

	このチュートリアルでは、log4j ログ ファイル (区切られたファイル) と Hive テーブルを Azure SQL データベースにエクスポートします。区切られたファイルは */example/data/sample.log* と呼ばれます。チュートリアルの前半で、log4j ログのサンプルをいくつか示しました。ログ ファイルには空白行がいくつかあり、その他の行は次のような内容です。
	
		java.lang.Exception: 2012-02-03 20:11:35 SampleClass2 [FATAL] unrecoverable system problem at id 609774657
			at com.osa.mocklogger.MockLogger$2.run(MockLogger.java:83)
	
	これは、このデータを使用する他の例に適していますが、Azure SQL データベースまたは SQL Server にインポートする前にこれらの例外を削除する必要があります。空の文字列がある場合、または Azure SQL データベース テーブルで定義されているフィールドの数よりも要素の数が少ない行がある場合、Sqoop エクスポートは失敗します。log4jlogs テーブルには 7 個の文字列型のフィールドがあります。

	この手順で、クラスター上に新しいファイル (tutorials/usesqoop/data/sample.log) が作成されます。変更されたデータ ファイルを確認する場合は、Azure ポータル、Azure Storage エクスプローラー ツール、または Azure PowerShell を使用できます。「[Azure HDInsight の概要][hdinsight-get-started]」に、Azure PowerShell を使用してファイルをダウンロードし、ファイルの内容を表示するコード例が示されています。

6. Azure SQL Database にデータ ファイルをエクスポートします。

	ソース ファイルは、tutorials/usesqoop/data/sample.log です。データのエクスポート先のテーブルは log4jlogs です。
	
	> [AZURE.NOTE] 接続文字列情報を除き、このセクションの手順は、Azure SQL データベースまたは SQL Server で動作します。これらの手順は次の構成を使用してテスト済みです。
	>
	> * **Azure Virtual Network ポイント対サイト構成**: HDInsight クラスターをプライベート データセンター内の SQL Server に接続する仮想ネットワーク。詳細については、「[管理ポータルでのポイント対サイト VPN の構成](../vpn-gateway/vpn-gateway-point-to-site-create.md)」を参照してください。
	> * **Azure HDInsight 3.1**: 仮想ネットワークでのクラスターの作成については、「[カスタム オプションを使用した HDInsight での Hadoop クラスターのプロビジョニング](hdinsight-provision-clusters.md)」を参照してください。
	> * **SQL Server 2014**: 仮想ネットワークに安全に接続するために、認証と、VPN クライアント構成パッケージの実行を許可する構成。

7. Azure SQL Database に Hive テーブルをエクスポートします。

8. HDInsight クラスターに mobiledata テーブルをインポートします。

	変更されたデータ ファイルを確認する場合は、プレビュー ポータル、Azure Storage エクスプローラー ツール、または Azure PowerShell を使用できます。「[HDInsight の概要][hdinsight-get-started]」に、Azure PowerShell を使用してファイルをダウンロードし、ファイルの内容を表示するコード例が示されています。


### PowerShell サンプル

	# Prepare an Azure SQL database to be used by the Sqoop tutorial
	
	#region - provide the following values
	
	$subscriptionID = "<Enter your Azure Subscription ID>"
	
	$sqlDatabaseLogin = "<Enter a SQL Database Login name>" #SQL Database server login
	$sqlDatabasePassword = "<Enter a Password>"
	
	$httpUserName = "admin"  #HDInsight cluster username
	$httpPassword = "<Enter a Password>"
	
	# used for creating Azure service names
	$nameToken = "<Enter an alias>" 
	$namePrefix = $nameToken.ToLower() + (Get-Date -Format "MMdd")
	#endregion
	
	#region - variables
	
	# Resource group variables
	$resourceGroupName = $namePrefix + "rg"
	$location = "East US 2" # used by all Azure services defined in this tutorial
	
	# SQL database varialbes
	$sqlDatabaseServerName = $namePrefix + "sqldbserver"
	$sqlDatabaseName = $namePrefix + "sqldb"
	$sqlDatabaseConnectionString = "Data Source=$sqlDatabaseServerName.database.windows.net;Initial Catalog=$sqlDatabaseName;User ID=$sqlDatabaseLogin;Password=$sqlDatabasePassword;Encrypt=true;Trusted_Connection=false;"
	$sqlDatabaseMaxSizeGB = 10
	
	# Used for retrieving external IP address and creating firewall rules
	$ipAddressRestService = "http://bot.whatismyipaddress.com"
	$fireWallRuleName = "UseSqoop"
	
	# Used for creating tables and clustered indexes
	$cmdCreateLog4jTable = "CREATE TABLE [dbo].[log4jlogs](
		[t1] [nvarchar](50),
		[t2] [nvarchar](50),
		[t3] [nvarchar](50),
		[t4] [nvarchar](50),
		[t5] [nvarchar](50),
		[t6] [nvarchar](50),
		[t7] [nvarchar](50))"
	
	$cmdCreateLog4jClusteredIndex = "CREATE CLUSTERED INDEX log4jlogs_clustered_index on log4jlogs(t1)"
	
	$cmdCreateMobileTable = " CREATE TABLE [dbo].[mobiledata](
	[clientid] [nvarchar](50),
	[querytime] [nvarchar](50),
	[market] [nvarchar](50),
	[deviceplatform] [nvarchar](50),
	[devicemake] [nvarchar](50),
	[devicemodel] [nvarchar](50),
	[state] [nvarchar](50),
	[country] [nvarchar](50),
	[querydwelltime] [float],
	[sessionid] [bigint],
	[sessionpagevieworder][bigint])"
	
	$cmdCreateMobileDataClusteredIndex = "CREATE CLUSTERED INDEX mobiledata_clustered_index on mobiledata(clientid)"
	
	# HDInsight variables
	$hdinsightClusterName = $namePrefix + "hdi"
	$defaultStorageAccountName = $namePrefix + "store"
	$defaultBlobContainerName = $hdinsightClusterName
	#endregion
	
	# Treat all errors as terminating
	$ErrorActionPreference = "Stop"
	
	#region - Connect to Azure subscription
	Write-Host "`nConnecting to your Azure subscription ..." -ForegroundColor Green
	try{Get-AzureRmContext}
	catch{Login-AzureRmAccount}
	#endregion
	
	#region - Create Azure resouce group
	Write-Host "`nCreating an Azure resource group ..." -ForegroundColor Green
	try{
		Get-AzureRmResourceGroup -Name $resourceGroupName
	}
	catch{
		New-AzureRmResourceGroup -Name $resourceGroupName -Location $location
	}
	#endregion
	
	#region - Create Azure SQL database server
	Write-Host "`nCreating an Azure SQL Database server ..." -ForegroundColor Green
	try{
		Get-AzureRmSqlServer -ServerName $sqlDatabaseServerName -ResourceGroupName $resourceGroupName}
	catch{
		Write-Host "`nCreating SQL Database server ..."  -ForegroundColor Green
	
		$sqlDatabasePW = ConvertTo-SecureString -String $sqlDatabasePassword -AsPlainText -Force
		$credential = New-Object System.Management.Automation.PSCredential($sqlDatabaseLogin,$sqlDatabasePW)
	
		$sqlDatabaseServerName = (New-AzureRmSqlServer `
									-ResourceGroupName $resourceGroupName `
									-ServerName $sqlDatabaseServerName `
									-SqlAdministratorCredentials $credential `
									-Location $location).ServerName
		Write-Host "`tThe new SQL database server name is $sqlDatabaseServerName." -ForegroundColor Cyan
	
		Write-Host "`nCreating firewall rule, $fireWallRuleName ..." -ForegroundColor Green
		$workstationIPAddress = Invoke-RestMethod $ipAddressRestService
		New-AzureRmSqlServerFirewallRule `
			-ResourceGroupName $resourceGroupName `
			-ServerName $sqlDatabaseServerName `
			-FirewallRuleName "$fireWallRuleName-workstation" `
			-StartIpAddress $workstationIPAddress `
			-EndIpAddress $workstationIPAddress
	
		#To allow other Azure services to access the server add a firewall rule and set both the StartIpAddress and EndIpAddress to 0.0.0.0. 
		#Note that this allows Azure traffic from any Azure subscription to access the server.
		New-AzureRmSqlServerFirewallRule `
			-ResourceGroupName $resourceGroupName `
			-ServerName $sqlDatabaseServerName `
			-FirewallRuleName "$fireWallRuleName-Azureservices" `
			-StartIpAddress "0.0.0.0" `
			-EndIpAddress "0.0.0.0"
	}
	
	#endregion
	
	#region - Create and validate Azure SQL database
	Write-Host "`nCreating an Azure SQL database ..." -ForegroundColor Green
	
	try {
		Get-AzureRmSqlDatabase `
			-ResourceGroupName $resourceGroupName `
			-ServerName $sqlDatabaseServerName `
			-DatabaseName $sqlDatabaseName
	}
	catch {
		Write-Host "`nCreating SQL Database, $sqlDatabaseName ..."  -ForegroundColor Green
		New-AzureRMSqlDatabase `
			-ResourceGroupName $resourceGroupName `
			-ServerName $sqlDatabaseServerName `
			-DatabaseName $sqlDatabaseName `
			-Edition "Standard" `
			-RequestedServiceObjectiveName "S1"
	}
	
	#endregion
	
	#region - Create tables
	Write-Host "Creating the log4jlogs table and the mobiledata table ..." -ForegroundColor Green
	
	$conn = New-Object System.Data.SqlClient.SqlConnection
	$conn.ConnectionString = $sqlDatabaseConnectionString
	$conn.Open()
	
	# Create the log4jlogs table and index
	$cmd = New-Object System.Data.SqlClient.SqlCommand
	$cmd.Connection = $conn
	$cmd.CommandText = $cmdCreateLog4jTable
	$ret = $cmd.ExecuteNonQuery()
	$cmd.CommandText = $cmdCreateLog4jClusteredIndex
	$cmd.ExecuteNonQuery()
	
	# Create the mobiledata table and index
	$cmd.CommandText = $cmdCreateMobileTable
	$cmd.ExecuteNonQuery()
	$cmd.CommandText = $cmdCreateMobileDataClusteredIndex
	$cmd.ExecuteNonQuery()
	
	$conn.close()
	
	#endregion
	
	
	#region - Create HDInsight cluster
	
	Write-Host "Creating the HDInsight cluster and the dependent services ..." -ForegroundColor Green
	
	# Create the default storage account
	New-AzureRmStorageAccount `
		-ResourceGroupName $resourceGroupName `
		-Name $defaultStorageAccountName `
		-Location $location `
		-Type Standard_LRS
	
	# Create the default Blob container
	$defaultStorageAccountKey = Get-AzureRmStorageAccountKey `
									-ResourceGroupName $resourceGroupName `
									-Name $defaultStorageAccountName |  %{ $_.Key1 }
	$defaultStorageAccountContext = New-AzureStorageContext `
										-StorageAccountName $defaultStorageAccountName `
										-StorageAccountKey $defaultStorageAccountKey 
	New-AzureStorageContainer `
		-Name $defaultBlobContainerName `
		-Context $defaultStorageAccountContext 
	
	# Create the HDInsight cluster
	$pw = ConvertTo-SecureString -String $httpPassword -AsPlainText -Force
	$httpCredential = New-Object System.Management.Automation.PSCredential($httpUserName,$pw)
	
	New-AzureRmHDInsightCluster `
		-ResourceGroupName $resourceGroupName `
		-ClusterName $HDInsightClusterName `
		-Location $location `
		-ClusterType Hadoop `
		-OSType Windows `
		-ClusterSizeInNodes 2 `
		-HttpCredential $httpCredential `
		-DefaultStorageAccountName "$defaultStorageAccountName.blob.core.windows.net" `
		-DefaultStorageAccountKey $defaultStorageAccountKey `
		-DefaultStorageContainer $defaultBlobContainerName 
	
	# Validate the cluster
	Get-AzureRmHDInsightCluster -ClusterName $hdinsightClusterName
	#endregion
	
	#region - pre-process the source file
	
	Write-Host "Preprocessing the source file ..." -ForegroundColor Green
	
	# This procedure creates a new file with $destBlobName
	$sourceBlobName = "example/data/sample.log"
	$destBlobName = "tutorials/usesqoop/data/sample.log"
	
	# Define the connection string
	$storageConnectionString = "DefaultEndpointsProtocol=https;AccountName=$defaultStorageAccountName;AccountKey=$defaultStorageAccountKey"
	
	# Create block blob objects referencing the source and destination blob.
	$storageAccount = [Microsoft.WindowsAzure.Storage.CloudStorageAccount]::Parse($storageConnectionString)
	$storageClient = $storageAccount.CreateCloudBlobClient();
	$storageContainer = $storageClient.GetContainerReference($defaultBlobContainerName)
	$sourceBlob = $storageContainer.GetBlockBlobReference($sourceBlobName)
	$destBlob = $storageContainer.GetBlockBlobReference($destBlobName)
	
	# Define a MemoryStream and a StreamReader for reading from the source file
	$stream = New-Object System.IO.MemoryStream
	$stream = $sourceBlob.OpenRead()
	$sReader = New-Object System.IO.StreamReader($stream)
	
	# Define a MemoryStream and a StreamWriter for writing into the destination file
	$memStream = New-Object System.IO.MemoryStream
	$writeStream = New-Object System.IO.StreamWriter $memStream
	
	# Pre-process the source blob
	$exString = "java.lang.Exception:"
	while(-Not $sReader.EndOfStream){
		$line = $sReader.ReadLine()
		$split = $line.Split(" ")
	
		# remove the "java.lang.Exception" from the first element of the array
		# for example: java.lang.Exception: 2012-02-03 19:11:02 SampleClass8 [WARN] problem finding id 153454612
		if ($split[0] -eq $exString){
			#create a new ArrayList to remove $split[0]
			$newArray = [System.Collections.ArrayList] $split
			$newArray.Remove($exString)
	
			# update $split and $line
			$split = $newArray
			$line = $newArray -join(" ")
		}
	
		# remove the lines that has less than 7 elements
		if ($split.count -ge 7){
			write-host $line
			$writeStream.WriteLine($line)
		}
	}
	
	# Write to the destination blob
	$writeStream.Flush()
	$memStream.Seek(0, "Begin")
	$destBlob.UploadFromStream($memStream)
	
	#endregion
	
	#region - export a log file from the cluster to the SQL database
	
	Write-Host "Preprocessing the source file ..." -ForegroundColor Green
	
	$tableName_log4j = "log4jlogs"
	
	# Connection string for Azure SQL Database.
	# Comment if using SQL Server
	$connectionString = "jdbc:sqlserver://$sqlDatabaseServerName.database.windows.net;user=$sqlDatabaseLogin@$sqlDatabaseServerName;password=$sqlDatabasePassword;database=$sqlDatabaseName"
	# Connection string for SQL Server.
	# Uncomment if using SQL Server.
	#$connectionString = "jdbc:sqlserver://$sqlDatabaseServerName;user=$sqlDatabaseLogin;password=$sqlDatabasePassword;database=$sqlDatabaseName"
	
	$exportDir_log4j = "/tutorials/usesqoop/data"
	
	# Submit a Sqoop job
	$sqoopDef = New-AzureRmHDInsightSqoopJobDefinition `
		-Command "export --connect $connectionString --table $tableName_log4j --export-dir $exportDir_log4j --input-fields-terminated-by \0x20 -m 1"
	$sqoopJob = Start-AzureRmHDInsightJob `
					-ClusterName $hdinsightClusterName `
					-HttpCredential $httpCredential `
					-JobDefinition $sqoopDef #-Debug -Verbose
	Wait-AzureRmHDInsightJob `
		-ResourceGroupName $resourceGroupName `
		-ClusterName $hdinsightClusterName `
		-HttpCredential $httpCredential `
		-JobId $sqoopJob.JobId
	
	Write-Host "Standard Error" -BackgroundColor Green
	Get-AzureRmHDInsightJobOutput -ResourceGroupName $resourceGroupName -ClusterName $hdinsightClusterName -DefaultStorageAccountName $defaultStorageAccountName -DefaultStorageAccountKey $defaultStorageAccountKey -DefaultContainer $defaultBlobContainerName -HttpCredential $httpCredential -JobId $sqoopJob.JobId -DisplayOutputType StandardError
	Write-Host "Standard Output" -BackgroundColor Green
	Get-AzureRmHDInsightJobOutput -ResourceGroupName $resourceGroupName -ClusterName $hdinsightClusterName -DefaultStorageAccountName $defaultStorageAccountName -DefaultStorageAccountKey $defaultStorageAccountKey -DefaultContainer $defaultBlobContainerName -HttpCredential $httpCredential -JobId $sqoopJob.JobId -DisplayOutputType StandardOutput
	
	#endregion
	
	#region - export a Hive table
	
	$tableName_mobile = "mobiledata"
	$exportDir_mobile = "/hive/warehouse/hivesampletable"
	
	$sqoopDef = New-AzureRmHDInsightSqoopJobDefinition `
		-Command "export --connect $connectionString --table $tableName_mobile --export-dir $exportDir_mobile --fields-terminated-by \t -m 1"
	$sqoopJob = Start-AzureRmHDInsightJob `
					-ClusterName $hdinsightClusterName `
					-HttpCredential $httpCredential `
					-JobDefinition $sqoopDef #-Debug -Verbose
	
	Wait-AzureRmHDInsightJob `
		-ResourceGroupName $resourceGroupName `
		-ClusterName $hdinsightClusterName `
		-HttpCredential $httpCredential `
		-JobId $sqoopJob.JobId
	
	Write-Host "Standard Error" -BackgroundColor Green
	Get-AzureRmHDInsightJobOutput `
		-ResourceGroupName $resourceGroupName `
		-ClusterName $hdinsightClusterName `
		-DefaultStorageAccountName $defaultStorageAccountName `
		-DefaultStorageAccountKey $defaultStorageAccountKey `
		-DefaultContainer $defaultBlobContainerName `
		-HttpCredential $httpCredential `
		-JobId $sqoopJob.JobId `
		-DisplayOutputType StandardError
	
	Write-Host "Standard Output" -BackgroundColor Green
	Get-AzureRmHDInsightJobOutput `
		-ResourceGroupName $resourceGroupName `
		-ClusterName $hdinsightClusterName `
		-DefaultStorageAccountName $defaultStorageAccountName `
		-DefaultStorageAccountKey $defaultStorageAccountKey `
		-DefaultContainer $defaultBlobContainerName `
		-HttpCredential $httpCredential `
		-JobId $sqoopJob.JobId `
		-DisplayOutputType StandardOutput
	
	#endregion
	
	#region - import a database
	
	$targetDir_mobile = "/tutorials/usesqoop/importeddata/"
	
	$sqoopDef = New-AzureRmHDInsightSqoopJobDefinition `
		-Command "import --connect $connectionString --table $tableName_mobile --target-dir $targetDir_mobile --fields-terminated-by \t --lines-terminated-by \n -m 1"
	
	$sqoopJob = Start-AzureRmHDInsightJob `
					-ClusterName $hdinsightClusterName `
					-HttpCredential $httpCredential `
					-JobDefinition $sqoopDef #-Debug -Verbose
	
	Wait-AzureRmHDInsightJob `
		-ResourceGroupName $resourceGroupName `
		-ClusterName $hdinsightClusterName `
		-HttpCredential $httpCredential `
		-JobId $sqoopJob.JobId
	
	Write-Host "Standard Error" -BackgroundColor Green
	Get-AzureRmHDInsightJobOutput `
		-ResourceGroupName $resourceGroupName `
		-ClusterName $hdinsightClusterName `
		-DefaultStorageAccountName $defaultStorageAccountName `
		-DefaultStorageAccountKey $defaultStorageAccountKey `
		-DefaultContainer $defaultBlobContainerName `
		-HttpCredential $httpCredential `
		-JobId $sqoopJob.JobId `
		-DisplayOutputType StandardError
	
	Write-Host "Standard Output" -BackgroundColor Green
	Get-AzureRmHDInsightJobOutput `
		-ResourceGroupName $resourceGroupName `
		-ClusterName $hdinsightClusterName `
		-DefaultStorageAccountName $defaultStorageAccountName `
		-DefaultStorageAccountKey $defaultStorageAccountKey `
		-DefaultContainer $defaultBlobContainerName `
		-HttpCredential $httpCredential `
		-JobId $sqoopJob.JobId `
		-DisplayOutputType StandardOutput
	
	#endregion

## .NET SDK を使用した Sqoop の実行

このセクションでは、C# コンソール アプリケーションを作成し、このチュートリアルで作成した SQL Database テーブルに hivesampletable をエクスポートします。

**Sqoop ジョブを送信するには**

1. Visual Studio パッケージ マネージャー、コンソールから、次の Nuget コマンドを実行し、パッケージをインポートします。

		Install-Package Microsoft.Azure.Management.HDInsight.Job -Pre
2. Program.cs ファイルで次の using ステートメントを使用します。

		using System;
		using Microsoft.Azure.Management.HDInsight.Job;
		using Microsoft.Azure.Management.HDInsight.Job.Models;
		using Hyak.Common;
3. Main() 関数に次のコードを追加します。HDInsight .NET SDK の使用に関する一般的な情報については、「[プログラムによる Hadoop ジョブの送信][hdinsight-submit-jobs]」を参照してください。

		var ExistingClusterName = "<HDInsightClusterName>";
		var ExistingClusterUri = ExistingClusterName + ".azurehdinsight.net";
		var ExistingClusterUsername = "<HDInsightClusterHttpUsername>";
		var ExistingClusterPassword = "<HDInsightClusterHttpUserPassword>";
		
		var sqlDatabaseServerName = "<AzureSQLDatabaseServerName>";
		var sqlDatabaseLogin = "<AzureSQLDatabaseLogin>";
		var sqlDatabaseLoginPassword = "<AzureSQLDatabaseLoginPassword>";
		var sqlDatabaseDatabaseName = "<AzureSQLDatabaseDatabaseName>";
		
		var sqlDatabaseTableName = "log4jlogs";
		var exportDir = "/hive/warehouse/hivesampletable";

		var cmdExport = @"export";
		// Connection string for using Azure SQL Database.
		// Comment if using SQL Server
		cmdExport = cmdExport + @" --connect 'jdbc:sqlserver://" + sqlDatabaseServerName + ".database.windows.net;user=" + sqlDatabaseLogin + "@" + sqlDatabaseServerName + ";password=" + sqlDatabaseLoginPassword + ";database=" + sqlDatabaseDatabaseName +"'"; 
		// Connection string for using SQL Server.
		// Uncomment if using SQL Server
		//cmdExport = cmdExport + @" --connect jdbc:sqlserver://" + sqlDatabaseServerName + ";user=" + sqlDatabaseLogin + ";password=" + sqlDatabaseLoginPassword + ";database=" + sqlDatabaseDatabaseName;
		cmdExport = cmdExport + @" --table " + sqlDatabaseTableName;
		cmdExport = cmdExport + @" --export-dir " + exportDir;
		cmdExport = cmdExport + @" --input-fields-terminated-by \0x20 -m 1";
		
		HDInsightJobManagementClient _hdiJobManagementClient;
		var clusterCredentials = new BasicAuthenticationCloudCredentials { Username = ExistingClusterUsername, Password = ExistingClusterPassword };
		_hdiJobManagementClient = new HDInsightJobManagementClient(ExistingClusterUri, clusterCredentials);

		var parameters = new SqoopJobSubmissionParameters
		{
		    UserName = ExistingClusterUsername,
		    Command = cmdExport
		};
		
		System.Console.WriteLine("Submitting the Sqoop job to the cluster...");
		var response = _hdiJobManagementClient.JobManagement.SubmitSqoopJob(parameters);
		System.Console.WriteLine("Validating that the response is as expected...");
		System.Console.WriteLine("Response status code is " + response.StatusCode);
		System.Console.WriteLine("Validating the response object...");
		System.Console.WriteLine("JobId is " + response.JobSubmissionJsonResponse.Id);
		Console.WriteLine("Press ENTER to continue ...");
		Console.ReadLine();
4. **F5** キーを押して、プログラムを実行します。 




##次のステップ

ここでは Sqoop の使用方法を説明しました。詳細については、次を参照してください。

- [HDInsight での Oozie の使用][hdinsight-use-oozie]\: Oozie ワークフローで Sqoop アクションを使用します。
- [HDInsight を使用したフライト遅延データの分析][hdinsight-analyze-flight-data]\: Hive を使用してフライト遅延データを分析し、Sqoop を使用して Azure SQL データベースにデータをエクスポートします。
- [HDInsight へのデータのアップロード][hdinsight-upload-data]\: HDInsight/Azure BLOB ストレージにデータをアップロードするその他の方法を説明します。

[azure-management-portal]: https://portal.azure.com/

[hdinsight-versions]: hdinsight-component-versioning.md
[hdinsight-provision]: hdinsight-provision-clusters.md
[hdinsight-get-started]: hdinsight-hadoop-linux-tutorial-get-started.md
[hdinsight-storage]: ../hdinsight-hadoop-use-blob-storage.md
[hdinsight-analyze-flight-data]: hdinsight-analyze-flight-delay-data.md
[hdinsight-use-oozie]: hdinsight-use-oozie.md
[hdinsight-upload-data]: hdinsight-upload-data.md
[hdinsight-submit-jobs]: hdinsight-submit-hadoop-jobs-programmatically.md

[sqldatabase-get-started]: ../sql-database-get-started.md
[sqldatabase-create-configue]: ../sql-database-create-configure.md

[powershell-start]: http://technet.microsoft.com/library/hh847889.aspx
[powershell-install]: powershell-install-configure.md
[powershell-script]: http://technet.microsoft.com/library/ee176949.aspx

[sqoop-user-guide-1.4.4]: https://sqoop.apache.org/docs/1.4.4/SqoopUserGuide.html

<!---HONumber=AcomDC_0218_2016-->
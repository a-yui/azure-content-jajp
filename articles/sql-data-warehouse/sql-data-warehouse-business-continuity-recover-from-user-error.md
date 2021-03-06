<properties
   pageTitle="SQL Data Warehouse でデータベースをユーザー エラーから復旧する |Microsoft Azure"
   description="SQL Data Warehouse でデータベースをユーザー エラーから復旧するための手順です。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="sahaj08"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-services"
   ms.date="02/17/2016"
   ms.author="sahajs;barbkess;sonyama"/>

# SQL Data Warehouse でのデータベースのユーザー エラーからの復旧

SQL Data Warehouse には、意図しないデータの破損または削除の原因となるユーザー エラーから復旧するための主な機能が 2 つあります。

- ライブ データベースの復元
- 削除されたデータベースの復元

これらの機能はいずれも、同じサーバー上の新しいデータベースに復元を実行します。

SQL Data Warehouse データベースの復元をサポートする API には、Azure PowerShell と REST API の 2 種類があります。いずれかを使用して、SQL Data Warehouse の復元機能にアクセスすることができます。

## ライブ データベースの復元
ユーザー エラーが意図しないデータ変更の原因になっている場合は、保有期間内の復元ポイントのいずれかにデータベースを復元できます。ライブ データベースのデータベース スナップショットは少なくとも 8 時間ごとに作成され、7 日間保持されます。

### PowerShell

Azure PowerShell を使用して、プログラムでデータベースの復元を実行します。Azure PowerShell モジュールをダウンロードするには、[Microsoft Web Platform Installer](http://go.microsoft.com/fwlink/p/?linkid=320376&clcid=0x409) を実行します。Get-Module -ListAvailable -Name Azure を実行することで、バージョンを確認できます。この記事は、Microsoft Azure PowerShell バージョン 1.0.4 に基づいています。

データベースを復元するには、[Start-AzureSqlDatabaseRestore][] コマンドレットを使用します。

1. Windows PowerShell を開きます。
2. Azure アカウントに接続して、アカウントに関連付けられているすべてのサブスクリプションを一覧表示します。
3. 復元するデータベースを含むサブスクリプションを選択します。
4. データベースの復元ポイントを一覧表示します (Azure リソース管理モードにする必要があります)。
5. RestorePointCreationDate を使用して、目的の復元ポイントを選択します。
6. 目的の復元ポイントに、データベースを復元します。
7. 復元の進行状況を監視します。

```

Login-AzureRmAccount
Get-AzureRmSubscription
Select-AzureRmSubscription -SubscriptionName "<Subscription_name>"

# List the last 10 database restore points
((Get-AzureRMSqlDatabaseRestorePoints -ServerName "<YourServerName>" -DatabaseName "<YourDatabaseName>" -ResourceGroupName "<YourResourceGroupName>").RestorePointCreationDate)[-10 .. -1]

	# Or for all restore points
	Get-AzureRmSqlDatabaseRestorePoints -ServerName "<YourServerName>" -DatabaseName "<YourDatabaseName>" -ResourceGroupName "<YourResourceGroupName>"

# Pick desired restore point using RestorePointCreationDate
$PointInTime = "<RestorePointCreationDate>"

# Get the specific database name to restore
(Get-AzureRmSqlDatabase -ServerName "<YourServerName>" -ResourceGroupName "<YourResourceGroupName>").DatabaseName | where {$_ -ne "master" }
#or
Get-AzureRmSqlDatabase -ServerName "<YourServerName>" –ResourceGroupName "<YourResourceGroupName>"

# Restore database
$RestoreRequest = Start-AzureSqlDatabaseRestore -SourceServerName "<YourServerName>" -SourceDatabaseName "<YourDatabaseName>" -TargetDatabaseName "<NewDatabaseName>" -PointInTime $PointInTime

# Monitor progress of restore operation
Get-AzureSqlDatabaseOperation -ServerName "<YourServerName>" –OperationGuid $RestoreRequest.RequestID
```

サーバーが foo.database.windows.net の場合は、上の PowerShell コマンドレットで -ServerName として "foo" を使用してください。

### REST API
プログラムでデータベースの復元を実行するには、REST を使用します。

1. 「データベース復元ポイントの取得」操作を使用して、データベースの復元ポイントの一覧を取得します。
2. 「[データベースの復元要求の作成][]」の操作を使用して、復元を開始します。
3. 「[データベース操作の状態][]」の操作を使用して、復元の状態を追跡します。

復元が終了したら、[Finalize a recovered database (復旧データベースの最終処理)][] に関するガイドに従って、復旧されたデータベースを構成できます。

## 削除されたデータベースの復旧
データベースが削除された場合、削除されたデータベースを削除時の状態に復元することができます。Azure SQL Data Warehouse では、データベースが削除され 7 日間保持される前に、データベースのスナップショットが作成されます。

### PowerShell
Azure PowerShell を使用して、削除済みデータベースの復元をプログラムで実行します。Azure PowerShell モジュールをダウンロードするには、[Microsoft Web Platform Installer](http://go.microsoft.com/fwlink/p/?linkid=320376&clcid=0x409) を実行します。

削除済みデータベースを復元するには、[Start-AzureSqlDatabaseRestore][] コマンドレットを使用します。

1. Microsoft Azure PowerShell を開きます。
2. Azure アカウントに接続して、アカウントに関連付けられているすべてのサブスクリプションを一覧表示します。
3. 復元する削除済みデータベースを含むサブスクリプションを選択します。
4. 削除済みデータベースの一覧で、データベースと削除日を確認します。

```
Get-AzureSqlDatabase -RestorableDropped -ServerName "<YourServerName>"
```

5. 削除された特定のデータベースを取得し、復元を開始します。

```
$Database = Get-AzureSqlDatabase -RestorableDropped -ServerName "<YourServerName>" –DatabaseName "<YourDatabaseName>" -DeletionDate "1/01/2015 12:00:00 AM"

$RestoreRequest = Start-AzureSqlDatabaseRestore -SourceRestorableDroppedDatabase $Database –TargetDatabaseName "<NewDatabaseName>"

Get-AzureSqlDatabaseOperation –ServerName "<YourServerName>" –OperationGuid $RestoreRequest.RequestID
```

サーバーが foo.database.windows.net の場合は、上の PowerShell コマンドレットで -ServerName として "foo" を使用してください。

### REST API
プログラムでデータベースの復元を実行するには、REST を使用します。

1.	「[削除された復元可能なデータベースの一覧表示][]」の操作を使用して、復元可能なすべての削除済みデータベースのリストを表示します。
2.	「[削除された復元可能なデータベースの取得][]」の操作を使用して、復元する削除済みデータベースの詳細を取得します。
3.	「[データベースの復元要求の作成][]」の操作を使用して、復元を開始します。
4.	「[データベース操作の状態][]」の操作を使用して、復元の状態を追跡します。

復元が終了したら、[Finalize a recovered database (復旧データベースの最終処理)][] に関するガイドに従って、復旧されたデータベースを構成できます。


## 次のステップ
Azure SQL Database のその他のエディションのビジネス継続性については、[Azure SQL Database のビジネス継続性の概要][]を参照してください。


<!--Image references-->

<!--Article references-->
[Azure SQL Database のビジネス継続性の概要]: sql-database/sql-database-business-continuity.md
[Finalize a recovered database (復旧データベースの最終処理)]: sql-database/sql-database-recovered-finalize.md

<!--MSDN references-->
[データベースの復元要求の作成]: http://msdn.microsoft.com/library/azure/dn509571.aspx
[データベース操作の状態]: http://msdn.microsoft.com/library/azure/dn720371.aspx
[削除された復元可能なデータベースの取得]: http://msdn.microsoft.com/library/azure/dn509574.aspx
[削除された復元可能なデータベースの一覧表示]: http://msdn.microsoft.com/library/azure/dn509562.aspx
[Start-AzureSqlDatabaseRestore]: https://msdn.microsoft.com/library/dn720218.aspx

<!--Other Web references-->

<!---HONumber=AcomDC_0224_2016-->
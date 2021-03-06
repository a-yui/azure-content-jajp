<properties 
	pageTitle="Azure Redis Cache の使用方法" 
	description="Azure Redis Cache を使用して Azure アプリケーションのパフォーマンスを向上させる方法を説明します" 
	services="redis-cache,app-service" 
	documentationCenter="" 
	authors="steved0x" 
	manager="erikre" 
	editor=""/>

<tags 
	ms.service="cache" 
	ms.workload="tbd" 
	ms.tgt_pltfrm="cache-redis" 
	ms.devlang="dotnet" 
	ms.topic="hero-article" 
	ms.date="01/21/2016" 
	ms.author="sdanie"/>

# Azure Redis Cache の使用方法

> [AZURE.SELECTOR]
- [.NET](cache-dotnet-how-to-use-azure-redis-cache.md)
- [Node.JS](cache-nodejs-get-started.md)
- [Java](cache-java-get-started.md)
- [Python](cache-python-get-started.md)

このガイドでは、**Azure Redis Cache** の基本的な使用方法について説明します。Microsoft Azure Redis Cache は、広く支持されているオープン ソースの Redis Cache がベースとなっています。マイクロソフトによって管理されている、セキュリティで保護された専用の Redis cache にアクセスすることができます。Azure Redis Cache を使用して作成されたキャッシュには、Microsoft Azure 内のあらゆるアプリケーションからアクセスすることができます。

Microsoft Azure Redis Cache には、次のレベルがあります。

-	**Basic** – 単一ノード、複数のサイズ、最大 53 GB
-	**Standard** – 2 ノード (プライマリ/レプリカ)。複数のサイズ、最大 53 GB99.9% の SLA。
-	**Premium** – 最大 10 個のシャードがある 2 ノード (プライマリ/レプリカ)。6 GB から 530 GB までの複数のサイズ (詳細はお問い合わせください)。Standard レベルのすべての機能と、[Redis クラスター](cache-how-to-premium-clustering.md)、[Redis の永続化](cache-how-to-premium-persistence.md)、[Azure Virtual Network](cache-how-to-premium-vnet.md) のサポートを含むその他の機能。99.9% の SLA。

各レベルは、機能と価格ごとに異なります。価格の詳細については、[Cache の価格詳細][]に関するページを参照してください。

このガイドでは、C# を使用する [StackExchange.Redis][] クライアントの使用方法について説明します。紹介するシナリオは、**キャッシュの作成と構成**、**キャッシュ クライアントの構成**、**キャッシュでのオブジェクトの追加と削除**などです。Azure Redis Cache の使用方法の詳細については、「[次のステップ][]」を参照してください。

<a name="getting-started-cache-service"></a>
## Azure Redis Cache の使用

Azure Redis Cache の導入は簡単です。使い始めるには、キャッシュをプロビジョニングして構成します。次に、キャッシュ クライアントを構成してキャッシュにアクセスできるようにします。キャッシュ クライアントを構成すると、使い始めることができます。

-	[キャッシュの作成][]
-	[キャッシュ クライアントの構成][]

<a name="create-cache"></a>
## キャッシュの作成

キャッシュを作成するには、まず [Azure ポータル][]にサインインし、**[新規]**、**[データ + ストレージ]**、**[Redis Cache]** の順にクリックします。

>[AZURE.NOTE] キャッシュは、Azure ポータルだけでなく、ARM テンプレート、PowerShell、または Azure CLI を使用して作成することもできます。
>
>-	ARM テンプレートを使用してキャッシュを作成する方法については、「[テンプレートを使用して Redis Cache を作成する](cache-redis-cache-arm-provision.md)」をご覧ください。
>-	Azure PowerShell を使用してキャッシュを作成する方法については、「[Azure PowerShell を使用した Azure Redis Cache の管理](cache-howto-manage-redis-cache-powershell.md)」をご覧ください。
>-	Azure CLI を使用してキャッシュを作成する方法については、「[Azure コマンド ライン インターフェイス (Azure CLI) を使用して Azure Redis Cache を作成および管理する方法](cache-manage-cli.md)」を参照してください。

![新しいキャッシュ][NewCacheMenu]

>[AZURE.NOTE] Azure アカウントがない場合は、無料アカウントを数分で作成することができます。詳細については、[Azure の無料試用版サイト][]を参照してください。

**[Redis Cache の新規作成]** ブレードで、必要なキャッシュ構成を指定します。

![キャッシュの作成][CacheCreate]

-	キャッシュ エンドポイントに使用するキャッシュ名を **[DNS 名]** に入力します。キャッシュ名は 1 ～ 63 文字の文字列で、数字、英字、`-` 文字だけを使用できます。キャッシュ名では、先頭と末尾の `-` 文字および連続する `-` 文字は無効です。
-	**[サブスクリプション]** で、キャッシュに使用する Azure サブスクリプションを選択します。アカウントにサブスクリプションが 1 つしかない場合は自動的に選択されるため、**[サブスクリプション]** ドロップダウン リストは表示されません。
-	**[リソース グループ]** で、キャッシュのリソース グループを選択または作成します。詳細については、[リソース グループを使用した Azure リソースの管理][]に関するページを参照してください。 
-	**[場所]** を使用して、キャッシュのホストの地理的位置を指定します。パフォーマンスを最大限に引き出すために、キャッシュは、キャッシュ クライアント アプリケーションと同じリージョンに作成することを強くお勧めします。
-	**[価格レベル]** を使用して、必要なキャッシュ サイズと機能を選択します。
-	**Redis クラスター**では、53 GB を超えるキャッシュを作成でき、複数の Redis ノード間でデータを共有することもできます。詳細については、「[Premium Azure Redis Cache のクラスタリングの構成方法](cache-how-to-premium-clustering.md)」を参照してください。
-	**Redis の永続化**を使用して、Azure ストレージ アカウントにキャッシュを保持できます。永続化の構成手順については、「[Premium Azure Redis Cache の永続性の構成方法](cache-how-to-premium-persistence.md)」を参照してください。
-	**Virtual Network** では、指定された Azure Virtual Network 内にあるクライアントのみにキャッシュへのアクセス権を制限することで、セキュリティと分離が強化されます。サブネット、アクセス制御ポリシー、およびその他の Redis へのアクセスをさらに制限する機能を始め、VNet のすべての機能を使用できます。詳細については、「[Premium Azure Redis Cache の Virtual Network のサポートを構成する方法](cache-how-to-premium-vnet.md)」を参照してください。
-	**診断**を使用して、キャッシュ メトリックにストレージ アカウントを指定できます。キャッシュ メトリックの構成と表示の詳細については、「[Azure Redis Cache の監視方法](cache-how-to-monitor.md)」を参照してください。

新しいキャッシュ オプションを構成したら、**[作成]** をクリックします。キャッシュが作成されるまで数分かかる場合があります。状態を確認するには、スタート画面で進行状況を監視してください。キャッシュが作成されると、新しいキャッシュの状態が "**実行中**" になって、既定の設定で使用できるようになります。

![作成されたキャッシュ][CacheCreated]

作成されたキャッシュには、**[参照]** ブレードからアクセスすることができます。

![ブレードの表示][BrowseCaches]

**[Redis Caches]** をクリックしてキャッシュを表示します。

![キャッシュ][Caches]

<a name="NuGet"></a>
## キャッシュ クライアントの構成

Azure Redis Cache を使用して構成されたキャッシュは、あらゆる Azure アプリケーションからアクセスできます。Visual Studio で開発した .NET アプリケーションであれば、**StackExchange.Redis** キャッシュ クライアントを使用できます。NuGet パッケージを使用すると、このキャッシュ クライアント アプリケーションを簡単に構成できます。

>[AZURE.NOTE] 詳細については、GitHub の [StackExchange.Redis に関するページ][]と [StackExchange.Redis キャッシュ クライアントのドキュメント][]を参照してください。

Visual Studio で StackExchange.Redis NuGet パッケージを使用してクライアント アプリケーションを構成するには、**ソリューション エクスプローラー**でプロジェクトを右クリックし、**[NuGet パッケージの管理]** をクリックします。

![Manage NuGet packages][NuGetMenu]

**[オンライン検索]** ボックスに「**StackExchange.Redis**」または「**StackExchange.Redis.StrongName**」と入力し、結果の中から必要なバージョンを選択して、**[インストール]** をクリックします。

>[AZURE.NOTE] 厳密な名前を持つバージョンの **StackExchange.Redis** クライアント ライブラリを希望する場合は、**[StackExchange.Redis.StrongName]** を選択してください。それ以外の場合は、**[StackExchange.Redis]** を選択します。

![StackExchange.Redis NuGet package][StackExchangeNuget]

クライアント アプリケーションから StackExchange.Redis Cache クライアントを使用して Azure Redis Cache にアクセスするために必要なアセンブリ参照が NuGet パッケージによってダウンロードされ追加されます。

クライアント プロジェクトをキャッシュ用に構成できたら、以降のセクションで説明されている、キャッシュを操作するための技法を使用できます。

<a name="working-with-caches"></a>
## キャッシュの操作

このセクションの手順では、キャッシュに対する一般的なタスクを行う方法について説明します。

-	[キャッシュに接続する][]
-   [オブジェクトをキャッシュに追加する、キャッシュから削除する][]
-   [キャッシュ内で .NET オブジェクトを使用する](#work-with-net-objects-in-the-cache)

<a name="connect-to-cache"></a>
## キャッシュに接続する

プログラムでキャッシュを操作するには、キャッシュへの参照が必要です。StackExchange.Redis クライアントを使用して Azure Redis Cache にアクセスするすべてのファイルの先頭に次のコードを追加します。

    using StackExchange.Redis;

>[AZURE.NOTE] StackExchange.Redis クライアントには、.NET Framework 4 以降が必要です。

Azure Redis Cache への接続には、`ConnectionMultiplexer` クラスを使用します。このクラスはクライアント アプリケーションの開始から終了まで共有、再利用する前提で設計されており、操作単位で作成する必要はありません。

Azure Redis Cache に接続して、接続済みの `ConnectionMultiplexer` インスタンスを取得するには、次の例のように、静的 `Connect` メソッドを呼び出して、キャッシュのエンドポイントとキーを渡します。password パラメーターには、Azure ポータルから生成されたキーを使用してください。

	ConnectionMultiplexer connection = ConnectionMultiplexer.Connect("contoso5.redis.cache.windows.net,abortConnect=false,ssl=true,password=...");

>[AZURE.IMPORTANT] 警告: ソース コード内に資格情報を保存することは絶対に避けてください。このサンプルでは、単純化するためにあえてソース コード内に記述しています。資格情報を保存する方法については、「[How Application Strings and Connection Strings Work (アプリケーション文字列と接続文字列の動作)][]」を参照してください。

SSL を使用しない場合は、`ssl=false` を設定するか、`ssl` パラメーターを省略します。

>[AZURE.NOTE] 既定では、新しいキャッシュに対して非 SSL ポートは無効になっています。非 SSL ポートを有効にする手順については、「[アクセス ポート](cache-configure.md#access-ports)」を参照してください。

アプリケーション内の `ConnectionMultiplexer` インスタンスを共有する方法の 1 つは、次の例のように、接続されたインスタンスを返す静的プロパティの設定です。これにより、接続された `ConnectionMultiplexer` インスタンス 1 つのみがスレッド セーフな方法で初期化されます。これらの例では、`abortConnect` が false に設定されており、Azure Redis Cache への接続が確立されていない場合でも呼び出しが成功します。`ConnectionMultiplexer` の主な機能の 1 つは、ネットワーク問題などの原因が解決されると、キャッシュへの接続が自動的に復元されることです。

	private static Lazy<ConnectionMultiplexer> lazyConnection = new Lazy<ConnectionMultiplexer>(() =>
	{
	    return ConnectionMultiplexer.Connect("contoso5.redis.cache.windows.net,abortConnect=false,ssl=true,password=...");
	});
	
	public static ConnectionMultiplexer Connection
	{
	    get
	    {
	        return lazyConnection.Value;
	    }
	}

高度な接続構成オプションの詳細については、[StackExchange.Redis の構成モデル][]を参照してください。

キャッシュのエンドポイントとキーは、ご利用のキャッシュ インスタンスの **Redis Cache** ブレードから取得できます。

![キャッシュのプロパティ][CacheProperties]

![キーの管理][ManageKeys]

接続が確立されたら、`ConnectionMultiplexer.GetDatabase` メソッドを呼び出して Redis Cache データベースへの参照を取得します。`GetDatabase` メソッドから返されるオブジェクトは、手付かずで受け渡しされる軽量のオブジェクトであり、保存する必要はありません。

	// Connection refers to a property that returns a ConnectionMultiplexer
	// as shown in the previous example.
	IDatabase cache = Connection.GetDatabase();

	// Perform cache operations using the cache object...
	// Simple put of integral data types into the cache
	cache.StringSet("key1", "value");
	cache.StringSet("key2", 25);

	// Simple get of data types from the cache
	string key1 = cache.StringGet("key1");
	int key2 = (int)cache.StringGet("key2");

以上、Azure Redis Cache インスタンスに接続して、キャッシュ データベースへの参照を取得する方法を説明しました。今度は実際にキャッシュを使用してみましょう。

<a name="add-object"></a>
## オブジェクトをキャッシュに追加する、キャッシュから削除する

キャッシュに項目を格納する、またはキャッシュから項目を取得するには、`StringSet` メソッドと `StringGet` メソッドを使用します。

	// If key1 exists, it is overwritten.
	cache.StringSet("key1", "value1");

	string value = cache.StringGet("key1");

Redis では、ほとんどのデータが Redis 文字列として保存されますが、これらの文字列には、さまざまなデータ型を格納することができます。シリアル化したバイナリ データもその 1 つで、.NET のオブジェクトをキャッシュに保存する際に使用することができます。

`StringGet` を呼び出すと、オブジェクトが存在する場合はそのオブジェクトが返され、存在しない場合は `null` が返されます。その場合、目的のデータ ソースから値を取得してキャッシュに格納しておき、後で使用することができます。これを "キャッシュ アサイド パターン" といいます。

    string value = cache.StringGet("key1");
    if (value == null)
    {
        // The item keyed by "key1" is not in the cache. Obtain
        // it from the desired data source and add it to the cache.
        value = GetValueFromDataSource();

        cache.StringSet("key1", value);
    }

キャッシュ内の項目の有効期限を指定するには、`StringSet` の `TimeSpan` パラメーターを使用します。

	cache.StringSet("key1", "value1", TimeSpan.FromMinutes(90));

## キャッシュ内で .NET オブジェクトを使用する

Azure Redis Cache はプリミティブ データ型に加え、.NET オブジェクトをキャッシュできますが、.NET オブジェクトをキャッシュするためには、あらかじめシリアル化しておく必要があります。この処理はアプリケーション開発者が行わなければなりません。逆にそのことでシリアライザーの選択に幅が生まれ、開発者にとってのメリットとなっています。

オブジェクトをシリアル化する簡単な方法の 1 つは、[Newtonsoft.Json.NET](https://www.nuget.org/packages/Newtonsoft.Json/8.0.1-beta1) の `JsonConvert` シリアル化方法を使用して、JSON へおよび JSON からシリアル化する方法です。次の例では、`Employee` オブジェクト インスタンスを使用した get および set を示しています。


	class Employee
	{
	    public int Id { get; set; }
	    public string Name { get; set; }
	
	    public Employee(int EmployeeId, string Name)
	    {
	        this.Id = EmployeeId;
	        this.Name = Name;
	    }
	}

    // Store to cache
    cache.StringSet("e25", JsonConvert.SerializeObject(new Employee(25, "Clayton Gragg")));

    // Retrieve from cache
    Employee e25 = JsonConvert.DeserializeObject<Employee>(cache.StringGet("e25"));

<a name="next-steps"></a>
## 次のステップ

これで、基本を学習できました。Azure Redis Cache の詳細については、次のリンク先を参照してください。

-	Azure Redis Cache の ASP.NET プロバイダーを参照してください。
	-	[Azure Redis セッション状態プロバイダー](cache-asp.net-session-state-provider.md)
	-	[Azure Redis Cache ASP.NET 出力キャッシュ プロバイダー](cache-asp.net-output-cache-provider.md)
-	[キャッシュ診断の有効化](cache-how-to-monitor.md#enable-cache-diagnostics)によってキャッシュの正常性を[監視](cache-how-to-monitor.md)できるようにします。Azure ポータルでメトリックを表示できますが、お好みのツールを使用して、メトリックを[ダウンロードして確認](https://github.com/rustd/RedisSamples/tree/master/CustomMonitoring)することも可能です。
-	[StackExchange.Redis キャッシュ クライアントのドキュメント][]を参照してください。
	-	Azure Redis Cache は、さまざまな Redis クライアントや開発言語からアクセスできます。詳細については、[http://redis.io/clients][] および「[他の言語での Azure Redis Cache の開発][]」を参照してください。
	-	Azure Redis Cache は、Redsmin などのサービスと共に使用することもできます。詳細については、「[Azure Redis 接続文字列を取得し、Redsmin と共に使用する方法][]」を参照してください。
-	[redis][] のドキュメント、[redis のデータ型に関するページ][]、[redis のデータ型の概念に関するページ][]を参照してください。



<!-- INTRA-TOPIC LINKS -->
[次のステップ]: #next-steps
[Introduction to Azure Redis Cache (Video)]: #video
[What is Azure Redis Cache?]: #what-is
[Create an Azure Cache]: #create-cache
[Which type of caching is right for me?]: #choosing-cache
[Prepare Your Visual Studio Project to Use Azure Caching]: #prepare-vs
[Configure Your Application to Use Caching]: #configure-app
[Get Started with Azure Redis Cache]: #getting-started-cache-service
[キャッシュの作成]: #create-cache
[Configure the cache]: #enable-caching
[キャッシュ クライアントの構成]: #NuGet
[Working with Caches]: #working-with-caches
[キャッシュに接続する]: #connect-to-cache
[オブジェクトをキャッシュに追加する、キャッシュから削除する]: #add-object
[Specify the expiration of an object in the cache]: #specify-expiration
[ASP.NET セッション状態をキャッシュに格納する]: #store-session

  
<!-- IMAGES -->
[NewCacheMenu]: ./media/cache-dotnet-how-to-use-azure-redis-cache/redis-cache-new-cache-menu.png

[CacheCreate]: ./media/cache-dotnet-how-to-use-azure-redis-cache/redis-cache-cache-create.png

[StackExchangeNuget]: ./media/cache-dotnet-how-to-use-azure-redis-cache/redis-cache-stackexchange-redis.png

[NuGetMenu]: ./media/cache-dotnet-how-to-use-azure-redis-cache/redis-cache-manage-nuget-menu.png

[CacheProperties]: ./media/cache-dotnet-how-to-use-azure-redis-cache/redis-cache-properties.png

[ManageKeys]: ./media/cache-dotnet-how-to-use-azure-redis-cache/redis-cache-manage-keys.png

[SessionStateNuGet]: ./media/cache-dotnet-how-to-use-azure-redis-cache/redis-cache-session-state-provider.png

[BrowseCaches]: ./media/cache-dotnet-how-to-use-azure-redis-cache/redis-cache-browse-caches.png

[Caches]: ./media/cache-dotnet-how-to-use-azure-redis-cache/redis-cache-caches.png

[CacheCreated]: ./media/cache-dotnet-how-to-use-azure-redis-cache/redis-cache-cache-created.png




   
<!-- LINKS -->
[http://redis.io/clients]: http://redis.io/clients
[他の言語での Azure Redis Cache の開発]: http://msdn.microsoft.com/library/azure/dn690470.aspx
[Azure Redis 接続文字列を取得し、Redsmin と共に使用する方法]: https://redsmin.uservoice.com/knowledgebase/articles/485711-how-to-connect-redsmin-to-azure-redis-cache
[Azure Redis セッション状態プロバイダー]: http://go.microsoft.com/fwlink/?LinkId=398249
[How to: Configure a Cache Client Programmatically]: http://msdn.microsoft.com/library/windowsazure/gg618003.aspx
[Session State Provider for Azure Cache]: http://go.microsoft.com/fwlink/?LinkId=320835
[Azure AppFabric Cache: Caching Session State]: http://www.microsoft.com/showcase/details.aspx?uuid=87c833e9-97a9-42b2-8bb1-7601f9b5ca20
[Output Cache Provider for Azure Cache]: http://go.microsoft.com/fwlink/?LinkId=320837
[Azure Shared Caching]: http://msdn.microsoft.com/library/windowsazure/gg278356.aspx
[Team Blog]: http://blogs.msdn.com/b/windowsazure/
[Azure Caching]: http://www.microsoft.com/showcase/Search.aspx?phrase=azure+caching
[How to Configure Virtual Machine Sizes]: http://go.microsoft.com/fwlink/?LinkId=164387
[Azure Caching Capacity Planning Considerations]: http://go.microsoft.com/fwlink/?LinkId=320167
[Azure Caching]: http://go.microsoft.com/fwlink/?LinkId=252658
[How to: Set the Cacheability of an ASP.NET Page Declaratively]: http://msdn.microsoft.com/library/zd1ysf1y.aspx
[How to: Set a Page's Cacheability Programmatically]: http://msdn.microsoft.com/library/z852zf6b.aspx
[Azure Redis Cache でのキャッシュの構成]: http://msdn.microsoft.com/library/azure/dn793612.aspx

[StackExchange.Redis の構成モデル]: http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Configuration.md

[キャッシュ内で .NET オブジェクトを使用する]: http://msdn.microsoft.com/library/dn690521.aspx#Objects


[NuGet Package Manager Installation]: http://go.microsoft.com/fwlink/?LinkId=240311
[Cache の価格詳細]: http://www.windowsazure.com/pricing/details/cache/
[Azure ポータル]: https://portal.azure.com/

[Azure Redis Cache の概要に関するページ]: http://go.microsoft.com/fwlink/?LinkId=320830
[Azure Redis Cache]: http://go.microsoft.com/fwlink/?LinkId=398247

[Migrate to Azure Redis Cache]: http://go.microsoft.com/fwlink/?LinkId=317347
[Azure Redis Cache Samples]: http://go.microsoft.com/fwlink/?LinkId=320840
[リソース グループを使用した Azure リソースの管理]: http://azure.microsoft.com/documentation/articles/resource-group-overview/

[StackExchange.Redis]: http://github.com/StackExchange/StackExchange.Redis
[StackExchange.Redis に関するページ]: http://github.com/StackExchange/StackExchange.Redis
[StackExchange.Redis キャッシュ クライアントのドキュメント]: http://github.com/StackExchange/StackExchange.Redis#documentation

[Redis]: http://redis.io/documentation
[redis のデータ型に関するページ]: http://redis.io/topics/data-types
[redis のデータ型の概念に関するページ]: http://redis.io/topics/data-types-intro

[How Application Strings and Connection Strings Work (アプリケーション文字列と接続文字列の動作)]: http://azure.microsoft.com/blog/2013/07/17/windows-azure-web-sites-how-application-strings-and-connection-strings-work/

[Azure の無料試用版サイト]: http://azure.microsoft.com/pricing/free-trial/?WT.mc_id=redis_cache_hero

<!---HONumber=AcomDC_0309_2016-->
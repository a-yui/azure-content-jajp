<properties 
   pageTitle="Data Lake Tools for Visual Studio を使用する U-SQL スクリプトの開発 | Azure" 
   description="Data Lake Tools for Visual Studio のインストール方法と、U-SQL スクリプトの開発およびテスト方法について説明します。" 
   services="data-lake-analytics" 
   documentationCenter="" 
   authors="mumian" 
   manager="paulettm" 
   editor="cgronlun"/>
 
<tags
   ms.service="data-lake-analytics"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="big-data" 
   ms.date="01/04/2016"
   ms.author="jgao"/>

# チュートリアル: Azure Data Lake Analytics U-SQL 言語の使用

U-SQL は、SQL のメリットと、任意の規模ですべてのデータを処理する独自のコードの表現力を融合した言語です。U-SQL のスケーラブルな分散クエリ機能を使用することで、ストアおよび Azure SQL Database などのリレーショナル ストアのデータを効率的に分析できます。読み取り時にスキーマを適用して非構造化データを処理し、カスタム ロジックと UDF を挿入できます。また、規模に応じて実行する方法を細かく制御できる拡張性もあります。U-SQL の背景にある設計理念の詳細については、この [Visual Studio ブログの投稿](http://blogs.msdn.com/b/visualstudio/archive/2015/09/28/introducing-u-sql.aspx)を参照してください。

ANSI SQL や T-SQL とは異なる点がいくつかあります。たとえば、SELECT などのキーワードは大文字である必要があります。

select 句、where 述語などの型システムと式言語は C# です。これは、データ型が C# 型であり、そのデータ型で C# NULL セマンティクスを使用し、述語内の比較演算子が C# 構文に従うことを意味します (たとえば、a == "foo")。また、値が完全な .NET オブジェクトであるため、任意のメソッドを使用してオブジェクトを簡単に操作できることも意味します (たとえば、"f o o o".Split(' ') )。

詳細については、[U-SQL リファレンス](http://go.microsoft.com/fwlink/p/?LinkId=691348)に関するページを参照してください。

###前提条件

先に「[チュートリアル: Data Lake Tools for Visual Studio を使用する U-SQL スクリプトの開発](data-lake-analytics-data-lake-tools-get-started.md)」を完了しておく必要があります。

このチュートリアルでは、次の U-SQL スクリプトを使用して Data Lake Analytics ジョブを実行しました。

    @searchlog =
        EXTRACT UserId          int,
                Start           DateTime,
                Region          string,
                Query           string,
                Duration        int?,
                Urls            string,
                ClickedUrls     string
        FROM "/Samples/Data/SearchLog.tsv"
        USING Extractors.Tsv();
    
    OUTPUT @searchlog   
        TO "/output/SearchLog-first-u-sql.csv"
    USING Outputters.Csv();

このスクリプトに変換手順は含まれていません。**SearchLog.tsv** というソース ファイルから読み取り、体系化して、**SearchLog-from-adltools.csv** というファイルに行セットを出力し直します。

Duration フィールドのデータ型の横にある疑問符に注目してください。これは、Duration フィールドを null にできることを意味します。

スクリプトには以下のいくつかの概念とキーワードがあります。

- **行セットの変数**: 行セットを生成する各クエリ式を変数に割り当てることができます。U-SQL は、スクリプトでは **@searchlog** などの T-SQL 変数命名パターンに従います。この割り当ては実行を強制するものではないことに注意してください。単に式に名前を付け、より複雑な式を構築できるようにするためのものです。
- **EXTRACT** では、読み取り時にスキーマを定義できます。スキーマは、列名と列ごとの C# 型名のペアで指定できます。いわゆる**エクストラクター**を使用します。たとえば、tsv ファイルを抽出する場合は、**Extractors.Tsv()** を使用します。カスタムのエクストラクターを開発することができます。
- **OUTPUT** では行セットを使用し、それをシリアル化します。Outputters.Csv() は、指定した場所にコンマ区切りファイルを出力します。また、カスタムのアウトプッターを作成することもできます。
- 次の 2 つのパスは相対パスであることに注意してください。絶対パスを使用することもできます。たとえば、次のように入力します。 
    
        adl://<ADLStorageAccountName>.azuredatalakestore.net:443/Samples/Data/SearchLog.tsv
        
    リンクされたストレージ アカウント内のファイルへのアクセスには、絶対パスを使用する必要があります。リンクされた Azure Storage アカウントに格納されているファイルの構文は以下のとおりです。
    
        wasb://<BlobContainerName>@<StorageAccountName>.blob.core.windows.net/Samples/Data/SearchLog.tsv

    >[AZURE.NOTE]パブリック BLOB またはパブリック コンテナーのアクセス許可を持つ Azure BLOB コンテナーは、現在サポートされていません。

## スカラー変数の使用

スカラー変数も使用して、スクリプトのメンテナンスを容易にすることができます。次のように、前の U-SQL スクリプトを記述することもできます。

    DECLARE @in  string = "/Samples/Data/SearchLog.tsv";
    DECLARE @out string = "/output/SearchLog-scalar-variables.csv";
    
    @searchlog =
        EXTRACT UserId          int,
                Start           DateTime,
                Region          string,
                Query           string,
                Duration        int?,
                Urls            string,
                ClickedUrls     string
        FROM @in
        USING Extractors.Tsv();
    
    OUTPUT @searchlog   
        TO @out
        USING Outputters.Csv();
      
## 行セットの変換

以下のように、**SELECT** を使用して行セットを変換します。

    @searchlog =
        EXTRACT UserId          int,
                Start           DateTime,
                Region          string,
                Query           string,
                Duration        int?,
                Urls            string,
                ClickedUrls     string
        FROM "/Samples/Data/SearchLog.tsv"
        USING Extractors.Tsv();
    
    @rs1 =
        SELECT Start, Region, Duration
        FROM @searchlog
    WHERE Region == "en-gb";
    
    OUTPUT @rs1   
        TO "/output/SearchLog-transform-rowsets.csv"
        USING Outputters.Csv();

WHERE 句では [C# ブール式](https://msdn.microsoft.com/library/6a71f45d.aspx)を使用します。C# 式言語を使用して、独自の式と関数を実行することができます。論理積 (and) および論理和 (or) と組み合わせることによって、より複雑なフィルター処理を実行することもできます。

次のスクリプトでは、DateTime.Parse() メソッドと論理積を使用します。

    @searchlog =
        EXTRACT UserId          int,
                Start           DateTime,
                Region          string,
                Query           string,
                Duration        int?,
                Urls            string,
                ClickedUrls     string
        FROM "/Samples/Data/SearchLog.tsv"
        USING Extractors.Tsv();
    
    @rs1 =
        SELECT Start, Region, Duration
        FROM @searchlog
    WHERE Region == "en-gb";
    
    @rs1 =
        SELECT Start, Region, Duration
        FROM @rs1
        WHERE Start >= DateTime.Parse("2012/02/16") AND Start <= DateTime.Parse("2012/02/17");
    
    OUTPUT @rs1   
        TO "/output/SearchLog-transform-datatime.csv"
        USING Outputters.Csv();
        
2 番目のクエリは最初の行セットの結果で動作するため、結果は 2 つのフィルターを組み合わせたものになることに注意してださい。また、変数名を再利用することもできます。その場合、名前は字句単位でスコープされます。

## 行セットの集計

U-SQL では、使い慣れた **ORDER BY**、**GROUP BY** および集計が提供されます。

次のクエリでは、リージョンごとの合計期間を検索してから、上位 5 つの期間を順に出力します。

次のクエリの場合、U-SQL 行セットの順序は保持されません。そのため、出力を順序付けるには、次のように OUTPUT ステートメントに ORDER BY を追加する必要があります。

    DECLARE @outpref string = "/output/Searchlog-aggregation";
    DECLARE @out1    string = @outpref+"_agg.csv";
    DECLARE @out2    string = @outpref+"_top5agg.csv";
    
    @searchlog =
        EXTRACT UserId          int,
                Start           DateTime,
                Region          string,
                Query           string,
                Duration        int?,
                Urls            string,
                ClickedUrls     string
        FROM "/Samples/Data/SearchLog.tsv"
        USING Extractors.Tsv();
    
    @rs1 =
        SELECT
            Region,
            SUM(Duration) AS TotalDuration
        FROM @searchlog
    GROUP BY Region;
    
    @res =
    SELECT *
    FROM @rs1
    ORDER BY TotalDuration DESC
    FETCH 5 ROWS;
    
    OUTPUT @rs1
        TO @out1
        ORDER BY TotalDuration DESC
        USING Outputters.Csv();
    OUTPUT @res
        TO @out2 
        ORDER BY TotalDuration DESC
        USING Outputters.Csv();
        
U-SQL の ORDER BY 句は、SELECT 式では FETCH 句と組み合わせる必要があります。

以下のように、U-SQL の HAVING 句を使用して、HAVING 条件を満たすグループに出力を制限することができます。

    @searchlog =
        EXTRACT UserId          int,
                Start           DateTime,
                Region          string,
                Query           string,
                Duration        int?,
                Urls            string,
                ClickedUrls     string
        FROM "/Samples/Data/SearchLog.tsv"
        USING Extractors.Tsv();
    
    @res =
        SELECT
            Region,
            SUM(Duration) AS TotalDuration
        FROM @searchlog
    GROUP BY Region
    HAVING SUM(Duration) > 200;
    
    OUTPUT @res
        TO "/output/Searchlog-having.csv"
        ORDER BY TotalDuration DESC
        USING Outputters.Csv();

## データの結合

U-SQL ではテーブルだけでなく、すべての行セット (ファイルから生成されたものでも) を結合するために、INNER JOIN、LEFT/RIGHT/FULL OUTER JOIN、SEMI JOIN などの一般的な結合演算子が提供されます。

次のスクリプトでは、検索ログと広告インプレッション ログを結合し、指定日のクエリ文字列の広告を示します。

    @adlog =
        EXTRACT UserId int,
                Ad string,
                Clicked int
        FROM "/Samples/Data/AdsLog.tsv"
        USING Extractors.Tsv();
    
    @join =
        SELECT a.Ad, s.Query, s.Start AS Date
        FROM @adlog AS a JOIN <insert your DB name>.dbo.SearchLog1 AS s 
                        ON a.UserId == s.UserId
        WHERE a.Clicked == 1;
    
    OUTPUT @join   
        TO "/output/Searchlog-join.csv"
        USING Outputters.Csv();


U-SQL でサポートされるのは、ANSI 準拠の結合構文である Rowset1 JOIN Rowset2 ON 述語のみです。FROM Rowset1, Rowset2 WHERE 述語の古い構文はサポートされていません。JOIN の述語は、式のない等価結合である必要があります。式を使用する場合は、前述の行セットの select 句に追加します。別の比較を実行する場合は、WHERE 句に移動できます。

        
## データベース、テーブル値関数、ビュー、およびテーブルの作成

U-SQL では、データベースおよびスキーマのコンテキストでデータを使用することができます。そのため、ファイルに対して常に読み取りまたは書き込みを行う必要はありません。

すべての U-SQL スクリプトが、その既定のコンテキストとして、既定のデータベース (master) と既定のスキーマ (dbo) で実行されます。独自のデータベースやスキーマを作成することができます。コンテキストを変更する場合は、**USE** ステートメントを使用してコンテキストを変更します。


### テーブル値関数 (TVF) の作成

前述の U-SQL スクリプトでは、同じソース ファイルからの読み取りで EXTRACT の使用を繰り返しました。U-SQL のテーブル値関数では、後で再利用するためにデータをカプセル化することができます。

次のスクリプトでは、既定のデータベースとスキーマで *Searchlog()* という TVF を作成します。

    DROP FUNCTION IF EXISTS Searchlog;
    
    CREATE FUNCTION Searchlog() 
    RETURNS @searchlog TABLE
    (
                UserId          int,
                Start           DateTime,
                Region          string,
                Query           string,
                Duration        int?,
                Urls            string,
                ClickedUrls     string
    )
    AS BEGIN 
    @searchlog =
        EXTRACT UserId          int,
                Start           DateTime,
                Region          string,
                Query           string,
                Duration        int?,
                Urls            string,
                ClickedUrls     string
        FROM "/Samples/Data/SearchLog.tsv"
    USING Extractors.Tsv();
    RETURN;
    END;
    
次のスクリプトは、前述のスクリプトで定義された TVF を使用する方法を示しています。

    @res =
        SELECT
            Region,
            SUM(Duration) AS TotalDuration
        FROM Searchlog() AS S
    GROUP BY Region
    HAVING SUM(Duration) > 200;
    
    OUTPUT @res
        TO "/output/SerachLog-use-tvf.csv"
        ORDER BY TotalDuration DESC
        USING Outputters.Csv();
        
### ビューの作成

抽象化するものの、パラメーター化しないクエリ式が 1 つだけある場合は、テーブル値関数ではなく、ビューを作成できます。

次のスクリプトでは、既定のデータベースとスキーマで *SearchlogView* というビューを作成します。

    DROP VIEW IF EXISTS SearchlogView;
    
    CREATE VIEW SearchlogView AS  
        EXTRACT UserId          int,
                Start           DateTime,
                Region          string,
                Query           string,
                Duration        int?,
                Urls            string,
                ClickedUrls     string
        FROM "/Samples/Data/SearchLog.tsv"
    USING Extractors.Tsv();
    
次のスクリプトは定義されたビューの使用方法を示しています。

    @res =
        SELECT
            Region,
            SUM(Duration) AS TotalDuration
        FROM SearchlogView
    GROUP BY Region
    HAVING SUM(Duration) > 200;
    
    OUTPUT @res
        TO "/output/Searchlog-use-view.csv"
        ORDER BY TotalDuration DESC
        USING Outputters.Csv();

### テーブルの作成 

リレーショナル データベースのテーブルと同様に、U-SQL では、定義済みのスキーマでテーブルを作成したり、テーブルを作成して、そのテーブルを設定するクエリからスキーマを推測したりすることができます (CREATE TABLE AS SELECT または CTAS ともいう)。

次のスクリプトでは、1 つのデータベースと 2 つのテーブルを作成します。

    DROP DATABASE IF EXISTS SearchLogDb;
    CREATE DATABASE SeachLogDb
    USE DATABASE SearchLogDb;
    
    DROP TABLE IF EXISTS SearchLog1;
    DROP TABLE IF EXISTS SearchLog2;
    
    CREATE TABLE SearchLog1 (
                UserId          int,
                Start           DateTime,
                Region          string,
                Query           string,
                Duration        int?,
                Urls            string,
                ClickedUrls     string,
    
                INDEX sl_idx CLUSTERED (UserId ASC) 
                    PARTITIONED BY HASH (UserId)
    );
    
    INSERT INTO SearchLog1 SELECT * FROM master.dbo.Searchlog() AS s;
    
    CREATE TABLE SearchLog2(
        INDEX sl_idx CLUSTERED (UserId ASC) 
                PARTITIONED BY HASH (UserId)
    ) AS SELECT * FROM master.dbo.Searchlog() AS S; // You can use EXTRACT or SELECT here


### テーブルの照会

データ ファイルの照会と同じ方法で、テーブル (前述のスクリプトで作成されたもの) を照会できます。EXTRACT を使用して行セットを作成する代わりに、テーブル名のみを参照できるようになりました。

テーブルから読み取る場合、前に使用した変換スクリプトは以下のように変更されます。

    @rs1 =
        SELECT
            Region,
            SUM(Duration) AS TotalDuration
        FROM SearchLogDb.dbo.SearchLog2
    GROUP BY Region;
    
    @res =
        SELECT *
        FROM @rs1
        ORDER BY TotalDuration DESC
        FETCH 5 ROWS;
    
    OUTPUT @res
        TO "/output/Searchlog-query-table.csv"
        ORDER BY TotalDuration DESC
        USING Outputters.Csv();

現時点では、テーブルを作成する場合と同じスクリプトで、テーブルに対して SELECT を実行できないことに注意してください。


##まとめ

このチュートリアルの内容は U-SQL のほんの一部です。このチュートリアルには適用範囲があるため、以下を含むすべてを説明することはできません。

- CROSS APPLY を使用して、文字列、配列およびマップの部分を行にアンパックする。
- データのパーティション セット (ファイル セットおよびパーティション テーブル) を操作する。
- エクストラクター、アウトプッター、プロセッサー、ユーザー定義のアグリゲーターなどのユーザー定義演算子を C# で開発する。
- U-SQL ウィンドウ化関数を使用する。
- U-SQL コードをビュー、テーブル値関数およびストアド プロシージャで管理する。
- 処理ノードで任意のカスタム コードを実行する。 
- Azure SQL Database に接続し、それらと U-SQL および Azure Data Lake データ間のクエリを統合する。

## 関連項目 

- [Microsoft Azure Data Lake Analytics の概要](data-lake-analytics-overview.md)
- [Data Lake Tools for Visual Studio を使用する U-SQL スクリプトの開発](data-lake-analytics-data-lake-tools-get-started.md)
- [Azure Data Lake Analytics ジョブに U-SQL ウィンドウ関数を使用する](data-lake-analytics-use-window-functions.md)
- [Azure ポータルを使用する Azure Data Lake Analytics ジョブの監視とトラブルシューティング](data-lake-analytics-monitor-and-troubleshoot-jobs-tutorial.md)

## 意見の投稿

- [新しいドキュメント バックログの提案](data-lake-analytics-documentation-backlog.md)
- [機能要求の送信](http://aka.ms/adlafeedback)
- [フォーラムでサポートを受ける](http://aka.ms/adlaforums)
- [U-SQL に関するフィードバックの提供](http://aka.ms/usqldiscuss)

<!---HONumber=AcomDC_0107_2016-->
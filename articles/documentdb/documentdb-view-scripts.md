<properties
	pageTitle="DocumentDB スクリプト エクスプローラーという JavaScript エディター | Microsoft Azure"
	description="DocumentDB のサーバー側プログラミング アーティファクト (ストアド プロシージャ、トリガー、ユーザー定義関数など) を管理するための Azure ポータル ツール、DocumentDB スクリプト エクスプローラーについて説明します。"
	keywords="javascript エディター"
	services="documentdb"
	authors="AndrewHoh"
	manager="jhubbard"
	editor="monicar"
	documentationCenter=""/>

<tags
	ms.service="documentdb"
	ms.workload="data-services"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="02/23/2016"
	ms.author="anhoh"/>

# DocumentDB スクリプト エクスプローラーを使用したストアド プロシージャ、トリガー、ユーザー定義関数の作成と実行

この記事では、DocumentDB のサーバー側プログラミング アーティファクト (ストアド プロシージャ、トリガー、ユーザー定義関数など) の表示と実行を可能にする Azure ポータルの JavaScript エディター、[Microsoft Azure DocumentDB](https://azure.microsoft.com/services/documentdb/) スクリプト エクスプローラーの概要を説明します。DocumentDB のサーバー側プログラミングの詳細については、「[ストアド プロシージャ、データベース トリガー、UDF](documentdb-programming.md)」という記事を参照してください。

## スクリプト エクスプローラーを起動する

1. Azure ポータルで、ジャンプバーの **[DocumentDB アカウント]** をクリックします。**[DocumentDB アカウント]** が表示されない場合は、**[参照]** をクリックし、**[DocumentDB アカウント]** をクリックします。

2. **[DocumentDB アカウント]** ブレードの上部にある **[スクリプト エクスプローラー]** をクリックします。

	![スクリプト エクスプローラー コマンドのスクリーンショット](./media/documentdb-view-scripts/scriptexplorercommand.png)
 
    >[AZURE.NOTE] スクリプト エクスプローラーは、データベース ブレードとコレクション ブレードにも表示されます。

    **[データベース]** と **[コレクション]** の各ドロップダウン リスト ボックスには、スクリプト エクスプローラーを起動したコンテキストに応じて値が設定されています。たとえば、データベース ブレードから起動した場合には、現在のデータベースが設定されます。コレクション ブレードから起動した場合には、現在のコレクションが設定されます。

	![スクリプト エクスプローラーのスクリーンショット](./media/documentdb-view-scripts/scriptexplorerinitial.png)

4.  **[データベース]** ボックスと **[コレクション]** ボックスを使用すると、スクリプト エクスプローラーを閉じて再度起動することなく、現在表示されているドキュメントが含まれるコレクションを簡単に変更できます。

5. スクリプト エクスプローラーでは、現在読み込まれているドキュメントのセットを ID プロパティでフィルター処理することもできます。フィルター ボックスに入力します。スクリプト エクスプローラーの一覧の結果が指定された条件に基づいてフィルター処理されます。

	![フィルターの結果が表示されたスクリプト エクスプローラーのスクリーンショット](./media/documentdb-view-scripts/scriptexplorerfilterresults.png)


	> [AZURE.IMPORTANT] スクリプト エクスプローラーのフィルター機能では、現在読み込まれているドキュメントのみがフィルター処理されます。現在選択されているコレクションに対してスクリプトが自動的に更新されることはありません。

5. スクリプト エクスプローラーに読み込まれたスクリプトトの一覧を更新するには、ブレードの上部にある **[更新]** をクリックするだけです。

	![スクリプト エクスプローラーの [最新の情報に更新] コマンドのスクリーンショット](./media/documentdb-view-scripts/scriptexplorerrefresh.png)


## ストアド プロシージャ、トリガー、ユーザー定義関数の作成、表示、編集

スクリプト エクスプローラーを使用すると、DocumentDB サーバー側プログラミング アーティファクトに対する CRUD 操作を簡単に実行できます。

- スクリプトを作成するには、スクリプト エクスプローラーで適切な作成コマンドをクリックし、ID を指定し、スクリプトの内容を入力し、**[保存]** をクリックします。

	![スクリプト エクスプローラーの作成オプションと JavaScript エディターのスクリーンショット](./media/documentdb-view-scripts/scriptexplorercreatecommand.png)

- トリガーを作成する場合は、トリガーの種類とトリガー操作を指定する必要があります。

	![スクリプト エクスプローラーのトリガー作成オプションのスクリーンショット](./media/documentdb-view-scripts/scriptexplorercreatetrigger.png)

- スクリプトを表示するには、関心のあるスクリプトをクリックします。

	![スクリプト エクスプローラーのスクリプト表示エクスペリエンスのスクリーンショット](./media/documentdb-view-scripts/scriptexplorerviewscript.png)

- スクリプトを編集するには、JavaScript エディターで目的の変更を行い、**[保存]** をクリックします。

	![スクリプト エクスプローラーのスクリプト表示エクスペリエンスのスクリーンショット](./media/documentdb-view-scripts/scriptexplorereditscript.png)

- スクリプトの保留中の変更を破棄するには、**[破棄]** コマンドをクリックします。

	![スクリプト エクスプローラーの変更破棄エクスペリエンスのスクリーンショット](./media/documentdb-view-scripts/scriptexplorerdiscardchanges.png)

- スクリプト エクスプローラーで **[プロパティ]** をクリックすると、現在読み込まれているスクリプトのシステム プロパティを簡単に表示することもできます。

	![スクリプト エクスプローラーのスクリプトのプロパティ表示のスクリーンショット](./media/documentdb-view-scripts/scriptproperties.png)

	> [AZURE.NOTE] タイムスタンプ (\_ts) プロパティは内部ではエポック時間として表現されますが、スクリプト エクスプローラーでは、人間が読むことができる GMT 形式で値が表示されます。

- スクリプトを削除するには、スクリプト エクスプ ローラーで削除するスクリプトを選択し、**[削除]** コマンドをクリックします。

	![スクリプト エクスプローラーの削除コマンドのスクリーンショット](./media/documentdb-view-scripts/scriptexplorerdeletescript1.png)

- 削除アクションを **[はい]** をクリックして確認するか、**[いいえ]** をクリックして取り消します。

	![スクリプト エクスプローラーの削除コマンドのスクリーンショット](./media/documentdb-view-scripts/scriptexplorerdeletescript2.png)

## ストアド プロシージャの実行

スクリプト エクスプローラーでは、Azure ポータルからサーバー側のストアド プロシージャを実行できます。

- 新規作成のストアド プロシージャ ブレードを開くと、既定のスクリプト (*プレフィックス*) が提供されます。*プレフィックス* スクリプトまたは独自のスクリプトを実行するには、*ID* と*入力*を追加します。複数のパラメーターを受け取るストアド プロシージャの場合、すべて配列内に入力します (例 *["foo", "bar"]*)。

	![スクリプト エクスプローラーの [ストアド プロシージャ] ブレードで入力を追加し、ストアド プロシージャを実行するスクリーンショット](./media/documentdb-view-scripts/documentdb-execute-a-stored-procedure-input.png)

- ストアド プロシージャを実行するには、スクリプト エディター ウィンドウ内で **[保存して実行]** コマンドをクリックします。

	> [AZURE.NOTE] **[保存して実行]** コマンドを実行すると、実行前にストアド プロシージャが保存されます。つまり、ストアド プロシージャの前回保存されたバージョンが上書きされます。

- ストアド プロシージャが正常に実行されると、そのステータスが「*ストアド プロシージャが正常に保存され、実行されました*」となり、返された結果が *[結果]* ウィンドウに入力されます。

	![スクリプト エクスプローラーの [ストアド プロシージャ] ブレードでストアド プロシージャを実行するスクリーンショット](./media/documentdb-view-scripts/documentdb-execute-a-stored-procedure.png)

- 実行中にエラーが発生した場合、エラーは *[結果]* ウィンドウに入力されます。

	![スクリプト エクスプローラーのスクリプト プロパティのスクリーンショットストアド プロシージャの実行とエラー](./media/documentdb-view-scripts/documentdb-execute-a-stored-procedure-error.png)

## ポータルの外部のスクリプトを使用する

Azure ポータルのスクリプト エクスプローラーは、DocumentDB のストアド プロシージャ、トリガー、ユーザー定義関数を使用する方法の 1 つです。REST API と[クライアント SDK](documentdb-sdk-dotnet.md) を使用してスクリプトを操作することもできます。REST API ドキュメントには、[REST を利用したストアド プロシージャ](https://msdn.microsoft.com/library/azure/mt489092.aspx)、[REST を利用したユーザー定義関数](https://msdn.microsoft.com/library/azure/dn781481.aspx)、[REST を利用したトリガー](https://msdn.microsoft.com/library/azure/mt489116.aspx)のサンプルが含まれています。[C# を利用したスクリプト](documentdb-dotnet-samples.md#server-side-programming-examples)と [Node.js を利用したスクリプト](documentdb-nodejs-samples.md#server-side-programming-examples)の使用方法を示すサンプルもあります。

## 次のステップ

DocumentDB のサーバー側プログラミングの詳細については、「[ストアド プロシージャ、データベース トリガー、UDF](documentdb-programming.md)」という記事を参照してください。

[ラーニング パス](https://azure.microsoft.com/documentation/learning-paths/documentdb/)も、DocumentDB の詳細について学習する際に便利なリソースです。

<!---HONumber=AcomDC_0224_2016-->
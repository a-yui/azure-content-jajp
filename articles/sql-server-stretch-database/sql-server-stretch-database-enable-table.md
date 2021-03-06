<properties
	pageTitle="デーブルの Stretch Database を有効にする | Microsoft Azure"
	description="Stretch Database のテーブルを設定する方法について説明します。"
	services="sql-server-stretch-database"
	documentationCenter=""
	authors="douglasl"
	manager="jhubbard"
	editor="monicar"/>

<tags
	ms.service="sql-server-stretch-database"
	ms.workload="data-management"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="02/26/2016"
	ms.author="douglasl"/>

# テーブルの Stretch Database を有効にする

Stretch Database のテーブルを設定するには、SQL Server Management Studio でテーブルの **[Stretch]、[有効化]** の順に選択し、**[Stretch テーブルを有効にする]** ウィザードを起動します。Transact-SQL を利用し、テーブルで Stretch Database を有効にすることもできます。

-   別個のテーブルに履歴データを保存する場合、テーブル全体を移行できます。

-   テーブルに過去と現在の両方のデータが含まれている場合、移行する行を選択するフィルター述語を指定できます。CTP 3.1 ～ RC0 では、[Stretch Database を有効にする] ウィザードにはフィルター述語を指定するオプションがありません。このオプションで Stretch Database のテーブルを設定するには、ALTER TABLE ステートメントを使用する必要があります。詳細については、「[ALTER TABLE (Transact-SQL)](https://msdn.microsoft.com/library/ms190273.aspx)」をご覧ください。

**前提条件**。テーブルに **[Stretch]、[有効化]** を選択したとき、データベースの Stretch Database を有効にしていない場合、ウィザードにより Stretch Database のデータベースが最初に設定されます。このトピックの手順ではなく、「[[Stretch Database を有効にする] ウィザード](sql-server-stretch-database-wizard.md)」の手順に従ってください。

**アクセス許可**。データベースまたはテーブルで Stretch Database を有効にするには、db\_owner アクセス許可が必要です。テーブルで Stretch Database を有効にするには、テーブルの ALTER アクセス許可も必要です。

## <a name="EnableWizardTable"></a>ウィザードを使用し、テーブルで Stretch Database を有効にする
**ウィザードを起動する**
1.  SQL Server Management Studio のオブジェクト エクスプローラーで、Stretch を有効にするテーブルを選択します。

2.  右クリックして **[Stretch]** を選択し、**[有効化]** を選択してウィザードを起動します。

**はじめに** ウィザードの目的と前提条件を確認します。

**データベース テーブルを選択する** 有効にするテーブルが表示され、選択されていることを確認します。

CTP 3.1 ～ RC0 では、ウィザードで利用し、テーブル全体を移行することだけが可能です。行を選択する述語を指定し、履歴データと現行データの両方を含むテーブルから移行する場合、ALTER TABLE ステートメントを実行し、ウィザードの終了後に述語を指定するか、ウィザードを終了し、ALTER TABLE ステートメントを実行します。詳細はこのトピックの後半にあります。

**概要** 入力した値とウィザードで選択したオプションを確認します。**[完了]** を選択し、Stretch を有効にします。

**結果** 結果を確認します。

## <a name="EnableTSQLTable"></a>Transact-SQL を使用し、テーブルで Stretch Database を有効にする
Stretch Database のテーブルを設定するには、ALTER TABLE コマンドを実行します。

1.  テーブルに履歴データと現行データの両方が含まれている場合、`FILTER_PREDICATE = <predicate>` 句を使用し、移行する行を選択する述語を指定することもできます。述語では、インライン テーブル値関数を呼び出す必要があります。詳細については、「[行を選択するインライン テーブル値関数を記述する (Stretch Database)](sql-server-stretch-database-predicate-function.md)」を参照してください。フィルター述語を指定しない場合、テーブル全体が移行されます。

    > [!重要] 指定したフィルター述語のパフォーマンスが悪いと、データ移行のパフォーマンスも悪くなります。Stretch Database は CROSS APPLY 演算子を利用し、テーブルにフィルター述語を適用します。

    CTP 3.1 ～ RC0 では、[Stretch Database を有効にする] ウィザードにはこのオプションがありません。このオプションで Stretch Database のテーブルを設定するには、ALTER TABLE ステートメントを使用する必要があります。詳細については、「ALTER TABLE (Transact-SQL)」をご覧ください (https://msdn.microsoft.com/library/ms190273.aspx)。

2.  データの移行をすぐに開始する場合は `MIGRATION_STATE = OUTBOUND` を指定し、データの移行の開始を先送りする場合は `MIGRATION_STATE = PAUSED` を指定します。

テーブル全体を移行し、データ移行をすぐに開始する例を次に示します。

```tsql
ALTER TABLE <table name>
    SET ( REMOTE_DATA_ARCHIVE = ON ( MIGRATION_STATE = OUTBOUND ) ) ;
GO;
```
`dbo.fn_stretchpredicate` インライン テーブル値関数で特定される行のみを移行し、データ移行を先送りする例は次のようになります。フィルター述語の詳細については、「[行を選択するインライン テーブル値関数を記述する (Stretch Database)](sql-server-stretch-database-predicate-function.md)」を参照してください。

```tsql
ALTER TABLE <table name>
    SET ( REMOTE_DATA_ARCHIVE = ON (
        FILTER_PREDICATE = dbo.fn_stretchpredicate(date),
        MIGRATION_STATE = PAUSED ) )
```

## 関連項目
[ALTER TABLE (Transact-SQL)](https://msdn.microsoft.com/library/ms190273.aspx)

<!---HONumber=AcomDC_0302_2016-->
<properties
   pageTitle="Logic Apps での HDInsight コネクタの使用 | Microsoft Azure App Service"
   description="HDInsight コネクタまたは API アプリを作成、構成して、Azure App Service のロジック アプリで使用する方法"
   services="app-service\logic"
   documentationCenter=".net,nodejs,java"
   authors="anuragdalmia"
   manager="erikre"
   editor=""/>

<tags
   ms.service="app-service-logic"
   ms.devlang="multiple"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="integration"
   ms.date="02/10/2016"
   ms.author="sameerch"/>


# HDInsight コネクタの使用開始とロジック アプリへの追加
>[AZURE.NOTE] 本記事は、ロジック アプリの 2014-12-01-preview スキーマ バージョンを対象としています。

HDInsight コネクタでは、Azure 上で Hadoop クラスターを作成し、Hive ジョブ、Pig ジョブ、MapReduce ジョブ、ストリーミング MapReduce ジョブなど、さまざまな Hadoop ジョブを送信できます。Azure HDInsight サービスは、Apache Hadoop クラスターをクラウドにデプロイしてプロビジョニングするためのサービスです。ビッグ データの管理、分析、レポート生成を支援するソフトウェア フレームワークとなります。Hadoop コアには、データ ストレージの信頼性を高める Hadoop 分散ファイル システム (HDFS) と、この分散システムに格納されているデータの処理と分析を同時に行うシンプルな MapReduce プログラミング モデルが用意されています。HDInsight コネクタを使用して、クラスターの作成または削除、ジョブの送信、ジョブの完了待機を実行できます。

コネクタをロジック アプリで使用すると、フローの一部として、データをフェッチ、処理、またはプッシュ転送できます。この HDInsight コネクタをビジネス ワークフローに追加し、ロジック アプリ内のそのワークフローの一部としてデータを処理できます。

### 基本的なアクション

- クラスターを作成
- クラスターの作成を待機
- Pig ジョブを送信
- Hive ジョブを送信
- MapReduce ジョブを送信
- ジョブの完了を待機
- クラスターを削除


## HDInsight コネクタ API App の作成 ##

コネクタは、ロジック アプリ内で作成することも、Azure Marketplace から直接作成することもできます。Marketplace からコネクタを作成するには、次の操作を実行します。

1. Azure のスタート画面で、**[Marketplace]** を選択します。
2. "HDInsight コネクタ" を検索して選択し、**[作成]** を選択します。
3. 名前、App Service プラン、その他のプロパティを入力します。
4. パッケージ設定で HDInsight クラスターのユーザー名とパスワードを入力します。**[OK]** を選択します。
5. **[作成]** を選択します。![][1]  

## 証明書の構成 (省略可能) ##

> [AZURE.NOTE] この手順は、ロジック アプリで管理操作 (クラスターの作成と削除) を実行する場合のみ必要です。

作成した HDInsight コネクタ API App を参照すると、[セキュリティ] コンポーネントが 0 になっています。これは、アップロードされた管理証明書がないことを意味します: ![][2]

API アプリに管理証明書をアップロードするには、次の手順を実行します。

1. [セキュリティ] コンポーネントを選択します。
2. [セキュリティ] ブレードで **[証明書のアップロード]** を選択します。
3. 次のブレードで証明書ファイルを参照して選択します。
4. 証明書を選択したら、**[OK]** を選択します。

証明書が正常にアップロードされると、証明書の詳細が表示されます。![][3]

> [AZURE.NOTE] 証明書を変更する場合は、既存の証明書を置き換える別の証明書をアップロードできます。

## ロジック アプリでのコネクタの使用 ##

HDInsight コネクタは、ロジック アプリでアクションとしてのみ使用できます。クラスターを作成して Hive ジョブを実行し、ジョブの完了時にクラスターを削除する、簡単なロジック アプリを見てみましょう。


1. [ロジックの開始] カードで、[このロジックを手動で実行] をクリックします。
2. ギャラリーで、以前に作成した HDInsight コネクタ API App を選択します (作成した HDInsight コネクタは画面の右の API Apps リストに表示されます)。黒の右矢印を選択します。使用可能なアクションが表示されます。![][12]

3. [クラスターの作成] を選択し、必要なクラスター パラメーターをすべて入力したら、[✓] を選択します。![][6]

4. これで、アクションは、ロジック アプリで構成されたものとして表示されます。アクションの出力が表示されます。これは、後続のアクションの入力として使用できます。![][7]

5. ギャラリーからアクションと同じ HDInsight コネクタを選択します。[クラスターの作成を待機] アクションを選択し、必要なパラメーターをすべて入力したら、[✓] を選択します。![][8]

6. ギャラリーからアクションと同じ HDInsight コネクタを選択します。[Hive ジョブの送信] アクションを選択し、必要なパラメーターをすべて入力したら、[✓] を選択します。![][9]

7. ギャラリーからアクションと同じ HDInsight コネクタを選択します。[ジョブの完了を待機] アクションを選択し、必要なパラメーターをすべて入力したら、[✓] を選択します。![][10]

8. ギャラリーからアクションと同じ HDInsight コネクタを選択します。[クラスターを削除] アクションを選択し、必要なパラメーターをすべて入力したら、[✓] を選択します。![][11]

9. デザイナー上部の [保存] コマンドを使用してロジック アプリを保存します。

シナリオをテストするには、**[今すぐ実行]** を選択してロジック アプリを手動で開始します。

## コネクタでできること
コネクタが作成されたため、ロジック アプリを使用してコネクタをビジネス ワークフローに追加できます。「[Logic Apps とは](app-service-logic-what-are-logic-apps.md)」を参照してください。

>[AZURE.NOTE] Azure アカウントにサインアップする前に Azure Logic Apps の使用を開始する場合は、「[Azure App Service アプリケーションの作成](https://tryappservice.azure.com/?appservice=logic)」を参照してください。App Service で有効期間の短いスターター ロジック アプリをすぐに作成できます。このサービスの利用にあたり、クレジット カードは必要ありません。契約も必要ありません。

「[Connectors and API Apps Reference (コネクタと API Apps のリファレンス)](http://go.microsoft.com/fwlink/p/?LinkId=529766)」で Swagger REST API のリファレンスを参照してください。

パフォーマンス統計をレビューし、コネクタに対するセキュリティを制御することもできます。[組み込みの API Apps とコネクタの管理と監視](app-service-logic-monitor-your-connectors.md)に関するページを参照してください。


<!--Image references-->
[1]: ./media/app-service-logic-connector-hdinsight/Create.png
[2]: ./media/app-service-logic-connector-hdinsight/CertNotConfigured.png
[3]: ./media/app-service-logic-connector-hdinsight/CertConfigured.png
[5]: ./media/app-service-logic-connector-hdinsight/LogicApp1.png
[6]: ./media/app-service-logic-connector-hdinsight/LogicApp2.png
[7]: ./media/app-service-logic-connector-hdinsight/LogicApp3.png
[8]: ./media/app-service-logic-connector-hdinsight/LogicApp4.png
[9]: ./media/app-service-logic-connector-hdinsight/LogicApp5.png
[10]: ./media/app-service-logic-connector-hdinsight/LogicApp6.png
[11]: ./media/app-service-logic-connector-hdinsight/LogicApp7.png
[12]: ./media/app-service-logic-connector-hdinsight/LogicApp8.PNG

<!---HONumber=AcomDC_0224_2016-->
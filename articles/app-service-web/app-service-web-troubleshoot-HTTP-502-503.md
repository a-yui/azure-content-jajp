<properties
	pageTitle="トラブルシューティング: HTTP 502/503 が原因で Web アプリが利用できない"
	description="この記事では、Azure App Service でホストされている Web アプリで発生した HTTP 502/503 エラーのトラブルシューティングについて取り上げます。"
	services="app-service\web"
	documentationCenter=""
	authors="cephalin"
	manager="wpickett"
	editor=""
	tags="top-support-issue"/>

<tags
	ms.service="app-service-web"
	ms.workload="web"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="01/13/2016"
	ms.author="cephalin"/>

# トラブルシューティング: HTTP 502/503 が原因で Web アプリが利用できない

この記事では、[Azure App Service](http://go.microsoft.com/fwlink/?LinkId=529714) でホストされている Web アプリで発生した HTTP 502/503 エラーのトラブルシューティングについて取り上げます。

この記事についてさらにヘルプが必要な場合は、いつでも [MSDN の Azure フォーラムとスタック オーバーフロー フォーラム](https://azure.microsoft.com/support/forums/)で Azure エキスパートに問い合わせることができます。または、Azure サポート インシデントを送信できます。その場合は、[Azure サポートのサイト](https://azure.microsoft.com/support/options/)に移動して、**[サポートの要求]** をクリックします。

## 症状

ブラウザーで Web アプリにアクセスすると、HTTP "502 Bad Gateway" または HTTP "503 Service Unavailable" が返されます。

## 原因

この症状は多くの場合、アプリケーション レベルの問題が原因で発生します。その例を次に示します。

-	要求に時間がかかっている
-	アプリケーションのメモリ/CPU 使用率が高い
-	例外が発生してアプリケーションがクラッシュする

## トラブルシューティングの手順

トラブルシューティングは、大きく次の 3 つのタスクに分けられます。この 3 つのタスクを上から順に行います。

1.	[アプリケーションの動作を観察、監視する](#observe)
2.	[データを収集する](#collect)
3.	[問題を緩和する](#mitigate)

これらの手順には、[App Service Web Apps](/services/app-service/web/) を活用できます。

<a name="observe" />
### 1\.アプリケーションの動作を観察、監視する

####	サービス正常性を追跡する

Microsoft Azure は、サービスの中断やパフォーマンスの低下があるたびに、毎回公表します。サービスの正常性は、[Azure ポータル](https://portal.azure.com/)で追跡できます。詳細については、[サービスの正常性の追跡](../azure-portal/insights-service-health.md)に関するページを参照してください。

####	Web アプリを監視する

Web アプリに問題が発生しているかどうかは、アプリを監視することによって確認することができます。Web アプリのブレードで **[要求とエラー]** タイルをクリックします。**[メトリック]** ブレードには、追加できるすべてのメトリックが表示されます。

Web アプリに関しては、次のメトリックを監視するようお勧めします。

-	平均メモリ ワーキング セット
-	平均応答時間
-	CPU 時間
-	メモリ ワーキング セット
-	要求数

![](./media/app-service-web-troubleshoot-HTTP-502-503/1-monitor-metrics.png)

詳細については、次を参照してください。

-	[Azure App Service の Web Apps の監視](web-sites-monitor.md)
-	[アラート通知の受信](../azure-portal/insights-receive-alert-notifications.md)

<a name="collect" />
### 2\.データを収集する

####	Azure App Service サポート ポータルを使用する

Web Apps には、HTTP ログ、イベント ログ、処理ダンプなどを参照することによって、Web アプリに関連した問題をトラブルシューティングする機能があります。その情報はすべて、**http://&lt;your app name>.scm.azurewebsites.net/Support** のサポート ポータルで提供されます。

一般的なトラブルシューティングの状況に合わせて、Azure App Service サポート ポータルには、3 つのタブが用意されています。次の 3 つの手順が想定されています。

1.	現在の動作を観察する
2.	診断情報を集め、組み込みのアナライザーを実行して分析する
3.	緩和する

現時点で問題が発生している場合は、**[分析]**、**[診断]**、**[今すぐ診断]** の順にクリックして診断セッションを作成します。これで、HTTP ログ、イベント ビューアー ログ、メモリ ダンプ、PHP エラー ログ、および PHP プロセス レポートが収集されます。

データの収集後にも、データに対する分析が実行され、HTML レポートが出力されます。

そのデータをダウンロードすることもできます。D:\\home\\data\\DaaS フォルダーが既定の保存先となります。

Azure App Service サポート ポータルの詳細については、[Azure Websites のサポート サイト拡張機能で新たに行われた更新](/blog/new-updates-to-support-site-extension-for-azure-websites)に関するページを参照してください。

####	Kudu デバッグ コンソールを使用する

Web Apps には、ファイルのデバッグ、調査、アップロード用のデバッグ コンソールのほか、ご利用の環境についての情報を入手するための JSON エンドポイントが用意されています。このコンソールは、Web アプリの _Kudu コンソール_または _SCM ダッシュボード_と呼ばれます。

ダッシュボードには、**https://&lt;Your app name>.scm.azurewebsites.net/** リンクからアクセスできます。

Kudu には次のような機能があります。

-	アプリケーションの環境設定
-	ログ ストリーム
-	診断ダンプ
-	デバッグ コンソール (Powershell のコマンドレットや基本的な DOS コマンドを実行可能)


Kudu にはもう 1 つ便利な機能があり、アプリケーションからファーストチャンス例外がスローされた場合に、Kudu と SysInternals ツール Procdump を使用してメモリ ダンプを作成することができます。このメモリ ダンプはプロセスのスナップショットです。Web アプリに関して、通常より複雑な問題をトラブルシューティングできる場合も少なくありません。

Kudu で利用できる機能の詳細については、[知っておくべき Azure Websites のオンライン ツール](/blog/windows-azure-websites-online-tools-you-should-know-about/)に関するページを参照してください。

<a name="mitigate" />
### 3\.問題を緩和する

####	Web アプリをスケーリングする

Azure App Service では、アプリケーションが実行されるスケールを調整することによって、パフォーマンスとスループットを高めることができます。Web アプリのスケール アップには、2 つの関連する措置が伴います。1 つは、App Service プランの価格レベルを引き上げること、もう 1 つは、価格レベルを引き上げた後に特定の設定を構成することです。

スケーリングの詳細については、「[Azure App Service の Web アプリをスケーリングする](web-sites-scale.md)」を参照してください。

加えて、アプリケーションを複数のインスタンスで実行することもできます。処理能力がアップするだけでなく、ある程度の耐障害性を確保することができます。この場合、1 つのインスタンスでプロセスがダウンしても、他のインスタンスが要求の処理を続行します。

スケーリングは、[手動] または [自動] に設定することができます。

####	AutoHeal を使用する

AutoHeal は、選択された設定 (構成の変更、要求、メモリに基づく制限、要求の実行に必要な時間など) に従って、アプリのワーカー プロセスをリサイクルします。ほとんどの場合、問題を回復するための一番の近道は、プロセスをリサイクルすることです。Web アプリはいつでも、Azure ポータル内から直接、再起動できますが、AutoHeal はユーザーの介入なしでそれを自動的に実行します。必要な作業は、Web アプリのルート web.config にいくつかのトリガーを追加することだけです。.Net アプリケーション以外でも、これらの設定は同じように作用します。

詳細については、[Azure Web Sites の自動復旧](/blog/auto-healing-windows-azure-web-sites/)に関するページを参照してください。


####	Web アプリを再起動する

1 回限りの問題であれば、通常これが最も簡単な復旧方法です。アプリを停止または再起動するためのオプションは、[Azure ポータル](https://portal.azure.com/)の Web アプリ ブレードにあります。

 ![](./media/app-service-web-troubleshoot-HTTP-502-503/2-restart.png)

Web アプリの管理には、Azure PowerShell を使用することもできます。詳細については、[リソース マネージャーでの Azure PowerShell の使用](../powershell-azure-resource-manager.md)をご覧ください。

<!---HONumber=AcomDC_0211_2016-->
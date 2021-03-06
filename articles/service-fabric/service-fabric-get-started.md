<properties
   pageTitle="開発環境のセットアップ | Microsoft Azure"
   description="ランタイム、SDK、およびツールをインストールし、ローカル開発クラスターを作成します。このセットアップを終えれば、アプリケーションを構築する準備は完了です。"
   services="service-fabric"
   documentationCenter=".net"
   authors="seanmck"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.devlang="dotNet"
   ms.topic="get-started-article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="02/09/2016"
   ms.author="seanmck"/>

# 開発環境を準備する
 開発コンピューターで [Azure Service Fabric アプリケーション][1]をビルドして実行するには、ランタイム、SDK、ツールをインストールする必要があります。また、SDK に含まれる Windows PowerShell スクリプトの実行を有効にする必要があります。

## 前提条件
### サポートされるオペレーティング システムのバージョン
次のオペレーティング システムのバージョンがサポートされます。

- Windows 8/Windows 8.1
- Windows Server 2012 R2
- Windows 10

### Visual Studio 2015

Service Fabric のツールは Visual Studio 2015 に依存しています。Visual Studio 2015 は、[Visual Studio の Web サイト][2]から入手できます。

> [AZURE.NOTE] サポートされる OS のバージョンを実行していないか、コンピューターに Visual Studio 2015 をインストールしたくない場合は、Windows Server 2012 R2 と Visual Studio 2015 がプレインストールされている Azure 仮想マシンを設定できます。この操作は、Azure 仮想マシン ギャラリーからのイメージを使用して実行できます。

## ランタイム、SDK、およびツールのインストール

Service Fabric のコンポーネントは、Web Platform Installer によってインストールされます。次の手順に従ってインストールします。

1. Web Platform Installer を使用して [SDK をダウンロード][3]します。

2. **[インストール]** をクリックして、インストール プロセスを開始します。

3. 使用許諾契約書を確認し、同意します。

自動的にインストールが続行します。

## PowerShell スクリプトの実行の有効化

Service Fabric は、ローカル開発クラスターの作成、および Visual Studio からのアプリケーションのデプロイに、Windows PowerShell スクリプトを使用します。既定では、Windows はこれらのスクリプトの実行をブロックします。これらを有効にするには、PowerShell 実行ポリシーを変更する必要があります。管理者として PowerShell を開き、次のコマンドを入力します。

```powershell
Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Force -Scope CurrentUser
```

## 次のステップ
開発環境の設定が完了しました。これで、アプリのビルドと実行を開始できます。

- [Visual Studio で最初の Service Fabric アプリケーションを作成する](service-fabric-create-your-first-application-in-visual-studio.md)
- [ローカル クラスター上でアプリケーションをデプロイし管理する方法](service-fabric-get-started-with-a-local-cluster.md)
- [サービスのフレームワークを選択する](service-fabric-choose-framework.md)
- [GitHub での Service Fabric コード サンプルの確認](https://aka.ms/servicefabricsamples)
- [Service Fabric エクスプローラーを使用したクラスターの視覚化](service-fabric-visualizing-your-cluster.md)
- [Service Fabric のラーニング パスに沿ってプラットフォームの広範な概要を理解する](https://azure.microsoft.com/documentation/learning-paths/service-fabric/)

[1]: http://azure.microsoft.com/campaigns/service-fabric/ "Service Fabric キャンペーン ページ"
[2]: http://go.microsoft.com/fwlink/?LinkId=517106 "VS RC"
[3]: http://www.microsoft.com/web/handlers/webpi.ashx?command=getinstallerredirect&appid=MicrosoftAzure-ServiceFabric "WebPI のリンク"

<!---HONumber=AcomDC_0218_2016-->
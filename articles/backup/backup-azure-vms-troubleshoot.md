<properties
	pageTitle="Azure 仮想マシンのバックアップのトラブルシューティング | Microsoft Azure"
	description="Azure 仮想マシンのバックアップと復元のトラブルシューティング"
	services="backup"
	documentationCenter=""
	authors="trinadhk"
	manager="shreeshd"
	editor=""/>

<tags
	ms.service="backup"
	ms.workload="storage-backup-recovery"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="01/28/2016"
	ms.author="trinadhk;jimpark;"/>


# Azure 仮想マシンのバックアップのトラブルシューティング
次の表に示す情報を使って、Azure Backup の使用中に発生したエラーのトラブルシューティングを行うことができます。

## 探索

| バックアップ操作 | エラーの詳細 | 対処法 |
| -------- | -------- | -------|
| 探索 | 新しい項目を探索できませんでした - Microsoft Azure Backup で、内部エラーが発生しました。しばらくしてから操作をやり直してください。 | 15 分後に探索プロセスを再試行します。
| 探索 | 新しい項目を探索できませんでした - 別の探索操作が既に進行中です。現在の探索操作が完了するまでお待ちください。 | なし |

## 登録
| バックアップ操作 | エラーの詳細 | 対処法 |
| -------- | -------- | -------|
| 登録 | 仮想マシンに接続されたデータ ディスクの数がサポートされている上限を超えています - この仮想マシンのデータ ディスクをいくつか切断してから、操作をやり直してください。Azure Backup では、バックアップ用に Azure の仮想マシンに接続できるデータ ディスクは 16 個までです。 | なし |
| 登録 | Microsoft Azure Backup で内部エラーが発生しました。しばらくしてから操作をやり直してください。引き続き問題が発生する場合は、Microsoft サポートにお問い合わせください。 | 次のいずれかの構成がサポートされていないことが原因で、このエラーが発生する場合があります。<ul><li>Premium LRS </ul> |
| 登録 | エージェントのインストール処理がタイムアウトしたため、登録できませんでした | 仮想マシンの OS のバージョンがサポート対象かどうかを確認します。 |
| 登録 | コマンド実行に失敗しました - この項目で別の操作が実行中です。前の操作が完了するまでお待ちください。 | なし |
| 登録 | 仮想ハード ディスクが Premium Storage に格納されている仮想マシンは、バックアップでサポートされていません。 | なし |
| 登録 | 仮想マシン エージェントが仮想マシン上に存在しません - 必要な前提条件である VM エージェントをインストールしてから、操作をやり直してください。 | VM エージェントのインストール方法と、VM エージェントのインストールを検証する方法については、[こちら](#vm-agent)を参照してください。 |

## バックアップ

| バックアップ操作 | エラーの詳細 | 対処法 |
| -------- | -------- | -------|
| バックアップ | バックアップ資格情報コンテナーからの VHD のコピーがタイムアウトしました - 数分以内に操作をやり直してください。問題が解決しない場合は、Microsoft サポートにお問い合わせください。 | これは、データが多すぎてコピーできなかった場合に発生します。データ ディスクが 15 個以下であることを確認してください。 |
| バックアップ | スナップショットの状態について VM エージェントと通信できませんでした。スナップショット VM サブタスクがタイムアウトしました。 - この問題を解決する方法については、トラブルシューティング ガイドを参照してください。 | VM エージェントに問題があるか、Azure インフラストラクチャへのネットワーク アクセスが何らかの原因でブロックされている場合に、このエラーがスローされます。<ul> <li>[VM エージェントの問題のデバッグ](#vm-agent)について <li>[ネットワークの問題のデバッグ](#networking)について <li>VM エージェントが正常に実行されている場合は、[VM スナップショットに関する問題のトラブルシューティングについて](#Troubleshoot-VM-Snapshot-Issues)</ul><br>VM エージェントが問題の原因でない場合は、VM を再起動してください。VM の状態が正しくないため問題が発生する場合があります。その場合は、VM を再起動すると、この "問題のある状態" がリセットされます。 |
| バックアップ | バックアップ操作は内部エラーのため失敗しました - 数分以内に操作をやり直してください。問題が解決しない場合は、Microsoft サポートにお問い合わせください。 | このエラーが発生する原因は、次の 2 つです。<ol><li> データが多すぎてコピーできない。<li>元の VM が削除されているため、バックアップを実行できない。削除された VM のバックアップ データを保持しながら、バックアップ エラーを停止するには、VM の保護を解除し、データを保持するオプションを選択します。これにより、バックアップ スケジュールを停止しながら、エラー メッセージが繰り返し表示されることも回避できます。 |
| バックアップ | 選択した項目に Azure Recovery Services 拡張機能をインストールできませんでした - VM エージェントは、Azure Recovery Services 拡張機能の前提条件です。Azure VM エージェントをインストールしてから、登録操作をやり直してください。 | <ol> <li>VM エージェントが正しくインストールされていることを確認します。<li>VM 構成のフラグが正しく設定されていることを確認します。</ol> VM エージェントのインストール方法と、VM エージェントのインストールを検証する方法については、[こちら](#validating-vm-agent-installation)を参照してください。 |
| バックアップ | コマンド実行に失敗しました - 現在、この項目で別の操作が実行中です。前の操作が完了するまで待ってから、やり直してください。 | VM で既存のバックアップ ジョブまたは復元ジョブが実行されている場合、既存のジョブの実行中に新しいジョブを開始することはできません。 |
| バックアップ | "COM+: Microsoft 分散トランザクション コーディネーターと通信できませんでした" というエラーで拡張機能のインストールが失敗しました。 | 通常、これは、COM + サービスが実行されていないことを意味します。この問題の修正については、Microsoft サポートにお問い合わせください。 |
| バックアップ | スナップショットの操作は、"このドライブは、BitLocker ドライブ暗号化でロックされています。コントロール パネルからドライブのロックを解除してください。" という VSS 操作エラーで失敗しました。 | VM 上のすべてのドライブで BitLocker をオフにして、VSS の問題が解決されたかどうかを確認します。 |
| バックアップ | 仮想ハード ディスクが Premium Storage に格納されている仮想マシンは、バックアップでサポートされていません。 | なし |
| バックアップ | Azure 仮想マシンが見つかりません。 | これは、プライマリ VM が削除されているのに、バックアップ ポリシーによってバックアップを実行する VM が検索され続ける場合に発生します。このエラーを解決するには、次の手順に従います。<ol><li>同じ名前と同じリソース グループ名 [クラウド サービス名] を使用して仮想マシンを作成し直します。<br>(または)<li>バックアップ ジョブが作成されないように、この VM の保護を無効にします。</ol> |
| バックアップ | 仮想マシン エージェントが仮想マシン上に存在しません - 必要な前提条件である VM エージェントをインストールしてから、操作をやり直してください。 | VM エージェントのインストール方法と、VM エージェントのインストールを検証する方法については、[こちら](#vm-agent)を参照してください。 |

## ジョブ
| 操作 | エラーの詳細 | 対処法 |
| -------- | -------- | -------|
| ジョブの取り消し | このジョブの種類では取り消しがサポートされていません - ジョブが完了するまでお待ちください。 | なし |
| ジョブの取り消し | ジョブが取り消しできる状態にありません - ジョブが完了するまでお待ちください。<br>または<br> 選択したジョブは取り消しできる状態にありません。ジョブが完了するまでお待ちください。| ジョブはほぼ完了しています。ジョブが完了するまでお待ちください。 |
| ジョブの取り消し | 進行中ではないためジョブを取り消すことができません - 取り消しがサポートされているのは、進行中のジョブだけです。進行中のジョブの取り消しを試してください。 | これは一時的な状態が原因で発生しています。しばらく待ってから取り消し操作をやり直してください。 |
| ジョブの取り消し | ジョブを取り消すことができませんでした - ジョブが終了するまでお待ちください。 | なし |


## 復元
| 操作 | エラーの詳細 | 対処法 |
| -------- | -------- | -------|
| 復元 | クラウドの内部エラーの復元に失敗しました | <ol><li>復元を試みているクラウド サービスが DNS 設定で構成されています。<br>$deployment = Get-AzureDeployment -ServiceName "ServiceName" -Slot "Production" Get-AzureDns -DnsSettings $deployment.DnsSettings<br> を確認します。構成済みのアドレスがある場合は、DNS 設定が構成されていることを意味します。<br> <li>復元を試みているクラウド サービスが ReservedIP で構成されています。<br>次の PowerShell コマンドレットを使用して、クラウド サービスに予約済み IP があることを確認できます。<br>$deployment = Get-AzureDeployment -ServiceName "servicename" -Slot "Production" $dep.ReservedIPName <br><li>次の特殊なネットワーク構成の仮想マシンを同じクラウド サービスに復元しようとしています。<br>- ロード バランサー構成の仮想マシン (内部と外部)<br>- 複数の予約済み IP がある仮想マシン<br>- 複数の NIC がある仮想マシン<br>UI で新しいクラウド サービスを選択するか、特殊なネットワーク構成の VM の[復元に関する考慮事項](backup-azure-restore-vms.md/#restoring-vms-with-special-network-configurations)を参照してください</ol> |
| 復元 | 選択した DNS 名は既に使用されています - 別の DNS 名を指定してからやり直してください。 | この場合、DNS 名はクラウド サービス名 (通常、末尾に cloudapp.net が付いています) を表します。これは一意である必要があります。このエラーが発生した場合は、復元中に別の VM の名前を選択する必要があります。<br><br> このエラーは Azure ポータルのユーザーのみに表示されることに注意してください。PowerShell による復元操作は、ディスクを復元するだけで、VM を作成しないため、成功します。ディスクの復元操作後に VM を明示的に作成すると、このエラーが発生します。 |
| 復元 | 指定された仮想ネットワークの構成が正しくありません - 別の仮想ネットワークの構成を指定してからやり直してください。 | なし |
| 復元 | 指定したクラウド サービスでは、復元対象の仮想マシンの構成と一致しない予約済み IP が使用されています - 予約済み IP を使用していない別のクラウド サービスを指定するか、復元元に別の回復ポイントを選択してください。 | なし |
| 復元 | クラウド サービスが入力エンドポイントの数に制限に達しました - 別のクラウド サービスを指定するか既存のエンドポイントを使用して、操作をやり直してください。 | なし |
| 復元 | バックアップ資格情報コンテナーと対象のストレージ アカウントが 2 つの異なるリージョンに存在します - 復元操作で指定したストレージ アカウントが、バックアップ資格情報コンテナーと同じ Azure リージョンに存在するようにしてください。 | なし |
| 復元 | 復元操作に指定されたストレージ アカウントがサポートされていません - サポートされているのは、ローカル冗長レプリケーションまたは geo 冗長レプリケーションの設定が指定された Basic または Standard ストレージ アカウントのみです。サポートされているストレージ アカウントを選択してください。 | なし |
| 復元 | 復元操作に指定されたストレージ アカウントの種類がオンラインではありません - 復元操作で指定したストレージ アカウントがオンラインであることを確認してください。 | これは、Azure Storage の一時的なエラーや障害が原因で発生する可能性があります。別のストレージ アカウントを選択してください。 |
| 復元 | リソース グループのクォータに達しました - プレビュー ポータルの一部のリソース グループを削除するか、Azure サポートに問い合わせて上限を引き上げてください。 | なし |
| 復元 | 選択したサブネットが存在しません - 存在するサブネットを選択してください。 | なし |


## ポリシー
| 操作 | エラーの詳細 | 対処法 |
| -------- | -------- | -------|
| ポリシーの作成 | ポリシーを作成できませんでした - リテンション期間の選択肢を減らしてポリシーの構成を続行してください。 | なし |


## VM エージェント

### VM エージェントの設定
通常、VM エージェントは、Azure ギャラリーから作成された仮想マシン内に既に存在しています。しかし、オンプレミスのデータセンターから移行された仮想マシンには VM エージェントがインストールされていません。このような VM については、VM エージェントを明示的にインストールする必要があります。既存の VM に VM エージェントをインストールする方法については、[こちら](http://blogs.msdn.com/b/mast/archive/2014/04/08/install-the-vm-agent-on-an-existing-azure-vm.aspx)を参照してください。

Windows VM の場合:

- [エージェント MSI](http://go.microsoft.com/fwlink/?LinkID=394789&clcid=0x409) をダウンロードしてインストールします。インストールを実行するには、管理者特権が必要です。
- [VM プロパティを更新](http://blogs.msdn.com/b/mast/archive/2014/04/08/install-the-vm-agent-on-an-existing-azure-vm.aspx)して、エージェントがインストールされていることを示します。

Linux VM の場合:

- github から最新の [Linux エージェント](https://github.com/Azure/WALinuxAgent)をインストールします。
- [VM プロパティを更新](http://blogs.msdn.com/b/mast/archive/2014/04/08/install-the-vm-agent-on-an-existing-azure-vm.aspx)して、エージェントがインストールされていることを示します。


### VM エージェントの更新
Windows VM の場合:

- VM エージェントを更新するには、単純に [VM エージェント バイナリ](http://go.microsoft.com/fwlink/?LinkID=394789&clcid=0x409)を再インストールします。ただし、VM エージェントの更新中にバックアップ操作が実行されないようにする必要があります。

Linux VM の場合:

- [Linux VM エージェントの更新](../virtual-machines/virtual-machines-linux-update-agent.md)に関する手順に従ってください。


### VM エージェントのインストールの検証
Windows VM 上で VM エージェントのバージョンを確認する方法:

1. Azure 仮想マシンにログオンし、フォルダー *C:\\WindowsAzure\\Packages* に移動します。WaAppAgent.exe ファイルを探します。
2. このファイルを右クリックして、**[プロパティ]** をクリックした後、**[詳細]** タブを選択します。[製品バージョン] が 2.6.1198.718 以上であることを確認します。

## VM スナップショットに関する問題のトラブルシューティング
VM のバックアップは、基になるストレージのスナップショット コマンドの発行に依存します。ストレージへのアクセスがない、またはスナップショット タスクの実行に遅延が発生すると、バックアップが失敗することがあります。次の場合にスナップショットのタスクが失敗することがあります。

1. NSG を使用してストレージへのネットワーク アクセスがブロックされています<br> IP のホワイトラベルを使用またはプロキシ サーバーを通じてストレージへの[ネットワーク アクセスを有効にする](backup-azure-vms-prepare.md#2-network-connectivity)方法について学習します。
2.  SQL Server のバックアップが構成されている VM はスナップショット タスクの遅延を引き起こすことがあります <br> 既定では VM のバックアップが Windows VM 上の VSS フル バックアップを発行します。SQL Server を実行しており、SQL Server のバックアップが構成されている VM では、これによりスナップショットの遅延が引き起こされる可能性があります。スナップショットに関する問題によりバックアップが失敗する場合は、次のレジストリ キーを設定してください。

	```
	[HKEY_LOCAL_MACHINE\SOFTWARE\MICROSOFT\BCDRAGENT]
	"USEVSSCOPYBACKUP"="TRUE"
	```
3.  VM が RDP でシャットダウンされているため、VM の状態が正しく報告されません。<br> RDP で仮想マシンをシャットダウンした場合は、ポータルに戻って VM の状態が正しく反映されていることを確認してください。していない場合は、VM のダッシュボードで [シャットダウン] オプションを使用して、ポータルの VM をシャットダウンしてください。
4.  同じクラウド サービスから多数の VM が同時にバックアップするように構成されています。<br> 同じクラウド サービスからの VM は異なるバックアップ スケジュールに分散することをお勧めします。
5.  VM の CPU/メモリの使用率が高くなっています。<br> 仮想マシンの CPU またはメモリの使用率が高くなる (90% 以上) と、スナップショット タスクがキューに配置されて遅延し、最終的にはタイムアウトします。このような状況では、オンデマンド バックアップをお試しください。

<br>

## ネットワーク
Backup 拡張機能は、他の拡張機能と同様に、パブリックなインターネットへのアクセスが必要です。パブリックなインターネットにアクセスできない場合、さまざまな問題が発生する可能性があります。

- 拡張機能のインストールが失敗する
- ディスク スナップショットなどのバックアップ操作が失敗する
- バックアップ操作の状態を表示できない

パブリックなインターネット アドレスを解決する必要性については、[この記事](http://blogs.msdn.com/b/mast/archive/2014/06/18/azure-vm-provisioning-stuck-on-quot-installing-extensions-on-virtual-machine-quot.aspx)をご覧ください。VNET 用の DNS 構成を確認し、Azure の URI が解決できることを確認する必要があります。

名前解決が正しく実行された後で、Azure IP へのアクセスも提供する必要があります。Azure インフラストラクチャへのアクセスのブロックを解除するには、次のいずれかの手順に従います。

1. Azure データ センターの IP 範囲をホワイトリストに登録する。
    - ホワイトリストに登録する [Azure データセンター IP](https://www.microsoft.com/download/details.aspx?id=41653) の一覧を取得します。
    - [New-NetRoute](https://technet.microsoft.com/library/hh826148.aspx) コマンドレットを使用して、IP アドレスのブロックを解除します。管理者特権の PowerShell ウィンドウ (管理者として実行) で、Azure VM 内でこのコマンドレットを実行します。
    - 規則を NSG に追加して IP にアクセスできるようにします (NSG を使用している場合)。
2. フローに対する HTTP トラフィック用のパスを作成する
    - 何らかのネットワーク制限 (ネットワーク セキュリティ グループなど) を設定している場合は、トラフィックをルーティングするための HTTP プロキシ サーバーをデプロイします。HTTP プロキシ サーバーをデプロイするための手順は、[ここ](backup-azure-vms-prepare.md#2-network-connectivity)を参照してください。
    - 規則を NSG に追加して HTTP プロキシからインターネットにアクセスできるようにします (NSG を使用している場合)。

>[AZURE.NOTE] IaaS VM Backup が正しく機能するためには、ゲスト内で DHCP が有効になっている必要があります。静的プライベート IP が必要な場合は、プラットフォームを通じて構成する必要があります。VM 内の DHCP オプションは有効のままにしておいてください。静的内部プライベート IP の設定の詳細については、[こちら](virtual-networks-reserved-private-ip.md)を参照してください。

<!---HONumber=AcomDC_0204_2016-->
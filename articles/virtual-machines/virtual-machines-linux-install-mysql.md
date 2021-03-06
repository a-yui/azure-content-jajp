<properties
	pageTitle="MySQL を Linux VM にセットアップする | Microsoft Azure"
	description="Azure 上の Linux 仮想マシン (Ubuntu または RedHat ファミリ OS) に MySQL スタックをインストールする方法について説明します。"
	services="virtual-machines"
	documentationCenter=""
	authors="SuperScottz"
	manager="timlt"
	editor=""
	tags="azure-resource-manager,azure-service-management"/>

<tags
	ms.service="virtual-machines"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-linux"
	ms.devlang="na"
	ms.topic="article"
	ms.date="02/01/2016"
	ms.author="mingzhan"/>


#MySQL を Azure でインストールする方法


この記事では、Linux を実行している Azure 仮想マシンに MySQL をインストールして構成する方法を説明します。

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-both-include.md)]


##仮想マシンに MySQL をインストールする

> [AZURE.NOTE] このチュートリアルを実行するには、Linux を実行する Microsoft Azure Virtual Machine が既に存在している必要があります。続行する前に、[Azure Linux VM チュートリアル](virtual-machines-linux-tutorial.md)を見て、VM 名を`mysqlnode`、ユーザー名を `azureuser` として、Linux VM を作成およびセットアップしてください。

この例では、MySQL ポートとしてポート 3306 を使用します。

putty を使用して作成した Linux VM に接続します。初めて Azure Linux VM を使用する場合、putty を使用して Linux VM に接続する方法については、[こちら](virtual-machines-linux-use-ssh-key.md)を参照してください。

この記事では、例として MySQL5.6 のインストールにパッケージのリポジトリを使用します。実際、MySQL5.6 は、MySQL5.5 よりもパフォーマンスが向上しています。詳細情報は[こちら](http://www.mysqlperformanceblog.com/2013/02/18/is-mysql-5-6-slower-than-mysql-5-5/)です。


###Ubuntu で MySQL 5.6 をインストールする方法
ここでは、Azure の Ubuntu Linux VM を使用します。

- 手順 1. MySQL Server 5.6 をインストールし、ユーザー `root` に切り替える

            #[azureuser@mysqlnode:~]sudo su -

    mysql-server 5.6 をインストールする

            #[root@mysqlnode ~]# apt-get update
            #[root@mysqlnode ~]# apt-get -y install mysql-server-5.6

    インストール中に、MySQL ルート パスワードを設定するよう求めるダイアログ ウィンドウが表示されるので、パスワードを設定します。

    ![image](./media/virtual-machines-linux-install-mysql/virtual-machines-linux-install-mysql-p1.png)


    確認のため、もう一度パスワードを入力します。

    ![イメージ](./media/virtual-machines-linux-install-mysql/virtual-machines-linux-install-mysql-p2.png)

- 手順 2. MySQL サーバー にログインする

    MySQL サーバー のインストールが完了したら、MySQL サービスが自動的に開始されます。MySQL サーバーには、ユーザー `root` としてログインできます。以下のコマンドを使用してログインし、パスワードを入力します。

             #[root@mysqlnode ~]# mysql -uroot -p

- 手順 3. MySQL サービスの実行を管理する

    (a) MySQL サービスの状態を取得する

             #[root@mysqlnode ~]# service mysql status

    (b) MySQL サービスを開始する

             #[root@mysqlnode ~]# service mysql start

    (c) MySQL サービスを停止する

             #[root@mysqlnode ~]# service mysql stop

    (d) MySQL サービスを再起動する

             #[root@mysqlnode ~]# service mysql restart


###Cent OS、Oracle Linux など、Red Hat OS ファミリに MySQL をインストールする方法
ここでは CentOS または Oracle Linux と Linux VM を使用します。

- 手順 1. MySQL Yum リポジトリ スイッチを追加して、ユーザー `root` に切り替える

            #[azureuser@mysqlnode:~]sudo su -

    MySQL リリース パッケージをダウンロードして、インストールします。

            #[root@mysqlnode ~]# wget http://repo.mysql.com/mysql-community-release-el6-5.noarch.rpm
            #[root@mysqlnode ~]# yum localinstall -y mysql-community-release-el6-5.noarch.rpm

- 手順 2. MySQL リポジトリを有効にして MySQL5.6 パッケージをダウンロードするため、以下のファイルを編集する

            #[root@mysqlnode ~]# vim /etc/yum.repos.d/mysql-community.repo

    このファイルの各値を以下のように更新します。

        # *Enable to use MySQL 5.6*

        [mysql56-community]
        name=MySQL 5.6 Community Server

        baseurl=http://repo.mysql.com/yum/mysql-5.6-community/el/6/$basearch/

        enabled=1

        gpgcheck=1

        gpgkey=file:/etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

- 手順 3. MySQL リポジトリから Install MySQL を使用して MySQL をインストールする

           #[root@mysqlnode ~]#yum install mysql-community-server

    MySQL RPM パッケージとすべての関連するパッケージがインストールされます。

- 手順 4. MySQL サービスの実行を管理する

    (a) MySQL サーバーのサービスの状態を確認する

           #[root@mysqlnode ~]#service mysqld status

    (b) MySQL Server の既定のポートが実行されているかどうかを確認する

           #[root@mysqlnode ~]#netstat  –tunlp|grep 3306


    (c) MySQL サーバーを開始する

           #[root@mysqlnode ~]#service mysqld start

    (d) MySQL サーバーを停止する

           #[root@mysqlnode ~]#service mysqld stop

    (e) 次回システム起動時に MySQL を開始するよう設定する

           #[root@mysqlnode ~]#chkconfig mysqld on


###MySQL を Suse LinuX にインストールする方法
ここでは、OpenSUSE と Linux 仮想マシンを使用します。

- 手順 1. MySQL Server をダウンロードしてインストールする

    以下のコマンドを使用し、ユーザー `root` に切り替えます。

           #sudo su -

    MySQL Server パッケージをダウンロードしてインストールします。

           #[root@mysqlnode ~]# zypper update

           #[root@mysqlnode ~]# zypper install mysql-server mysql-devel mysql

- 手順 2. MySQL サービスの実行を管理する

    (a) MySQL サーバーの状態を確認する

           #[root@mysqlnode ~]# rcmysql status

    (b) MySQL Server の既定のポートを確認する

           #[root@mysqlnode ~]# netstat  –tunlp|grep 3306


    (c) MySQL サーバーを開始する

           #[root@mysqlnode ~]# rcmysql start

    (d) MySQL サーバーを停止する

           #[root@mysqlnode ~]# rcmysql stop

    (e) 次回システム起動時に MySQL を開始するよう設定する

           #[root@mysqlnode ~]# insserv mysql

###次の手順
MySQL の使用法などの情報は[こちら](https://www.mysql.com/)です。

<!---HONumber=AcomDC_0204_2016-->
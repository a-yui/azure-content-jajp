<properties
 pageTitle="MPI アプリケーションを実行するための Linux RDMA クラスター |Microsoft Azure"
 description="RDMA を使用して MPI アプリケーションを実行する、サイズ A8 または A9 の VM の Linux クラスターを作成します。"
 services="virtual-machines"
 documentationCenter=""
 authors="dlepow"
 manager="timlt"
 editor=""
 tags="azure-service-management"/>
<tags
ms.service="virtual-machines"
 ms.devlang="na"
 ms.topic="article"
 ms.tgt_pltfrm="vm-linux"
 ms.workload="infrastructure-services"
 ms.date="01/21/2016"
 ms.author="danlep"/>

# MPI アプリケーションを実行するように Linux RDMA クラスターを設定する

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-classic-include.md)]リソース マネージャー モデル。


この記事では、Azure で[サイズ A8 および A9 の仮想マシン](virtual-machines-a8-a9-a10-a11-specs.md)を使用して Linux RDMA クラスターを設定し、並列 Message Passing Interface (MPI) アプリケーションを実行する方法について説明します。サイズ A8 および A9 の Linux ベースの VM を構成して、サポートされている MPI 実装を実行すると、MPI アプリケーションは、リモート ダイレクト メモリ アクセス (RDMA) テクノロジに基づく Azure の低待機時間で高スループットのネットワークを介して効率的に通信します。

>[AZURE.NOTE] Azure Linux RDMA は、現在、Azure Marketplace の SUSE Linux Enterprise Server 12 (SLES 12) イメージで実行されている Intel MPI Library バージョン 5 でサポートされています。この記事は、Intel MPI バージョン 5.0.3.048 に基づきます。
>
> Azure は、RDMA バックエンド ネットワークに接続せずに、A8 および A9 インスタンスと同じ処理機能を持つ、A10 および A11 のコンピューティング集中型インスタンスも提供します。Azure で MPI ワークロードを実行する場合、一般に、A8 および A9 インスタンスで最良のパフォーマンスが得られます。


## Linux クラスターのデプロイ オプション

ジョブ スケジューラを使用する場合、または使用しない場合に、Linux の RDMA クラスターの作成に使用できる方法を次に示します。

* **HPC Pack** - Azure で Microsoft HPC Pack クラスターを作成して、サポートされる Linux ディストリビューションを実行するコンピューティング ノードを追加します。Linux の特定のノードを構成して、RDMA ネットワークにアクセスできます。「[Azure の HPC Pack クラスターで Linux コンピューティング ノードの使用を開始する](virtual-machines-linux-cluster-hpcpack.md)」を参照してください。

* **Azure CLI スクリプト** - この記事の以降の手順に示されるように、Mac、Linux、および Windows 用の [Azure コマンド ライン インターフェイス](../xplat-cli-install.md) (CLI) を使用して、仮想ネットワークやその他の必要なコンポーネントのデプロイのスクリプトを作成し、Linux クラスターを作成することができます。クラシック (サービス管理) デプロイ モード の CLI では、クラスター ノードを順番に作成します。そのため、多くのコンピューティング ノードをデプロイしている場合は、デプロイの完了に数分かかる場合があります。

* **Azure リソース マネージャー テンプレート** - Azure リソース マネージャー デプロイ モデルを使用して、複数の A8 および A9 Linux VM をデプロイすると共に、MPI ワークロードを実行するために RDMA ネットワークを活用できるコンピューティング クラスターの仮想ネットワーク、静的 IP アドレス、DNS 設定、その他のリソースを定義します。[独自のテンプレートを作成](../resource-group-authoring-templates.md)するか、または [「Azure クイック スタート テンプレート」ページ](https://azure.microsoft.com/documentation/templates/)で Microsoft またはコミュニティから提供されたテンプレートを確認して、目的のソリューションをデプロイすることができます。リソース マネージャー テンプレートは、Linux クラスターをデプロイするための高速で信頼性の高い方法を提供します。

## Azure CLI スクリプトによる Azure サービス管理でのデプロイ

次の手順は、Azure CLI を使用して、SUSE Linux Enterprise Server 12 VM をデプロイし、Intel MPI Library およびその他のカスタマイズをインストールして、カスタム VM イメージを作成し、A8 または A9 VM のクラスターをデプロイするスクリプトを作成する際に役立ちます。

### 前提条件

*   **クライアント コンピューター** - Azure と通信するための Mac、Linux、または Windows ベースのクライアント コンピューターが必要です。これらの手順は、Linux クライアントを使用していることを前提とします。

*   **Azure サブスクリプション** - アカウントがない場合は、無料試用版のアカウントを数分で作成することができます。詳細については、[Azure の無料試用版サイト](https://azure.microsoft.com/pricing/free-trial/)を参照してください。

*   **コア クォータ** - A8 または A9 VM のクラスターをデプロイするために、コア クォータを増やすことが必要な場合があります。たとえば、8 個の A9 VM をデプロイする場合は、少なくとも 128 コアが必要です。クォータを増やすには、[オンライン カスタマー サポートに申請](https://azure.microsoft.com/blog/2014/06/04/azure-limits-quotas-increase-requests/) (無料) してください。

*   **Azure CLI** - Azure CLI を[インストール](../xplat-cli-install.md)し、クライアント コンピューターから Azure サブスクリプションに接続するように[構成](../xplat-cli-connect.md)します。

*   **Intel MPI** - クラスター用の Linux VM イメージのカスタマイズの一環として (この記事の後半の説明を参照)、Intel MPI Library 5 ランタイムを [Intel.com のサイト](https://software.intel.com/ja-JP/intel-mpi-library/)からダウンロードし、インストールする必要があります。この準備をするために、Intel に登録した後、確認の電子メールに含まれる関連 Web ページへのリンクをクリックし、適切なバージョンの Intel MPI (.tgz ファイル) のダウンロード リンクをコピーします。この記事は、Intel MPI バージョン 5.0.3.048 に基づきます。

### SLES 12 VM のプロビジョニング

Azure CLI を使用して Azure にログインした後に `azure config list` を実行して、出力に **asm** モードが示されている (CLI が Azure サービス管理モードである) ことを確認します。asm モードでない場合は、次のコマンドを実行して asm モードに設定します。

```
azure config mode asm
```

使用する権限があるすべてのサブスクリプションを一覧表示するには、次を入力します。

```
azure account list
```

現在のアクティブなサブスクリプションは、`Current` が `true` に設定されています。クラスターの作成に使用するサブスクリプションがこれではない場合、適切なサブスクリプション番号をアクティブなサブスクリプションとして設定します。

```
azure account set <subscription-number>
```

Azure でパブリックに利用可能な SLES 12 HPC イメージを表示するには、次のようなコマンドを実行します (シェル環境で **grep** がサポートされていると仮定)。

```
azure vm image list | grep "suse.*hpc"
```

>[AZURE.NOTE]SLES 12 HPC イメージは、必要な Azure 用 Linux RDMA ドライバーを使用して事前構成されています。

次のようなコマンドを実行して、使用可能な SLES 12 HPC イメージを使用し、サイズ A9 VM をプロビジョニングします。

```
azure vm create -g <username> -p <password> -c <cloud-service-name> -l <location> -z A9 -n <vm-name> -e 22 b4590d9e3ed742e4a1d46e5424aa335e__suse-sles-12-hpc-v20150708
```

各値の説明:

* サイズには A8 または A9 (この例では A9) を指定できます。

* 外部 SSH ポート番号 (この例では、SSH の規定である 22) は任意の有効なポート番号です。内部 SSH ポート番号は 22 に設定されます。

* 新しいクラウド サービスは、場所で指定された Azure リージョンに作成されます。A8 および A9 インスタンスに使用できる、"米国西部" などの場所を指定します。

* イメージ名は、現在、`b4590d9e3ed742e4a1d46e5424aa335e__suse-sles-12-hpc-v20150708` (無料) または `b4590d9e3ed742e4a1d46e5424aa335e__suse-sles-12-hpc-priority-v20150708` (SUSE 優先サポート。料金がかかります) です。

### VM のカスタマイズ

VM のプロビジョニングの完了後、VM の外部 IP アドレス (または DNS 名) と構成済みポート番号を使用して VM に SSH 接続し、カスタマイズします。接続の詳細については、「[Linux を実行する仮想マシンにログオンする方法](virtual-machines-linux-how-to-log-on.md)」を参照してください。手順の完了にルート アクセスが必要でない限り、VM で構成したユーザーとしてコマンドを実行する必要があります。

>[AZURE.IMPORTANT]Microsoft Azure では Linux VM にルート アクセスが提供されません。ユーザーとして VM に接続されているときに管理アクセス権を取得するには、`sudo` を使用してコマンドを実行します。

*   **更新プログラム** - **zypper** を使用して更新プログラムをインストールします。NFS ユーティリティをインストールすることもできます。  

    >[AZURE.IMPORTANT]現時点では、カーネルの更新プログラムを適用しないことをお勧めします。適用した場合、Linux RDMA ドライバーに関連する問題が発生する可能性があります。

*   **Intel MPI** - 次のようなコマンドを実行して、Intel MPI Library 5 ランタイムを Intel.com のサイトからダウンロードしてインストールします。

    ```
    sudo wget <download link for your registration>

    sudo tar xvzf <tar-file>

    cd <mpi-directory>

    sudo ./install.sh

    ```

    既定の設定をそのまま使用して、Intel MPI を VM にインストールします。

*   **メモリのロック** - MPI コードによって、RDMA で使用可能なメモリをロックするには、/etc/security/limits.conf ファイルで、次の設定を追加または変更する必要があります(このファイルを編集するにはルート アクセスが必要です)。

    ```
    <User or group name> hard    memlock <memory required for your application in KB>

    <User or group name> soft    memlock <memory required for your application in KB>
    ```

    >[AZURE.NOTE]テスト目的で、memlock を無制限に設定することもできます。(例: `<User or group name>    hard    memlock unlimited`)。


*   **SSH キー** - MPI ジョブの実行時にクラスター内のすべてのコンピューティング ノード間でユーザー アカウントの信頼を確立するために、SSH キーを生成します。次のコマンドを実行して、SSH キーを作成します。パスフレーズを設定せずに、単に Enter を押して既定の場所でキーを生成します。

    ```
    ssh-keygen
    ```

    公開キーを、既知の公開キー用の authorized\_keys ファイルに追加します。

    ```
    cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
    ```

    ~/.ssh ディレクトリにある "config" ファイルを編集 (存在しない場合は作成) します。Azure で使用するプライベート ネットワークの IP アドレスの範囲を指定します (この例では 10.32.0.0/16)。

    ```
    host 10.32.0.*
    StrictHostKeyChecking no
    ```

    または、クラスター内の各 VM のプライベート ネットワーク IP アドレスを、次のように一覧表示します。

    ```
    host 10.32.0.1
     StrictHostKeyChecking no
    host 10.32.0.2
     StrictHostKeyChecking no
    host 10.32.0.3
     StrictHostKeyChecking no
    ```

    >[AZURE.NOTE]特定の IP アドレスまたは範囲を指定しない場合、これらの例のように `StrictHostKeyChecking no` を構成すると、潜在的なセキュリティ リスクが生じる可能性があります。

* **アプリケーション** - イメージをキャプチャする前に、必要なアプリケーションをこの VM にインストールしたり、その他のカスタマイズを実行したりします。

### イメージのキャプチャ

イメージをキャプチャするには、最初に、Linux VM で次のコマンドを実行します。これにより、VM がプロビジョニング解除されますが、設定したユーザー アカウントと SSH キーは維持されます。

```
sudo waagent -deprovision
```

次に、クライアント コンピューターで次の Azure CLI コマンドを実行し、イメージをキャプチャします。詳細については、「[従来の Linux 仮想マシンをイメージとしてキャプチャする方法](virtual-machines-linux-capture-image.md)」を参照してください。

```
azure vm shutdown <vm-name>

azure vm capture -t <vm-name> <image-name>

```

これらのコマンドを実行した後、使用する VM イメージがキャプチャされ、VM が削除されます。カスタム イメージをクラスターにデプロイする準備ができました。

### イメージを使用したクラスターのデプロイ

次の Bash スクリプトの該当部分を環境に合った適切な値に変更し、クライアント コンピューターで実行します。サービス管理のデプロイ方法では VM を順番にデプロイするため、このスクリプトで示されている 8 個の A9 VM をデプロイするまでに数分かかります。

```
#!/bin/bash -x
# Script to create a compute cluster without a scheduler in a VNet in Azure
# Create a custom private network in Azure
# Replace 10.32.0.0 with your virtual network address space
# Replace <network-name> with your network identifier
# Select a region where A8 and A9 VMs are available, such as West US
# See Azure Pricing pages for prices and availability of A8 and A9 VMs

azure network vnet create -l "West US" -e 10.32.0.0 -i 16 <network-name>

# Create a cloud service. All the A8 and A9 instances need to be in the same cloud service for Linux RDMA to work across InfiniBand.
# Note: The current maximum number of VMs in a cloud service is 50. If you need to provision more than 50 VMs in the same cloud service in your cluster, contact Azure Support.

azure service create <cloud-service-name> --location "West US" –s <subscription-ID>

# Define a prefix naming scheme for compute nodes, e.g., cluster11, cluster12, etc.

vmname=cluster

# Define a prefix for external port numbers. If you want to turn off external ports and use only internal ports to communicate between compute nodes via port 22, don’t use this option. Since port numbers up to 10000 are reserved, use numbers after 10000. Leave external port on for rank 0 and head node.

portnumber=101

# In this cluster there will be 8 size A9 nodes, named cluster11 to cluster18. Specify your captured image in <image-name>. Specify the username and password you used when creating the SSH keys.

for (( i=11; i<19; i++ )); do
        azure vm create -g <username> -p <password> -c <cloud-service-name> -z A9 -n $vmname$i -e $portnumber$i -w <network-name> -b Subnet-1 <image-name>
done

# Save this script with a name like makecluster.sh and run it in your shell environment to provision your cluster
```

## SLES 12 用 Linux RDMA ドライバーの更新

SLES 12 HPC イメージに基づいて Linux RDMA クラスターを作成したら、RDMA ネットワーク接続のために VM の RDMA ドライバーを更新することが必要になる場合があります。

>[AZURE.IMPORTANT]現在、この手順は、ほとんどの Azure リージョンにおける Linux RDMA クラスターのデプロイで**必須**となっています。**更新する必要がない SLES 12 VM は、米国西部、西ヨーロッパ、東日本のいずれかの Azure リージョンに作成された VM のみです。**

ドライバーを更新する前に、すべての **zypper** プロセス、または VM の SUSE リポジトリ データベースをロックするプロセスを停止します。そうしないと、ドライバーが正しく更新されない場合があります。


クライアント コンピューターで次の Azure CLI コマンド セットのいずれかを実行して、各 VM の Linux RDMA ドライバーを更新します。

**Azure サービス管理内でプロビジョニングされた VM の場合**

```
azure config mode asm

azure vm extension set <vm-name> RDMAUpdateForLinux Microsoft.OSTCExtensions 0.1
```

**Azure リソース マネージャーでプロビジョニングされた VM の場合**

```
azure config mode arm

azure vm extension set <resource-group> <vm-name> RDMAUpdateForLinux Microsoft.OSTCExtensions 0.1
```

>[AZURE.NOTE]ドライバーのインストールには時間がかかる場合があります。また、コマンドは出力なしで終了します。更新後、VM は再起動され、数分で使用できるようになります。

クラスター内のすべてのノードでドライバーを更新するスクリプトを作成することができます。たとえば、次のスクリプトは、前の手順のスクリプトによって作成された 8 ノード クラスター内のドライバーを更新します。

```

#!/bin/bash -x

# Define a prefix naming scheme for compute nodes, e.g., cluster11, cluster12, etc.

vmname=cluster

# Plug in appropriate numbers in the for loop below.

for (( i=11; i<19; i++ )); do

# For ASM VMs use the following command in your script.

azure vm extension set $vmname$i RDMAUpdateForLinux Microsoft.OSTCExtensions 0.1

# For ARM VMs use the following command in your script.

# azure vm extension set <resource-group> $vmname$i RDMAUpdateForLinux Microsoft.OSTCExtensions 0.1

done

```

## Intel MPI の構成と実行

Azure Linux RDMA で MPI アプリケーションを実行するには、Intel MPI に固有な特定の環境変数を構成する必要があります。次に、変数を構成してアプリケーションを実行するサンプル Bash スクリプトを示します。

```
#!/bin/bash -x

source /opt/intel/impi_latest/bin64/mpivars.sh

export I_MPI_FABRICS=shm:dapl

# THIS IS A MANDATORY ENVIRONMENT VARIABLEAND MUST BE SET BEFORE RUNNING ANY JOB
# Setting the variable to shm:dapl gives best performance for some applications
# If your application doesn’t take advantage of shared memory and MPI together, then set only dapl

export I_MPI_DAPL_PROVIDER=ofa-v2-ib0

# THIS IS A MANDATORY ENVIRONMENT VARIABLE AND MUST BE SET BEFORE RUNNING ANY JOB

export I_MPI_DYNAMIC_CONNECTION=0

# THIS IS A MANDATORY ENVIRONMENT VARIABLE AND MUST BE SET BEFORE RUNNING ANY JOB

# Command line to run the job

mpirun -n <number-of-cores> -ppn <core-per-node> -hostfile <hostfilename>  /path <path to the application exe> <arguments specific to the application>

#end
```

ホスト ファイルの形式は、次のとおりです。クラスター内の各ノードに対して 1 つの行を追加します。DNS 名ではなく、前に VNet から定義したプライベート IP アドレスを指定します。たとえば、IP アドレス 10.32.0.1 と 10.32.0.2 を持つ 2 つのホストでは、ファイルに次の内容が含まれています。

```
10.32.0.1:16
10.32.0.2:16
```

## Intel MPI をインストールした後に、基本的な 2 ノード クラスターを確認します。

この操作をまだ行っていない場合は、まず Intel MPI の環境を設定します。

```
source /opt/intel/impi_latest/bin64/mpivars.sh
```

### 単純な MPI コマンドの実行

MPI が正しくインストールされており、少なくとも 2 つのコンピューティング ノード間で通信できることを示すために、いずれかのコンピューティング ノードで単純な MPI コマンドを実行します。次の **mpirun** コマンドは、2 つのノードで **hostname** コマンドを実行します。

```
/opt/intel/impi_latest/bin64/mpirun -ppn 1 -n 2 -hosts <host1>,<host2> -env I_MPI_FABRICS=shm:dapl -env I_MPI_DAPL_PROVIDER=ofa-v2-ib0 -env I_MPI_DYNAMIC_CONNECTION=0 hostname
```
出力では、`-hosts` の入力として渡したすべてのノードの名前が一覧表示されます。たとえば、2 つのノードを含む **mpirun** コマンドは、次のような出力を返します。

```
cluster11
cluster12
```

### MPI ベンチマークの実行

次の Intel MPI コマンドでは、pingpong ベンチマークを使用して、クラスターの構成および RDMA ネットワークへの接続を確認します。

```
/opt/intel/impi_latest/bin64/mpirun -hosts <host1>,<host2> -ppn 1 -n 2 -env I_MPI_FABRICS=dapl -env I_MPI_DAPL_PROVIDER=ofa-v2-ib0 -env I_MPI_DYNAMIC_CONNECTION=0 /opt/intel/impi_latest/bin64/IMB-MPI1 pingpong
```

2 つのノードで動作中のクラスターに、次のような出力が表示されます。Azure RDMA ネットワークでは、最大 512 バイトのメッセージ サイズに対して、3 マイクロ秒以下の待機時間が必要です。

```
#------------------------------------------------------------
#    Intel (R) MPI Benchmarks 4.0 Update 1, MPI-1 part
#------------------------------------------------------------
# Date                  : Fri Jul 17 23:16:46 2015
# Machine               : x86_64
# System                : Linux
# Release               : 3.12.39-44-default
# Version               : #5 SMP Thu Jun 25 22:45:24 UTC 2015
# MPI Version           : 3.0
# MPI Thread Environment:
# New default behavior from Version 3.2 on:
# the number of iterations per message size is cut down
# dynamically when a certain run time (per message size sample)
# is expected to be exceeded. Time limit is defined by variable
# "SECS_PER_SAMPLE" (=> IMB_settings.h)
# or through the flag => -time

# Calling sequence was:
# /opt/intel/impi_latest/bin64/IMB-MPI1 pingpong
# Minimum message length in bytes:   0
# Maximum message length in bytes:   4194304
#
# MPI_Datatype                   :   MPI_BYTE
# MPI_Datatype for reductions    :   MPI_FLOAT
# MPI_Op                         :   MPI_SUM
#
#
# List of Benchmarks to run:
# PingPong
#---------------------------------------------------
# Benchmarking PingPong
# #processes = 2
#---------------------------------------------------
       #bytes #repetitions      t[usec]   Mbytes/sec
            0         1000         2.23         0.00
            1         1000         2.26         0.42
            2         1000         2.26         0.85
            4         1000         2.26         1.69
            8         1000         2.26         3.38
           16         1000         2.36         6.45
           32         1000         2.57        11.89
           64         1000         2.36        25.81
          128         1000         2.64        46.19
          256         1000         2.73        89.30
          512         1000         3.09       157.99
         1024         1000         3.60       271.53
         2048         1000         4.46       437.57
         4096         1000         6.11       639.23
         8192         1000         7.49      1043.47
        16384         1000         9.76      1600.76
        32768         1000        14.98      2085.77
        65536          640        25.99      2405.08
       131072          320        50.68      2466.64
       262144          160        80.62      3101.01
       524288           80       145.86      3427.91
      1048576           40       279.06      3583.42
      2097152           20       543.37      3680.71
      4194304           10      1082.94      3693.63

# All processes entering MPI_Finalize

```

## ネットワーク トポロジに関する考慮事項

* Linux VM で、Eth1 は RDMA のネットワーク トラフィック用に予約されています。Eth1 設定や、このネットワークを参照する構成ファイル内の情報を変更しないでください。

* Azure では、IP over Infiniband (IB) はサポートされません。RDMA over IB のみがサポートされます。

* Linux VM で、Eth0 は Azure の通常のネットワーク トラフィック用に予約されています。

## 次のステップ

* Linux クラスター上で、Linux MPI アプリケーションのデプロイと実行を試します。

* Intel MPI のガイダンスについては、[Intel MPI Library のドキュメント](https://software.intel.com/ja-JP/articles/intel-mpi-library-documentation/)を参照してください。

<!---HONumber=AcomDC_0204_2016-->
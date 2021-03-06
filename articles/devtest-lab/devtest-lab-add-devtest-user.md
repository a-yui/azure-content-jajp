<properties
	pageTitle="DevTest ラボへの所有者とユーザーの追加 | Microsoft Azure"
	description="Azure DevTest ラボにサブスクリプション外のユーザーを安全に追加する方法を説明します。"
	services="devtest-lab,virtual-machines"
	documentationCenter="na"
	authors="tomarcher"
	manager="douge"
	editor=""/>

<tags
	ms.service="devtest-lab"
	ms.workload="na"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="02/18/2016"
	ms.author="tarcher"/>

# DevTest ラボへの所有者とユーザーの追加

> [AZURE.NOTE] この記事の付属のビデオを表示するには、次のリンクをクリックします。[How to set security in your DevTest Lab](/documentation/videos/how-to-set-security-in-your-devtest-lab) (DevTest ラボでセキュリティを設定する方法)

## 概要

DevTest ラボへのアクセスは、Azure のロール ベースのアクセス制御 (RBAC) で制御します。詳細については、Azure プレビュー ポータルの[ロール ベースのアクセス制御 (RBAC)](https://azure.microsoft.com/searchresults?query=Role%20Based%20Access%20Control%20%28RBAC%29) に関するページを参照してください。

DevTest ラボへのアクセスを許可するには 2 つのロールを使用します。

 - **所有者**: Azure サブスクリプション レベルで**所有者**ロールに割り当てられたユーザーは、管理機能や監視機能を含む、ラボへの完全なアクセス権を持ちます。

     > [AZURE.NOTE] サブスクリプション レベルではなく、RBAC レベルで割り当てられた**所有者**ロールは、DevTest ラボではサポートされていません。DevTest ラボ内でユーザーを**所有者**ロールに割り当てることはサポートされていません。

 -  **DevTest ラボ ユーザー**: **DevTest ラボ ユーザー** ロールに割り当てられたユーザーは、指定されたラボで VM を作成、更新、および削除することができます。ユーザーには、*内部* (サブスクリプションの Azure Active Directory のメンバー) と*外部* (パートナー組織のメンバーなど、Azure AD のメンバー以外のユーザー) の 2 種類があります。
	-  **DevTest ラボ ユーザー** ロールは、ラボの **[ユーザーの追加]** タイルを使用して割り当てる必要があります。
	-  **DevTest ラボ ユーザー** ロールのユーザーは、割り当てられたラボ内でのみ、これらの操作を実行できます。たとえば、**DevTest ラボ ユーザー**は、サブスクリプションの仮想マシン サービスを使用して仮想マシンを作成することはできません。仮想マシンの作成は、DevTest ラボ アカウントから行う場合のみ許可されます。
	- *外部*ユーザーは、Microsoft アカウント ドメイン (@hotmail.com、@live.com、@msn.com、@passport.com、@outlook.com、または特定の国の任意のバリアント) のいずれかのアカウントを持っている必要があります。

## ラボへの所有者の追加

DevTest ラボでは、ラボを含む Azure サブスクリプションの所有者がこれらのラボの所有者と見なされます。Azure プレビュー ポータルでラボのブレードを使用して DevTest ラボに他の所有者を追加できますが、これは現在サポートされていません。

既にラボを作成している、または新しいラボを作成する Azure サブスクリプションに所有者を追加するには、次の手順に従います。

1. [Azure プレビュー ポータル](https://portal.azure.com)にサインインします。

1. 左側のナビゲーションで、**[サブスクリプション]** をタップします。

	![Subscriptions link](./media/devtest-lab-add-devtest-user/subscriptions.png)
	
1. ラボを含めるサブスクリプションをタップします。

1. **[アクセス]** アイコンをタップします。

	![Access users](./media/devtest-lab-add-devtest-user/access-users.png)

1. **[ユーザー]** ブレードで、**[追加]** をタップします。

	![ユーザーの追加](./media/devtest-lab-add-devtest-user/devtest-users-blade.png)

1. **[ロールの選択]** ブレードで、**[所有者]** をタップします。

1. **[ユーザー]** ボックスに、所有者として追加するユーザーの電子メール アドレスを入力します。ユーザーが見つからない場合は、問題を示すエラー メッセージが表示されます。ユーザーが見つかった場合は、**[ユーザー]** ボックスの下に表示されます。

1. 見つかったユーザー名をタップします。

1. **[選択]** をタップします。

1. **[OK]** をタップして、**[アクセスを追加]** ブレードを閉じます。

1. **[ユーザー]** ブレードに戻ると、このユーザーが所有者として追加されています。これで、このユーザーはこのサブスクリプションで作成されたすべてのラボの所有者となり、所有者タスクを実行できるようになります。

## ラボへの DevTest ラボ ユーザーの追加

DevTest ラボ ユーザーをラボに追加するには、次の手順に従います。

1. [Azure プレビュー ポータル](https://portal.azure.com)にサインインします。

1. **[参照]** をタップします。

1. **[DevTest ラボ]** をタップします。

1. ラボの一覧で目的のラボをタップします。

1. **[アクセス]** アイコンをタップします。

	![User access](./media/devtest-lab-add-devtest-user/devtest-lab-home-blade.png)

1. **[ユーザー]** ブレードで、**[追加]** をタップします。

	![ユーザーの追加](./media/devtest-lab-add-devtest-user/devtest-users-blade.png)

1. **[ロールの選択]** ブレードで、**[DevTest ラボ ユーザー]** をタップします。

1. **[ユーザーの追加]** ブレードで、次の操作を実行します。

	1. **[ユーザーの追加]** ブレードに、組み込みのユーザーの一覧が表示されます。目的のユーザーが既に一覧にある場合は、そのユーザーの行をタップして選択するだけです。そのユーザーが選択されていることを示すチェックマークが左側に表示されます。複数のユーザーを選択するには、**Ctrl** キーを押しながら各ユーザーをクリックします。ユーザーの選択を解除するには、**Ctrl** キーを押しながら、選択解除するユーザーをクリックします。ブレードの下部のカウンターが、選択したユーザーの数を示します。

	1. 目的のユーザーが一覧にない場合は、**[ユーザー]** ボックスに有効な Microsoft 電子メール アカウントを入力します。電子メール アドレスが有効な場合は、**[ユーザー]** ボックスの下にそのユーザーが表示されます。そのユーザーをタップして選択します。

	1. ラボに追加するユーザーを選択したら、**[選択]** をタップします。

	1. **[OK]** をタップして、**[アクセスを追加]** ブレードを閉じます。

	1. **[ユーザー]** ブレードに、追加したロールとユーザーが表示されます。

<!---HONumber=AcomDC_0224_2016-->
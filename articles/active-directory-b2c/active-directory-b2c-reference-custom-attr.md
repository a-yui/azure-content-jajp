<properties
	pageTitle="Azure Active Directory B2C プレビュー: カスタム属性 | Microsoft Azure"
	description="Azure Active Directory B2C でカスタム属性を使用してコンシューマーに関する情報を収集する方法"
	services="active-directory-b2c"
	documentationCenter=""
	authors="swkrish"
	manager="msmbaldwin"
	editor="bryanla"/>

<tags
	ms.service="active-directory-b2c"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="01/04/2016"
	ms.author="swkrish"/>

#  Azure Active Directory B2C プレビュー: カスタム属性を使用してコンシューマーに関する情報を収集する

[AZURE.INCLUDE [active-directory-b2c-preview-note](../../includes/active-directory-b2c-preview-note.md)]

Azure Active Directory (Azure AD) B2C ディレクトリには、組み込みの情報セット (属性) が用意されています (名、姓、市区町村、郵便番号など)。ただし、どのようなコンシューマー向けアプリケーションにも、コンシューマーから収集する属性について固有の要件があります。Azure AD B2C では、各コンシューマー アカウントで保持される属性セットを拡張できます。以下で示すように、[Azure ポータル](https://portal.azure.com/)でカスタム属性を作成し、サインアップ ポリシーで使用できます。また、[Azure AD Graph API](active-directory-b2c-devquickstarts-graph-dotnet.md) を使用して属性を読み書きすることもできます。

> [AZURE.NOTE]
カスタム属性では、[Azure AD Graph API ディレクトリ スキーマ拡張機能](https://msdn.microsoft.com/library/azure/dn720459.aspx)が使用されています。

## カスタム属性を作成する

1. [この手順に従って、Azure ポータルで B2C 機能ブレードに移動します。](active-directory-b2c-app-registration.md#navigate-to-the-b2c-features-blade)
2. **[ユーザー属性]** をクリックします。
3. ブレードの上部にある **[+追加]** をクリックします。
4. **[名前]** にカスタム属性の名前を入力し (例: "ShoeSize")、必要に応じて **[説明]** を入力します。**[作成]** をクリックします。

    > [AZURE.NOTE]
    現時点では、"String" **データ型**のみを使用できます。

**[ユーザー属性]** の一覧およびサインアップ ポリシーで、カスタム属性を使用できるようになります。

## サインアップ ポリシーでカスタム属性を使用する

1. [この手順に従って、Azure ポータルで B2C 機能ブレードに移動します。](active-directory-b2c-app-registration.md#navigate-to-the-b2c-features-blade)
2. **[サインアップ ポリシー]** をクリックします。
3. クリックしてサインアップ ポリシーを開きます (例: "B2C\_1\_SiUp")。ブレードの上部にある **[編集]** をクリックします。
4. **[サインアップ属性]** をクリックして、カスタム属性 (例: "ShoeSize") を選択します。**[OK]** をクリックします。
5. **[アプリケーションの要求]** をクリックして、カスタム属性を選択します。**[OK]** をクリックします。
6. ブレードの上部にある **[保存]** をクリックします。

ポリシーで [今すぐ実行] 機能を使用して、コンシューマー エクスペリエンスを確認できます。コンシューマーのサインアップ時に収集される属性の一覧、およびアプリケーションに返送されるトークンに、"ShoeSize" が表示されるようになります。

<!---HONumber=AcomDC_0224_2016-->
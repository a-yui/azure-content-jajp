<properties 
	pageTitle="組織での Microsoft Passport for Work の有効化 | Microsoft Azure" 
	description="組織で Microsoft Passport を有効にするためのデプロイメント手順" 
	services="active-directory" 
	documentationCenter="" 
	authors="femila" 
	manager="stevenpo" 
	editor=""
	tags="azure-classic-portal"/>

<tags 
	ms.service="active-directory" 
	ms.workload="identity" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="11/19/2015" 
	ms.author="femila"/>

# 組織での Microsoft Passport for Work の有効化

Windows 10 ドメイン参加済みデバイスを Azure AD に接続したら、以下の手順を実行して組織で Microsoft Passport for Work を有効にします。

## System Center Configuration Manager Version 1509 for Technical Preview をデプロイする
Microsoft Passport のキーに基づいてユーザー証明書をデプロイするには、以下が必要です。

- **System Center Configuration Manager Version 1509 for Technical Preview**。詳細については、「[Microsoft System Center Configuration Manager Technical Preview](https://technet.microsoft.com/library/dn965439.aspx#BKMK_TP3Update)」および「[System Center Configuration Manager Team Blog (System Center Configuration Manager チーム ブログ)](http://blogs.technet.com/b/configmgrteam/archive/2015/09/23/now-available-update-for-system-center-config-manager-tp3.aspx)」をご覧ください。
- **PKI インフラストラクチャ**: ユーザー証明書を使用して Microsoft Passport for Work を有効にするには、PKI インフラストラクチャが準備されている必要があります。PKI インフラストラクチャがない場合や、ユーザー証明書のために PKI インフラストラクチャを使用したくない場合は、新しいバージョンの Windows Server のドメイン コントローラー (DC) をデプロイできます。
 - **新しいバージョンの Windows Server の DC のデプロイ**: 最新の Windows Server ビルド 10551 以降 (ISO は [Signiant Media Exchange](https://datatransfer.microsoft.com/signiant_media_exchange/spring/main?sdkAccessible=true) でダウンロード可能) で、[レプリカ DC を既存のドメインにインストールする](https://technet.microsoft.com/ja-JP/library/jj574134.aspx)手順に従うか、[新しい環境を作成する場合は新しい Active Directory フォレストをインストールする](https://technet.microsoft.com/ja-JP/library/jj574134.aspx)手順に従います。

## Active Directory のグループ ポリシーを使用して Microsoft Passport for Work を構成する

 Active Directory グループ ポリシーを使用して、Windows へのユーザー ログオン時に Microsoft Passport のユーザー資格情報をプロビジョニングするように、Windows 10 ドメイン参加済みデバイスを構成できます。

1. 	サーバー マネージャーを開き、**[ツール]** > **[グループ ポリシーの管理]** に移動します。
2.	[グループ ポリシーの管理] から、Azure AD 参加を有効にするドメインに対応するドメイン ノードに移動します。
3.	**[グループ ポリシー オブジェクト]** を右クリックし、**[新規]** を選択します。グループ ポリシー オブジェクトに **Enable Microsoft Passport** などの名前を付けます。**[OK]** をクリックします。
4.	新しいグループ ポリシー オブジェクトを右クリックし、**[編集]** を選択します。
5.	**[コンピューターの構成]** > **[ポリシー]** > **[管理用テンプレート]** > **[Windows コンポーネント]** > **[Passport for Work]** の順に移動します。
6.	**[Passport for Work の有効化]** を右クリックし、**[編集]** 選択します。
7.	**[有効]** を選択し、**[適用]** をクリックします。**[OK]** をクリックします。
8.	これで、グループ ポリシー オブジェクトを選択した場所にリンクできるようになりました。組織内のすべての Windows 10 ドメイン参加済みデバイスに対してこのポリシーを有効にするには、グループ ポリシーをドメインにリンクします。次に例を示します。
 - Windows 10 ドメイン参加済みコンピューターを配置する AD の 特定の組織単位 (OU)。
 - Azure AD に自動登録する Windows 10 ドメイン参加済みコンピューターを含む特定のセキュリティ グループ。

## Configuration Manager を使用した PowerShell のデプロイにより Microsoft Passport for Work を構成する 

次の PowerShell コマンドを実行します。

    powershell.exe -ExecutionPolicy Bypass -NoLogo -NoProfile -Command "& {New-ItemProperty "HKLM:\Software\Policies\Microsoft\PassportForWork" -Name "Enabled" -Value 1 -PropertyType "DWord" -Force}"

## Configuration Manager で "Passport for Work" の登録証明書を登録するように証明書プロファイルを構成する
Passport for Work 証明書ベースのログオンまたは Microsoft Hello を使用するには、スマート カード ログオン EKU を使用するテンプレートを選択して、証明書プロファイルを構成します ([資産とコンプライアンス] -> [コンプライアンス設定] -> [会社のリソースへのアクセス] -> [証明書プロファイル])。

##Passport for Work コンテナーが有効になっているときにトリガーされ、証明書の評価を要求する、スケジュールされたタスクをセットアップする
これは、Passport for Work コンテナーの作成をリッスンし、証明書の評価を要求するスケジュールされたタスクを管理者が作成する必要がある短期的な修正です。これにより、コンテナーと PIN をセットアップし、次回のログオン時に使用できるようにする際の遅延が短縮されます。

**スケジュールされたタスクを作成するには、次のコマンドを使用するか、UI の **schtasks /create /xml %0..<EnrollCertificate.xml> /tn <Task Name> を使用します。

サンプル xml を次に示します。

    <?xml version="1.0" encoding="UTF-16"?>
    <Task version="1.2" xmlns="http://schemas.microsoft.com/windows/2004/02/mit/task">
       <RegistrationInfo>
         <Date>2015-09-04T17:48:45.9973761</Date>
          <Author><DomainName/UserName></Author>
        <URI>\Enroll Certificates</URI>
      </RegistrationInfo>
      <Triggers>
        <EventTrigger>
          <Enabled>true</Enabled>
          <Subscription>&lt;QueryList&gt;&lt;Query Id="0" Path="Microsoft-Windows-User Device Registration/Admin"&gt;&lt;Select Path="Microsoft-Windows-User Device Registration/Admin"&gt;*[System[Provider[@Name='Microsoft-Windows-User Device Registration'] and EventID=300]]&lt;/Select&gt;&lt;/Query&gt;&lt;/QueryList&gt;</Subscription>
        </EventTrigger>
      </Triggers>
      <Principals>
      </Principals>
      <Settings>
        <MultipleInstancesPolicy>IgnoreNew</MultipleInstancesPolicy>
        <DisallowStartIfOnBatteries>false</DisallowStartIfOnBatteries>
        <StopIfGoingOnBatteries>true</StopIfGoingOnBatteries>
        <AllowHardTerminate>true</AllowHardTerminate>
        <StartWhenAvailable>true</StartWhenAvailable>
        <RunOnlyIfNetworkAvailable>false</RunOnlyIfNetworkAvailable>
        <IdleSettings>
          <StopOnIdleEnd>true</StopOnIdleEnd>
          <RestartOnIdle>false</RestartOnIdle>
        </IdleSettings>
        <AllowStartOnDemand>true</AllowStartOnDemand>
        <Enabled>true</Enabled>
        <Hidden>false</Hidden>
        <RunOnlyIfIdle>false</RunOnlyIfIdle>
        <WakeToRun>false</WakeToRun>
        <ExecutionTimeLimit>PT1H</ExecutionTimeLimit>
        <Priority>7</Priority>
        <RestartOnFailure>
          <Interval>PT1M</Interval>
          <Count>3</Count>
        </RestartOnFailure>
      </Settings>
      <Actions Context="Author">
        <Exec>
          <Command>wmic</Command>
          <Arguments>/namespace:\\root\ccm\dcm path SMS_DesiredConfiguration call TriggerEvaluation 1,0,"ScopeId_73F3BB5E-5EDC-4928-87BD-4E75EB4BBC34/ConfigurationPolicy_db89c51e-1d1b-4a17-8e42-a5459b742ccd",8</Arguments>
        </Exec>
      </Actions>
    </Task>

ScopeId\_73F3BB5E-5EDC-4928-87BD-4E75EB4BBC34/ConfigurationPolicy\_db89c51e-1d1b-4a17-8e42-a5459b742ccd は、「Configuration Manager で "Passport for Work" の登録証明書を登録するように証明書プロファイルを構成する」の手順で作成された証明書プロファイルの ID であり、<8> はバージョンです。

## 追加情報
* [Windows 10 for the enterprise: Ways to use devices for work (エンタープライズ向け Windows 10: デバイスを仕事に使用する方法)](active-directory-azureadjoin-windows10-devices-overview.md)
* [Azure Active Directory 参加を使用したクラウド機能の Windows 10 デバイスへの拡張](active-directory-azureadjoin-user-upgrade.md)
* [Microsoft Passport 経由でのパスワードを使用しない ID の認証](active-directory-azureadjoin-passport.md)
* [Azure AD 参加の使用シナリオについて](active-directory-azureadjoin-deployment-aadjoindirect.md)
* [Connect domain-joined devices to Azure AD for Windows 10 experiences (Windows 10 エクスペリエンスのためのドメイン参加済みデバイスの Azure AD への接続)](active-directory-azureadjoin-devices-group-policy.md)
* [Azure AD 参加の設定](active-directory-azureadjoin-setup.md)

<!---HONumber=AcomDC_1125_2015-->
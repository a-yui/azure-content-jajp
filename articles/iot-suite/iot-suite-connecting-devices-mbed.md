<properties
   pageTitle="mbed で C を使用してデバイスを接続する |Microsoft Azure"
   description="C で記述され、mbed デバイスで実行されるアプリケーションを使用して、デバイスを Azure IoT Suite 構成済みリモート監視ソリューションに接続する方法について説明します。"
   services=""
   suite="iot-suite"
   documentationCenter="na"
   authors="dominicbetts"
   manager="timlt"
   editor=""/>

<tags
   ms.service="iot-suite"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="02/05/2016"
   ms.author="dobett"/>


# デバイスを IoT Suite リモート監視構成済みソリューションに接続する

[AZURE.INCLUDE [iot-suite-selector-connecting](../../includes/iot-suite-selector-connecting.md)]

## mbed で C のサンプル ソリューションをビルドして実行する

次に、[mbed 対応の Freescale FRDM-K64F][lnk-mbed-home] デバイスをリモート監視ソリューションに接続する手順を説明します。

### デバイスをネットワークおよびデスクトップ コンピューターに接続する

1. イーサネット ケーブルを使用して mbed デバイスをネットワークに接続する方法を説明します。サンプル アプリケーションでは、インターネットへのアクセスが必要なため、この手順は必須です。

2. mbed デバイスをデスクトップ PC に接続する方法の詳細については、「[mbed の概要][lnk-mbed-getstarted]」を参照してください。

3. デスクトップ PC で Windows を実行している場合は、「[PC の構成][lnk-mbed-pcconnect]」を参照して、mbed デバイスへのシリアル ポート アクセスを構成します。

### mbed プロジェクトを作成してサンプル コードをインポートする

1. Web ブラウザーで、mbed.org の[開発者向けサイト](https://developer.mbed.org/)に移動します。サインアップしていない場合は、新しいアカウントを作成するオプションが表示されます (アカウントの作成は無料です)。既にサインアップしている場合は、アカウントの資格情報を使用してログインします。次に、ページの右上隅の **[Compiler]** をクリックします。これにより、ワークスペース管理インターフェイスが表示されます。

2. 使用しているハードウェア プラットフォームがウィンドウの右上隅に表示されていることを確認するか、右上隅にあるアイコンをクリックしてハードウェア プラットフォームを選択します。

3. メイン メニューの **[インポート]** をクリックします。次に、mbed の地球ロゴの横にある **[ここをクリック]** をクリックし、URL リンクからインポートします。

    ![][6]

4. ポップアップ ウィンドウで、サンプル コード用のリンク https://developer.mbed.org/users/AzureIoTClient/code/remote_monitoring/ を入力し、**[インポート]** をクリックします。

    ![][7]

5. mbed コンパイラのウィンドウでは、このプロジェクトをインポートしたことでさまざまなライブラリがインポートされたことを確認できます。ライブラリには、Azure IoT チームが提供および管理するライブラリ ([azureiot\_common](https://developer.mbed.org/users/AzureIoTClient/code/azureiot_common/)、[iothub\_client](https://developer.mbed.org/users/AzureIoTClient/code/iothub_client/)、[iothub\_amqp\_transport](https://developer.mbed.org/users/AzureIoTClient/code/iothub_amqp_transport/)、[proton-c-mbed](https://developer.mbed.org/users/AzureIoTClient/code/proton-c-mbed/)) もあれば、mbed ライブラリ カタログで入手可能なサード パーティのライブラリもあります。

    ![][8]

6. remote\_monitoring\\remote\_monitoring.c ファイルを開き、ファイル内で次のコードを見つけます。

    ```
    static const char* deviceId = "[Device Id]";
    static const char* deviceKey = "[Device Key]";
    static const char* hubName = "[IoTHub Name]";
    static const char* hubSuffix = "[IoTHub Suffix, i.e. azure-devices.net]";
    ```

7. [Device Id] と [Device Key] を自身のデバイス データに置き換えます。IoT Hub ホスト名を使用して、プレースホルダーの [IoTHub Name] と [IoTHub Suffix, i.e. azure-devices.net] を置き換えます。たとえば、IoT Hub ホスト名が contoso.azure-devices.net である場合は、contoso が **hubName**、残りの部分が **hubSuffix** になります。

    ```
    static const char* deviceId = "mydevice";
    static const char* deviceKey = "mykey";
    static const char* hubName = "contoso";
    static const char* hubSuffix = "azure-devices.net";
    ```

    ![][9]

### プログラムをビルドして実行する

1. **[コンパイル]** をクリックしてプログラムをビルドします。警告は無視してかまいません。ただし、ビルドでエラーが発生する場合は、続行する前にそのエラーを修正してください。

2. ビルドが成功した場合、mbed コンパイラの Web サイトによって、プロジェクト名の付いた .bin ファイルが生成され、ローカル コンピューターにダウンロードされます。.bin ファイルをデバイスにコピーします。.bin ファイルをデバイスに保存すると、デバイスが再起動し、.bin ファイルに含まれるプログラムが実行されます。mbed デバイスのリセット ボタンを押すことで、いつでもプログラムを手動で再起動することができます。

3. PuTTY などの SSH クライアント アプリケーションを使用して、デバイスに接続します。Windows デバイス マネージャーを確認すると、デバイスで使用されているシリアル ポートを特定できます。

    ![][11]

4. PuTTY で、接続タイプとして **[シリアル]** をクリックします。デバイスは通常、115200 ボーで接続するため、**[速度]** ボックスに「115200」と入力します。その後、**[開く]** をクリックします。

5. プログラムの実行が開始されます。接続時にプログラムが自動的に開始されない場合は、ボードのリセットが必要になることがあります (Ctrl キーを押しながら Break キーを押すか、ボードのリセット ボタンを押します)。

    ![][10]

[AZURE.INCLUDE [iot-suite-visualize-connecting](../../includes/iot-suite-visualize-connecting.md)]


[6]: ./media/iot-suite-connecting-devices-mbed/mbed1.png
[7]: ./media/iot-suite-connecting-devices-mbed/mbed2a.png
[8]: ./media/iot-suite-connecting-devices-mbed/mbed3a.png
[9]: ./media/iot-suite-connecting-devices-mbed/suite6.png
[10]: ./media/iot-suite-connecting-devices-mbed/putty.png
[11]: ./media/iot-suite-connecting-devices-mbed/mbed6.png

[lnk-mbed-home]: https://developer.mbed.org/platforms/FRDM-K64F/
[lnk-mbed-getstarted]: https://developer.mbed.org/platforms/FRDM-K64F/#getting-started-with-mbed
[lnk-mbed-pcconnect]: https://developer.mbed.org/platforms/FRDM-K64F/#pc-configuration

<!---HONumber=AcomDC_0218_2016-->
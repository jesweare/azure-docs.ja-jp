---
title: クイック スタート:Azure IoT からデバイスを制御する (Node.js)
description: このクイック スタートでは、2 つのサンプル Node.js アプリケーションを実行します。 1 つのアプリケーションは、ハブに接続されたデバイスをリモートで制御できるバックエンド アプリケーションです。 もう 1 つのアプリケーションは、ハブに接続されたリモートで制御できるデバイスをシミュレートします。
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.devlang: nodejs
ms.topic: quickstart
ms.custom: mvc, seo-javascript-september2019, seo-javascript-october2019
ms.date: 06/21/2019
ms.openlocfilehash: cc0ff372b4680f02410f1f58304d0be95228c965
ms.sourcegitcommit: 9add86fb5cc19edf0b8cd2f42aeea5772511810c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/09/2020
ms.locfileid: "77108913"
---
# <a name="quickstart-use-nodejs-to-control-a-device-connected-to-an-azure-iot-hub"></a>クイック スタート:Node.js を使用して、Azure IoT ハブに接続されたデバイスを制御する

[!INCLUDE [iot-hub-quickstarts-2-selector](../../includes/iot-hub-quickstarts-2-selector.md)]

IoT Hub は、クラウドから IoT デバイスを管理し、大量のデバイス テレメトリを格納または処理のためにクラウドに取り込むことができるようにする Azure サービスです。 このクイック スタートでは、"*ダイレクト メソッド*" を使って、IoT ハブに接続されているシミュレートされたデバイスを制御します。 ダイレクト メソッドを使うと、IoT ハブに接続されたデバイスの動作をリモートで変更できます。

このクイック スタートでは、あらかじめ作成されている次の 2 つの Node.js アプリケーションを使います。

* シミュレートされたデバイス アプリケーションは、バックエンド アプリケーションから呼び出されたダイレクト メソッドに応答します。 ダイレクト メソッドの呼び出しを受け取るため、このアプリケーションは IoT ハブ上のデバイス固有のエンドポイントに接続します。

* バックエンド アプリケーションは、シミュレートされたデバイスでダイレクト メソッドを呼び出します。 デバイスでダイレクト メソッドを呼び出すため、このアプリケーションは IoT ハブ上のサービス側エンドポイントに接続します。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Azure サブスクリプションがない場合は、開始する前に[無料アカウント](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)を作成してください。

## <a name="prerequisites"></a>前提条件

このクイック スタートで実行する 2 つのサンプル アプリケーションは、Node.js を使って書かれています。 開発用コンピューター上に Node.js v10.x.x 以降が必要です。

複数のプラットフォームに対応する Node.js を [nodejs.org](https://nodejs.org) からダウンロードできます。

開発コンピューターに現在インストールされている Node.js のバージョンは、次のコマンドを使って確認できます。

```cmd/sh
node --version
```

次のコマンドを実行して、Microsoft Azure IoT Extension for Azure CLI を Cloud Shell インスタンスに追加します。 IoT Hub、IoT Edge、IoT Device Provisioning Service (DPS) 固有のコマンドが Azure CLI に追加されます。

```azurecli-interactive
az extension add --name azure-cli-iot-ext
```

まだ行っていない場合は、 https://github.com/Azure-Samples/azure-iot-samples-node/archive/master.zip からサンプル Node.js プロジェクトをダウンロードし、ZIP アーカイブを抽出します。

ポート 8883 がファイアウォールで開放されていることを確認してください。 このクイックスタートのデバイス サンプルでは、ポート 8883 を介して通信する MQTT プロトコルを使用しています。 このポートは、企業や教育用のネットワーク環境によってはブロックされている場合があります。 この問題の詳細と対処方法については、「[IoT Hub への接続 (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub)」を参照してください。

## <a name="create-an-iot-hub"></a>IoT Hub の作成

前出の[デバイスから IoT ハブへの利用統計情報の送信に関するクイック スタート](quickstart-send-telemetry-node.md)を完了した場合は、この手順を省略できます。

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-device"></a>デバイスの登録

前出の[デバイスから IoT ハブへの利用統計情報の送信に関するクイック スタート](quickstart-send-telemetry-node.md)を完了した場合は、この手順を省略できます。

デバイスを IoT Hub に接続するには、あらかじめ IoT Hub に登録しておく必要があります。 このクイック スタートでは、Azure Cloud Shell を使用して、シミュレートされたデバイスを登録します。

1. Azure Cloud Shell で次のコマンドを実行してデバイス ID を作成します。

   **YourIoTHubName**: このプレースホルダーは、実際の IoT Hub に対して選んだ名前に置き換えてください。

   **MyNodeDevice**: これは、登録するデバイスの名前です。 示されているように、**MyNodeDevice** を使用することをお勧めします。 デバイスに別の名前を選択した場合は、この記事全体でその名前を使用する必要があります。また、サンプル アプリケーションを実行する前に、アプリケーション内のデバイス名を更新してください。

    ```azurecli-interactive
    az iot hub device-identity create \
      --hub-name {YourIoTHubName} --device-id MyNodeDevice
    ```

2. Azure Cloud Shell で次のコマンドを実行して、登録したデバイスの "_デバイス接続文字列_" を取得します。

    **YourIoTHubName**: このプレースホルダーは、実際の IoT Hub に対して選んだ名前に置き換えてください。

    ```azurecli-interactive
    az iot hub device-identity show-connection-string \
      --hub-name {YourIoTHubName} \
      --device-id MyNodeDevice \
      --output table
    ```

    次のようなデバイス接続文字列をメモしておきます。

   `HostName={YourIoTHubName}.azure-devices.net;DeviceId=MyNodeDevice;SharedAccessKey={YourSharedAccessKey}`

    この値は、このクイック スタートの後の方で使います。

3. また、バックエンド アプリケーションが IoT ハブに接続してメッセージを取得できるようにするには、"_サービス接続文字列_" が必要です。 次のコマンドを実行すると、IoT ハブのサービス接続文字列が取得されます。

    **YourIoTHubName**: このプレースホルダーは、実際の IoT Hub に対して選んだ名前に置き換えてください。

    ```azurecli-interactive
    az iot hub show-connection-string \
      --policy-name service --name {YourIoTHubName} --output table

    ```

    次のようなサービス接続文字列をメモしておきます。

   `HostName={YourIoTHubName}.azure-devices.net;SharedAccessKeyName=service;SharedAccessKey={YourSharedAccessKey}`

    この値は、このクイック スタートの後の方で使います。 このサービス接続文字列は、前の手順でメモしたデバイス接続文字列とは異なります。

## <a name="listen-for-direct-method-calls"></a>ダイレクト メソッドの呼び出しをリッスンする

シミュレートされたデバイス アプリケーションは、IoT ハブ上のデバイス固有エンドポイントに接続し、シミュレートされた利用統計情報を送信して、ハブからのダイレクト メソッド呼び出しをリッスンします。 このクイック スタートでは、ハブからのダイレクト メソッド呼び出しは、利用統計情報の送信間隔を変更するようデバイスに指示します。 シミュレートされたデバイスでは、ダイレクト メソッドを実行した後、ハブに受信確認が返送されます。

1. ローカル ターミナル ウィンドウで、サンプルの Node.js プロジェクトのルート フォルダーに移動します。 次に、**iot-hub\Quickstarts\simulated-device-2** フォルダーに移動します。

2. 適当なテキスト エディターで **SimulatedDevice.js** ファイルを開きます。

    `connectionString` 変数の値を、前にメモしたデバイス接続文字列に置き換えます。 その後、変更を **SimulatedDevice.js** に保存します。

3. ローカル ターミナル ウィンドウで次のコマンドを実行して、必要なライブラリをインストールし、シミュレートされたデバイス アプリケーションを実行します。

    ```cmd/sh
    npm install
    node SimulatedDevice.js
    ```

    次のスクリーンショットは、シミュレートされたデバイス アプリケーションが IoT Hub にテレメトリを送信したときの出力を示しています。

    ![シミュレートされたデバイスを実行する](./media/quickstart-control-device-node/simulated-device-telemetry-iot-hub.png)

## <a name="call-the-direct-method"></a>ダイレクト メソッドを呼び出す

バックエンド アプリケーションは、IoT ハブ上のサービス側エンドポイントに接続します。 アプリケーションにより、IoT ハブを通してデバイスへのダイレクト メソッド呼び出しが行われた後、受信確認がリッスンされます。 通常、IoT Hub のバックエンド アプリケーションはクラウドで実行されます。

1. 別のローカル ターミナル ウィンドウで、サンプルの Node.js プロジェクトのルート フォルダーに移動します。 その後、**iot-hub\Quickstarts\back-end-application** フォルダーに移動します。

2. 適当なテキスト エディターで **BackEndApplication.js** ファイルを開きます。

    `connectionString` 変数の値を、前にメモしたサービス接続文字列に置き換えます。 変更を **BackEndApplication.js** に保存します。

3. ローカル ターミナル ウィンドウで次のコマンドを実行して、必要なライブラリをインストールし、バックエンド アプリケーションを実行します。

    ```cmd/sh
    npm install
    node BackEndApplication.js
    ```

    次のスクリーンショットは、アプリケーションによりデバイスに対してダイレクト メソッド呼び出しが行われ、受信確認が受診されたときの出力を示します。

    ![アプリケーションがデバイスに対してダイレクト メソッドの呼び出しを行うときの出力](./media/quickstart-control-device-node/direct-method-device-call.png)

    バックエンド アプリケーションを実行した後、シミュレートされたデバイスを実行しているコンソール ウィンドウにメッセージが表示され、メッセージの送信速度が変わります。

    ![シミュレートされたクライアントに変更があったときの出力](./media/quickstart-control-device-node/simulated-device-message-change.png)

## <a name="clean-up-resources"></a>リソースをクリーンアップする

[!INCLUDE [iot-hub-quickstarts-clean-up-resources](../../includes/iot-hub-quickstarts-clean-up-resources.md)]

## <a name="next-steps"></a>次のステップ

このクイック スタートでは、バックエンド アプリケーションからデバイス上のダイレクト メソッドを呼び出し、シミュレートされたデバイス アプリケーションでダイレクト メソッド呼び出しに応答しました。

デバイスからクラウドへのメッセージをクラウド内の異なる宛先にルーティングする方法を学習するには、次のチュートリアルに進んでください。

> [!div class="nextstepaction"]
> [チュートリアル:処理のために利用統計情報を異なるエンドポイントにルーティングする](tutorial-routing.md)

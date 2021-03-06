---
title: Application Gateway の Azure Monitor メトリック
description: メトリックを使用してアプリケーション ゲートウェイのパフォーマンスを監視する方法について説明します
services: application-gateway
author: abshamsft
ms.service: application-gateway
ms.topic: article
ms.date: 2/5/2019
ms.author: absha
ms.openlocfilehash: 1fa9c72f7ca305a03cdc90ea02cefe973932792b
ms.sourcegitcommit: 57669c5ae1abdb6bac3b1e816ea822e3dbf5b3e1
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/06/2020
ms.locfileid: "77046313"
---
# <a name="metrics-for-application-gateway"></a>Application Gateway のメトリック

Application Gateway からは、ご利用の Application Gateway インスタンスおよびバックエンド インスタンスのパフォーマンスに関する、メトリックと呼ばれるデータ ポイントが [Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/overview) に発行されます。 これらのメトリックは、順序付けられた時系列データ セットに含まれる数値であり、特定の時間におけるご利用のアプリケーション ゲートウェイの一部の側面を記述しています。 Application Gateway を通過する要求がある場合は、それによって 60 秒間隔でメトリックが測定されて送信されます。 Application Gateway を通過する要求がない場合、またはメトリックのデータがない場合、メトリックは報告されません。 詳細については、「[Azure Monitor metrics](https://docs.microsoft.com/azure/azure-monitor/platform/data-platform-metrics)」(Azure Monitor メトリック) を参照してください。

## <a name="metrics-supported-by-application-gateway-v2-sku"></a>Application Gateway V2 SKU でサポートされるメトリック

### <a name="timing-metrics"></a>タイミングのメトリック

Application Gateway には、要求と応答に関連するいくつかのタイミング メトリックが組み込まれています。これらはすべて、ミリ秒単位で測定されます。 

![](./media/application-gateway-metrics/application-gateway-metrics.png)

> [!NOTE]
>
> Application Gateway に複数のリスナーがある場合は、意味のある推論を得るために、異なる待機時間メトリックを比較しながら、常に *[リスナー]* ディメンションでフィルター処理します。

- **バックエンド接続時間**

  バックエンド アプリケーションとの接続を確立するために費やされた時間です。 

  これには、新しい接続を確立するためにバックエンド サーバーの TCP スタックが要した時間だけでなく、ネットワーク待機時間も含まれます。 SSL の場合、これにはハンドシェイクに費やされた時間も含まれます。 

- **バックエンド先頭バイト応答時間**

  バックエンド サーバーへの接続の確立を開始してから応答ヘッダーの最初のバイトを受信するまでの時間間隔です。 

  これにより、Application Gateway からバックエンドへのアクセス要求にかかった時間、バックエンド アプリケーションが応答するまでの時間 (サーバーがコンテンツを生成し、その後データベース クエリをフェッチするためにかかった時間)、および応答の先頭バイトがバックエンドから Application Gateway に到着するまでにかかった時間の合計である*バックエンド接続時間*を概算します。

- **バックエンド最終バイト応答時間**

  バックエンド サーバーへの接続の確立を開始してから応答本文の最後のバイトを受信するまでの時間間隔です。 

  これにより、*バックエンド先頭バイト応答時間*とデータ転送時間の合計を概算します (この数は、要求されたオブジェクトのサイズとサーバー ネットワークの待機時間によって大きく異なる場合があります)。

- **アプリケーション ゲートウェイの合計時間**

  要求の受信、処理、および応答の送信にかかった平均時間。 

  これは、Application Gateway が HTTP 要求の最初のバイトを受信してから、最後の応答バイトがクライアントに送信されるまでの間隔です。 これには、Application Gateway によって行われた処理時間、*バックエンド最終バイト応答時間*、Application Gateway がすべての応答と*クライアント RTT*を送信するのにかかった時間が含まれます。

- **クライアント RTT**

  クライアントと Application Gateway の間の平均ラウンドトリップ時間。



これらのメトリックを使用することで、観測された速度低下の原因が、クライアント ネットワーク、Application Gateway のパフォーマンス、バックエンド ネットワークとバックエンド サーバーの TCP スタックの飽和、バックエンド アプリケーションのパフォーマンス、または大きなファイル サイズの内どれかを判断できます。

たとえば、*バックエンド先頭バイト応答時間*に急増傾向が見られる一方、*バックエンド接続時間*が安定傾向を見せている場合、Application Gateway からバックエンドへの待機時間と、接続の確立にかかった時間が安定しており、急増はバックエンド アプリケーションの応答時間の増加が原因であると推測できます。 一方、*バックエンド先頭バイト応答時間*の急増が*バックエンド接続時間*に対応する急増に関連付けられている場合は、Application Gateway とバックエンドサーバーの間のネットワーク、またはバックエンド サーバーの TCP スタックが飽和状態になっていることが推測できます。 

*バックエンド最終バイト応答時間*が急増しているにもかかわらず、*バックエンド先頭バイト応答時間*が安定している場合は、この急増は要求されているファイルが大きいことが原因であると推測できます。

同様に、*Application Gateway の合計時間*が急増しているにもかかわらず、*バックエンド最終バイト応答時間*が安定している場合は、Application Gateway でのパフォーマンスのボトルネック、またはクライアントと Application Gateway 間のネットワークのボトルネックのいずれかの兆候である場合があります。 さらに、*クライアント RTT* に対応する急増も見られる場合、この低下はクライアントと Application Gateway 間のネットワークが原因であることを示します。

### <a name="application-gateway-metrics"></a>Application Gateway メトリック

Application Gateway に関しては、次のメトリックを利用できます。

- **受信バイト数**

   Application Gateway がクライアントから受信したバイト数です。

- **送信バイト数**

   Application Gateway からクライアントに送信されたバイト数です。

- **クライアント TLS プロトコル**

   Application Gateway との接続を確立したクライアントによって開始された TLS 要求と非 TLS 要求の数です。 TLS プロトコルの分布を表示するには、ディメンションの TLS プロトコルでフィルター処理します。

- **現在の容量ユニット**

   使用された容量ユニットの数です。 容量ユニットでは、固定費に追加して課金される使用量ベースの費用が測定されます。 容量ユニットには、コンピューティング ユニット、永続的接続、スループットの 3 つの決定項があります。 各容量ユニットは最大で次のものにより構成されます: 1 コンピューティング ユニット、または 2500 永続的接続、または 2.22 Mbps のスループット。

- **現在のコンピューティング ユニット**

   消費されたプロセッサ容量の数です。 コンピューティング ユニットに影響する要因は、1 秒あたりの TLS 接続数、URL 書き換え計算、WAF ルールの処理です。 

- **現在の接続数**

   クライアントから Application Gateway に対してアクティブになっているコンカレント接続の合計数

- **失敗した要求**

   Application Gateway が処理した失敗した要求の数です。 要求の数をさらにフィルター処理することで、各々のまたは特定のバックエンド プール http 設定の組み合わせごとに数を表示できます。


- **応答の状態**

   Application Gateway によって返される HTTP 応答の状態です。 応答状態コードの分布をさらに分類し、2xx、3xx、4xx、5xx のカテゴリで応答を表示できます。

- **スループット**

   Application Gateway が処理した 1 秒あたりのバイト数

- **要求の合計数**

   Application Gateway が処理した成功した要求の数です。 要求の数をさらにフィルター処理することで、各々のまたは特定のバックエンド プール http 設定の組み合わせごとに数を表示できます。

- **Web アプリケーション ファイアウォールの一致ルール**

- **Web アプリケーション ファイアウォールのトリガー ルール**

### <a name="backend-metrics"></a>バックエンド メトリック

Application Gateway に関しては、次のメトリックを利用できます。

- **バックエンド応答の状態**

  バックエンドによって返された HTTP 応答状態コードの数です。 これには、Application Gateway によって生成された応答コードは含まれません。 応答状態コードの分布をさらに分類し、2xx、3xx、4xx、5xx のカテゴリで応答を表示できます。

- **正常なホストの数**

  正常性プローブによって正常であると判定されたバックエンドの数です。 バックエンド プールごとにフィルター処理を行って、特定のバックエンド プールの正常なホストの数を表示できます。

- **異常なホストの数**

  正常性プローブによって異常であると判定されたバックエンドの数です。 バックエンド プールごとにフィルター処理を行って、特定のバックエンド プールの異常なホストの数を表示できます。
  
- **正常なホストごとの 1 分あたりの要求**は、バックエンド プール内の正常なメンバーが 1 分間に受信した要求の平均数です。 *BackendPool HttpSettings* ディメンションを使用してバックエンド プールを指定する必要があります。  
  

## <a name="metrics-supported-by-application-gateway-v1-sku"></a>Application Gateway V1 SKU でサポートされるメトリック

### <a name="application-gateway-metrics"></a>Application Gateway メトリック

Application Gateway に関しては、次のメトリックを利用できます。

- **CPU 使用率**

  Application Gateway に割り当てられた CPU の使用率を表示します。  通常の状況では、CPU 使用率が 90% を超えることは望ましくありません。これは Application Gateway の背後でホストされている Web サイトで待機時間が発生し、クライアント エクスペリエンスを乱す可能性があるためです。 インスタンス数を増やすか、SKU のサイズを大きくするか、またはその両方を行うことによって、Application Gateway の構成を変更し、CPU 使用率を間接的に制御または向上させることができます。

- **現在の接続数**

  Application Gateway で確立された現在の接続の数

- **失敗した要求**

  Application Gateway が処理した失敗した要求の数です。 要求の数をさらにフィルター処理することで、各々のまたは特定のバックエンド プール http 設定の組み合わせごとに数を表示できます。

- **応答の状態**

  Application Gateway によって返される HTTP 応答の状態です。 応答状態コードの分布をさらに分類し、2xx、3xx、4xx、5xx のカテゴリで応答を表示できます。

- **スループット**

  Application Gateway が処理した 1 秒あたりのバイト数

- **要求の合計数**

  Application Gateway が処理した成功した要求の数です。 要求の数をさらにフィルター処理することで、各々のまたは特定のバックエンド プール http 設定の組み合わせごとに数を表示できます。

- **Web アプリケーション ファイアウォールの一致ルール**

- **Web アプリケーション ファイアウォールのトリガー ルール**

### <a name="backend-metrics"></a>バックエンド メトリック

Application Gateway に関しては、次のメトリックを利用できます。

- **正常なホストの数**

  正常性プローブによって正常であると判定されたバックエンドの数です。 バックエンド プールごとにフィルター処理を行って、特定のバックエンド プールの正常なホストの数を表示できます。

- **異常なホストの数**

  正常性プローブによって異常であると判定されたバックエンドの数です。 バックエンド プールごとにフィルター処理を行って、特定のバックエンド プールの異常なホストの数を表示できます。

## <a name="metrics-visualization"></a>メトリックの視覚化

アプリケーション ゲートウェイに移動して **[監視]** の **[メトリック]** を選択します。 利用できる値を表示するには、 **[メトリック]** ドロップダウン リストを選択します。

次の画像では、最後の 30 分間に表示された 3 つのメトリックの例を確認できます。

[![](media/application-gateway-diagnostics/figure5.png "Metric view")](media/application-gateway-diagnostics/figure5-lb.png#lightbox)

現在のメトリックの一覧を確認するには、「[Azure Monitor のサポートされるメトリック](../azure-monitor/platform/metrics-supported.md)」を参照してください。

### <a name="alert-rules-on-metrics"></a>メトリックに基づくアラート ルール

リソースのメトリックに基づいてアラート ルールを開始できます。 たとえば、Application Gateway のスループットが指定した期間にしきい値を上回るか下回る場合、またはしきい値に等しい場合、アラートで webhook を呼び出したり、管理者に電子メールを送信したりできます。

次の例では、スループットのしきい値を超えた後に管理者に電子メールを送信するアラート ルールの作成手順を説明します。

1. **[メトリック アラートの追加]** を選択して **[ルールの追加]** ページを開きます。 このページには、メトリック ページからもアクセスできます。

   ![[メトリック アラートの追加] ボタン][6]

2. **[ルールの追加]** ページで、名前、条件、通知の各セクションに入力して、 **[OK]** を選択します。

   * **[条件]** セレクターに入力できるのは、 **[より大きい]** 、 **[以上]** 、 **[未満]** 、または **[以下]** の 4 つの値です。

   * **[期間]** セレクターで、5 分から 6 時間までの期間を選択します。

   * **[メールの所有者、投稿者、閲覧者]** を選択すると、そのリソースのアクセス権を持つユーザーに基づいて電子メールを動的に送信できます。 それ以外の場合は、ユーザーのコンマ区切りの一覧を **[追加の管理者メール]** ボックスに指定できます。

   ![[ルールの追加] ページ][7]

しきい値を超えた場合、次の図のような電子メールが送信されます。

![しきい値違反の電子メール][8]

メトリック アラートを作成すると、アラートの一覧が表示されます。 この一覧には、すべてのアラート ルールの概要が示されます。

![アラートとルールの一覧][9]

アラート通知の詳細については、「[Microsoft Azure のアラートの概要](../monitoring-and-diagnostics/insights-receive-alert-notifications.md)」を参照してください。

webhook の詳細および webhook とアラートを使用する方法については、「[Azure メトリック アラートでの webhook の構成](../azure-monitor/platform/alerts-webhooks.md)」を参照してください。

## <a name="next-steps"></a>次のステップ

* [Azure Monitor ログ](../azure-monitor/insights/azure-networking-analytics.md)を使用して、カウンターとイベント ログを視覚化します。
* [Power BI を使用した Azure アクティビティ ログの視覚化](https://blogs.msdn.com/b/powerbi/archive/2015/09/30/monitor-azure-audit-logs-with-power-bi.aspx)に関するブログ
* [Power BI などでの Azure アクティビティ ログの表示と分析](https://azure.microsoft.com/blog/analyze-azure-audit-logs-in-powerbi-more/)に関するブログ

[1]: ./media/application-gateway-diagnostics/figure1.png
[2]: ./media/application-gateway-diagnostics/figure2.png
[3]: ./media/application-gateway-diagnostics/figure3.png
[4]: ./media/application-gateway-diagnostics/figure4.png
[5]: ./media/application-gateway-diagnostics/figure5.png
[6]: ./media/application-gateway-diagnostics/figure6.png
[7]: ./media/application-gateway-diagnostics/figure7.png
[8]: ./media/application-gateway-diagnostics/figure8.png
[9]: ./media/application-gateway-diagnostics/figure9.png
[10]: ./media/application-gateway-diagnostics/figure10.png

---
title: V3 機械学習エンティティに移行する
titleSuffix: Azure Cognitive Services
description: V3 作成には新しいエンティティ型である機械学習エンティティが用意されているほか、アプリケーションの機械学習エンティティやその他のエンティティ、または機能にリレーションシップを追加することができます。
services: cognitive-services
author: diberry
manager: nitinme
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 12/30/2019
ms.author: diberry
ms.openlocfilehash: b5dbcd9033d9a41e43ea907d043e0c0486b236db
ms.sourcegitcommit: 5925df3bcc362c8463b76af3f57c254148ac63e3
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/31/2019
ms.locfileid: "75563823"
---
# <a name="migrate-to-v3-authoring-entity"></a>V3 作成エンティティに移行する

V3 作成には新しいエンティティ型である機械学習エンティティが用意されているほか、アプリケーションの機械学習エンティティやその他のエンティティ、または機能にリレーションシップを追加することができます。

## <a name="entities-are-decomposable-in-v3"></a>V3 ではエンティティが分解可能

[API](https://westeurope.dev.cognitive.microsoft.com/docs/services/luis-programmatic-apis-v3-0-preview) または[プレビュー ポータル](https://preview.luis.ai/)のいずれかを使って V3 作成 API で作成されたエンティティを使用すると、親と子が含まれる多層エンティティ モデルを構築できます。 親は**機械学習エンティティ**、子は機械学習エンティティの**サブコンポーネント**として知られています。

各サブコンポーネントも機械学習エンティティですが、制約と記述子の構成オプションが追加されています。

* **制約**は正確なテキスト マッチング ルールで、これによりルールと一致したエンティティが確実に抽出されます。 このルールは、正確なテキスト マッチング エンティティ、つまり現時点では[事前構築済みエンティティ](luis-reference-prebuilt-entities.md)、[正規表現エンティティ](reference-entity-regular-expression.md)、または[リスト エンティティ](reference-entity-list.md)によって定義されます。
* **記述子**はフレーズ リストやエンティティなどの[機能](luis-concept-feature.md)で、エンティティを厳密に示すために使用されます。

V3 作成には新しいエンティティ型である機械学習エンティティが用意されているほか、アプリケーションの機械学習エンティティやその他のエンティティ、または機能にリレーションシップを追加することができます。

## <a name="how-do-these-new-relationships-compare-to-v2-authoring"></a>これらの新しいリレーションシップと V2 作成の比較

V2 作成には、階層エンティティと複合エンティティ、およびこの同じタスクを実行するための役割と機能が用意されていました。 エンティティ、機能、および役割が明示的かつ相互に関連していなかったため、予測中、LUIS が示すリレーションシップを理解するのが困難でした。

V3 では、リレーションシップは明示的で、アプリの作成者によって設計されています。 これによりアプリの作成者は以下を行うことができます。

* 発話の例で、LUIS がこれらのリレーションシップをどのように予測しているかを視覚的に確認する
* [対話型のテスト ウィンドウ](luis-interactive-test.md)またはエンドポイントで、これらのリレーションシップをテストする
* これらのリレーションシップを、適切に構造化された入れ子になった名前付き [json オブジェクト](reference-entity-machine-learned-entity.md)を介して、クライアント アプリケーションで使用する

## <a name="planning"></a>計画

移行するときは、次の移行プランを考慮してください。

* LUIS アプリをバックアップし、別のアプリで移行を実行します。 V2 と V3 のアプリを同時に使用できるようにすると、必要な変更と、予測結果への影響を検証することができます。
* 現在の予測成功メトリックをキャプチャします
* 現在のダッシュボード情報を、アプリの状態のスナップショットとしてキャプチャします
* 既存の意図、エンティティ、フレーズ リスト、パターン、およびバッチ テストを確認します
* 次の要素は、**変更せずに**移行できます。
    * 意図
    * [エンティティ]
        * 正規表現エンティティ
        * リスト エンティティ
    * [機能]
        * フレーズ リスト
* 次の要素は、**変更せずに**移行する必要があります。
    * [エンティティ]
        * 階層構造エンティティ
        * 複合エンティティ
    * ロール - ロールは機械学習 (親) エンティティにのみ適用できます。 サブコンポーネントには適用できません
    * 階層エンティティと複合エンティティが使用されるバッチ テストとパターン

移行プランを設計するときは、すべての階層エンティティと複合エンティティが移行された後に、最終的な機械学習エンティティをレビューする時間を確保してください。 直接的な移行が動作している間、変更を加えてバッチ テストの結果と予測 JSON をレビューした後、より統一した JSON によって変更を行うことになる可能性があります。この場合、クライアント側アプリに配信される最終的な情報は、異なる方法で整理されます。 これはコードのリファクタリングに似ており、組織のレビュー プロセスで処理する必要があります。

V2 モデルに対してバッチ テストを適用していない場合、移行の一環としてバッチ テストを V3 モデルに移行すると、その移行が、エンドポイントの予測結果にどのように影響するかを検証できなくなります。

## <a name="migrating-from-v2-entities"></a>V2 エンティティからの移行

V3 作成モデルへの移行を開始するとき、機械学習エンティティとそのサブコンポーネント (制約と記述子を含む) に移行する方法を検討する必要があります。

次の表は、V2 から V3 エンティティ設計に移行する必要があるエンティティを示しています。

|V2 作成エンティティ型|V3 作成エンティティ型|例|
|--|--|--|
|複合エンティティ|機械学習エンティティ|[詳細情報](#migrate-v2-composite-entity)|
|階層構造エンティティ|機械学習エンティティの役割|[詳細情報](#migrate-v2-hierarchical-entity)|

## <a name="migrate-v2-composite-entity"></a>V2 複合エンティティを移行する

V2 複合の子それぞれを、V3 機械学習エンティティのサブコンポーネントで表す必要があります。 複合の子が事前構築済み、正規表現、またはリスト エンティティの場合、これは、子を表すサブコンポーネントで**制約**として適用する必要があります。

機械学習エンティティへの複合エンティティの移行を計画する際の考慮事項:
* 子エンティティはパターンでは使用できません
* 子エンティティは共有されなくなりました
* 以前は機械学習以外の子エンティティだった場合、そのエンティティにはラベルを付ける必要があります

### <a name="existing-descriptors"></a>既存の記述子

複合エンティティ内のワードをブーストするためのフレーズ リストは、機械学習 (親) エンティティ、サブコンポーネント (子) エンティティ、意図 (フレーズ リストが 1 つの意図にのみ適用される場合) のいずれかに、記述子として適用する必要があります。 記述子は、最も大きくブーストする必要があるエンティティに追加するようにしてください。 記述子を追加することでサブコンポーネント (子) の予測が最も大幅にブーストする場合は、一般的に、機械学習 (親) エンティティには記述子を追加しないでください。

### <a name="new-descriptors"></a>新しい記述子

V3 作成で、エンティティを評価するための計画手順を、すべてのエンティティと意図の使用可能な記述子として追加します。

### <a name="example-entity"></a>エンティティの例

このエンティティは一例です。 ご自身のエンティティ移行では、他の考慮事項が必要になる場合があります。

以下が使用されているピザ `order` を変更する V2 複合について考えてみましょう。
* 配達時刻のための事前構築済み datetimeV2
* ピザ、パイ、生地、トッピングなどの特定のワードをブーストするフレーズ リスト
* マッシュルーム、オリーブ、ペペロニなどのトッピングを検出するリスト エンティティ。

このエンティティの発話の例を次に示します。

`Change the toppings on my pie to mushrooms and delivery it 30 minutes later`

次の表は移行を示しています。

|V2 モデル|V3 モデル|
|--|--|
|親 - `Order` という名前のコンポーネント エンティティ|親 - `Order` という名前の機械学習エンティティ|
|子 - 事前構築済み datetimeV2|* 事前構築済みエンティティを新しいアプリに移行します。<br>* 事前構築済み datetimeV2 の親で制約を追加します。|
|子 - トッピングのリスト エンティティ|* リスト エンティティを新しいアプリに移行します。<br>* 次に、リスト エンティティの親で制約を追加します。|


## <a name="migrate-v2-hierarchical-entity"></a>V2 階層構造エンティティを移行する

V2 作成では、LUIS に存在する役割の前に階層構造エンティティが提供されていました。 両方とも目的は、コンテキスト使用に基づいてエンティティを抽出するというものでした。 階層構造エンティティがある場合、そのエンティティは、役割が含まれる単純なエンティティと考えることができます。

V3 作成:
* 機械学習 (親) エンティティで役割を適用できます。
* サブコンポーネントに役割を適用できません。

このエンティティは一例です。 ご自身のエンティティ移行では、他の考慮事項が必要になる場合があります。

次のピザ `order` を変更するための V2 階層構造エンティティを考えてみましょう。
* それぞれの子によって元のトッピングか最終的なトッピングかが判断される状況

このエンティティの発話の例を次に示します。

`Change the topping from mushrooms to olives`

次の表は移行を示しています。

|V2 モデル|V3 モデル|
|--|--|
|親 - `Order` という名前のコンポーネント エンティティ|親 - `Order` という名前の機械学習エンティティ|
|子 - ピザの元のトッピングと最終的なトッピングが含まれる階層構造エンティティ|* 各トッピングに対して役割を `Order` に追加します。|

## <a name="next-steps"></a>次のステップ

* [開発者向けリソース](developer-reference-resource.md)

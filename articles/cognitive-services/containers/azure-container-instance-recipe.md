---
title: Azure コンテナー インスタンスのレシピ
titleSuffix: Azure Cognitive Services
description: Azure コンテナー インスタンスに Cognitive Services コンテナーをデプロイする方法について説明します
services: cognitive-services
author: IEvangelist
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.topic: conceptual
ms.date: 01/23/2020
ms.author: dapine
ms.openlocfilehash: 78f35042678aa7c30cebf73796df3e5d564b4502
ms.sourcegitcommit: f52ce6052c795035763dbba6de0b50ec17d7cd1d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/24/2020
ms.locfileid: "76717004"
---
# <a name="deploy-and-run-container-on-azure-container-instance"></a>Azure コンテナー インスタンスにコンテナーをデプロイして実行する

次の手順に従えば、Azure [Container Instances](https://docs.microsoft.com/azure/container-instances/) を使用してクラウド内の Azure Cognitive Services アプリケーションを簡単にスケーリングすることができます。 コンテナー化により、インフラストラクチャを管理することにではなく、アプリケーションの構築に集中することができます。 コンテナーの使用方法の詳細については、「[機能とメリット](../cognitive-services-container-support.md#features-and-benefits)」を参照してください。

## <a name="prerequisites"></a>前提条件

レシピは、任意の Cognitive Services コンテナーで機能します。 このレシピを使用する前に、Azure portal で Cognitive Service リソースを作成する必要があります。 コンテナーをサポートする各 Cognitive Service には、コンテナーのサービスをインストールして構成するための "インストール方法" のドキュメントがあります。 一部のサービスにはコンテナーへの入力としてファイルまたは一連のファイルが必要です。このソリューションを使用する前に、コンテナーを正しく理解して使用していることが重要です。

* Azure portal で作成された Cognitive Service リソース。
* Cognitive Service **エンドポイント URL** - Azure portal 内からエンドポイント URL が表示されている場所と正しい URL の形式の例を確認する方法については、お使いのサービスのコンテナーの "インストール方法" を確認してください。 正確な形式はサービスによって異なる可能性があります。
* Cognitive Service **キー** - キーは Azure リソースの **[キー]** ページにあります。 必要なのは 2 つのキーのうち 1 つだけです。 キーは 32 文字の英数字の文字列です。
* ローカル ホスト (コンピューター) 上の 1 つの Cognitive Services コンテナー。 以下を実行できることを確認します。
  * `docker pull` コマンドでイメージをプルする。
  * `docker run` コマンドを使用し、必要なすべての構成設定を使用してローカル コンテナーを正常に実行する。
  * コンテナーのエンドポイントを呼び出し、HTTP 2xx の応答と JSON 応答が返される。

山かっこ `<>` 内のすべての変数は、ご自分の値に置き換える必要があります。 この置き換えには山かっこも含まれます。

[!INCLUDE [Create a Text Analytics Containers on Azure Container Instances](includes/create-container-instances-resource.md)]

## <a name="use-the-container-instance"></a>コンテナー インスタンスを使用する

1. **[概要]** を選択し、IP アドレスをコピーします。 これは `55.55.55.55` のような数値の IP アドレスです。
1. 新しいブラウザー タブを開き、`http://<IP-address>:5000 (http://55.55.55.55:5000` のような IP アドレスを使用します。 コンテナーのホーム ページが表示され、コンテナーが実行中であることが示されます。

1. **[Service API Description]\(サービス API の説明\)** を選択し、コンテナーの Swagger ページを表示します。

1. いずれかの **POST** API を選択して **[試してみる]** を選択します。入力を含むパラメーターが表示されます。 パラメーターを入力します。

1. **[実行]** を選択して、要求をコンテナー インスタンスに送信します。

    Azure コンテナー インスタンスへの Cognitive Services コンテナーの作成と使用は以上で完了です。

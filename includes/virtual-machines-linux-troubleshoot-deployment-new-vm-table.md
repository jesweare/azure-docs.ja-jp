---
author: cynthn
ms.service: virtual-machines-linux
ms.topic: include
ms.date: 10/26/2018
ms.author: cynthn
ms.openlocfilehash: d93de4ed758afb5e951bb5e19f4f7adb290e461c
ms.sourcegitcommit: 3e98da33c41a7bbd724f644ce7dedee169eb5028
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/18/2019
ms.locfileid: "67181178"
---
次の表に、Linux の一般化された OS イメージ (gen.) と特殊化された OS イメージ (spec.) のアップロードとキャプチャの考えられる組み合わせを示します。 エラーなしで処理される組み合わせは Y で示され、エラーがスローされる組み合わせは N で示されています。発生するさまざまなエラーの原因と解決策が表の下に記載されています。

| OS | spec. のアップロード | gen. のアップロード | spec. のキャプチャ | gen. のキャプチャ |
| --- | --- | --- | --- | --- |
| Linux gen. |N<sup>1</sup> |Y |N<sup>3</sup> |Y |
| Linux spec. |Y |N<sup>2</sup> |Y |N<sup>4</sup> |


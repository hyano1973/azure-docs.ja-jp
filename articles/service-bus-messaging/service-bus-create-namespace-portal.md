---
title: "Azure Portal で Service Bus 名前空間を作成する方法 | Microsoft Docs"
description: "Azure Portal を使用して Service Bus 名前空間を作成する"
services: service-bus-messaging
documentationcenter: .net
author: sethmanheim
manager: timlt
editor: 
ms.assetid: fbb10e62-b133-4851-9d27-40bd844db3ba
ms.service: service-bus-messaging
ms.devlang: tbd
ms.topic: get-started-article
ms.tgt_pltfrm: dotnet
ms.workload: na
ms.date: 10/16/2017
ms.author: sethm
ms.openlocfilehash: 38c79495683f0aa619247d486bfb8d843dfdfdc0
ms.sourcegitcommit: 9ae92168678610f97ed466206063ec658261b195
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/17/2017
---
# <a name="create-a-service-bus-namespace-using-the-azure-portal"></a>Azure Portal を使用して Service Bus 名前空間を作成する

名前空間は、すべてのメッセージング コンポーネントのスコープ コンテナーです。 複数のキューとトピックを 1 つの名前空間に格納できます。多くの場合、名前空間はアプリケーション コンテナーとして機能します。 Service Bus 名前空間の作成には 2 つの方法があります。

1. Azure ポータル (この記事)
2. [リソース マネージャーのテンプレート][create-namespace-using-arm]

## <a name="create-a-namespace-in-the-azure-portal"></a>Azure Portal での名前空間の作成

[!INCLUDE [service-bus-create-namespace-portal](../../includes/service-bus-create-namespace-portal.md)]

ご利用ありがとうございます。 これで、Service Bus メッセージング名前空間が作成されました。

## <a name="next-steps"></a>次のステップ

Service Bus メッセージングのさらに高度な機能を紹介するサンプルについては、[GitHub サンプル][github-samples]を参照してください。

[create-namespace-using-arm]: service-bus-resource-manager-overview.md
[github-samples]: https://github.com/Azure/azure-service-bus/tree/master/samples

---
title: "Azure Application Insights でユーザー フローを使用してユーザーのナビゲーション パターンを分析する | Microsoft docs"
description: "ユーザーが Web アプリのページ間および機能間をどのように移動しているかを分析します。"
services: application-insights
documentationcenter: 
author: numberbycolors
manager: carmonm
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: multiple
ms.topic: article
ms.date: 08/02/2017
ms.author: cfreeman
ms.openlocfilehash: d17ed3dff08f00a1d6a2108608e42b29f95fbd84
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/11/2017
---
# <a name="analyze-user-navigation-patterns-with-user-flows-in-application-insights"></a>Application Insights でユーザー フローを使用してユーザーのナビゲーション パターンを分析する

![Application Insights ユーザー フロー ツール](./media/app-insights-usage-flows/flows.png)

ユーザー フロー ツールは、ユーザーがサイトのページ間および機能間をどのように移動しているかを目で見てわかるようにします。 次のような疑問の答えを得るのに役立ちます。
* ユーザーは対象サイトのページから他のサイトにどのように移動しているのか
* ユーザーは対象サイトのページで何をクリックしているのか
* 対象サイト内でユーザーが他のサイトに移動するのが最も多い場所はどこか
* ユーザーが同じ操作を何回も繰り返している場所があるか

ユーザー フロー ツールは、指定された最初のページ ビューまたはイベントから開始します。 このページ ビューまたはカスタム イベントから始めて、ユーザー フローは、ユーザーがセッションにおいて次に送信したページ ビューやカスタム イベント、次の次に送信したページ ビューやカスタム イベント、といった具合に順番に表示します。 太さが異なる線は、ユーザーが各パスを通過した回数を示します。 特別な [Session Ended]\(セッション終了\) ノードは、前のノードの後でページ ビューまたはカスタム イベントを送信しなかったユーザーの数を示し、おそらくユーザーが対象サイトを離れた場所を表します。



> [!NOTE]
> ユーザー フロー ツールを使うには、Application Insights のリソースにページ ビューやカスタム イベントが含まれる必要があります。 [アプリをセットアップし、Application Insights JavaScript SDK を使用してページ ビューを自動的に収集する方法について説明します](app-insights-javascript.md)。
> 
> 

## <a name="start-by-choosing-an-initial-page-view-or-custom-event"></a>最初のページ ビューまたはカスタム イベントを選択して開始する

![ユーザー フローの最初のイベントを選ぶ](./media/app-insights-usage-flows/flows-initial-event.png)

ユーザー フロー ツールを使って前に示したような疑問への回答を得るには、まず、表示の起点となる最初のページ ビューやカスタム イベントを選びます。
1. [What do users do after...?]\(... の後でユーザーは何を行うか?\) タイトルのリンクをクリックするか、 [Edit]\(編集\) ボタンをクリックします。 
2. [Initial event]\(最初のイベント\) ボックスの一覧から、ページ ビューまたはカスタム イベントを選びます。
3. [Create graph]\(グラフの作成\) をクリックします。

視覚化の [Step 1]\(ステップ 1\) 列には、ユーザーが最初のイベントの直後に最も頻繁に行ったことが、上から頻度の高い順に表示されます。 [Step 2]\(ステップ 2\) 以降の列には、それ以降にユーザーが行ったことが表示されて、ユーザーがサイト内を移動したすべての経路が図示されます。

既定では、ユーザー フロー ツールはサイトから過去 24 時間だけのページ ビューとカスタム イベントをランダムにサンプリングします。 [Edit]\(編集\) メニューで、時間範囲を広げたり、ランダム サンプリングのパフォーマンスと精度のバランスを変更したりできます。

ページ ビューやカスタム イベントに関係のないものが含まれる場合は、ノードの [X] をクリックして非表示にできます。 非表示にするノードを選んだ後は、視覚化の下の [Create graph]\(グラフの作成\) ボタンをクリックします。 非表示にしたノードをすべて確認するには、[Edit]\(編集\) ボタンをクリックし、[Excluded events]\(除外されたイベント\) セクションを見ます。

視覚化に表示する必要があるページ ビューまたはカスタム イベントが表示されていない場合は、次のようにします。
* [Edit]\(編集\) メニューの [Excluded events]\(除外されたイベント\) セクションを確認します。
* [Edit]\(編集\) メニューの [Detail level]\(詳細レベル\) コントロールを使って、頻度の低いイベントを視覚化に含めます。
* 予想されるページ ビューまたはカスタム イベントがユーザーによってあまり送信されない場合は、[Edit]\(編集\) メニューで視覚化の時間範囲を広げてみます。
* 予想されるページ ビューまたはカスタム イベントが、サイトのソース コードで Application Insights SDK によって収集されるように設定されていることを確認します。 [カスタム イベントの収集について詳しくは、こちらをご覧ください。](app-insights-api-custom-events-metrics.md)

視覚化に表示されるステップの数を増やす場合は、[Edit]\(編集\) メニューの [Number of steps]\(ステップ数\) スライダーを使います。

## <a name="after-visiting-a-page-or-feature-where-do-users-go-and-what-do-they-click"></a>ページまたは機能にアクセスした後で、ユーザーが移動する先とクリックするもの

![ユーザー フローを使ってユーザーがクリックする場所を理解する](./media/app-insights-usage-flows/flows-one-step.png)

最初のイベントがページ ビューである場合、視覚化の最初の列 ([Step 1]\(ステップ 1\)) を見ると、ユーザーがページにアクセスした直後に行ったことが簡単にわかります。 ユーザー フローの視覚化の横にあるウィンドウでサイトを開いてみてください。 ユーザーによるページの操作方法の予想と、[Step 1]\(ステップ 1\) 列のイベントの一覧を比較します。 多くの場合、チームにとってはあまり重要でないと思われるページの UI 要素が、ページで最も使われているものに含まれます。 これは、サイトのデザイン改善のよい出発点になる場合があります。

最初のイベントがカスタム イベントの場合、最初の列にはユーザーがそのアクションを実行した直後に行ったことが示されます。 ページ ビューと同様に、観察されたユーザーの行動が、チームの目標や予想と一致するかどうかを検討します。 選んだ最初のイベントが [Added Item to Shopping Cart]\(買い物かごに商品を追加した\) である場合、そのすぐ後に [Go to Checkout]\(会計に進む\) および [Completed Purchase]\(購入を完了した\) が表示されているかどうかを確認します。 ユーザーの行動が予想と大きく異なる場合は、視覚化を使って、サイトの現在のデザインのどこにユーザーの行動が "阻害される" 原因があるかを理解します。

## <a name="where-are-the-places-that-users-churn-most-from-your-site"></a>対象サイト内でユーザーが他のサイトに移動するのが最も多い場所はどこか

視覚化の上位に表示される [Session Ended]\(セッション終了\) ノード (特にフローの早い段階) を調べます。 これは、多くのユーザーがそこに至るまでのページ パスと UI 操作の後でサイトから離れた可能性があることを意味します。 予期される離脱もありますが (電子商取引サイトで購入を完了した後など)、通常、サイトからの離脱は、デザイン上の問題、パフォーマンスの悪さ、またはサイトに関するその他の改善可能な問題を示しています。

[Session Ended]\(セッション終了\) ノードはその Application Insights リソースによって収集されたテレメトリのみに基づくことに注意してください。 Application Insights が特定のユーザー操作に対するテレメトリを受け取らない場合、ユーザーはユーザー フロー ツールがセッション終了を示した後もまだサイトを操作している可能性があります。

## <a name="are-there-places-where-users-repeat-the-same-action-over-and-over"></a>ユーザーが同じ操作を何回も繰り返している場所があるか

視覚化の後続ステップで多くのユーザーによって繰り返されているページ ビューやカスタム イベントを探します。 これは通常、ユーザーがサイトで反復的なアクションを実行していることを意味します。 繰り返しが見つかった場合は、サイトのデザインの変更または繰り返しを減らす新しい機能の追加を検討します。 たとえば、ユーザーがテーブル要素の行ごとに反復的アクションを実行していることがわかった場合は、一括編集機能を追加します。

## <a name="common-questions"></a>一般的な質問

### <a name="why-do-steps-appear-disconnected"></a>ステップが切り離されて表示されるのはなぜですか?

![切り離されたステップのあるユーザー フロー](./media/app-insights-usage-flows/flows-disconnected.png)

ユーザー フローの視覚化のステップ (列) が切り離されている場合は、ユーザーがそのステップ間のパスを移動した頻度が表示に十分なほど多くなかったことを意味します。 ステップが接続されるように低頻度のイベントを視覚化に表示するには、[Edit]\(編集\) メニューの [Detail level]\(詳細レベル\) スライダーを調整します。

### <a name="does-the-initial-event-represent-the-first-time-the-event-appears-in-a-session-or-any-time-it-appears-in-a-session"></a>最初のイベントは、そのイベントがセッション内で出現する最初の時を表しますか、それともセッション内の任意の時を表しますか?

視覚化の最初のイベントは、ユーザーがセッションの間にそのページ ビューまたはカスタム イベントを初めて送信した時のみを表します。 ユーザーがセッション内で最初のイベントを何回も送信できる場合、[Step 1]\(ステップ 1\) 列は、最初のイベントのすべてのインスタンスではなく、最初のイベントの "*最初の*" インスタンスの後でユーザーが行ったことだけを示します。

## <a name="next-steps"></a>次のステップ

* [利用状況の概要](app-insights-usage-overview.md)
* [ユーザー、セッション、およびイベント](app-insights-usage-segmentation.md)
* [保持](app-insights-usage-retention.md)
* [アプリにカスタム イベントを追加する](app-insights-api-custom-events-metrics.md)
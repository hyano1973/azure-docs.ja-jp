---
title: "Azure HDInsight でデータ視覚化ツールを使用する Spark BI | Microsoft Docs"
description: "HDInsight クラスター上で Apache Spark BI を使用して分析用のデータ視覚化ツールを使用する"
keywords: "apache spark bi,spark bi, spark データ視覚化, spark ビジネス インテリジェンス"
services: hdinsight
documentationcenter: 
author: mumian
manager: cgronlun
editor: cgronlun
tags: azure-portal
ms.assetid: 1448b536-9bc8-46bc-bbc6-d7001623642a
ms.service: hdinsight
ms.custom: hdinsightactive,hdiseo17may2017
ms.workload: big-data
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/29/2017
ms.author: jgao
ms.openlocfilehash: 074415ba50ecdb1799093a3ead3bdd22fd02cc15
ms.sourcegitcommit: 9890483687a2b28860ec179f5fd0a292cdf11d22
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/24/2018
---
# <a name="apache-spark-bi-using-data-visualization-tools-with-azure-hdinsight"></a>Azure HDInsight のデータ視覚化ツールを使用する Apache Spark BI

[Microsoft Power BI](http://powerbi.microsoft.com) と [Tableau](http://www.tableau.com) を使用して、Azure HDInsight の Apache Spark クラスター内のデータを視覚化する方法について説明します。

## <a name="prerequisites"></a>前提条件

* **記事「[HDInsight で Spark クラスターに対して対話型クエリを実行する](./apache-spark-load-data-run-query.md)」を完了する**。
* **Power BI**: [Power BI Desktop](https://powerbi.microsoft.com/en-us/desktop/) と [Power BI 試用版サブスクリプション](https://app.powerbi.com/signupredirect?pbi_source=web) (省略可能)。
* **Tableau**: [Tableau Desktop](http://www.tableau.com/products/desktop) と [Microsoft Spark ODBC ドライバー](http://go.microsoft.com/fwlink/?LinkId=616229)。


## <a name="hivetable"></a>データを検証する

[前のチュートリアル](apache-spark-load-data-run-query.md)で作成した Jupyter Notebook には、`hvac` テーブルを作成するコードが含まれています。 このテーブルは、**\HdiSamples\HdiSamples\SensorSampleData\hvac\hvac.csv** のすべての HDInsight Spark クラスターで使用可能な CSV ファイルに基づいています。 データの検証に使用する手順は、以下のとおりです。

1. Jupyter Notebook から次のコードを貼り付けて、**Shift + Enter** キーを押します。 このコードによってテーブルの存在が検証されます。

    ```PySpark
    %%sql
    SHOW TABLES
    ```

    出力は次のようになります。

    ![Spark でのテーブルの表示](./media/apache-spark-use-bi-tools/show-tables.png)

    このチュートリアルを開始する前に Notebook を閉じると、`hvactemptable` はクリーンアップされるため、出力に含まれません。
    BI ツールからアクセスできるのは、metastore に保存された Hive テーブル (**isTemporary** 列に **False** と表示される) のみです。 このチュートリアルでは、作成した **hvac** テーブルに接続します。

2. 次のコードを空のセルに貼り付け、**Shift + Enter** キーを押します。 このコードによってテーブル内のデータが検証されます。

    ```PySpark
    %%sql
    SELECT * FROM hvac LIMIT 10
    ```

    出力は次のようになります。

    ![Spark での hvac テーブルの行の表示](./media/apache-spark-use-bi-tools/select-limit.png)

3. Notebook の **[ファイル]** メニューの **[Close and Halt]\(閉じて停止\)** をクリックします。 Notebook をシャットダウンしてリソースを解放します。 















## <a name="powerbi"></a>Power BI の使用

このセクションでは、Power BI を使用して Spark クラスター データから視覚エフェクト、レポート、およびダッシュボードを作成します。 

### <a name="create-a-report-in-power-bi-desktop"></a>Power BI Desktop でレポートを作成する
Spark を操作する最初のステップでは、Power BI Desktop のクラスターに接続し、クラスターからデータを読み込み、そのデータを基に基本的な視覚エフェクトを作成します。

> [!NOTE]
> この記事で説明するコネクタは、現在プレビューの段階です。 お客様のフィードバックを [Power BI コミュニティ](https://community.powerbi.com/) サイトや [Power BI Ideas](https://ideas.powerbi.com/forums/265200-power-bi-ideas) を通じてお寄せください。

1. [Power BI Desktop](https://powerbi.microsoft.com/en-us/desktop/) を開きます。
1. **[ホーム]** タブで、**[データの取得]**、**[詳細]** の順にクリックします。

    ![HDInsight Apache Spark から Power BI Desktop にデータを取得](./media/apache-spark-use-bi-tools/hdinsight-spark-power-bi-desktop-get-data.png "Apache Spark BI から Power BI にデータを取得")


2. 検索ボックスに「`Spark`」と入力し、**[Azure HDInsight Spark (ベータ版)]** を選択して、**[接続]** をクリックします。

    ![Apache Spark BI から Power BI にデータを取得](./media/apache-spark-use-bi-tools/apache-spark-bi-import-data-power-bi.png "Apache Spark BI から Power BI にデータを取得")

3. クラスターの URL (`mysparkcluster.azurehdinsight.net` の形式) を入力し、**[DirectQuery]** を選択して **[OK]** をクリックします。

    Spark では、いずれかのデータ接続モードをご利用いただけます。 DirectQuery を使用する場合、データセット全体を更新しなくても変更がレポートに反映されます。 データをインポートする場合は、データセットを更新して変更内容を確認する必要があります。 DirectQuery をいつ、どのように使用するかの詳細については、「[Power BI で DirectQuery を使用する](https://powerbi.microsoft.com/documentation/powerbi-desktop-directquery-about/)」をご覧ください。 

4. HDInsight のログイン アカウント情報を入力し、**[接続]** をクリックします。 既定のアカウント名は *admin* です。

5. `hvac` テーブルを選択して、データのプレビューが表示されるのを待ち、**[読み込み]** をクリックします。

    ![Spark クラスターのユーザー名とパスワード](./media/apache-spark-use-bi-tools/apache-spark-bi-select-table.png "Spark クラスターのユーザー名とパスワード")

    Spark クラスターに接続し、`hvac` テーブルからデータを読み込むために必要な情報が Power BI Desktop に整いました。 **[フィールド]** ウィンドウにテーブルとその列が表示されます。  次のスクリーンショットをご覧ください。

6. 各ビルの目標温度と実際の温度の差を視覚化します。 

    1. **[視覚エフェクト]** ウィンドウで **[面グラフ]** を選択します。 
    2. **[BuildingID]** フィールドを**軸**にドラッグし、**[ActualTemp]** と **[TargetTemp]** の各フィールドを**値**にドラッグします。

        ![Apache Spark BI を使用して Spark データ視覚化を作成](./media/apache-spark-use-bi-tools/apache-spark-bi-add-value-columns.png "Apache Spark BI を使用して Spark データ視覚化を作成")

        図は次のようになります。

        ![Apache Spark BI を使用して Spark データ視覚化を作成](./media/apache-spark-use-bi-tools/apache-spark-bi-area-graph.png "Apache Spark BI を使用して Spark データ視覚化を作成")

        既定では、**ActualTemp** および **TargetTemp** の合計が表示されます。 [視覚化] ウィンドウで **ActualTemp** と **TragetTemp** の隣にある下矢印をクリックすると、**[合計]** が選択されていることを確認できます。

    3. [視覚化] ウィンドウで **ActualTemp** と **TragetTemp** の隣にある下矢印をクリックし、**[平均]** を選択して、各ビルの実際の温度および目標温度の平均を取得します。

        ![Apache Spark BI を使用して Spark データ視覚化を作成](./media/apache-spark-use-bi-tools/apache-spark-bi-average-of-values.png "Apache Spark BI を使用して Spark データ視覚化を作成")

        次のスクリーンショットのようにデータが視覚化されます。 グラフの上にカーソルを移動すると、関連データを含むツール ヒントが表示されます。

        ![Apache Spark BI を使用して Spark データ視覚化を作成](./media/apache-spark-use-bi-tools/apache-spark-bi-area-graph-sum.png "Apache Spark BI を使用して Spark データ視覚化を作成")

7. **[ファイル]**、**[保存]** の順にクリックして、ファイル名 `BuildingTemperature.pbix` を入力します。 

### <a name="publish-the-report-to-the-power-bi-service-optional"></a>Power BI サービスにレポートを発行する (省略可能)

Power BI サービスを使用すると、組織全体でレポートとダッシュボードを共有できます。 このセクションでは、まずデータセットとレポートを発行します。 次に、このレポートをダッシュボードにピン留めします。 通常、ダッシュボードは、レポート内のデータのサブセットにフォーカスするために使用します。レポート内にある視覚エフェクトは 1 つのみですが、それでも手順を実行するのに役立ちます。

1. Power BI Desktop を開きます。
2. **[ホーム]** タブで **[発行]** をクリックします。

    ![Power BI Desktop から発行](./media/apache-spark-use-bi-tools/apache-spark-bi-publish.png "Power BI Desktop から発行")

2. データセットを発行してレポートするワークスペースを選択して、**[選択]** をクリックします。 次の図では、既定値の **[マイ ワークスペース]** が選択されています。

    ![データセットを発行してレポートするワークスペースを選択する](./media/apache-spark-use-bi-tools/apache-spark-bi-select-workspace.png "データセットを発行してレポートするワークスペースを選択する") 

3. 発行に成功したら、**[Open 'BuildingTemperature.pbix' in Power BI]\('BuildingTemperature.pbix' を Power BI で開く\)** をクリックします。

    ![発行の成功、クリックして資格情報を入力](./media/apache-spark-use-bi-tools/apache-spark-bi-publish-success.png "発行の成功、クリックして資格情報を入力") 

4. Power BI サービスで、**[資格情報を入力する]** をクリックします。

    ![Power BI サービスで資格情報を入力する](./media/apache-spark-use-bi-tools/apache-spark-bi-enter-credentials.png "Power BI サービスで資格情報を入力する")

5. **[資格情報を編集]** をクリックします。

    ![Power BI サービスで資格情報を編集する](./media/apache-spark-use-bi-tools/apache-spark-bi-edit-credentials.png "Power BI サービスで資格情報を編集する")

6. HDInsight のログイン アカウント情報を入力し、**[サインイン]** をクリックします。 既定のアカウント名は *admin* です。

    ![Spark クラスターへのサインイン](./media/apache-spark-use-bi-tools/apache-spark-bi-sign-in.png "Spark クラスターへのサインイン")

7. 左ウィンドウ枠で **[ワークスペース]** > **[マイ ワークスペース]** > **[レポート]** の順に移動して、**[BuildingTemperature]** をクリックします。

    ![左ウィンドウ枠の [レポート] に表示されるレポート](./media/apache-spark-use-bi-tools/apache-spark-bi-service-left-pane.png "左ウィンドウ枠の [レポート] に表示されるレポート")

    左ウィンドウ枠の **[データセット]** にも **[BuildingTemperature]** が表示されます。

    これで、Power BI Desktop で作成したビジュアルが Power BI サービスで使用できるようになりました。 

8. 視覚エフェクトにカーソルを合わせ、右上隅のピン アイコンをクリックします。

    ![Power BI サービスのレポート](./media/apache-spark-use-bi-tools/apache-spark-bi-service-report.png "Power BI サービスのレポート")

9. [新しいダッシュボード] を選択して、名前 `Building temperature` を入力し、**[ピン留め]** をクリックします。

    ![新しいダッシュボードにピン留めする](./media/apache-spark-use-bi-tools/apache-spark-bi-pin-dashboard.png "新しいダッシュボードにピン留めする")

10. レポートで、**[ダッシュボードへ移動]** をクリックします。 

ビジュアルはダッシュボードにピン留めされます。他のビジュアルをレポートに追加して、同じダッシュボードにピン留めすることもできます。 レポートとダッシュボードの詳細については、「[Power BI のレポート](https://powerbi.microsoft.com/documentation/powerbi-service-reports/)」および [Power BI のダッシュボード](https://powerbi.microsoft.com/documentation/powerbi-service-dashboards/)に関する記事をご覧ください。

## <a name="tableau"></a>Tableau Desktop を使用する 

> [!NOTE]
> このセクションは、Azure HDInsight で作成された Spark 1.5.2 クラスターにのみ適用できます。
>
>

1. この Apache Spark BI チュートリアルを行っているコンピューターに [Tableau Desktop](http://www.tableau.com/products/desktop) をインストールします。

2. そのコンピューターに Microsoft Spark ODBC ドライバーもインストールされていることを確認します。 ドライバーは [ここ](http://go.microsoft.com/fwlink/?LinkId=616229)からインストールできます。

1. Tableau Desktop を起動します。 左側のウィンドウの接続先サーバー一覧で **[Spark SQL]**をクリックします。 Spark SQL が既定で左側のウィンドウに表示されない場合は、 **[その他のサーバー]**をクリックして検索できます。
2. Spark SQL 接続ダイアログ ボックスに、次のスクリーンショットのように値を指定して **[OK]** をクリックします。

    ![Apache Spark BI のクラスターに接続](./media/apache-spark-use-bi-tools/connect-to-tableau-apache-spark-bi.png "Apache Spark BI のクラスターに接続")

    **Microsoft Spark ODBC** ドライバーをコンピューターにインストールしてある場合にのみ、認証ドロップダウンに [[Microsoft Azure HDInsight サービス]](http://go.microsoft.com/fwlink/?LinkId=616229) がオプションとして表示されます。
3. 次の画面で、**[スキーマ]** ドロップダウンの **[検索]** アイコンをクリックし、**[デフォルト]** をクリックします。

    ![Apache Spark BI のスキーマを検索](./media/apache-spark-use-bi-tools/tableau-find-schema-apache-spark-bi.png "Apache Spark BI のスキーマを検索")
4. **[テーブル]** フィールドで **[検索]** アイコンをクリックし、クラスターで使用可能なすべての Hive テーブルを一覧表示します。 Notebook を使用して前に作成した **hvac** テーブルが表示されます。

    ![Apache Spark BI のテーブルを検索](./media/apache-spark-use-bi-tools/tableau-find-table-apache-spark-bi.png "Apache Spark BI のテーブルを検索")
5. テーブルをドラッグして右側上部のボックスにドロップします。 Tableau はデータをインポートし、赤い四角で強調表示されているようにスキーマを表示します。

    ![Apache Spark BI の Tableau にデータを追加](./media/apache-spark-use-bi-tools/tableau-add-table-apache-spark-bi.png "Apache Spark BI の Tableau にデータを追加")
6. 左下の **[Sheet1]** タブをクリックします。 すべてのビルの各日の目標温度と実際の温度の平均を表示するグラフを作成します。 **[日付]** と **[ビル ID]** を **[列]** に、**[実際の温度]**/**[目標温度]** を **[行]** にドラッグします。 **[マーク]** で **[領域]** を選択して、Spark データ視覚化で領域マップを使用します。

     ![Spark データ視覚化のフィールドを追加](./media/apache-spark-use-bi-tools/spark-data-visualization-add-fields.png "Spark データ視覚化のフィールドを追加")
7. 既定では、温度フィールドは集計として表示されます。 代わりに平均温度を表示する場合は、次のスクリーンショットに示すように、ドロップダウンから行うことができます。

    ![Spark データ視覚化の平均値の取得](./media/apache-spark-use-bi-tools/spark-data-visualization-average-temperature.png "Spark データ視覚化の平均値の取得")

8. 一方の温度を他の温度にスーパーインポーズして、温度の違いを見やすくすることもできます。 マウスを下部領域マップの隅に移動し、赤い丸で囲んだハンドル形状にします。 マップを上部の他のマップにドラッグし、マウスが赤い四角で囲んだ形状になったら放します。

    ![Spark データ視覚化のマップをマージ](./media/apache-spark-use-bi-tools/spark-data-visualization-merge-maps.png "Spark データ視覚化のマップをマージ")

     データの視覚化が、次のスクリーンショットのように変化します。

    ![Spark データ視覚化の Tableau 出力](./media/apache-spark-use-bi-tools/spark-data-visualization-tableau-output.png "Spark データ視覚化の Tableau 出力")
9. **[保存]** をクリックしてワークシートを保存します。 ダッシュボードを作成して 1 つまたは複数のシートを追加できます。

## <a name="next-steps"></a>次の手順

これまでに、クラスターを作成し、データを照会するための Spark データ フレームを作成し、BI ツールからそのデータにアクセスする方法を学習しました。 次は、クラスターのリソースを管理し、HDInsight Spark クラスターで実行されているジョブをデバッグする方法を見ていきましょう。

* [Azure HDInsight での Apache Spark クラスターのリソースの管理](apache-spark-resource-manager.md)
* [HDInsight の Apache Spark クラスターで実行されるジョブの追跡とデバッグ](apache-spark-job-debugging.md)


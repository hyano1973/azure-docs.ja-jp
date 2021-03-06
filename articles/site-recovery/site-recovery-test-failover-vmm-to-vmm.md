---
title: "Azure Site Recovery でのテスト フェールオーバー (VMM から VMM) | Microsoft Docs"
description: "Azure Site Recovery は、仮想マシンと物理サーバーのレプリケーション、フェールオーバー、回復を調整します。 Azure またはセカンダリ データセンターへのフェールオーバーについて説明します。"
services: site-recovery
documentationcenter: 
author: ponatara
manager: abhemraj
editor: 
ms.assetid: 44813a48-c680-4581-a92e-cecc57cc3b1e
ms.service: site-recovery
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: storage-backup-recovery
ms.date: 11/22/2017
ms.author: ponatara
ms.openlocfilehash: 2730cebc1cdc47db283ae851560d93fdbf50ee48
ms.sourcegitcommit: 310748b6d66dc0445e682c8c904ae4c71352fef2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/28/2017
---
# <a name="test-failover-vmm-to-vmm-in-site-recovery"></a>Site Recovery でのテスト フェールオーバー (VMM から VMM)


この記事では、Azure Site Recovery を使用して保護されている仮想マシン (VM) と物理サーバーのテスト フェールオーバーまたはディザスター リカバリー (DR) のドリルを行うための情報と手順について説明します。 復旧サイトとして、System Center Virtual Machine Manager (VMM) で管理されたオンプレミスのサイトを使用します。

テスト フェールオーバーを実行し、データの損失またはダウンタイムを発生させることなく、レプリケーション戦略の検証または DR ドリルを行います。 テスト フェールオーバーは、実行中のレプリケーションや運用環境に影響はありません。 仮想マシンまたは[復旧計画](site-recovery-create-recovery-plans.md)で実行することができます。 テスト フェールオーバーをトリガーするときは、テスト仮想マシンの接続先のネットワークを指定する必要があります。 テスト フェールオーバーの進行状況は、**[ジョブ]** ページで追跡できます。  

コメントや質問がある場合は、この記事の末尾、または [Azure Recovery Services フォーラム](https://social.msdn.microsoft.com/forums/azure/home?forum=hypervrecovmgr)で投稿してください。


## <a name="prepare-the-infrastructure-for-test-failover"></a>テスト フェールオーバー用のインフラストラクチャの準備
既存のネットワークを使用してテスト フェールオーバーを実行する場合は、そのネットワーク内に Active Directory、DHCP、および DNS を準備します。

VM ネットワークを自動的に作成するオプションを使用してテスト フェールオーバーを実行する場合は、テスト フェールオーバーで使用する予定の復旧計画の Group-1 の前に手動の手順を追加します。 次に、テスト フェールオーバーを実行する前に自動的に作成されたネットワークにインフラストラクチャ リソースを追加します。

### <a name="things-to-note"></a>注意する点
セカンダリ サイトにレプリケートする場合、レプリカ マシンで使用されるネットワークの種類は、テスト フェールオーバーに使用される論理ネットワークの種類と一致している必要はありませんが、組み合わせによっては動作しない可能性があります。 レプリカが DHCP と VLAN ベースの分離を使用する場合、レプリカの VM ネットワークには、静的 IP アドレス プールは必要ありません。 このため、Windows ネットワーク仮想化を使用してテスト フェールオーバーを実行しようとすると、使用できるアドレス プールがないため失敗します。 

さらに、レプリカ ネットワークが分離を使用せず、テスト ネットワークが Windows ネットワーク仮想化を使用する場合、テスト フェールオーバーは失敗します。 これは、分離なしのネットワークには、Windows ネットワーク仮想化ネットワークの作成に必要なサブネットがないためです。

フェールオーバー後にマップされた VM ネットワークにレプリカ仮想マシンが接続される方法は、VMM コンソールでの VM ネットワークの構成方法によって異なります。

#### <a name="vm-network-configured-with-no-isolation-or-vlan-isolation"></a>分離なしまたは VLAN 分離で構成された VM ネットワーク
VM ネットワークに DHCP が定義されている場合、レプリカ仮想マシンは、関連する論理ネットワーク内のネットワーク サイトに指定されている設定を使用して、VLAN ID に接続されます。 仮想マシンは、使用可能は DHCP サーバーから IP アドレスを受け取ります。 

ターゲットの VM ネットワークに対して静的 IP アドレス プールを定義する必要はありません。 VM ネットワークに静的 IP アドレス プールが使用されている場合、レプリカ仮想マシンは、関連する論理ネットワーク内のネットワーク サイトに指定されている設定を使用して、VLAN ID に接続されます。

仮想マシンは、VM ネットワークに定義されているプールから IP アドレスを受け取ります。 静的 IP アドレス プールがターゲット VM ネットワーク上で定義されていない場合、IP アドレスの割り当ては失敗します。 IP アドレス プールは、保護と復旧に使用するソースとターゲットの両方の VMM サーバー上で作成します。

#### <a name="vm-network-with-windows-network-virtualization"></a>Windows ネットワーク仮想化を使用する VM ネットワーク
VM ネットワークが Windows ネットワーク仮想化を使用して構成されている場合、ソース VM ネットワークが DHCP または静的 IP アドレス プールを使用するように構成されているかどうかに関係なく、ターゲット VM ネットワークに対して静的プールを定義する必要があります。 

DHCP を定義する場合、ターゲット VMM サーバーは、DHCP サーバーとして機能し、ターゲット VM ネットワークに対して定義されているプールから IP アドレスを提供します。 ソース サーバーに静的 IP アドレス プールの使用が定義されている場合、ターゲット VMM サーバーは、プールから IP アドレスを割り当てます。 どちらの場合も、静的 IP アドレス プールが定義されていないと、IP アドレスの割り当ては失敗します。


### <a name="prepare-dhcp"></a>DHCP の準備
テスト フェールオーバーに関係している仮想マシンが DHCP を使用する場合は、テスト フェールオーバー用の分離されたネットワーク内でテスト DHCP サーバーを作成します。

### <a name="prepare-active-directory"></a>Active Directory の準備
アプリケーションのテストのためにテスト フェールオーバーを実行するには、テスト環境内に Active Directory 運用環境のコピーが必要です。 詳細については、[Active Directory 用のテスト フェールオーバーの考慮事項](site-recovery-active-directory.md#test-failover-considerations)を参照してください。

### <a name="prepare-dns"></a>DNS の準備
次のように、テスト フェールオーバー用の DNS サーバーを準備します。

* **DHCP**: 仮想マシンが DHCP を使用する場合、テスト DHCP サーバーでテスト DNS の IP アドレスを更新する必要があります。 Windows ネットワーク仮想化のネットワーク タイプを使用している場合、VMM サーバーは DHCP サーバーとして機能します。 したがって、テスト フェールオーバー ネットワークの DNS の IP アドレスを更新する必要があります。 この場合、仮想マシンは関連する DNS サーバーに自身を登録します。
* **静的アドレス**: 仮想マシンが静的 IP アドレスを使用する場合、テスト フェールオーバー ネットワークでテスト DNS サーバーの IP アドレスを更新する必要があります。 場合によっては、テスト仮想マシンの IP アドレスを DNS に反映することも必要です。 この目的のために、次のサンプル スクリプトを使用することができます。

        Param(
        [string]$Zone,
        [string]$name,
        [string]$IP
        )
        $Record = Get-DnsServerResourceRecord -ZoneName $zone -Name $name
        $newrecord = $record.clone()
        $newrecord.RecordData[0].IPv4Address  =  $IP
        Set-DnsServerResourceRecord -zonename $zone -OldInputObject $record -NewInputObject $Newrecord



## <a name="run-a-test-failover"></a>テスト フェールオーバーの実行
この手順では、復旧計画のテスト フェールオーバーを実行する方法について説明します。 別の方法として、**[仮想マシン]** タブで、単一の仮想マシンに対するフェールオーバーを実行することもできます。

![[テスト フェールオーバー] ブレード](./media/site-recovery-test-failover-vmm-to-vmm/TestFailover.png)

1. **[復旧計画]**  >  *recoveryplan_name* を選択します。 **フェールオーバー** > **Test フェールオーバー**で投稿してください。
1. **[テスト フェールオーバー]** ブレードで、テスト フェールオーバー後に仮想マシンをネットワークに接続する方法を指定します。 詳細については、[ネットワーク オプション](#network-options-in-site-recovery)を参照してください。
1. **[ジョブ]** タブで、フェールオーバーの進行状況を追跡します。
1. フェールオーバーが完了したら、仮想マシンが正常に起動することを確認します。
1. 完了したら、復旧計画の **[テスト フェールオーバーのクリーンアップ]** をクリックします。 **[メモ]** を使用して、テスト フェールオーバーに関連する観察結果をすべて記録し、保存します。 この手順により、テスト フェールオーバー中に作成された仮想マシンとネットワークが削除されます。


## <a name="network-options-in-site-recovery"></a>Site Recovery のネットワーク オプション

テスト フェールオーバーの実行時には、テスト レプリカ マシン用のネットワーク設定を選択することが求められます。 これには複数のオプションがあります。  

| **テスト フェールオーバーのオプション** | **説明** | **フェールオーバーの確認** | **詳細** |
| --- | --- | --- | --- |
| **セカンダリ VMM サイトへのフェールオーバー -- ネットワークなし** |VM ネットワークを選択しません。 |フェールオーバーでは、テスト マシンが作成されることが確認されます。<br/><br/>レプリカ仮想マシンが存在するホストに、テスト仮想マシンが作成されます。 これは、レプリカ仮想マシンが配置されているクラウドには追加されません。 |<p>フェールオーバーされたマシンは、いずれのネットワークにも接続されません。<br/><br/>マシンは作成された後で VM ネットワークに接続できます。 |
| **セカンダリ VMM サイトへのフェールオーバー -- ネットワークあり** |既存の VM ネットワークを選択します。 |フェールオーバーでは、仮想マシンが作成されることが確認されます。 |レプリカ仮想マシンが存在するホストに、テスト仮想マシンが作成されます。 これは、レプリカ仮想マシンが配置されているクラウドには追加されません。<br/><br/>運用ネットワークから分離された VM ネットワークを作成します。<br/><br/>VLAN ベースのネットワークを使用する場合は、この目的用の独立した (運用環境で使用されない) 論理ネットワークを VMM で作成することをお勧めします。 この論理ネットワークは、テスト フェールオーバー用の VM ネットワークの作成に使用されます。<br/><br/>この論理ネットワークは、仮想マシンをホストしているすべての Hyper-V サーバーのネットワーク アダプターのうち少なくとも 1 つに関連付けする必要があります。<br/><br/>VLAN 論理ネットワークの場合、論理ネットワークに追加するネットワーク サイトは分離されている必要があります。<br/><br/>Windows ネットワーク仮想化ベースの論理ネットワークを使用している場合、分離された VM ネットワークが Azure Site Recovery によって自動的に作成されます。 |
| **セカンダリ VMM サイトへのフェールオーバー -- ネットワークの作成** |一時的なテスト ネットワークは、**論理ネットワーク**とそれに関連するネットワーク サイトで指定した設定に基づいて、自動的に作成されます。 |フェールオーバーでは、仮想マシンが作成されることが確認されます。 |このオプションは、復旧計画で複数の VM ネットワークを使用する場合に使用します。 Windows ネットワーク仮想化ネットワークを使用する場合には、このオプションにより、レプリカ仮想マシンのネットワーク内に同じ設定 (サブネットおよび IP アドレス プール) を持つ VM ネットワークを自動的に作成できます。 これらの VM ネットワークは、テスト フェールオーバーの完了後に自動的にクリーンアップされます。</p><p>レプリカ仮想マシンが存在するホストに、テスト仮想マシンが作成されます。 これは、レプリカ仮想マシンが配置されているクラウドには追加されません。 |

> [!TIP]
> テスト フェールオーバー中に仮想マシンに指定された IP アドレスは、仮想マシンが計画されたフェールオーバーや計画されていないフェールオーバー中に受信するのと同じ IP アドレスです (IP アドレスはテスト フェールオーバー ネットワークで利用可能であることが前提)。 テスト フェールオーバー ネットワークで同じ IP が使用できない場合、仮想マシンは、テスト フェールオーバー ネットワークで利用できる別の IP を受信します。
>
>


## <a name="test-failover-to-a-production-network-on-a-recovery-site"></a>復旧サイトの運用ネットワークへのテスト フェールオーバー
テスト フェールオーバーを行うときは、[[ネットワーク マッピング]](https://docs.microsoft.com/azure/site-recovery/site-recovery-network-mapping) で指定した運用復旧サイト ネットワークとは異なるネットワークを選択することをお勧めします。 ただし、フェールオーバーされた仮想マシンでエンド ツー エンドのネットワーク接続を実際に検証したい場合は、次の点に注意してください。

* テスト フェールオーバーを実行するときは、プライマリ仮想マシンがシャットダウンされていることを確認します。 そうしないと、同じ ID を持つ 2 つの仮想マシンが同じネットワークで同時に実行されることになります。 その場合、望ましくない結果になることがあります。
* テスト フェールオーバー仮想マシンに加えた変更は、テスト フェールオーバー仮想マシンをクリーンアップすると失われます。 これらの変更は、プライマリ仮想マシンにはレプリケートされません。
* このようなテスト方法では、運用アプリケーションのダウンタイムにつながります。 DR ドリルの実行中はアプリケーションのユーザーにアプリケーションを使用しないように指示してください。  


## <a name="next-steps"></a>次のステップ
テスト フェールオーバーが正常に実行されたら、[フェールオーバー](site-recovery-failover.md)を試すことができます。

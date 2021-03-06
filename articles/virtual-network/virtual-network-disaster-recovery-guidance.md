---
title: "Azure Virtual Networks に影響を与える Azure サービスの中断が発生した場合の対処方法 | Microsoft Docs"
description: "Azure Virtual Networks に影響を与える Azure サービスの中断が発生した場合の対処方法について説明します。"
services: virtual-network
documentationcenter: 
author: NarayanAnnamalai
manager: jefco
editor: 
ms.assetid: ad260ab9-d873-43b3-8896-f9a1db9858a5
ms.service: virtual-network
ms.workload: virtual-network
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/16/2016
ms.author: narayan;aglick
ms.openlocfilehash: 4e125406d2e798138c45e3fbbf61a610afab69fc
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/11/2017
---
# <a name="virtual-network--business-continuity"></a>Virtual Network – ビジネス継続性
## <a name="overview"></a>概要
Virtual Network (VNet) は、クラウド内のユーザーのネットワークを論理的に表したものです。 これにより、独自のプライベート IP アドレス空間を定義し、ネットワークをサブネットに分割することができます。 VNet は、Azure Virtual Machines や Cloud Services (web/worker ロール) などのコンピューティング リソースをホストする際に信頼の境界として機能します。 VNet の内部にホストされているリソースどうしは、プライベート IP で直接通信できます。 Virtual Network は、VPN Gateway や ExpressRoute などのハイブリッド オプションのいずれかを介して、オンプレミスのネットワークにもリンクすることができます。

VNet は、1 つのリージョンの範囲内に作成します。 同じアドレス空間を持つ VNet を異なる 2 つのリージョンに作成できます (つまり、米国東部と米国西部に作成できますが、これらの間を直接接続することはできません)。 

## <a name="business-continuity"></a>ビジネス継続性
アプリケーションが中断される原因はさまざまです。 特定のリージョンで、自然災害によりネットワークが完全に切断される可能性や、いくつかのデバイスやサービスの障害により部分的に障害が発生する可能性があります。 VNet サービスへの影響は、これらの状況によって異なります。

**Q: リージョン全体の機能が停止した場合、つまり、自然災害により、リージョンが完全に切断された場合、どうなりますか。そのリージョンでホストされている Virtual Network はどうなりますか。**

A: サービスが中断されている間は、影響を受けたリージョン内の Virtual Network とリソースにアクセスできなくなります。

![単純な Virtual Network の図](./media/virtual-network-disaster-recovery-guidance/vnet.png)

**Q: 別のリージョンで同じ仮想ネットワークを再作成するには、どうすればよいですか。**

A: Virtual Network (VNet) は、非常に軽量なリソースです。 Azure API を呼び出して、別のリージョンに同じアドレス空間を持つ VNet を作成することができます。 影響を受けたリージョンに存在していたのと同じ環境を再作成するには、API を呼び出して、Cloud Services (web/worker ロール) と Virtual Machines を再デプロイします。 また、オンプレミス接続 (ハイブリッド デプロイなど) があった場合は、VPN Gateway を起動し、オンプレミスのネットワークに接続する必要があります。

VNet を作成する手順については、 [こちら](virtual-networks-create-vnet-arm-pportal.md)をご覧ください。 

**Q: 特定のリージョンの VNet のレプリカを別のリージョンに前もって再作成できますか。**

A: はい。VNet の同じプライベート IP アドレス空間とリソースを使用して、2 つの異なるリージョンに 2 つの VNet を前もって作成できます。 お客様が VNet 内でインターネットに接続するサービスをホストしていた場合は、Traffic Manager をセットアップして、アクティブなリージョンにトラフィックを地理的に分散させることができます。 ただし、ルーティングの問題の原因となるため、同じアドレス空間を持つ 2 つの VNet をオンプレミスのネットワークに接続することはできません。 一方のリージョンで災害が発生し、VNet が失われた場合は、アドレス空間が一致する利用可能なリージョンのもう一方の VNet をオンプレミスのネットワークに接続できます。

VNet を作成する手順については、 [こちら](virtual-networks-create-vnet-arm-pportal.md)をご覧ください。


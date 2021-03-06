---
title: "Azure Virtual Machines に対するネットワーク インターフェイスの追加と削除 | Microsoft Docs"
description: "仮想マシンにネットワーク インターフェイスを追加する、または仮想マシンからネットワーク インターフェイスを削除する方法について説明します。"
services: virtual-network
documentationcenter: na
author: jimdial
manager: timlt
editor: 
tags: azure-resource-manager
ms.assetid: 
ms.service: virtual-network
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 12/15/2017
ms.author: jdial
ms.openlocfilehash: abe6abb942d206330e809f3aef388b846d7d7c7f
ms.sourcegitcommit: 3f33787645e890ff3b73c4b3a28d90d5f814e46c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/03/2018
---
# <a name="add-network-interfaces-to-or-remove-network-interfaces-from-virtual-machines"></a>仮想マシンのネットワーク インターフェイスの追加と削除

VM を作成する際に既存のネットワーク インターフェイスを追加する方法と、停止 (割り当て解除) 状態の既存の VM に対してネットワーク インターフェイスの追加または削除を行う方法について説明します。 ネットワーク インターフェイスは、Azure 仮想マシン (VM) がインターネット、Azure、およびオンプレミスのリソースと通信できるようにします。 VM には、1 つまたは複数のネットワーク インターフェイスを実装できます。 

ネットワーク インターフェイスに対して IP アドレスの追加、変更、または削除が必要な場合は、[ネットワーク インターフェイスの IP アドレスの管理](virtual-network-network-interface-addresses.md)についての記事を参照してください。 ネットワーク インターフェイスを作成、変更、または削除する必要がある場合は、[ネットワークインターフェイスの管理](virtual-network-network-interface.md)に関する記事を参照してください。

## <a name="before"></a>開始する前に

この記事のいずれかのセクションの手順を実行する前に、次のタスクを完了します。

- Azure アカウントを使用して、Azure [Portal](https://portal.azure.com)、Azure コマンド ライン インターフェイス (CLI)、または Azure PowerShell にログインします。 まだ Azure アカウントを持っていない場合は、[無料試用版アカウント](https://azure.microsoft.com/free)にサインアップしてください。
- PowerShell コマンドを使用してこの記事のタスクを実行する場合は、[Azure PowerShell をインストールして構成します](/powershell/azureps-cmdlets-docs?toc=%2fazure%2fvirtual-network%2ftoc.json)。 最新バージョンの Azure PowerShell コマンドレットがインストールされていることを確認してください。 PowerShell コマンドのヘルプとサンプルを表示するには、「`get-help <command> -full`」と入力します。 Azure PowerShell をインストールする代わりに、Azure Cloud Shell を使うことができます。 Azure Cloud Shell は、Azure Portal 内で直接実行できる無料の PowerShell です。 Azure PowerShell が事前にインストールされており、アカウントで使用できるように構成されています。 Cloud Shell を使用するには、[ポータル](https://portal.azure.com)の上部にある Cloud Shell **>_** ボタンをクリックし、シェル ウィンドウが表示されたら、左上隅の PowerShell を選択します。
- Azure コマンド ライン インターフェイス (CLI) のコマンドを使ってこの記事のタスクを実行する場合は、[Azure CLI をインストールして構成します](/cli/azure/install-azure-cli?toc=%2fazure%2fvirtual-network%2ftoc.json)。 最新バージョンの Azure CLI がインストールされていることを確認してください。 CLI コマンドのヘルプを表示するには、「`az <command> --help`」と入力します。 CLI とその前提条件をインストールする代わりに、Azure Cloud Shell を使うことができます。 Azure Cloud Shell は、Azure Portal 内で直接実行できる無料の Bash シェルです。 Azure CLI が事前にインストールされており、アカウントで使用できるように構成されています。 Cloud Shell を使用するには、[ポータル](https://portal.azure.com)の上部にある Cloud Shell **>_** ボタンをクリックし、シェル ウィンドウが表示されたら、左上隅の Bash を選択します。

## <a name="vm-create"></a>既存のネットワーク インターフェイスを新しい VM に追加する

ポータルで VM を作成すると、ネットワーク インターフェイスが既定の設定で自動的に作成され、VM にアタッチされます。 新しい VM への既存のネットワーク インターフェイスの追加と、複数のネットワーク インターフェイスがアタッチされた VM の作成は、Azure ポータルでは実行できません。 この 2 つは CLI または PowerShell を使用して行うことができます。 ただし、PowerShell または CLI を使用して、既存のネットワーク インターフェイスで VM を作成する前に、[制約](#constraints)について理解を深めます。 複数のネットワーク インターフェイスで仮想マシンを作成する場合は、VM が作成された後に、そのネットワーク インターフェイスが適切に使用されるようにオペレーティング システムを構成する必要もあります。 詳細については、[Linux](../virtual-machines/linux/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#configure-guest-os-for-multiple-nics) または [Windows](../virtual-machines/windows/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#configure-guest-os-for-multiple-nics) の構成に関するページで、複数のネットワーク インターフェイスについてご確認ください。

**コマンド**: VM を作成する前に、「[ネットワーク インターフェイスの作成](virtual-network-network-interface.md#create-a-network-interface)」の手順を使用して、ネットワーク インターフェイスを作成してください。

|ツール|コマンド|
|---|---|
|CLI|[az vm create](/cli/azure/vm?toc=%2fazure%2fvirtual-network%2ftoc.json#create)|
|PowerShell|[New-AzureRmVM](/powershell/module/azurerm.compute/new-azurermvm?toc=%2fazure%2fvirtual-network%2ftoc.json)|

## <a name="vm-add-nic"></a>ネットワーク インターフェイスを既存の VM に追加する

1. Azure ポータルにログインします。
2. ポータルの上部にある検索ボックスで、ネットワーク インターフェイスを追加する VM の名前を検索するか、**[すべてのサービス]**、**[仮想マシン]** の順にクリックして VM を参照します。 VM が見つかったら、クリックします。 ネットワーク インターフェイスを追加する VM は、追加するネットワーク インターフェイスの数をサポートしている必要があります。 各サイズの VM でサポートされるネットワーク インターフェイスの数については、[Linux](../virtual-machines/linux/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json) または [Windows](../virtual-machines/virtual-machines-windows-sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json) の VM のサイズに関する記事をご覧ください。  
3. **[設定]** で、**[概要]** をクリックします。 **[停止]** をクリックし、VM の**状態**が *[停止済み (割り当て解除)]* に変わるまで待ちます。 
4. **[設定]** で、**[ネットワーク]** をクリックします。
5. **[ネットワーク インターフェイスの接続]** をクリックします。 現在別の VM に接続されていない既存のネットワーク インターフェイスの一覧で、接続するネットワーク インターフェイスをクリックします。 選択したネットワーク インターフェイスで高速ネットワークを有効にしたり、IPv6 アドレスを割り当てたりすることはできません。また、このネットワーク インターフェイスの仮想ネットワークは、現在 VM に接続されているネットワーク インターフェイスが存在する仮想ネットワークと同じである必要があります。 既存のネットワーク インターフェイスがない場合は、最初に作成する必要があります。 ネットワーク インターフェイスを作成するには、**[ネットワーク インターフェイスの作成]** をクリックします。 ネットワーク インターフェイスの作成の詳細については、「[ネットワーク インターフェイスの作成](virtual-network-network-interface.md#create-a-network-interface)」を参照してください。 ネットワーク インターフェイスを仮想マシンに追加するときの追加の制約に関する詳細については、「[制約](#constraints)」を参照してください。
6. Click **OK**.
7. **[設定]** で、**[概要]** をクリックします。 **[開始]** をクリックして、仮想マシンを開始します。
8. 複数のネットワーク インターフェイスが適切に使用されるように VM オペレーティング システムを構成します。 詳細については、[Linux](../virtual-machines/linux/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#configure-guest-os-for-multiple-nics) または [Windows](../virtual-machines/windows/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#configure-guest-os-for-multiple-nics) の構成に関するページで、複数のネットワーク インターフェイスについてご確認ください。

|ツール|コマンド|
|---|---|
|CLI|[az vm nic add](/cli/azure/vm/nic?toc=%2fazure%2fvirtual-network%2ftoc.json#add) (参照) または [detailed steps](../virtual-machines/linux/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#add-a-nic-to-a-vm)|
|PowerShell|[Add-AzureRmVMNetworkInterface](/powershell/module/azurerm.compute/add-azurermvmnetworkinterface?toc=%2fazure%2fvirtual-network%2ftoc.json) (参照) or [detailed steps](../virtual-machines/windows/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#add-a-nic-to-an-existing-vm)|

## <a name="vm-view-nic"></a>VM のネットワーク インターフェイスを表示する

VM にアタッチされているネットワーク インターフェイスを表示して、各ネットワーク インターフェイスの構成の詳細や、各ネットワーク インターフェイスに割り当てられている IP アドレスを確認できます。 

1. ご利用のサブスクリプションの所有者、共同作成者、またはネットワーク共同作成者いずれかのロールが割り当てられているアカウントで、[Azure Portal](https://portal.azure.com) にログインします。 アカウントへのロールの割り当ての詳細については、「[Azure のロールベースのアクセス制御のための組み込みロール](../active-directory/role-based-access-built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor)」を参照してください。
2. Azure Portal 上部に "*リソースの検索*" というテキストが表示されたボックスがあります。そこに "*仮想マシン*" と入力します。 検索結果に **[仮想マシン]** が表示されたら、それをクリックします。
3. ネットワーク インターフェイスを表示する VM の名前をクリックします。
4. 選択した VM の **[設定]** セクションで、**[ネットワーク]** をクリックします。 ネットワーク インターフェイスの設定とそれを変更する方法については、[ネットワーク インターフェイスの管理](virtual-network-network-interface.md)に関するページをご覧ください。 ネットワーク インターフェイスに割り当てられる IP アドレスの追加、変更、または削除については、[IP アドレスの管理](virtual-network-network-interface-addresses.md)に関する記事を参照してください。

**コマンド**

|ツール|コマンド|
|---|---|
|CLI|[az vm show](/cli/azure/vm?toc=%2fazure%2fvirtual-network%2ftoc.json#show)|
|PowerShell|[Get-AzureRmVM](/powershell/module/azurerm.compute/get-azurermvm?toc=%2fazure%2fvirtual-network%2ftoc.json)|

## <a name="vm-remove-nic"></a> ネットワーク インターフェイスを VM から削除する

1. Azure ポータルにログインします。
2. ポータルの上部にある検索ボックスで、ネットワーク インターフェイスを削除 (デタッチ) する VM の名前を検索するか、**[すべてのサービス]**、**[仮想マシン]** の順にクリックして VM を参照します。 VM が見つかったら、クリックします。
3. **[設定]** で、**[概要]** をクリックします。 **[停止]** をクリックし、VM の**状態**が *[停止済み (割り当て解除)]* に変わるまで待ちます。 
4. **[設定]** で、**[ネットワーク]** をクリックします。
5. **[ネットワーク インターフェイスの切断]** をクリックします。 現在仮想マシンに接続されているネットワーク インターフェイスの一覧で、デタッチするネットワーク インターフェイスをクリックします。 1 つのネットワーク インターフェイスしか表示されていない場合、そのインターフェイスはデタッチできません。仮想マシンには、少なくとも 1 つのネットワーク インターフェイスが必ず接続されている必要があります。
6. Click **OK**.

**コマンド**

|ツール|コマンド|
|---|---|
|CLI|[az vm nic remove](/cli/azure/vm/nic?toc=%2fazure%2fvirtual-network%2ftoc.json#remove) (参照) または [detailed steps](../virtual-machines/linux/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#remove-a-nic-from-a-vm)|
|PowerShell|[Remove-AzureRMVMNetworkInterface](/powershell/module/azurerm.compute/remove-azurermvmnetworkinterface?toc=%2fazure%2fvirtual-network%2ftoc.json) (参照) または [detailed steps](../virtual-machines/windows/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#remove-a-nic-from-an-existing-vm)|

## <a name="next-steps"></a>次のステップ
複数のネットワーク インターフェイスまたは IP アドレスを持つ VM を作成する方法については、次の記事をご覧ください。

**コマンド**

|タスク|ツール|
|---|---|
|複数 NIC を持つ VM の作成|[CLI](../virtual-machines/linux/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json)、[PowerShell](../virtual-machines/windows/multiple-nics.md?toc=%2fazure%2fvirtual-network%2ftoc.json)|
|複数の IPv4 アドレスが割り当てられた 1 つの NIC VM の作成|[CLI](virtual-network-multiple-ip-addresses-cli.md)、[PowerShell](virtual-network-multiple-ip-addresses-powershell.md)|
|プライベート IPv6 アドレスが割り当てられた 1 つの NIC VM の作成 (Azure Load Balancer の背後)|[CLI](../load-balancer/load-balancer-ipv6-internet-cli.md?toc=%2fazure%2fvirtual-network%2ftoc.json)、[PowerShell](../load-balancer/load-balancer-ipv6-internet-ps.md?toc=%2fazure%2fvirtual-network%2ftoc.json)、[Azure Resource Manager テンプレート](../load-balancer/load-balancer-ipv6-internet-template.md?toc=%2fazure%2fvirtual-network%2ftoc.json)|

## <a name="constraints"></a>制約

- VM には少なくとも 1 つのネットワーク インターフェイスが接続されている必要があります。
- VM には、VM のサイズが対応できるだけの数のネットワーク インターフェイスしか接続できません。 各サイズの VM でサポートされるネットワーク インターフェイスの数については、[Linux](../virtual-machines/linux/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json) または [Windows](../virtual-machines/virtual-machines-windows-sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json) の VM のサイズに関するページをご覧ください。 すべてのサイズが、少なくとも 2 つのネットワーク インターフェイスに対応します。
- その時点で別の VM にアタッチされているネットワーク インターフェイスを VM に追加することはできません。 ネットワーク インターフェイスの作成の詳細については、「[ネットワーク インターフェイスの作成](virtual-network-network-interface.md#create-a-network-interface)」を参照してください。
- 以前は、複数のネットワーク インターフェイスをサポートしていて、少なくとも 2 つのネットワーク インターフェイスが作成された VM にのみ、ネットワーク インターフェイスを追加できました。 1 つのネットワーク インターフェイスが作成された VM は、VM のサイズが複数のネットワーク インターフェイスをサポートしている場合でも、ネットワーク インターフェイスを追加することはできませんでした。 逆に、ネットワーク インターフェイスを削除できるのは、少なくとも 3 つのネットワーク インターフェイスが存在する VM からのみでした。これは、少なくとも 2 つのネットワーク インターフェイスが作成された VM には、常に少なくとも 2 つのネットワーク インターフェイスが存在する必要があったためです。 これらの制約は、どちらも当てはまらなくなっています。 現在、任意の数 (VM のサイズでサポートされている最大数) のネットワーク インターフェイスで VM を作成できます。
- 既定では、VM に接続されている最初のネットワーク インターフェイスが、"*プライマリ*" ネットワーク インターフェイスとして定義されます。 VM 内の他のすべてのネットワーク インターフェイスは、*セカンダリ* ネットワーク インターフェイスになります。
- 送信トラフィックをどのネットワーク トラフィックに送信するかを制御することはできますが、既定では、VM からのすべての送信トラフィックが、プライマリ ネットワーク インターフェイスのプライマリ IP 構成に割り当てられた IP アドレスで送信されます。
- 以前は、同じ可用性セット内のすべての VM は、アタッチされるネットワーク インターフェイスを 1 つまたは複数に統一する必要がありました。 現在は、VM のサイズでサポートされている最大数までのネットワーク インターフェイスがアタッチされた VM が同じ可用性セットに存在できます。 ただし、VM を可用性セットに追加できるのは、VM の作成時のみです。 可用性セットについて詳しくは、[Azure での VM の可用性の管理](../virtual-machines/windows/manage-availability.md?toc=%2fazure%2fvirtual-network%2ftoc.json#configure-multiple-virtual-machines-in-an-availability-set-for-redundancy)に関する記事をご覧ください。
- 同じ VM にアタッチされている複数のネットワーク インターフェイスは、VNet 内の異なるサブネットに接続できますが、異なる VNet に接続することはできません。
- 任意のプライマリまたはセカンダリ ネットワーク インターフェイスの任意の IP 構成の任意の IP アドレスを Azure Load Balancer バックエンド プールに追加できます。 以前は、プライマリ ネットワーク インターフェイスのプライマリ IP アドレスのみをバックエンド プールに追加できました。 IP アドレスと IP 構成について詳しくは、[IP アドレスの追加、変更、削除](virtual-network-network-interface-addresses.md)に関する記事をご覧ください。
- VM を削除しても VM にアタッチされたネットワーク インターフェイスは削除されません。 VM を削除すると、ネットワーク インターフェイスは VM からデタッチされます。 デタッチされたネットワーク インターフェイスは、別の VM に追加することも削除することもできます。
- ネットワーク インターフェイスにプライベート IPv6 アドレスが割り当てられている場合は、VM の作成時に VM に追加 (接続) する必要があります。 VM の作成後に、IPv6 アドレスが割り当てられているネットワーク インターフェイスを VM に接続することはできません。 仮想マシンの作成時に、プライベート IPv6 アドレスが割り当てられたネットワーク インターフェイスを追加する場合、VM サイトがサポートしているネットワーク インターフェイス数にかかわらず、その仮想マシンに追加できるのはそのネットワーク インターフェイスのみです。 IP アドレスをネットワーク インターフェイスに割り当てる方法については、[ネットワーク インターフェイスの IP アドレス](virtual-network-network-interface-addresses.md)に関する記事を参照してください。
- IPv6 と同様、VM の作成後、高速ネットワークが有効になっているネットワーク インターフェイスは、VM に接続できません。 また、高速ネットワークを利用するには、VM オペレーティング システムで手順を実行する必要もあります。 高速ネットワークの詳細、および使用時の他の制約については、[Windows](create-vm-accelerated-networking-powershell.md) または [Linux](create-vm-accelerated-networking-cli.md) 仮想マシンの高速ネットワークに関する記事をご覧ください。

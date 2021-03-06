---
title: "Azure Virtual Networks と Windows 仮想マシン | Microsoft Docs"
description: "チュートリアル - Azure PowerShell を使用した Azure Virtual Networks と Windows 仮想マシンの管理"
services: virtual-machines-windows
documentationcenter: virtual-machines
author: davidmu1
manager: timlt
editor: tysonn
tags: azure-resource-manager
ms.assetid: 
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 10/12/2017
ms.author: davidmu
ms.custom: mvc
ms.openlocfilehash: 21f2d586a4c468071bec55c65005b35baf323fe7
ms.sourcegitcommit: 76a3cbac40337ce88f41f9c21a388e21bbd9c13f
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/25/2017
---
# <a name="manage-azure-virtual-networks-and-windows-virtual-machines-with-azure-powershell"></a>Azure PowerShell を使用した Azure Virtual Networks と Windows 仮想マシンの管理

Azure 仮想マシンでは、内部と外部のネットワーク通信に Azure ネットワークが使用されます。 このチュートリアルでは、2 台の仮想マシンをデプロイし、それらの VM に使用する Azure ネットワークを構成する手順について説明します。 このチュートリアルの例では、これらの VM が、データベース バックエンドを持つ Web アプリケーションのホストになっていることを想定していますが、アプリケーションのデプロイは、このチュートリアルでは行いません。 このチュートリアルで学習する内容は次のとおりです。

> [!div class="checklist"]
> * 仮想ネットワークとサブネットの作成
> * パブリック IP アドレスの作成
> * フロントエンド VM の作成
> * ネットワーク トラフィックのセキュリティ保護
> * バックエンド VM の作成

このチュートリアルを実行すると、次のようなリソースが作成されます。

![2 つのサブネットのある仮想ネットワーク](./media/tutorial-virtual-network/networktutorial.png)

- *myVNet* - 各 VM が相互間、またインターネットと通信するために使用する仮想ネットワーク。
- *myFrontendSubnet* - フロントエンド リソースが使用する、*myVNet* 内のサブネット。
- *myPublicIPAddress* - インターネットから *myFrontendVM* にアクセスするために使用するパブリック IP アドレス。
- *myFrontentNic* - *myFrontendVM* が *myBackendVM* と通信するために使用するネットワーク インターフェイス。
- *myFrontendVM* -インターネットと *myBackendVM* との間の通信に使用する VM。
- *myBackendNSG* - *myFrontendVM* と *myBackendVM* 間の通信を制御するネットワーク セキュリティ グループ。
- *myBackendSubnet* - *myBackendNSG* に関連付けられ、バックエンド リソースが使用するサブネット。
- *myBackendNic* - *myBackendVM* が *myFrontendVM* と通信するために使用するネットワーク インターフェイス。
- *myBackendVM* - ポート 1433 を使用して *myFrontendVM* と通信する VM。

このチュートリアルには、Azure PowerShell モジュール バージョン 3.6 以降が必要です。 バージョンを確認するには、`Get-Module -ListAvailable AzureRM` を実行します。 アップグレードする必要がある場合は、[Azure PowerShell モジュールのインストール](/powershell/azure/install-azurerm-ps)に関するページを参照してください。

## <a name="vm-networking-overview"></a>VM ネットワークの概要

Azure 仮想ネットワークを使用すると、仮想マシン、インターネット、その他の Azure サービス (Azure SQL Database など) の間でセキュリティ保護されたネットワーク接続を実現できます。 仮想ネットワークは、サブネットと呼ばれる論理セグメントに分割することができます。 サブネットは、ネットワーク フローを制御する目的のほか、セキュリティ境界としても使用されます。 VM をデプロイするときは、通常、そこに仮想ネットワーク インターフェイスが含まれているので、それをサブネットに接続することになります。

## <a name="create-a-virtual-network-and-subnet"></a>仮想ネットワークとサブネットの作成

このチュートリアルでは、2 つのサブネットから成る単一の仮想ネットワークを作成します。 Web アプリケーションのホストとなるフロントエンド サブネットと、データベース サーバーのホストとなるバックエンド サブネットです。

仮想ネットワークを作成する前に、[New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) を使用してリソース グループを作成します。 次の例では、*myRGNetwork* という名前のリソース グループを場所 *EastUS* に作成します。

```azurepowershell-interactive
New-AzureRmResourceGroup -ResourceGroupName myRGNetwork -Location EastUS
```

### <a name="create-subnet-configurations"></a>サブネットの構成の作成

[New-AzureRmVirtualNetworkSubnetConfig](/powershell/module/azurerm.network/new-azurermvirtualnetworksubnetconfig) を使用して *myFrontendSubnet* という名前のサブネット構成を作成します。

```azurepowershell-interactive
$frontendSubnet = New-AzureRmVirtualNetworkSubnetConfig `
  -Name myFrontendSubnet `
  -AddressPrefix 10.0.0.0/24
```

また、*myBackendSubnet* という名前のサブネット構成を作成します。

```azurepowershell-interactive
$backendSubnet = New-AzureRmVirtualNetworkSubnetConfig `
  -Name myBackendSubnet `
  -AddressPrefix 10.0.1.0/24
```

### <a name="create-virtual-network"></a>Create virtual network

[New-AzureRmVirtualNetwork](/powershell/module/azurerm.network/new-azurermvirtualnetwork) を使用して、*myFrontendSubnet* と *myBackendSubnet* を使用する *myVNet* という名前の VNET を作成します。

```azurepowershell-interactive
$vnet = New-AzureRmVirtualNetwork `
  -ResourceGroupName myRGNetwork `
  -Location EastUS `
  -Name myVNet `
  -AddressPrefix 10.0.0.0/16 `
  -Subnet $frontendSubnet, $backendSubnet
```

この時点で、1 つのネットワークが作成され、フロントエンド サービス用とバックエンド サービス用の 2 つのサブネットに分割されました。 次のセクションでは、仮想マシンを作成してこれらのサブネットに接続します。

## <a name="create-a-public-ip-address"></a>パブリック IP アドレスの作成

Azure リソースにインターネットからアクセスするためには、パブリック IP アドレスが必要です。 パブリック IP アドレスの割り当て方法は、動的または静的として構成することができます。 既定では、パブリック IP アドレスが動的に割り当てられます。 動的 IP アドレスは、VM の割り当てが解除されると解放されます。 この動作により、VM の割り当て解除を伴う操作の過程で IP アドレスが変化することになります。

割り当て方法を "静的" に設定することで、VM に割り当てた IP アドレスを確実に維持し、割り当てが解除された状態でも変更されないようにすることができます。 静的に割り当てられた IP アドレスを使用するときに、IP アドレス自体を指定することはできません。 それは、利用可能なアドレスのプールから割り当てられます。

[New-AzureRmPublicIpAddress](/powershell/module/azurerm.network/new-azurermpublicipaddress) を使用して *myPublicIPAddress* という名前の IP アドレスを作成します。

```azurepowershell-interactive
$pip = New-AzureRmPublicIpAddress `
  -ResourceGroupName myRGNetwork `
  -Location EastUS `
  -AllocationMethod Dynamic `
  -Name myPublicIPAddress
```

静的なパブリック IP アドレスを割り当てるには、-AllocationMethod パラメーターを `Static` に変更します。

## <a name="create-a-front-end-vm"></a>フロントエンド VM の作成

VM が仮想ネットワーク内で通信するには、仮想ネットワーク インターフェイス (NIC) が必要です。 [New-AzureRmNetworkInterface](/powershell/module/azurerm.network/new-azurermnetworkinterface) を使用して、NIC を作成します。

```azurepowershell-interactive
$frontendNic = New-AzureRmNetworkInterface `
  -ResourceGroupName myRGNetwork `
  -Location EastUS `
  -Name myFrontendNic `
  -SubnetId $vnet.Subnets[0].Id `
  -PublicIpAddressId $pip.Id
```

[Get-Credential](https://msdn.microsoft.com/powershell/reference/5.1/microsoft.powershell.security/Get-Credential) を使用して、VM の管理者アカウントに必要なユーザー名とパスワードを設定します。 追加の手順でこれらの資格情報を使用して VM に接続します。

```azurepowershell-interactive
$cred = Get-Credential
```

[New-AzureRmVMConfig](/powershell/module/azurerm.compute/new-azurermvmconfig)、[Set-AzureRmVMOperatingSystem](/powershell/module/azurerm.compute/set-azurermvmoperatingsystem)、[Set-AzureRmVMSourceImage](/powershell/module/azurerm.compute/set-azurermvmsourceimage)、[Set-AzureRmVMOSDisk](/powershell/module/azurerm.compute/set-azurermvmosdisk)、[Add-AzureRmVMNetworkInterface](/powershell/module/azurerm.compute/add-azurermvmnetworkinterface)、[New-AzureRmVM](/powershell/module/azurerm.compute/new-azurermvm) を使用して、VM を作成します。

```azurepowershell-interactive
$frontendVM = New-AzureRmVMConfig `
    -VMName myFrontendVM `
    -VMSize Standard_D1
$frontendVM = Set-AzureRmVMOperatingSystem `
    -VM $frontendVM `
    -Windows `
    -ComputerName myFrontendVM `
    -Credential $cred `
    -ProvisionVMAgent `
    -EnableAutoUpdate
$frontendVM = Set-AzureRmVMSourceImage `
    -VM $frontendVM `
    -PublisherName MicrosoftWindowsServer `
    -Offer WindowsServer `
    -Skus 2016-Datacenter `
    -Version latest
$frontendVM = Set-AzureRmVMOSDisk `
    -VM $frontendVM `
    -Name myFrontendOSDisk `
    -DiskSizeInGB 128 `
    -CreateOption FromImage `
    -Caching ReadWrite
$frontendVM = Add-AzureRmVMNetworkInterface `
    -VM $frontendVM `
    -Id $frontendNic.Id
New-AzureRmVM `
    -ResourceGroupName myRGNetwork `
    -Location EastUS `
    -VM $frontendVM
```

## <a name="secure-network-traffic"></a>ネットワーク トラフィックのセキュリティ保護

ネットワーク セキュリティ グループ (NSG) には、Azure Virtual Network (VNet) に接続されたリソースへのネットワーク トラフィックを許可または拒否する一連のセキュリティ規則が含まれています。 NSG はサブネットに関連付けることができるほか、個々のネットワーク インターフェイスに関連付けることができます。 NSG をネットワーク インターフェイスに関連付けた場合、関連付けられている VM のみにその NSG が適用されます。 NSG をサブネットに関連付けた場合、そのサブネットに接続されているすべてのリソースにその NSG のルールが適用されます。

### <a name="network-security-group-rules"></a>ネットワーク セキュリティ グループ ルール

NSG ルールは、トラフィックが許可または拒否されるネットワーク ポートを定義します。 このルールには、特定のシステム (またはサブネット) 間のトラフィックが制御されるように送信元 IP と送信先 IP のアドレス範囲を含めることができます。 また、NSG ルールには優先順位 (1 ～ 4096) も含まれます。 ルールは、優先順位に従って評価されます。 優先順位が 100 のルールは、優先順位が 200 のルールよりも前に評価されます。

すべての NSG に既定のルール一式が含まれています。 既定のルールは削除できませんが、これには最も低い優先順位が割り当てられているため、ルールを作成することで上書きできます。

- **仮想ネットワーク** - 仮想ネットワーク内で発信および着信するトラフィックについては、受信方向と送信方向の両方で許可されます。
- **インターネット** - 送信トラフィックは許可されますが、受信トラフィックはブロックされます。
- **ロード バランサー** - Azure のロード バランサーによる VM とロール インスタンスの正常性プローブが許可されます。 負荷分散セットを使用していない場合は、このルールを上書きできます。

### <a name="create-network-security-groups"></a>ネットワーク セキュリティ グループの作成

[New-AzureRmNetworkSecurityRuleConfig](/powershell/module/azurerm.network/new-azurermnetworksecurityruleconfig) を使用して、*myFrontendVM* 上で受信 Web トラフィックを可能にする *myFrontendNSGRule* という名前の受信規則を作成します。

```azurepowershell-interactive
$nsgFrontendRule = New-AzureRmNetworkSecurityRuleConfig `
  -Name myFrontendNSGRule `
  -Protocol Tcp `
  -Direction Inbound `
  -Priority 200 `
  -SourceAddressPrefix * `
  -SourcePortRange * `
  -DestinationAddressPrefix * `
  -DestinationPortRange 80 `
  -Access Allow
```

バックエンド サブネットの NSG を作成することで、*myBackendVM* への内部トラフィックを *myFrontendVM* からのみに制限できます。 次の例では、*myBackendNSGRule* という名前の NSG 規則を作成します。

```azurepowershell-interactive
$nsgBackendRule = New-AzureRmNetworkSecurityRuleConfig `
  -Name myBackendNSGRule `
  -Protocol Tcp `
  -Direction Inbound `
  -Priority 100 `
  -SourceAddressPrefix 10.0.0.0/24 `
  -SourcePortRange * `
  -DestinationAddressPrefix * `
  -DestinationPortRange 1433 `
  -Access Allow
```

[New-AzureRmNetworkSecurityGroup](/powershell/module/azurerm.network/new-azurermnetworksecuritygroup) を使用して、*myFrontendNSG* という名前のネットワーク セキュリティ グループを追加します。

```azurepowershell-interactive
$nsgFrontend = New-AzureRmNetworkSecurityGroup `
  -ResourceGroupName myRGNetwork `
  -Location EastUS `
  -Name myFrontendNSG `
  -SecurityRules $nsgFrontendRule
```

ここで、New-AzureRmNetworkSecurityGroup を使用して、*myBackendNSG* という名前のネットワーク セキュリティ グループを追加します。

```azurepowershell-interactive
$nsgBackend = New-AzureRmNetworkSecurityGroup `
  -ResourceGroupName myRGNetwork `
  -Location EastUS `
  -Name myBackendNSG `
  -SecurityRules $nsgBackendRule
```

ネットワーク セキュリティ グループをサブネットに追加します。

```azurepowershell-interactive
$vnet = Get-AzureRmVirtualNetwork `
  -ResourceGroupName myRGNetwork `
  -Name myVNet
$frontendSubnet = $vnet.Subnets[0]
$backendSubnet = $vnet.Subnets[1]
$frontendSubnetConfig = Set-AzureRmVirtualNetworkSubnetConfig `
  -VirtualNetwork $vnet `
  -Name myFrontendSubnet `
  -AddressPrefix $frontendSubnet.AddressPrefix `
  -NetworkSecurityGroup $nsgFrontend
$backendSubnetConfig = Set-AzureRmVirtualNetworkSubnetConfig `
  -VirtualNetwork $vnet `
  -Name myBackendSubnet `
  -AddressPrefix $backendSubnet.AddressPrefix `
  -NetworkSecurityGroup $nsgBackend
Set-AzureRmVirtualNetwork -VirtualNetwork $vnet
```

## <a name="create-a-back-end-vm"></a>バックエンド VM の作成

SQL Server イメージを使用すると、このチュートリアル用に最も簡単にバックエンド VM を作成できます。 このチュートリアルではデータベース サーバーを使用して VM を作成しますが、データベースへのアクセスに関する情報は提供しません。

*myBackendNic* を作成します。

```azurepowershell-interactive
$backendNic = New-AzureRmNetworkInterface `
  -ResourceGroupName myRGNetwork `
  -Location EastUS `
  -Name myBackendNic `
  -SubnetId $vnet.Subnets[1].Id
```

Get-Credential を使用して、VM の管理者アカウントに必要なユーザー名とパスワードを設定します。

```azurepowershell-interactive
$cred = Get-Credential
```

*myBackendVM* を作成します。

```azurepowershell-interactive
$backendVM = New-AzureRmVMConfig `
  -VMName myBackendVM `
  -VMSize Standard_D1
$backendVM = Set-AzureRmVMOperatingSystem `
  -VM $backendVM `
  -Windows `
  -ComputerName myBackendVM `
  -Credential $cred `
  -ProvisionVMAgent `
  -EnableAutoUpdate
$backendVM = Set-AzureRmVMSourceImage `
  -VM $backendVM `
  -PublisherName MicrosoftSQLServer `
  -Offer SQL2016SP1-WS2016 `
  -Skus Enterprise `
  -Version latest
$backendVM = Set-AzureRmVMOSDisk `
  -VM $backendVM `
  -Name myBackendOSDisk `
  -DiskSizeInGB 128 `
  -CreateOption FromImage `
  -Caching ReadWrite
$backendVM = Add-AzureRmVMNetworkInterface `
  -VM $backendVM `
  -Id $backendNic.Id
New-AzureRmVM `
  -ResourceGroupName myRGNetwork `
  -Location EastUS `
  -VM $backendVM
```

使用されているイメージには SQL Server がインストールされていますが、このチュートリアルでは使用しません。 これが含まれているのは、Web トラフィックを処理する VM と、データベースの管理を処理する VM の構成の方法を示すためです。

## <a name="next-steps"></a>次のステップ

このチュートリアルでは、仮想マシンとの関連で Azure ネットワークを作成し、セキュリティで保護しました。 

> [!div class="checklist"]
> * 仮想ネットワークとサブネットの作成
> * パブリック IP アドレスの作成
> * フロントエンド VM の作成
> * ネットワーク トラフィックのセキュリティ保護
> * バックエンド VM の作成

次のチュートリアルに進み、仮想マシンのデータを Azure Backup で監視する方法を学習してください。

> [!div class="nextstepaction"]
> [Azure の Windows 仮想マシンのバックアップ](./tutorial-backup-vms.md)

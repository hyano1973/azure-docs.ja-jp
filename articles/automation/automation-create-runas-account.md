---
title: "Azure Automation 実行アカウントを作成する | Microsoft Docs"
description: "この記事では、PowerShell またはポータルを使用して、Automation アカウントを更新し実行アカウントを作成する方法について説明します。"
services: automation
documentationcenter: 
author: georgewallace
manager: carmonm
editor: 
ms.assetid: 
ms.service: automation
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/27/2017
ms.author: magoedte
ms.openlocfilehash: 74d363be48972b40ba6a50b845acea78e1b5cc20
ms.sourcegitcommit: 9292e15fc80cc9df3e62731bafdcb0bb98c256e1
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/10/2018
---
# <a name="update-your-automation-account-authentication-with-run-as-accounts"></a>Automation アカウントの認証を実行アカウントで更新する 
次の条件に該当する場合は、Azure Portal または PowerShell を使用して既存の Automation アカウントを更新できます。

* Automation アカウントは作成済みであるものの、実行アカウントの作成を拒否した場合。
* Resource Manager リソースを管理するための Automation アカウントを既に所有しており、そのアカウントを更新する形で Runbook 認証用の実行アカウントを追加する場合。
* クラシック リソースを管理するための Automation アカウントを既に所有しており、それを、新しいアカウントを作成する代わりにクラシック実行アカウントを使用するように更新し、Runbook と資産をそのアカウントに移行する場合。   

Automation アカウントに次の項目が作成されます。

**実行アカウントの場合:**

* 自己署名証明書を含む Azure AD アプリケーションを作成し、このアプリケーションの Azure AD におけるサービス プリンシパル アカウントを作成します。また、現在のサブスクリプションにおける、このアカウントの共同作成者ロールが割り当てられます。 この設定は所有者ロールや他の任意のロールに変更できます。 詳細については、「[Azure Automation におけるロールベースのアクセス制御](automation-role-based-access-control.md)」を参照してください。
* 指定された Automation アカウントに *AzureRunAsCertificate* という名前の Automation 証明書資産を作成します。 この証明書資産には、Azure AD アプリケーションによって使用される証明書の秘密キーが格納されます。
* 指定された Automation アカウントに *AzureRunAsConnection* という名前の Automation 接続資産を作成します。 この接続資産には、アプリケーション ID、テナント ID、サブスクリプション ID、証明書の拇印が格納されます。

**クラシック実行アカウントの場合:**

* 指定された Automation アカウントに *AzureClassicRunAsCertificate* という名前の Automation 証明書資産を作成します。 この証明書資産には、管理証明書によって使用される証明書の秘密キーが格納されます。
* 指定された Automation アカウントに *AzureClassicRunAsConnection* という名前の Automation 接続資産を作成します。 この接続資産には、サブスクリプション名、サブスクリプション ID、証明書の資産名が格納されます。

## <a name="prerequisites"></a>前提条件
[PowerShell を使用して実行アカウントを作成する](#create-run-as-account-using-powershell)場合、このプロセスでは以下のものが必要です。

* Azure Resource Manager モジュール 3.4.1 以降がインストールされた Windows 10 と Windows Server 2016。 PowerShell スクリプトは以前のバージョンの Windows をサポートしていません。
* Azure PowerShell 1.0 以降。 PowerShell 1.0 リリースについては、[Azure PowerShell のインストールと構成方法](/powershell/azureps-cmdlets-docs)に関するページを参照してください。
* Automation アカウント。このアカウントは、*–AutomationAccountName* パラメーターと *-ApplicationDisplayName* パラメーターの値として参照されます。

スクリプトの必須パラメーターである *SubscriptionID*、*ResourceGroup*、*AutomationAccountName* の値を取得するには、次の手順に従います。

1. Azure ポータルで、左下隅にある **[その他のサービス]** をクリックします。 リソースの一覧で、「**Automation**」と入力します。 入力を始めると、入力内容に基づいて、一覧がフィルター処理されます。 **[Automation アカウント]**を選択します。
2. [Automation アカウント] ページで自分の Automation アカウントを選択し、**[アカウント設定]** で **[プロパティ]** を選択します。  
3. **[プロパティ]** ページに表示される値をメモします。<br><br> ![Automation アカウントの "プロパティ" ブレード](media/automation-create-runas-account/automation-account-properties.png)  

### <a name="required-permissions-to-update-your-automation-account"></a>Automation アカウントを更新するために必要なアクセス許可
Automation アカウントを更新するには、このトピックの作業で要求される以下に記載した特権とアクセス許可が必要となります。   
 
* ご利用の AD ユーザー アカウントが、「[Azure Automation におけるロールベースのアクセス制御](automation-role-based-access-control.md#contributor-role-permissions)」の記事に記載されている Microsoft.Automation リソースの共同作成者ロールに相当するアクセス許可を備えたロールに追加されている必要があります。  
* **[ユーザー設定]** ページで Azure AD テナントの **[ユーザーはアプリケーションを登録できる]** オプションが **[はい]** に設定されている場合は、Azure AD テナントの管理者以外のユーザーが[AD アプリケーションを登録](../azure-resource-manager/resource-group-create-service-principal-portal.md#check-azure-subscription-permissions)できます。 [アプリの登録] が **[いいえ]** に設定されている場合、この操作を行うユーザーは、Azure AD の全体管理者であることが必要です。

サブスクリプションの Active Directory インスタンスのメンバーになっていない状態で、サブスクリプションの全体管理者/共同管理者ロールに追加された場合、Active Directory にゲストとして追加されることになります。 この場合、"…を作成するためのアクセス許可がありません" という 警告が **[Automation アカウントの追加]** ブレードに表示されます。 先に全体管理者/共同管理者ロールに追加されていたユーザーは、サブスクリプションの Active Directory インスタンスから削除した後、Active Directory の完全なユーザーとして再度追加できます。 このような状況を検証するには、Azure Portal の **[Azure Active Directory]** ウィンドウで、**[ユーザーとグループ]**、**[すべてのユーザー]**、特定のユーザー、**[プロファイル]** の順に選択します。 ユーザーのプロファイルの下部にある **[ユーザー タイプ]** 属性の値は、**[ゲスト]** と一致しないようにする必要があります。

## <a name="create-run-as-account-from-the-portal"></a>ポータルで実行アカウントを作成する
このセクションでは、以下の手順に従って、Azure Portal で Azure Automation アカウントを更新します。  実行アカウントとクラシック実行アカウントをそれぞれ作成します。 クラシック リソースを管理する必要がない場合は、Azure 実行アカウントのみを作成できます。  

1. サブスクリプション管理ロールのメンバーかつサブスクリプションの共同管理者であるアカウントを使用して、Azure Portal にサインインします。
2. Azure ポータルで、左下隅にある **[その他のサービス]** をクリックします。 リソースの一覧で、「**Automation**」と入力します。 入力を始めると、入力内容に基づいて、一覧がフィルター処理されます。 **[Automation アカウント]**を選択します。
3. **[Automation アカウント]** ページで、Automation アカウントの一覧からご使用の Automation アカウントを選択します。
4. 左側のウィンドウの **[アカウント設定]** セクションで、**[実行アカウント]** を選択します。  
5. 必要なアカウントに応じて、**[Azure 実行アカウント]** または **[Azure クラシック実行アカウント]** を選択します。  **[Azure 実行アカウントを追加する]** または **[Azure クラシック実行アカウントを追加する]** のどちらかを選択すると、ウィンドウが表示されます。概要情報を確認してから、**[作成]** をクリックして実行アカウントの作成を進めます。  
6. Azure によって実行アカウントが作成されている間、メニューの **[通知]** で進行状況を追跡できます。  アカウントの作成中であることを示すバナーも表示されます。  このプロセスが完了するまでに数分かかることがあります。  

## <a name="create-run-as-account-using-powershell-script"></a>PowerShell スクリプトを使用して実行アカウントを作成する
この PowerShell スクリプトでは、以下の構成がサポートされます。

* 自己署名証明書を使用して実行アカウントを作成する。
* 自己署名証明書を使用して実行アカウントとクラシック実行アカウントを作成する。
* エンタープライズ証明機関 (CA) によって発行された証明書を使用して実行アカウントとクラシック実行アカウントを作成する。
* Azure Government クラウドで自己署名証明書を使用して実行アカウントとクラシック実行アカウントを作成する。

>[!NOTE]
> クラシック実行アカウントを作成するオプションを選択した場合、スクリプトの実行後に、Automation アカウントが作成されたサブスクリプションの管理ストアに公開証明書 (.cer ファイル名拡張子) をアップロードしてください。
> 

1. ご使用のコンピューターに次のスクリプトを保存します。 この例では、*New-RunAsAccount.ps1* というファイル名で保存します。

         #Requires -RunAsAdministrator
         Param (
         [Parameter(Mandatory=$true)]
         [String] $ResourceGroup,

         [Parameter(Mandatory=$true)]
         [String] $AutomationAccountName,

         [Parameter(Mandatory=$true)]
         [String] $ApplicationDisplayName,

         [Parameter(Mandatory=$true)]
         [String] $SubscriptionId,

         [Parameter(Mandatory=$true)]
         [Boolean] $CreateClassicRunAsAccount,

         [Parameter(Mandatory=$true)]
         [String] $SelfSignedCertPlainPassword,

         [Parameter(Mandatory=$false)]
         [String] $EnterpriseCertPathForRunAsAccount,

         [Parameter(Mandatory=$false)]
         [String] $EnterpriseCertPlainPasswordForRunAsAccount,

         [Parameter(Mandatory=$false)]
         [String] $EnterpriseCertPathForClassicRunAsAccount,

         [Parameter(Mandatory=$false)]
         [String] $EnterpriseCertPlainPasswordForClassicRunAsAccount,

         [Parameter(Mandatory=$false)]
         [ValidateSet("AzureCloud","AzureUSGovernment")]
         [string]$EnvironmentName="AzureCloud",

         [Parameter(Mandatory=$false)]
         [int] $SelfSignedCertNoOfMonthsUntilExpired = 12
         )

         function CreateSelfSignedCertificate([string] $certificateName, [string] $selfSignedCertPlainPassword,
                                        [string] $certPath, [string] $certPathCer, [string] $selfSignedCertNoOfMonthsUntilExpired ) {
         $Cert = New-SelfSignedCertificate -DnsName $certificateName -CertStoreLocation cert:\LocalMachine\My `
             -KeyExportPolicy Exportable -Provider "Microsoft Enhanced RSA and AES Cryptographic Provider" `
             -NotAfter (Get-Date).AddMonths($selfSignedCertNoOfMonthsUntilExpired) -HashAlgorithm SHA256

         $CertPassword = ConvertTo-SecureString $selfSignedCertPlainPassword -AsPlainText -Force
         Export-PfxCertificate -Cert ("Cert:\localmachine\my\" + $Cert.Thumbprint) -FilePath $certPath -Password $CertPassword -Force | Write-Verbose
         Export-Certificate -Cert ("Cert:\localmachine\my\" + $Cert.Thumbprint) -FilePath $certPathCer -Type CERT | Write-Verbose
         }

         function CreateServicePrincipal([System.Security.Cryptography.X509Certificates.X509Certificate2] $PfxCert, [string] $applicationDisplayName) {  
         $keyValue = [System.Convert]::ToBase64String($PfxCert.GetRawCertData())
         $keyId = (New-Guid).Guid

         $startDate = Get-Date
         $endDate = (Get-Date $PfxCert.GetExpirationDateString()).AddDays(-1)
         
         #Create an Azure AD application, AD App Credential, AD ServicePrincipal
         $Application = New-AzureRmADApplication -DisplayName $ApplicationDisplayName -HomePage ("http://" + $applicationDisplayName) -IdentifierUris ("http://" + $keyId) 
         $ApplicationCredential = New-AzureRmADAppCredential -ApplicationId $Application.ApplicationId -CertValue $keyValue -StartDate $startDate -EndDate $endDate 
         $ServicePrincipal = New-AzureRMADServicePrincipal -ApplicationId $Application.ApplicationId 
         $GetServicePrincipal = Get-AzureRmADServicePrincipal -ObjectId $ServicePrincipal.Id

         # Sleep here for a few seconds to allow the service principal application to become active (ordinarily takes a few seconds)
         Sleep -s 15
         $NewRole = New-AzureRMRoleAssignment -RoleDefinitionName Contributor -ServicePrincipalName $Application.ApplicationId -ErrorAction SilentlyContinue
         $Retries = 0;
         While ($NewRole -eq $null -and $Retries -le 6)
         {
             Sleep -s 10
             New-AzureRMRoleAssignment -RoleDefinitionName Contributor -ServicePrincipalName $Application.ApplicationId | Write-Verbose -ErrorAction SilentlyContinue
             $NewRole = Get-AzureRMRoleAssignment -ServicePrincipalName $Application.ApplicationId -ErrorAction SilentlyContinue
             $Retries++;
         }
             return $Application.ApplicationId.ToString();
         }

         function CreateAutomationCertificateAsset ([string] $resourceGroup, [string] $automationAccountName, [string] $certifcateAssetName,[string] $certPath, [string] $certPlainPassword, [Boolean] $Exportable) {
         $CertPassword = ConvertTo-SecureString $certPlainPassword -AsPlainText -Force   
         Remove-AzureRmAutomationCertificate -ResourceGroupName $resourceGroup -AutomationAccountName $automationAccountName -Name $certifcateAssetName -ErrorAction SilentlyContinue
         New-AzureRmAutomationCertificate -ResourceGroupName $resourceGroup -AutomationAccountName $automationAccountName -Path $certPath -Name $certifcateAssetName -Password $CertPassword -Exportable:$Exportable  | write-verbose
         }

         function CreateAutomationConnectionAsset ([string] $resourceGroup, [string] $automationAccountName, [string] $connectionAssetName, [string] $connectionTypeName, [System.Collections.Hashtable] $connectionFieldValues ) {
         Remove-AzureRmAutomationConnection -ResourceGroupName $resourceGroup -AutomationAccountName $automationAccountName -Name $connectionAssetName -Force -ErrorAction SilentlyContinue
         New-AzureRmAutomationConnection -ResourceGroupName $ResourceGroup -AutomationAccountName $automationAccountName -Name $connectionAssetName -ConnectionTypeName $connectionTypeName -ConnectionFieldValues $connectionFieldValues
         }

         Import-Module AzureRM.Profile
         Import-Module AzureRM.Resources

         $AzureRMProfileVersion= (Get-Module AzureRM.Profile).Version
         if (!(($AzureRMProfileVersion.Major -ge 3 -and $AzureRMProfileVersion.Minor -ge 4) -or ($AzureRMProfileVersion.Major -gt 3)))
         {
             Write-Error -Message "Please install the latest Azure PowerShell and retry. Relevant doc url : https://docs.microsoft.com/powershell/azureps-cmdlets-docs/ "
             return
         }

         Login-AzureRmAccount -Environment $EnvironmentName 
         $Subscription = Select-AzureRmSubscription -SubscriptionId $SubscriptionId

         # Create a Run As account by using a service principal
         $CertifcateAssetName = "AzureRunAsCertificate"
         $ConnectionAssetName = "AzureRunAsConnection"
         $ConnectionTypeName = "AzureServicePrincipal"

         if ($EnterpriseCertPathForRunAsAccount -and $EnterpriseCertPlainPasswordForRunAsAccount) {
         $PfxCertPathForRunAsAccount = $EnterpriseCertPathForRunAsAccount
         $PfxCertPlainPasswordForRunAsAccount = $EnterpriseCertPlainPasswordForRunAsAccount
         } else {
            $CertificateName = $AutomationAccountName+$CertifcateAssetName
            $PfxCertPathForRunAsAccount = Join-Path $env:TEMP ($CertificateName + ".pfx")
            $PfxCertPlainPasswordForRunAsAccount = $SelfSignedCertPlainPassword
            $CerCertPathForRunAsAccount = Join-Path $env:TEMP ($CertificateName + ".cer")
            CreateSelfSignedCertificate $CertificateName $PfxCertPlainPasswordForRunAsAccount $PfxCertPathForRunAsAccount $CerCertPathForRunAsAccount $SelfSignedCertNoOfMonthsUntilExpired
         }

         # Create a service principal
         $PfxCert = New-Object -TypeName System.Security.Cryptography.X509Certificates.X509Certificate2 -ArgumentList @($PfxCertPathForRunAsAccount, $PfxCertPlainPasswordForRunAsAccount)
         $ApplicationId=CreateServicePrincipal $PfxCert $ApplicationDisplayName

         # Create the Automation certificate asset
         CreateAutomationCertificateAsset $ResourceGroup $AutomationAccountName $CertifcateAssetName $PfxCertPathForRunAsAccount $PfxCertPlainPasswordForRunAsAccount $true

         # Populate the ConnectionFieldValues
         $SubscriptionInfo = Get-AzureRmSubscription -SubscriptionId $SubscriptionId
         $TenantID = $SubscriptionInfo | Select TenantId -First 1
         $Thumbprint = $PfxCert.Thumbprint
         $ConnectionFieldValues = @{"ApplicationId" = $ApplicationId; "TenantId" = $TenantID.TenantId; "CertificateThumbprint" = $Thumbprint; "SubscriptionId" = $SubscriptionId}

         # Create an Automation connection asset named AzureRunAsConnection in the Automation account. This connection uses the service principal.
         CreateAutomationConnectionAsset $ResourceGroup $AutomationAccountName $ConnectionAssetName $ConnectionTypeName $ConnectionFieldValues

         if ($CreateClassicRunAsAccount) {
              # Create a Run As account by using a service principal
              $ClassicRunAsAccountCertifcateAssetName = "AzureClassicRunAsCertificate"
              $ClassicRunAsAccountConnectionAssetName = "AzureClassicRunAsConnection"
              $ClassicRunAsAccountConnectionTypeName = "AzureClassicCertificate "
              $UploadMessage = "Please upload the .cer format of #CERT# to the Management store by following the steps below." + [Environment]::NewLine +
                      "Log in to the Microsoft Azure portal (https://portal.azure.com) and select Subscriptions -> Management Certificates." + [Environment]::NewLine +
                      "Then click Upload and upload the .cer format of #CERT#"

               if ($EnterpriseCertPathForClassicRunAsAccount -and $EnterpriseCertPlainPasswordForClassicRunAsAccount ) {
               $PfxCertPathForClassicRunAsAccount = $EnterpriseCertPathForClassicRunAsAccount
               $PfxCertPlainPasswordForClassicRunAsAccount = $EnterpriseCertPlainPasswordForClassicRunAsAccount
               $UploadMessage = $UploadMessage.Replace("#CERT#", $PfxCertPathForClassicRunAsAccount)
         } else {
               $ClassicRunAsAccountCertificateName = $AutomationAccountName+$ClassicRunAsAccountCertifcateAssetName
               $PfxCertPathForClassicRunAsAccount = Join-Path $env:TEMP ($ClassicRunAsAccountCertificateName + ".pfx")
               $PfxCertPlainPasswordForClassicRunAsAccount = $SelfSignedCertPlainPassword
               $CerCertPathForClassicRunAsAccount = Join-Path $env:TEMP ($ClassicRunAsAccountCertificateName + ".cer")
               $UploadMessage = $UploadMessage.Replace("#CERT#", $CerCertPathForClassicRunAsAccount)
               CreateSelfSignedCertificate $ClassicRunAsAccountCertificateName $PfxCertPlainPasswordForClassicRunAsAccount $PfxCertPathForClassicRunAsAccount $CerCertPathForClassicRunAsAccount $SelfSignedCertNoOfMonthsUntilExpired
         }

         # Create the Automation certificate asset
         CreateAutomationCertificateAsset $ResourceGroup $AutomationAccountName $ClassicRunAsAccountCertifcateAssetName $PfxCertPathForClassicRunAsAccount $PfxCertPlainPasswordForClassicRunAsAccount $false

         # Populate the ConnectionFieldValues
         $SubscriptionName = $subscription.Subscription.Name
         $ClassicRunAsAccountConnectionFieldValues = @{"SubscriptionName" = $SubscriptionName; "SubscriptionId" = $SubscriptionId; "CertificateAssetName" = $ClassicRunAsAccountCertifcateAssetName}

         # Create an Automation connection asset named AzureRunAsConnection in the Automation account. This connection uses the service principal.
         CreateAutomationConnectionAsset $ResourceGroup $AutomationAccountName $ClassicRunAsAccountConnectionAssetName $ClassicRunAsAccountConnectionTypeName $ClassicRunAsAccountConnectionFieldValues

         Write-Host -ForegroundColor red $UploadMessage
         }

2. コンピューターの**スタート**画面から、昇格されたユーザー権限で **Windows PowerShell** を起動します。
3. 昇格されたコマンドライン シェルから、手順 1. で作成したスクリプトがあるフォルダーに移動します。  
4. 必要な構成に応じたパラメーター値を使ってスクリプトを実行します。

    **自己署名証明書を使用して実行アカウントを作成する**  
    `.\New-RunAsAccount.ps1 -ResourceGroup <ResourceGroupName> -AutomationAccountName <NameofAutomationAccount> -SubscriptionId <SubscriptionId> -ApplicationDisplayName <DisplayNameofAADApplication> -SelfSignedCertPlainPassword <StrongPassword> -CreateClassicRunAsAccount $false`

    **自己署名証明書を使用して実行アカウントとクラシック実行アカウントを作成する**  
    `.\New-RunAsAccount.ps1 -ResourceGroup <ResourceGroupName> -AutomationAccountName <NameofAutomationAccount> -SubscriptionId <SubscriptionId> -ApplicationDisplayName <DisplayNameofAADApplication> -SelfSignedCertPlainPassword <StrongPassword> -CreateClassicRunAsAccount $true`

    **エンタープライズ証明書を使用して実行アカウントとクラシック実行アカウントを作成する**  
    `.\New-RunAsAccount.ps1 -ResourceGroup <ResourceGroupName> -AutomationAccountName <NameofAutomationAccount> -SubscriptionId <SubscriptionId> -ApplicationDisplayName <DisplayNameofAADApplication>  -SelfSignedCertPlainPassword <StrongPassword> -CreateClassicRunAsAccount $true -EnterpriseCertPathForRunAsAccount <EnterpriseCertPfxPathForRunAsAccount> -EnterpriseCertPlainPasswordForRunAsAccount <StrongPassword> -EnterpriseCertPathForClassicRunAsAccount <EnterpriseCertPfxPathForClassicRunAsAccount> -EnterpriseCertPlainPasswordForClassicRunAsAccount <StrongPassword>`

    **Azure Government クラウドで自己署名証明書を使用して実行アカウントとクラシック実行アカウントを作成する**  
    `.\New-RunAsAccount.ps1 -ResourceGroup <ResourceGroupName> -AutomationAccountName <NameofAutomationAccount> -SubscriptionId <SubscriptionId> -ApplicationDisplayName <DisplayNameofAADApplication> -SelfSignedCertPlainPassword <StrongPassword> -CreateClassicRunAsAccount $true  -EnvironmentName AzureUSGovernment`

    > [!NOTE]
    > スクリプトの実行後、Azure に対する認証が求められます。 サブスクリプション管理ロールのメンバーかつサブスクリプションの共同管理者であるアカウントを使用してサインインしてください。
    >
    >

スクリプトが正常に実行されたら、次のことを確認してください。
* 自己署名公開証明書 (.cer ファイル) を使ってクラシック実行アカウントを作成した場合、コンピューターの一時ファイル フォルダーに証明書が作成されて保存されます。PowerShell セッションの実行に使用したユーザー プロファイル下の *%USERPROFILE%\AppData\Local\Temp* を探してください。
* エンタープライズ公開証明書 (.cer ファイル) を使ってクラシック実行アカウントを作成した場合は、その証明書を使用します。 [Management API 証明書を Azure Portal にアップロード](../azure-api-management-certs.md)する手順を実行した後、[Azure クラシック デプロイメント リソースで認証を行うサンプル コード](automation-verify-runas-authentication.md#classic-run-as-authentication)を使用して、クラシック デプロイメント リソースに対する資格情報の構成を確認します。 
* クラシック実行アカウントを "*作成しなかった場合*" は、「[サービス管理リソースで認証を行うサンプル コード](automation-verify-runas-authentication.md#automation-run-as-authentication)」を参照して、Resource Manager リソースに対する認証を行い、資格情報の構成を確認します。

## <a name="next-steps"></a>次の手順
* サービス プリンシパルの詳細については、 [アプリケーション オブジェクトおよびサービス プリンシパル オブジェクト](../active-directory/active-directory-application-objects.md)に関するページを参照してください。
* 証明書と Azure サービスの詳細については、「[Azure Cloud Services の証明書の概要](../cloud-services/cloud-services-certs-create.md)」を参照してください。

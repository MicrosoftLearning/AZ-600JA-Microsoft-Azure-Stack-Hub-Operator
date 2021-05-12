---
lab:
    title: 'Lab: Azure サブスクリプションを使用して Azure Stack Hub を登録する'
    module: 'モジュール 3: データ センター統合を実装する'
---

# ラボ - Azure サブスクリプションを使用して Azure Stack Hub を登録する
# 受講生用ラボ マニュアル

## ラボの依存関係

- なし

## 推定所要時間

30 分

## ラボ シナリオ

あなたは Azure Stack Hub 環境のオペレーターとして、Azure サブスクリプションを使用して Azure Stack Hub を登録することで、Azure Marketplace 項目をダウンロードし、Azure へのデータ レポートを設定する必要があります。 

## 目標

このラボを終了すると、次のことができるようになります。

- Azure サブスクリプションを使用して Azure Stack Hub を登録する 

## ラボ環境 

このラボでは、Active Directory フェデレーション サービス (AD FS) (ID プロバイダーとしてバックアップされた Active Directory) に統合された ADSK インスタンスを使用します。 

ラボ環境は次のように構成されています。

- 次のアクセス ポイントを持つ、**AzS-HOST1** サーバーで実行されている ASDK デプロイ。

  - 管理者ポータル: https://adminportal.local.azurestack.external
  - 管理者 ARM エンドポイント: https://adminmanagement.local.azurestack.external
  - ユーザー ポータル: https://portal.local.azurestack.external
  - ユーザー ARM エンドポイント: https://management.local.azurestack.external

- 管理ユーザー:

  - ASDK クラウド オペレーターのユーザー名: **CloudAdmin@azurestack.local**
  - ASDK クラウド オペレーターのパスワード: **Pa55w.rd1234**
  - ASDK ホスト管理者のユーザー名: **AzureStackAdmin@azurestack.local**
  - ASDK ホスト管理者のパスワード: **Pa55w.rd1234**

このラボでは、PowerShell を介して Azure Stack Hub を管理するために必要なソフトウェアをインストールします。 


### 演習 1: Azure サブスクリプションを使用して Azure Stack Hub を登録する

この演習では、Azure サブスクリプションを使用して Azure Stack Hub を登録します。

1. Azure Stack Hub リソース プロバイダーを登録する
1. Azure Stack Hub の登録を行う
1. Azure Stack Hub の登録を検証する

#### タスク 1: Azure Stack Hub リソース プロバイダーを登録する

このタスクでは、次のことを行います。

- Azure サブスクリプションのもとで対象の Azure Stack Hub リソース プロバイダーを登録する

1. 必要であれば、次の資格情報を使用して **AzS-HOST1** にサインインします。

    - ユーザー名: **AzureStackAdmin@azurestack.local**
    - パスワード: **Pa55w.rd1234**

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、管理者として PowerShell 7 を起動します。
1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」ウィンドウで以下のコマンドを実行して、Azure Stack Hub 用の PowerShell Az モジュールをインストールします。

    ```powershell
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    Install-Module -Name Az.BootStrapper -Force -AllowPrerelease -AllowClobber
    Install-AzProfile -Profile 2019-03-01-hybrid -Force
    Install-Module -Name AzureStack -RequiredVersion 2.0.2-preview -AllowPrerelease
    ```

1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」ウィンドウで以下のコマンドを実行して、このラボで使用する Azure サブスクリプションを認証します。

    ```powershell
    Connect-AzAccount -EnvironmentName 'AzureCloud'
    ```

1. サインインするよう求められたら、このラボで使用する Azure サブスクリプションのもとで共同作成者ロールを持つ、Azure Active Directory (Azure AD) ユーザーの資格情報を使用してサインインします。
1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」ウィンドウで以下のコマンドを実行して、Azure Stack リソース プロバイダーを登録します。

    ```powershell
    Register-AzResourceProvider -ProviderNamespace Microsoft.AzureStack
    ```

1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」ウィンドウで以下のコマンドを実行して、登録が完了したかどうかを確認します。

    ```powershell
    Get-AzResourceProvider -ProviderNamespace Microsoft.AzureStack | Where-Object {$_.RegistrationState -eq 'Registered'}
    ```

    >**注**: 登録が完了するまで待ちます。**Get-AzResourceProvider** コマンドレットを再実行して、登録の状態を確認します。


#### タスク 2: Azure Stack Hub の登録を行う

このタスクでは、次のことを行います。

- Azure Stack Hub の登録を行う

1. **AzSHOST-1** へのリモート デスクトップ セッション内で、「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」ウィンドウから以下のコマンドを実行して、特権エンドポイント (PEP) セッションを確立します。

    ```powershell
    Enter-PSSession -ComputerName AzS-ERCS01 -ConfigurationName PrivilegedEndpoint
    ```

1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」ウィンドウから、特権エンドポイントへの PowerShell リモート セッション内で以下のコマンドを実行して、Azure Stack Hub スタンプの概要構成を表示します。

    ```powershell
    Get-AzureStackStampInformation
    ```

1. 前のステップで実行したコマンドの出力で、「**CloudId**」プロパティの値を確認し、記録しておきます。
1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」ウィンドウで、特権エンドポイントへの PowerShell リモート セッション内から以下のコマンドを実行してセッションを終了します。

    ```powershell
    Exit-PSSession
    ```

    >**注**: 通常は、**Exit_PSSession** を使用して特権エンドポイントへのセッションを終了することは避けてください。代わりに **Close-PrivilegedEndpoint** コマンドレットを使用しますが、そのためにはファイル共有を設定してセッション トランスクリプト ログをホストしなくてはなりません。この演習では内容を簡素化するため、この正規の手順は実行しません。

    >**注**: スタンプの「**Cloud ID**」プロパティの値は、Azure Stack Hub 管理者ポータルの「**local \| プロパティ**」ブレードでも確認できます。

1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」ウィンドウで以下のコマンドを実行して、Azure Stack インスタンスをターゲットとする Azure Stack PowerShell 環境を登録します (`[cloud_ID]` プレースホルダーを、**Get-AzureStackStampInformation** コマンドレットの出力で確認した **CloudId** プロパティの値に必ず置き換えてください)。

    ```powershell
    Import-Module .\Registration\RegisterWithAzure.psm1
    $RegistrationName = "[cloud_ID]"
    Set-AzsRegistration `
       -PrivilegedEndpointCredential $adminCredentials `
       -PrivilegedEndpoint 'AzS-ERCS01' `
       -BillingModel 'Development' `
       -RegistrationName $RegistrationName `
       -UsageReportingEnabled:$true
    ```

1. サインインするよう求められたら、Azure サブスクリプションの共同作成者ロールを持つ Azure ユーザー アカウントを使用してサインインします。

    > **注:** 登録が完了するまで待ちます。登録には 20 分ほどかかる場合があります。


#### タスク 3: Azure Stack Hub の登録を検証する

このタスクでは、次のことを行います。

- Azure サブスクリプションを使用して Azure Stack Hub の登録を確認する

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、Web ブラウザーを開いて画面に [Azure Stack Hub 管理者ポータル](https://adminportal.local.azurestack.external/)を表示させ、CloudAdmin@azurestack.local としてサインインします。
1. Web ブラウザーで Azure Stack Hub 管理者ポータルを表示し、「**ダッシュボード**」ブレードで「**リージョンの管理**」タイルを選択します。
1. 「**ローカル**」ブレードで「**プロパティ**」を選択します。 
1. 「**ローカル \| プロパティ**」ブレードで、「**登録の状態**」が「**登録済み**」となっていることを確認します。 

    > **注:** 状態は「**登録済み**」、「**未登録**」、「**期限切れ**」のいずれかとなります。

>**確認**: この演習では、Azure サブスクリプションを使用して Azure Stack Hub を登録しました。

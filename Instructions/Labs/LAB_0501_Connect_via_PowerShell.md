---
lab:
    title: 'ラボ: PowerShell を介して Azure Stack Hub に接続する'
    module: 'モジュール 5: インフラストラクチャを管理する'
---

# ラボ - PowerShell を介して Azure Stack Hub に接続する
# 受講生用ラボ マニュアル

## ラボの依存関係

- なし

## 推定所要時間

30 分

## ラボ シナリオ

あなたは Azure Stack Hub 環境のオペレーターとして、PowerShell を使用して作業環境を管理できるようにしておかなければなりません。また、ときにはユーザーとして PowerShell を介して Azure Stack Hub に接続できるようにしておくことも必要となります。 

## 目標

このラボを終了すると、次のことができるようになります。

- PowerShell を介して ASDK のオペレーター環境とユーザー環境に接続する

## ラボ環境

このラボでは、Active Directory フェデレーション サービス (AD FS) (ID プロバイダーとしてバックアップされた Active Directory) に統合された ADSK インスタンスを使用します。 

>**注**: Azure Active Directory (Azure AD) に統合された Azure Stack Hub に接続する方法については、「[PowerShell を使用して Azure Stack Hub に接続する](https://docs.microsoft.com/ja-jp/azure-stack/operator/azure-stack-powershell-configure-admin?view=azs-2008&tabs=az1%2Caz2%2Caz3#connect-with-azure-ad)」を参照してください。

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

## 手順

### 演習 1: Azure PowerShell を介して ASDK に接続する

この演習では、ASDK ホストから PowerShell を介して、ASDK の 管理者 Admin ARM エンドポイントに接続します。

1. Azure Stack Hub と互換性のある Az PowerShell モジュールをインストールする
1. Azure Stack Hub ツールをダウンロードする
1. PowerShell を介して Azure Stack Hub のオペレーター環境を構成して接続する
1. PowerShell を介して Azure Stack Hub のユーザー環境を構成して接続する

#### タスク 1: Azure Stack Hub と互換性のある Azure PowerShell モジュールをインストールする

このタスクでは、次のことを行います。

- 既存の Azure モジュールと Az PowerShell モジュールをすべて削除する
- Azure Stack Hub と互換性のある Az PowerShell モジュールをインストールし、その前提条件を構成する
- Azure Stack Hub と互換性のある Az PowerShell モジュールをインストールする

>**注**: Azure Resource Manager (AzureRM) の PowerShell モジュールのバージョンはいずれも古くなっていますが、依然サポート対象となっています。ただし、Azure と Azure Stack Hub を操作するための PowerShell モジュールとして、現在 Az PowerShell モジュールの使用が推奨されています。 

1. 必要であれば、次の資格情報を使用して **AzS-HOST1** にサインインします。

    - ユーザー名: **AzureStackAdmin@azurestack.local**
    - パスワード: **Pa55w.rd1234**

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、管理者として **Windows PowerShell** を起動します。
1. 「**管理者: Windows PowerShell**」プロンプトで以下のコマンドを実行して、Azure PowerShell モジュールと Az PowerShell モジュールの既存のバージョンをすべて削除します。

    ```powershell
    Get-Module -Name Azure* -ListAvailable | Uninstall-Module -Force -Verbose -ErrorAction Continue
    Get-Module -Name Azs.* -ListAvailable | Uninstall-Module -Force -Verbose -ErrorAction Continue
    Get-Module -Name Az.* -ListAvailable | Uninstall-Module -Force -Verbose -ErrorAction Continue
    ```

    >**注**: 使用中のモジュールに関するエラー メッセージが表示された場合、Windows PowerShell セッションをいったん閉じてから再度開き、上記のコマンドを再実行します。

1. 「**管理者: Windows PowerShell**」プロンプトで以下のコマンドを実行して、名前が **Azure**、**Az**、**Azs** のいずれかから始まるすべてのフォルダーを、「**C:\\Program Files\\WindowsPowerShell\\Modules**」フォルダーと「**C:\\Users\\AzureStackAdmin\\Documents\\PowerShell\\Modules\\**」フォルダーから削除します。

    ```powershell
    Get-ChildItem -Path 'C:\Program Files\WindowsPowerShell\Modules' -Include 'Az*' -Recurse -Force | Remove-Item -Force -Recurse
    Get-ChildItem -Path 'C:\Users\AzureStackAdmin\Documents\PowerShell\Modules' -Include 'Az*' -Recurse -Force | Remove-Item -Force -Recurse
    ```

    >**注**: 使用中のモジュールに関するエラー メッセージが表示された場合、Windows PowerShell セッションをいったん閉じてから再度開き、上記のコマンドを再実行します。


1. **AzS-HOST1** へのリモート デスクトップ セッション内で、Microsoft Edge を起動し、「[PowerShell のリリース](https://github.com/PowerShell/PowerShell/releases/tag/v7.1.2)」ページに移動します。 
1. 「[PowerShell のリリース](https://github.com/PowerShell/PowerShell/releases/tag/v7.1.2)」ページで、PowerShell の最新リリースをダウンロードしてインストールします。 
1. **AzS-HOST1** へのリモート デスクトップ セッション内で、管理者として PowerShell 7 を起動します。
1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」プロンプトで以下のコマンドを実行して、PowerShell Gallery を信頼されたレポジトリとして構成します。

    ```powershell
    Set-PSRepository -Name 'PSGallery' -InstallationPolicy Trusted
    ```

1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」プロンプトで以下のコマンドを実行して、PowerShellGet をインストールします。

    ```powershell
    Install-Module PowerShellGet -MinimumVersion 2.2.3 -Force
    ```

    >**注**: 使用中のモジュールに関する警告メッセージはすべて無視してください。

1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」ウィンドウで以下のコマンドを実行して、Azure Stack Hub 用の PowerShell Az モジュールをインストールします。

    ```powershell
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    Install-Module -Name Az.BootStrapper -Force -AllowPrerelease -AllowClobber
    Install-AzProfile -Profile 2019-03-01-hybrid -Force
    Install-Module -Name AzureStack -RequiredVersion 2.0.2-preview -AllowPrerelease
    ```

    >**注**: すでに利用可能となっているコマンドに関するエラー メッセージはすべて無視してください。


#### タスク 2: Azure Stack Hub ツールをダウンロードする 

このタスクでは、次のことを行います。

- GitHub から Azure Stack Hub ツールをダウンロードする

1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」ウィンドウで以下のコマンドを実行して、Azure Stack ツールをダウンロードします。

    ```powershell
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    Set-Location -Path 'C:\'
    Invoke-WebRequest https://github.com/Azure/AzureStack-Tools/archive/az.zip -OutFile az.zip
    Expand-Archive az.zip -DestinationPath . -Force
    Set-Location -Path '\AzureStack-Tools-az'
    ```

    >**注**: この手順により、Azure Stack Hub ツールをホストするための GitHub リポジトリが含まれるアーカイブがローカル コンピューターにコピーされ、アーカイブが「**C:\\AzureStack-Tools-master**」フォルダーに展開されます。これらのツールには PowerShell モジュールが含まれており、これによって Azure Stack Hub 機能の特定、Azure Stack Hub VM インフラストラクチャとイメージの管理、Resource Manager ポリシーの構成、Azure Stack Hub の Azure への登録、Azure Stack Hub のデプロイ、Azure Stack Hub への接続、Azure Stack Hub テナントの管理、Azure Stack Hub Resource Manager テンプレートの検証といった、さまざまな機能が提供されます。 


#### タスク 3: PowerShell を介して Azure Stack Hub のオペレーター環境を構成して接続する

このタスクでは、次のことを行います。

- PowerShell を介して Azure Stack Hub のオペレーター環境を構成する
- PowerShell を介して Azure Stack Hub のオペレーター環境に接続する
- PowerShell を介して Azure Stack Hub のオペレーター環境への接続を検証する

1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」プロンプトで以下のコマンドを実行して、Azure Stack Hub オペレーターの PowerShell 環境を登録します。

    ```powershell
    Add-AzEnvironment -Name 'AzureStackAdmin' -ArmEndpoint 'https://adminmanagement.local.azurestack.external' `
       -AzureKeyVaultDnsSuffix adminvault.local.azurestack.external `
       -AzureKeyVaultServiceEndpointResourceId https://adminvault.local.azurestack.external
    ```

    >**注**: このコマンドによって次の出力が返されることを確認します。

    ```powershell
    Name            Resource Manager Url                              ActiveDirectory Authority
    ----            --------------------                              -------------------------
    AzureStackAdmin https://adminmanagement.local.azurestack.external https://adfs.local.azurestack.external/adfs/
    ```

1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」プロンプトで以下のコマンドを実行して、AzureStack\CloudAdmin 資格情報を使用して Azure Stack Hub オペレーターの PowerShell 環境にサインインします。

    ```powershell
    Connect-AzAccount -EnvironmentName 'AzureStackAdmin' -UseDeviceAuthentication
    ```

1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」ウィンドウで、生成されたメッセージを確認し、Web ブラウザーの画面を開いて「[adfs.local.azurestack.external](https://adfs.local.azurestack.external/adfs/oauth2/deviceauth)」ページに移動し、確認したメッセージに記されているコードを入力して「**続行**」をクリックします。 

1. サインインを求められたら、次の認証情報を使用してサインインします。

    - ユーザー名: **CloudAdmin@azurestack.local**
    - パスワード: **Pa55w.rd1234**

1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」ウィンドウに戻り、**CloudAdmin@azurestack.local** として認証されたことを確認します。
1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」プロンプトで以下のコマンドを実行して、Azure Stack Hub 管理者サブスクリプションを一覧表示します。

    ```powershell
    Get-AzSubscription
    ```

    >**注**: 出力に「**既定のプロバイダー サブスクリプション**」、「**計測サブスクリプション**」、「**従量課金サブスクリプション**」が含まれていることを確認します。

1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」プロンプトで以下のコマンドを実行して、対応する PowerShell 環境コンテキストを検証します。

    ```powershell
    Get-AzContext
    ```


#### タスク 4: PowerShell を介して Azure Stack Hub のユーザー環境を構成して接続する

このタスクでは、次のことを行います。

- PowerShell を介して Azure Stack Hub のユーザー環境を構成する
- PowerShell を介して Azure Stack Hub のユーザー環境に接続する
- PowerShell を介して Azure Stack Hub のユーザー環境への接続を検証する

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、PowerShell 7 を起動します。
1. 「**C:\Program Files\PowerShell\7\pwsh.exe**」プロンプトで以下のコマンドを実行して、Azure Stack Hub ユーザー環境をターゲットとする Azure Resource Manager 環境を登録します。

    ```powershell
    Add-AzEnvironment -Name 'AzureStackUser' -ArmEndpoint 'https://management.local.azurestack.external'
    ```

    >**注**: このコマンドによって次の出力が返されることを確認します。

    ```powershell
    Name            Resource Manager Url                              ActiveDirectory Authority
    ----            --------------------                              -------------------------
    AzureStackUser https://management.local.azurestack.external https://adfs.local.azurestack.external/adfs/
    ```

1. 「**C:\Program Files\PowerShell\7\pwsh.exe**」プロンプトで以下のコマンドを実行して、Azure Stack Hub PowerShell 環境にサインインします。

    ```powershell
    Connect-AzAccount -EnvironmentName 'AzureStackUser'
    ```

    >**注**: この操作により、Web ブラウザー アプリケーションが自動的に開き、認証するよう求められます。

1. サインインを求められたら、次の認証情報を使用してサインインします。

    - ユーザー名: **CloudAdmin@azurestack.local**
    - パスワード: **Pa55w.rd1234**

1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」プロンプトで以下のコマンドを実行して、Azure Stack Hub 管理者サブスクリプションを一覧表示します。

    ```powershell
    Get-AzSubscription
    ```

    >**注**: 出力に「**既定のプロバイダー サブスクリプション**」、「**計測サブスクリプション**」、「**従量課金サブスクリプション**」が**含まれていない**ことを確認します。

>**確認**: この演習では、PowerShell を使用して Azure Stack Hub のオペレーター環境とユーザー環境に接続しました。

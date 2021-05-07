---
lab:
    title: 'Lab: Azure Stack Hub で特権エンドポイントにアクセスする'
    module: 'モジュール 5: インフラストラクチャを管理する'
---

# ラボ - Azure Stack Hub で特権エンドポイントにアクセスする
# 受講生用ラボ マニュアル

## ラボの依存関係

- なし

## 推定所要時間

30 分

## ラボ シナリオ

あなたは Azure Stack Hub 環境のオペレーターとして、特権エンドポイントへのアクセス方法を特定する必要があります。

## 目標

このラボを終了すると、次のことができるようになります。

- Azure Stack Hub の特権エンドポイントにアクセスする 

## ラボ環境 

このラボでは、Active Directory フェデレーション サービス (AD FS) (ID プロバイダーとしてバックアップされた Active Directory) に統合された ADSK インスタンスを使用します。 

ラボ環境は、次のコンポーネントから構成されています。

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


### 演習 1: 特権エンドポイントを介して Azure Stack Hub を管理する

この演習では、特権エンドポイントへの PowerShell リモート セッションを確立し、リモート セッションを介してアクセス可能な Windows PowerShell コマンドレットを実行します。この演習は以下のタスクから構成されます。

1. Windows PowerShell を介して特権エンドポイントに接続する
1. 特権エンドポイントを介して利用できる機能について確認する
1. 特権エンドポイントへの接続を閉じ、セッション トランスクリプトを収集する

#### タスク 1: Windows PowerShell を介して特権エンドポイントに接続する

このタスクでは、次のことを行います。

- Windows PowerShell を介して特権エンドポイントに接続する

1. 必要であれば、次の資格情報を使用して **AzS-HOST1** にサインインします。

    - ユーザー名: **AzureStackAdmin@azurestack.local**
    - パスワード: **Pa55w.rd1234**

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、管理者として PowerShell ISE を起動します。
1. 「**管理者: Windows PowerShell ISE**」コンソールで以下のコマンドを実行して、特権エンドポイントが実行されているインフラストラクチャ VM の IP アドレスを特定します。 

    ```powershell
    $ipAddress = (Resolve-DnsName -Name AzS-ERCS01).IPAddress
    ```

1. (すべてのホストがすでに許可されていない場合) 「**管理者: Windows PowerShell ISE**」ウィンドウで以下のコマンドを実行して、特権エンドポイントが実行されているインフラストラクチャ VM の IP アドレスを、WinRM の信頼されたホストの一覧に追加します。

    ```powershell
    $trustedHosts = (Get-Item -Path WSMan:\localhost\Client\TrustedHosts).Value
    If ($trustedHosts -ne '*') {
	If ($trustedHosts -ne '') {
		$trustedHosts += ",ipAddress"
	} else {
	$trustedHosts = "$ipAddress"
	}
    }
    Set-Item WSMan:\localhost\Client\TrustedHosts -Value $TrustedHosts -Force
    ```

1. 「**管理者: Windows PowerShell ISE**」ウィンドウで以下のコマンドを実行して、Azure Stack Hub 管理者資格情報を変数に格納します。

    ```powershell
    $adminUserName = 'CloudAdmin@azurestack.local'
    $adminPassword = 'Pa55w.rd1234' | ConvertTo-SecureString -Force -AsPlainText
    $adminCredentials = New-Object PSCredential($adminUserName,$adminPassword)
    ```

1. 「**管理者: Windows PowerShell ISE**」ウィンドウで以下のコマンドを実行して、特権エンドポイントへの PowerShell リモート セッションを確立します。

    ```powershell
    Enter-PSSession -ComputerName $ipAddress -ConfigurationName PrivilegedEndpoint -Credential $adminCredentials
    ```

1. PowerShell リモート セッションが正常に確立されたことを確認します。これで「Windows PowerShell ISE」ウィンドウの「コンソール」ペインに、特権エンドポイントが実行されているインフラストラクチャ VM の IP アドレスから始まるプロンプトが表示されます (角かっこで囲まれています)。


#### タスク 2: 特権エンドポイントを介して利用できる機能について確認する

このタスクでは、次のことを行います。

- 特権エンドポイントを介して利用できる機能について確認する

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell ISE**」ウィンドウの PowerShell リモート セッションの「コンソール」ペインから以下のコマンドを実行して、利用可能なすべての PowerShell コマンドレットを特定します。

    ```powershell
    Get-Command
    ```

1. 「**管理者: Windows PowerShell ISE**」ウィンドウの PowerShell リモート セッションで以下のコマンドを実行して、現在のクラウド管理者ユーザー アカウントを特定します。

    ```powershell
    Get-CloudAdminUserList
    ```

    >**注**: この一覧には、CloudAdmin と AzureStackAdmin の 2 つのアカウントのみ表示されていなければなりません。

1. 「**管理者: Windows PowerShell ISE**」ウィンドウの PowerShell リモート セッションで以下のコマンドを実行して、Azure Stack Hub の更新準備について検証し、その結果を確認します。

    ```powershell
    Test-AzureStack -Group UpdateReadiness 
    ```

    >**注**: ASDK では更新がサポートされていないため、この手順はあくまでデモ目的であることに留意してください。 

    >**注**: **Test-AzureStack** 機能については、「[Azure Stack Hub システム状態の検証](https://docs.microsoft.com/ja-jp/azure-stack/operator/azure-stack-diagnostic-test?view=azs-2008)」を参照してください。

    >**注**: サポート シナリオでは、Microsoft サポート エンジニアが特権エンドポイント PowerShell セッションを昇格させて、Azure Stack Hub インフラストラクチャの内部にアクセスすることが必要となる場合があります。このプロセスは、特権エンドポイントのロック解除と呼ばれます。セッションの昇格プロセスには、2 つの手順、2 人のユーザー、2 つの組織認証プロセスが伴います。ロック解除の手順は Azure Stack Hub オペレーターが開始し、オペレーターは常に環境の制御を保持します。この演習では、このプロセスを表すエミュレート シナリオを通して作業に加わります。

1. 「**管理者: Windows PowerShell ISE**」ウィンドウの PowerShell リモート セッションで以下のコマンドを実行して、特権エンドポイントが現在ロックされていることを検証します。

    ```powershell
    Get-SupportSessionInfo
    ```

1. 「**管理者: Windows PowerShell ISE**」ウィンドウの PowerShell リモート セッションで以下のコマンドを実行して、サポート セッション トークンを生成します。

    ```powershell
    Get-SupportSessionToken
    ```

    >**注**: サポート シナリオでは、ユーザーは任意のメディア (チャットやメールなど) を介して、この要求トークンを Microsoft サポート エンジニアに送ります。続いて Microsoft サポート エンジニアは要求トークンを使用してサポート セッション認証トークンを生成し、その値をユーザーへと中継します。ユーザーは次に、同じ PowerShell リモート セッション内で **Unlock-SupportSession** コマンドレットを実行し、サポート セッション認証トークンの値を提示するよう求められたら、これを実行します。PowerShell リモート セッションはこの時点で昇格され、完全な管理者機能と、インフラストラクチャへの完全な到達可能性が付与されます。


#### タスク 3: 特権エンドポイントへのセッションを閉じ、セッション トランスクリプトを収集する

このタスクでは、次のことを行います。

- 特権エンドポイントへのセッションを閉じ、セッション トランスクリプトを収集する

>**注**: 特権エンドポイントでは、すべてのアクションとその出力がログに記録されます。ログを収集するには、「**Close-PrivilegedEndpoint**」コマンドレットを使用してセッションを閉じます。これよってエンドポイントが閉じられ、保管のためログ ファイルが外部ファイル共有に転送されます。

>**注**: まずは、特権エンドポイントのログを格納するためのファイル共有を作成することから始めます。

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、別の PowerShell ISE を管理者として起動します。
1. 「**管理者: Windows PowerShell ISE**」コンソールで以下のコマンドを実行して、特権エンドポイント セッションのログを格納するための共有を作成して構成します。

    ```powershell
    $pepGroup = 'AZURESTACK\CloudAdmins'
    New-Item -Path 'C:\PEPLogs' -ItemType Directory -Force
    $pepShare = New-SmbShare -Name 'PEPLogs' -Description 'PEPLogs' -Path 'C:\PEPLogs'
    Grant-SmbShareAccess -Name $pepShare.Name -AccountName $pepGroup -AccessRight Full -Force
    Revoke-SmbShareAccess -Name $pepShare.Name -AccountName 'Everyone' -Force
    ```

1. 「**管理者: Windows PowerShell ISE**」ウィンドウの PowerShell リモート セッションに戻り、以下のコマンドを実行して特権エンドポイント セッションを閉じてから、保管のためセッション ログ ファイルを外部ファイル共有に転送します。

    ```powershell
    Close-PrivilegedEndpoint -TranscriptsPathDestination '\\AzS-HOST1.azurestack.local\PEPLogs' -Credential $using:adminCredentials
    ```

1. コマンドレットが完了するまで待ってから、エクスプローラーを使用して「**C:\\PEPLogs**」フォルダーの中身を確認します。

>**確認**: この演習では、特権エンドポイントへの PowerShell リモート セッションを確立し、その機能について確認してからセッションを閉じました。
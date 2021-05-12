---
lab:
    title: 'Lab: SQL Server リソース プロバイダーを Azure Stack Hub に実装する'
    module: 'モジュール 2: サービスを提供する'
---

# ラボ - SQL Server リソース プロバイダーを Azure Stack Hub に実装する
# 受講生用ラボ マニュアル

## ラボの依存関係

- なし

## 推定所要時間

150 分

## ラボ シナリオ

あなたは Azure Stack Hub 環境のオペレーターとして、テナントが SQL Server データベースをデプロイできるようにする必要があります。 

## 目標

このラボを終了すると、次のことができるようになります。

- SQL Server リソース プロバイダーを Azure Stack Hub に実装する

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


### 演習 1: SQL Server リソース プロバイダーを Azure Stack Hub にインストールする

この演習では、SQL Server リソース プロバイダーを Azure Stack Hub にインストールします。

1. SQL Server リソース プロバイダー バイナリをダウンロードする 
1. SQL Server リソース プロバイダーをインストールする
1. SQL SQL Server リソース プロバイダーのインストールを確認する

>**注**: 演習の所要時間を可能な限り短縮するため、Azure Stack Hub SQL Server リソース プロバイダーのインストールに必要な、以下をはじめとする一部のタスクはすでに完了した状態となっています。

- Azure Marketplace シンジケーションを実装する
- **Microsoft AzureStack Add-On RP Windows Server** を Azure Marketplace からダウンロードする

#### タスク 1: SQL Server リソース プロバイダー バイナリをダウンロードする

このタスクでは、次のことを行います。

- SQL Server リソース プロバイダー バイナリをダウンロードする

1. 必要であれば、次の資格情報を使用して **AzS-HOST1** にサインインします。

    - ユーザー名: **AzureStackAdmin@azurestack.local**
    - パスワード: **Pa55w.rd1234**

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、Web ブラウザーを開いて画面に [Azure Stack Hub 管理者ポータル](https://adminportal.local.azurestack.external/)を表示させ、CloudAdmin@azurestack.local としてサインインします。
1. **AzSHOST-1** へのリモート デスクトップ セッション内で、Web ブラウザーに Azure Stack 管理者ポータルが表示されている状態で、ハブ メニューの「**すべてのサービス**」をクリックします。
1. 「**すべてのサービス**」ブレードで、「**Marketplace Management**」を検索して選択します。
1. 「Marketplace Management」ブレードで、利用可能なサービスの一覧に **Microsoft AzureStack Add-On RP Windows Server**」が表示されていることを確認します。
1. **AzSHOST-1** へのリモート デスクトップ セッション内で、ブラウザーの画面を別に開き、SQL リソース プロバイダーの自己展開型の実行可能ファイルを (https://aka.ms/azshsqlrp11931) からダウンロードし、そのコンテンツを **C:\\Downloads\\SQLRP** フォルダー (最初にフォルダーを作成する必要があります) に展開します。


#### タスク 2: SQL Server リソース プロバイダーをインストールする

このタスクでは、次のことを行います。

- SQL Server リソース プロバイダーをインストールする

1. **AzSHOST-1** へのリモート デスクトップ セッション内で、管理者として Windows PowerShell を起動します。

    > **注:** 必ず新しい PowerShell セッションを起動してください。

1. 「**管理者: Windows PowerShell**」プロンプトで以下のコマンドを実行して、PowerShell Gallery を信頼されたレポジトリとして構成します。

    ```powershell
    Set-PSRepository -Name 'PSGallery' -InstallationPolicy Trusted
    ```

1. 「**管理者: Windows PowerShell**」プロンプトで以下のコマンドを実行して、SQL Server リソース プロバイダーに必要な AzureRM.Bootstrapper モジュールをインストールします。

    ```powershell
    Get-Module -Name Azs.* -ListAvailable | Uninstall-Module -Force -Verbose
    Get-Module -Name Azure* -ListAvailable | Uninstall-Module -Force -Verbose

    Install-Module -Name AzureRm.BootStrapper -RequiredVersion 0.5.0 -Force
    Install-Module -Name AzureStack -RequiredVersion 1.6.0
    ```

1. 「**管理者: Windows PowerShell**」プロンプトで以下のコマンドを実行して、Azure Stack Hub オペレーターの PowerShell 環境を登録します。

    ```powershell
    Add-AzureRmEnvironment -Name 'AzureStackAdmin' -ArmEndpoint 'https://adminmanagement.local.azurestack.external' `
       -AzureKeyVaultDnsSuffix adminvault.local.azurestack.external `
       -AzureKeyVaultServiceEndpointResourceId https://adminvault.local.azurestack.external
    ```

1. 「**管理者: Windows PowerShell**」プロンプトで以下のコマンドを実行して、現在の環境を設定します。

    ```powershell
    Set-AzureRmEnvironment -Name 'AzureStackAdmin'
    ```

1. 「**管理者: Windows PowerShell**」プロンプトで以下のコマンドを実行し、現在の環境に対して認証を行います (サインインを求められたら、**CloudAdmin@azurestack.local** ユーザーとしてサインインし、パスワードに **Pa55w.rd1234** と入力します)。

    ```powershell
    Connect-AzureRmAccount -EnvironmentName 'AzureStackAdmin'
    ```

1. 「**管理者: Windows PowerShell**」プロンプトでで以下のコマンドを実行し、認証に成功したこと、そして対応するコンテキストが設定されたことを確認します。

    ```powershell
    Get-AzureRmContext
    ```

1. 「**管理者: Windows PowerShell**」プロンプトで以下のコマンドを実行して、SQL Server リソース プロバイダーのインストールに必要な変数を設定します。

    ```powershell
    $domain = 'azurestack.local'
    $privilegedEndpoint = 'AzS-ERCS01'
    $downloadDir = 'C:\Downloads\SQLRP'

    #  AzureStack\AzureStackAdmin 資格情報を設定する
    $serviceAdmin = 'AzureStackAdmin@azurestack.local'
    $serviceAdminPass = ConvertTo-SecureString 'Pa55w.rd1234' -AsPlainText -Force
    $serviceAdminCreds = New-Object System.Management.Automation.PSCredential ($serviceAdmin, $serviceAdminPass)

    # AzureStack\CloudAdmin 資格情報を設定する
    $cloudAdminName = 'AzureStack\CloudAdmin'
    $cloudAdminPass = ConvertTo-SecureString 'Pa55w.rd1234' -AsPlainText -Force
    $cloudAdminCreds = New-Object PSCredential($cloudAdminName, $cloudAdminPass)

    # 新しいリソース プロバイダーの VM ローカル管理者アカウントを設定する
    $vmLocalAdminPass = ConvertTo-SecureString 'Pa55w.rd1234' -AsPlainText -Force
    $vmLocalAdminCreds = New-Object System.Management.Automation.PSCredential ('sqlrpadmin', $vmLocalAdminPass)

    # 生成された自己署名証明書の秘密キーを保護するためのパスワードを設定して、SQL Server リソース プロバイダーをセキュリティ保護する
    $pfxPass = ConvertTo-SecureString 'Pa55w.rd1234pfx' -AsPlainText -Force

    # SQL Server リソース プロバイダー モジュールが含まれるよう、PowerShell モジュールのパス環境変数を更新する
    $rpModulePath = Join-Path -Path $env:ProgramFiles -ChildPath 'SqlMySqlPsh'
    $env:PSModulePath = $env:PSModulePath + ';' + $rpModulePath 
    ```

1. 「**管理者: Windows PowerShell**」プロンプトで、現在のディレクトリを、展開した SQL Server リソース プロバイダーのインストール ファイルの場所に変更し、DeploySQLProvider.ps1 スクリプトを実行します。

    ```powershell
    Set-Location -Path 'C:\Downloads\SQLRP'

    .\DeploySQLProvider.ps1 `
        -AzCredential $serviceAdminCreds `
        -VMLocalCredential $vmLocalAdminCreds `
        -CloudAdminCredential $cloudAdminCreds `
        -PrivilegedEndpoint $privilegedEndpoint `
        -DefaultSSLCertificatePassword $pfxPass
    ```

    > **注:** インストールが完了するまで待ちます。1 時間ほどかかる場合があります。

#### タスク 3: SQL SQL Server リソース プロバイダーのインストールを確認する

このタスクでは、次のことを行います。

- SQL SQL Server リソース プロバイダーのインストールを確認する

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、Azure Stack 管理者ポータルが表示されている Web ブラウザーの画面に切り替え、ハブ メニューの「**リソース グループ**」をクリックします。 
1. 「**リソース グループ**」ブレードで「**system.local.sqladapter**」をクリックします。
1. 「**system.local.sqladapter**」ブレードで「**デプロイ**」エントリを確認し、すべてのデプロイに成功したことを検証します。
1. Azure Stack 管理者ポータルで「**仮想マシン**」に移動し、SQL リソース プロバイダーが正常に作成され、実行されていることを確認します。
1. **AzS-HOST1** へのリモート デスクトップ セッション内で、Web ブラウザーを開いて画面に [Azure Stack Hub テナント ポータル](https://portal.local.azurestack.external/)を表示させ、サインインを求められたら CloudAdmin@azurestack.local としてサインインします。
1. Azure Stack Hub テナント ポータルのハブ メニューで、「**リソースの作成**」をクリックします。
1. 「**新規作成**」ブレードで「**データとストレージ**」を選択し、利用可能なリソースの種類の一覧に「**SQL データベース**」が表示されることを確認します。

>**確認**: この演習では、SQL Server リソース プロバイダーを Azure Stack Hub にインストールしました。


### 演習 2: SQL Server リソース プロバイダーを Azure Stack Hub で構成する

この演習では、SQL Server リソース プロバイダーを Azure Stack Hub で構成します。

1. (クラウド オペレーターとして) SQL Server ホスティング サーバー用のプラン、オファー、サブスクリプションを作成する
1. (クラウド オペレーターとして) SQL Server ホスティング サーバーの役割を担う Azure Stack Hub VM をデプロイする
1. (クラウド オペレーターとして) SQL ホスティング サーバーを追加する
1. (クラウド オペレーターとして) ユーザーが SQL データベースを使用できるようにする
1. (ユーザーとして) SQL データベースを作成する

>**注**: 演習の所要時間を可能な限り短縮するため、Azure Stack Hub SQL Server リソース プロバイダーの構成を促進することを目的とした、以下をはじめとする一部のタスクはすでに完了しています。

- Azure Marketplace の SQL Server イメージをダウンロードする
- Azure Marketplace から **Sql IaaS VM 拡張機能**をダウンロードする


#### タスク 1: (クラウド オペレーターとして) SQL Server ホスティング サーバー用のプラン、オファー、サブスクリプションを作成する

このタスクでは、次のことを行います。

- (クラウド オペレーターとして) SQL Server ホスティング サーバー用のプラン、オファー、サブスクリプションを作成する

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、Web ブラウザーを開いて画面に [Azure Stack Hub 管理者ポータル](https://adminportal.local.azurestack.external/)を表示させ、CloudAdmin@azurestack.local としてサインインします。
1. Web ブラウザーの画面に Azure Stack Hub 管理者ポータルが表示されている状態で、「**+ リソースの作成**」をクリックします。
1. 「**新規作成**」ブレードで、「**オファーとプラン**」をクリックしてから「**プラン**」をクリックします。
1. 「**新しいプラン**」ブレードの「**基本**」タブで、次のように設定を行います。

    - 表示名: **sql-server-hosting-plan1**
    - リソース名: **sql-server-hosting-plan1**
    - リソース グループ: 新しいリソース グループの名前 **sql-server-hosting-plans-RG**

1. 「**次へ: サービス >**」をクリックします。
1. 「**新しいプラン**」ブレードの「**サービス**」タブで、「**Microsoft.Compute**」、「**Microsoft.Storage**」、「**Microsoft.Network**」のチェックボックスをオンにします。
1. 「**次へ: クォータ >**」をクリックします。
1. 「**新しいプラン**」ブレードの「**クォータ**」タブで、次のように設定を行います。

    - Microsoft.Compute: **既定のクォータ**
    - Microsoft.Network: **既定のクォータ**
    - Microsoft.Storage: **既定のクォータ**

1. 「**確認して作成**」をクリックしてから「**作成**」をクリックします。

    >**注**: デプロイが完了するまで待ちます。通常は数秒で完了します。

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、Web ブラウザーの画面に Azure Stack Hub 管理者ポータルが表示されている状態で、「**新規作成**」ブレードに戻って「**オファー**」クリックします。
1. 「**新しいオファーの作成**」ブレードの「**基本**」タブで、次のように設定を行います。

    - 表示名: **sql-server-hosting-offer1**
    - リソース名: **sql-server-hosting-offer1**
    - リソース グループ: **sql-server-hosting-offers-RG**
    - このオファーをパブリックに設定する: **いいえ**

1. 「**次へ: 基本プラン >**」をクリックします。 
1. 「**新しいオファーの作成**」ブレードの「**基本プラン**」タブで、「**sql-server-hosting-plan1**」エントリの横にあるチェックボックスをオンにします。
1. 「**次へ: アドオン プラン >**」をクリックします。
1. 「**アドオン プラン**」の設定は既定値にしたまま、「**確認して作成**」をクリックしてから「**作成**」をクリックします。

    >**注**: デプロイが完了するまで待ちます。通常は数秒で完了します。

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、Web ブラウザーの画面に Azure Stack Hub 管理者ポータルが表示されている状態で、「**新規作成**」ブレードに戻って「**サブスクリプション**」クリックします。
1. 「**ユーザー サブスクリプションの作成**」ブレードで、次のように設定してから「**作成**」を選択します。

    - 名前: **sql-server-hosting-subscription1**
    - ユーザー: **cloudadmin@azurestack.local**
    - ディレクトリ テナント: **ADFS.azurestack.local**
    - オファー名: **sql-server-hosting-offer1**

    >**注**: デプロイが完了するまで待ちます。通常は数秒で完了します。

1. Azure Stack Hub 管理者ポータルのウィンドウは開いたままにします。


#### タスク 2: (クラウド オペレーターとして) SQL Server ホスティング サーバーの役割を担う Azure Stack Hub VM をデプロイする

このタスクでは、次のことを行います。

- (クラウド オペレーターとして) SQL Server ホスティング サーバーの役割を担う Azure Stack Hub VM をデプロイする

    >**注**: 請求可能なユーザー サブスクリプションに、SQL Server ホスティング サーバーとして機能する Azure Stack Hub VM を作成する必要があります。

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、[Azure Stack Hub テナント ポータル](https://portal.local.azurestack.external/)が表示されている Web ブラウザーの画面に切り替えます。
1. Azure Stack Hub テナント ポータルのハブ メニューで、「**リソースの作成**」をクリックします。
1. 「**新規作成**」ブレードで「**コンピューティング**」を選択し、利用可能なリソースの種類の一覧で「**{WS-BYOL} 無料の SQL Server ライセンス: Windows Server 2016 上の SQL Server 2017 Express**」を選択します。
1. 「**仮想マシンの作成**」ブレードの「**基本**」ペインで、次のように設定してから「**OK**」をクリックします (他の設定は規定値のままにします)。

    - 名前: **sql-host-vm0**
    - VM ディスクの種類: **Premium SSD**
    - ユーザー名: **sqladmin**
    - パスワード: **Pa55w.rd**
    - サブスクリプション: **sql-server-hosting-subscription1**
    - リソース グループ: 新しいリソース グループの名前 **sql-server-hosting-RG**
    - 場所: **ローカル**

1. 「**サイズの選択**」ブレードで、「**DS1_v2**」を選択してから「**選択**」をクリックします。
1. 「**仮想マシンの作成**」ブレードの「**設定**」ペインで、「**ネットワーク セキュリティ グループ**」を「**詳細**」に設定してから、「**ネットワーク セキュリティ グループ (ファイアウォール)**」をクリックします。
1. 「**ネットワーク セキュリティ グループの作成**」ブレードで、「**+ 受信規則の追加**」をクリックします。
1. 「**受信セキュリティ規則の追加**」ブレードで次のように設定してから「**追加**」を選択します (他の設定は既定値のままにします)。

    - 宛先ポート範囲: **1433**
    - プロトコル: **TCP**
    - アクション: **許可**
    - 名前: **SQL**

1. 「**ネットワーク セキュリティ グループの作成**」ブレードに戻って「**OK**」をクリックします。
1. 「**仮想マシンの作成**」ブレードの「**設定**」タブに戻り、次のように設定してから「**OK**」をクリックします (他の設定は規定値のままにします)。

    - ブート診断: 無効
    - ゲスト OS の診断: 無効

1. 「**仮想マシンの作成**」ブレードの「**SQL Server の設定**」タブで、次のように設定してから「**OK**」をクリックします (他の設定は規定値のままにします)。

    - SQL への接続: **パブリック (インターネット)**
    - ポート: **1433**
    - SQL 認証: **有効**
    - ログイン名: **SQLAdmin**
    - パスワード: **Pa55w.rd**
    - ストレージの構成: **一般**
    - 自動修正: **無効**
    - 自動バックアップ: **無効**
    - Azure Key Vault の統合: **無効**

1. 「**仮想マシンの作成**」ブレードの「**概要**」ペインで、「**OK**」をクリックします。

    >**注**: デプロイが完了するまで待ちます。20 分ほどかかる場合があります。

1. デプロイが完了したら、「**sql-host-vm0** 仮想マシン」ブレードに移動し、「**概要**」セクションの「**DNS 名**」ラベルの真下にある「**構成**」をクリックします。
1. 「**sql-host-vm0-ip \| 構成**」ブレードの「**DNS 名ラベル (オプション)**」テキスト ボックスに、「**sql-host-vm0**」と入力して「**保存**」をクリックします。

    >**注**: これにより、「**sql-host-vm0.local.cloudapp.azurestack.external**」DNS 名を介して「**sql-host-vm0**」が利用できるようになります。

1. 「**sql-host-vm0-ip \| 構成**」ブレードで、「**割り当て**」オプションを「**静的**」に設定して「**保存**」をクリックします。

    >**注**: これにより、「**sql-host-vm0**」仮想マシンの再起動がトリガーされます。


#### タスク 3: (クラウド オペレーターとして) SQL ホスティング サーバーを追加する

このタスクでは、次のことを行います。

- (クラウド オペレーターとして) SQL ホスティング サーバーを追加する

1. **AzSHOST-1** へのリモート デスクトップ セッション内で、Web ブラウザーに Azure Stack 管理者ポータルが表示されている状態で「**すべてのサービス**」をクリックし、「**管理リソース**」セクションで「**SQL ホスティング サーバー**」をクリックします。

    > **注:** **SQL Hosting Servers** リソースの種類を表示させるには、Azure Stack 管理者ポータルが表示されているブラウザー ページを更新する必要があります。

1. 「**SQL ホスティング サーバー**」ブレードで「**+ 追加**」をクリックします。
1. 「**SQL ホスティング サーバーの追加**」ブレードで、次のように設定を行います。

    - SQL サーバー名: **sql-host-vm0.local.cloudapp.azurestack.external**
    - ユーザー名: **sqladmin**
    - パスワード: **Pa55w.rd**
    - ホスティング サーバーのサイズ (GB): **50**
    - 常時接続可用性グループ: オフ
    - サブスクリプション: **既定のプロバイダー サブスクリプション**
    - リソース グループ: 新しいリソース グループの名前 **sql.resources-RG**
    - 場所: **ローカル**

1. 「**SQL ホスティング サーバーの追加**」ブレードで「**SKU**」をクリックし、「**SKU**」ブレードで「**新しい SKU の作成**」をクリックしてから、「**SKU の作成**」ブレードで以下のように設定を行います。

    - 名前: **MSSQL2017Exp**
    - ファミリ: **SQL Server 2017**
    - サービス レベル: **スタンドアロン**
    - エディション: **Express**

1. 「**SKU の作成**」ブレードで「**OK**」をクリックし、「**SQL ホスティング サーバー**」ブレードに戻って「**作成**」をクリックします。

    > **注:** 操作が完了するまで待ちます。通常は 1 分もかかりません。

1. 「**SQL ホスティング サーバー**」ブレードで「**更新**」をクリックし、サーバーの一覧に「**sqlhost1.local.cloudapp.azurestack.external**」が表示されていることを確認します。


#### タスク 4: (クラウド オペレーターとして) ユーザーが SQL データベースを使用できるようにする

このタスクでは、次のことを行います。

- (クラウド オペレーターとして) ユーザーが SQL データベースを使用できるようにする

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、Web ブラウザーの画面に Azure Stack Hub 管理者ポータルが表示されている状態で、「**+ リソースの作成**」をクリックします。
1. 「**新規作成**」ブレードで、「**オファーとプラン**」をクリックしてから「**プラン**」をクリックします。
1. 「**新しいプラン**」ブレードの「**基本**」タブで、次のように設定を行います。

    - 表示名: **sql-server-2017-express-db-plan1**
    - リソース名: **sql-server-2017-express-db-plan1**
    - リソース グループ: 新しいリソース グループの名前 **sqldb-plans-RG**

1. 「**次へ: サービス >**」をクリックします。
1. 「**新しいプラン**」ブレードの「**サービス**」タブで、「**Microsoft.SQLAdapter**」チェックボックスをオンにします。
1. 「**次へ: クォータ >**」をクリックします。
1. 「**新しいプラン**」ブレードの「**クォータ**」タブで、「**Microsoft.SQLAdapter**」エントリの横にある「**新規作成**」をクリックします。
1. 「**クォータの作成*」ブレードで、次のように設定してから「**作成**」をクリックします。

    - クォータ名: **sql-server-2017-express-db-quota1**
    - データベースの最大サイズ (GB): **2**
    - データベースの最大数: **20**

1. 「**確認して作成**」をクリックしてから「**作成**」をクリックします。

    >**注**: デプロイが完了するまで待ちます。通常は数秒で完了します。

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、Web ブラウザーの画面に Azure Stack Hub 管理者ポータルが表示されている状態で、「**新規作成**」ブレードに戻って「**オファー**」クリックします。
1. 「**新しいオファーの作成**」ブレードの「**基本**」タブで、次のように設定を行います。

    - 表示名: **sql-server-2017-express-db-offer1**
    - リソース名: **sql-server-2017-express-db-offer1**
    - リソース グループ: **sqldb-offers-RG**
    - このオファーをパブリックに設定する: **はい**

1. 「**次へ: 基本プラン >**」をクリックします。 
1. 「**新しいオファーの作成**」ブレードの「**基本プラン**」タブで、「**sql-server-2017-express-db-plan1**」エントリの横にあるチェックボックスをオンにします。
1. 「**次へ: アドオン プラン >**」をクリックします。
1. 「**アドオン プラン**」の設定は既定値にしたまま、「**確認して作成**」をクリックしてから「**作成**」をクリックします。

    >**注**: デプロイが完了するまで待ちます。通常は数秒で完了します。


#### タスク 5: (ユーザーとして) SQL データベースを作成する

このタスクでは、次のことを行います。

- テスト用のユーザー アカウントを作成する
- 新たに作成されたユーザー アカウントを使用して SQL データベースを作成する

1. **AzS-HOST1** へのリモート デスクトップ セッション内でスタート メニューの「**スタート**」をクリックし、「**Windows 管理ツール**」をクリックしてから、管理ツールの一覧で「**Active Directory 管理センター**」をダブルクリックします。
1. 「**Active Directory 管理センター**」コンソールで「**azurestack (ローカル)**」をクリックします。
1. 詳細ペインで「**ユーザー**」コンテナーをダブルクリックします。
1. 「**タスク**」ペインの「**ユーザー**」セクションで、**「新規作成」>「ユーザー」**の順にクリックします。
1. 「**ユーザーの作成**」ウィンドウで、次のように設定してから「**OK**」をクリックします。 

    - 完全名: **T1U1**
    - ユーザー UPN ログオン: **t1u1@azurestack.local**
    - ユーザー SamAccountName: **azurestack\t1u1**
    - パスワード: **Pa55w.rd**
    - パスワード オプション: **その他のパスワード オプション -> パスワードを無期限にする**

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、Web ブラウザーの InPrivate セッションを開始します。
1. Web ブラウザー ウィンドウで [Azure Stack Hub ユーザー ポータル](https://portal.local.azurestack.external)に移動した後、**t1u1@azurestack.local** としてサインインし、パスワードに **Pa55w.rd** と入力します。
1. Azure Stack Hub ユーザー ポータルのダッシュボードで、「**サブスクリプションの取得**」タイルをクリックします。
1. 「**サブスクリプションの取得**」ブレードの「**名前**」テキスト ボックスに「**t1u1-sqldb-subscription1**」と入力します。
1. オファーの一覧で「**sql-server-2017-express-db-offer1**」を選択し、「**作成**」をクリックします。
1. 「**サブスクリプションが作成されました。サブスクリプションの使用を開始するに、ポータルを更新する必要があります**」のメッセージが表示されたら、「**更新**」をクリックします。 
1. Azure Stack Hub テナント パネルのハブ メニューで、「**すべてのサービス**」をクリックします。
1. サービスの一覧で「**SQL データベース**」をクリックします。
1. 「**SQL データベース**」ブレードで「**+ 追加**」をクリックします。
1. 「**データベースの作成**」ブレードで、次のように設定を行います。

    - データベース名: **sqldb1**
    - 照合順序: **SQL_Latin1_General_CP1_CI_AS**
    - 最大サイズ (MB): **200**
    - サブスクリプション: **t1u1-sqldb-subscription1**
    - リソース グループ: 新しいリソース グループの名前 **sqldb-RG**
    - 場所: **ローカル**
    - SKU: **MSSQL2017Exp**

    > ** 注**: 新たに作成した SKU がテナント ポータルで利用できるようになるまで、しばらく待たなければならない場合があります。

1. 「**データベースの作成**」ブレードで「**ログイン**」をクリックします。
1. 「**ログインの選択**」ブレードで「**新しいログインの作成**」をクリックします。
1. 「**新しいログイン**」ブレードで、以下のように設定してから「**OK**」をクリックします。

    - データベース ログイン: **dbAdmin**
    - パスワード: **Pa55w.rd**

1. 「**データベースの作成**」ブレードに戻って「**作成**」をクリックします。

    >**注**: デプロイが完了するまで待ちます。通常は 1 分もかかりません。 

>**確認**: この演習では、SQL Server ホスティング サーバーを Azure Stack Hub に追加し、これをテナントが利用できるよう設定したほか、SQL データベースをテナント ユーザーとしてデプロイしました。

---
lab:
    title: 'Lab: App Service リソース プロバイダーを Azure Stack Hub に実装する'
    module: 'モジュール 2: サービスを提供する'
---

# ラボ - App Service リソース プロバイダーを Azure Stack Hub に実装する
# 受講生用ラボ マニュアル

## ラボの依存関係

- SQL Server リソース プロバイダーを Azure Stack Hub に実装する

## 推定所要時間

4 時間

## ラボ シナリオ

あなたは Azure Stack Hub 環境のオペレーターとして、テナントが App Service アプリと Azure Functions をデプロイできるようにする必要があります。

## 目標

このラボを終了すると、次のことができるようになります。

 - App Service リソース プロバイダーを Azure Stack Hub に実装する

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


### 演習 1: App Service リソース プロバイダーを Azure Stack Hub にインストールする

この演習では、App Service リソース プロバイダーを Azure Stack Hub にインストールします。

1. SQL Server ホスティング サーバーをプロビジョニングする
1. ファイル サーバーをプロビジョニングする
1. App Service リソース プロバイダーをインストールする
1. App Service リソース プロバイダーのインストールを検証する

>**注**: 演習の所要時間を可能な限り短縮するため、Azure Stack Hub App Service リソース プロバイダーのインストールに必要な、以下をはじめとする一部のタスクはすでに完了した状態となっています。

- Azure Marketplace シンジケーションを実装する
- 次の Azure Marketplace 項目をダウンロードする

  - **[smalldisk] Windows Server 2019 Datacenter Server Core - ライセンス持ち込み**
  - **Windows Server 2016 Datacenter - ライセンス持ち込み** 
  - **カスタム スクリプト拡張機能**


#### タスク 1: SQL Server ホスティング サーバーをプロビジョニングする

このタスクでは、次のことを行います。

- SQL Server ホスティング サーバーをプロビジョニングする

1. 必要であれば、次の資格情報を使用して **AzS-HOST1** にサインインします。

    - ユーザー名: **AzureStackAdmin@azurestack.local**
    - パスワード: **Pa55w.rd1234**

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、Web ブラウザーを開いて画面に [Azure Stack Hub 管理者ポータル](https://adminportal.local.azurestack.external/)を表示させ、CloudAdmin@azurestack.local としてサインインします。
1. Azure Stack Hub 管理者ポータルのハブ メニューで、「**リソースの作成**」をクリックします。
1. 「**新規作成**」ブレードで「**コンピューティング**」を選択し、利用可能なリソースの種類の一覧で「**{WS-BYOL} 無料の SQL Server ライセンス: Windows Server 2016 上の SQL Server 2017 Express**」を選択します。
1. 「**仮想マシンの作成**」ブレードの「**基本**」ペインで、次のように設定してから「**OK**」をクリックします (他の設定は規定値のままにします)。

    - 名前: **SqlHOST1**
    - VM ディスクの種類: **Premium SSD**
    - ユーザー名: **sqladmin**
    - パスワード: **Pa55w.rd**
    - サブスクリプション: **既定のプロバイダー サブスクリプション**
    - リソース グループ: 新しいリソース グループの名前 **sql.resources-RG**
    - 場所: **ローカル**

1. 「**サイズの選択**」ブレードで、「**DS1_v2**」を選択してから「**選択**」をクリックします。
1. 「**仮想マシンの作成**」ブレードの「**設定**」ペインで、「**ネットワーク セキュリティ グループ**」を「**詳細**」に設定してから、「**ネットワーク セキュリティ グループ (ファイアウォール)**」をクリックします。
1. 「**ネットワーク セキュリティ グループの作成**」ブレードで、「**+ 受信規則の追加**」をクリックします。
1. 「**受信セキュリティ規則の追加**」ブレードで次のように設定してから「**追加**」を選択します (他の設定は既定値のままにします)。

    - 宛先ポート範囲: **1433**
    - プロトコル: **TCP**
    - アクション: **許可**
    - 優先順位: **200**
    - 名前: **custom-allow-sql**

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

1. デプロイが完了したら、**SqlHOST1** 仮想マシンブレードに移動し、「**概要**」セクションの「**DNS 名**」ラベルの真下にある「**構成**」をクリックします。
1. 「**SqlHOST1-ip \| 構成**」ブレードの「**DNS 名ラベル (オプション)**」テキスト ボックスに「**sqlhost1**」と入力して「**保存**」をクリックします。

    >**注**: これにより、「**sqlhost1.local.cloudapp.azurestack.external**」DNS 名を介して「**sqlhost1**」が利用できるようになります。

1. 「**sqlhost1-ip \| 構成**」ブレードで、「**割り当て**」オプションを「**静的**」に設定して「**保存**」をクリックします。

    >**注**: これにより、**sqlhost1** 仮想マシンの再起動がトリガーされます。再起動が完了するのを待ってから、次の手順に進みます。

1. **AzSHOST-1** へのリモート デスクトップ セッション内で、**sqlhost1.local.cloudapp.azurestack.external** へのリモート デスクトップ セッションを開始し、サインインを求められたら、次の資格情報を使用してサインインします。

    - ユーザー名: **SQLAdmin**
    - パスワード: **Pa55w.rd**

1. **SqlHOST1** へのリモート デスクトップ セッション内で、「**スタート**」を右クリックし、右クリック メニューから「**コマンド プロンプト (管理者)**」を選択します。 
1. **SqlHOST1** へのリモート デスクトップ セッション内で、「**管理者: コマンド プロンプト**」から以下のコマンドを実行し、ローカル SQL Server インスタンスに対して SQLCMD セッションを開始します。

    ```
    sqlcmd
    ```

1. **SqlHOST1** へのリモート デスクトップ セッション内で、「**管理者: コマンド プロンプト**」から以下のコマンドを実行し、SQL サーバーの包含データベース認証を有効にします。

    ```
    sp_configure 'contained database authentication', 1;
    GO
    RECONFIGURE;
    GO
    ```

    > **注:** この操作は、ラボの後半で App Service リソース プロバイダーを実装する際に、このホスティング サーバーを使用するために必要となります。

    > **注:** **sqlhost1.local.cloudapp.azurestack.external** へのリモート デスクトップ セッションは開いたままにします。この値はこのラボの後半で使用します。


#### タスク 2: ファイル サーバーをプロビジョニングする

このタスクでは、次のことを行います。

- ファイル サーバーをプロビジョニングする

1. **AzSHOST-1** へのリモート デスクトップ セッションに切り替え、Azure Stack 管理者ポータルのハブ メニューで「**すべてのサービス**」をクリックします。
1. サービスの一覧で「**Marketplace Management**」をクリックします。
1. 「**Marketplace Management - Marketplace 項目**」ブレードで、「**[smalldisk] Windows Server 2019 Datacenter Server Core - ライセンス持ち込み**」項目を検索し、これが利用可能となっていることを確認します。
1. **AzSHOST-1** へのリモート デスクトップ セッション内で、ブラウザーの画面で新しいタブを開き、(https://aka.ms/appsvconmasdkfstemplate) に移動します。
1. 「**AzureStack-QuickStart-Templates / appservice-fileserver-standalone**」ページで、「**azuredeploy.json**」をクリックしてから「**Raw**」をクリックします。
1. ページの全コンテンツを選択して、クリップボードにコピーします。
1. Azure Stack 管理者ポータルに戻って、「**+ リソースの作成**」をクリックします。
1. 「**新規作成**」ブレードで、「**カスタム**」をクリックしてから「**テンプレートのデプロイ**」をクリックします。
1. 「**カスタム デプロイ**」ブレードで、「**エディターで独自のテンプレートを作成**」を選択します。 
1. 「**テンプレートの編集**」ブレードで、作成済みのテンプレートをクリップボードのコンテンツに置き換えます。
1. 「**カスタム デプロイ**」ブレードで「**テンプレートの編集**」をクリックします。
1. 「**テンプレートの編集**」ブレードの「**パラメーター**」セクションで、以下の値を設定します。

    - **imageReference** の **defaultValue**: **MicrosoftWindowsServer** に設定** | WindowsServer | 2019-Datacenter-Core-smalldisk | latest**
    - **imageReference** の **allowedValues**: **MicrosoftWindowsServer** に設定** | WindowsServer | 2019-Datacenter-Core-smalldisk | latest**
    - **fileServerVirtualMachineSize** の **defaultValue**: **Standard_A1_v2** に設定
    - **fileServerVirtualMachineSize** の **allowedValues**: **Standard_A1_v2** に設定
    - **adminPassword** の **defaultValue**: **Pa55w.rd1234** に設定
    - **fileShareOwnerPassword** の **defaultValue**: **Pa55w.rd1234** に設定
    - **fileShareUserPassword** の **defaultValue**: **Pa55w.rd1234** に設定

    > **注:** これにより、「**パラメーター**」セクションのコンテンツは以下のようになります。

    ```json
      "parameters": {
        "fileServerVirtualMachineSize": {
          "type": "string",
          "defaultValue": "Standard_A1_v2",
          "allowedValues": [
            "Standard_A1_v2",
          ],
          "metadata": {
            "description": "Size of vm"
          }
        },
        "imageReference": {
          "type": "string",
          "defaultValue": "MicrosoftWindowsServer | WindowsServer | 2019-Datacenter-Core-smalldisk | latest",
          "allowedValues": [
            "MicrosoftWindowsServer | WindowsServer | 2019-Datacenter-Core-smalldisk | latest"
          ],
          "metadata": {
            "description": "Please ensure the image is available. publisher: MicrosoftWindowsServer | offer: WindowsServer | sku: 2016-Datacenter"
          }
        },
        "dnsNameForPublicIP": {
          "type": "string",
          "defaultValue": "appservicefileshare",
          "maxLength": 63,
          "metadata": {
            "description": "Unique DNS Name for the Public IP used to access the file share.It must be lowercase. It should match the following regular expression, or it will raise an error: ^[a-z][a-z0-9-]{1,61}[a-z0-9]$"
          }
        },
        "adminUsername": {
          "type": "string",
          "defaultValue": "fileshareowner",
          "metadata": {
            "description": "File server Admin user"
          }
        },
        "adminPassword": {
          "type": "securestring",
          "defaultValue": "Pa55w.rd1234",
          "metadata": {
            "description": "File server Admin password"
          }
        },
        "fileShareOwner": {
          "type": "string",
          "defaultValue": "fileshareowner",
          "metadata": {
            "description": "fileshare owner username"
          }
        },
        "fileShareOwnerPassword": {
          "type": "securestring",
          "defaultValue": "Pa55w.rd1234",
          "metadata": {
            "description": "fileshare owner password"
          }
        },
        "fileShareUser": {
          "type": "string",
          "defaultValue": "fileshareuser",
          "metadata": {
            "description": "fileshare user"
          }
        },
        "fileShareUserPassword": {
          "type": "securestring",
          "defaultValue": "Pa55w.rd1234",
          "metadata": {
            "description": "fileshare user password"
          }
        },
        "vmExtensionScriptLocation": {
          "type": "string",
          "defaultValue": "https://raw.githubusercontent.com/Azure/azurestack-quickstart-templates/master/appservice-fileserver-standalone",
          "metadata": {
            "description": "File Server extension script Url"
          }
        }
      },
    ```

1. 「**テンプレートの編集**」ブレードの「**変数**」セクションで、「**"sku": "2016-Datacenter",**」を「**"sku": "2019-Datacenter-Core-smalldisk",**」に置き換えて、「**保存**」を選択します。
1. 「**カスタム デプロイ**」ブレードに戻り、「**サブスクリプション**」ドロップダウン リストで「**既定のプロバイダー サブスクリプション**」を選択し、「**リソース グループ**」セクションで「**sql.resources-RG**」を選択します。
1. 「**カスタム デプロイ**」ブレードで、「**確認して作成**」をクリックしてから「**作成**」をクリックします。

    > **注:** デプロイが完了するまで待ちます。通常は 15 分ほどかかります。


#### タスク 3: App Service リソース プロバイダーをインストールする

このタスクでは、次のことを行います。

- App Service リソース プロバイダーをインストールする

1. **AzSHOST-1** へのリモート デスクトップ セッション内で、Azure Stack 管理者ポータルのハブ メニューで「**すべてのサービス**」をクリックします。
1. サービスの一覧で「**Marketplace Management**」をクリックします。
1. 「**Marketplace Management - Marketplace 項目**」ブレードで、「**Windows Server 2016 Datacenter - ライセンス持ち込み**」項目を検索し、これが利用可能となっていることを確認します。
1. 「**Marketplace Management - Marketplace 項目**」ブレードで、「**カスタム スクリプト拡張機能**」項目を検索し、これが利用可能となっていることを確認します。
1. **AzSHOST-1** へのリモート デスクトップ セッション内で、Web ブラウザーを起動して (https://aka.ms/appsvconmasinstaller) に移動し、**AppService.exe** をダウンロードします。ダウンロードが完了したら、ファイルを「**C:\\Downloads\\AppServiceRP**」フォルダー (必要であれば作成する必要があります) にコピーします。
1. Web ブラウザーで (https://aka.ms/appsvconmashelpers) に移動し、**AppServiceHelperScripts.zip** をダウンロードします。ダウンロードが完了したら、そのコンテンツを「**C:\\Downloads\\AppServiceRP**」フォルダーに展開します。
1. **AzSHOST-1** へのリモート デスクトップ セッション内で、管理者として Windows PowerShell を起動します。
1. 「**管理者: Windows PowerShell**」ウィンドウで以下のコマンドを実行して、Azure Stack 上の App Service に必要な証明書を作成します。

    ```powershell
    Set-Location -Path C:\Downloads\AppServiceRP
    Get-ChildItem -Path '.\' -File -Recurse | Unblock-File

    $pfxPass = ConvertTo-SecureString 'Pa55w.rd1234pfx' -AsPlainText -Force

    .\Create-AppServiceCerts.ps1 `
	-pfxPassword $pfxPass `
	-DomainName 'local.azurestack.external'
    ```

1. 「**管理者: Windows PowerShell**」ウィンドウで以下のコマンドを実行して、App Service プロバイダーのインストールに必要な、Azure Stack Hub 用のルート証明書を作成します。

    ```
    $domain = 'azurestack.local'
    $privilegedEndpoint = 'AzS-ERCS01'

    # Add the cloudadmin credential that's required for privileged endpoint access.
    $cloudAdminPass = ConvertTo-SecureString 'Pa55w.rd1234' -AsPlainText -Force
    $cloudAdminCreds = New-Object System.Management.Automation.PSCredential ("CloudAdmin@$domain", $cloudAdminPass)

    .\Get-AzureStackRootCert.ps1 -PrivilegedEndpoint $privilegedEndpoint -CloudAdminCredential $cloudAdminCreds
    ```

    > **注:** このスクリプトにより、「AzureStackCertificationAuthority.cer」と名付けられたファイル (Azure Stack Hub 用の Azure Resource Manager ルート証明書が含まれています) がローカル フォルダーに作成されます。

1. 「**管理者: Windows PowerShell**」ウィンドウで以下のコマンドを実行して、App Service プロバイダーのインストールに必要な AD FS アプリを作成します。

    ```
    $domain = 'azurestack.local'
    $privilegedEndpoint = 'AzS-ERCS01'
    $adminArmEndpoint = 'adminmanagement.local.azurestack.external'
    $certificateFilePath = 'C:\Downloads\AppServiceRP\sso.appservice.local.azurestack.external.pfx'
    $certificatePassword = ConvertTo-SecureString 'Pa55w.rd1234pfx' -AsPlainText -Force

    .\Create-ADFSIdentityApp.ps1 `
       -AdminArmEndpoint $adminArmEndpoint `
       -PrivilegedEndpoint $privilegedEndpoint `
       -CloudAdminCredential $cloudAdminCreds `
       -CertificateFilePath $certificateFilePath `
       -CertificatePassword $certificatePassword
    ```

1. スクリプトの出力で、生成された AD FS アプリケーションの ID を表す GUID をコピーします。 

    > **注:** この GUID は必ず記録しておいてください。このタスクの後半で必要になります。

1. 「**管理者: Windows PowerShell**」ウィンドウと「**管理者: Windows PowerShell**」ウィンドウで、以下のコマンドを実行して AppService.exe を起動します。

    ```
    .\AppService.exe
    ```

    > **注:** これにより、「Microsoft Azure App Service セットアップ」ウィザードが起動します。

1. 「**App Service をデプロイするか、または最新バージョンにアップグレードする**」をクリックします。
1. 「**Microsoft ソフトウェア ライセンスの補足条項**」ページでその内容を確認し、「**ライセンス条項の内容を読み、理解し、同意しました**」チェックボックスをオンにして「**次へ**」をクリックします。
1. 「サード パーティ ライセンス条項」ページでその内容を確認し、「**ライセンス条項の内容を読み、理解し、同意しました**」チェックボックスをオンにして「**次へ**」をクリックします。
1. 管理者およびテナントの ARM エンドポイントが表示されているページで、情報が正しいことを確認してから「**次へ**」をクリックします。
1. 「Azure Stack App Service クラウド情報」ページで、「**資格情報**」オプションが選択されていることを確認してから「**接続**」をクリックします。
1. サインインするよう求められたら、**CloudAdmin@AzureStack.local** としてサインインし、パスワードに **Pa55w.rd1234** と入力します。
1. 「Azure Stack App Service クラウド情報」ページに戻り、「**Azure Stack サブスクリプション**」ドロップダウン リストで「**既定のプロバイダー サブスクリプション**」を選択してから、「**Azure Stack の場所**」ドロップダウン リストで「**ローカル**」を選択して「**次へ**」をクリックします。
1. 「**仮想ネットワークの構成**」で、既定の設定を使用して「**次へ**」をクリックします。
1. 次のページで、以下のように設定してから「**次へ**」をクリックします。

    - ファイル共有 UNC パス: **\\appservicefileshare.local.cloudapp.azurestack.external\websites**
    - ファイル共有所有者: **fileshareowner**
    - ファイル共有所有者のパスワード:**Pa55w.rd1234**
    - ファイル共有ユーザー: **fileshareuser**
    - ファイル共有ユーザーのパスワード: **Pa55w.rd1234**

1. 次のページで、このタスクの前半で生成したアプリケーション ID が特定されるよう設定を行い、「**次へ**」をクリックします。

    - ID アプリケーション ID: このタスクの前半でコピーした GUID
    - ID アプリケーション証明書ファイル (*.pfx): **C:\Downloads\AppServiceRP\sso.appservice.local.azurestack.external.pfx**
    - ID アプリケーション証明書ファイル (*.pfx) のパスワード: **Pa55w.rd1234pfx**
    - Azure Resource Manager (ARM) ルート証明書ファイル (*.cer): **C:\Downloads\AppServiceRP\AzureStackCertificationAuthority.cer**

1. 次のページで、証明書ファイルとその対応パスワードが特定されるよう設定を行います。

    - App Service の既定の SSL 証明書ファイル (*.pfx): **C:\Downloads\AppServiceRP\_.appservice.local.azurestack.external.pfx**
    - App Service の既定の SSL 証明書 (*.pfx) のパスワード: **Pa55w.rd1234pfx**
    - App Service の API SSL 証明書ファイル (*.pfx): **C:\Downloads\AppServiceRP\api.appservice.local.azurestack.external.pfx**
    - App Service API SSL 証明書 (*.pfx) のパスワード: **Pa55w.rd1234pfx**
    - App Service 発行元の証明書ファイル (*.pfx): **C:\Downloads\AppServiceRP\ftp.appservice.local.azurestack.external.pfx**
    - App Service 発行元の SSL 証明書 (*.pfx) のパスワード: **Pa55w.rd1234pfx**

1. 次のページで、SQL Server の設定を行います。

    - SQL Server 名: **sqlhost1.local.cloudapp.azurestack.external**
    - SQL sysadmin ログイン: **SQLAdmin**
    - SQL sysadmin のパスワード: **Pa55w.rd1234**

1. 次のページで、App Service デプロイのインスタンスの数と SKU を指定します。

    - コントローラー ロール: **2 つのインスタンス - Standard_A1_v2 - [1 コア、2048 MB]**
    - 管理ロール: **1 つのインスタンス - Standard_A2_v2 - [2 コア、4096 MB]**
    - 発行元ロール: **1 つのインスタンス - Standard_A1_v2 - [1 コア、2048 MB]**
    - FrontEnd ロール: **1 つのインスタンス - Standard_A1_v2 - [1 コア、2048 MB]**
    - 共有 worker ロール: **1 つのインスタンス - Standard_A1_v2 - [1 コア、2048 MB]**

1. 「**次へ**」をクリックします。
1. 次のページの「**プラットフォーム イメージの選択**」ドロップダウン リストで、「**2016 Datacenter - 最新**」イメージを選択して「**次へ**」をクリックします。
1. 次のページで、デプロイ用の管理者資格情報を以下のように設定します。

    - worker ロールの仮想マシンの管理者: **SAWorkerAdmin**
    - worker ロールの仮想マシンのパスワード: **Pa55w.rd1234**
    - パスワードの確認: **Pa55w.rd1234**
    - 他のロールの仮想マシンの管理者: **SAORoleAdmin**
    - 他のロールの仮想マシンのパスワード: **Pa55w.rd1234**
    - パスワードの確認: **Pa55w.rd1234**

1. 「**次へ**」をクリックします。
1. 「概要」ページで「**デプロイを開始するには「次へ」を選択します**」チェックボックスをクリックし、デプロイを起動して「**次へ**」をクリックします。

    > **注:** インストールが完了するまで待ちます。2～3 時間かかる場合があります。

1. インストールが完了したら「**終了**」をクリックします。


#### タスク 4: App Service リソース プロバイダーのインストールを検証する

このタスクでは、次のことを行います。

- App Service リソース プロバイダーのインストールを検証する

1. **AzSHOST-1** へのリモート デスクトップ セッション内で、Web ブラウザーに Azure Stack 管理者ポータルが表示されている状態で、ハブ メニューの「**すべてのサービス**」を選択し、「**すべてのサービス**」ブレードで「**管理**」を選択してから、サービスの一覧で「**App Service**」をクリックします。 

    > **注:** 「**App Service**」エントリが利用できるようにするには、ブラウザー ページを更新しなければならない場合があります。

1. 「**App Service**」ブレードの「**要点**」セクションで、「**状態**」ラベルに「**すべてのロールの準備が完了しました**」のメッセージが表示されていることを確認します。

    > **注:** すべてのロールが正常に起動するまで待ちます。追加で15～20 分かかる場合があります。

>**確認**: この演習では、App Service リソース プロバイダーを Azure Stack Hub にインストールしました。


### 演習 2: Azure Stack Hub 上の App Service リソース プロバイダーの管理タスクについて学習する

この演習では、Azure Stack Hub 上の App Service リソース プロバイダーの管理タスクについて学習します。

1. App Service リソースのスケーリング機能について学習する
1. App Service リソース プロバイダーのバックアップ設定について学習する


### タスク 1: App Service リソースのスケーリング機能について学習する

このタスクでは、次のことを行います。

- App Service リソースのスケーリング機能について確認する
- App Service リソース プロバイダーのバックアップ設定について確認する

1. **AzSHOST-1** へのリモート デスクトップ セッション内で、Web ブラウザーに Azure Stack 管理者ポータルが表示されている状態で、「**App Service**」の「**ロール**」をクリックします。
1. 「**App Service | ロール**」ブレードで、ロールとその対応インスタンスの一覧を確認します。
1. 「**App Service | ロール**」ブレードの「**コントローラー**」行で、右側にある省略記号をクリックし、「**仮想マシン**」エントリを確認します。

    > **注:** コントローラー ロールは、仮想マシンによって使用されます。App Service リソース プロバイダーのインストール時に、コントローラーの 2 つのインスタンスが選択されるのはそのためです。

1. 「**App Service | ロール**」ブレードの残りの行で、右側にある省略記号をクリックし、ドロップダウン リストの「**ScaleSet**」エントリを確認します。

    > **注:** 他のロールはいずれもスケール セットによって実装されることで、スケーリングが可能となります。

1. 「**App Service | ロール**」ブレードで、**共有** worker 階層には現在 Web Worker ロールしかないことを確認します。 
1. 「**App Service | ロール**」ブレードの左側の垂直メニューで、「**worker 階層**」を選択します。
1. 「**App Service | worker 階層**」ブレードで「**+ 追加**」をクリックします。 
1. 「**作成**」ブレードで、利用可能なオプション (「**共有**」と「**専用**」の間で切り替えるための「**コンピューティング モード**」ドロップダウン リストを含む) を確認します。

    > **注:** 仮想マシンは、カスタム ソフトウェアを使用することで、任意の worker 階層内の仮想マシンとしてさまざまなサイズでデプロイすることができます。

1. 何も変更を加えずに「**作成**」ブレードを閉じます。

    > **注:** プロビジョニング プロセスには 1 時間以上かかる場合があります。


### タスク 2: App Service リソース プロバイダーのバックアップ設定について学習する

このタスクでは、次のことを行います。

- App Service リソース プロバイダーのバックアップ設定について確認する

> **注:** Azure Stack Hub 上の App Service バックアップは、次の主要コンポーネントから構成されます。

- リソース プロバイダー インフラストラクチャ
- リソース プロバイダー シークレット 
- 計測データベースをホストするための SQL Server インスタンス
- App Service ファイル共有に格納されたユーザー ワークロード コンテンツ

    > **注:** リソース プロバイダーのインフラストラクチャ構成は、App Service リカバリー PowerShell コマンドレットを使用して、復旧中にバックアップから再作成できます。復旧プロセスの詳細については、「[Azure Stack 上の App Service の復旧](https://docs.microsoft.com/ja-jp/azure-stack/operator/app-service-recover?view=azs-2008)」を参照してください。

1. **AzSHOST-1** へのリモート デスクトップ セッション内で、Web ブラウザーに Azure Stack 管理者ポータルが表示されている状態で、「**App Service**」の「**シークレット**」をクリックします。
1. 「**App Service \| シークレット**」ブレードで、「**シークレットのダウンロード**」をクリックしてから「**保存**」をクリックします。
1. 「**SystemSecrets.json**」ファイルが、**AzSHOST-1** の「**Downloads**」フォルダーにダウンロードされたことを確認します。

    > **注:** 「**SystemSecrets.json**」ファイルは安全な場所にコピーしてください。以後シークレットがローテーションされるたびにこのプロセスを繰り返す必要があります。 

    > **注:** 「**Appservice_hosting**」データベースと「**Appservice_metering**」データベースをバックアップする際のアプローチとして、Azure Backup Server の SQL Server メンテナンス プランを使用することが推奨されますが、これらは SQL Server PowerShell モジュール コマンドレットを使用してもバックアップできます。

1. **AzSHOST-1** へのリモート デスクトップ セッション内で、**sqlhost1.local.cloudapp.azurestack.external** へのリモート デスクトップ セッションに切り替えます。 
1. **SqlHOST1** へのリモート デスクトップ セッション内で、管理者として Windows PowerShell を起動します。
1. **SqlHOST1** へのリモート デスクトップ セッション内で、「**管理者: Windows PowerShell**」プロンプトから以下のコマンドを実行して、App Service データベースのローカル バックアップを実行します。

    ```powershell
    $date = Get-Date -Format 'yyyyMMdd'
    New-Item -ItemType Directory -Path 'C:\Backups'
    Backup-SqlDatabase -ServerInstance 'localhost' -Database 'appservice_hosting' -BackupFile "C:\Backups\appservice_hosting_$date.bak" -CopyOnly
    Backup-SqlDatabase -ServerInstance 'localhost' -Database 'appservice_metering' -BackupFile "C:\Backups\appservice_metering_$date.bak" -CopyOnly
    ```

    > **注:** App Service では、指定したファイル共有にテナント アプリの情報が格納されます。ファイル共有をバックアップする際のアプローチとして Azure Backup Server の使用が推奨されますが、このようなバックアップにおいては任意のファイル コピー ユーティリティを使用することもできます。

1. **AzSHOST-1** へのリモート デスクトップ セッションに切り替え、**AzSHOST-1** へのリモート デスクトップ セッション内で、Web ブラウザーに Azure Stack 管理者ポータルが表示されている状態で、「**App Service**」ブレードの「**システム構成**」をクリックします。
1. Web ブラウザーに Azure Stack 管理者ポータルが表示されている状態で、「**App Service \| システム構成**」ブレードに表示される**ファイル共有**の完全なパスを記録しておきます (**\\\\appservicefileshare.local.cloudapp.azurestack.external\\websites**)。
1. **AzSHOST-1** へのリモート デスクトップ セッションに切り替え、「**管理者: Windows PowerShell**」ウィンドウで以下のコマンドを実行して、App Service ファイル共有のコンテンツをローカル ファイル システムにコピーします。

    ```powershell
    $source = '\\appservicefileshare.local.cloudapp.azurestack.external\websites'
    $date = Get-Date -Format 'yyyyMMdd'
    $destination = "C:\Backups\FileShare\$date"
    New-Item -ItemType Directory -Path $destination -Force

    $fileshareusername = 'fileshareowner'
    $fileshareuserpassword = ConvertTo-SecureString 'Pa55w.rd1234' -AsPlainText -Force
    $fileshareusercreds = New-Object System.Management.Automation.PSCredential ($fileshareusername, $fileshareuserpassword)
    New-PSDrive -Name 'S' -Root $source -PSProvider 'FileSystem' -Credential $fileshareusercreds
    robocopy $source $destination /e /r:1 /w:1
    Remove-PSDrive -Name 'S'
    ```

>**確認**: この演習では、Azure Stack Hub 上の App Service リソース プロバイダーの管理タスクについて学習しました。


### 演習 3: App Service リソースを Azure Stack Hub に提供する

この演習では、App Service リソースを Azure Stack Hub ユーザーに提供します。

1. (クラウド オペレーターとして) ユーザーが App Service リソースを使用できるようにする
1. (ユーザーとして) Web アプリを作成する


### タスク 1: (クラウド オペレーターとして) ユーザーが App Service リソースを使用できるようにする

このタスクでは、次のことを行います。

- (クラウド オペレーターとして) ユーザーが App Service リソースを使用できるようにする

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、[Azure Stack Hub 管理者ポータル](https://adminportal.local.azurestack.external/)が表示されているブラウザーの画面に切り替えます。
1. Web ブラウザーの画面に Azure Stack Hub 管理者ポータルが表示されている状態で、「**+ リソースの作成**」をクリックします。
1. 「**新規作成**」ブレードで、「**オファーとプラン**」をクリックしてから「**プラン**」をクリックします。
1. 「**新しいプラン**」ブレードの「**基本**」タブで、次のように設定を行います。

    - 表示名: **app-service-plan1**
    - リソース名: **app-service-plan1**
    - リソース グループ: 新しいリソース グループの名前 **app-service-plans-RG**

1. 「**次へ: サービス >**」をクリックします。
1. 「**新しいプラン**」ブレードの「**サービス**」タブで、「**Microsoft.Web**」チェックボックスをオンにします。
1. 「**次へ: クォータ >**」をクリックします。
1. 「**新しいプラン**」ブレードの「**クォータ**」タブで「**新規作成**」を選択し、「**作成**」ブレードで以下のように設定してから「**OK**」をクリックします。

    - 名前: **app-service-quota1**
    - App Service プラン: **カスタム** **20**
    - 共有 App Service プラン: **カスタム** **10**
    - 専用 App Service プラン: **カスタム** **10**
    - 価格 SKU: **カスタム** **2 つ選択** (**無料**と**共有**)
    - 従量課金プラン**有効**

    >**注**: 従量課金プラン モデルで Azure Functions を提供するには、共有 Web worker をデプロイする必要があります。

1. 「**新しいプラン**」ブレードの「**クォータ**」タブに戻り、「**確認して作成**」をクリックしてから「**作成**」をクリックします。

    >**注**: デプロイが完了するまで待ちます。通常は数秒で完了します。

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、Web ブラウザーの画面に Azure Stack Hub 管理者ポータルが表示されている状態で、「**新規作成**」ブレードに戻って「**オファー**」クリックします。
1. 「**新しいオファーの作成**」ブレードの「**基本**」タブで、次のように設定を行います。

    - 表示名: **app-service-offer1**
    - リソース名: **app-service-offer1**
    - リソース グループ: **app-service-offers-RG**
    - このオファーをパブリックに設定する: **はい**

1. 「**次へ: 基本プラン >**」をクリックします。 
1. 「**新しいオファーの作成**」ブレードの「**基本プラン**」タブで、「**app-service-plan1**」エントリの横にあるチェックボックスをオンにします。
1. 「**次へ: アドオン プラン >**」をクリックします。
1. 「**アドオン プラン**」の設定は既定値にしたまま、「**確認して作成**」をクリックしてから「**作成**」をクリックします。

    >**注**: デプロイが完了するまで待ちます。通常は数秒で完了します。


#### タスク 2: (ユーザーとして) Web アプリを作成する

このタスクでは、次のことを行います。

- テスト用のユーザー アカウントを作成する
- (ユーザーとして) Web アプリを作成する

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
1. 「**サブスクリプションの取得**」ブレードの「**名前**」テキスト ボックスに「**t1u1-app-service-subscription1**」と入力します。
1. オファーの一覧で「**app-service-offer1**」を選択し、「**作成**」をクリックします。
1. 「**サブスクリプションが作成されました。サブスクリプションの使用を開始するに、ポータルを更新する必要があります**」のメッセージが表示されたら、「**更新**」をクリックします。 
1. Azure Stack Hub テナント パネルのハブ メニューで、「**+ リソースの作成**」をクリックします。
1. サービスの一覧で、「**Web とモバイル**」をクリックしてから「**Web アプリ**」をクリックします。 
1. 「**Web アプリ**」ブレードで、次のように設定します。

    - サブスクリプション: **t1u1-app-service-subscription1**
    - アプリ名: **t1u1webapp1**
    - リソース グループ: 新しいリソース グループの名前 **webapps-RG**

1. 「**Web アプリ**」ブレードで「**App Service プラン/場所**」をクリックしてから、「**App Service プラン**」ブレードで「**+ 新規作成**」をクリックします。 
1. 「**新しい App Service プラン**」ブレードで、次のように設定します。

    - App Service プラン: **appserviceplan1**
    - 場所: **ローカル**

1. 「**新しい App Service プラン**」ブレードで「**価格レベル**」をクリックします。
1. 「**スペック ピッカー**」ブレードで、「**F1**」価格レベルを選択して「**適用**」をクリックします。
1. 「**新しい App Service プラン**」ブレードに戻って「**OK**」をクリックします。
1. 「**Web アプリ**」ブレードに戻って「**作成**」をクリックします。

    >**注**: デプロイが完了するまで待ちます。通常は 1 分もかかりません。

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、Web ブラウザーの InPrivate セッションに Azure Stack Hub ユーザー ポータルが表示されている状態で、ハブ メニューの「**すべてのリソース**」を選択します。
1. 「**すべてのリソース**」ブレードの「サブスクリプション フィルター」ドロップダウン リストで、「**t1u1-app-service-subscription1**」エントリを選択してから「**更新**」をクリックします。
1. 「**すべてのリソース**」ブレードのリソースの一覧で、「**t1u1webapp1**」エントリをクリックします。
1. 「**t1u1webapp1**」ブレードで「**参照**」をクリックします。

    >**注**: これで別のブラウザーが開き、新たにプロビジョニングした Web アプリの既定のホーム ページが表示されます。

>**確認**: この演習では、App Service をユーザーが利用できるように設定したほか、Web アプリをテナント ユーザーとして作成しました。

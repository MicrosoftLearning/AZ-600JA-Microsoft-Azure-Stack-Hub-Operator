---
lab:
    title: 'Lab: Azure Stack Hub で Azure Resource Manager (ARM) テンプレートを検証する'
    module: 'モジュール 2: サービスを提供する'
---

# ラボ - Azure Stack Hub で Azure Resource Manager (ARM) テンプレートを検証する
# 受講生用ラボ マニュアル

## ラボの依存関係

- なし

## 推定所要時間

30 分

## ラボ シナリオ

あなたは Azure Stack Hub 環境のオペレーターとして、既存の Azure Resource Manager (ARM) テンプレートを使用して、Azure Stack Hub リソースのデプロイを自動化する必要があります。 

## 目標

このラボを終了すると、次のことができるようになります。

- Azure Stack Hub デプロイ用の ARM テンプレートを検証する

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


### 演習 1: Azure Stack Hub を使用して ARM テンプレートを検証する

この演習では、Azure Stack Hub を使用して ARM テンプレートを検証します。

1. クラウド機能ファイルを構築する 
1. テンプレート検証の成功例を実行する
1. テンプレート検証の失敗例を実行する
1. テンプレートに関する問題を修正する


#### タスク 1: クラウド機能ファイルを構築する

このタスクでは、次のことを行います。

- クラウド機能ファイルを構築する

1. 必要であれば、次の資格情報を使用して **AzS-HOST1** にサインインします。

    - ユーザー名: **AzureStackAdmin@azurestack.local**
    - パスワード: **Pa55w.rd1234**

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

1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」ウィンドウで以下のコマンドを実行して、Azure Stack Hub オペレーターの PowerShell 環境を登録します。

    ```powershell
    Add-AzEnvironment -Name 'AzureStackAdmin' -ArmEndpoint 'https://adminmanagement.local.azurestack.external' `
       -AzureKeyVaultDnsSuffix adminvault.local.azurestack.external `
       -AzureKeyVaultServiceEndpointResourceId https://adminvault.local.azurestack.external

1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」ウィンドウで以下のコマンドを実行して、新たに登録した **AzureStackAdmin** 環境にサインインします。

    ```powershell
    Connect-AzAccount -EnvironmentName 'AzureStackAdmin'
    ```

1. **CloudAdmin@azurestack.local** アカウントを使用して認証するよう求められたら、これを実行します。
1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」ウィンドウで以下のコマンドを実行して、Azure Stack ツールをダウンロードします。

    ```powershell
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    Set-Location -Path 'C:\'
    Invoke-WebRequest https://github.com/Azure/AzureStack-Tools/archive/az.zip -OutFile az.zip
    Expand-Archive az.zip -DestinationPath . -Force
    Set-Location -Path '\AzureStack-Tools-az'
    ```

    >**注**: この手順により、Azure Stack Hub ツールをホストするための GitHub リポジトリが含まれるアーカイブがローカル コンピューターにコピーされ、アーカイブが「**C:\\AzureStack-Tools-master**」フォルダーに展開されます。このツールには PowerShell モジュールが含まれています。このモジュールによって、Azure Stack Hub Resource Manager テンプレートの検証をはじめとする、さまざまな機能が提供されます。 

1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」ウィンドウで以下のコマンドを実行して、AzureRM.CloudCapabilities PowerShell モジュールをインポートします。

    ```powershell
    Import-Module .\CloudCapabilities\Az.CloudCapabilities.psm1 -Force
    ```

1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」ウィンドウで以下のコマンドを実行して、クラウド機能 JSON ファイルを生成します。 

    ```powershell
    $path = 'C:\Templates'
    New-Item -Path $path -ItemType Directory -Force
    Get-AzCloudCapability -Location 'local' -OutputPath $path\AzureCloudCapabilities.Json -Verbose
    ```

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、エクスプローラーを使用して「**C:\\Templates**」フォルダーに移動し、「**AzureCloudCapabilities.Json**」ファイルが正常に作成されたことを確認します。

#### タスク 2: テンプレート検証の成功例を実行する

このタスクでは、次のことを行います。

- Azure Stack Hub Quickstart テンプレートに対してテンプレート検証ツールを実行する

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、Web ブラウザーを起動して Azure Stack Hub QuickStart Templates リポジトリの「[**AzureStack 用 Windows 上の MySql Server**](https://github.com/Azure/AzureStack-QuickStart-Templates/tree/master/mysql-standalone-server-windows)」ページに移動します。 
1. 「**AzureStack 用 Windows 上の MySql Server**」ページで「**azuredeploy.json**」をクリックします。
1. 「[AzureStack-QuickStart-Templates/mysql-standalone-server-windows/azuredeploy.json](https://github.com/Azure/AzureStack-QuickStart-Templates/blob/master/mysql-standalone-server-windows/azuredeploy.json)」ページで、テンプレートの内容を確認します。
1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」ウィンドウに切り替え、以下のコマンドを実行して「azuredeploy.json」ファイルをダウンロードし、これに「**sampletemplate1.json**」と名前を付けて「**C:\\Templates**」フォルダーに保存します。

    ```powershell
    Invoke-WebRequest -Uri 'https://raw.githubusercontent.com/Azure/AzureStack-QuickStart-Templates/master/mysql-standalone-server-windows/azuredeploy.json' -UseBasicParsing -OutFile $path\sampletemplate1.json
    ```

1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」ウィンドウで以下のコマンドを実行して、AzureRM.TemplateValidator PowerShell モジュールをインポートします。

    ```powershell
    Set-Location -Path 'C:\AzureStack-Tools-az\TemplateValidator'
    Import-Module .\AzureRM.TemplateValidator.psm1 -Force
    ```
  
1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」ウィンドウで以下のコマンドを実行して、テンプレートを検証します。

    ```powershell
    Test-AzureRMTemplate `
        -TemplatePath $path `
        -TemplatePattern sampletemplate1.json `
        -CapabilitiesPath $path\AzureCloudCapabilities.json `
        -IncludeComputeCapabilities `
        -IncludeStorageCapabilities `
        -Report sampletemplate1validationreport.html `
        -Verbose
    ```

1. 検証の出力に目を通し、問題がないことを確認します。

    >**注**: 出力は次の形式でなければなりません。

    ```
    Validation Summary:
        Passed: 1
        NotSupported: 0
        例外: 0
        Warning: 0
        Recommend: 0
        Total Templates: 1
    Report available at - C:\AzureStack-Tools-az\TemplateValidator\sampletemplate1validationreport.html
    ```

#### タスク 3: テンプレート検証の失敗例を実行する

このタスクでは、次のことを行います。

- Azure Quickstart テンプレートに対してテンプレート検証ツールを実行する

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、AzureStack QuickStart テンプレート レポジトリが表示されている Web ブラウザーから、Azure QuickStart テンプレート レポジトリの「[**Ubuntu VM 上の MySQL Server 5.6**](https://github.com/Azure/azure-quickstart-templates/tree/master/mysql-standalone-server-ubuntu)」ページに移動します。
1. 「**Ubuntu VM 上の MySQL Server**」ページで「**azuredeploy.json**」をクリックします。
1. 「[azure-quickstart-templates/mysql-standalone-server-ubuntu/azuredeploy.json](https://github.com/Azure/azure-quickstart-templates/blob/master/mysql-standalone-server-ubuntu/azuredeploy.json)」ページで、テンプレートの内容を確認します。
1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」ウィンドウに切り替え、以下のコマンドを実行して「azuredeploy.json」ファイルをダウンロードし、これに「**sampletemplate2.json**」と名前を付けて「**C:\\Templates**」フォルダーに保存します。

    ```powershell
    Invoke-WebRequest -Uri 'https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/mysql-standalone-server-ubuntu/azuredeploy.json' -UseBasicParsing -OutFile $path\sampletemplate2.json
    ```

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、Web ブラウザーから[仮想マシン](https://docs.microsoft.com/ja-jp/rest/api/compute/virtualmachines)用の REST API リファレンスに移動し、Azure API の最新バージョンを確認します (本コンテンツの作成時点では **2020-12-01**)。 
1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」ウィンドウで以下のコマンドを実行して、「**sampletemplate2.json**」ファイルをメモ帳で開きます。

    ```powershell
    notepad.exe $path\sampletemplate2.json
    ```

1. メモ帳の画面に「**sampletemplate2.json**」ファイルの内容が表示されている状態で、テンプレートの `resources` セクションから、仮想マシン リソースを表すセクション (以下の形式) を探します。

    ```json
    {
      "apiVersion": "2017/03/30",
      "type": "Microsoft.Compute/virtualMachines",
      "name": "[variables('vmName')]",
      "location": "[parameters('location')]",
      "dependsOn": [
        "[concat('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]",
        "[concat('Microsoft.Network/networkInterfaces/', variables('nicName'))]"
      ],
    ```

1. `apiVersion` キーの値を、仮想マシンの Azure REST API の最新バージョン (このタスクの前半で確認したバージョン) に設定して変更内容を保存しますが、ファイルは開いたままにします。

    >**注**: 変更を加える前に、元の値を記録しておいてください。この値は、次のタスクで元に戻す必要がなります。

    >**注**: REST API のバージョンが **2020-12-01** の場合、変更を加えることで以下のような結果になります。

    ```json
    {
      "apiVersion": "2020/12/01",
      "type": "Microsoft.Compute/virtualMachines",
      "name": "[variables('vmName')]",
      "location": "[parameters('location')]",
      "dependsOn": [
        "[concat('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]",
        "[concat('Microsoft.Network/networkInterfaces/', variables('nicName'))]"
      ],
    ```

    >**注**: これは、構成を意図的に実装するためのものですが、Azure Stack Hub ではまだサポートされていません。

1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」ウィンドウで以下のコマンドを実行して、新たに修正したテンプレートを検証します。

    ```powershell
    Test-AzureRMTemplate `
        -TemplatePath $path `
        -TemplatePattern sampletemplate2.json `
        -CapabilitiesPath $path\AzureCloudCapabilities.json `
        -IncludeComputeCapabilities `
        -IncludeStorageCapabilities `
        -Report sampletemplate2validationreport.html `
        -Verbose
    ```

1. 検証の出力に目を通し、今回はサポートの問題が発生していることに確認します。

    >**注**: 出力は次の形式でなければなりません。

    ```
    Validation Summary:
        Passed: 0
        NotSupported: 1
        Exception: 0
        Warning: 0
        Recommend: 0
        Total Templates: 1
    Report available at - C:\AzureStack-Tools-az\TemplateValidator\sampletemplate2validationreport.html
    ```

1. 「**C:\AzureStack-Tools-az\TemplateValidator\sampletemplate2validationreport.html**」ファイルを開いて、レポートを確認します。 

    >**注**: レポートには、次の形式のエントリが含まれている必要があります。 **NotSupported: apiversion (Resource type: Microsoft.Compute/virtualMachines). Not Supported Values - 2020-12-01**.


#### タスク 4: テンプレートに関する問題を修正する

このタスクでは、次のことを行います。

- テンプレートに関する問題を修正する

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、エクスプローラーを使用して「**C:\\Templates**」フォルダーに移動し、「**AzureCloudCapabilities.json**」ファイルを開きます。
1. 「**AzureCloudCapabilities.json**」ファイルで、`"ResourceTypeName": "virtualMachines",` セクションを探します。これは次の形式になっています。

    ```json
    {
      "ProviderNamespace": "Microsoft.Compute",
      "ResourceTypeName": "virtualMachines",
      "Locations": [
        "local"
      ],
      "ApiVersions": [
        "2020/06/01",
        "2019/12/01",
        "2019/07/01",
        "2019/03/01",
        "2018/10/01",
        "2018/06/01",
        "2018/04/01",
        "2017/12/01",
        "2017/03/30",
        "2016/08/30",
        "2016/03/30",
        "2015/11/01",
        "2015/06/15"
      ],
      "ApiProfiles": [
        "2017-03-09-profile",
        "2018-03-01-hybrid"
      ]
    },
    ```

1. 「**Sampletemplate2.json**」ファイルに切り替え、前のタスクで修正した REST API バージョンを元の値に戻します。このバージョンが、上記の「**AzureCloudCapabilities.json**」の API バージョンと一致していることを確認してから、変更内容を保存します。
1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」ウィンドウに切り替え、以下のコマンドを実行して、新たに修正したテンプレートを検証します。

    ```powershell
    Test-AzureRMTemplate `
        -TemplatePath $path `
        -TemplatePattern sampletemplate2.json `
        -CapabilitiesPath $path\AzureCloudCapabilities.json `
        -IncludeComputeCapabilities `
        -IncludeStorageCapabilities `
        -Report sampletemplate2validationreport.html `
        -Verbose
    ```

1. 検証の出力に目を通し、今回は問題が発生していないことを確認します。

>**確認**: この演習では、クラウド機能ファイルを作成し、これを使用して Azure Resource Manager テンプレートを検証しました。また、検証の結果をもとに、テンプレートの修正も行いました。

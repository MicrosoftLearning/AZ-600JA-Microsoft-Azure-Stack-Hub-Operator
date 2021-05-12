---
lab:
    title: 'Lab: Azure Gallery Packager を使用してカスタム Marketplace 項目を追加する'
    module: 'モジュール 2: サービスを提供する'
---

# ラボ - Azure Gallery Packager を使用してカスタム Marketplace 項目を追加する
# 受講生用ラボ マニュアル

## ラボの依存関係

- なし 

## 推定所要時間

45 分

## ラボ シナリオ

あなたは Azure Stack Hub 環境のオペレーターとして、Azure Gallery Packager ツールを使用して、Azure Stack のカスタム Marketplace 項目を作成する必要があります。

## 目標

このラボを終了すると、次のことができるようになります。

- Azure Gallery Packager を使用して、カスタム Azure Stack Hub Marketplace 項目を作成する

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

このラボでは、カスタム Azure Stack Marketplace 項目の作成に必要なソフトウェアをダウンロードしてインストールします。

### 演習 1: Azure Stack Hub Marketplace 項目を作成して発行する

この演習では、Azure Gallery Packager ツールを使用して、Azure Marketplace 項目をカスタマイズして発行します。

1. Azure Gallery Packager ツールと、パッケージのサンプルをダウンロードする
1. 既存の Azure Gallery Packager パッケージを修正する
1. パッケージを Azure Stack Hub ストレージ アカウントにアップロードする 
1. パッケージを Azure Stack Hub Marketplace に発行する
1. 発行した Azure Stack Hub Marketplace 項目の可用性を確認する

#### タスク 1: Azure Gallery Packager ツールと、パッケージ ファイルのサンプルをダウンロードする

このタスクでは、次のことを行います。

- Azure Gallery Packager ツールと、パッケージ ファイルのサンプルをダウンロードする

1. 必要であれば、次の資格情報を使用して **AzS-HOST1** にサインインします。

    - ユーザー名: **AzureStackAdmin@azurestack.local**
    - パスワード: **Pa55w.rd1234**

1. **AzS-HOST1** へのリモート デスクトップ セッション内で Web ブラウザーの画面を開き、「[Azure Gallery Packager ツールのダウンロード](https://aka.ms/azsmarketplaceitem)」ページに移動して、「**Microsoft Azure Stack Gallery Packaging Tool and Sample 3.0.zip**」アーカイブ ファイルを「**Downloads**」フォルダーにダウンロードします。
1. ダウンロードが完了したら、zip ファイル内の「**Packager**」フォルダーを **C:\\Downloads** フォルダー (必要であれば作成します) に展開します。


#### タスク 2: 既存の Azure Gallery Packager パッケージを修正する

このタスクでは、次のことを行います。

- (Windows Server 2016 ではなく) Windows Server 2019 イメージが使用されるよう、既存の Azure Gallery Packager パッケージを修正する

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、エクスプローラーを使用して「**C:\\Downloads\\Packager\\Samples for Packager**」フォルダーに移動し、「**Sample.SingleVMWindowsSample.1.0.0.azpkg**」パッケージを「**C:\\Downloads**」フォルダーにコピーしてから、その拡張子を **zip** に変更します。
1. 「**Sample.SingleVMWindowsSample.1.0.0.zip**」アーカイブのコンテンツを「**C:\\Downloads\\SamplePackage**」フォルダーに展開します (最初にフォルダーを作成する必要があります)。
1. エクスプローラーを使用して「**C:\\Downloads\\SamplePackage\\DeploymentTemplates**」フォルダーに移動し、「**createuidefinition.json**」ファイルをメモ帳で開きます。
1. ファイルの「**imageReference**」セクションにある「**sku**」パラメーターを、先ほど ASDK インスタンスにダウンロードした 2019 Datacenter Core イメージが参照されるように修正します。

    ```json
    "imageReference": {
      "publisher": "MicrosoftWindowsServer",
      "offer": "WindowsServer",
      "sku": "2019-Datacenter-Core-smalldisk"
    },
    ```

1. 変更内容を保存し、メモ帳を閉じます。
1. エクスプローラーを使用して「**C:\\Downloads\\SamplePackage\\strings**」フォルダーに移動し、「**resources.resjson**」ファイルをメモ帳で開きます。
1. キーと値のペアを次のように設定することで、「**resources.resjson**」ファイルのコンテンツを修正します。

    - displayName: **Custom Windows Server 2019 Core VM**
    - publisherDisplayName: **Contoso**
    - summary: **Custom Windows Server 2019 Core VM (small disk)**
    - longSummary: **Custom Contoso Windows Server 2019 Core VM (small disk)**
    - description: **<P>カスタマイズした Windows Server 2019 Azure Stack Hub VM のサンプル</p><p>Azure Stack Hub Quickstart テンプレートに基づくもの</p>**
    - documentationLink: **Documentation**

    >**注**: これにより、次のコンテンツが生成されます。

    ```json
    {
      "displayName": "Custom Windows Server 2019 Core VM",
      "publisherDisplayName": "Contoso",
      "summary": "Custom Windows Server 2019 Core VM (small disk)",
      "longSummary": "Custom Contoso Windows Server 2019 Core VM (small disk)",
      "description": "<p>カスタマイズした Windows Server 2019 Azure Stack Hub VM のサンプル</p><p>Azure Stack Hub Quickstart テンプレートに基づくもの</p>",
      "documentationLink": "Documentation"
    }
    ```

1. 変更内容を保存し、メモ帳を閉じます。
1. エクスプローラーを使用して「**C:\\Downloads\\SamplePackage**」フォルダーに移動し、「**manifest.json**」ファイルをメモ帳で開きます。
1. キーと値のペアを次のように設定することで、「**manifest.json**」ファイルのコンテンツを修正します。

    - name: **CustomVMWindowsSample**
    - publisher: **Contoso**
    - version: **1.0.1**
    - displayName: **Custom Windows Server 2019 Core VM**
    - publisherDisplayName: **Contoso**
    - publisherLegalName: **Contoso Inc.**
    - uri of the documentation link: **https://docs.contoso.com**

    >**注**: これにより、次のコンテンツが生成されます (ファイルの行 2 から開始)。

    ```json
        "$schema": "https://gallery.azure.com/schemas/2015-10-01/manifest.json#",
        "name": "CustomVMWindowsSample",
        "publisher": "Contoso",
        "version": "1.0.1",
        "displayName": "Custom Windows Server 2019 Core VM",
        "publisherDisplayName": "Contoso",
        "publisherLegalName": "Contoso Inc.",
        "summary": "ms-resource:summary",
        "longSummary": "ms-resource:longSummary",
        "description": "ms-resource:description",
        "longDescription": "ms-resource:description",
	    "uiDefinition": {
		"path": "UIDefinition.json"
	},
        "links": [
            { "displayName": "ms-resource:documentationLink", "uri": "https://docs.contoso.com/" }
        ], 
    ```

1. 変更内容を保存し、メモ帳を閉じます。


#### タスク 3: カスタマイズした Azure Gallery Packager パッケージを生成する

このタスクでは、次のことを行います。

- 新たにカスタマイズした Azure Gallery Packager パッケージを再生成する

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、**コマンド プロンプト**を起動します。

1. **コマンド プロンプト**で以下のコマンドを実行して、現在のディレクトリを変更します。

    ```cmd
    cd C:\Downloads\Packager
    ```

1. **コマンド プロンプト**で以下のコマンドを実行して、前のタスクで修正したコンテンツに基づいて、新しいパッケージを生成します。

    ```cmd
    AzureStackHubGallery.exe package -m C:\Downloads\SamplePackage\manifest.json -o C:\Downloads\
    ```

1. 「**Contoso.CustomVMWindowsSample.1.0.1.azpkg**」パッケージが自動的に「**C:\\Downloads**」フォルダーに保存されたことを確認します。


#### タスク 4: パッケージを Azure Stack Hub ストレージ アカウントにアップロードする

このタスクでは、次のことを行います。

- Azure Stack Hub Marketplace 項目パッケージを Azure Stack Hub ストレージ アカウントにアップロードする

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、Web ブラウザーを開いて画面に [Azure Stack Hub 管理者ポータル](https://adminportal.local.azurestack.external/)を表示させ、CloudAdmin@azurestack.local としてサインインします。
1. Web ブラウザーの画面に Azure Stack Hub 管理者ポータルが表示されている状態で、「**+ リソースの作成**」をクリックします。 
1. 「**新規**」ブレードで「**データとストレージ**」をクリックします。
1. 「**データとストレージ**」ブレードで「**ストレージ アカウント**」をクリックします。
1. 「**ストレージ アカウントの作成**」ブレードの「**基本**」タブで、次のように設定を行います。

    - サブスクリプション: **既定のプロバイダー サブスクリプション
    - リソース グループ: 新しいリソース グループの名前 **marketplace-pkgs-RG**
    - 名前: 3～24 の小文字または数字から成る一意の名前
    - 場所: **ローカル**
    - パフォーマンス: **Standard**
    - アカウントの種類: **ストレージ (汎用 v1)**
    - レプリケーション: **ローカル冗長ストレージ (LRS)**

1. 「**ストレージ アカウントの作成**」ブレードの「**基本**」タブで、「**次へ: 詳細 >**」をクリックします。
1. 「**ストレージ アカウントの作成**」ブレードの「**詳細**」タブで、設定を規定値にしたまま「**確認して作成**」をクリックします。
1. 「**ストレージ アカウントの作成**」ブレードの「**確認して作成**」タブで、「**作成**」を選択します。

    >**注**: ストレージ アカウントがプロビジョニングされるまで待ちます。通常 1 分ほどかかります。

1. Web ブラウザーの画面に Azure Stack Hub 管理者ポータルが表示されている状態で、ハブ メニューで「**リソース グループ**」を選択します。
1. 「**リソース グループ**」ブレードのリソース グループの一覧で、「**marketplace-pkgs-RG**」エントリをクリックします。
1. 「**marketplace-pkgs-RG**」ブレードで、新たに作成したストレージ アカウントを表すエントリをクリックします。
1. 「ストレージ アカウント」ブレードで「**コンテナー**」をクリックします。
1. 「コンテナー」ブレードで「**+ コンテナー**」をクリックします。
1. 「**新しいコンテナー**」ブレードの「**名前**」テキスト ボックスに「**gallerypackages**」と入力し、「**パブリック アクセス レベル**」ドロップダウン リストから「**BLOB (BLOB 用の匿名読み取りアクセスのみ)**」を選択し、「**作成**」を選択します。
1. 「コンテナー」ブレードに戻り、新たに作成したコンテナーを表す「**gallerypackages**」エントリをクリックします。
1. 「**gallerypackages**」ブレードで「**アップロード**」をクリックします。
1. 「**BLOB のアップロード**」ブレードで、「**ファイルの選択**」テキスト ボックスの横にあるフォルダー アイコンをクリックします。 
1. 「**開く**」ダイアログ ボックスで「**C:\Downloads**」フォルダーに移動し、「**Contoso.CustomVMWindowsSample.1.0.1.azpkg**」パッケージ ファイルを選択して「**開く**」をクリックします。
1. 「**BLOB のアップロード**」ブレードに戻り、「**アップロード**」をクリックします。


#### タスク 5: パッケージを Azure Stack Hub Marketplace に発行する

このタスクでは、次のことを行います。

- パッケージを Azure Stack Hub Marketplace に発行する

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、管理者として PowerShell 7 を起動します。
1. **AzS-HOST1** へのリモート デスクトップ セッション内で、「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」プロンプトから以下のコマンドを実行して、このラボに必要な Azure Stack Hub PowerShell モジュールをインストールします。 

    ```powershell
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    Install-Module -Name Az.BootStrapper -Force -AllowPrerelease -AllowClobber
    Install-AzProfile -Profile 2019-03-01-hybrid -Force
    Install-Module -Name AzureStack -RequiredVersion 2.0.2-preview -AllowPrerelease
    ```

    >**注**: すでに利用可能となっているコマンドに関するエラー メッセージはすべて無視してください。

1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」プロンプトで以下のコマンドを実行して、Azure Stack Hub オペレーター環境を登録します。

    ```powershell
    Add-AzEnvironment -Name 'AzureStackAdmin' -ArmEndpoint 'https://adminmanagement.local.azurestack.external' `
       -AzureKeyVaultDnsSuffix adminvault.local.azurestack.external `
       -AzureKeyVaultServiceEndpointResourceId https://adminvault.local.azurestack.external
    ```

1. Azure Stack Hub 管理者ポータルへの既存のブラウザー セッションを活用するため、「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」プロンプトで以下のコマンドを実行して、AzureStack\CloudAdmin 資格情報を使用して Azure Stack Hub オペレーターの PowerShell 環境にサインインします。

    ```powershell
    Connect-AzAccount -EnvironmentName 'AzureStackAdmin'
    ```

    >**注**: これによって別のブラウザー タブが開き、認証に成功したことを伝えるメッセージが表示されます。

1. ブラウザー タブを閉じて「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」ウィンドウに戻り、**CloudAdmin@azurestack.local** として認証されたことを確認します。
1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」で以下のコマンドを実行して、パッケージを Azure Stack Hub Marketplace に公開します (`<storage_account_name>` プレースホルダーは、前のタスクで割り当てたストレージ アカウントの名前を表します)。

    ```powershell
    Add-AzsGalleryItem -GalleryItemUri `
    https://<storage_account_name>.blob.local.azurestack.external/gallerypackages/Contoso.CustomVMWindowsSample.1.0.1.azpkg -Verbose
    ```

1. **Add-AzsGalleryItem** コマンドの出力を確認して、コマンドが正常に完了したことを確認します。


#### タスク 6: 発行した Azure Stack Hub Marketplace 項目の可用性を確認する

このタスクでは、次のことを行います。

- 発行した Azure Stack Hub Marketplace 項目の可用性を確認する

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、Web ブラウザーを開き、画面に [Azure Stack Hub ユーザー ポータル](https://portal.local.azurestack.external)を表示させます。
1. Web ブラウザーの画面に Azure Stack Hub 管理者ポータルが表示されている状態で、「**Marketplace**」をクリックします。 
1. 「**Marketplace**」ブレードで、「**コンピューティング**」をクリックしてから「**表示を増やす**」をクリックします。
1. 「**Marketplace**」ブレードで、「**Custom Windows Server 2019 Core VM**」が選択の一覧に表示されていることを確認します。 

    >**注**: 新しい Azure Stack Hub Marketplace 項目が意図されたとおりに機能していることを確認するため、テンプレートによって参照される OS イメージといった、そのデプロイの前提条件についてもすべて満たされていることを確認する必要があります。このラボでは、この点については取り上げません。

>**確認**: この演習では、Azure Gallery Packager を使用して Azure Stack Hub Marketplace 項目をカスタムで作成し、これを **Add-AzsGalleryItem** コマンドレットを使用して発行しました。

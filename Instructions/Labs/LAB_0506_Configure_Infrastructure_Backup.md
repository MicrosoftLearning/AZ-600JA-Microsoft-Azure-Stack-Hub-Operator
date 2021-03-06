---
lab:
    title: 'ラボ: Azure Stack Hub インフラストラクチャ バックアップを構成する'
    module: 'モジュール 5: インフラストラクチャを管理する'
---

# ラボ - Azure Stack Hub インフラストラクチャ バックアップを構成する
# 受講生用ラボ マニュアル

## ラボの依存関係

- なし

## 推定所要時間

30 分

## ラボ シナリオ

あなたは Azure Stack Hub 環境のオペレーターとして、インフラストラクチャ バックアップができるよう準備する必要があります。 

## 目標

このラボを終了すると、次のことができるようになります。

- Azure Stack Hub インフラストラクチャ バックアップを構成する

## ラボ環境 

このラボでは、Active Directory フェデレーション サービス (AD FS) (ID プロバイダーとしてバックアップされた Active Directory) に統合された ADSK インスタンスを使用します。 

ラボ環境は、次のコンポーネントから構成されています。

- 次のアクセス ポイントを持つ、**AzSHOST-1** サーバーで実行されている ASDK デプロイ。

  - 管理者ポータル: https://adminportal.local.azurestack.external
  - 管理者 ARM エンドポイント: https://adminmanagement.local.azurestack.external
  - ユーザー ポータル: https://portal.local.azurestack.external
  - ユーザー ARM エンドポイント: https://management.local.azurestack.external

- 管理ユーザー:

  - ASDK クラウド オペレーターのユーザー名: **CloudAdmin@azurestack.local**
  - ASDK クラウド オペレーターのパスワード: **Pa55w.rd1234**
  - ASDK ホスト管理者のユーザー名: **AzureStackAdmin@azurestack.local**
  - ASDK ホスト管理者のパスワード: **Pa55w.rd1234**

このラボでは、PowerShell を介して Azure Stack Hub を管理するために必要なソフトウェアをインストールします。追加のユーザー アカウントも作成します。

### 演習 1: Azure Stack Hub インフラストラクチャ バックアップを構成する

この演習では、Azure Stack Hub インフラストラクチャ バックアップを構成する準備を行います。

1. バックアップ ユーザーを作成する
1. バックアップ共有を作成する
1. 暗号化キーを生成する
1. バックアップ コントローラーを構成する

#### タスク 1: バックアップ ユーザーを作成する

このタスクでは、次のことを行います。

- バックアップ ユーザーを作成する

1. 必要であれば、次の資格情報を使用して **AzSHOST1** にサインインします。

    - ユーザー名: **AzureStackAdmin@azurestack.local**
    - パスワード: **Pa55w.rd1234**

1. **AzS-HOST1** へのリモート デスクトップ セッション内でスタート メニューの「**スタート**」をクリックし、「**Windows 管理ツール**」をクリックしてから、管理ツールの一覧で「**Active Directory 管理センター**」をダブルクリックします。
1. 「**Active Directory 管理センター**」コンソールで「**azurestack (ローカル)**」をクリックします。
1. 詳細ペインで「**ユーザー**」コンテナーをダブルクリックします。
1. 「**タスク**」ペインの「**ユーザー**」セクションで、**「新規作成」>「ユーザー」** の順にクリックします。
1. 「**ユーザーの作成**」ウィンドウで、次のように設定してから「**OK**」をクリックします。 

    - 完全名: **AzS-BackupOperator**
    - ユーザー UPN ログオン: **AzS-BackupOperator@azurestack.local**
    - ユーザーの SamAccountName: **azurestack\AzS-BackupOperator**
    - パスワード: **Pa55w.rd**
    - パスワード オプション: **その他のパスワード オプション -> パスワードを無期限にする**

#### タスク 2: バックアップ共有を作成する

このタスクでは、次のことを行います。

- バックアップ共有を作成する 

    >**注**: ラボ以外のシナリオでは、この共有は Azure Stack Hub デプロイの外部に作成されます。このラボでは簡素化のため、共有を Azure Stack Hub ホストに直接作成します。

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、エクスプローラーを起動します。 
1. エクスプローラーを使用して、新しいフォルダー「**C:\\Backup**」を作成します。
1. エクスプローラーで「**Backup**」フォルダーを右クリックし、右クリック メニューで「**プロパティ**」をクリックします。
1. 「**Backup プロパティ**」ウィンドウで、「**共有**」タブをクリックしてから「**詳細な共有**」をクリックします。
1. 「**詳細な共有**」ダイアログ ボックスで、「**このフォルダーを共有する**」をクリックしてから「**アクセス許可**」をクリックします。
1. 「**Backup のアクセス許可**」ウィンドウで、「**すべてのユーザー**」エントリが選択されていることを確認してから「**削除**」をクリックします。
1. 「**追加**」をクリックし、「**ユーザー、コンピューター、サービス アカウント、またはグループの選択**」ダイアログ ボックスに「**AzS-BackupOperator**」と入力して「**OK**」をクリックします。
1. 「**AzS-BackupOperator**」エントリが選択されていることを確認してから、「**許可**」列の「**フル コントロール**」チェックボックスをクリックします。
1. 「**追加**」をクリックし、「**ユーザー、コンピューター、サービス アカウント、またはグループの選択**」ダイアログ ボックスで「**場所**」をクリックします。
1. 「**場所**」ダイアログ ボックスで、ローカル コンピューターを表すエントリ (**AzS-HOST1**) をクリックしてから「**OK**」をクリックします。
1. 「**選択するオブジェクト名を入力**」テキスト ボックスに「**Administrators**」と入力して「**OK**」をクリックします。
1. 「**Administrators**」エントリが選択されていることを確認してから、「**許可**」列の「**フル コントロール**」チェックボックスをクリックして、「**OK**」をクリックします。
1. 「**詳細な共有**」ダイアログ ボックスに戻り、「**OK**」をクリックします。
1. 「**Backup プロパティ**」ウィンドウに戻り、「**セキュリティ**」タブをクリックしてから「**編集**」をクリックします。
1. 「**追加**」をクリックし、「**ユーザー、コンピューター、サービス アカウント、またはグループの選択**」ダイアログ ボックスに「**AzS-BackupOperator**」と入力して「**OK**」をクリックします。
1. 「**Backup のアクセス許可**」ダイアログ ボックスで、「**グループまたはユーザー名**」ペインの「**AzS-BackupOperator**」をクリックし、「**AzS-BackupOperator のアクセス許可**」ペインの「**許可**」列で「**フル コントロール**」をクリックしてから、「**OK**」をクリックします。 
1. 「**Backup プロパティ**」ウィンドウに戻り、「**閉じる**」をクリックします。

#### タスク 3: 暗号化キーを生成する

このタスクでは、次のことを行います。

- 暗号化キーを生成する 

    >**注**: インフラストラクチャ バックアップはすべて暗号化する必要があるため、インフラストラクチャ バックアップを構成するには、暗号化キーのペアに対応する証明書を提示する必要があります。キーの生成には Windows PowerShell を使用します。 

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、管理者として PowerShell 7 を起動します。
1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」プロンプトで以下のコマンドを実行して、暗号化キーのペアと、それに対応する証明書を生成します。
    
    ```powershell
    $cert = New-SelfSignedCertificate `
               -DnsName "azsh.contoso.com" `
               -CertStoreLocation 'cert:\LocalMachine\My'

    New-Item -Path 'C:\' -Name 'CertStore' -ItemType 'Directory'

    Export-Certificate `
        -Cert $cert `
        -FilePath C:\CertStore\AzSHIBPK.cer 
    ```

    >**注**: Azure Stack Hub では、インフラストラクチャ バックアップにおいて自己署名証明書がサポートされています。復元中には秘密キーを提供する必要があるため、運用環境では、このキーを安全な場所に保管してください。


#### タスク 4: バックアップ コントローラーを構成する

このタスクでは、次のことを行います。

- バックアップ コントローラーを構成する

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、Web ブラウザーを開いて画面に [Azure Stack Hub 管理者ポータル](https://adminportal.local.azurestack.external/)を表示させ、CloudAdmin@azurestack.local としてサインインします。
1. Azure Stack Hub 管理者ポータルで「**すべてのサービス**」を選択します。
1. 「**すべてのサービス**」ブレードで、「**管理**」を選択してから「**インフラストラクチャ バックアップ**」を選択します。 
1. 「**インフラストラクチャ バックアップ**」ブレードで「**構成**」をクリックします。
1. 「**バックアップ コントローラーの設定**」ブレードで、次のように設定してから「**OK**」をクリックします。

    - バックアップ ストレージの場所: **\\AzS-HOST1.azurestack.local\Backup**
    - ユーザー名: **AzS-BackupOperator@azurestack.local**
    - パスワード: **Pa55w.rd**
    - パスワードの確認: **Pa55w.rd**
    - バックアップ頻度 (時間単位): **12**
    - 保有期間 (日単位): **7**
    - 証明書 .cer ファイル: **C:\CertStore\AzSHIBPK.cer**

1. インフラストラクチャ バックアップが有効になったことを確認するには、Azure Stack Hub 管理者ポータルが表示されている Web ブラウザーのページを更新して、「**インフラストラクチャ バックアップ**」ブレードに戻ります。
1. 「**インフラストラクチャ バックアップ**」ブレードでバックアップ設定を確認します。
1. オプションとして、「**今すぐバックアップ**」をクリックしてオンデマンド バックアップを開始します。

    >**注**: バックアップ プロセスが完了するには 15 分以上かかり、その間は一時停止や停止することもできません。

>**確認**: この演習では、Azure Stack Hub インフラストラクチャ バックアップをホストする共有と、(Azure Stack Hub インフラストラクチャ バックアップから) この共有に接続するために用いる Azure Directory ユーザー アカウントを作成したほか、暗号化キーを生成し、バックアップ コントローラーを構成しました。 

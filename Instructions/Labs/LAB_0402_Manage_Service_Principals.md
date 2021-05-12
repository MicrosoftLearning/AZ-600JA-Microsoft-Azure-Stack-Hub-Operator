---
lab:
    title: 'Lab: Azure Stack Hub でサービス プリンシパルを管理する'
    module: 'モジュール 4: ID とアクセスを管理する'
---

# ラボ - Azure Stack Hub でサービス プリンシパルを管理する
# 受講生用ラボ マニュアル

## ラボの依存関係

- なし

## 推定所要時間

30 分

## ラボ シナリオ

あなたは Azure Stack Hub 環境のオペレーターとして、社内開発したアプリケーションを使用して Azure Stack Hub を管理しようと計画しています。このアプリケーションが認証されるには、サービス プリンシパルを作成し、既定のプロバイダー サブスクリプションのもとでこれを共同作成者ロールに割り当てる必要があります。

>**注**: アプリケーションのリソースのデプロイや構成を Azure Resource Manager を介して行う必要がある場合は、そのアプリケーションをその ID で表す必要があります。ユーザーは、ユーザー プリンシパルと呼ばれるセキュリティ プリンシパルを用いて表されるように、アプリはサービス プリンシパルを用いて表されます。サービス プリンシパルは、開発者が開発するアプリの ID となり、開発者は必要なアクセス許可のみをそのアプリに委任することができます。

アプリでは認証時に資格情報を提示する必要があります。この認証は、次の 2 つの要素で構成されます。

- アプリケーション ID。クライアント ID と呼ばれることもあります。Active Directory テナント内でのそのアプリの登録を一意に識別する GUID です。
- アプリケーション ID に関連付けられているシークレット。クライアント シークレット文字列 (パスワードに相当) を生成することも、X509 証明書を指定することもできます

このラボでは、シークレットを使用します。証明書を使用した認証について詳しくは、「[アプリ ID を使用して Azure Stack Hub リソースにアクセスする](https://docs.microsoft.com/ja-jp/azure-stack/operator/azure-stack-create-service-principals?view=azs-2008&tabs=az1%2Caz2&pivots=state-disconnected)」を参照してください。

## 目標

このラボを終了すると、次のことができるようになります。

- Azure Stack Hub AD FS 統合シナリオでサービス プリンシパルを作成して管理する

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

このラボで取り上げるアプローチは、AD FS 統合シナリオに特有のものです。Azure Active Directory (Azure AD) を ID プロバイダーとして使用する際に、サービス プリンシパル ベースの認証を導入する方法については、「[アプリ ID を使用して Azure Stack Hub リソースにアクセスする](https://docs.microsoft.com/ja-jp/azure-stack/operator/azure-stack-create-service-principals?view=azs-2008&tabs=az1%2Caz2&pivots=state-connected)」を参照してください。


### 演習 1: Azure Stack Hub でサービス プリンシパルを作成して構成する

この演習では、特権エンドポイントへの PowerShell リモート セッションを確立してサービス プリンシパルを作成し、Azure Stack Hub 管理者ポータルを使用してこれを共同作成者ロールに割り当てます。この演習は以下のタスクから構成されます。

1. サービス プリンシパルを作成する
1. サービス プリンシパルに共同作成者ロールを割り当てる


#### タスク 1: サービス プリンシパルを作成する

このタスクでは、次のことを行います。

- PowerShell を介して特権エンドポイントに接続し、サービス プリンシパルを作成する

1. 必要であれば、次の資格情報を使用して **AzS-HOST1** にサインインします。

    - ユーザー名: **AzureStackAdmin@azurestack.local**
    - パスワード: **Pa55w.rd1234**

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、管理者として PowerShell 7 を起動します。
1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」ウィンドウで以下のコマンドを実行して、特権エンドポイントへの PowerShell リモート セッションを確立します。

    ```powershell
    $session = New-PSSession -ComputerName AzS-ERCS01 -ConfigurationName PrivilegedEndpoint
    ```

1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」ウィンドウで以下のコマンドを実行して、新しいアプリ登録 (およびサービス プリンシパル オブジェクト) を作成し、これに対する参照を **$spObject** 変数に格納します。

    ```powershell
    $spObject = Invoke-Command -Session $session -ScriptBlock {New-GraphApplication -Name 'azsmgmt-app1' -GenerateClientSecret}
    ```

1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」ウィンドウで以下のコマンドを実行して、Azure Stack Hub スタンプ情報を取得し、これに対する参照を **$azureStackInfo** 変数に格納します。

    ```powershell
    $azureStackInfo = Invoke-Command -Session $session -ScriptBlock {Get-AzureStackStampInformation}
    ```

1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」ウィンドウで以下のコマンドを実行して、特権エンドポイントへの PowerShell リモート セッションを終了します。

    ```powershell
    $session | Remove-PSSession
    ```

    >**注**: 通常、特権エンドポイント セッションを終了するには **Close-PrivilegedEndpoint** コマンドレットを使用しますが、そのためにはファイル共有を設定してセッション トランスクリプト ログをホストしなくてはなりません。この演習では内容を簡素化するため、この正規の手順は実行しません。

1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」ウィンドウで以下のコマンドを実行することで、このタスクの前半で取得した Azure Stack Hub スタンプ情報を使用して、サービス プリンシパルを構成するための変数の値を設定します。その参照先はそれぞれ、Azure Resource Manager ユーザー操作用の Azure Stack Hub エンドポイント、Graph API へのアクセスに使用する Oauth トークンを取得するための対象者、そして ID プロバイダーの GUID となります。

    ```powershell
    $armUseEndpoint = $azureStackInfo.TenantExternalEndpoints.TenantResourceManager
    $graphAudience = "https://graph." + $azureStackInfo.ExternalDomainFQDN + "/"
    $tenantID = $azureStackInfo.AADTenantID
    ```

1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」ウィンドウで以下のコマンドを実行して、Azure Stack Hub ユーザー環境を登録および設定します。

    ```powershell
    Add-AzEnvironment -Name 'AzureStackUser' -ArmEndpoint $armUseEndpoint
    ```

1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」ウィンドウで以下のコマンドを実行して、サービス プリンシパルとして AzureStackUser 環境にサインインします。

    ```powershell
    $securePassword = $spObject.ClientSecret | ConvertTo-SecureString -AsPlainText -Force
    $credential = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $spObject.ClientId, $securePassword
    $spUserSignIn = Connect-AzAccount -Environment 'AzureStackUser' -ServicePrincipal -Credential $credential -TenantId $tenantID
    ```

1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」ウィンドウで以下のコマンドを実行して、サインインに成功したことを確認します。

    ```powershell
    $spUserSignIn
    ```

1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」ウィンドウで以下のコマンドを実行して、現在の認証コンテキストを削除します。

    ```powershell
    Remove-AzAccount -Username $credential.UserName
    ```

    >**注**: これ以降も、同等の手順を順次実行して、Azure Stack Hub 管理者環境の認証ができることを検証します。

1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」ウィンドウで以下のコマンドを実行することで、このタスクの前半で取得した Azure Stack Hub スタンプ情報を使用して、サービス プリンシパルを構成するための変数の値を設定します。その参照先はそれぞれ、Azure Resource Manager 管理者操作用の Azure Stack Hub エンドポイント、Graph API へのアクセスに使用する Oauth トークンを取得するための対象者、そして ID プロバイダーの GUID となります。

    ```powershell
    $armAdminEndpoint = $azureStackInfo.AdminExternalEndpoints.AdminResourceManager
    $graphAudience = "https://graph." + $azureStackInfo.ExternalDomainFQDN + "/"
    $tenantID = $azureStackInfo.AADTenantID
    ```

1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」ウィンドウで以下のコマンドを実行して、Azure Stack Hub 管理者環境を登録および設定します。

    ```powershell
    Add-AzEnvironment -Name 'AzureStackAdmin' -ArmEndpoint $armAdminEndpoint
    ```

1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」ウィンドウで以下のコマンドを実行して、サービス プリンシパルとして AzureStackAdmin 環境にサインインします。

    ```powershell
    $spAdminSignIn = Connect-AzAccount -Environment 'AzureStackAdmin' -ServicePrincipal -Credential $credential -TenantId $tenantID
    ```

1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」ウィンドウで以下のコマンドを実行して、サインインに成功したことを確認します。

    ```powershell
    $spAdminSignIn
    ```

1. 「**管理者: C:\Program Files\PowerShell\7\pwsh.exe**」ウィンドウで以下のコマンドを実行して、新しいサービス プリンシパルのプロパティを表示します。

    ```powershell
    $spObject
    ```

    >**注**: 出力は次の形式でなければなりません。

    ```
    ApplicationIdentifier : S-1-5-21-2657257302-3827180852-1812683747-1510
    ClientId              : 6a3fb4ad-838a-47b1-a93c-f3e4a1b683c8
    Thumbprint            :
    ApplicationName       : Azurestack-azsmgmt-app2-53508ec5-d7d9-4761-876a-3602542c2965
    ClientSecret          : 3fKPtUg37YraCk1IaFtdqeyTpVplXDqc25Dj3bUs
    PSComputerName        : AzS-ERCS01
    RunspaceId            : 6b142339-b67f-490e-a258-40983c0cd8ea
    ```

    >**注**: 「**ApplicationName**」プロパティの値を記録しておきます。これは、次のタスクで必要になります。また、「**ClientSecret**」プロパティの値を記録し、これを管理アプリケーションを実装している開発者に示す必要があります。


#### タスク 2: サービス プリンシパルに共同作成者ロールを割り当てる

このタスクでは、次のことを行います。

- Azure Stack Hub 管理者ポータルを使用して、サービス プリンシパルに共同作成者ロールを割り当てる

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、Web ブラウザーを開いて画面に [Azure Stack Hub 管理者ポータル](https://adminportal.local.azurestack.external/)を表示させ、CloudAdmin@azurestack.local としてサインインします。
1. Web ブラウザーに Azure Stack Hub 管理者ポータルが表示されている状態で、ハブ メニューで「**すべてのサービス**」を選択します。 
1. 「**すべてのサービス**」ブレードで「**全般**」を選択し、サービスの一覧で「**サブスクリプション**」を選択します。
1. 「**サブスクリプション**」ブレードで「**既定のプロバイダー サブスクリプション**」を選択します。
1. 「**既定のプロバイダー サブスクリプション**」ブレードで「**アクセス制御 (IAM)**」を選択します。
1. 「**既定のプロバイダー サブスクリプション | アクセス制御 (IAM)**」ブレードで「**+ 追加**」をクリックし、ドロップダウン メニューで「**ロール割り当ての追加**」をクリックします。
1. 「**ロール割り当ての追加**」ブレードで、次のように設定してから「**保存**」をクリックします。

    - ロール: **共同作成者**
    - アクセスの割り当て先: **Azure AD ユーザー、グループ、またはサービス プリンシパル**
    - 以下を選択: 前のタスクで確認したサービス プリンシパルの「**ApplicationName**」プロパティの値を検索して選択する

1. ロールの割り当てが成功したことを確認します。

>**確認**: この演習では、サービス プリンシパルを作成し、これを共同作成者ロールに割り当てました。 

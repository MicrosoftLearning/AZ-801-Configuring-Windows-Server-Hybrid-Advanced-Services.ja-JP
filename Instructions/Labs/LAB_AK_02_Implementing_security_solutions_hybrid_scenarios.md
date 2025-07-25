---
lab:
  title: 'ラボ: ハイブリッド シナリオでのセキュリティ ソリューションの実装'
  type: Answer Key
  module: 'Module 2: Implementing Security Solutions in Hybrid Scenarios'
---

# ラボ解答キー: ハイブリッド シナリオでのセキュリティ ソリューションの実装

## 演習 1: Azure Log Analytics ワークスペースを作成する

#### タスク 1: Azure Log Analytics ワークスペースを作成する 

1. **SEA-SVR2** に接続し、必要に応じて、講師から提供された資格情報でサインインします。
1. **SEA-SVR2** で Microsoft Edge を起動し、Azure portal (`https://portal.azure.com/`) に移動し、このラボで使用するサブスクリプションの所有者ロールを持つユーザー アカウントの資格情報を使用してサインインします。
1. **SEA-SVR2** で、Azure portal のツールバーの **[リソース、サービス、ドキュメントの検索]** テキスト ボックスで、 **[Log Analytics ワークスペース]** を検索して選択し、 **[Log Analytics ワークスペース]** ページで **[+ 作成]** を選択します。
1. **[Log Analytics ワークスペースの作成]** ページの **[基本]** タブで、次の設定を入力し、**[確認および作成]** を選択してから、**[作成]** を選択します。

   | 設定 | 値 |
   | --- | --- |
   | サブスクリプション | このラボで使用している Azure サブスクリプションの名前 |
   | リソース グループ | 新しいリソース グループの名前 **AZ801-L0201-RG** |
   | Log Analytics ワークスペース | 任意の一意の名前 |
   | リージョン | 最寄りのリージョンを選択します |

   >**注**: デプロイが完了するまで待ちます。 デプロイには約 1 分かかります。

## 演習 2: Microsoft Defender for Cloud の構成

#### タスク 1: Defender for Cloud とエージェントの自動インストールを有効にする

1. **SEA-SVR2** で、Azure portal のツールバーの **[リソース、サービス、ドキュメントの検索]** テキスト ボックスで、 **[Microsoft Defender for Cloud]** を検索して選択します。
1. **[Microsoft Defender for Cloud \|の概要]** ページで、**[ここをクリックしてアップグレードしてください]** リンクを選択します。 **[はじめに]** ページで **[アップグレード]** を選択します。

   > **注:** ご使用のサブスクリプションでは、Defender for Cloud のセキュリティ強化が既に有効になっている可能性があります。その場合は、アップグレードする必要はなく、次のタスクに進みます。

#### タスク 2: Defender for Cloud のセキュリティ強化を有効にする

1. **SEA-SVR2** で、Azure portal が表示されている Microsoft Edge ウィンドウの **[Microsoft Defender for Cloud | 概要]** ページで、左側にある垂直メニューの **[管理]** セクションにある **[環境設定]** を選択します。
1. **[環境設定]** ページで、Azure サブスクリプションを表すエントリを選択します。
1. **[設定 \| Defender プラン]** ページで、 **[すべてのプランを有効にする]** を選択します。 **[プランの選択]** ページで、**[Microsoft Defender for API プラン 1]** を選択し、**[保存]** を選択します。
1. **[設定 | Defender プラン]** ページで、**[保存]** を選択します。

   > **注: ** 自動プロビジョニングの更新エラーに関する通知が発生する場合があります。 このラボではサーバー プランのみを使用するため、これらの通知は無視しても問題ありません。 注: 同じページに記載されている個々の Microsoft Defender プランを選択的に無効にできます。

1. **[サーバー]** を除くすべてのプランを **[オフ]** に設定し、 **[保存]** を選択します。 ダウングレードするかどうかを確認するメッセージが表示されたら確認します。

   > **注: ** データベース プランのリソースの種類の選択が表示されたら、各エントリをオフに切り替え、[続行] を選択します。 このラボの目的上、個々のデータベース リソースの保存に失敗した場合に関する通知は、無視しても問題ありません。

1.   **[設定と監視]**  タブを選択し、拡張機能の一覧で **[ゲスト構成エージェント (プレビュー)]** を  **[オン]** に設定します。
1. **[設定と監視]** タブの拡張機能の一覧で、**[マシンの脆弱性評価]** を  **[オン]** に設定し、 **[構成の編集]**  リンクを選択します。
1. **[拡張機能のデプロイ構成]** ページで、**[Microsoft Defender 脆弱性の管理]** オプションが選択されていることを確認し、**[適用]** を選択します。
1. **[設定と監視]** ページで、 **[続行]** を選択します。   
1. **[Defender プラン]** ページで **[保存]** を選択してから、ページを閉じます。
1. **[Microsoft Defender for Cloud | 概要]** ページに戻り、左側の垂直メニューの **[管理]** セクションで、 **[環境設定]** を選択します。
1. **[環境設定]** ページで、**Azure サブスクリプション**を表すエントリを展開し、前の演習で作成した **Log Analytics ワークスペース**を表すエントリを選択します。
1. **[設定 \| Defender プラン]** ページで、 **[サーバー** Defender プラン] を有効にし、**[保存]** を選択します。

   > **注: ** 脅威保護機能を含むすべての Defender for Cloud 機能を有効にするには、該当するワークロードを含むサブスクリプションで、強化されたセキュリティ機能を有効にする必要があります。 ワークスペース レベルで有効にしても、Just-In-Time VM アクセス、適応型アプリケーション制御、Azure リソースのネットワーク検出は有効になりません。 さらに、ワークスペース レベルで使用できる Microsoft Defender プランは、Microsoft Defender for servers と Microsoft Defender for SQL servers on machines のみです。

1. **[設定 \| Defender プラン]** ページの左側にある垂直メニューの **[設定]** セクションで、 **[データ コレクション]** を選択します。
1. **[設定 \| データコレクション]** で、 **[すべてのイベント]** を選択し、 **[保存]** を選択します。

   > **注:** Defender for Cloud でデータ コレクションのレベルを選択することは、Log Analytics ワークスペースでのセキュリティ イベントのストレージにのみ影響します。 ワークスペースに保存するように選択したセキュリティ イベントのレベルに関係なく、Log Analytics エージェントでは、Defender for Cloud の脅威保護に必要なセキュリティ イベントが引き続き収集され、分析されます。 セキュリティ イベントを保存することを選択すると、ワークスペース内でそれらのイベントの調査、検索、監査が可能になります。

## 演習 3: Windows Server を実行する Azure VM のプロビジョニング

#### タスク 1: Azure Cloud Shell を起動する

1. **SEA-SVR2** の Azure portal で、[検索] テキスト ボックスの横にあるツールバー アイコンを直接選択して、Cloud Shell ペインを開きます。
1. **Bash** または **PowerShell** の選択を求めるメッセージが表示されたら、 **[PowerShell]** を選択します。

   >**注**: Cloud Shell を起動するのが初めてで、"**ストレージがマウントされていません**" というメッセージが表示される場合は、このラボで使用しているサブスクリプションを選択し、 **[ストレージの作成]** を選択します。

#### タスク 2: Azure Resource Manager テンプレートを使用して Azure VM をデプロイする

1. 次のスクリプトを実行して、新しいリソース グループを作成します (`<Azure_region>` プレースホルダーを、このラボでリソースをデプロイする予定の Azure リージョンの名前に置き換えます)。

   >**注**: **(Get-AzLocation).Location** コマンドを使用して、使用可能な Azure リージョンの名前を一覧表示できます。

   ```powershell 
   New-AzResourceGroup -Name 'AZ801-L0202-RG' -Location '<Azure_region>'
   ```

1. Cloud Shell を閉じます。
1. ツール バーの **[リソース、サービス、ドキュメントの検索] テキスト ボックス**で、**[カスタム テンプレートのデプロイ]** を検索して選択します。
1. 「**カスタム デプロイ**」ページで、**[エディターで独自のテンプレートを作成する]** を選択します。
1. **[テンプレートの編集]** ページで、**[ファイルの読み込み]** を選択し、テンプレート ファイル **L02-rg_template.json** をアップロードして、**[保存]** を選択します。
1. **[カスタム デプロイ]** ページで、次の設定を指定して、その他の設定は既定値のままにします。

   |設定|値|
   |---|---|
   |サブスクリプション|このラボで使用している Azure サブスクリプションの名前|
   |リソース グループ|**AZ801-L0202-RG**|
   |リージョン|Azure VM をプロビジョニングできる Azure リージョンの名前|
   |管理ユーザー名| 任意の管理者名
   |管理者パスワード|任意の強力なパスワードを選択する

1. **[確認と作成]** を選択し、次に **[作成]** を選択します。

   >**注**: デプロイには約 3 分かかります。

1. デプロイが正常に完了したことを確認します。


## 演習 4: オンプレミスの Windows Server を Microsoft Defender for Cloud および Azure Update Manager にオンボードする

#### タスク 1: オンプレミスのサーバーに Azure Arc をインストールする

1. **SEA-SVR2** で、Azure Portal が表示されている Microsoft Edge ウィンドウで、「**Arc**」と入力し、**[Azure Arc]** を選択します。
1.  **[Azure Arc リソース]** の下にあるナビゲーション ウィンドウで  **[マシン]** を選択します
1.  **[+ Add/Create] (+ 追加/作成)** を選択し、ドロップダウンで **[マシンの追加]** を選択します。 
1. **[単一サーバーの追加]** セクションから  **[スクリプトの生成]**  を選択します。 
1.   **[Azure Arc でサーバーを追加]]**  ページの  **[プロジェクトの詳細]** の下で、先ほど作成したリソース グループ (AZ801-L0201-RG) を選択します。 
1. **[サーバーの詳細]** の下で、リージョンとして **(米国) 東部**を選択します。 
1. SQL Server と接続のオプションを確認します。 既定値のままにして  **[次へ]** を選択します。 
1. **[タグ]** タブで、使用可能な既定のタグを確認し、 **[次へ]** を選択します。 
1. **[Azure Arc でサーバーを追加**] タブで、下にスクロールして **[ダウンロード]**  ボタンを選択します。

   >ヒント: ブラウザーがダウンロードをブロックしている場合は、Microsoft Edge ブラウザーで許可します。省略記号ボタン (...) を選択し、[保持する] を選択します。

1. **Windows の [スタート]** ボタンを右クリックし、 **[Windows PowerShell (管理者)]** を選択します。

   > UAC プロンプトが表示された場合は、*[ユーザー名]* に  **[管理者]** と入力し *[パスワード]* には「Passw0rd!」と入力します。

1. 「**cd C:\Users\Administrator\Downloads** ]を入力するか、スクリプトをダウンロードしたフォルダーに移動します。  
1. 「 `Set-ExecutionPolicy -ExecutionPolicy Unrestricted` 」と入力して **Enter** キーを押します。 
1. **[すべてにはい]** には「 **A** 」を入力し、**Enter** キーを押します。
1. 「 `.\OnboardingScript.ps1` 」と入力して **Enter** キーを押します。 
1. 「 **R** 」を入力し **1 回実行し**、**Enter **キーを押します (これには数分かかる場合があります)。

   >セットアップ プロセスにより、Azure Arc エージェントを認証するための新しい Microsoft Edge ブラウザー タブが開きます。 管理者アカウントを選択し、メッセージ認証が完了するまで待ちます。 Windows PowerShell に戻り、インストールが完了するまで待ってからウィンドウを閉じます。

1. スクリプトをダウンロードした [Azure portal] ページに戻り、 **[閉じる]** を選択します。
1.  **[Azure Arc でサーバーを追加]**  ページを閉じて、**[Azure Arc マシン]**  ページに戻ります。
1.  ARC コンソールで **SEA-SVR2** サーバー名が表示され、[状態] が  **[接続済み]** になるまで  **[更新]**  を選択します。 

#### タスク 2: ARC マシンで [Change Tracking とインベントリ] を有効にする

1. **Azure Arc- Machines- SEA-SVR2** の下にあるナビゲーション ウィンドウで、 **[操作]** の下にある **[インベントリ]** を選択します。   
1. **[Change Tracking とインベントリ]** ページで、**[有効にする]** をクリックします。

   >作成した Log Analytics ワークスペースが、**[AMA を使用した変更履歴とインベントリ機能を有効にする] **の下に一覧表示されることに注意してください。

1. Change Tracking 機能のデプロイが完了するまで待ちます。 これには最大 5 分かかる場合があるため、次の手順に進みます。


#### タスク 3: 分析情報を使って [監視] を有効にする

1. **SEA-SVR2** Azure Arc マシンに移動し、**[設定]** の下で **[拡張機能]** を選択します。  

   >ChangeTracking と Azure Monitor エージェント拡張機能の追加に注意してください。

1. **[監視]** の下で **[分析情報]** を選択し、**[有効にする]** を選択します。  
1. **[データ収集規則]** の下の **[監視構成]** ページで、**[新規作成]** を選択します。
1. **[新しいルールの作成]** ページで、**[データ収集規則]** の名前に「**ARC**」と入力します。
1. **[プロセスと依存関係]** の下で、**[プロセスと依存関係を有効にする (マップ)]** を選択します。
1. このラボで使用している Azure サブスクリプションの名前のままにします。
1. **[Log Analytics ワークスペース]** ドロップダウン リストで、前の手順で作成した [Log Analytics ワークスペース] を選択します。
1. **[作成]** を選択し、**[設定]** を選択します。

   >**注意**: デプロイには少し時間がかかることがあります。 他のタスクに進みます。後でこれに戻ることができます。

#### タスク 4: Windows 更新プログラムによる [監視] を有効にする 

   **SEA-SVR2** では、Windows Update は既定で無効になっています。 **SEA-SVR2** サーバーの Windows Update が無効にされていないことを確認します。 次の手順に進む前に、これを有効にする必要があります。

1. サービス コンソールを開き、**[Windows Update]** サービスを選択します。

1. **[Windows Update]** を右クリックし、コンテキスト メニューから **[プロパティ]** を選択します。 スタートアップの種類を **[自動]** に設定し、サービスを起動します。 
1. サービス コンソールを閉じます。
1. Azure portal の検索バーに「**Azure Update Manager**」と入力します。
1. ナビゲーション ウィンドウの **[リソース]** の下にある **[マシン]** を選択します **[マシン]** ページで一覧表示されているAzure Arc 対応サーバー **SEA-SVR2** が表示されます。 **[SEA-SVR2]** マシンを選択します。
1. **[推奨更新プログラムの**] タブの **[定期評価]** の下で、**[今すぐ有効にする]** をクリックします。
1. **SEA-SVR2** Arc 対応サーバーのドロップダウン メニューから、**[定期評価]** の下の **[有効にする]** を選択して、**[保存]** を選択します。 
1. **[更新プログラム]** ページで、**[更新プログラムの確認]** を選択します。 
1.  **[今すぐ評価をトリガー]** ウィンドウで、**[OK]** を選択します。

   > **注:** 不足している更新プログラムは数分後に表示されます。 Azure Arc マシンを定期的に見直すことができます。その直後に更新プログラムが反映されます。


#### タスク 5: Azure Policy のコンプライアンス、Change tracking、インベントリ、分析情報の監視、Azure 更新プログラムを確認する

1.  **SEA-SVR2** Azure Arc マシンに移動し、**[監視]** の下のナビゲーション ウィンドウで **[分析情報]** を選択します。

1.  **[データ分析]** を選択します。 パフォーマンス データを表示できるようになります。  
1.  **[マップ]** タブを選択し、依存関係マップを表示します。  

   >**注**: データが表示されない場合は、**[パフォーマンス]** タブに戻り、更新します。 次に、**[マップ]** セクションを更新します。 **SEA-SVR2** のデータとプロセスの依存関係情報が表示されます。

1.  ナビゲーション ウィンドウの **[操作]** の下で **[インベントリ]** を選択してインベントリ データを表示します。   
1.  **[Change Tracking]** を選択して、変更されたデータを表示します。  

## 演習 5: Azure 環境のプロビジョニング解除 

#### タスク 1: Cloud Shell で PowerShell セッションを開始する

1. **SEA-SVR2** の Azure portal を表示している Microsoft Edge ウィンドウで、Cloud Shell アイコンを選んで Cloud Shell ペインを開きます。

#### タスク 2: ラボでプロビジョニングしたすべての Azure リソースを特定する

1. Cloud Shell ペインで次のコマンドを実行して、このラボで作成されたすべてのリソース グループの一覧を表示します。

   ```powershell
   Get-AzResourceGroup -Name 'AZ801-L02*'
   ```

   > **注**: 出力に含まれているのが、このラボで作成したリソース グループのみであることを確認してください。 このグループは、このタスクで削除されます。

1. 次のコマンドを実行して、このラボで作成したすべてのリソース グループを削除します。

   ```powershell
   Get-AzResourceGroup -Name 'AZ801-L02*' | Remove-AzResourceGroup -Force -AsJob
   ```

   >**注**: このコマンドは非同期で実行されるため ( *-AsJob* パラメーターによって決定されます)、同じ PowerShell セッション内で直後に別の PowerShell コマンドを実行できますが、リソース グループが実際に削除されるまでに数分かかります。

---
lab:
  title: 'ラボ: ハイブリッド シナリオでのセキュリティ ソリューションの実装'
  type: Answer Key
  module: 'Module 2: Implementing Security Solutions in Hybrid Scenarios'
---

# <a name="lab-answer-key-implementing-security-solutions-in-hybrid-scenarios"></a>ラボ解答キー: ハイブリッド シナリオでのセキュリティ ソリューションの実装

                **メモ:** このラボをご自分のペースでクリックして進めることができる、 **[ラボの対話型シミュレーション](https://mslabs.cloudguides.com/guides/AZ-801%20Lab%20Simulation%20-%20Implementing%20security%20solutions%20in%20hybrid%20scenarios)** が用意されています。 対話型シミュレーションとホストされたラボの間に若干の違いがある場合がありますが、示されている主要な概念とアイデアは同じです。 

## <a name="exercise-1-creating-an-azure-log-analytics-workspace-and-an-azure-automation-account"></a>演習 1: Azure Log Analytics ワークスペースと Azure Automation アカウントの作成

#### <a name="task-1-create-an-azure-log-analytics-workspace"></a>タスク 1: Azure Log Analytics ワークスペースを作成する 

1. **SEA-SVR2** に接続し、必要であれば、パスワード **Pa55w.rd** を使用して **CONTOSO\\Administrator** としてサインインします。
1. **SEA-SVR2** で Microsoft Edge を起動し、 **[Azure portal](https://portal.azure.com)** に移動し、このラボで使用するサブスクリプションの所有者ロールを持つユーザー アカウントの資格情報を使用してサインインします。
1. **SEA-SVR2** で、Azure portal のツールバーの **[リソース、サービス、ドキュメントの検索]** テキスト ボックスで、 **[Log Analytics ワークスペース]** を検索して選択し、 **[Log Analytics ワークスペース]** ページで **[+ 作成]** を選択します。
1. **[Log Analytics ワークスペースの作成]** ページの **[基本]** タブで、次の設定を入力し、 **[確認と作成]** を選択してから、 **[作成]** を選択します。

   | 設定 | 値 |
   | --- | --- |
   | サブスクリプション | このラボで使用している Azure サブスクリプションの名前 |
   | リソース グループ | 新しいリソース グループの名前 **AZ801-L0201-RG** |
   | Log Analytics ワークスペース | 任意の一意の名前 |
   | リージョン | 最寄りのリージョンを選択します |

   >**注**: デプロイが完了するまで待ちます。 デプロイには約 1 分かかります。

#### <a name="task-2-create-and-configure-an-azure-automation-account"></a>タスク 2: Azure Automation アカウントを作成して構成する

1. **SEA-SVR2** で、Azure portal のツールバーの **[リソース、サービス、ドキュメントの検索]** テキスト ボックスで、 **[Automation アカウント]** を検索して選択し、 **[Automation アカウント]** ページで **[+ 作成]** を選択します。
1. **[Automation アカウントの作成]** ページで、次の設定を指定し、 **[確認 + 作成]** を選択します。 検証時に、 **[作成]** を選択します。

   | 設定 | 値 |
   | --- | --- |
   | サブスクリプション | このラボで使用している Azure サブスクリプションの名前 |
   | リソース グループ | **AZ801-L0201-RG** |
   | 名前 | 任意の一意の名前 |
   | リージョン | **[ワークスペース マッピングに関するドキュメント](https://docs.microsoft.com/en-us/azure/automation/how-to/region-mappings)** に基づいて決定された Azure リージョンの名前 |

   >**注**: **[ワークスペース マッピングのドキュメント](https://docs.microsoft.com/en-us/azure/automation/how-to/region-mappings)** に基づいて、Azure リージョンを指定してください。

   >**注**: デプロイが完了するまで待ちます。 デプロイには約 3 分かかります。

1. [デプロイ] ページで、 **[リソースに移動]** を選択します。
1. **[Automation アカウント]** ページの **[構成管理]** セクションで、 **[インベントリ]** を選択します。
1. **[インベントリ]** ページの **[Log Analytics ワークスペース]** ドロップダウン リストで、このタスクの前の手順で作成した Log Analytics ワークスペースを選択し、 **[有効にする]** を選択します。

   >**注**: 対応する Log Analytics ソリューションのインストールが完了するまで待ちます。 これには 3 分ほどかかる場合があります。 

   >**注**: これにより、**Change tracking** ソリューションも自動的にインストールされます。

1. **[Automation アカウント]** ページの **[Update Management]** セクションで、 **[更新管理]** を選択し、 **[有効にする]** を選択します。

   >**注**: インストールが完了するまで待ちます。 これには 5 分ほどかかる場合があります。

## <a name="exercise-2-configuring-microsoft-defender-for-cloud"></a>演習 2: Microsoft Defender for Cloud の構成

#### <a name="task-1-enable-defender-for-cloud-and-automatic-agent-installation"></a>タスク 1: Defender for Cloud とエージェントの自動インストールを有効にする

1. **SEA-SVR2** で、Azure portal のツールバーの **[リソース、サービス、ドキュメントの検索]** テキスト ボックスで、 **[Microsoft Defender for Cloud]** を検索して選択します。
1. **[Microsoft Defender for Cloud \| 作業の開始]** ページで、 **[アップグレード]** を選択し、 **[エージェントのインストール]** を選択します。

   > **注:** ご利用のサブスクリプションでは、Defender for Cloud のセキュリティ強化が既に有効になっている可能性があります。その場合は、次のタスクに進みます。

#### <a name="task-2-enable-enhanced-security-of-defender-for-cloud"></a>タスク 2: Defender for Cloud のセキュリティ強化を有効にする

1. **SEA-SVR2** で、Azure portal が表示されている Microsoft Edge ウィンドウの **[Microsoft Defender for Cloud | 概要]** ページで、左側にある垂直メニューの **[管理]** セクションにある **[環境設定]** を選択します。
1. **[環境設定]** ページで、Azure サブスクリプションを表すエントリを選択します。
1. **[設定 \| Defender プラン]** ページで、 **[Enable all Microsoft Defender for Cloud plans]\(クラウドプランにすべての Microsoft Defender を有効にする\)** タイルを選択します。

   > **注:** 同じページに記載されている個々の Microsoft Defender プランを選択的に無効にできることに注意してください。

1. **Microsoft Defender for Servers** を表す最初のプランを除き、すべてのプランを**オフ**に設定し、 **[保存]** を選択します。
1. **[設定 \| Defender プラン]** ページの左側にある垂直メニューの **[設定]** セクションで、 **[自動プロビジョニング]** を選択します。
1. **[設定 \| 自動プロビジョニング]** ページの拡張機能の一覧で、 **[Log Analytics agent for Azure VMs]\(Azure VM の Log Analytics エージェント\)** エントリの右側にある **[構成の編集]** リンクを選択します。
1. **[拡張機能のデプロイ構成]** ページの **[ワークスペースの構成]** セクションで、 **[Azure VM を別のワークスペースに接続する]** オプションを選択します。 ドロップダウン メニューで、前の演習で作成したワークスペースを表すエントリを選択し、 **[適用]** を選択します。
1. **[設定 \| 自動プロビジョニング]** ページに戻り、 **[Azure Arc マシンの Log Analytics エージェント (プレビュー)]** を**オン**に設定します。 これにより、 **[拡張機能のデプロイ構成]** ページが自動的に表示されます。 
1. **[拡張機能のデプロイ構成]** ページの **[Log Analytics ワークスペースの選択]** ドロップダウン リストで、前の演習で作成したワークスペースを表すエントリを選択し、 **[適用]** を選択します。
1. **[設定 \| 自動プロビジョニング]** ページに戻り、 **[マシンの脆弱性評価]** を**オン**に設定します。 **[拡張機能のデプロイ構成]** ページで、 **[Microsoft 脅威と脆弱性の管理]** オプションが選択されていることを確認し、 **[適用]** を選択します。
1. ページの最上部で **[保存]** を選択します。
1. **[Microsoft Defender for Cloud | 概要]** ページに戻り、左側の垂直メニューの **[管理]** セクションで、 **[環境設定]** を選択します。
1. **[環境設定]** ページで、Azure サブスクリプションを表すエントリを展開し、前の演習で作成した Log Analytics ワークスペースを表すエントリを選択します。
1. **[設定 \| Defender プラン]** ページで、 **[Enable all Microsoft Defender for Cloud plans]\(クラウドプランにすべての Microsoft Defender を有効にする\)** タイルを選択し、 **[保存]** を選択します。

   > **注:** 脅威保護機能を含むすべての Defender for Cloud 機能を有効にするには、該当するワークロードを含むサブスクリプションで、強化されたセキュリティ機能を有効にする必要があります。 ワークスペース レベルで有効にしても、Just-In-Time VM アクセス、適応型アプリケーション制御、Azure リソースのネットワーク検出は有効になりません。 さらに、ワークスペース レベルで使用できる Microsoft Defender プランは、Microsoft Defender for servers と Microsoft Defender for SQL servers on machines のみです。

1. **[設定 \| Defender プラン]** ページの左側にある垂直メニューの **[設定]** セクションで、 **[データ コレクション]** を選択します。
1. **[設定 \| データコレクション]** で、 **[すべてのイベント]** を選択し、 **[保存]** を選択します。

   > **注:** Defender for Cloud でデータ コレクションのレベルを選択することは、Log Analytics ワークスペースでのセキュリティ イベントのストレージにのみ影響します。 ワークスペースに保存するように選択したセキュリティ イベントのレベルに関係なく、Log Analytics エージェントでは、Defender for Cloud の脅威保護に必要なセキュリティ イベントが引き続き収集され、分析されます。 セキュリティ イベントを保存することを選択すると、ワークスペース内でそれらのイベントの調査、検索、監査が可能になります。

## <a name="exercise-3-provisioning-azure-vms-running-windows-server"></a>演習 3: Windows Server を実行する Azure VM のプロビジョニング

#### <a name="task-1-start-azure-cloud-shell"></a>タスク 1: Azure Cloud Shell を起動する

1. **SEA-SVR2** の Azure portal で、[検索] テキスト ボックスの横にあるツールバー アイコンを直接選択して、Cloud Shell ペインを開きます。
1. **Bash** または **PowerShell** の選択を求めるメッセージが表示されたら、 **[PowerShell]** を選択します。

   >**注**: Cloud Shell を起動するのが初めてで、"**ストレージがマウントされていません**" というメッセージが表示される場合は、このラボで使用しているサブスクリプションを選択し、 **[ストレージの作成]** を選択します。

#### <a name="task-2-deploy-an-azure-vm-by-using-an-azure-resource-manager-template"></a>タスク 2: Azure Resource Manager テンプレートを使用して Azure VM をデプロイする

1. [Cloud Shell] ペインのツールバーで、 **[ファイルのアップロード/ダウンロード]** アイコンをクリックし、表示されるボックスの一覧で **[アップロード]** をクリックし、ファイル **C:\\Labfiles\\Lab02\\L02-sub_template.json** を Cloud Shell のホーム ディレクトリにアップロードします。
1. 前の手順を 2 回繰り返して、**C:\\Labfiles\\Lab02\\L02-rg_template.json** ファイルと **C:\\Labfiles\\Lab02\\L02-rg_template.parameters.json** ファイルを Cloud Shell ホーム ディレクトリにアップロードします。
1. ラボ環境をホスティングするリソース グループを作成するために、Cloud Shell ペインの **PowerShell** セッションで次のコマンドを入力し、各コマンドの入力後には Enter キーを押します (`<Azure_region>` プレースホルダーを、このラボでリソースをデプロイする Azure リージョンの名前に置き換えます)。

   >**注**: **(Get-AzLocation).Location** コマンドを使用して、使用可能な Azure リージョンの名前を一覧表示できます。

   ```powershell 
   $location = '<Azure_region>'
   New-AzSubscriptionDeployment -Location $location -Name az801l2001deployment -TemplateFile ./L02-sub_template.json -rgLocation $location -rgName 'AZ801-L0202-RG'
   ```

1. 新しく作成したリソース グループに Azure 仮想マシンをデプロイするには、次のコマンドを入力し、Enter キーを押します。

   ```powershell 
   New-AzResourceGroupDeployment -Name az801l2002deployment -ResourceGroupName AZ801-L0202-RG -TemplateFile ./L02-rg_template.json -TemplateParameterFile ./L02-rg_template.parameters.json
   ```

   >**注**: デプロイが完了するまで待ちます。 これには 3 分ほどかかります。

1. Cloud Shell を閉じます。

## <a name="exercise-4-onboarding-on-premises-windows-server-into-microsoft-defender-for-cloud-and-azure-automation"></a>演習 4: オンプレミスの Windows Server を Microsoft Defender for Cloud および Azure Automation にオンボードする

#### <a name="task-1-perform-manual-installation-of-the-log-analytics-agent"></a>タスク 1: Log Analytics エージェントの手動インストールを実行する

1. **SEA-SVR2** の、Azure portal を表示する Microsoft Edge ウィンドウで、 **[Microsoft Defender for Cloud \| 概要]** ページに戻り、左側の垂直メニューの **[全般]** セクションで **[インベントリ]** を選択します。
1. **[Microsoft Defender for Cloud \| インベントリ]** ページで、 **[+ 非 Azure サーバーの追加]** を選択します。
1. **[Defender for Cloud へのサーバーのオンボード]** ページの、このラボで前にプロビジョニングした Log Analytics ワークスペースを表すエントリの横にある **[アップグレード]** を選択します。

   > **注:** アップグレードが正常に完了すると、 **[アップグレード]** ボタンのラベルが **[+ サーバーの追加]** に変わります。

1. **[+ サーバーの追加]** ボタンを選択します。 これにより、自動的に **[エージェント管理]** ページが表示されます。そこから Log Analytics エージェントのインストーラーをダウンロードし、エージェントのインストールを完了するために必要なワークスペース ID とキーを識別できます。
1. **[エージェント管理]** ページで、 **[ワークスペース ID]** と **[プライマリ キー]** の値を記録します。 これらはこのタスクの後半で必要になります。
1. **[エージェント管理]** ページで、 **[Windows エージェントのダウンロード (64 ビット)]** リンクを選択します。
1. ダウンロードが完了したら、 **[ファイルを開く]** を選択します。 これにより、**Microsoft Monitoring Agent のセットアップ** ウィザードが開始されます。
1. **Microsoft Monitoring Agent セットアップ** ウィザードの、 **[Microsoft Monitoring Agent セットアップ ウィザードへようこそ]** ページで、 **[次へ]** を選択します。
1. **Microsoft Monitoring Agent セットアップ** ウィザードの **[重要な通知]** ページで、 **[同意する]** を選択します。
1. **Microsoft Monitoring Agent セットアップ** ウィザードの **[インストール先フォルダー]** ページで、 **[次へ]** を選択します。
1. **Microsoft Monitoring Agent のセットアップ ウィザード**の **[エージェントのセットアップ オプション]** ページで、 **[Connect the agent to Azure Log Analytics (OMS)]\(エージェントを Azure Log Analytics (OMS) に接続する\)** チェックボックスをオンにし、 **[次へ]** を選択します。
1. **Microsoft Monitoring Agent セットアップ** ウィザードの **[Azure Log Analytics]** ページで、このタスクで前に記録した **[ワークスペース ID]** と **[ワークスペース キー]** の値を入力し、 **[次へ]** を選択します。
1. **Microsoft Monitoring Agent セットアップ** ウィザードの **[Microsoft Update]** ページで、 **[次へ]** を選択します。
1. **Microsoft Monitoring Agent セットアップ** ウィザードの **[インストールの準備完了]** ページで、 **[インストール]** を選択します。
1. インストールが完了したら、 **[Microsoft Monitoring Agent の構成が正常に完了しました。]** ページで **[完了]** を選択します。

#### <a name="task-2-perform-unattended-installation-of-the-log-analytics-agent"></a>タスク 2: Log Analytics エージェントの無人インストールを実行する

1. **SEA-SVR2** 上で **[スタート]** を選択し、 **[Windows PowerShell (管理者)]** を選択します。
1. **MMASetup-AMD64.exe** ファイルの内容を抽出するには、**Windows PowerShell** コンソールで次のコマンドを入力し、各コマンドを入力した後、Enter キーを押します。
    
   ```powershell
   New-Item -ItemType Directory -Path 'C:\Labfiles\L02\' -Force
   Copy-Item -Path $env:USERPROFILE\Downloads\MMASetup-amd64.exe -Destination 'C:\Labfiles\L02\' -Force
   Set-Location -Path C:\Labfiles\L02
   .\MMASetup-amd64.exe /c /t:C:\Labfiles\L02
   Remove-Item -Path .\MMASetup-amd64.exe
   ```

1. インストール ファイルをターゲット **SEA-SVR1** にコピーするには、**Windows PowerShell** コンソールで次のコマンドを入力し、各コマンドを入力した後、Enter キーを押します。
    
   ```powershell
   New-Item -ItemType Directory -Path '\\SEA-SVR1\c$\Labfiles\L02' -Force
   Copy-Item -Path 'C:\Labfiles\L02\*' -Destination '\\SEA-SVR1\c$\Labfiles\L02' -Recurse -Force
   ```

1. **SEA-SVR1** で Log Analytics エージェントのインストールを実行するために、**Windows PowerShell** コンソールで次のコマンドを入力し、Enter キーを押します (`<WorkspaceID>` と `<PrimaryKey>` の各プレースホルダーを、この演習の前のタスクで記録した**ワークスペース ID** と**ワークスペース キー**の値に置き換えます)。

   ```powershell
   Invoke-Command -ComputerName SEA-SVR1.contoso.com -ScriptBlock { Start-Process -FilePath C:\Labfiles\L02\setup.exe -ArgumentList '/qn NOAPM=1 ADD_OPINSIGHTS_WORKSPACE=1 OPINSIGHTS_WORKSPACE_AZURE_CLOUD_TYPE=0 OPINSIGHTS_WORKSPACE_ID="<WorkspaceID>" OPINSIGHTS_WORKSPACE_KEY="<PrimaryKey>" AcceptEndUserLicenseAgreement=1' -Wait }
   ```

   > **注**: インストールが完了するまで待ちます。 これには 1 分ほどかかります。

#### <a name="task-3-enable-azure-automation-solutions-for-azure-vms"></a>タスク 3: Azure VM 向けの Azure Automation ソリューションを有効にする

1. **SEA-SVR2** で、Azure portal が表示されている Microsoft Edge ウィンドウに切り替え、このラボで前にプロビジョニングした Azure Automation アカウントのページを参照します。 
1. **[Automation アカウント]** ページの **[構成管理]** セクションで、 **[インベントリ]** を選択します。
1. **[インベントリ]** ページのツール バーで、 **[+ Azure VM の追加]** を選択します。
1. **[Enable Inventory]\(インベントリの有効化\)** ページの VM の一覧で、**az801l02-vm0** エントリの横にあるチェック ボックスがオンになっていることを確認し、 **[有効にする]** を選択します。

   > **注:** この VM が **[有効にする準備ができました]** としてリストされるためには、Automation アカウント ソリューションに関連付けられている Log Analytics ワークスペースに接続されている必要があります。

1. **[Automation アカウント]** ページに戻り、 **[Update Management]** セクションで、 **[更新管理]** を選択します。
1. **[更新管理]** ページのツール バーで、 **[+ Azure VM の追加]** を選択します。
1. **[更新管理の有効化]** ページの VM の一覧で、**az801l02-vm0** エントリの横にあるチェック ボックスがオンになっていることを確認し、 **[有効にする]** を選択します。

   > **注:** インベントリおよび Change tracking ソリューションの場合と同様に、この VM が **[有効にする準備ができました]** としてリストされるためには、Automation アカウント ソリューションに関連付けられている Log Analytics ワークスペースに接続されている必要があります。

#### <a name="task-4-enable-azure-automation-solutions-for-on-premises-servers"></a>タスク 4: オンプレミス サーバー向けの Azure Automation ソリューションを有効にする

1. **SEA-SVR2** の、Azure portal が表示されている Microsoft Edge ウィンドウで、 **[Automation アカウント]** ページに戻り、 **[構成管理]** セクションで **[インベントリ]** を選択します。
1. **[インベントリ]** ページで、 **[クリックしてマシンを管理します]** リンクを選択します。
1. **[マシンの管理]** ページで、 **[使用可能なマシンと今後のマシンすべてで有効にします]** オプションを選択し、 **[有効にする]** を選択します。

   > **注:** このオプションは、Log Analytics エージェントがインストールされ、インベントリ、Change Tracking、Update Management ソリューションをホストしている Azure Automation アカウントに関連付けられている Azure Log Analytics ワークスペースに登録されているオンプレミス サーバーに適用されます。

1. **[Automation アカウント]** ページに戻り、 **[Update Management]** セクションで、 **[更新管理]** を選択します。
1. **[更新管理]** ページで、 **[クリックしてマシンを管理します]** リンクを選択します。
1. **[マシンの管理]** ページで、 **[使用可能なマシンと今後のマシンすべてで有効にします]** オプションを選択し、 **[有効にする]** を選択します。

   > **注:** インベントリおよび Change tracking ソリューションの場合と同様に、このオプションは、Log Analytics エージェントがインストールされ、インベントリ、Change Tracking、Update Management ソリューションをホストしている Azure Automation アカウントに関連付けられている Azure Log Analytics ワークスペースに登録されているオンプレミス サーバーに適用されます。

## <a name="exercise-5-verifying-the-hybrid-capabilities-of-microsoft-defender-for-cloud-and-azure-automation-solutions"></a>演習 5: Microsoft Defender for Cloud と Azure Automation ソリューションのハイブリッド機能を確認する

#### <a name="task-1-validate-threat-detection-capabilities-for-azure-vms"></a>タスク 1: Azure VM の脅威検出機能を検証する

1. **SEA-SVR2** で、Azure portal が表示されている Microsoft Edge ウィンドウに切り替えます。 
1. Azure portal のツール バーにある **[リソース、サービス、ドキュメントの検索]** テキスト ボックスで、**仮想マシン**を検索して選択します。
1. **[仮想マシン]** ページで、 **[az801l02-vm0]** を選択します。
1. **az801l02-vm0** ページの **[操作]** セクションで **[コマンドの実行]** を選択し、 **[RunPowerShellScript]** を選択します。
1. **[コマンド スクリプトの実行]** ページの **[PowerShell スクリプト]** セクションで次のコマンドを入力し、 **[実行]** を選択して脅威検出アラートをトリガーします。

   ```powershell
   New-Item -ItemType Directory -Path 'C:\Temp' -Force
   Start-Process -FilePath powershell.exe -ArgumentList '-nop -exec bypass -EncodedCommand "cABvAHcAZQByAHMAaABlAGwAbAAgAC0AYwBvAG0AbQBhAG4AZAAgACIAJgAgAHsAIABpAHcAcgAgAGgAdAB0AHAAcwA6AC8ALwBkAG8AdwBuAGwAbwBhAGQALgBzAHkAcwBpAG4AdABlAHIAbgBhAGwAcwAuAGMAbwBtAC8AZgBpAGwAZQBzAC8AUwB5AHMAbQBvAG4ALgB6AGkAcAAgAC0ATwB1AHQARgBpAGwAZQAgAGMAOgBcAHQAZQBtAHAAXABzAHYAYwBoAG8AcwB0AC4AZQB4AGUAIAB9ACIA"' -Wait
   ```

1. **SEA-SVR2** の Azure portal で、 **[Microsoft Defender for Cloud \| 概要]** ページを参照します。 
1. **[Microsoft Defender for Cloud \| 概要]** ページで、左側の垂直メニューの **[全般]** セクションの、 **[セキュリティ アラート]** を選択します。
1. **[Microsoft Defender for Cloud \| セキュリティ アラート]** ページで、**az801l02-vm0** での PowerShell の疑わしい使用を示す重大度の高いアラートに注目します。
1. セキュリティ アラートを選択し、 **[セキュリティ アラート]** ページで **[アクションの実行]** を選択し、実行可能なアクションを確認します。

   > **注:** 今後の攻撃の可能性を最小限に抑えるために、セキュリティに関する推奨事項の実装を検討してください。

#### <a name="task-2-validate-the-threat-detection-capabilities-for-on-premises-servers"></a>タスク 2: オンプレミス サーバーの脅威検出機能を検証する

1. **SEA-SVR2** 上で、**Windows PowerShell** コンソールに切り替えます。
1. 脅威検出アラートをトリガーするには、**Windows PowerShell** コンソールで次のコマンドを入力し、各コマンドを入力した後で、Enter キーを押します。
    
   ```powershell
   New-Item -ItemType Directory -Path 'C:\Temp' -Force
   powershell -nop -exec bypass -EncodedCommand "cABvAHcAZQByAHMAaABlAGwAbAAgAC0AYwBvAG0AbQBhAG4AZAAgACIAJgAgAHsAIABpAHcAcgAgAGgAdAB0AHAAcwA6AC8ALwBkAG8AdwBuAGwAbwBhAGQALgBzAHkAcwBpAG4AdABlAHIAbgBhAGwAcwAuAGMAbwBtAC8AZgBpAGwAZQBzAC8AUwB5AHMAbQBvAG4ALgB6AGkAcAAgAC0ATwB1AHQARgBpAGwAZQAgAGMAOgBcAHQAZQBtAHAAXABzAHYAYwBoAG8AcwB0AC4AZQB4AGUAIAB9ACIA"
   ```

1. **SEA-SVR2** で、Azure portal を表示している Microsoft Edge ウィンドウに切り替え、 **[Microsoft Defender for Cloud \| セキュリティ アラート]** ページに戻ります。
1. **[Microsoft Defender for Cloud \| セキュリティ アラート]** ページで、**SEA-SVR2** での PowerShell の疑わしい使用を示す重大度の高いアラートに注目します。
1. セキュリティ アラートを選択し、 **[セキュリティ アラート]** ページで **[アクションの実行]** を選択し、実行可能なアクションを確認します。

   > **注:** 今後の攻撃の可能性を最小限に抑えるために、セキュリティに関する推奨事項の実装を検討してください。

#### <a name="task-3-review-the-features-and-capabilities-that-apply-to-hybrid-scenarios"></a>タスク 3: ハイブリッド シナリオに適用される機能を確認する

1. **SEA-SVR2** の、Azure portal を表示する Microsoft Edge ウィンドウで、 **[Microsoft Defender for Cloud \| 概要]** ページに戻り、左側の垂直メニューの **[全般]** セクションで **[インベントリ]** を選択します。
1. **[インベントリ]** ページのリソースの一覧で、**az801l02-vm0** Azure VM、**SEA-SVR1.contoso.com** および **SEA-SVR2.contoso.com** オンプレミス サーバーを表すエントリを識別します。

   > **注:** Azure とオンプレミスの VM を表すエントリが **[インベントリ]** ページに表示されるまでに、数分かかる場合があります。

   > **注:** **az801l02-vm0** が **[監視エージェント]** 列に **[未インストール]** と報告されている場合は、**az801l02-vm0** のリンクを選択します。 **[リソースの正常性 (プレビュー)]** ページで、 **[推奨事項]** を確認し、 **[Log Analytics エージェントは仮想マシンにインストールする必要がある]** エントリを選択します。 **[Log Analytics エージェントは仮想マシンにインストールする必要がある]** ページで、 **[修正]** を選びます。 **[リソースの修正]** ページの **[ワークスペース ID]** ドロップダウン リストで、Defender for Cloud によって作成された既定のワークスペースを選択し、 **[1 個のリソースの修正]** を選択します。 

1. **[インベントリ]** ページで **[az801l02-vm0]** リンクを選択してから、 **[リソースの正常性 (プレビュー)]** ページで **[推奨事項]** を確認します。
1. **[インベントリ]** ページに戻り、**SEA-SVR2** を表すリンクを選択してから、 **[リソースの正常性 (プレビュー)]** ページで **[推奨事項]** を確認します。

#### <a name="task-4-validate-azure-automation-solutions"></a>タスク 4: Azure Automation のソリューションを検証する

1. **SEA-SVR2** の、Azure portal が表示されている Microsoft Edge ウィンドウで、このラボで前にプロビジョニングした Azure Automation アカウントのページに戻り、 **[構成管理]** セクションで **[インベントリ]** を選択します。
1. **[インベントリ]** ページで **[マシン]** タブを確認し、このラボの前の手順で Log Analytics ワークスペースに登録した Azure VM とオンプレミス サーバーの両方が含まれていることを確認します。

   >**注**: いずれかまたは両方の種類のシステムが **[マシン]** タブに表示されていない場合は、もうしばらく待つ必要がある場合があります。

1. **[インベントリ]** ページで、残りのタブ ( **[ソフトウェア]** 、 **[ファイル]** 、 **[Windows レジストリ]** 、 **[Windows サービス]** タブなど) を確認します。

   >**注**: 収集されたデータとファイルは、 **[インベントリ]** ページのツールバーの **[設定の編集]** オプションを使用して構成できます。

   >**注**: これにより、**Change tracking** ソリューションも自動的にインストールされます。

1. このラボで前にプロビジョニングした Azure Automation アカウントのページに戻り、 **[構成管理]** セクションで **[変更の追跡]** を選択します。 
1. **[イベント]** 、 **[ファイル]** 、 **[レジストリ]** 、 **[ソフトウェア]** 、 **[Windows サービス]** の各エントリに関連付けられている数を確認します。 これらの値が 0 より大きい場合は、対応する変更に関する詳細について、ページの下部にある **[変更]** および **[イベント]** タブで参照できます。

   >**注**: この場合も、追跡される変更は、 **[変更の追跡]** ページのツール バーの **[設定の編集]** オプションを使用して構成できます。

1. このラボで前にプロビジョニングした Azure Automation アカウントのページに戻り、 **[Update Management]** セクションで **[更新の管理]** を選択します。
1. **[更新管理]** ページで **[マシン]** タブを確認し、このラボの前の手順で Log Analytics ワークスペースに登録した Azure VM とオンプレミス サーバーの両方が含まれていることを確認します。
1. **[マシン]** タブで各エントリのコンプライアンス状態を確認し、 **[不足している更新プログラム]** タブを参照します。

   >**注**: オンプレミス サーバーと Azure VM の両方に対して、不足している更新プログラムの自動デプロイをスケジュールするオプションがあります。

## <a name="exercise-6-deprovisioning-the-azure-environment"></a>演習 6: Azure 環境のプロビジョニング解除

#### <a name="task-1-start-a-powershell-session-in-cloud-shell"></a>タスク 1: Cloud Shell で PowerShell セッションを開始する

1. **SEA-SVR2** の Azure portal を表示している Microsoft Edge ウィンドウで、Cloud Shell アイコンを選んで Cloud Shell ペインを開きます。

#### <a name="task-2-identify-all-azure-resources-provisioned-in-the-lab"></a>タスク 2: ラボでプロビジョニングしたすべての Azure リソースを特定する

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

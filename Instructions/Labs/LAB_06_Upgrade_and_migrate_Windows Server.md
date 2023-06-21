---
lab:
  title: 'ラボ: Windows Server でのアップグレードと移行'
  module: 'Module 6: Upgrade and migrate in Windows Server'
---

# ラボ: Windows Server でのアップグレードと移行

## ラボのシナリオ

Contoso は、オンプレミスの Windows Server から Azure 仮想マシン (VM) への移行が容易になる、インフラストラクチャ サービスのためのハイブリッド モデルを探しています。 このイニシアティブを支援するため、あなたは Azure VM に Active Directory Domain Services (AD DS) ドメイン コントローラーをデプロイするプロセスを評価する作業を任されました。 目的は、現在オンプレミスの展開に使用されている手動プロセスと、Azure で使用できるデプロイ方法の違いを明らかにすることです。 さらに、記憶域移行サービスの機能を使用してオンプレミス ファイル サーバーを移行する方法について、テストを実施し、文書化する必要があります。 

                **メモ:** このラボをご自分のペースでクリックして進めることができる、 **[ラボの対話型シミュレーション](https://mslabs.cloudguides.com/guides/AZ-801%20Lab%20Simulation%20-%20Upgrading%20and%20migrating%20in%20Windows%20Server)** が用意されています。 対話型シミュレーションとホストされたラボの間に若干の違いがある場合がありますが、示されている主要な概念とアイデアは同じです。 

## 目標

このラボでは、次のことを行います。

- Azure に AD DS ドメイン コントローラーをデプロイします。
- 記憶域移行サービスを使用してファイル サーバーを移行します。

## 予想所要時間: 60 分

## ラボ環境
  
仮想マシン: **AZ-801T00A-SEA-DC1**、**AZ-801T00A-SEA-SVR1**、**AZ-801T00A-SEA-SVR2** が実行されている必要があります。 他の VM が実行されていてもかまいませんが、このラボでは必要ありません。

> **注**: **AZ-801T00A-SEA-DC1**、**AZ-801T00A-SEA-SVR1**、**AZ-801T00A-SEA-SVR2** 仮想マシンは、それぞれ **SEA-DC1**、**SEA-SVR1**、および **SEA-SVR2** のインストールをホストしています。

1. **[SEA-SVR2]** を選択します。
1. 次の資格情報を使用してサインインします。

   - ユーザー名: **Administrator**
   - パスワード: **Pa55w.rd**
   - ドメイン: **CONTOSO**

このラボでは、使用可能な VM 環境と Azure サブスクリプションを使用します。 ラボを開始する前に、Azure サブスクリプションと、そのサブスクリプションの所有者ロールを持つユーザー アカウントがあることを確認してください。

## 演習 1: AD DS ドメイン コントローラーを Azure にデプロイする

> **注**: ハイブリッドのシナリオでは、一般に、オンプレミスの既存のドメインから Azure VM に追加のドメイン コントローラーをデプロイすることで、オンプレミスの AD DS 環境を Azure に拡張する必要があります。 ラボでこのようなタスクを実行するには、Azure 仮想ネットワークへのサイト間 VPN 接続を設定するか、Azure にラボ環境全体をプロビジョニングし、その一部でオンプレミス サイトをエミュレートする必要があります。 わかりやすくするため、この演習では、Azure VM のドメイン コントローラーを新しいフォレストとドメインにデプロイします。 焦点は、ドメイン コントローラーの構成とプロビジョニング プロセスの、Azure VM を使用する場合に固有の側面を明らかにすることです。

この演習の主なタスクは次のとおりです。

1. Azure Resource Manager (ARM) テンプレートを使用して、ドメイン コントローラーをデプロイします。
1. Azure Bastion をデプロイします。 
1. Azure portal を使用して Azure VM をデプロイします。
1. Azure VM でドメイン コントローラーを手動で昇格させます。
1. 演習でデプロイした Azure リソースを削除します。

#### タスク 1: Azure Resource Manager (ARM) テンプレートを使用してドメイン コントローラーをデプロイする

1. **SEA-SVR2** で Microsoft Edge を起動し、Azure portal (`https://portal.azure.com/`) に移動し、このラボで使用するサブスクリプションの所有者ロールを持つユーザー アカウントの資格情報を使用してサインインします。
1. **SEA-SVR2** で Microsoft Edge を起動し、「**[新しい Windows VM を作成し、新しい AD フォレスト、ドメイン、DC を作成する](https://github.com/az140mp/azure-quickstart-templates/tree/master/application-workloads/active-directory/active-directory-new-domain)**」にあるカスタマイズされたバージョンのクイックスタート テンプレートにアクセスします。 
1. 「**新しい Windows VM を作成し、新しい AD フォレスト、ドメイン、DC を作成する**」ページで、Azure へのデプロイを始めます。 
1. **[Create an Azure VM with a new AD Forest](新しい AD フォレストで Azure VM を作成する)** ページで、**[テンプレートの編集]** を選びます。
1. **[Edit template](テンプレートの編集)** ページで、**storageProfile** セクション (**195** 行目以降) を参照し、**sku** (**199** 行目) が**2022-Datacenter** に設定されていて、**dataDisks** **caching** (**213** 行目) が **None** に設定されていることを確認します。

   > **注**: AD DS のデータベース ファイルとログ ファイルがホストされているディスクでは、キャッシュを **None** に設定する必要があります。

1. **[Edit template](テンプレートの編集)** ページで、**extension** セクション (**233** 行目以降) を参照し、デプロイされた Azure 仮想マシン (VM) 内で **CreateADPDC.ps1** スクリプトを実行するために、テンプレートで PowerShell Desired State Configuration が使用されていることに注目します。

   > **注**: 次の手順のようにしてスクリプトを確認できます。

   1. **SEA-SVR2** の Microsoft Edge ウィンドウで別のタブを開き、「**[新しい Windows VM を作成し、新しい AD フォレスト、ドメイン、DC を作成する](https://github.com/az140mp/azure-quickstart-templates/tree/master/application-workloads/active-directory/active-directory-new-domain)**」にあるカスタマイズされたバージョンのクイックスタート テンプレートにアクセスします。
   1. 「**新しい Windows VM を作成し、新しい AD フォレスト、ドメイン、DC を作成する**」 ページのリポジトリの内容の一覧で **DSC** フォルダーを選択し、**CreateADPDC.ps1** ファイルを選択します。
   1. **azure-quickstart-templates/application-workloads/active-directory/active-directory-new-domain/DSC/CreateADPDC.ps1** ページで、スクリプトの内容を確認します。Active Directory Domain Services や DNS などの多数のサーバーの役割がインストールされ、NTDS のデータベースとログおよび SYSOVL 共有がドライブ **F** に配置されることに注目してください。 
   1. Microsoft Edge を閉じ、Azure portal の **[テンプレートの編集]** ページが表示されているタブに戻ります。

1. **[テンプレートの編集]** ページで、可用性セットをプロビジョニングするセクション (**110** 行目以降) を参照し、テンプレートによって可用性セットが作成され、VM がそれにデプロイされることに注目します (**181** 行目の **dependsOn** 要素で示されます)。

   > **注**: この演習の後半では、同じ可用性セットに別の Azure VM をデプロイし、同じドメイン内の追加のドメイン コントローラーとして構成します。 可用性セットを使用すると、回復性が向上します。

1. Azure VM のネットワーク インターフェイスをプロビジョニングするセクション (**110** 行目以降) を参照し、プライベート IP アドレスの割り当て方法が **Static** に設定されていることに注目します (**164** 行目)。

   > **注**: ドメイン コントローラーをデプロイするときは静的割り当てを使うのが一般的ですが、DNS サーバーの役割をホストするサーバーの場合は不可欠です。

1. 入れ子になったテンプレートをデプロイするセクション (**266** 行目以降) を参照し、DNS サーバーの役割がインストールされてドメイン コントローラーとして動作する Azure VM をホストする仮想ネットワーク内の DNS サーバーのアドレスが、テンプレートによって更新されることに注目します。

   > **注**: DNS サーバーの役割を使用してドメイン コントローラーを実行している Azure VM を指すカスタム DNS サーバー仮想ネットワークの設定を構成すると、それ以降に同じ仮想ネットワークにデプロイされるすべての Azure VM で、名前解決にその DNS サーバーが自動的に使用されるようになり、ドメイン参加機能が効果的に提供されます。

1. テンプレートに変更を適用せずに、**[テンプレートの編集]** ページを閉じます。
1. **[Create an Azure VM with a new AD Forest](新しい AD フォレストで Azure VM を作成する)** ページに戻り、**[パラメーターの編集]** を選びます。
1. **[パラメーターの編集]** ページで、**C:\\Labfiles\\Lab06\\L06-rg_template.parameters.json** ファイルをアップロードして、既定のパラメーターを置き換えます。
1. 次の設定でデプロイを始めます (その他は既定値のままにします)。

   | 設定 | 値 | 
   | --- | --- |
   | サブスクリプション | このラボで使用している Azure サブスクリプションの名前 |
   | リソース グループ | 新しいリソース グループの名前 **AZ801-L0601-RG** |
   | リージョン | Azure VM をプロビジョニングできる Azure リージョンの名前 |
   | 管理ユーザー名 | **学生** |
   | 管理パスワード | **Pa55w.rd1234** |
   | ドメイン名 | **contoso.com** |
   | VM サイズ | **Standard_DS2_v2** |
   | 仮想マシン名 | **az801l06a-dc1** |
   | 仮想ネットワーク名 | **az801l06a-vnet** |
   | Virtual Network のアドレス範囲 | **10.6.0.0/16** |
   | ネットワーク インターフェイス名 | **az801l06a-dc1-nic1** |
   | プライベート IP アドレス | **10.6.0.4** |
   | サブネット名 | **adSubnet** |
   | サブネット範囲 | **10.6.0.0/24** |
   | 可用性セット名 | **adAvailabilitySet** |

   > **注**: デプロイが完了するまで待ってから、次のタスクに進んでください。 これには 15 分ほどかかる場合があります。 

#### タスク 2: Azure Bastion をデプロイする 

> **注**: Azure Bastion を使用すると、この演習の前のタスクでデプロイしたパブリック エンドポイントを使用せずに Azure VM に接続できると同時に、オペレーティング システム レベルの資格情報を対象とするブルート フォース攻撃から保護することができます。

1. **SEA-SVR2** の Azure portal を表示している Microsoft Edge ウィンドウで、Cloud Shell で PowerShell セッションを開きます。
1. Cloud Shell ウィンドウの PowerShell セッションから次のコマンドを実行して、この演習で先ほど作成した仮想ネットワーク **az801l06a-vnet** に **AzureBastionSubnet** という名前のサブネットを追加します。

   ```powershell
   $resourceGroupName = 'AZ801-L0601-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az801l06a-vnet'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
    -Name 'AzureBastionSubnet' `
    -AddressPrefix 10.6.255.0/24 `
    -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. Azure portal で、次の設定を使用して Azure Bastion をデプロイします。

   | 設定 | 値 | 
   | --- | --- |
   | サブスクリプション | このラボで使用している Azure サブスクリプションの名前 |
   | リソース グループ | 新しいリソース グループの名前 **AZ801-L0602-RG** |
   | 名前 | **az801l06a-bastion** |
   | リージョン | この演習の前のタスクでリソースをデプロイしたのと同じ Azure リージョン |
   | レベル | **Basic** |
   | 仮想ネットワーク | **az801l06a-vnet** |
   | サブネット | **AzureBastionSubnet (10.6.255.0/24)** |
   | パブリック IP アドレス | **新規作成** |
   | パブリック IP の名前 | **az801l06a-vnet-ip** |

   > **注**: デプロイが完了するのを待たずに、次のタスクに進んでください。 デプロイには約 5 分かかります。

#### タスク 3: Azure portal を使用して Azure VM をデプロイする

> **注**: この演習の最初のタスクでプロビジョニングした最初の VM と同じドメインへの、2 つ目の Azure VM のデプロイと、追加ドメイン コントローラーとしてのそのセットアップは、完全に自動化できます。 一方、この場合にグラフィカル インターフェイスを使うと、オンプレミスでのドメイン コントローラーのプロビジョニングと Azure ベースのシナリオでの違いに関する追加のガイダンスが提供されます。

1. **SEA-SVR2** の Azure portal が表示されている Microsoft Edge ウィンドウで、次の設定を使用して仮想マシンを作成します (その他は既定値のままにします)。

   | 設定 | 値 |
   | --- | --- |
   | サブスクリプション | このラボで使用している Azure サブスクリプションの名前 |
   | リソース グループ | 既存のリソース グループ **AZ801-L0601-RG** を選択します |
   | 仮想マシン名 | **az801l06a-dc2** |
   | リージョン | この演習で前に最初の仮想マシンをデプロイしたのと同じ Azure リージョンを選択します |
   | 可用性のオプション | **可用性セット** |
   | 可用性セット | **adAvailabilitySet** |
   | Image | **Windows Server 2022 Datacenter: Azure Edition - Gen2** |
   | Azure Spot インスタンス | **No** |
   | サイズ | **Standard D2s v3** |
   | ユーザー名 | **Student** |
   | パスワード | **Pa55w.rd1234** |
   | パブリック受信ポート | **なし** |
   | 既存の Windows Server ライセンスを使用しますか? | **No** |
   | OS ディスクの種類 | **Standard SSD** |
   | データ ディスク名 | **az801l06a-dc2_DataDisk_0** |
   | データ ディスクのソースの種類 | **[なし (空のディスク)]** |
   | データ ディスク サイズ | **32 GiB** **Premium SSD** |
   | 仮想ネットワーク | **az801l06a-vnet** |
   | サブネット | **adSubnet (10.6.0.0/24)** |
   | パブリック IP | **なし** |
   | NIC ネットワーク セキュリティ グループ | **なし** |
   | Accelerated Networking | enabled |
   | この仮想マシンを既存の負荷分散ソリューションの後ろに配置しますか? | disabled |
   | ブート診断 | **マネージド ストレージ アカウントで有効にする (推奨)** |
   | パッチ オーケストレーション オプション | **手動更新** |  

   > **注**: デプロイが完了するまで待ちます。 デプロイには約 3 分かかります。

#### タスク 4: Azure VM でドメイン コントローラーを手動で昇格させる

1. **SEA-SVR2** の Azure portal が表示されている Microsoft Edge ウィンドウで、静的割り当てを使用するように、**az801l06a-dc2** 仮想マシンのネットワーク インターフェイスに割り当てられるプライベート IP アドレスを構成します。 

   > **注**: ドメイン コントローラーをデプロイするときは静的割り当てを使うのが一般的ですが、DNS サーバーの役割をホストするサーバーの場合は不可欠です。

   > **注**: Azure VM のネットワーク インターフェイスに静的 IP アドレスを割り当てると、そのオペレーティング システムの再起動がトリガーされます。

1. **SEA-SVR2** の Azure portal が表示されている Microsoft Edge ウィンドウで、**az801l06a-dc2** ページを参照します。
1. **az801l06a-dc2** ページで、Bastion サービスを介して **az801l06a-dc2** への RDP セッションを確立し、次の資格情報を使用して認証を行います。

   | 設定 | 値 | 
   | --- | --- |
   | [ユーザー名] |**Student** |
   | パスワード |**Pa55w.rd1234** |

1. **az801l06a-dc2** へのリモート デスクトップ セッション内で、Windows PowerShell セッションを開始します。
1. AD DS と DNS サーバーの役割をインストールするには、Windows PowerShell プロンプトで次のコマンドを実行します。
    
   ```powershell
   Install-WindowsFeature -Name AD-Domain-Services,DNS -IncludeManagementTools
   ```

   > **注**: インストールが完了するまで待ちます。 これには 3 分ほどかかる場合があります。

1. データ ディスクを構成するには、Windows PowerShell のプロンプトで次のコマンドを実行します。

   ```powershell
   Get-Disk | Where PartitionStyle -eq 'RAW' |  Initialize-Disk -PartitionStyle MBR
   New-Partition -DiskNumber 2 -UseMaximumSize -AssignDriveLetter
   Format-Volume -DriveLetter F -FileSystem NTFS
   ```

1. **az801l06a-dc2** へのリモート デスクトップ セッション内で、**サーバー マネージャー** ウィンドウに切り替えます。
1. **サーバー マネージャー**から **Active Directory Domain Services 構成ウィザード**を開始して、ドメイン コントローラーの昇格を実行します。
1. **Active Directory Domain Services 構成ウィザード**で、 **[既存のドメインにドメイン コントローラーを追加する]** オプションを選び、ターゲット ドメインとして **contoso.com** を指定します。
1. 昇格を実行するための資格情報として、ユーザー名 **Student@contoso.com** とパスワード **Pa55w.rd1234** を使用します。
1. 新しいドメイン コントローラーを書き込み可能として指定し、**ドメイン ネーム システム (DNS) サーバー**および**グローバル カタログ (GC)** コンポーネントを含めるオプションを指定します。
1. **ディレクトリ サービス復元モード (DSRM) のパスワード**を **Pa55w.rd1234** に設定します。
1. AD DS のデータベース、ログ ファイル、SYSVOL をホストするフォルダーがホストされているドライブを、ドライブ **C** からドライブ **F** に変更します。
1. **[前提条件の確認]** ページで、ネットワーク アダプターに静的 IP アドレスが設定されていないことを示す警告に注意し、昇格を開始します。 

   > **注**: 静的 IP アドレスはオペレーティング システム内ではなく、プラットフォーム レベルで割り当てられるため、この警告は予想されるものです。

   > **注**: 昇格プロセスを完了するため、オペレーティング システムが自動的に再起動されます。

1. **SEA-SVR2** で、Bastion サービスを介して **az801l06a-dc2** に再接続します。
1. **az801l06a-dc2** へのリモート デスクトップ セッションで、**サーバー マネージャー**を使用して、ローカル環境にインストールされている役割に **AD DS** と **DNS** が含まれることを確認します。

#### タスク 5: 演習でデプロイした Azure リソースを削除する

#### タスク 1: Cloud Shell で PowerShell セッションを開始する

1. **SEA-SVR2** の Azure portal を表示している Microsoft Edge ウィンドウで、Cloud Shell アイコンを選んで Cloud Shell ウィンドウを開きます。
1. Cloud Shell ウィンドウで次のコマンドを実行して、この演習で作成したすべてのリソース グループの一覧を表示します。

   ```powershell
   Get-AzResourceGroup -Name 'AZ801-L060*'
   ```

   > **注**: 出力に、この演習で作成したリソース グループのみが含まれていることを確認してください。 これらのグループは、このタスクで削除されます。

1. 次のコマンドを実行して、この演習で作成したすべてのリソース グループを削除します。

   ```powershell
   Get-AzResourceGroup -Name 'AZ801-L060*' | Remove-AzResourceGroup -Force -AsJob
   ```

   > **注:** このコマンドは非同期で実行されるため (*-AsJob* パラメーターによって決定されます)、同じ PowerShell セッション内で別の PowerShell コマンドをすぐに実行できますが、リソース グループが実際に削除されるまでに数分かかります。

## 演習 2: 記憶域移行サービスを使用してファイル サーバーを移行する

この演習の主なタスクは次のとおりです。

1. Windows Admin Center をインストールします。
1. ファイル サービスを設定します。
1. 記憶域移行サービスを使用して移行を実行します。
1. 移行の結果を検証します。

#### タスク 1: Windows Admin Center をインストールする

1. **SEA-ADM1** で、管理者として **Windows PowerShell** を起動します。

   >**注**: **SEA-ADM1** にまだ Windows Admin Center をインストールしていない場合は、次の 2 つの手順を行います。

1. **Windows PowerShell** コンソールで、次のコマンドを実行して、最新バージョンの Windows Admin Center をダウンロードします。
    
   ```powershell
   Start-BitsTransfer -Source https://aka.ms/WACDownload -Destination "$env:USERPROFILE\Downloads\WindowsAdminCenter.msi"
   ```
1. 次のコマンドを実行して、Windows Admin Center をインストールします。
    
   ```powershell
   Start-Process msiexec.exe -Wait -ArgumentList "/i $env:USERPROFILE\Downloads\WindowsAdminCenter.msi /qn /L*v log.txt REGISTRY_REDIRECT_PORT_80=1 SME_PORT=443 SSL_CERTIFICATE_OPTION=generate"
   ```

   > **注**: インストールが完了するまで待ちます。 これには 2 分ほどかかります。

#### タスク 2: ファイル サービスを設定する

1. **SEA-SVR2** で、ファイル **C:\\Labfiles\\Lab06\\L06_SetupFS.ps1** を Windows PowerShell ISE で開きます。
1. Windows PowerShell ISE スクリプト ペインで、スクリプトを確認して実行します。 

   >**注:** スクリプトが完了するのを待ちます。 これには 1 分ほどかかります。

   >**注:** このスクリプトでは、**SEA-SVR1** と **SEA-SVR2** で追加のデータ ディスクが初期化され、それぞれに NTFS ボリュームが作成され、各ボリュームにドライブ文字 **S:** が割り当てられ、**SEA-SVR1** の **S:\Data** フォルダーを使用して **Data** という名前の共有が作成されて、合計サイズ約 1 GB のサンプル ファイルが追加されます。 

#### タスク 3: 記憶域移行サービスを使用して移行を実行する

1. **SEA-ADM1** で Microsoft Edge を起動し、**https://SEA-ADM1.contoso.com** で Windows Admin Center のローカル インスタンスに接続します。 
1. メッセージが表示されたら、次の資格情報を使用して認証を行います。

   - ユーザー名: **CONTOSO\\Administrator**
   - パスワード: **Pa55w.rd**

1. **SEA-ADM1** の Windows Admin Center で、インストールされている拡張機能を確認し、一覧に**記憶域移行サービス**拡張機能が含まれることを確認します。

   >**注:** 利用可能な更新プログラムがある場合は、**記憶域移行サービス**拡張機能のエントリを選び、**[更新]** を選びます。

1. **[すべての接続]** ペインから、**sea-svr2.contoso.com** に接続します。
1. **sea-svr2.contoso.com** ページの **[ツール]** メニューから**記憶域移行サービス**を開始し、**[インストール]** 操作を呼び出します。

   >**注:** これにより、記憶域移行サービスとその必要なコンポーネントが自動的にインストールされます。

1. **[Migrate storage in three steps](3 つのステップで記憶域を移行する)** ペインを閉じます。
1. **[記憶域移行サービス]** ペインで、**SVR1toSVR2** という名前の移行ジョブを作成し、**[ソース デバイス]** を **[Windows servers and clusters](Windows サーバーとクラスター)** に設定します。
1. **[記憶域移行サービス > SVR1toSVR2]** ペインの **[Inventory servers](インベントリ サーバー)** タブで、**[Check the prerequisites](前提条件の確認)** ペインを確認します。
1. **[Inventory servers](インベントリ サーバー)** タブの **[資格情報の入力]** ペインで、必要に応じて、**CONTOSO\\Administrator** ユーザー アカウントの資格情報を入力し、**[Migrate from failover clusters](フェールオーバー クラスターから移行する)** チェック ボックスをオフにします。
1. **[Inventory servers](インベントリ サーバー)** タブの **[Add and scan devices](デバイスの追加とスキャン)** ペインで、次の資格情報を使用して **SEA-SVR1.contoso.com** サーバーを追加します。

   - ユーザー名: **CONTOSO\\Administrator**
   - パスワード: **Pa55w.rd**

   > **注**: シングル サインオンを実行するには、Kerberos の制約付き委任を設定する必要があります。

1. デバイスの一覧から、新しく追加した **SEA-SVR1.contoso.com** エントリを選び、そのスキャンを開始します。

   >**注:** スキャンが正常に完了するまで待ちます。 これには 1 分ほどかかります。

   >**注:** スキャンが完了したら、**[記憶域移行サービス > SVR1toSVR2]** ペインの **[Transfer data](データの転送)** タブからアクセスできる移行ジョブの 2 番目のステージに進みます。

1. **CONTOSO\\Administrator** ユーザー アカウントがデータ転送に使用されていることを確認します。
1. 転送先のデバイスを **SEA-SVR2.contoso.com** に設定します。

   >**注:** スキャンが正常に完了するまで待ちます。 これには 1 分ほどかかります。

   >**注: ** ハイブリッド シナリオでは、移行ジョブの移行先になる Azure VM を自動的に作成することもできます。

1. スキャンが完了したら、**[Specify the destination for: sea-svr1.contoso.com](転送先の指定先: sea-svr1.contoso.com)** ペインで、**[Map each source volume to a destination volume](各転送元ボリュームを転送先ボリュームにマップする)** セクションを調べて、**S:** 転送元ボリュームが **S:** 転送先ボリュームにマップされていることを確認します。
1. **[Specify the destination for: sea-svr1.contoso.com](転送先の指定先: sea-svr1.contoso.com)** ペインで、**[Select the shares to transfer](転送する共有の選択)** セクションを調べて、**[データ]** 転送元共有が転送に含まれていることを確認します。
1. **[Transfer data](データの転送)** タブの **[Adjust transfer settings](転送設定の調整)** ペインで、次の設定を指定します (他の設定は既定値のままにします)。

   | 設定 | 値 | 
   | --- | --- |
   | Back up folders that would be overwritten (Azure File Sync-enabled shares aren't backed up) (上書きされるフォルダーをバックアップする (Azure File Sync が有効な共有はバックアップしない)) | enabled |
   | Validation method (検証方法) | **CRC 64** |
   | Max duration (minutes) (最大継続時間 (分)) | **60** |
   | Migrate users and groups (ユーザーとグループを移行する) | **Reuse accounts with the same name (アカウントを同じ名前で再利用する)** |
   | Max retries (最大再試行回数) | **3** |
   | Delay between retries (seconds) (再試行の間隔 (秒)) | **60** |

   >**注:** **[Transfer data](データの転送)** タブの **[Install required features](必要な機能のインストール)** ペインで、**SEA-SVR2.contoso.com** への **SMS-Proxy** のインストールが完了するまで待ちます。

1. スキャンが完了したら、**[Transfer data](データの転送)** タブの **[Validate source and destination device](転送元と転送先のデバイスの検証)** ペインで検証を開始し、正常に完了するまで待ちます。
1. **[Transfer data](データの転送)** タブの **[Start the transfer](転送の開始)** ペインで、データの転送を開始します。

   >**注:** 転送が正常に完了するまで待ちます。 これにかかる時間は 1 分未満です。

   >**注:** これにより、**[記憶域移行サービス > SVR1toSVR2]** ペインの **[Cut over to the new servers](新しいサーバーへの切り替え)** タブからアクセスできる移行ジョブの 3 番目のステージに移ります。

1. **[Cut over to the new servers](新しいサーバーへの切り替え)** タブの **[Enter credentials for the source devices](移行元デバイスの資格情報の入力)** セクションと **[Enter credentials for the destination devices](移行先デバイスの資格情報の入力)** セクションで、**CONTOSO\\Administrator** ユーザー アカウントの保存されている資格情報を受け入れます。
1. **[Cut over to the new servers](新しいサーバーへの切り替え)** タブの **[Configure cutover from sea-svr1.contoso.com to sea-svr2.contoso.com](sea-svr1.contoso.com から sea-svr2.contoso.com への切り替えの構成)** ペインの **[Source network adapters](移行元ネットワーク アダプター)** セクションで、次の設定を指定します。

   | 設定 | 値 | 
   | --- | --- |
   | DHCP を使用する | disabled |
   | IP アドレス | **172.16.10.111** |
   | サブネット | **255.255.0.0** |
   | Gateway | **172.16.10.1** |

1. **[Cut over to the new servers](新しいサーバーへの切り替え)** タブの **[Configure cutover from sea-svr1.contoso.com to sea-svr2.contoso.com](sea-svr1.contoso.com から sea-svr2.contoso.com への切り替えの構成)** ペインの **[Destination network adapters](移行先ネットワーク アダプター)** ドロップダウン リストで、**Seattle** を選びます。
1. **[Cut over to the new servers](新しいサーバーへの切り替え)** タブの **[Configure cutover from sea-svr1.contoso.com to sea-svr2.contoso.com](sea-svr1.contoso.com から sea-svr2.contoso.com への切り替えの構成)** ペインの **[Rename the source device after cutover](切り替え後に移行元デバイスの名前を変更する)** セクションで、**[Choose a new name](新しい名前の選択)** オプションを選び、**[New source computer name](新しい移行元コンピューター名)** ダイアログ ボックスで「**SEA-SVR1-OLD**」と入力します。
1. **[Cut over to the new servers](新しいサーバーへの切り替え)** タブの **[Adjust cutover settings](切り替え設定の調整)** ペインの **[Cutover timeout (minutes)](切り替えタイムアウト (分))** テキスト ボックスに「**30**」と入力し、**[Enter AD credentials](AD 資格情報の入力)** セクションの **[Stored credentials](保存された資格情報)** オプションは有効のままにします。
1. **[Cut over to the new servers](新しいサーバーへの切り替え)** タブの **[Validate source and destination devices](移行元と移行先のデバイスの検証)** ペインで、検証を開始します。
1. 検証が完了して問題がなければ、 **[Cut over to the new servers](新しいサーバーへの切り替え)** タブで切り替えステージを開始します。

   >**注:** 切り替えによって、**SEA-SVR1** と **SEA-SVR2** 両方 の 2 回連続した再起動がトリガーされます。

#### タスク 4: 移行の結果を検証する

1. **SEA-ADM1** で、管理者として **Windows PowerShell** を起動します。
1. **SEA-SVR2** のネットワーク インターフェイスに割り当てられた IPv4 アドレスを確認するには、**Windows PowerShell** コンソールで次のコマンドを実行します。
    
   ```powershell
   Get-NetIPAddress | Where-Object AddressFamily -eq 'IPv4' | Select-Object IPAddress
   ```

   >**注:** 出力に **172.16.10.11** と **172.16.10.12** の両方が含まれていることを確認します。

1. **SEA-SVR2** に割り当てられている NetBIOS 名を確認するには、**Windows PowerShell** コンソールで次のコマンドを実行します。
    
   ```powershell
   nbtstat -n
   ```

   >**注:** 出力に **SEA-SVR1** と **SEA-SVR2** の両方が含まれていることを確認します。

1. **SEA-SVR2** のローカル共有を確認するには、**Windows PowerShell** コンソールで次のコマンドを実行します。
    
   ```powershell
   Get-SMBShare
   ```

   >**注:** 出力に **S:\Data** フォルダーでホストされている **Data** 共有が含まれることを確認します。

1. **SEA-SVR2** の **Data** 共有の内容を確認するには、**Windows PowerShell** コンソールで次のコマンドを実行します。
    
   ```powershell
   Get-ChildItem -Path 'S:\Data'
   ```

#### 確認

このラボでは、次のことを行いました。

- Azure に AD DS ドメイン コントローラーをデプロイしました。
- 記憶域移行サービスを使用してファイル サーバーを移行しました。

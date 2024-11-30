---
lab:
  title: 'ラボ: Windows Server でのアップグレードと移行'
  type: Answer Key
  module: Module 6 - Upgrade and migrate in Windows Server
---

# ラボ回答キー: Windows Server でのアップグレードと移行

**メモ:** このラボをご自分のペースでクリックして進めることができる、 **[ラボの対話型シミュレーション](https://mslabs.cloudguides.com/guides/AZ-801%20Lab%20Simulation%20-%20Upgrading%20and%20migrating%20in%20Windows%20Server)** が用意されています。 対話型シミュレーションとホストされたラボの間に若干の違いがある場合がありますが、示されている主要な概念とアイデアは同じです。 

## 演習 1: AD DS ドメイン コントローラーを Azure にデプロイする

#### タスク 1: Azure Resource Manager (ARM) テンプレートを使用してドメイン コントローラーをデプロイする

1. **SEA-SVR2** に接続し、必要であれば、パスワード **Pa55w.rd** を使用して **CONTOSO\\Administrator** としてサインインします。
1. **SEA-SVR2** で Microsoft Edge を起動し、「**[新しい Windows VM を作成し、新しい AD フォレスト、ドメイン、DC を作成する](https://github.com/az140mp/azure-quickstart-templates/tree/master/application-workloads/active-directory/active-directory-new-domain)**」にあるカスタマイズされたバージョンのクイックスタート テンプレートにアクセスします。 
1. 「**新しい Windows VM を作成し、新しい AD フォレスト、ドメイン、DC を作成する**」ページで、**[Azure に配置する]** を選択します。 これにより、ブラウザーが Azure portal の **[Create an Azure VM with a new AD Forest](新しい AD フォレストで Azure VM を作成する)** ページに自動的にリダイレクトされます。
1. **[Create an Azure VM with a new AD Forest](新しい AD フォレストで Azure VM を作成する)** ページで、**[テンプレートの編集]** を選びます。
1. **[テンプレートの編集]** ページで、**storageProfile** セクション (**195** 行目以降) を参照し、**sku** (**199** 行目) が **2022-Datacenter** に設定され、**version** (**200** 行目) が **latest** に設定され、**dataDisks** **caching** (**213** 行目) が **None** に設定されていることを確認します。

   > **注**: AD DS のデータベース ファイルとログ ファイルがホストされているディスクでは、キャッシュを **None** に設定する必要があります。

1. **[Edit template](テンプレートの編集)** ページで、**extension** セクション (**233** 行目以降) を参照し、デプロイされた Azure 仮想マシン (VM) 内で **CreateADPDC.ps1** スクリプトを実行するために、テンプレートで PowerShell Desired State Configuration が使用されていることに注目します。

   > **注**: 次の手順のようにしてスクリプトを確認できます。

   1. **SEA-SVR2** の Microsoft Edge ウィンドウで別のタブを開き、「**[新しい Windows VM を作成し、新しい AD フォレスト、ドメイン、DC を作成する](https://github.com/az140mp/azure-quickstart-templates/tree/master/application-workloads/active-directory/active-directory-new-domain)**」にあるカスタマイズされたバージョンのクイックスタート テンプレートにもう一度アクセスします。
   1. 「**新しい Windows VM を作成し、新しい AD フォレスト、ドメイン、DC を作成する**」 ページのリポジトリの内容の一覧で **DSC** フォルダーを選択し、**CreateADPDC.ps1** ファイルを選択します。
   1. **azure-quickstart-templates/application-workloads/active-directory/active-directory-new-domain/DSC/CreateADPDC.ps1** ページで、スクリプトの内容を確認します。Active Directory Domain Services や DNS などの多数のサーバーの役割がインストールされ、NTDS のデータベースとログおよび SYSOVL 共有がドライブ **F** に配置されることに注目してください。 
   1. Microsoft Edge を閉じ、Azure portal の **[テンプレートの編集]** ページが表示されているタブに戻ります。

1. **[テンプレートの編集]** ページで、可用性セットをプロビジョニングするセクション (**110** 行目以降) を参照し、テンプレートによって可用性セットが作成され、VM がそれにデプロイされることに注目します (**181** 行目の **dependsOn** 要素で示されます)。

   > **注**: この演習の後半では、同じ可用性セットに別の Azure VM をデプロイし、同じドメイン内の追加のドメイン コントローラーとして構成します。 可用性セットを使用すると、回復性が向上します。

1. Azure VM のネットワーク インターフェイスをプロビジョニングするセクション (**110** 行目以降) を参照し、プライベート IP アドレスの割り当て方法が **Static** に設定されていることに注目します (**164** 行目)。

   > **注**: ドメイン コントローラーをデプロイするときは静的割り当てを使うのが一般的ですが、DNS サーバーの役割をホストするサーバーの場合は不可欠です。

1. 入れ子になったテンプレートをデプロイするセクション (**266** 行目以降) を参照し、DNS サーバーの役割がインストールされてドメイン コントローラーとして動作する Azure VM をホストする仮想ネットワーク内の DNS サーバーのアドレスが、テンプレートによって更新されることに注目します。

   > **注**: DNS サーバーの役割を使用してドメイン コントローラーを実行している Azure VM を指すカスタム DNS サーバー仮想ネットワークの設定を構成すると、それ以降に同じ仮想ネットワークにデプロイされるすべての Azure VM で、名前解決にその DNS サーバーが自動的に使用されるようになり、ドメイン参加機能が効果的に提供されます。

1. **[テンプレートの編集]** ページで、**[破棄]** を選択します。
1. **[Create an Azure VM with a new AD Forest](新しい AD フォレストで Azure VM を作成する)** ページに戻り、**[パラメーターの編集]** を選びます。
1. **[パラメーターの編集]** ページで **[ファイルの読み込み]** を選択し、**[ファイルのアップロード]** ダイアログ ボックスで **C:\\Labfiles\\Lab06** フォルダーを参照し、**L06-rg_template.parameters.json** ファイルを選択して、**[開く]** を選択します。
1. **[テンプレートの編集]** ページで、 **[保存]** を選択します。
1. **[Create an Azure VM with a new AD Forest](新しい AD フォレストで Azure VM を作成する)** ページに戻り、**[リソース グループ]** ドロップダウン リストの下にある **[新規作成]** を選択し、**[名前]** テキスト ボックスに「**AZ801-L0601-RG**」と入力し、**[OK]** を選択します。
1. **[Create an Azure VM with a new AD Forest](新しい AD フォレストで Azure VM を作成する)** ページで、必要に応じてデプロイ設定を調整して、次の値を設定します (他の設定は既定値のまま)。

   | 設定 | 値 | 
   | --- | --- |
   | サブスクリプション | このラボで使用している Azure サブスクリプションの名前 |
   | リソース グループ | 新しいリソース グループの名前 **AZ801-L0601-RG** |
   | リージョン | Azure VM をプロビジョニングできる Azure リージョンの名前 |
   | 管理ユーザー名 | **Student** |
   | 管理者パスワード | **Pa55w.rd1234** |
   | ドメイン名 | **contoso.com** |
   | VM サイズ | **Standard_DS2_v2** |
   | _artifacts の場所 | **`https://raw.githubusercontent.com/az140mp/azure-quickstart-templates/master/application-workloads/active-directory/active-directory-new-domain/`** |
   | 仮想マシン名 | **az801l06a-dc1** |
   | 仮想ネットワーク名 | **az801l06a-vnet** |
   | Virtual Network のアドレス範囲 | **10.6.0.0/16** |
   | ネットワーク インターフェイス名 | **az801l06a-dc1-nic1** |
   | プライベート IP アドレス | **10.6.0.4** |
   | サブネット名 | **adSubnet** |
   | サブネット範囲 | **10.6.0.0/24** |
   | 可用性セット名 | **adAvailabilitySet** |

1. **[Create an Azure VM with a new AD Forest](新しい AD フォレストで Azure VM を作成する)** ページで、**[確認と作成]** を選択し、**[作成]** を選択します。

   > **注**: デプロイが完了するまで待ってから、次のタスクに進んでください。 これには 15 分ほどかかる場合があります。 

#### タスク 2: Azure Bastion をデプロイする 

> **注**: Azure Bastion を使用すると、この演習の前のタスクでデプロイしたパブリック エンドポイントを使用せずに Azure VM に接続できると同時に、オペレーティング システム レベルの資格情報を対象とするブルート フォース攻撃から保護することができます。

1. **SEA-SVR2** の Azure portal を表示している Microsoft Edge ウィンドウで、Azure portal の [Cloud Shell] をクリックして [Azure Cloud Shell] ペインを開きます。
1. **Bash** または **PowerShell** の選択を求めるメッセージが表示されたら、 **[PowerShell]** を選択します。

   > **注**: Cloud Shell を起動するのが初めてで、"**ストレージがマウントされていません**" というメッセージが表示される場合は、このラボで使用しているサブスクリプションを選択してから、**[ストレージの作成]** を選択します。

1. Cloud Shell ペインの PowerShell セッションから次のコマンドを実行して、この演習で先ほど作成した仮想ネットワーク **az801l06a-vnet** に **AzureBastionSubnet** という名前のサブネットを追加します。

   ```powershell
   $resourceGroupName = 'AZ801-L0601-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az801l06a-vnet'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
    -Name 'AzureBastionSubnet' `
    -AddressPrefix 10.6.255.0/24 `
    -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. [Cloud Shell] ペインを閉じます。
1. Azure portal のツールバーの **[リソース、サービス、ドキュメントの検索]** テキスト ボックスで、**[Bastions]** を検索して選択し、**[Bastions]** ページで **[+ 作成]** を選択します。
1. **[Bastion の作成]** ページの **[基本]** タブで、次の設定を指定し、**[確認と作成]** を選択します。

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

1. **[Bastion の作成]** ページの **[確認と作成]** タブで、**[作成]** を選択します。

   > **注**: デプロイが完了するのを待たずに、次のタスクに進んでください。 デプロイには約 5 分かかります。

#### タスク 3: Azure portal を使用して Azure VM をデプロイする

> **注**: この演習の最初のタスクでプロビジョニングした最初の VM と同じドメインへの、2 つ目の Azure VM のデプロイと、追加ドメイン コントローラーとしてのそのセットアップは、完全に自動化できます。 一方、この場合にグラフィカル インターフェイスを使うと、オンプレミスでのドメイン コントローラーのプロビジョニングと Azure ベースのシナリオでの違いに関する追加のガイダンスが提供されます。

1. **SEA-SVR2** の Azure portal を表示している Microsoft Edge ウィンドウで、ツール バーにある **[リソース、サービス、ドキュメントの検索]** テキスト ボックスで、「**仮想マシン**」を検索して選択します。 
1. **[仮想マシン]** ページで、 **[+ 作成]** を選択した後、ドロップダウン メニューの **[Azure 仮想マシン]** を選択します。
1. **[仮想マシンの作成]** ブレードの **[基本]** タブで、以下の設定を指定します (他の設定は既定値のまま)。

   | 設定 | 値 |
   | --- | --- |
   | サブスクリプション | このラボで使用している Azure サブスクリプションの名前 |
   | リソース グループ | 既存のリソース グループ **AZ801-L0601-RG** を選択します |
   | 仮想マシン名 | **az801l06a-dc2** |
   | リージョン | この演習で前に最初の仮想マシンをデプロイしたのと同じ Azure リージョンを選択します |
   | 可用性のオプション | **可用性セット** |
   | 可用性セット | **adAvailabilitySet** |
   | Image | **Windows Server 2022 Datacenter: Azure Edition - Gen2** |
   | Azure Spot 割引で実行する | **No** |
   | サイズ | **Standard D2s v3** |
   | ユーザー名 | **Student** |
   | パスワード | **Pa55w.rd1234** |
   | パブリック受信ポート | **なし** |
   | 既存の Windows Server ライセンスを使用しますか? | **No** |

1. **[次へ: ディスク >]** を選択し、**[仮想マシンの作成]** ブレードの **[ディスク]** タブで、以下の設定を構成します (他の設定は既定値のまま)。

   | 設定 | 値 |
   | --- | --- |
   | OS ディスクの種類 | **Standard SSD** |

1. **[仮想マシンの作成]** ブレードの **[ディスク]** タブの **[データ ディスク]** セクションで、**[新しいディスクを作成し接続する]** を選択します。
1. **[新しいディスクを作成する]** ページで、次の設定を指定し (他の設定は既定値のまま)、**[OK]** を選択します。

   | 設定 | 値 |
   | --- | --- |
   | 名前 | **az801l06a-dc2_DataDisk_0** |
   | 送信元の種類 | **[なし (空のディスク)]** |
   | サイズ | **32 GiB** **Premium SSD** |

1. **[仮想マシンの作成]** ブレードの **[ディスク]** タブに戻り、**[次へ: ネットワーク]** を選択し、**[仮想マシンの作成]** ブレードの **[ネットワーク]** タブで、以下の設定を構成します (他の設定は既定値のまま)。

   | 設定 | 値 |
   | --- | --- |
   | 仮想ネットワーク | **az801l06a-vnet** |
   | サブネット | **adSubnet (10.6.0.0/24)** |
   | パブリック IP | **なし** |
   | NIC ネットワーク セキュリティ グループ | **なし** |
   | 高速ネットワークを有効にする | disabled |
   | この仮想マシンを既存の負荷分散ソリューションの後ろに配置しますか? | disabled |

1. **[次へ: 管理 >]** を選択し、**[仮想マシンの作成]** ブレードの **[管理]** タブで、以下の設定を指定します (他の設定は既定値のまま)。

   | 設定 | 値 |
   | --- | --- |
   | パッチ オーケストレーション オプション | **手動更新** |

1. **[次へ: 監視 >]** を選択した後、 **[仮想マシンの作成]** ブレードの **[監視]** タブで、次の設定を指定します (他は既定値のままにします)。

   | 設定 | 値 |
   | --- | --- |
   | ブート診断 | **マネージド ストレージ アカウントで有効にする (推奨)** |

1. **[次へ: 詳細設定 >]** を選択し、**[仮想マシンの作成]** ブレードの **[詳細設定]** タブで、使用可能な設定を変更せずに確認し、**[確認と作成]** を選択します。
1. **[確認および作成]** ブレードで、**[作成]** を選択します。

   > **注**: デプロイが完了するまで待ちます。 デプロイには約 3 分かかります。

#### タスク 4: Azure VM でドメイン コントローラーを手動で昇格させる

1. **SEA-SVR2** で、Azure portal を表示している Microsoft Edge ウィンドウ内のデプロイ ページで **[リソースに移動]** を選択します。
1. **[az801l06a-dc2]** ページの左側の垂直メニューの **[設定]** セクションで、 **[ネットワーク]** を選択します。
1. **[az801l06a-dc2 \| ネットワーク]** ページで、**az801l06a-dc2** 仮想マシンのネットワーク インターフェイスへのリンクを選択します。
1. ネットワーク インターフェイス ページの左側の垂直メニューの **[設定]** セクションで、**[IP 構成]** を選択します。
1. **[IP 構成]** ページで、**[ipconfig1]** エントリを選択します。
1. **[ipconfig1]** ページの **[プライベート IP アドレスの設定]** セクションで、**[静的]** を選択し、**[保存]** を選択します。

   > **注**: ドメイン コントローラーをデプロイするときは静的割り当てを使うのが一般的ですが、DNS サーバーの役割をホストするサーバーの場合は不可欠です。

   > **注**: Azure VM のネットワーク インターフェイスに静的 IP アドレスを割り当てると、そのオペレーティング システムの再起動がトリガーされます。

1. **SEA-SVR2** の Azure portal が表示されている Microsoft Edge ウィンドウで、**az801l06a-dc2** ページに戻ります。
1. **[az801l06a-dc2]** ページで、**[接続]** を選択し、ドロップダウン メニューで **[Bastion]** を選択します。 
1. Bastion ページで、次の資格情報を入力し、 **[接続]** を選択します。

   | 設定 | 値 | 
   | --- | --- |
   | [ユーザー名] |**Student** |
   | パスワード |**Pa55w.rd1234** |

> **注**: 既定では、**Edge** によってポップアップがブロックされます。 **Bastion** のポップアップを許可するには、**Edge** の **[設定]** に移動し、左側の **[Cookie とサイトのアクセス許可]** を選択し、 **[すべてのアクセス許可]** の下の **[ポップアップとリダイレクト]** を選び、最後に **[ブロック (推奨)]** をオフに切り替えます。

1. **az801l06a-dc2** へのリモート デスクトップ セッション内で、**[スタート]** を選択し、**[Windows PowerShell]** を選択します。
1. AD DS と DNS サーバーの役割をインストールするには、Windows PowerShell コマンド プロンプトで次のコマンドを入力してから Enter キーを押します。
    
   ```powershell
   Install-WindowsFeature -Name AD-Domain-Services,DNS -IncludeManagementTools
   ```

   > **注**: インストールが完了するまで待ちます。 これには 3 分ほどかかる場合があります。

1. データ ディスクを構成するには、Windows PowerShell プロンプトで次のコマンドを入力し、各コマンドを入力した後、Enter キーを押します。

   ```powershell
   Get-Disk | Where PartitionStyle -eq 'RAW' |  Initialize-Disk -PartitionStyle MBR
   New-Partition -DiskNumber 2 -UseMaximumSize -AssignDriveLetter
   Format-Volume -DriveLetter F -FileSystem NTFS
   ```

1. **az801l06a-dc2** へのリモート デスクトップ セッション内で、**サーバー マネージャー** ウィンドウに切り替えます。
1. **サーバーマネージャー**で、**[通知]** フラグ シンボルを選択し、**[展開後構成]** セクションで **[このサーバーをドメイン コントローラーに昇格する]** リンクを選択します。 これにより、**Active Directory Domain Services 構成ウィザード**が開始されます。
1. **Active Directory Domain Services 構成ウィザード**の **[デプロイ構成]** ページの **[配置操作を選択してください]** の下で、**[既存のドメインにドメイン コントローラーを追加する]** が選択されていることを確認します。
1. **[ドメイン]** テキスト ボックスに、「**contoso.com**」ドメインを入力します。
1. **[この操作を実行するには資格情報を指定してください]** セクションで **[変更]** を選択します。
1. **[配置操作の資格情報]** ダイアログ ボックスの **[ユーザー名]** ボックスに「 **Student@contoso.com** 」と入力してから、 **[パスワード]** ボックスに「**Pa55w.rd1234**」と入力し、 **[OK]** を選択します。 
1. **Active Directory Domain Services 構成ウィザード**の **[デプロイ構成]** ページに戻り、 **[次へ]** を選択します。
1. **[ドメイン コントローラーのオプション]** ページで、**[ドメイン ネーム システム (DNS) サーバー]** と **[グローバル カタログ (GC)]** チェックボックスがオンになっていることを確かめます。 **[読み取り専用ドメイン コントローラー (RODC)]** チェック ボックスがオフになっていることを確かめます。
1. **[ディレクトリ サービス復元モード (DSRM) のパスワードを入力してください]** セクションで、パスワード「**Pa55w.rd1234**」を入力して確認し、**[次へ]** を選択します。
1. **Active Directory Domain Services 構成ウィザード**の **[DNS オプション]** ページで、**[次へ]** を選択します。
1. **[追加オプション]** ページで、**[次へ]** を選択します。
1. **[パス]** ページで、**[データベース]** フォルダー、**[ログ ファイル]** フォルダー、および **SYSVOL** フォルダーのパス設定のドライブを **C:** から **F:** に変更し、**[次へ]** を選択します。
1. **[オプションの確認]** ページで、**[次へ]** を選択します。
1. **[前提条件の確認]** ページで、ネットワーク アダプターに静的 IP アドレスが設定されていないことを示す警告に注意し、**[インストール]** を選択します。

   > **注**: 静的 IP アドレスはオペレーティング システム内ではなく、プラットフォーム レベルで割り当てられるため、この警告は予想されるものです。

   > **注**: 昇格プロセスを完了するため、オペレーティング システムが自動的に再起動されます。

1. **SEA-SVR2** の Azure portal が表示されている Microsoft Edge ウィンドウで **az801l06a-dc2** ページに戻り、**[接続]** を選択し、ドロップダウン メニューの **[Bastion]** を選択します。  
1. Bastion ページで、次の資格情報を入力し、 **[接続]** を選択します。

   | 設定 | 値 | 
   | --- | --- |
   | [ユーザー名] |**Student** |
   | パスワード |**Pa55w.rd1234** |

1. **az801l06a-dc2** へのリモート デスクトップ セッションで、**サーバー マネージャー** ウィンドウが開くまで待機し、ローカル環境にインストールされている役割に **AD DS** と **DNS** が含まれることを確認します。

#### タスク 5: 演習でデプロイした Azure リソースを削除する

1. **SEA-SVR2** の Azure portal を表示している Microsoft Edge ウィンドウで、Azure portal の [Cloud Shell] をクリックして [Azure Cloud Shell] ペインの PowerShell セッションを開きます。
1. Cloud Shell ペインで次のコマンドを実行して、このラボで作成したすべてのリソース グループの一覧を表示します。

   ```powershell
   Get-AzResourceGroup -Name 'AZ801-L060*'
   ```

   > **メモ**: 出力に、このラボで作成したリソース グループのみが含まれていることを確認してください。 このグループは、このタスクで削除されます。

1. Cloud Shell ペインで次のコマンドを実行して、このラボで作成したすべてのリソース グループを削除します。

   ```powershell
   Get-AzResourceGroup -Name 'AZ801-L060*' | Remove-AzResourceGroup -Force -AsJob
   ```

   > **注:** このコマンドは非同期で実行されるため (*-AsJob* パラメーターによって決定されます)、同じ PowerShell セッション内で別の PowerShell コマンドをすぐに実行できますが、リソース グループが実際に削除されるまでに数分かかります。

## 演習 2: 記憶域移行サービスを使用してファイル サーバーを移行する

#### タスク 1: Windows Admin Center をインストールする

1. **SEA-SVR2** 上で **[スタート]** を選択し、**[Windows PowerShell]** を選択します。
1. **Windows PowerShell** コンソールで、次のコマンドを入力し、Enter キーを押して、最新バージョンの Windows Admin Center をダウンロードします。
    
   ```powershell
   Start-BitsTransfer -Source https://aka.ms/WACDownload -Destination "$env:USERPROFILE\Downloads\WindowsAdminCenter.msi"
   ```
1. 次のコマンドを入力してから Enter キーを押して、Windows Admin Center をインストールします。
    
   ```powershell
   Start-Process msiexec.exe -Wait -ArgumentList "/i $env:USERPROFILE\Downloads\WindowsAdminCenter.msi /qn /L*v log.txt REGISTRY_REDIRECT_PORT_80=1 SME_PORT=443 SSL_CERTIFICATE_OPTION=generate"
   ```

   > **注**: インストールが完了するまで待ちます。 これには 2 分ほどかかります。

#### タスク 2: ファイル サービスを設定する

1. **SEA-SVR2** のタスク バーで **[エクスプローラー]** を選択します。
1. エクスプローラーで、**C:\\Labfiles\\Lab06** フォルダーに移動します。
1. エクスプローラーの詳細ペインで、ファイル **L06_SetupFS.ps1** を選択してコンテキスト メニューを表示し、そのメニューで **[編集]** を選択します。

   >**注:** これにより、[Windows PowerShell ISE] のスクリプト ペインでファイル **L06_SetupFS.ps1** が自動的に開きます。

1. [Windows PowerShell ISE] のスクリプト ペインでスクリプトを確認した後、ツール バーで **[スクリプトの実行]** アイコンを選択するか、F5 キーを押してスクリプトを実行します。 

   >**注:** スクリプトが完了するのを待ちます。 これには 1 分ほどかかります。

   >**注:** このスクリプトでは、**SEA-SVR1** と **SEA-SVR2** で追加のデータ ディスクが初期化され、それぞれに NTFS ボリュームが作成され、各ボリュームにドライブ文字 **S:** が割り当てられ、**SEA-SVR1** の **S:\Data** フォルダーを使用して **Data** という名前の共有が作成されて、合計サイズ約 1 GB のサンプル ファイルが追加されます。 

#### タスク 3: 記憶域移行サービスを使用して移行を実行する

1. **SEA-SVR2** で Microsoft Edge を起動し、**https://SEA-SVR2.contoso.com** に移動します。 
   
   >**注**: リンクが機能しない場合は、**SEA-SVR2** でエクスプローラー開き、ダウンロード フォルダーを選択し、ダウンロード フォルダー内の **WindowsAdminCenter.msi** ファイルを選択して、手動でインストールします。 インストールが完了したら、Microsoft Edge を更新します。

   >**注**: **NET::ERR_CERT_DATE_INVALID** エラーが発生した場合は、Edge ブラウザー ページの **[詳細設定]** を選択し、ページの下部にある **[sea-svr2-contoso.com (アンセーフ) に移動]** を選択します。

1. ダイアログが表示されたら、**[Windows セキュリティ]** ダイアログ ボックスに次の資格情報を入力して、**[OK]** を選択します。

   - ユーザー名: **CONTOSO\\Administrator**
   - パスワード: **Pa55w.rd**

1. **[このリリースの新機能]** ポップアップ ウィンドウを確認し、右上隅の **[閉じる]** を選択します。
1. Windows Admin Center の **[すべての接続]** ペインで、右上隅にある **[設定]** アイコン (歯車) を選択します。
1. 左側のウィンドウで、**[拡張機能]** を選択します。 使用可能な拡張機能を確認します。
1. 詳細ペインで、**[インストール済みの拡張機能]** を選択し、一覧に**記憶域移行サービス**拡張機能が含まれていることを確認します。

   >**注:** 利用可能な更新プログラムがある場合は、**記憶域移行サービス**拡張機能のエントリを選び、**[更新]** を選びます。

1. 上部メニューの **[設定]** の横にあるドロップダウン矢印を選択してから、**[サーバー マネージャー]** を選択します。
1. **[すべての接続]** ペインで **sea-svr2.contoso.com** リンクを選択します。
1.  **[sea-svr2.contoso.com]** ページで **[ツール]** メニューの **[記憶域移行]** エントリを選択します。
1. **[記憶域移行]** ペインで、**[インストール]** を選択します。

   >**注:** これにより、記憶域移行サービスとその必要なコンポーネントが自動的にインストールされます。

1. **[Migrate storage in three steps](3 つのステップで記憶域を移行する)** ペインで、**[閉じる]** を選択します。
1. **[記憶域移行サービス]** ペインで、ページの一番下までスクロールし、**[+ 新しいジョブ]** を選択します。
1. **[新しいジョブ]** ペインの **[ジョブ名]** テキスト ボックスに「**SVR1toSVR2**」と入力し、**Windows サーバーとクラスター**の **[ソース デバイス]** オプションが選択されていることを確認して、**[OK]** を選択します。
1. **[記憶域移行サービス > SVR1toSVR2]** ペインの **[Inventory servers](インベントリ サーバー)** タブで、 **[Check the prerequisites](前提条件の確認)** ペインを確認し、 **[次へ]** を選択します。
1. **[Inventory servers](インベントリ サーバー)** タブの **[資格情報の入力]** ペインで、必要に応じて、**CONTOSO\\Administrator** ユーザー アカウントの資格情報を入力し、 **[Migrate from failover clusters](フェールオーバー クラスターから移行する)** チェック ボックスをオフにし、 **[次へ]** を選択します。
1. **[インベントリ サーバー]** タブの **[Install required features](必要な機能のインストール)** ペインで、 **[次へ]** を選択します。
1. **[インベントリ サーバー]** タブの **[Add and scan devices](デバイスの追加とスキャン)** ペインで、**[デバイスの追加]** を選択します。
1. **[ソース デバイスの追加]** で、**[デバイス名]** オプションが選択されていることを確認し、**[名前]** テキスト ボックスに「**SEA-SVR1.contoso.com**」と入力して、**[追加]** を選択します。
1. **[Specify your credentials](資格情報の指定)** ペインで、**[この接続に別のアカウントを使用する]** オプションを選択し、次の資格情報を入力し、**[Use these credentials for all connections](すべての接続にこれらの資格情報を使用する)** を選択し、**[続行]** を選択します。

   - ユーザー名: **CONTOSO\\Administrator**
   - パスワード: **Pa55w.rd**

   > **注**: シングル サインオンを実行するには、Kerberos の制約付き委任を設定する必要があります。<!--Marcin can this be 'a Kerberos constrained delegation'?-->=

1. デバイスの一覧で、新しく追加した **SEA-SVR1.contoso.com** エントリを選択し、**[Add and scan devices](デバイスの追加とスキャン)** ペインのツールバーで、省略記号ボタン ([**...**]) を選択してから、ドロップダウンメニューで **[スキャンの開始]** を選択します。

   >**注:** スキャンが正常に完了するまで待ちます。 これには 1 分ほどかかります。

1. **[インベントリ サーバー]** タブの **[Add and scan devices](デバイスの追加とスキャン)** ペインで、**[次へ]** を選択します。 

   >**注:** これにより、**[記憶域移行サービス > SVR1toSVR2]** ペインの **[Transfer data](データの転送)** タブからアクセスできる移行ジョブの 2 番目のステージに移行します。

1. **[Transfer data](データの転送)** タブの **[Enter credentials for the destination device](宛先デバイスの資格情報を入力します)** ペインで、**CONTOSO\\Administrator** ユーザー アカウントが使用されていることを確認し、**[次へ]** を選択します。
1. **[Specify the destination for: sea-svr1.contoso.com](転送先の指定先: sea-svr1.contoso.com)** ペインで、**[宛先]** オプションが **[Use an existing server or VM](既存のサーバーまたは VM を使用する)** に設定されていることを確認し、**[宛先デバイス]** ボックスに「**SEA-SVR2.contoso.com**」と入力し、**[スキャン]** を選択します。

   >**注:** スキャンが正常に完了するまで待ちます。 これには 1 分ほどかかります。

   >**注: ** ハイブリッド シナリオでは、移行ジョブの移行先になる Azure VM を自動的に作成することもできます。

1. スキャンが完了したら、**[Specify the destination for: sea-svr1.contoso.com](転送先の指定先: sea-svr1.contoso.com)** ペインで、**[Map each source volume to a destination volume](各転送元ボリュームを転送先ボリュームにマップする)** セクションを調べて、**S:** 転送元ボリュームが **S:** 転送先ボリュームにマップされていることを確認します。
1. **[Specify the destination for: sea-svr1.contoso.com](転送先の指定先: sea-svr1.contoso.com)** ペインで、**[Select the shares to transfer](転送する共有の選択)** セクションを調べて、**[データ]** 転送元共有が転送に含まれていることを確認し、**[次へ]** を選択します。
1. **[Transfer data](データの転送)** タブの **[Adjust transfer settings](転送設定の調整)** ペインで、次の設定を指定し (他の設定は既定値のまま)、**[次へ]** を選択します。

   | 設定 | 値 | 
   | --- | --- |
   | Back up folders that would be overwritten (Azure File Sync-enabled shares aren't backed up) (上書きされるフォルダーをバックアップする (Azure File Sync が有効な共有はバックアップしない)) | enabled |
   | Validation method (検証方法) | **CRC 64** |
   | Max duration (minutes) (最大継続時間 (分)) | **60** |
   | Migrate users and groups (ユーザーとグループを移行する) | **Reuse accounts with the same name (アカウントを同じ名前で再利用する)** |
   | Max retries (最大再試行回数) | **3** |
   | Delay between retries (seconds) (再試行の間隔 (秒)) | **60** |

1. **[Transfer data](データの転送)** タブの **[Install required features](必要な機能のインストール)** ペインで、**SEA-SVR2.contoso.com** への **SMS-Proxy** のインストールが完了するまで待ってから、 **[次へ]** を選択します。
1. **[Transfer data](データの転送)** タブの **[Validate source and destination devices](転送元と転送先のデバイスの検証)** ペインで **[検証]** を選択し、検証が正常に完了するまで待ってから **[次へ]** を選択します。
1. **[Transfer data](データの転送)** タブの **[Start the transfer](転送の開始)** ペインで、 **[転送の開始]** を選択し、完了するまで待ち、 **[次へ]** を選択します。

   >**注:** 転送が正常に完了するまで待ちます。 これにかかる時間は 1 分未満です。

   >**注:** これにより、**[記憶域移行サービス > SVR1toSVR2]** ペインの **[Cut over to the new servers](新しいサーバーへの切り替え)** タブからアクセスできる移行ジョブの 3 番目のステージに移ります。

1. **[Cut over to the new servers](新しいサーバーへの切り替え)** タブの **[Enter credentials for the source devices](移行元デバイスの資格情報の入力)** セクションと **[Enter credentials for the destination devices](移行先デバイスの資格情報の入力)** セクションで、**CONTOSO\\Administrator** ユーザー アカウントの保存されている資格情報を受け入れ、**[次へ]** を選択します。
1. **[Cut over to the new servers](新しいサーバーへの切り替え)** タブの **[Configure cutover from sea-svr1.contoso.com to sea-svr2.contoso.com](sea-svr1.contoso.com から sea-svr2.contoso.com への切り替えの構成)** ペインの **[Source network adapters](移行元ネットワーク アダプター)** セクションで、次の設定を指定します。

   | 設定 | 値 | 
   | --- | --- |
   | DHCP を使用する | disabled |
   | IP アドレス | **172.16.10.111** |
   | サブネット | **255.255.0.0** |
   | Gateway | **172.16.10.1** |

1. **[Cut over to the new servers](新しいサーバーへの切り替え)** タブの **[Configure cutover from sea-svr1.contoso.com to sea-svr2.contoso.com](sea-svr1.contoso.com から sea-svr2.contoso.com への切り替えの構成)** ペインの **[Destination network adapters](移行先ネットワーク アダプター)** ドロップダウン リストで、 **[イーサネット]** を選びます。
1. **[Cut over to the new servers](新しいサーバーへの切り替え)** タブの **[Configure cutover from sea-svr1.contoso.com to sea-svr2.contoso.com](sea-svr1.contoso.com から sea-svr2.contoso.com への切り替えの構成)** ペインの **[Rename the source device after cutover](切り替え後に移行元デバイスの名前を変更する)** セクションで、 **[Choose a new name](新しい名前の選択)** オプションを選び、 **[New source computer name](新しい移行元コンピューター名)** で、<!--text box?-->「**SEA-SVR1-OLD**」と入力し、**[次へ]** を選択します。
1. **[Cut over to the new servers](新しいサーバーへの切り替え)** タブの **[Adjust cutover settings](切り替え設定の調整)** ペインの **[Cutover timeout (minutes)](切り替えタイムアウト (分))** テキスト ボックスに「**30**」と入力し、**[Enter AD credentials](AD 資格情報の入力)** セクションの **[Stored credentials](保存された資格情報)** オプションは有効のままにし、**[次へ]** を選択します。
1. **[Cut over to the new servers](新しいサーバーへの切り替え)** タブの **[Validate source and destination device](転送元と転送先のデバイスの検証)** ペインで **[検証]** を選択し、検証が正常に完了するまで待ってから **[次へ]** を選択します。
1. **[Cut over to the new servers](新しいサーバーへの切り替え)** タブの **[Cut over to the new servers](新しいサーバーへの切り替え)** ペインで、**[一括移行の開始]** を選択します。

   >**注:** 切り替えによって、**SEA-SVR1** と **SEA-SVR2** 両方の 2 回連続した再起動がトリガーされます。

#### タスク 4: 移行の結果を検証する

1. **SEA-SVR2** で、パスワード **Pa55w.rd** を使用して **CONTOSO\\Administrator** としてサインインします。
1. **SEA-SVR2** 上で **[スタート]** を選択し、**[Windows PowerShell]** を選択します。
1. **SEA-SVR2** のネットワーク インターフェイスに割り当てられた IPv4 アドレスを確認するには、**Windows PowerShell** コンソールで次のコマンドを入力し、Enter キーを押します。
    
   ```powershell
   Get-NetIPAddress | Where-Object AddressFamily -eq 'IPv4' | Select-Object IPAddress
   ```

   >**注:** 出力に **172.16.10.11** と **172.16.10.12** の両方が含まれていることを確認します。

1. **SEA-SVR2** に割り当てられた NetBIOS 名を確認するには、**Windows PowerShell** コンソールで次のコマンドを入力し、Enter キーを押します。
    
   ```powershell
   nbtstat -n
   ```

   >**注:** 出力に **SEA-SVR1** と **SEA-SVR2** の両方が含まれていることを確認します。

1. **SEA-SVR2** のローカル共有を確認するには、**Windows PowerShell** コンソールで次のコマンドを入力し、Enter キーを押します。
    
   ```powershell
   Get-SMBShare
   ```

   >**注:** 出力に **S:\Data** フォルダーでホストされている **Data** 共有が含まれることを確認します。

1. **SEA-SVR2** の **Data** 共有の内容を確認するには、**Windows PowerShell** コンソールで次のコマンドを入力し、Enter キーを押します。
    
   ```powershell
   Get-ChildItem -Path 'S:\Data'
   ```


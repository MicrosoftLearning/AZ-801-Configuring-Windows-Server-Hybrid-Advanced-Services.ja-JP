---
lab:
  title: 'ラボ: フェールオーバー クラスタリングの実装'
  type: Answer Key
  module: 'Module 3: High availability in Windows Server'
ms.openlocfilehash: 0aebdfd4b42079ec2266db4724a3087a3623723f
ms.sourcegitcommit: 9a51ea796ef3806ab9e7ec1ff75034b2f929ed2a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/28/2022
ms.locfileid: "137907090"
---
# <a name="lab-answer-key-implementing-failover-clustering"></a>ラボ応答キー: フェールオーバー クラスタリングの実装

## <a name="exercise-1-configuring-iscsi-storage"></a>演習 1: iSCSI ストレージの構成

#### <a name="task-1-install-failover-clustering"></a>タスク 1: フェールオーバー クラスタリングをインストールする

1. **SEA-SVR2** に接続し、必要であればパスワード **Pa55w.rd** を使用して **Contoso\\管理者** としてサインインします。
1. **SEA-SVR2** 上で **[スタート]** を選択し、 **[Windows PowerShell (管理者)]** を選択します。
1. **SEA-SVR1** と **SEA-SVR2** 上に、管理ツールを含むフェールオーバー クラスタリング サーバー機能をインストールするには、Windows PowerShell コマンド プロンプトで次のコマンドを入力し、各コマンドを入力した後、Enter キーを押します。

   ```powershell
   Install-WindowsFeature –Name Failover-Clustering –IncludeManagementTools
   Install-WindowsFeature -ComputerName 'SEA-SVR1.contoso.com' –Name Failover-Clustering –IncludeManagementTools
   ```

   > **注**: インストール プロセスが完了するまで待ちます。 インストールには約 1分かかります。

1. **SEA-DC1** 上に iSCSI ターゲットサーバーの役割サービスをインストールするには、Windows PowerShell コマンド プロンプトで次のコマンドを入力し、Enter キーを押します。

   ```powershell
   Install-WindowsFeature -ComputerName 'SEA-DC1.contoso.com' –Name FS-iSCSITarget-Server –IncludeManagementTools
   ```

   > **注**: インストール プロセスが完了するまで待ちます。 インストールには約 1分かかります。

#### <a name="task-2-configure-iscsi-virtual-disks"></a>タスク 2: iSCSI 仮想ディスクを構成する

> **重要:** このラボでは、**SEA-DC1** を使用します。これは、Windows サーバーベースのクラスターの共有 iSCI ストレージをホストするための Active Directory Domain Services (AD DS) ドメイン コントローラーとして機能します。 これは、推奨される構成を任意の方法で表すことを目的としたものではなく、ラボの構成を簡略化し、ラボの仮想マシンの数を最小限に抑えるために行われます。 運用環境では、フェールオーバー クラスター用の共有ストレージをホストするためにドメイン コントローラーを使用しないでください。 そうせずに、そのようなストレージは、高可用性インフラストラクチャでホストする必要があります。 

1. **SEA-SVR2** 上で **[スタート]** を選択し、 **[Windows PowerShell (管理者)]** を選択します。
1. **SEA-DC1** への PowerShell リモート処理セッションを確立するには、新しく開いた **Windows PowerShell** ウィンドウで次のコマンドを入力し、Enter キーを押します。

   ```powershell
   Enter-PSSession -ComputerName SEA-DC1.contoso.com
   ```

1. **SEA-DC1**、**SEA-SVR2** 上で iSCSI 仮想ディスクを作成するには、**SEA-DC1** への PowerShell リモート処理セッションで次のコマンドを入力し、各コマンドを入力した後、Enter キーを押します。

   ```powershell
   New-Item -ItemType Directory C:\Storage -Force
   New-IscsiVirtualDisk C:\Storage\disk1.VHDX –size 10GB
   New-IscsiVirtualDisk C:\Storage\disk2.VHDX –size 10GB
   New-IscsiVirtualDisk C:\Storage\disk3.VHDX –size 10GB
   ```

1. **SEA-SVR2** 上で **[スタート]** を選択し、 **[Windows PowerShell (管理者)]** を選択します。
1. **SEA-SVR1** への PowerShell リモート処理セッションを確立するには、新しく開いた **Windows PowerShell** ウィンドウで次のコマンドを入力し、Enter キーを押します。

   ```powershell
   Enter-PSSession -ComputerName SEA-SVR1.contoso.com
   ```

   > **注:** この時点で、3 つの **Windows PowerShell** ウィンドウが開いている必要があります。 最初のウィンドウを使用して **SEA-SVR2** 上でローカルにコマンドを実行し、他の 2 つを使用して **SEA-DC1** および **SEA-SVR1** と対話します。 PowerShell プロンプトを識別することで、それぞれを簡単に認識できます (2 番目と 3番目のプロンプトでは、プロンプトにはそれぞれ **[SEA-DC1.contoso.com]** と **[SEA-SVR1.contoso.com]** のプレフィックスが含まれます)。

1. **SEA-SVR2** で Microsoft iSCSI イニシエーター サービスを開始するには、ローカル セッションへのアクセスを提供する **Windows PowerShell** プロンプトで次のコマンドを入力し、各コマンドを入力したら Enter キーを押します。

   ```powershell
   Start-Service -ServiceName MSiSCSI
   Set-Service -ServiceName MSiSCSI -StartupType Automatic
   ```

1. **SEA-SVR1** 上で Microsoft iSCSI イニシエーター サービスを開始するには、**SEA-SVR1** への PowerShell リモート処理セッションをホストしている **Windows PowerShell** ウィンドウに切り替え、次のコマンドを入力します。各コマンドを入力したら、Enter キーを押します。

   ```powershell
   Start-Service -ServiceName MSiSCSI
   Set-Service -ServiceName MSiSCSI -StartupType Automatic
   ```

1. **SEA-DC1** 上に Microsoft iSCSI ターゲットを作成するには、**SEA-DC1** への PowerShell リモート処理セッションをホストしている **Windows PowerShell** ウィンドウに切り替え、次のコマンドを入力して、Enter キーを押します。

   ```powershell
   New-IscsiServerTarget iSCSI-L03 –InitiatorIds “IQN:iqn.1991-05.com.microsoft:sea-svr1.contoso.com","IQN:iqn.1991-05.com.microsoft:sea-svr2.contoso.com"
   ```

## <a name="exercise-2-configuring-a-failover-cluster"></a>演習 2: フェールオーバー クラスターの構成

#### <a name="task-1-connect-clients-to-the-iscsi-targets"></a>タスク 1: クライアントを iSCSI ターゲットに接続する

1. **SEA-SVR2** から **SEA-DC1** 上に iSCSI ディスクをマウントするには、**SEA-DC1** への PowerShell リモート処理セッションをホストしている **Windows PowerShell** ウィンドウで次のコマンドを入力し、各コマンドを入力した後に Enter キーを押します。

   ```powershell
   Add-IscsiVirtualDiskTargetMapping iSCSI-L03 C:\Storage\Disk1.VHDX
   Add-IscsiVirtualDiskTargetMapping iSCSI-L03 C:\Storage\Disk2.VHDX
   Add-IscsiVirtualDiskTargetMapping iSCSI-L03 C:\Storage\Disk3.VHDX
   ```

1. **SEA-DC1** 上でホストされている iSCSI ターゲットに **SEA-SVR2** から接続するには、ローカル セッションへのアクセスを提供する **Windows PowerShell** プロンプトに切り替え、次のコマンドを入力します。各コマンドを入力したら、Enter キーを押します。

   ```powershell
   New-iSCSITargetPortal –TargetPortalAddress SEA-DC1.contoso.com  
   Connect-iSCSITarget –NodeAddress iqn.1991-05.com.microsoft:sea-dc1-iSCSI-L03-target
   Get-iSCSITarget | fl
   ```

   > **注:** 最後のコマンドを実行した後、*IsConnected* 変数の値が [True] であることを確認してください。

1. **SEA-SVR1** から、**SEA-DC1** 上でホストされている iSCSI ターゲットに接続するには、**SEA-SVR1** への PowerShell リモート処理セッションをホストしている Windows PowerShell ウィンドウに切り替え、次のコマンドを入力します。各コマンドを入力したら、Enter キーを押します。

   ```powershell
   New-iSCSITargetPortal –TargetPortalAddress SEA-DC1.contoso.com 
   Connect-iSCSITarget –NodeAddress iqn.1991-05.com.microsoft:sea-dc1-iSCSI-L03-target
   Get-iSCSITarget | fl
   ```

   > **注:** 最後のコマンドを実行した後、*IsConnected* 変数の値が [True] であることを確認してください。

#### <a name="task-2-initialize-the-disks"></a>タスク 2: ディスクを初期化する

1. **SEA-SVR2** 上のディスクを一覧表示するには、ローカル セッションへのアクセスを提供する Windows PowerShell プロンプトに切り替え、次のコマンドを入力して、Enter キーを押します。

   ```powershell
   Get-Disk
   ```

   > **注:** 3 つの iSCSI ディスクが **オフライン** の動作状態で一覧表示されていることを確認します。 これらは番号 1、2、3 のディスクである必要があります。

1. ディスクを初期化するには、ローカル セッションへのアクセスを提供する Windows PowerShell プロンプトで次のコマンドを入力し、各コマンドを入力したら Enter キーを押します。

   ```powershell
   Get-Disk | Where OperationalStatus -eq 'Offline' | Initialize-Disk -PartitionStyle MBR
   New-Partition -DiskNumber 1 -Size 5gb -AssignDriveLetter
   New-Partition -DiskNumber 2 -Size 5gb -AssignDriveLetter
   New-Partition -DiskNumber 3 -Size 5gb -AssignDriveLetter
   Format-Volume -DriveLetter E -FileSystem NTFS
   Format-Volume -DriveLetter F -FileSystem NTFS
   Format-Volume -DriveLetter G -FileSystem NTFS
   ```

   > **注:** 各コマンドが正常に完了したことを確認します。

#### <a name="task-3-create-a-failover-cluster"></a>タスク 3: フェールオーバー クラスターを作成する

1. フェールオーバー クラスターを作成するには、**SEA-SVR2** 上の、ローカル セッションへのアクセスを提供する Windows PowerShell プロンプトで、次のコマンドを入力して、Enter キーを押します。

   ```powershell
   New-Cluster -Name SEA-CL03 -Node SEA-SVR2.contoso.com -StaticAddress 172.16.10.125
   ```

   > **注:** コマンドは、新しく作成されたクラスター (**SEA-CL03**) の名前を返す必要があります。 

1. 新規に作成したクラスターに **SEA-SVR1** を別のノードとして追加するには、**SEA-SVR2** 上の、ローカル セッションへのアクセスを提供する Windows PowerShell プロンプトで、次のコマンドを入力して、Enter キーを押します。

   ```powershell
   Add-ClusterNode -Cluster SEA-CL03 -Name SEA-SVR1.contoso.com
   ```

   > **注:** コマンドが正常に完了したことを確認します。

## <a name="exercise-3-deploying-and-configuring-a-highly-available-file-server"></a>演習 3: 高可用性ファイル サーバーのデプロイと構成

#### <a name="task-1-add-the-file-server-application-to-the-failover-cluster"></a>タスク 1: ファイル サーバー アプリケーションをフェールオーバー クラスターに追加する

1. **SEA-SVR2** 上で、 **[スタート]** メニューの **[スタート]** を選択し、 **[サーバー マネージャー]** を選択した後、 **[サーバー マネージャー]** で、 **[ツール]** メニューの **[フェールオーバー クラスター マネージャー]** を選択します。

   > **注:** **SEA-SVR2** がクラスター ノードの 1 つであるため、**フェールオーバー クラスター マネージャー** コンソールは **SEA-CL03** に自動的に接続されます。

1. **[SEA-CL03.contoso.com]** ノードを展開し、 **[ロール]** を選択して、この時点でクラスターがロールをホストしていないことを確認します。
1. **[ノード]** ノードを選択し、 **[SEA-SVR1]** および **[SEA-SVR2]** ノードが **[Up]** 状態で表示されていることを確認します。
1. **[ストレージ]** ノードを展開し、 **[ディスク]** を選択します。 3 つのクラスター ディスクが **[オンライン]** 状態で表示されていることに注意してください。
1. **[フェールオーバー クラスター マネージャー]** ページで、 **[ロール]** のコンテキスト メニューを右クリックするかアクセスし、 **[ロールの構成]** を選択します。 これにより、 **[高可用性ウィザード]** が開始されます。
1. **[高可用性ウィザード]** の **[開始する前に]** ページで、 **[次へ]** をクリックします。
1. **[高可用性ウィザード]** の **[ロールの選択]** ページで、 **[ファイル サーバー]** を選択し、 **[次へ]** を選択します。
1. **[高可用性ウィザード]** の **[ファイル サーバーの種類]** ページで、 **[汎用のファイル サーバー]** オプションが選択されていることを確認し、 **[次へ]** を選択します。
1. **[高可用性ウィザード]** の **[クライアント アクセス ポイント]** ページの、 **[名前]** ボックスに「**FSCluster**」と入力します。
1. **[アドレス]** ボックスに「**172.16.10.130**」と入力し、 **[次へ]** を選択します。
1. **[高可用性ウィザード]** の **[ストレージの選択]** ページで、 **[クラスター ディスク 1 ]** と **[クラスター ディスク 2]** を選択し、 **[次へ]** を選択します。
1. **[高可用性ウィザード]** の **[確認]** ページで、 **[次へ]** を選択します。
1. **[高可用性ウィザード]** の **[概要]** ページで、 **[完了]** を選択します。

   > **注:** **[ストレージ]** ノードで、 **[ディスク]** ノードを選択した状態で、3 つのクラスター ディスクがオンラインになっていることを確認します。 **クラスターディスク 1** と **クラスターディスク 2** を **FSCluster** に割り当てる必要があります。

#### <a name="task-2-add-a-shared-folder-to-a-highly-available-file-server"></a>タスク 2: 高可用性ファイル サーバーに共有フォルダーを追加する

1. **SEA-SVR2** 上の **[フェールオーバー クラスター マネージャー]** で、 **[ロール]** を選択し、 **[FSCluster]** を選択します。次に、操作ウィンドウで **[ファイル共有の追加]** を選択します。 

   > **注:** これにより、**新しい共有ウィザード** が開始されます。

1. **[プロファイルの選択]** ページで、 **[SMB 共有 - クイック]** プロファイルが選択されていることを確認し、 **[次へ]** を選択します。
1. **[共有の場所]** ページで、 **[次へ]** を選択します。
1. **[共有名]** ページで、共有名に「**ドキュメント**」と入力し、 **[次へ]** を選択します。
1. **[その他の設定]** ページで、 **[次へ]** を選択します。
1. **[アクセス許可]** ページで、 **[次へ]** を選択します。
1. **[確認]** ページで、 **[作成]** を選択します。
1. **[結果の表示]** ページで、 **[閉じる]** を選択します。

#### <a name="task-3-configure-the-failover-and-failback-settings"></a>タスク 3: フェールオーバーとフェールバックの設定を構成する

1. **SEA-SVR2** 上の **[フェールオーバー クラスター マネージャー]** コンソールで、 **[ロール]** ノードで **[FSCluster]** が選択された状態で、操作ウィンドウで **[プロパティ]** を選択します。
1. **[フェールオーバー]** タブを選択し、 **[フェールバックを許可する]** オプションを選択します。
1. **[Failback between]** \(フェールバック期間\) オプションを選択し、次の値を入力します。

   - 最初のテキスト ボックスには「**4**」を入力します
   - 2 つ目のテキスト ボックスには「**5**」と入力します。

1. **[全般]** タブを選びます。
1. **[優先所有者]** セクションで、**SEA-SVR1** が最初のエントリとしてリストされていることを確認し、 **[OK]** を選択します。


## <a name="exercise-4-validating-the-deployment-of-the-highly-available-file-server"></a>演習 4: 高可用性ファイル サーバーのデプロイの検証

#### <a name="task-1-validate-the-highly-available-file-server-deployment"></a>タスク 1: 高可用性ファイル サーバーのデプロイを検証する

1. **SEA-SVR2** で、エクスプローラーを開き、 **\\\\FSCluster\\Docs** フォルダーに移動します。
1. **Docs** フォルダー内で、フォルダーの空の領域にあるコンテキスト メニューを右クリックまたはアクセスして、 **[新規作成]** を選択し、 **[テキスト ドキュメント]** を選択します。
1. ドキュメントのデフォルト名「**New Text Document.txt**」を受け入れるには、Enter キーを押します。
1. **SEA-SVR2** で、 **[フェールオーバー クラスター マネージャー]** コンソールに切り替え、 **FSCluster** のコンテキスト メニューを右クリックするかアクセスして、 **[移動]** 、 **[ノードの選択]** の順に選択し、 **[OK]** を選択します。
1. **SEA-SVR2** 上で、エクスプローラーに戻り、 **\\\\FSCluster\\Docs** フォルダーの内容に引き続きアクセスできることを確認します。

#### <a name="task-2-validate-the-failover-and-quorum-configuration-for-the-file-server-role"></a>タスク 2: ファイル サーバー ロールのフェールオーバーとクォーラム構成を検証する

1. **SEA-SVR2** 上で、**フェールオーバー クラスター マネージャー** コンソールに切り替え、**FSCluster** ロールの現在の所有者を確認します。
1. **[ノード]** を選択し、前の手順で特定したノードのコンテキスト メニューを右クリックするか、アクセスします。
1. コンテキスト メ ニューで **[その他のアクション]** を選択し、 **[クラスター サービスの停止]** を選択します。
1. エクスプローラーに切り替え、 **\\\\FSCluster\\Docs** フォルダーの内容に引き続きアクセスできることを確認します。
1. **フェールオーバー クラスター マネージャー** コンソールに切り替えた後、 **[ダウン]** 状態のノードのコンテキスト メニューを右クリックするか、アクセスします。
1. コンテキスト メ ニューで **[その他のアクション]** を選択し、 **[クラスター サービスの開始]** を選択します。
1. **フェールオーバー クラスター マネージャー** コンソールで、 **SEA-CL03.Contoso.com** クラスターのコンテキスト メニューを右クリックするか、アクセスして、 **[その他のアクション]** 、 **[クラスター クォーラム設定の構成]** の順に選択します。 これにより、 **[クラスター クォーラム構成ウィザード]** が開始されます。
1. **[開始する前に]** ページで、**[次へ ]** を選択します。
1. **[クォーラム構成オプションの選択]** ページで、 **[既定のクォーラム構成を使用する]** オプションが選択されていることを確認し、 **[次へ]** を選択します。
1. **[確認]** ページで、既定では **[ディスク監視]** として **[クラスター ディスク 3]** が選択されていることを確認し、 **[次へ]** を選択します。 
1. **[概要]** ページで、 **[完了]** を選択します。
1. **フェールオーバー クラスター マネージャー** コンソールで、 **[ディスク]** ノードを参照し、ディスク監視として構成されている **[クラスター ディスク 3]** を選択してから、操作ウィンドウで **[オフラインにする]** を選択します。
1. 確認を求められたら、 **[はい]** を選択します。
1. エクスプローラーに切り替え、 **\\\\FSCluster\\Docs** フォルダーの内容に引き続きアクセスできることを確認します。
1. **フェールオーバー クラスター マネージャー** コンソールに切り替え、 **[ディスク]** ノード内のディスクの一覧で、ディスク監視として構成されている **[クラスター ディスク 3]** を選択し、操作ウィンドウで **[オンラインに戻す]** を選択します。

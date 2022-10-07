---
lab:
  title: 'ラボ: Hyper-V レプリカおよび Windows Server バックアップの実装'
  type: Answer Key
  module: 'Module 4: Disaster Recovery in Windows Server'
---

# <a name="lab-answer-key-implementing-hyper-v-replica-and-windows-server-backup"></a>ラボ回答キー: Hyper-V レプリカと Windows Server バックアップの実装

## <a name="exercise1-implementing-hyper-v-replica"></a>演習 1: Hyper-V レプリカの実装

#### <a name="task-1-install-and-configure-hyper-v-replica"></a>タスク 1: Hyper-V レプリカをインストールして構成する

1. **SEA-SVR2** に接続し、必要であればパスワード **Pa55w.rd** を使用して **Contoso\\管理者**としてサインインします。
1. **SEA-SVR2** 上で **[スタート]** を選択し、**[Windows PowerShell (管理者)]** を選択します。
1. **SEA-SVR2** でセキュリティが強化された **Hyper-V レプリカ HTTP Listener (TCP-In)** 規則を使用する Windows Defender ファイアウォールの状態を確認するには、Windows PowerShell プロンプトで次のコマンドを入力して Enter キーを押します。

   ```powershell
   Get-NetFirewallRule -DisplayName 'Hyper-V Replica HTTP Listener (TCP-In)'
   ```

1. **SEA-SVR2** でセキュリティが強化された **Hyper-V レプリカ HTTP Listener (TCP-In)** 規則を使用する Windows Defender ファイアウォールを有効にするには、次のコマンドを入力して Enter キーを押します。

   ```powershell
   Enable-NetFirewallRule -DisplayName 'Hyper-V Replica HTTP Listener (TCP-In)'
   ```

1. **HYPER-V レプリカ**のレプリカ サーバーとして **SEA-SVR2** を構成するには、次のコマンドを入力し、各コマンドを入力した後、Enter キーを押します。

   ```powershell
   New-Item -ItemType Directory -Path C:\ReplicaStorage -Force
   Set-VMReplicationServer -ReplicationEnabled $true -AllowedAuthenticationType Kerberos -KerberosAuthenticationPort 8080 -ReplicationAllowedFromAnyServer $true -DefaultStorageLocation C:\ReplicaStorage
   ```

1. **Hyper-V レプリカ**のレプリカ サーバーとして **SEA-SVR2** が構成されていることを確認するには、次のコマンドを入力して Enter キーを押します。

   ```powershell
   Get-VMReplicationServer
   ```
  
   > **注**: コマンドの出力に次の設定が含まれていることを確認してください。

   - **RepEnabled: True**
   - **AuthType: Kerb**
   - **KerAuthPort: 8080**
   - **CertAuthPort: 443**
   - **AllowAnyServer: True**

1. **SEA-SVR2** に存在する仮想マシンを識別するには、次のコマンドを入力して Enter キーを押します。

   ```powershell
   Get-VM
   ```

   > **注**: コマンドの出力に **SEA-CORE1** が含まれていることを確認してください。 

   > **注**: **[管理者: Windows PowerShell]** ウィンドウは開いたままにしておきます。

1. 別の **[管理者: Windows PowerShell]** ウィンドウを開くには、**SEA-SVR2** で **[開始]** を選択し、**[Windows PowerShell (管理者)]** を選択します。
1. **SEA-SVR1** への PowerShell リモート処理セッションを確立するには、新しく開いた Windows PowerShell ウィンドウで次のコマンドを入力し、Enter キーを押します。

   ```powershell
   Enter-PSSession -ComputerName SEA-SVR1.contoso.com
   ```
 
   > **注**: Powershell リモート処理セッションは、この場合は **[SEA-SVR1.contoso.com]** プレフィックスを含む PowerShell プロンプトに基づいて認識できます。

1. **SEA-SVR1** でセキュリティが強化された **Hyper-V レプリカ HTTP Listener (TCP-In)** 規則を使用する Windows Defender ファイアウォールの状態を特定するには、**SEA-SVR1** への PowerShell リモート処理セッションをホストしている Windows PowerShell ウィンドウで、次のコマンドを入力して Enter キーを押します。

   ```powershell
   Get-NetFirewallRule -DisplayName 'Hyper-V Replica HTTP Listener (TCP-In)'
   ```

   > **注**: 出力を確認し、**Enabled** プロパティが **False** に設定されていることを確認します。 **Hyper-V レプリカ**を使用するには、このファイアウォール規則を有効にする必要があります。

1. **SEA-SVR1** でセキュリティが強化された **Hyper-V レプリカ HTTP Listener (TCP-In)** 規則を使用する Windows Defender ファイアウォールを有効にするには、**SEA-SVR1** への PowerShell リモート処理セッションをホストしている Windows PowerShell ウィンドウで、次のコマンドを入力して Enter キーを押します。

   ```powershell
   Enable-NetFirewallRule -DisplayName 'Hyper-V Replica HTTP Listener (TCP-In)'
   ```

1. **SEA-SVR1** を **Hyper-V レプリカ**のレプリカ サーバーとして構成するには、**SEA-SVR1** への PowerShell リモート処理セッションをホストしている Windows PowerShell ウィンドウで次のコマンドを入力し、各コマンドを入力した後、Enter キーを押します。

   ```powershell
   New-Item -ItemType Directory -Path C:\ReplicaStorage -Force
   Set-VMReplicationServer -ReplicationEnabled $true -AllowedAuthenticationType Kerberos -ReplicationAllowedFromAnyServer $true -DefaultStorageLocation C:\ReplicaStorage
   ```

   > **注**: 2 つ目の **[管理者: Windows PowerShell]** ウィンドウは開いたままにしておきます。

#### <a name="task-2-configure-hyper-v-replication"></a>タスク 2: Hyper-V レプリケーションを構成する

1. **SEA-SVR2** で、ローカルの PowerShell セッションを表示している **[管理者: Windows PowerShell]** ウィンドウに切り替えます。
1. **SEA-SVR2** から **SEA-SVR1** への仮想マシン **SEA-CORE1** のレプリケーションを有効にするには、**SEA-SVR2** 上の ローカル セッションの Windows PowerShell プロンプトで、次のコマンドを入力して Enter キーを押します。

   ```powershell
   Enable-VMReplication SEA-CORE1 -ReplicaServerName SEA-SVR1.contoso.com -ReplicaServerPort 80 -AuthenticationType Kerberos -ComputerName SEA-SVR2.contoso.com
   ```

1. **SEA-SVR2** から **SEA-SVR1** への仮想マシン **SEA-CORE1** のレプリケーションを開始するには、**SEA-SVR2** 上で次のコマンドを入力して Enter キーを押します。

   ```powershell
   Start-VMInitialReplication SEA-CORE1
   ```

1. **SEA-SVR2** から **SEA-SVR1** への仮想マシン **SEA-CORE1** のレプリケーションの状態が正常に開始されたことを確認するには、**SEA-SVR2** で次のコマンドを入力して Enter キーを押します。

   ```powershell
   Get-VMReplication
   ```

   > **注**: コマンドの出力で、**状態** の値を特定し、その値が **InitialReplicationInProgress** として表示されていることを確認します。 約 5 分間待ってから、同じコマンドを再実行し、**状態** の値が **[レプリケート中]** に変更されたことを確認します。 これが発生するまで待ってから、次の手順に進みます。 また、**プライマリ サーバー**が **SEA-SVR2** として表示され、**ReplicaServer** が **SEA-SVR1** として表示されていることを確認します。

1. **SEA-SVR2** で、**SEA-SVR1** への PowerShell リモート処理セッションを表示している **[管理者: Windows PowerShell]** ウィンドウに切り替えます。
1. **SEA-CORE1** のレプリカが **SEA-SVR1** に存在することを確認するには、**SEA-SVR1** への PowerShell リモート処理セッションをホストしている Windows PowerShell ウィンドウで、次のコマンドを入力して Enter キーを押します。

   ```powershell
   Get-VM
   ```

   > **注**:コマンドの出力に **SEA-CORE1** がリストアップされており、レプリケーションが完了していることを確認してください。 レプリケートには 5 分から 10 分かかる場合があります。

   > **注**: Windows PowerShell のセッションは両方とも開いたままにしておきます。

#### <a name="task-3-validate-a-failover"></a>タスク 3: フェールオーバーを検証する

1. **SEA-SVR2** で、ローカルの PowerShell セッションを表示している **[管理者: Windows PowerShell]** ウィンドウに切り替えます。
1. **SEA-CORE1** 仮想マシンを **SEA-SVR1** にフェールオーバーする準備をするには、**SEA-SVR2** 上で、ローカル セッションをホストしている Windows PowerShell ウィンドウで、次のコマンドを入力して Enter キーを押します。

   ```powershell
   Start-VMFailover -Prepare -VMName SEA-CORE1 -ComputerName SEA-SVR2.contoso.com
   ```

   > **注**: メッセージが表示されたら、「**Y**」と入力し、Enter キーを押します。 このコマンドでは、保留中の変更のレプリケーションをトリガーすることによって、**SEA-CORE1** の計画フェールオーバーを準備します。

1. **SEA-SVR2** で、**SEA-SVR1** への PowerShell リモート処理セッションを表示している **[管理者: Windows PowerShell]** ウィンドウに切り替えます。
1. **SEA-CORE1** 仮想マシンの **SEA-SVR1** へのフェールオーバーを開始するには、**SEA-SVR2** 上で、**SEA-SVR1** への PowerShell リモート処理セッションをホストしている Windows PowerShell ウィンドウで、次のコマンドを入力して Enter キーを押します。

   ```powershell
   Start-VMFailover -VMName SEA-CORE1 -ComputerName SEA-SVR1.contoso.com
   ```

   > **注**: メッセージが表示されたら、「**Y**」と入力し、Enter キーを押します。

1. レプリカ VM を プライマリ VM として構成するには、**SEA-SVR2** 上で、**SEA-SVR1** への PowerShell リモート処理セッションを ホストしている Windows PowerShell ウィンドウで、次のコマンドを入力して Enter キーを押します。

   ```powershell
   Set-VMReplication -Reverse -VMName SEA-CORE1 -ComputerName SEA-SVR1.contoso.com
   ```

1. **SEA-SVR1** で新しく指定したプライマリ VM を起動するには、**SEA-SVR2** 上で、**SEA-SVR1** への PowerShell リモート処理セッションをホストしている Windows PowerShell ウィンドウで、次のコマンドを入力して Enter キーを押します。

   ```powershell
   Start-VM -VMName SEA-CORE1 -ComputerName SEA-SVR1.contoso.com
   ```

1. VM が正常に起動されたことを確認するには、**SEA-SVR2** の、**SEA-SVR1** への PowerShell リモート処理セッションをホストしている Windows PowerShell ウィンドウで、次のコマンドを入力して Enter キーを押します。

   ```powershell
   Get-VM
   ```

   > **注**: 結果テーブルで、**状態** が **[実行中]** と表示されていることを確認します。

1. **SEA-SVR1** から **SEA-SVR2** への仮想マシン **SEA-CORE1** のレプリケーションの状態を特定するには、**SEA-SVR2** 上で、**SEA-SVR1** への PowerShell リモート処理セッションをホストする Windows PowerShell ウィンドウで、次のコマンドを入力して Enter キーを押します。

   ```powershell
   Get-VMReplication
   ```

   > **注**: コマンドの出力で、**状態** の値を特定し、その値が **[レプリケート中]** として表示されていることを確認します。 また、**プライマリサーバー**が **SEA-SVR1** として表示され、**ReplicaServer** が **SEA-SVR2** として表示されていることを確認します。

1. プライマリ サーバーで VM のレプリケートを停止するには、**SEA-SVR2** 上で、**SEA-SVR1** への PowerShell リモート処理セッションを ホストしている Windows PowerShell ウィンドウで、次のコマンドを入力して Enter キーを押します。

   ```powershell
   Stop-VM -VMName SEA-CORE1
   ```

1. 両方の Windows PowerShell ウィンドウを開いたままにします。

   > **注**: グラフィカル ツールを使用してこの演習の結果を確認する場合は、**SEA-SVR2** 上で Hyper-V マネージャーを使用してから、**SEA-SVR1** および **SEA-SVR2** サーバーを **Hyper-V** コンソールに追加します。 その後、**SEA-CORE1** VM が **SEA-SVR1** と **SEA-SVR2** の両方に存在し、**SEA-SVR2** から **SEA-SVR1** へレプリケーションが実行されていることを確認できます。

## <a name="exercise-2-implementing-backup-and-restore-with-windows-server-backup"></a>演習 2: Windows Server Backup を使用したバックアップと復元の実装

### <a name="task1-configure-windows-server-backup-settings"></a>タスク 1: Windows Server Backup 設定を構成する

1. **SEA-SVR2** で、タスクバーで **[ファイル エクスプローラー]** アイコンを選択してファイル エクスプローラーを開きます。
1. ファイル エクスプローラーの、**ナビゲーション** ウィンドウで **[ローカル ディスク (C:)]** を選択します。
1. 詳細ウィンドウの空の領域を右クリックするかコンテキスト メニューを開き、**[新規]** を選択して **[フォルダー]** を選択します。 
1. フォルダーに **BackupShare** という名前を付けます。 右クリックするか、**BackupShare** フォルダーのコンテキスト メニューにアクセスし、** [アクセスを許可する]** を選択して、**[特定のユーザー]** を選択します。
1. **[ネットワーク アクセス]** ウィンドウで「**Authenticated Users**」と入力し、**[追加]** を選択します。 **[アクセス許可のレベル]** 列で **[Authenticated Users]** の値を **[読み取り/書き込み]** に設定して **[共有]** を選択し、**[完了]** を選択します。
1. **SEA-SVR2** で、**SEA-SVR1** への PowerShell リモート処理セッションを表示している **[管理者: Windows PowerShell]** ウィンドウに切り替えます。 
1. **SEA-SVR1** に **Windows Server Backup** ロールをインストールするには、**SEA-SVR2** で、**SEA-SVR1** への PowerShell リモート処理セッションをホストしている Windows PowerShell ウィンドウで、次のコマンドを入力して Enter キーを押します。

   ```powershell
   Install-WindowsFeature Windows-Server-Backup
   ```

   > **注**: インストールが完了するまで待ちます。

1. **wbadmin** コマンドライン ユーティリティの機能を確認するには、**SEA-SVR2** で、**SEA-SVR1** への PowerShell リモート処理セッションをホストしている Windows PowerShell ウィンドウで、次のコマンドを入力して Enter キーを押します。

   ```powershell
   wbadmin /?
   ```

1. **WindowsServerBackup** モジュールに含まれている Windows PowerShell コマンドレットの機能を確認するには、**SEA-SVR2** で、**SEA-SVR1** への PowerShell リモート処理セッションをホストしている Windows PowerShell ウィンドウで、次のコマンドを入力して Enter キーを押します。

   ```powershell
   Get-Command -Module WindowsServerBackup -CommandType Cmdlet
   ```

#### <a name="task-2-perform-a-backup-to-a-network-share"></a>タスク 2: ネットワーク共有へのバックアップを実行する

1. **SEA-SVR1** にバックアップするフォルダーとファイルを作成するには、**SEA-SVR2** で、**SEA-SVR1** への PowerShell リモート処理セッションをホストしている Windows PowerShell ウィンドウで、次のコマンドを入力し、各コマンドを入力した後、Enter キーを押します。

   ```powershell
   New-Item -ItemType Directory -Path 'C:\Files' -Force
   fsutil file createnew C:\Files\report1.docx 25432108
   fsutil file createnew C:\Files\report2.docx 25432108
   fsutil file createnew C:\Files\report3.docx 25432108
   fsutil file createnew C:\Files\report4.docx 25432108
   ```

1. Windows Server バックアップを使用し、バックアップ ポリシーの変数と、バックアップするファイル パスを定義するには、**SEA-SVR 2** で、**SEA-SVR1** への PowerShell リモート処理セッションをホストしている Windows PowerShell ウィンドウで、次のコマンドを入力し、各コマンドを入力した後、Enter キーを押します。

   ```powershell
   $policy = New-WBPolicy
   $fileSpec = New-WBFileSpec -FileSpec 'C:\Files'
   ```

1. 前の手順で定義した変数を参照する Windows Server Backup ポリシーを定義するには、**SEA-SVR2** で、**SEA-SVR1** への PowerShell リモート処理セッションをホストする Windows PowerShell ウィンドウで、次のコマンドを入力して Enter キーを押します。

   ```powershell
   Add-WBFileSpec -Policy $policy -FileSpec $fileSpec
   ```

1. 前のタスクで作成したネットワーク共有を使用し、**SEA-SVR2** でバックアップの場所を構成するには、**SEA-SVR2** で、**SEA-SVR1** への PowerShell リモート処理セッションをホストする Windows PowerShell ウィンドウで、次のコマンドを入力し、各コマンドを入力した後、Enter キーを押します (サイン インを求めるメッセージが表示されたら、**CONTOSO\\Administrator** ユーザー名と **Pa55w.rd** パスワードを入力します)。

   ```powershell
   $cred = Get-Credential
   $networkBackupLocation = New-WBBackupTarget -NetworkPath "\\SEA-SVR2.contoso.com\BackupShare" -Credential $cred
   ```

1. バックアップ ポリシーにバックアップの場所を追加するには、**SEA-SVR1** への PowerShell リモート処理セッションをホストしている Windows PowerShell ウィンドウで、次のコマンドを入力して Enter キーを押します。

   ```powershell
   Add-WBBackupTarget -Policy $policy -Target $networkBackupLocation
   ```

1. ボリューム シャドウ コピー サービスを有効にするには、**SEA-SVR1** への PowerShell リモート処理セッションをホストしている Windows PowerShell ウィンドウで、次のコマンドを入力して Enter キーを押します。

   ```powershell
   Set-WBVssBackupOptions -Policy $policy -VssCopyBackup
   ```

1. バックアップ ジョブを開始するには、**SEA-SVR1** への PowerShell リモート処理セッションをホストしている Windows PowerShell ウィンドウで、次のコマンドを入力して Enter キーを押します。

   ```powershell
   Start-WBBackup -Policy $policy
   ```

   > **注**: バックアップが完了するまで待ちます。 これには 1 分ほどかかります。

1. **SEA-SVR2** でファイル エクスプローラー に切り替えて **C:\\BackupShare** に移動し、フォルダーの **WindowsImageBackup** サブフォルダーに新しく作成されたバックアップが含まれることを確認します。

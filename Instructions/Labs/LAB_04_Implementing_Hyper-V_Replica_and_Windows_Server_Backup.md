---
lab:
  title: 'ラボ: Hyper-V レプリカおよび Windows Server バックアップの実装'
  module: 'Module 4: Disaster Recovery in Windows Server'
---

# ラボ: Hyper-V レプリカおよび Windows Server バックアップの実装

## シナリオ

あなたは、Contoso 社で管理者として作業しています。 Contoso 社は、新しいディザスター リカバリーおよびバックアップの機能とテクノロジを評価して構成したいと考えています。 システム管理者として、その評価と実装を担当してきました。 **Hyper-V レプリカ**と Windows Server バックアップを評価することに決定しました。

**メモ:** このラボをご自分のペースでクリックして進めることができる、 **[ラボの対話型シミュレーション](https://mslabs.cloudguides.com/guides/AZ-801%20Lab%20Simulation%20-%20Implementing%20Hyper-V%20Replica%20and%20Windows%20Server%20Backup)** が用意されています。 対話型シミュレーションとホストされたラボの間に若干の違いがある場合がありますが、示されている主要な概念とアイデアは同じです。 

## 目標

このラボを完了すると、次のことができるようになります。

- **Hyper-V レプリカ**を構成して実装する。
- Windows Server バックアップを使用してバックアップを構成し、実装する。

## 予想所要時間: 45 分

## ラボのセットアップ

仮想マシン: **AZ-801T00A-SEA-DC1**、**AZ-801T00A-SEA-SVR1**、**AZ-801T00A-SEA-SVR2** が実行されている必要があります。 他の VM が実行されていてもかまいませんが、このラボでは必要ありません。

> **注**: **AZ-801T00A-SEA-DC1**、**AZ-801T00A-SEA-SVR1**、**AZ-801T00A-SEA-SVR2** 仮想マシンは、**SEA-DC1**、 **SEA-SVR1**、および **SEA-SVR2** のインストールをホストしています

1. **[SEA-SVR2]** を選択します。
1. 次の資格情報を使用してサインインします。

   - ユーザー名: **Administrator**
   - パスワード: **Pa55w.rd**
   - ドメイン: **CONTOSO**

このラボでは、利用可能な VM 環境を使用します。

## 演習 1: Hyper-V レプリカの実装

### シナリオ

クラスターのデプロイを開始する前に、ホスト間で VM をレプリケートするための Hyper-V の新しいテクノロジを評価することにしました。 アクティブなコピーまたはホストで障害が発生した場合に、VM のコピーを別のホストに手動でマウントできるようにする必要があります。

この演習の主なタスクは次のとおりです。

1. Hyper-V レプリカをインストールおよび構成します。
1. Hyper-V レプリケーションを構成します。
1. フェールオーバーを検証します。

#### タスク 1: Hyper-V レプリカをインストールして構成する

1. **SEA-SVR2** で、管理者として Windows PowerShell を開始します。
1. **SEA-SVR2** でセキュリティが強化された **Hyper-V レプリカ HTTP Listener (TCP-In)** 規則を使用する Windows Defenderファイアウォールの状態を確認するには、Windows PowerShell プロンプトで次のコマンドを実行します。

   ```powershell
   Get-NetFirewallRule -DisplayName 'Hyper-V Replica HTTP Listener (TCP-In)'
   ```

1. **SEA-SVR2** でセキュリティが強化された **Hyper-V レプリカ HTTP Listener (TCP-In)** 規則を使用する Windows Defenderファイアウォールを有効にするには、Windows PowerShell プロンプトで次のコマンドを実行します。

   ```powershell
   Enable-NetFirewallRule -DisplayName 'Hyper-V Replica HTTP Listener (TCP-In)'
   ```

1. **Hyper-V レプリカ**のレプリカサーバーとして **SEA-SVR2** を構成するには、次のコマンドを実行します。

   ```powershell
   New-Item -ItemType Directory -Path C:\ReplicaStorage -Force
   Set-VMReplicationServer -ReplicationEnabled $true -AllowedAuthenticationType Kerberos -KerberosAuthenticationPort 8080 -ReplicationAllowedFromAnyServer $true -DefaultStorageLocation C:\ReplicaStorage
   ```

1. **Hyper-V レプリカ**のレプリカサーバーとして **SEA-SVR2** が構成されていることを確認するには、次のコマンドを実行します。

   ```powershell
   Get-VMReplicationServer
   ```
  
   > **注**: コマンドの出力に次の設定が含まれていることを確認してください。

   - **RepEnabled: True**
   - **AuthType: Kerb**
   - **KerAuthPort: 8080**
   - **CertAuthPort: 443**
   - **AllowAnyServer: True**

1. **SEA-SVR2** に存在する仮想マシンを特定するには、Windows PowerShell プロンプトで次のコマンドを実行します。

   ```powershell
   Get-VM
   ```

   > **注**: コマンドの出力に **SEA-CORE1** が含まれていることを確認してください。 

   > **注**: **[管理者: Windows PowerShell]** ウィンドウは開いたままにしておきます。

1. **SEA-SVR2** で、管理者として別の Windows PowerShell ウィンドウを開きます。
1. **SEA-SVR1** への PowerShell リモート処理セッションを確立するには、新しく開いた Windows PowerShell ウィンドウで次のコマンドを入力し、Enter キーを押します。

   ```powershell
   Enter-PSSession -ComputerName SEA-SVR1.contoso.com
   ```
 
   > **注**: Powershell のリモート処理セッションは、この場合は **[SEA-SVR1.contoso.com]** プレフィックスを含む PowerShell プロンプトに基づいて認識できます。

1. **SEA-SVR1** のセキュリティが強化された **Hyper-V レプリカ HTTP Listener (TCP-In)** 規則のある Windows Defender ファイアウォールの状態を特定するには、**SEA-SVR1** への PowerShell リモート処理セッションをホストしている Windows PowerShell ウィンドウで、次のコマンドを実行します。

   ```powershell
   Get-NetFirewallRule -DisplayName 'Hyper-V Replica HTTP Listener (TCP-In)'
   ```

   > **注**: 出力を確認し、 **Enabled** プロパティが **False** に設定されていることを確認します。 **Hyper-V レプリカ**を使用するには、このファイアウォール規則を有効にする必要があります。

1. **SEA-SVR1** のセキュリティが強化された **Hyper-V レプリカ HTTP Listener (TCP-In)** 規則のある Windows Defender ファイアウォールを有効にするには、**SEA-SVR1** への PowerShell リモート処理セッションをホストしている Windows PowerShell ウィンドウで、次のコマンドを実行します。

   ```powershell
   Enable-NetFirewallRule -DisplayName 'Hyper-V Replica HTTP Listener (TCP-In)'
   ```

1. **SEA-SVR1** を **Hyper-V レプリカ**のレプリカ サーバーとして構成するには、**SEA-SVR1** への PowerShell リモート処理セッションをホストしている Windows PowerShell ウィンドウで、次のコマンドを実行します。

   ```powershell
   New-Item -ItemType Directory -Path C:\ReplicaStorage -Force
   Set-VMReplicationServer -ReplicationEnabled $true -AllowedAuthenticationType Kerberos -ReplicationAllowedFromAnyServer $true -DefaultStorageLocation C:\ReplicaStorage
   ```

   > **注**: 2 つ目の **[管理者: Windows PowerShell]** ウィンドウは開いたままにしておきます。

#### タスク 2: Hyper-V レプリケーションを構成する

1. **SEA-SVR2** で、ローカルの PowerShell セッションを表示している **[管理者: Windows PowerShell]** ウィンドウに切り替えます。
1. **SEA-SVR2** から **SEA-SVR1** への仮想マシン **SEA-CORE1** のレプリケーションを有効にするには、**SEA-SVR2** 上のローカルセッションの Windows PowerShell プロンプトで次のコマンドを実行します。

   ```powershell
   Enable-VMReplication SEA-CORE1 -ReplicaServerName SEA-SVR1.contoso.com -ReplicaServerPort 80 -AuthenticationType Kerberos -ComputerName SEA-SVR2.contoso.com
   ```

1. **SEA-SVR2** から **SEA-SVR1** への仮想マシン **SEA-CORE1** のレプリケーションを開始するには、**SEA-SVR2** 上で次のコマンドを実行します。

   ```powershell
   Start-VMInitialReplication SEA-CORE1
   ```

1. **SEA-SVR2** から **SEA-SVR1** への仮想マシン **SEA-CORE1** のレプリケーションの状態が正常に開始されたことを確認するには、次のコマンドを実行します。

   ```powershell
   Get-VMReplication
   ```

   > **注**: コマンドの出力で、 **状態** の値を特定し、その値が **InitialReplicationInProgress** として表示されていることを確認します。 約 5 分間待ってから、同じコマンドを再実行し、**状態** の値が **[レプリケート中]** に変更されたことを確認します。 これが発生するまで待ってから、次の手順に進みます。 また、 **プライマリサーバー**が **SEA-SVR2** として表示され、 **ReplicaServer** が **SEA-SVR1** として表示されていることを確認します。

1. **SEA-SVR2** で、**SEA-SVR1** への PowerShell リモート処理セッションを表示している **[管理者: Windows PowerShell]** ウィンドウに切り替えます。
1. **SEA-CORE1** のレプリカが **SEA-SVR1** に存在することを確認するには、**SEA-SVR1** への PowerShell リモート処理セッションをホストしている Windows PowerShell ウィンドウで、次のコマンドを実行します。

   ```powershell
   Get-VM
   ```

   > **注**: コマンドの出力に **SEA-CORE1** がリストされていることを確認してください。

   > **注**: Windows PowerShell のセッションは両方とも開いたままにしておきます。

##### タスク 3: フェールオーバーを検証する

1. **SEA-SVR2** で、ローカルの PowerShell セッションを表示している **[管理者: Windows PowerShell]** ウィンドウに切り替えます。
1. **SEA-CORE1** 仮想マシンを **SEA-SVR1** にフェールオーバーする準備をするには、**SEA-SVR2** 上の、ローカルセッションをホストしている Windows PowerShell ウィンドウで、次のコマンドを実行します。

   ```powershell
   Start-VMFailover -Prepare -VMName SEA-CORE1 -ComputerName SEA-SVR2.contoso.com
   ```

   > **注**: メッセージが表示されたら、「**Y**」と入力し、Enter キーを押します。 このコマンドは、保留中の変更のレプリケーションをトリガーすることによって、**SEA-CORE1** の計画フェールオーバーを準備します。

1. **SEA-SVR2** で、**SEA-SVR1** への PowerShell リモート処理セッションを表示している **[管理者: Windows PowerShell]** ウィンドウに切り替えます。
1. **SEA-CORE1** 仮想マシンの **SEA-SVR1** へのフェールオーバーを開始するには、**SEA-SVR2** 上の、**SEA-SVR1** への PowerShell リモート処理セッションをホストしている Windows PowerShell ウィンドウで以下のコマンドを実行します。

   ```powershell
   Start-VMFailover -VMName SEA-CORE1 -ComputerName SEA-SVR1.contoso.com
   ```

1. **SEA-SVR2** 上でレプリカ VM を プライマリ VM として構成するには、**SEA-SVR1** への PowerShell リモート処理セッションを ホストしている Windows PowerShell ウィンドウで、次のコマンドを実行します。

   ```powershell
   Set-VMReplication -Reverse -VMName SEA-CORE1 -ComputerName SEA-SVR1.contoso.com
   ```

1. **SEA-SVR1**、**SEA-SVR2** 上で新しく指定したプライマリ VM を起動するには、 **SEA-SVR1** への PowerShell リモート処理セッションをホストしている Windows PowerShell ウィンドウで、次のコマンドを実行します。

   ```powershell
   Start-VM -VMName SEA-CORE1 -ComputerName SEA-SVR1.contoso.com
   ```

1. **SEA-SVR2** で VM が正常に起動されたことを確認するには、**SEA-SVR1** への PowerShell リモート処理セッションをホストしている Windows PowerShell ウィンドウで、次のコマンドを実行します。

   ```powershell
   Get-VM
   ```

   > **注**: 結果テーブルで、 **状態** が **[実行中]** と表示されていることを確認します。

1. **SEA-SVR1** から **SEA-SVR2** への仮想マシン **SEA-CORE1** のレプリケーションの状態を特定するには、**SEA-SVR2** 上の、**SEA-SVR1** への PowerShell リモート処理セッションをホストする Windows PowerShell ウィンドウで、次のコマンドを実行します。

   ```powershell
   Get-VMReplication
   ```

   > **注**: コマンドの出力で、 **状態** の値を特定し、その値が **[レプリケート中]** として表示されていることを確認します。 また、 **プライマリサーバー**が **SEA-SVR1** として表示され、 **ReplicaServer** が **SEA-SVR2** として表示されていることを確認します。

1. **SEA-SVR2** 上のプライマリ サーバーで VM のレプリケートを停止するには、**SEA-SVR1** への PowerShell リモート処理セッションを ホストしている Windows PowerShell ウィンドウで、次のコマンドを実行します。

   ```powershell
   Stop-VM -VMName SEA-CORE1
   ```

1. 両方の Windows PowerShell ウィンドウを開いたままにします。

   > **注**: グラフィカル ツールを使用してこの演習の結果を確認する場合は、**SEA-SVR2** 上で Hyper-V マネージャーを使用してから、**SEA-SVR1** および **SEA-SVR2** サーバーを **Hyper-V** コンソールに追加します。 その後、**SEA-CORE1** VM が **SEA-SVR1** と **SEA-SVR2** の両方に存在し、**SEA-SVR2** から **SEA-SVR1** へレプリケーションが実行されていることを確認できます。

## 演習 2: Windows Server バックアップを使用したバックアップと復元の実装

### シナリオ

メンバーサーバーの Windows Server バックアップを評価する必要があります。 **SEA-SVR2** サーバーの Windows Server バックアップを構成し、 **SEA-SVR2** のネットワーク共有への試用版バックアップを実行することにしました。

この演習の主なタスクは次のとおりです。

1. Windows Server バックアップ設定を構成します。
1. ネットワーク共有へのバックアップを実行します。

#### タスク 1: Windows Server バックアップ設定を構成する

1. **SEA-SVR2** 上で、エクスプローラーを使用して **SEA-SVR2** に **C:\\ BackupShare** フォルダーを作成します。 **認証されたユーザー** が読み取り/書き込みアクセス許可を持つようにフォルダーを共有します。
1. **SEA-SVR2** で、**SEA-SVR1** への PowerShell リモート処理セッションを表示している **[管理者: Windows PowerShell]** ウィンドウに切り替えます。
1. **SEA-SVR1** への PowerShell リモート処理セッションで、 **Install-WindowsFeature** コマンドレットを使用して **Windows-Server-Backup** 機能を **SEA-SVR1** 上にインストールします。
1. **SEA-SVR1** への PowerShell リモート処理セッションで、**wbadmin /?**、 および **Get-Command** コマンドを使用して、 **wbadmin** ユーティリティの機能と **WindowsServerBackup** モジュールのコマンドレットを確認します。

#### タスク 2: ネットワーク共有へのバックアップを実行する

1. **SEA-SVR2** から、**SEA-SVR1** 上にバックアップするフォルダーとファイルを作成します。そのためには、 **SEA-SVR1** への PowerShell リモート処理セッションを使用して次のコマンドを実行します。

   ```powershell
   New-Item -ItemType Directory -Path 'C:\Files' -Force
   fsutil file createnew C:\Files\report1.docx 25432108
   fsutil file createnew C:\Files\report2.docx 25432108
   fsutil file createnew C:\Files\report3.docx 25432108
   fsutil file createnew C:\Files\report4.docx 25432108
   ```

1. Windows Server バックアップを使用してバックアップするバックアップ ポリシーおよびファイル パスの変数を定義するには、**SEA-SVR2** 上の、**SEA-SVR1** への PowerShell リモート処理セッションをホストしている Windows PowerShell ウィンドウで、次のコマンドを実行します。

   ```powershell
   $policy = New-WBPolicy
   $fileSpec = New-WBFileSpec -FileSpec 'C:\Files'
   ```

1. 前の手順で定義した変数を参照する Windows Server バックアップ ポリシーを定義するには、 **SEA-SVR2** で上の、**SEA-SVR1** への PowerShell リモート処理セッションを使用して、次のコマンドを実行します。

   ```powershell
   Add-WBFileSpec -Policy $policy -FileSpec $fileSpec
   ```

1. 前のタスクで作成したネットワーク共有を使用して **SEA-SVR2** 上のバックアップの場所を構成するには、**SEA-SVR2** 上の、**SEA-SVR1** への PowerShell リモート処理セッションを使用して、次のコマンドを実行します (サインインを求めるメッセージが表示されたら、**CONTOSO\\ 管理者** ユーザー名と **Pa55w.rd** パスワードを入力します)。

   ```powershell
   $cred = Get-Credential
   $networkBackupLocation = New-WBBackupTarget -NetworkPath "\\SEA-SVR2.contoso.com\BackupShare" -Credential $cred
   ```

1. バックアップの場所をバックアップ ポリシーに追加するには、**SEA-SVR1** への PowerShell リモート処理セッションを使用して、次のコマンドを実行します。

   ```powershell
   Add-WBBackupTarget -Policy $policy -Target $networkBackupLocation
   ```

1. ボリューム シャドウ コピー サービスを有効にするには、 **SEA-SVR1** への PowerShell リモート処理セッションを使用して、次のコマンドを実行します。

   ```powershell
   Set-WBVssBackupOptions -Policy $policy -VssCopyBackup
   ```

1. バックアップ ジョブを開始するには、 **SEA-SVR1** への PowerShell リモート処理セッションを使用して次のコマンドを実行します。

   ```powershell
   Start-WBBackup -Policy $policy
   ```

   > **注**: バックアップが完了するまで待ちます。 これには 1 分ほどかかります。

1. **SEA-SVR2** で、エクスプローラー に切り替えて **C:\\BackupShare** に移動し、フォルダーの **WindowsImageBackup** サブフォルダーに新しく作成されたバックアップが含まれることを確認します。

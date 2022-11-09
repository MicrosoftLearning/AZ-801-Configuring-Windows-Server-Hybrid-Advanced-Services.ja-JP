---
lab:
  title: 'ラボ: Windows Server の監視とトラブルシューティング'
  module: 'Module 8: Monitoring, performance, and troubleshooting'
---

# <a name="lab-monitoring-and-troubleshooting-windows-server"></a>ラボ: Windows Server の監視とトラブルシューティング

## <a name="scenario"></a>シナリオ

Contoso 社 は、米国ワシントン州シアトルに本社を置くグローバルなエンジニアリングおよび製造企業です。 ITオフィスとデータセンターはシアトルにあり、シアトルやその他のオフィスをサポートしています。 Contoso は最近、Windows サーバーサーバーとクライアント インフラストラクチャをデプロイしました。

組織は新しいサーバーをデプロイしたため、これらの新しいサーバーの一般的な負荷でパフォーマンス ベースラインを確立することが重要です。 このプロジェクトで作業するように求められました。 さらに、監視とトラブルシューティングのプロセスを容易にするために、イベント ログの集中監視を行うことにしました。

                **メモ:** このラボをご自分のペースでクリックして進めることができる、 **[ラボの対話型シミュレーション](https://mslabs.cloudguides.com/guides/AZ-801%20Lab%20Simulation%20-%20Monitoring%20and%20troubleshooting%20Windows%20Server)** が用意されています。 対話型シミュレーションとホストされたラボの間に若干の違いがある場合がありますが、示されている主要な概念とアイデアは同じです。 

## <a name="objectives"></a>目標

このラボを完了すると、次のことができるようになります。

- パフォーマンス ベースラインを確立します。
- パフォーマンス問題の原因を特定する
- 一元管理されたイベントログを確認して構成します。

## <a name="estimated-time-40-minutes"></a>予想所要時間: **40 分**

## <a name="lab-setup"></a>ラボのセットアップ

仮想マシン: **AZ-801T00A-SEA-DC1** と **AZ-801T00A-SEA-SVR2** が実行されている必要があります。 他の VM が実行されていてもかまいませんが、このラボでは必要ありません。

> **注:** **AZ-801T00A-SEA-DC1** および **AZ-801T00A-SEA-SVR2** 仮想マシンは、**SEA-DC1** および **SEA-SVR2** のインストールをホストしています。

1. **[SEA-SVR2]** を選択します。
1. 次の資格情報を使用してサインインします。

   - ユーザー名: **Administrator**
   - パスワード: **Pa55w.rd**
   - ドメイン: **CONTOSO**

このラボでは、利用可能な VM 環境を使用します。

## <a name="exercise-1-establishing-a-performance-baseline"></a>演習 1: パフォーマンス ベースラインの確立

### <a name="scenario"></a>シナリオ

この演習では、サーバーでパフォーマンス モニターを使用し、一般的なパフォーマンス カウンターを使用してベースラインを作成します。

この演習の主なタスクは次のとおりです。

1. データ コレクター セットを作成して開始します。
1. サーバーで一般的なワークロードを作成します。
1. 収集したデータを分析します。

> **注**: データ コレクター セットを開始した後に、結果が表示されるまでに 10 分程の遅延が発生する場合があります。

#### <a name="task-1-create-and-start-a-data-collector-set"></a>タスク 1: データ コレクター セットを作成して開始する

1. **SEA-SVR2** で、パフォーマンス モニターを開きます。
1. パフォーマンス モニターを使用して、次の設定で新しい **ユーザー定義** データ コレクター セットを作成します。

   - 名前: **SEA-SVR2 パフォーマンス**
   - 作成: **手動で作成 (詳細)**
   - データの種類: **パフォーマンス カウンター**
   - パフォーマンス カウンター (既定のインスタンスオプションを使用):

      - **メモリー\\ページ/秒**
      - **Network Interface\\Bytes Total/sec**
      - **PhysicalDisk\\% Disk Time**
      - **PhysicalDisk\\Avg. Disk Queue Length**
      - **Processor\\% Processor Time**
      - **System\\ プロセッサ キューの長さ**

   - サンプル間隔: **1** 秒

1. パフォーマンス モニターで、 **SEA-SVR2 パフォーマンス** データ コレクターを起動します。 

#### <a name="task-2-create-a-typical-workload-on-the-server"></a>タスク 2: サーバーで一般的なワークロードを作成する

1. **SEA-SVR2** で、管理者として Windows PowerShell を開始します。
1. 一般的なワークロードをエミュレートするには、Windows PowerShell コマンド プロンプトで次のコマンドを実行します。


   ```powershell
   fsutil file createnew bigfile 104857600
   Copy-Item -Path .\bigfile -Destination \\SEA-DC1.contoso.com\c$\ -Force
   Copy-Item -Path \\SEA-DC1.contoso.com\c$\bigfile -Destination .\bigfile2 -Force
   Remove-Item -Path .\bigfile* -Force
   Remove-Item -Path \\SEA-DC1.contoso.com\c$\bigfile -Force
   ```

1. Windows PowerShell ウィンドウは開いたままにしておきます。

#### <a name="task-3-analyze-the-collected-data"></a>タスク 3: 収集したデータを分析する

1. **SEA-SVR2** で、パフォーマンス モニターに切り替えます。
1. **SEA-SVR2 パフォーマンス** データ コレクター セットを停止します。
1. パフォーマンス モニターのナビゲーション ウィンドウで、 **[レポート]** 、 **[ユーザー定義]** 、 **[SEA-SVR2]** 、 **[SEA-SVR2] \_ [DateTime-000001]** の順に移動し、レポート ビューを使用してレポート データを確認します。
1. レポートに一覧表示される値を記録して、後で分析できるようにします。 記録される値は次のとおりです。

   - **メモリー\\ページ/秒**
   - **Network Interface\\Bytes Total/sec**
   - **PhysicalDisk\\% Disk Time**
   - **PhysicalDisk\\Avg. Disk Queue Length**
   - **Processor\\% Processor Time**
   - **System\\ プロセッサ キューの長さ**

### <a name="results"></a>結果

この演習の後、パフォーマンス比較のためのベースラインを確立しておく必要があります。

## <a name="exercise-2-identifying-the-source-of-a-performance-problem"></a>演習 2: パフォーマンス問題の原因の特定

### <a name="scenario"></a>シナリオ

この演習では、実際の使用状況のシステムを表す負荷をシミュレートし、データ コレクター セットを使用してパフォーマンス データを収集し、パフォーマンスの問題の潜在的な原因を特定します。

この演習の主なタスクは次のとおりです。

1. サーバーで追加のワークロードを作成します。
1. データ コレクター セットを使用してパフォーマンス データをキャプチャします。
1. ワークロードを削除し、パフォーマンス データを確認します。

#### <a name="task-1-create-additional-workload-on-the-server"></a>タスク 1: サーバーで追加のワークロードを作成する

1. **SEA-SVR2** で、エクスプローラーを開きます。
1. エクスプローラーで、 **C:\Labfiles\Lab08** に移動し、 **CPUSTRES64** を起動します。
1. 最初に強調表示されたタスクが **ビジー状態 (75%)** で実行するように構成します。

   > **注**: **CPUSTRES64.EXE** は、ループ内で最大 64 個のスレッドを実行して CPU アクティビティをシミュレートするのに使用できる SysInternals ユーティリティです。

#### <a name="task-2-capture-performance-data-by-using-a-data-collector-set"></a>タスク 2: データ コレクター セットを使用してパフォーマンス データをキャプチャする

1. **SEA-SVR2** で、パフォーマンス モニターに切り替えます。
1. パフォーマンス モニターで、 **[データ コレクター セット]** 、 **[ユーザー定義]** の順に移動し、結果ウィンドウで **SEA-SVR2 パフォーマンス** データ コレクター セットを開始します。

   > **注**: データ キャプチャが行われるように 1 分待ってください。

#### <a name="task-3-remove-the-workload-and-review-the-performance-data"></a>タスク 3: ワークロードを削除し、パフォーマンス データを確認する

1. **SEA-SVR2** で、CPUSTRES64 を閉じます。
1. パフォーマンス モニターに切り替えます。
1. **SEA-SVR2 パフォーマンス** データ コレクター セットを停止します。
1. パフォーマンス モニターのナビゲーション ウィンドウで、 **[レポート]** 、 **[ユーザー定義]** 、 **[SEA-SVR2]** 、 **[SEA-SVR2] \_ [DateTime-000002]** の順に移動し、レポートを確認します。 
1. レポートに一覧表示される値を記録します。

   - **メモリー\\ページ/秒**
   - **Network Interface\\Bytes Total/sec**
   - **PhysicalDisk\\% Disk Time**
   - **PhysicalDisk\\Avg. Disk Queue Length**
   - **Processor\\% Processor Time**
   - **System\\ プロセッサ キューの長さ**

### <a name="results"></a>結果

この演習の後、パフォーマンス ツールを使用して潜在的なパフォーマンスのボトルネックを特定しておく必要があります。

### <a name="exercise-3-viewing-and-configuring-centralized-event-logs"></a>演習 3: 一元化されたイベント ログの表示と構成

### <a name="scenario"></a>シナリオ

この演習では、**SEA-DC1** からアプリケーション イベント ログエントリを収集するために **SEA-SVR2** を使用します。 

この演習の主なタスクは次のとおりです。

1. サブスクリプションの前提条件を構成します。
1. サブスクリプションを作成し、結果を確認します。

#### <a name="task-1-configure-subscription-prerequisites"></a>タスク 1: サブスクリプションの前提条件を構成する

1. **SEA-SVR2** で、Windows PowerShell に切り替えます。
1. **SEA-SVR2** に転送されるイベントのサブスクリプションの作成と管理を有効にするには、次のコマンドを実行します。

   ```powershell
   WECUtil qc /q
   ```

1. イベント ソースとコレクターのローカルの日付と時刻が同期されていることを確認するには、次のコマンドを実行します。

   ```powershell
   w32tm /resync /computer:SEA-DC1.contoso.com
   ```

1. Kerberos 認証の問題が発生した場合に WinRM の接続を許可するには、次のコマンドを実行します。

   ```powershell
   Set-Item WSMan:localhost\client\trustedhosts -Value *.contoso.com -Force
   ```

1. **SEA-DC1** への PowerShell リモート処理セッションを確立するには、次のコマンドを実行します。

   ```powershell
   Enter-PSSession -ComputerName SEA-DC1.contoso.com
   ```

1. **SEA-DC1** で Windows リモート管理 (WinRM) が有効になっていることを確認するには、次のコマンドを実行します。

   ```powershell
   winrm qc
   ```

   > **注**: WinRM サービスが既に実行されていて、リモート管理用に設定されていることを確認してください。

1. **SEA-DC1**で、高度なセキュリティ規則を備えた関連する Windows Defender ファイアウォールが有効になっていることを確認するには、次のコマンドを実行します。

   ```powershell
   Set-NetFirewallRule -DisplayGroup 'Remote Event Log Management' -Enabled True -Profile Domain -PassThru
   ```

   > **注**: Windows PowerShell ウィンドウは開いたままにしておきます。

1. **SEA-SVR2** 上で、 **[Active Directory ユーザーとコンピューター]** を開きます。
1. **[Active Directory ユーザーとコンピューター]** コンソールで、**SEA-SVR2** コンピューター アカウントを **イベント ログ閲覧者**グループに追加します。

#### <a name="task-2-create-a-subscription-and-verify-the-results"></a>タスク 2: サブスクリプションを作成し、結果を確認する

1. **SEA-SVR2** 上で **イベント ビューアー**を開きます。
1. 次のプロパティを使用して、新しいサブスクリプションを作成します。

   - 名前: **SEA-DC1 イベント**
   - コンピューター: **SEA-DC1**
   - コレクター: **開始済み**
   - イベント: **クリティカル**、**警告**、**情報**、**エラー**
   - ログ記録済み: **過去 24 時間**
   - ログ:**アプリケーション**と**システム**の **Windows ログ**

1. **SEA-SVR2** で、 **[イベント ビューアー]** ウィンドウに切り替え、ナビゲーションウィンドウで **[Windows のログ]** を展開します。
1. **[転送済みイベント]** を選択し、転送されたイベントに、**SEA-DC1** 上で生成されたイベントが含まれることを確認します。

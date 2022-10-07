---
lab:
  title: 'ラボ: Windows Server でのセキュリティの構成'
  type: Answer Key
  module: 'Module 1: Windows Server security'
---

# <a name="lab-answer-key-configuring-security-in-windows-server"></a>ラボの解答集: Windows Server でのセキュリティの構成

## <a name="exercise-1-configuring-windows-defender-credential-guard"></a>演習 1: Windows Defender Credential Guard の構成

> **注**: ラボ環境では、前提条件を満たしていないため、Credential Guard は VM で使用できません。 ただし、このことが、グループ ポリシーを使用して実装し、対応するツールを使用してその準備状況を評価するというプロセスを実行することの妨げになることはありません。

#### <a name="task-1-enable-windows-defender-credential-guard-using-group-policy"></a>タスク 1: グループ ポリシーを使用して Windows Defender Credential Guard を有効にする

1. **SEA-SVR2** に接続し、必要であればパスワード **Pa55w.rd** を使用して **CONTOSO\\Administrator** としてサインインします。
1. **[スタート]** ボタンの横にある **[ここに入力して検索]** テキスト ボックスに、「**グループ ポリシーの管理**」と入力します。
1. 結果の一覧で **[グループ ポリシーの管理]** を選択します。
1. **グループ ポリシー管理コンソール**で、 **[フォレスト: contoso.com]** を展開し、 **[ドメイン]** を展開し、**contoso.com** を展開します。 **[IT]** 組織単位 (OU) を右クリックして**コンテキスト** メニューにアクセスして、 **[このドメインに GPO を作成し、このコンテナーにリンクする]** を選択します。
1. **[新しい GPO]** ダイアログ ボックスの **[名前]** テキスト ボックスに「**CredentialGuard_GPO**」と入力し、**[OK]** をクリックします。
1. **[グループ ポリシーの管理]** ウィンドウの **[IT]** の下にある **[CredentialGuard_GPO]** を右クリックして**コンテキスト** メニューにアクセスして、 **[編集]** を選択します。
1. **グループ ポリシー管理エディター**で、 **[コンピューターの構成]\\[ポリシー]\\[管理用テンプレート]\\[システム]\\[Device Guard]** を参照します。
1. **[仮想化ベースのセキュリティを有効にする]** を選択し、 **[ポリシー設定]** リンクを選択します。
1. **[仮想化ベースのセキュリティを有効にする]** ウィンドウで、**[有効]** オプションを選択します。
1. **[プラットフォームのセキュリティ レベルを選択する]** ドロップダウン リストで、**[Secure Boot and DMA Protection]\(セキュア ブートと DMA 保護)\** というエントリが選択されている必要があります。
1. **[Credential Guard の構成]** ドロップダウン リストで、**[UEFI ロックで有効化]** というエントリを選択します。
1. **[Secure Launch Configuration]\(セキュア起動の構成\)** ドロップダウン リストで、**[有効]** のエントリ を選択し、**[OK]** を選択します。
1. **[グループ ポリシー管理エディター]** ウィンドウを閉じます。
1. **[グループ ポリシー管理]** コンソール ウィンドウを閉じます。

#### <a name="task-2-enable-windows-defender-credential-guard-using-the-hypervisor-protected-code-integrity-hvci-and-windows-defender-credential-guard-hardware-readiness-tool"></a>タスク 2: Hypervisor-Protected Code Integrity (HVCI) と Windows Defender Credential Guard ハードウェア準備ツールを使用して Windows Defender Credential Guard を有効にする

1. **SEA-SVR2** で、 **[スタート]** を選択し、**Windows PowerShell** を右クリックして**コンテキスト** メニューにアクセスして、 **[管理者として実行]** を選択します。
1. HVCI と Windows Defender Credential Guard ハードウェア準備ツールを実行するには、Windows PowerShell コマンド プロンプトで次のコマンドを入力し、最初のプロンプトでは **[[R] Run once]** を選択し、残りのプロンプトでは Enter キーを押します。

   ```powershell
   Set-Location -Path C:\Labfiles\Lab01\
   .\DG_Readiness_Tool.ps1 -Enable -AutoReboot
   ```

1. ツールの実行が完了するまで待ち、メッセージが表示されたら、 **[You're about to be signed out]\(サインアウトしようとしています\)** ダイアログ ボックスで **[OK]** を選択します。

   > **注**: オペレーティング システムが再起動します。 

1. 再起動が完了したら、**SEA-SVR2** に **CONTOSO\\Administrator** としてサインインし直します。パスワード **Pa55w.rd** を使用します。

## <a name="exercise-2-locating-problematic-accounts"></a>演習 2: 問題のあるアカウントの検出

#### <a name="task-1-locate-and-reconfigure-domain-accounts-with-non-expiring-passwords"></a>タスク 1: 無期限のパスワードを持つドメイン アカウントを検出して再構成する

1. **SEA-SVR2** で、 **[スタート]** を選択し、**Windows PowerShell** を右クリックして**コンテキスト** メニューにアクセスして、 **[管理者として実行]** を選択します。
1. 無期限のパスワードを持つ Active Directory 対応ユーザー アカウントを一覧表示するために、Windows PowerShell コマンド プロンプトで次のコマンドを入力し、Enter キーを押します。

   ```powershell
   Get-ADUser -Filter {Enabled -eq $true -and PasswordNeverExpires -eq $true}
   ```

1. 返されたユーザー アカウントの一覧を確認します。
1. 結果セット内のすべてのユーザー アカウントに対してパスワードの有効期限を有効にするために、Windows PowerShell コマンド プロンプトで次のコマンドを入力し、Enter キーを押します。

   ```powershell
   Get-ADUser -Filter {Enabled -eq $true -and PasswordNeverExpires -eq $true} | Set-ADUser -PasswordNeverExpires $false
   ```

1. 結果を確認するために、手順 2. のコマンドを再実行します。結果が返されないことに注目してください。

#### <a name="task-2-locate-and-disable-domain-accounts-that-have-not-been-used-to-sign-in-for-at-least-90-days"></a>タスク 2: 少なくとも 90 日間はサインインに使用されていないドメイン アカウントを検出して無効にする

1. 少なくとも 90 日間はサインインに使用されていない Active Directory ユーザー アカウントを特定するために、Windows PowerShell コマンド プロンプトで次のコマンドを入力し、Enter キーを押します。

   ```powershell
   $days = (Get-Date).AddDays(-90)
   Get-ADUser -Filter {LastLogonTimeStamp -lt $days -and enabled -eq $true} -Properties LastLogonTimeStamp
   ```

   > **注**: このラボ環境では、結果は返されません。

1. 少なくとも 90 日間はサインインに使用されていない Active Directory ユーザー アカウントを無効にするために、次のコマンドを入力して Enter キーを押します。

   ```powershell
   Get-ADUser -Filter {LastLogonTimeStamp -lt $days -and enabled -eq $true} -Properties LastLogonTimeStamp | Disable-ADAccount
   ```

   > **注**: このラボ環境では、結果は返されません。

## <a name="exercise-3-implementing-laps"></a>演習 3: LAPS の実装

#### <a name="task-1-prepare-computer-accounts-for-implementing-laps-local-administrator-password-solution"></a>タスク 1: LAPS (ローカル管理者パスワード ソリューション) の実装用にコンピューター アカウントを準備する

1. 指定の OU を作成し、**SEA-SVR1** コンピューター アカウントをその OU に移動するために、**SEA-SVR2** の Windows PowerShell コマンド プロンプトで次のコマンドを入力し、Enter キーを押します。 

   ```powershell
   New-ADOrganizationalUnit -Name "Seattle_Servers"
   Get-ADComputer SEA-SVR1 | Move-ADObject –TargetPath "OU=Seattle_Servers,DC=Contoso,DC=com"
   ```

1. LAPS をインストールするために、Windows PowerShell コマンド プロンプトで次のコマンドを入力し、Enter キーを押します。

   ```powershell
   Msiexec /i C:\Labfiles\Lab01\LAPS.x64.msi
   ```

1. **Local Administrator Password Solution Setup** ウィザードの **[Welcome to the Local Administrator Password Solution Setup Wizard]** ページで **[Next]** を選択します。
1. **Local Administrator Password Solution Setup** ウィザードの **[End-User License Agreement]** ページで、**[I accept the terms in the License Agreement]** を選択し、**[Next]** を選択します。
1. **Local Administrator Password Solution Setup** ウィザードの **[Custom Setup]** ページで、**[Management Tools]** の横にあるドロップダウン メニューで **[Entire feature will be installed on the local hard drive]** を選択し、**[Next]** を選択します。
1. **Local Administrator Password Solution Setup** ウィザードの **[Ready to install Local Administrator Password Solution]** ページで **[Next]** を選択します。 
1. インストールが完了したら、**Local Administrator Password Solution Setup** ウィザードの最後のページで **[Finish]** を選択します。
1. Windows Defender ファイアウォールで他のドメイン参加サーバーからの受信サーバー メッセージ ブロック (SMB) 接続を許可する高度なセキュリティ規則を有効にするために、Windows PowerShell コマンド プロンプトで次のコマンドを入力し、それぞれの後で Enter キーを押します。

   ```
   $rule = Get-NetFirewallRule | Where-Object DisplayName -eq 'File and Printer Sharing (SMB-In)' 
   $rule | Set-NetFirewallRule -Profile Domain
   $rule | Enable-NetFirewallRule
   ```

   > **注**: これは、このラボで後ほど **SEA-SVR1** から **SEA-SVR2** に接続するために必要です。

#### <a name="task-2-prepare-ad-ds-for-laps"></a>タスク 2: LAPS 用に AD DS を準備する

1. LAPS 用にドメインを準備するために、**SEA-SVR2** の Windows PowerShell コマンド プロンプトで次のコマンドを入力し、それぞれの後に Enter キーを押します。

   ```powershell
   Import-Module admpwd.ps
   Update-AdmPwdADSchema
   Set-AdmPwdComputerSelfPermission -Identity "Seattle_Servers"
   ```

1. **SEA-SVR2** で、**[スタート]** ボタンの横にある **[ここに入力して検索]** テキスト ボックスに、「**グループ ポリシーの管理**」と入力します。
1. 結果の一覧で **[グループ ポリシーの管理]** を選択します。
1. **グループ ポリシー管理コンソール**で、 **[フォレスト: contoso.com]** を展開し、 **[ドメイン]** を展開し、**contoso.com** を展開します。**Seattle_Servers** OU を右クリックして**コンテキスト** メニューにアクセスして、 **[このドメインに GPO を作成し、このコンテナーにリンクする]** を選択します。
1. **[新しい GPO]** ダイアログ ボックスの **[名前]** テキスト ボックスに「**LAPS_GPO**」と入力し、**[OK]** をクリックします。
1. **[グループ ポリシーの管理]** ウィンドウの **[Seattle_Servers]** の下にある **[LAPS_GPO]** を右クリックして**コンテキスト** メニューにアクセスして、**[編集]** を選択します。
1. **[グループ ポリシー管理エディター]** ウィンドウの **[コンピューターの構成]** で、**[ポリシー]** ノードを展開し、**[管理用テンプレート]** ノードを展開して、**[LAPS]** を選択します。
1. **[ローカル管理者のパスワード管理を有効にする]** ポリシーを選択し、**[ポリシー設定]** リンクを選択します。
1. **[ローカル管理者のパスワード管理を有効にする]** ウィンドウで、**[有効]** を選択し、**[OK]** を選択します。
1. **[パスワードの設定]** を選択し、**[ポリシー設定]** リンクを選択します。
1. **[パスワードの設定]** ポリシー ダイアログ ボックスで、**[有効]** を選択し、**[パスワードの長さ]** を **[20]** に構成します。
1. **[Password Age (Days)]\(パスワードの有効期間 (日数)\)** が **[30]** に構成されていることを確認し、**[OK]** を選択します。
1. グループ ポリシー管理エディターを閉じます。

#### <a name="task-3-deploy-laps-client-side-extension"></a>タスク 3: LAPS クライアント側拡張をデプロイする

1. **SEA-SVR1** のコンソール セッションに切り替え、必要に応じて、**Pa55w.rd** のパスワードを使用して **CONTOSO\\Administrator** としてサインインします。

   > **注:** パスワードの有効期限を有効にするスクリプトを前の演習で実行した結果として、パスワードを変更するように求められます。 任意のパスワードを選択し、ラボの残りの部分で使用します。

1. サインインしたら、Windows PowerShell コマンド プロンプトにアクセスするために、**SConfig** メニュー プロンプトで「**15**」と入力して、Enter キーを押します。
1. 既定の設定を使用して LAPS をサイレント モードでインストールするために、Windows PowerShell コマンド プロンプトで次のコマンドを入力し、Enter キーを押します。

   ```powershell
   Start-Process msiexec.exe -Wait -ArgumentList '/i \\SEA-SVR2.contoso.com\c$\Labfiles\Lab01\LAPS.x64.msi /quiet'
   ```

1. **LAPS** 設定をローカルに適用するグループ ポリシーの処理をトリガーするために、Windows PowerShell コマンド プロンプトで次のコマンドを入力し、Enter キーを押します。

   ```powershell
   gpupdate /force
   ```

#### <a name="task-4-verify-laps"></a>タスク 4: LAPS を確認する

1. **SEA-SVR2** のコンソール セッションに切り替えます。
1. **[スタート]** を選択します。 **[スタート]** メニューで **[LAPS]** を選択し、 **[LAPS UI]** を選択します。
1. **[LAPS UI]** ダイアログ ボックスの **[コンピューター名]** ボックスに「**SEA-SVR1**」と入力して、 **[検索]** を選択します。
1. **[パスワード]** と **[パスワードの有効期限]** の値を確認し、 **[終了]** を選択します。
1. **Windows PowerShell** コンソールに切り替えてから、パスワードの値を確認するために、Windows PowerShell コマンド プロンプトで次のコマンドを入力し、Enter キーを押します。

   ```powershell
   Get-ADComputer -Identity SEA-SVR1 -Properties ms-Mcs-AdmPwd
   ```

1. SEA-SVR1 に割り当てられているパスワードを確認し、**LAPS UI** ツールに表示されていたものと一致していることに注目します。

   > **注:** この場合、パスワードの値は中かっこのペアで囲まれています。

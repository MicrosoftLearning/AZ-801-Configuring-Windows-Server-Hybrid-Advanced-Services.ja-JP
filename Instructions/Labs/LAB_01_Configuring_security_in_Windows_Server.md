---
lab:
  title: 'ラボ: Windows Server でのセキュリティの構成'
  module: 'Module 1: Windows Server security'
---

# ラボ: Windows Server でのセキュリティの構成

この演習の所要時間は約 **40** 分です。 <!-- update with estimated duration -->

## シナリオ

Contoso 製薬は、世界各地に 5,000 人の従業員を抱える医療系リサーチ会社です。 医療記録とデータが公開されないようにするという特定のニーズがあります。 この会社は本社の拠点と世界の複数の拠点を有しています。 Contoso では最近、Windows Server サーバーとクライアント インフラストラクチャをデプロイしました。 あなたは、サーバーのセキュリティ構成に改善を実装するように求められました。

>**注**: 以前提供されていたラボ シミュレーションは廃止されました。

## 目標

このラボを完了すると、次のことができるようになります。

- Windows Defender Credential Guard を構成する。
- 問題のあるユーザーアカウントを検出する。
- LAPS (ローカル管理者パスワード ソリューション) を実装して検証する。

## 予想所要時間: 40 分

## ラボのセットアップ

仮想マシン: **AZ-801T00A-SEA-DC1**、**AZ-801T00A-SEA-SVR1**、**AZ-801T00A-SEA-SVR2** が実行されている必要があります。 他の VM が実行されていてもかまいませんが、このラボでは必要ありません。

> **注**: **AZ-801T00A-SEA-DC1**、**AZ-801T00A-SEA-SVR1**、**AZ-801T00A-SEA-SVR2** の各仮想マシンは、**SEA-DC1**、 **SEA-SVR1**、および **SEA-SVR2** のインストールをホストしています。

1. **[SEA-SVR2]** を選択します。
1. 講師から提供された資格情報を使用してサインインします。

このラボでは、利用可能な VM 環境を使用します。

## 演習 1: Windows Defender Credential Guard の構成

### シナリオ

Windows Defender Credential Guard をサーバーと管理ワークステーションに実装して、Pass-the-Hash と Pass-the-Ticket の資格情報盗難から保護することにしました。 グループ ポリシーを使用して、既存のサーバーで Credential Guard を有効にします。 すべての新しいサーバーに対して Hypervisor-Protected Code Integrity (HVCI) と Windows Defender Credential Guard ハードウェア準備ツールを使用して、それらの新しいサーバーがドメインに参加する前に Credential Guard を有効にします。

このラボでは、グループ ポリシーを設定し、既存のサーバーで HVCI および Windows Defender Credential Guard ハードウェア準備ツールを実行します。

> **注**: ラボ環境では、前提条件を満たしていないため、Credential Guard は VM で使用できません。 ただし、このことが、グループ ポリシーを使用して実装し、対応するツールを使用してその準備状況を評価するというプロセスを実行することの妨げになることはありません。

この演習の主なタスクは次のとおりです。

1. グループ ポリシーを使用して Windows Defender Credential Guard を有効にする。
1. HVCI と Windows Defender Credential Guard ハードウェア準備ツールを使用して Windows Defender Credential Guard を有効にする。

#### タスク 1: グループ ポリシーを使用して Windows Defender Credential Guard を有効にする

1. **SEA-SVR2** で、 **[グループ ポリシー管理]** コンソールを開きます。
1. **[グループ ポリシー管理]** コンソールで、 **[フォレスト: contoso.com]** 、 **[ドメイン]** 、**contoso.com** の順に参照し、**IT** 組織単位 (OU) にリンクされた **CredentialGuard_GPO** という名前のグループ ポリシー オブジェクト (GPO) を作成します。
1. グループポリシー管理エディターで **CredentialGuard_GPO** を開き、 **[コンピューターの構成]\\[ポリシー]\\[管理用テンプレート]\\[システム]\\[デバイスの保護]** ノード を参照します。
1. 次の設定を使用して、**[仮想化ベースのセキュリティを有効にする]** オプションを有効にします。

   - プラットフォームのセキュリティ レベルを選択する: **[Secure Boot and DMA Protection] \(セキュアブートと DMA 保護\)**
   - Credential Guard の構成: **UEFI ロックで有効化**
   - Secure Launch Configuration \(セキュア起動の構成\): **有効**

1. **[グループ ポリシー管理エディター]** ウィンドウを閉じます。
1. **[グループ ポリシー管理]** コンソール ウィンドウを閉じます。

#### タスク 2: HVCI と Windows Defender Credential Guard ハードウェア準備ツールを使用して Windows Defender Credential Guard を有効にする

1. **SEA-SVR2** で、管理者として Windows PowerShell を起動します。
1. HVCI と Windows Defender Credential Guard ハードウェア準備ツールを実行するために、Windows PowerShell コマンド プロンプトで次のコマンドを実行します。

   ```powershell
   Set-Location -Path C:\Labfiles\Lab01\
   .\DG_Readiness_Tool.ps1 -Enable -AutoReboot
   ```

1. ツールの実行が完了するまで待ってから、表示されるダイアログに従ってオペレーティング システムを再起動します。
1. 再起動が完了したら、講師から提供された資格情報で **SEA-SVR2** にサインインし直します。

### 結果

この演習を完了すると、次を完了することになります。

1. グループ ポリシー使用して、組織内のすべてのコンピューターに Windows Defender Credential Guard を実装する。
1. ローカル コンピューターで Windows Defender Credential Guard を直ちに有効にする。

## 演習 2: 問題のあるアカウントの検出

### シナリオ

無期限に設定されたパスワードを持つユーザー アカウントが組織に存在するかどうかを確認し、この設定を修復します。 また、90 日以上にサインインしていないアカウントを確認し、無効にします。

この演習の主なタスクは次のとおりです。

1. 無期限のパスワードを持つドメイン アカウントを検出して再構成する。
1. 少なくとも 90 日間はサインインに使用されていないドメイン アカウントを検出して無効にする。

#### タスク 1: 無期限のパスワードを持つドメイン アカウントを検出して再構成する

1. **SEA-SVR2** で、管理者として Windows PowerShell を起動します。
1. 無期限のパスワードを持つ Active Directory 対応ユーザー アカウントを一覧表示するために、Windows PowerShell コマンド プロンプトで次のコマンドを実行します。

   ```powershell
   Get-ADUser -Filter {Enabled -eq $true -and PasswordNeverExpires -eq $true}
   ```

1. 返されたユーザー アカウントの一覧を確認します。
1. 結果セット内のすべてのユーザー アカウントに対してパスワードの有効期限を有効にするために、Windows PowerShell コマンド プロンプトで次のコマンドを実行します。

   ```powershell
   Get-ADUser -Filter {Enabled -eq $true -and PasswordNeverExpires -eq $true} | Set-ADUser -PasswordNeverExpires $false
   ```

1. 結果を確認するために、手順 2. のコマンドを再実行します。結果が返されないことに注目してください。

#### タスク 2: 少なくとも 90 日間はサインインに使用されていないドメイン アカウントを検出して無効にする

1. 少なくとも 90 日間はサインインに使用されていない Active Directory ユーザー アカウントを特定するために、Windows PowerShell コマンド プロンプトで次のコマンドを実行します。

   ```powershell
   $days = (Get-Date).AddDays(-90)
   Get-ADUser -Filter {LastLogonTimeStamp -lt $days -and enabled -eq $true} -Properties LastLogonTimeStamp
   ```

   > **注**: このラボ環境では、結果は返されません。

1. 少なくとも 90 日間はサインインに使用されていない Active Directory ユーザー アカウントを無効にするために、次のコマンドを実行します。

   ```powershell
   Get-ADUser -Filter {LastLogonTimeStamp -lt $days -and enabled -eq $true} -Properties LastLogonTimeStamp | Disable-ADAccount
   ```

   > **注**: このラボ環境では、結果は返されません。

## 演習 3: LAPS の実装

### シナリオ

現在、Contoso のすべてのサーバーとワークステーションで同じローカル管理者アカウントのパスワードが使用されています。 この問題を解決するために、LAP を構成してデプロイします。

この演習の主なタスクは次のとおりです。

1. LAPS (ローカル管理者パスワード ソリューション) の実装用にコンピューター アカウントを準備する。
1. LAPS 用に Active Directory (AD DS) を準備する。
1. LAPS クライアント側拡張をデプロイする。
1. LAPS を確認する。

#### タスク 1: LAPS (ローカル管理者パスワード ソリューション) の実装用にコンピューター アカウントを準備する

1. 指定の OU を作成し、**SEA-SVR1** コンピューター アカウントをその OU に移動するために、**SEA-SVR2** の Windows PowerShell コマンド プロンプトで次のコマンドを実行します。

   ```powershell
   New-ADOrganizationalUnit -Name "Seattle_Servers"
   Get-ADComputer SEA-SVR1 | Move-ADObject –TargetPath "OU=Seattle_Servers,DC=Contoso,DC=com"
   ```

1. LAPS をインストールするために、Windows PowerShell コマンド プロンプトで次のコマンドを実行します。

   ```powershell
   Msiexec /i C:\Labfiles\Lab01\LAPS.x64.msi
   ```

1. ダイアログが表示されたら、**[Welcome to the Local Administrator Password Solution Setup Wizard]** の既定の設定を使用して、**[Custom Setup]** ページまで進みます。
1. **Local Administrator Password Solution Setup** ウィザードの **[Custom Setup]** ページで、**[Management Tools]** の横にあるドロップダウン メニューで **[Entire feature will be installed on the local hard drive]** を選択し、既定値を受け入れてセットアップを起動して、ウィザードを終了します。 
1. セットアップが完了するまで待ちます。 
1. Windows Defender ファイアウォールで他のドメイン参加サーバーからの受信サーバー メッセージ ブロック (SMB) 接続を許可する高度なセキュリティ規則を有効にするために、Windows PowerShell コマンド プロンプトで次のコマンドを入力し、それぞれの後で Enter キーを押します。

   ```
   $rule = Get-NetFirewallRule | Where-Object DisplayName -eq 'File and Printer Sharing (SMB-In)' 
   $rule | Set-NetFirewallRule -Profile Domain
   $rule | Enable-NetFirewallRule
   ```

   > **注**: これは、このラボで後ほど **SEA-SVR1** から **SEA-SVR2** に接続するために必要です。

#### タスク 2: LAPS 用に AD DS を準備する

1. LAPS 用にドメインを準備するために、**SEA-SVR2** の Windows PowerShell コマンド プロンプトで次のコマンドを実行します。

   ```powershell
   Import-Module admpwd.ps
   Update-AdmPwdADSchema
   Set-AdmPwdComputerSelfPermission -Identity "Seattle_Servers"
   ```

1. *SEA-SVR2* で、 **[グループ ポリシー管理]** コンソールを開きます。
1. **[グループ ポリシー管理]** コンソールで、 **[フォレスト: contoso.com]** 、 **[ドメイン]** 、**contoso.com** の順に参照し、**Seattle_Servers** OU にリンクされた **LAPS_GPO** という名前のグループ ポリシー オブジェクト (GPO) を作成します。
1. グループポリシー管理エディターで **LAPS_GPO** を開き、 **[コンピューターの構成]\\[ポリシー]\\[管理用テンプレート]\\[LAPS]** ノードを参照します。
1. **[ローカル管理者のパスワードの管理を有効にする]** オプションを有効にします。
1. 次の設定を使用して、**[パスワードの設定]** オプションを構成します。

   - パスワードの長さ: **20**
   - パスワードの有効期間 (日数): **30**

1. グループ ポリシー管理エディターを閉じます。
1. **[グループ ポリシー管理]** コンソール ウィンドウを閉じます。

#### タスク 3: LAPS クライアント側拡張をデプロイする

1. コンソール セッションを経て **SEA-SVR1** に切り替えたあと、必要に応じて、講師から提供された資格情報でサインインします。

   > **注:** パスワードの有効期限を有効にするスクリプトを前の演習で実行した結果として、パスワードを変更するように求められます。 任意のパスワードを選択し、ラボの残りの部分で使用します。

1. サインインしたら、Windows PowerShell のコマンドプロンプトにアクセスするために、**SConfig** メニューを終了します。
1. LAPS を既定の設定を使用してサイレント モードでインストールするために、Windows PowerShell コマンド プロンプトで次のコマンドを実行します。

   ```powershell
   Start-Process msiexec.exe -Wait -ArgumentList '/i \\SEA-SVR2.contoso.com\c$\Labfiles\Lab01\LAPS.x64.msi /quiet'
   ```

1. LAPS の設定をローカルに適用するグループ ポリシーの処理をトリガーするために、Windows PowerShell コマンド プロンプトで次のコマンドを実行します。

   ```powershell
   gpupdate /force
   ```

#### タスク 4: LAPS を確認する

1. **SEA-SVR2** のコンソール セッションに切り替えます。
1. **[スタート]** メニューから、**LAPS UI**を起動します。
1. **[LAPS UI]** ダイアログ ボックスの **[ComputerName]** ボックスに「**SEA-SVR1**」と入力し、**[検索]** を選択します。
1. **[パスワード]** と **[パスワードの有効期限]** の値を確認し、 **[終了]** を選択します。
1. **Windows PowerShell** コンソールに切り替えてから、パスワードの値を確認するために、Windows PowerShell コマンド プロンプトで次のコマンドを実行します。

   ```powershell
   Get-ADComputer -Identity SEA-SVR1 -Properties ms-Mcs-AdmPwd
   ```

1. SEA-SVR1 に割り当てられているパスワードを確認し、**LAPS UI** ツールに表示されていたものと一致していることに注目します。

   > **注:** この場合、パスワードの値は中かっこのペアで囲まれています。

### 結果

この演習を完了すると、次を完了することになります。

- LAPS 用に OU とコンピューター アカウントを準備する。
- LAPS 用に AD DS を準備する。
- LAPS クライアント側拡張をデプロイする。
- LAPS が正常に実装されたことを確認する。

---
lab:
  title: 'ラボ: フェールオーバー クラスタリングの実装'
  module: 'Module 3: High availability in Windows Server'
ms.openlocfilehash: dce41bd9411fc821c35e903255156bc6f8b558b3
ms.sourcegitcommit: 9a51ea796ef3806ab9e7ec1ff75034b2f929ed2a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/28/2022
ms.locfileid: "137907097"
---
# <a name="lab-implementing-failover-clustering"></a>ラボ: フェールオーバー クラスタリングの実装

## <a name="scenario"></a>シナリオ

Contoso, Ltd. のビジネスが成長するにつれて、そのネットワーク上のアプリケーションやサービスの多くを常に利用可能にしておくことがますます重要になっています。 Contoso は、世界中のさまざまなタイムゾーンで作業する内部および外部のユーザー向けに、多数のサービスとアプリケーションを使用可能にしておく必要があります。 これらのアプリケーションの多くは、ネットワーク負荷分散 (NLB) を使用して高可用性にすることができません。 そのため、これらのアプリケーションの高可用性を実現するには、異なるテクノロジを使用する必要があります。

Contoso のシニア ネットワーク管理者の 1 人として、あなたは Windows Server を実行しているサーバーにフェールオーバー クラスタリングを実装して、ネットワーク サービスとアプリケーションの高可用性を実現する責任があります。 また、フェールオーバー クラスターの構成を計画し、フェールオーバー クラスターにアプリケーションとサービスをデプロイする責任もあります。

## <a name="objectives"></a>目標

このラボを完了すると、次のことができるようになります。

- フェールオーバー クラスターを構成する。
- 高可用性ファイル サーバーをフェールオーバー クラスターにデプロイして構成する。
- 高可用性ファイル サーバーのデプロイを検証する。

## <a name="estimated-time-60-minutes"></a>予想所要時間: **60 分**

## <a name="lab-setup"></a>ラボのセットアップ

仮想マシン: **AZ-801T00A-SEA-DC1**、**AZ-801T00A-SEA-SVR1**、**AZ-801T00A-SEA-SVR2** が実行されている必要があります。 他の VM が実行されていてもかまいませんが、このラボでは必要ありません。

> **注**: **AZ-801T00A-SEA-DC1**、**AZ-801T00A-SEA-SVR1**、**AZ-801T00A-SEA-SVR2** 仮想マシンは、**SEA-DC1**、 **SEA-SVR1**、および **SEA-SVR2** のインストールをホストしています

1. **[SEA-SVR2]** を選択します。
1. 次の資格情報を使用してサインインします。

   - ユーザー名: **Administrator**
   - パスワード: **Pa55w.rd**
   - ドメイン: **CONTOSO**

このラボでは、利用可能な VM 環境を使用します。

## <a name="exercise-1-configuring-iscsi-storage"></a>演習 1: iSCSI ストレージの構成

### <a name="scenario"></a>シナリオ

Contoso には、高可用性を実現する必要がある重要なアプリケーションとサービスがあります。 これらのサービスの一部は NLB を使用できないため、フェールオーバー クラスタリングを実装することにしました。 フェールオーバー クラスタリングには、Internet Small Computer System Interface (iSCSI) ストレージを使用することにします。 最初に、フェールオーバー クラスターをサポートするように iSCSI ストレージを構成します。

この演習の主なタスクは次のとおりです。

- フェールオーバー クラスタリングをインストールする。
- iSCSI 仮想ディスクを構成する。

#### <a name="task-1-install-failover-clustering"></a>タスク 1: フェールオーバー クラスタリングをインストールする

1. **SEA-SVR2** で、管理者として Windows PowerShell を開始します。
1. **SEA-SVR2** で、Windows PowerShell を使用して、**SEA-SVR1** と **SEA-SVR2** に、管理ツールを含む **Failover-Clustering** 機能をインストールします。
1. **SEA-SVR2** で、Windows PowerShell を使用して、**iSCSITarget-Server** 機能を **SEA-DC1** にインストールします。
 
#### <a name="task-2-configure-iscsi-virtual-disks"></a>タスク 2: iSCSI 仮想ディスクを構成する

> **重要:** このラボでは、**SEA-DC1** を使用します。これは、Windows サーバーベースのクラスター用の共有 iSCSI ストレージをホストするための Active Directory Domain Services (AD DS) ドメイン コントローラーとして機能します。 これは、推奨される構成を示しているわけではなく、単にラボの構成を簡略化し、ラボの仮想マシンの数を最小限に抑えることを目的としています。 運用環境では、フェールオーバー クラスター用の共有ストレージをホストするためにドメイン コントローラーを使用しないでください。 代わりに、そのようなストレージは高可用性インフラストラクチャでホストする必要があります。 

1. **SEA-SVR2** で、管理者としてさらに 2 つの Windows PowerShell ウィンドウを起動します。
1. 2 番目と 3 番目の Windows PowerShell ウィンドウを使用して、それぞれ **SEA-DC1** と **SEA-SVR1** への PowerShell リモート処理セッションを確立します。
1. **SEA-DC1** への PowerShell リモート処理セッションを使用して **SEA-DC1** に 3 つの iSCSI 仮想ディスクを作成します。そのためには、**C:\\Storage** ディレクトリを作成して、次のパラメーターを使用して **New-IscsiVirtualDisk** コマンドレットを実行します。

   - ディスク 1:
      - ストレージの場所: **C:\\Storage**
      - ディスク名: **Disk1**
      - サイズ: **10 GB**
   - ディスク 2:
      - ストレージの場所: **C:\\Storage**
      - ディスク名: **Disk2**
      - サイズ: **10 GB**
   - ディスク 3:
      - ストレージの場所: **C:\\Storage**
      - ディスク名: **Disk3**
      - サイズ: **10 GB**

1. **SEA-SVR2** へのローカル Windows PowerShell セッションと **SEA-SVR1** への PowerShell リモート処理セッションを使用して **Start-Service** と **Set-Service** コマンドレットを実行し、**SEA-SVR2** と **SEA-SVR1** で iSCSI イニシエーター (msiscsi) サービスを開始して、自動的に開始するように構成します。
1. **SEA-DC1** への PowerShell リモート処理セッションを使用して、次のパラメーターで **New-IscsiServerTarget** コマンドレットを実行し、**SEA-DC1** 上に iSCSI ターゲットを作成します。

   - ターゲット名: **ISCSI-L03**
   - InitiatorsIds: 
      - **"IQN:iqn.1991-05.com.microsoft:sea-svr1.contoso.com"**
      - **"IQN:iqn.1991-05.com.microsoft:sea-svr2.contoso.com"**

### <a name="results"></a>結果

この演習を完了すると、**SEA-SVR1** と **SEA-SVR2** にフェールオーバー クラスタリング機能がインストールされ、**SEA-DC1**、**SEA-SVR1**、および **SEA-SVR2** に初期化された iSCSI コンポーネントが正常にインストールされています。

## <a name="exercise-2-configuring-a-failover-cluster"></a>演習 2: フェールオーバー クラスターの構成

### <a name="scenario"></a>シナリオ

この演習では、クラスターのインストールを準備してから、クラスターを作成します。

この演習の主なタスクは次のとおりです。

1. クライアントを iSCSI ターゲットに接続する。
1. ディスクを初期化する。
1. フェールオーバー クラスターを検証して作成する。

#### <a name="task-1-connect-clients-to-the-iscsi-targets"></a>タスク 1: クライアントを iSCSI ターゲットに接続する

1. **SEA-SVR2** で、**SEA-DC1** へのPowerShell リモート処理セッションをホストしている **Windows PowerShell** ウィンドウを使用して **IscsiVirtualDiskTargetMapping** コマンドレットを実行し、**SEA-DC1** に iSCSI ディスクをマウントします。
1. **SEA-SVR2** で、ローカル Windows PowerShell セッションを使用して次のコマンドを実行し、**SEA-DC1** でホストされている iSCSI ターゲットに **SEA-SVR2** から接続します。

   ```powershell
   New-iSCSITargetPortal -TargetPortalAddress SEA-DC1.contoso.com  
   Connect-iSCSITarget -NodeAddress iqn.1991-05.com.microsoft:sea-dc1-iSCSI-L03-target
   Get-iSCSITarget | fl
   ```

   > **注:** 最後のコマンドを実行した後、**IsConnected** 変数の値が True であることを確認してください。

1. **SEA-SVR2** で、**SEA-SVR1** への PowerShell リモート処理セッションを使用して次のコマンドを実行し、**SEA-DC1** でホストされている iSCSI ターゲットに **SEA-SVR1** から接続します。

   ```powershell
   New-iSCSITargetPortal -TargetPortalAddress SEA-DC1.contoso.com  
   Connect-iSCSITarget -NodeAddress iqn.1991-05.com.microsoft:sea-dc1-iSCSI-L03-target
   Get-iSCSITarget | fl
   ```

   > **注:** 最後のコマンドを実行した後、**IsConnected** 変数の値が True であることを確認してください。


#### <a name="task-2-initialize-the-disks"></a>タスク 2: ディスクを初期化する

1. **SEA-SVR2** で、ローカル Windows PowerShell セッションを使用して **Get-Disk** コマンドレットを実行し、ローカル ディスクの一覧を表示します。
1. **SEA-SVR2** で、ローカル Windows PowerShell セッションを使用して、次の設定で 3 つの iSCSI ディスクを初期化します。

   - PartitionStyle: **MBR**
   - New-Partition サイズ: **5GB**
   - ファイル システム: **NTFS**
   - ドライブ文字 **E**、**F**、**G** をそれぞれ割り当てます。

#### <a name="task-3-create-a-failover-cluster"></a>タスク 3: フェールオーバー クラスターを作成する

1. **SEA-SVR2** で、ローカル Windows PowerShell セッションを使用して **SEA-CL03** という名前のフェールオーバー クラスターを作成します。IP アドレスは **172.16.10.125** に設定し、最初のノードは **SEA-SVR2.contoso.com** にします。
1. **SEA-SVR2** で、ローカル Windows PowerShell セッションを使用して、新しく作成したクラスターに 2 番目のノードとして **SEA-SVR1** を追加します。

### <a name="results"></a>結果

この演習を完了すると、ディスクが構成され、フェールオーバー クラスターが作成されています。

## <a name="exercise-3-deploying-and-configuring-a-highly-available-file-server"></a>演習 3: 高可用性ファイル サーバーのデプロイと構成

### <a name="scenario"></a>シナリオ

Contoso では、ファイル サービスは重要なサービスであり、高可用性を実現する必要があります。 クラスター インフラストラクチャを作成したら、高可用性ファイル サーバーを構成し、フェールオーバーとフェールバックの設定を実装します。

この演習の主なタスクは次のとおりです。

1. ファイル サーバー アプリケーションをフェールオーバー クラスターに追加する。
1. 高可用性ファイル サーバーに共有フォルダーを追加する。
1. フェールオーバーとフェールバックの設定を構成する。

#### <a name="task-1-add-the-file-server-application-to-the-failover-cluster"></a>タスク 1: ファイル サーバー アプリケーションをフェールオーバー クラスターに追加する

1. **SEA-SVR2** で、**フェールオーバー クラスター マネージャー** コンソールを開き、**SEA-CL03.contoso.com** クラスターに自動的に接続されていることを確認します。
1. **[ノード]** ノードで、**SEA-SVR2** ノードと **SEA-SVR1** ノードの両方が実行中であることを確認します。
1. **[ストレージ]** ノードで **[ディスク]** を選択し、3 つのクラスター ディスクがオンラインになっていることを確認します。
1. **[汎用ファイル サーバー]** オプションと次の設定を使用して、**ファイル サーバー** をクラスター ロールとして追加します。

   - クライアント アクセス名: **FSCluster**
   - アドレス: **172.16.0.130**
   - ストレージ: **クラスター ディスク 1、クラスター ディスク 2**

#### <a name="task-2-add-a-shared-folder-to-a-highly-available-file-server"></a>タスク 2: 高可用性ファイル サーバーに共有フォルダーを追加する

1. **SEA-SVR2** の **フェールオーバー クラスター マネージャー** コンソールで、次の設定を使用して **FSCluster** ロールにファイル共有を追加します (その他はすべて既定値のままにします)。

   - ファイル共有プロファイル: **SMB 共有 - 簡易**
   - 名前: **Docs**

#### <a name="task-3-configure-the-failover-and-failback-settings"></a>タスク 3: フェールオーバーとフェールバックの設定を構成する

1. **SEA-SVR2** の **フェールオーバー クラスター マネージャー** コンソールで、**FSCluster** クラスター ロールの **[プロパティ]** ダイアログ ボックスを使用して、次の設定を構成します。

   - フェールバック: **4 ～ 5 時間**
   - 優先所有者: **SEA-SVR2** と **SEA-SVR1** (**SEA-SVR1** をリストの上部に配置)

### <a name="results"></a>結果

この演習を完了すると、高可用性ファイル サーバーが構成されています。

## <a name="exercise-4-validating-the-deployment-of-the-highly-available-file-server"></a>演習 4: 高可用性ファイル サーバーのデプロイの検証

### <a name="scenario"></a>シナリオ

フェールオーバー クラスターを実装する場合、フェールオーバーとフェールバックのテストを実行する必要があります。 さらに、クォーラム内の監視ディスクを変更することができます。

この演習の主なタスクは次のとおりです。

1. 高可用性ファイル サーバーのデプロイを検証する。
1. ファイル サーバー ロールのフェールオーバーとクォーラム構成を検証する。

#### <a name="task-1-validate-the-highly-available-file-server-deployment"></a>タスク 1: 高可用性ファイル サーバーのデプロイを検証する

1. **SEA-SVR2** で、エクスプローラーを開き、 **\\\\FSCluster\\Docs** 共有にアクセスできることを確認します。
1. 共有内にテスト テキスト ドキュメントを作成します。
1. **SEA-SVR2** で、**フェールオーバー クラスター マネージャー** コンソールを使用して、**FSCluster** ロールを別のノードに移動します。
1. **SEA-SVR2** のエクスプローラーで、 **\\\\FSCluster\\Docs** 共有にまだアクセスできることを確認します。

#### <a name="task-2-validate-the-failover-and-quorum-configuration-for-the-file-server-role"></a>タスク 2: ファイル サーバー ロールのフェールオーバーとクォーラム構成を検証する

1. **SEA-SVR2** の **フェールオーバー クラスター マネージャー** コンソールで、**FSCluster** ロールの現在の所有者を確認します。
1. **FSCluster** ロールの現在の所有者であるクラスター サービスをノードで停止します。
1. **SEA-SVR2** でエクスプローラーを使用して、 **\\\\FSCluster\\Docs** 共有がまだ使用可能であることを確認します。
1. **SEA-SVR2** で、**フェールオーバー クラスター マネージャー** コンソールを使用して、ステップ 2 でクラスター サービスを停止したノード上でサービスを起動します。
1. **SEA-SVR2** の **フェールオーバー クラスター マネージャー** コンソールで、既定の設定を使用して **FSCluster** のクラスター クォーラムを構成します。
1. **フェールオーバー クラスター マネージャー** コンソールで、 **[ディスク]** ノードを参照し、 **[クォーラム内の監視ディスク]** としてマークされたディスクをオフラインにします。
1. **SEA-SVR2** でエクスプローラーを使用して、 **\\\\FSCluster\\Docs** 共有がまだ使用可能であることを確認します。
1. 監視ディスクをオンラインにします。

### <a name="results"></a>結果

この演習を完了すると、フェールオーバー クラスタリングを使用して高可用性が検証されています。


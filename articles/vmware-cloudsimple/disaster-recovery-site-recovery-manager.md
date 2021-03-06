---
title: Azure VMware Solutions (AVS) - VMware Site Recovery Manager を使用して、AVS プライベート クラウドをディザスター リカバリー サイトとして設定する
description: オンプレミスの VMware ワークロード用のディザスター リカバリー サイトとして、AVS プライベート クラウドを設定する方法について説明します
author: sharaths-cs
ms.author: b-shsury
ms.date: 08/20/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 209eb6ed93ed12f97b116b648a36d14e09b822f7
ms.sourcegitcommit: 6ee876c800da7a14464d276cd726a49b504c45c5
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/19/2020
ms.locfileid: "77461183"
---
# <a name="set-up-avs-private-cloud-as-a-disaster-recovery-target-with-vmware-site-recovery-manager"></a>VMware Site Recovery Manager を使用して、AVS プライベート クラウドをディザスター リカバリー ターゲットとして設定する

AVS プライベート クラウドを、オンプレミスの VMware ワークロード用のディザスター リカバリー (DR) サイトとして使用できます。

DR ソリューションは、vSphere Replication と VMware Site Recovery Manager (SRM) をベースにしています。 同様のアプローチに従って、AVS プライベート クラウドを、オンプレミスの復旧サイトによって保護されているプライマリ サイトとして有効にすることができます。

AVS ソリューション:

* DR 専用のデータセンターを設定する必要がなくなります。
* AVS がデプロイされている Azure の場所を活用して、世界規模での地理的回復性を実現できます。
* DR を確立するためのデプロイ コストと総保有コストを削減するオプションが提供されます。

AVS ソリューションでは、次のことを行う必要があります。

* AVS プライベート クラウド内で vSphere Replication と SRM をインストール、構成、管理します。
* AVS プライベート クラウドが保護されたサイトの場合は、独自のライセンスを SRM に提供します。 AVS サイトが復旧サイトとして使用されている場合、追加の SRM ライセンスは必要ありません。

このソリューションでは、vSphere Replication と SRM を完全に制御できます。 使い慣れた UI、API、CLI のインターフェイスにより、既存のスクリプトとツールを使用できるようになります。

![Site Recovery Manager のデプロイ](media/srm-deployment.png)

AVS プライベート クラウドおよびオンプレミス環境と互換性のある、任意のバージョンの vRA と SRM を使用できます。 このガイドの例では、vRA 6.5 と SRM 6.5 を使用します。 これらのバージョンは、AVS でサポートされている、vSphere 6.5 と互換性があります。

## <a name="deploy-the-solution"></a>ソリューションのデプロイ方法

以下のセクションでは、AVS プライベート クラウド内で SRM を使用して、DR ソリューションをデプロイする方法について説明します。

1. [VMware 製品のバージョンに互換性があることを確認する](#verify-that-vmware-product-versions-are-compatible)
2. [DR 環境のサイズを見積もる](#estimate-the-size-of-your-dr-environment)
3. [ご使用の環境用の AVS プライベート クラウドを作成する](#create-an-avs-private-cloud-for-your-environment)
4. [SRM ソリューション用の AVS プライベート クラウド ネットワークを設定する](#set-up-avs-private-cloud-networking-for-the-srm-solution)
5. [オンプレミス ネットワークと AVS プライベート クラウドの間にサイト間 VPN 接続を設定し、必要なポートを開く](#set-up-a-site-to-site-vpn-connection-between-your-on-premises-network-and-the-avs-private-cloud-and-open-required-ports)
6. [AVS プライベート クラウド内にインフラストラクチャ サービスを設定する](#set-up-infrastructure-services-in-your-avs-private-cloud)
7. [オンプレミス環境に vSphere Replication アプライアンスをインストールする](#install-vsphere-replication-appliance-in-your-on-premises-environment)
8. [AVS プライベート クラウド環境に vSphere Replication アプライアンスをインストールする](#install-vsphere-replication-appliance-in-your-avs-private-cloud-environment)
9. [オンプレミス環境に SRM サーバーをインストールする](#install-srm-server-in-your-on-premises-environment)
10. [AVS プライベート クラウドに SRM サーバーをインストールする](#install-srm-server-in-your-avs-private-cloud)

### <a name="verify-that-vmware-product-versions-are-compatible"></a>VMware 製品のバージョンに互換性があることを確認する

このガイド内の構成には、次の互換性要件が適用されます。

* 同じバージョンの SRM が、AVS プライベート クラウドとオンプレミス環境にデプロイされている必要があります。
* 同じバージョンの vSphere Replication が、AVS プライベート クラウドとオンプレミス環境にデプロイされている必要があります。
* AVS プライベート クラウドとオンプレミス環境内の Platform Services Controller (PSC) のバージョンには、互換性がある必要があります。
* AVS プライベート クラウドとオンプレミス環境内の vCenter のバージョンには、互換性がある必要があります。
* SRM と vSphere Replication のバージョンが相互に互換で、PSC および vCenter とも互換性がある必要があります。

関連する VMware ドキュメントと互換性情報へのリンクについては、「[VMware Site Recovery Manager のドキュメント](https://docs.vmware.com/en/Site-Recovery-Manager/index.html)」をご覧ください。

AVS プライベート クラウド内の vCenter と PSC のバージョンを確認するには、AVS ポータルを開きます。 **[リソース]** に移動し、AVS プライベート クラウドを選択し、 **[vSphere 管理ネットワーク]** タブをクリックします。

![AVS プライベート クラウド内の vCenter と PSC のバージョン](media/srm-resources.png)

### <a name="estimate-the-size-of-your-dr-environment"></a>DR 環境のサイズを見積もる

1. 特定されたオンプレミス構成がサポートされている制限内であることを確認します。 SRM 6.5 の場合、制限は [Site Recovery Manager 6.5 の運用上の制限](https://kb.vmware.com/s/article/2147110)に関する VMware サポート技術情報の記事に記載されています。
2. ワークロード サイズと RPO の要件を満たすのに十分なネットワーク帯域幅があることを確認します。 [vSphere Replication の帯域幅要件の計算](https://docs.vmware.com/en/vSphere-Replication/6.5/com.vmware.vsphere.replication-admin.doc/GUID-4A34D0C9-8CC1-46C4-96FF-3BF7583D3C4F.html)に関する VMware サポート技術情報の記事には、帯域幅の制限に関するガイダンスが記載されています。
3. AVS サイズ設定ツールを使用して、オンプレミス環境を保護するために DR サイト内で必要なリソースを見積もります。

### <a name="create-an-avs-private-cloud-for-your-environment"></a>ご使用の環境用の AVS プライベート クラウドを作成する

「[AVS プライベート クラウドを作成する](create-private-cloud.md)」の手順とサイズ設定に関する推奨事項に従って、AVS ポータルから AVS プライベート クラウドを作成します。

### <a name="set-up-avs-private-cloud-networking-for-the-srm-solution"></a>SRM ソリューション用の AVS プライベート クラウド ネットワークを設定する

AVS ポータルにアクセスして、SRM ソリューション用の AVS プライベート クラウド ネットワークを設定します。

SRM ソリューション ネットワーク用の VLAN を作成し、それをサブネット CIDR に割り当てます。 手順については、[VLAN/サブネットの作成と管理](create-vlan-subnet.md)に関する記事をご覧ください。

### <a name="set-up-a-site-to-site-vpn-connection-between-your-on-premises-network-and-the-avs-private-cloud-and-open-required-ports"></a>オンプレミス ネットワークと AVS プライベート クラウドの間にサイト間 VPN 接続を設定し、必要なポートを開く

オンプレミスのネットワークと AVS プライベート クラウドの間にサイト間接続を設定します。 手順については、「[AVS プライベート クラウドへの VPN 接続を構成する](set-up-vpn.md)」を参照してください。

### <a name="set-up-infrastructure-services-in-your-avs-private-cloud"></a>AVS プライベート クラウド内にインフラストラクチャ サービスを設定する

ワークロードとツールを簡単に管理できるように、AVS プライベート クラウド内でインフラストラクチャ サービスを構成します。

次のいずれかを実行する場合は、「[AVS プライベート クラウド上で vCenter の ID プロバイダーとして Azure AD を使用する](azure-ad.md)」の説明に従って、外部 ID プロバイダーを追加できます。

* AVS プライベート クラウド内のオンプレミスの Active Directory (AD) からユーザーを識別します。
* すべてのユーザーに対して、AVS プライベート クラウドに AD を設定します。
* Azure AD を使用します。

AVS プライベート クラウド内のワークロードに対して IP アドレス参照、IP アドレス管理、名前解決サービスを提供するには、「[AVS プライベート クラウドで DNS および DHCP アプリケーションとワークロードを設定する](dns-dhcp-setup.md)」の説明に従って、DHCP および DNS サーバーを設定します。

AVS プライベート クラウド内の管理 VM とホストでは *.cloudsimple.io ドメインが使用されます。 このドメインに対する要求を解決するには、「[条件付きフォワーダーの作成](on-premises-dns-setup.md#create-a-conditional-forwarder)」の説明に従って、DNS サーバー上で DNS 転送を構成します。

### <a name="install-vsphere-replication-appliance-in-your-on-premises-environment"></a>オンプレミス環境に vSphere Replication アプライアンスをインストールする

VMware のドキュメントに従って、オンプレミス環境に vSphere Replication アプライアンス (vRA) をインストールします。 インストールは、次の大まかな手順で構成されています。

1. vRA インストール用のオンプレミス環境を準備します。

2. vmware.com の VR ISO の OVF を使用して、オンプレミス環境に vRA をデプロイします。 vRA 6.5 では、[この VMware ブログ](https://blogs.vmware.com/virtualblocks/2017/01/20/vr-65-ovf-choices)に関連情報が記載されています。

3. オンプレミスの vRA を、オンプレミス サイトの vCenter シングル サインオンに登録します。 vSphere Replication 6.5 での詳細な手順については、[VMware vSphere Replication 6.5 のインストールと構成](https://docs.vmware.com/en/vSphere-Replication/6.5/vsphere-replication-65-install.pdf)に関する VMware のドキュメントをご覧ください。

## <a name="install-vsphere-replication-appliance-in-your-avs-private-cloud-environment"></a>AVS プライベート クラウド環境に vSphere Replication アプライアンスをインストールする

開始する前に、以下があることを確認してください。

* オンプレミス環境内のサブネットから AVS プライベート クラウドの管理サブネットへの IP の到達可能性
* オンプレミスの vSphere 環境内のレプリケーション サブネットから AVS プライベート クラウドの SRM ソリューション サブネットへの IP の到達可能性

手順については、「[AVS プライベート クラウドへの VPN 接続を構成する](set-up-vpn.md)」を参照してください。 手順は、オンプレミス インストールの手順と似ています。

AVS では、vRA および SRM のインストール中に IP アドレスの代わりに FQDN を使用することをお勧めします。 AVS プライベート クラウド内の vCenter と PSC の FQDN を確認するには、AVS ポータルを開きます。 **[リソース]** に移動し、AVS プライベート クラウドを選択し、 **[vSphere 管理ネットワーク]** タブをクリックします。

![AVS プライベート クラウド内での vCenter と PSC の FQDN の検索](media/srm-resources.png)

AVS では、既定の "cloudowner" ユーザーを使用して vRA と SRM をインストールせずに、新しいユーザーを作成する必要があります。 これを行うと、AVS プライベート クラウドの vCenter 環境で高いアップタイムと可用性を確保するのに役立ちます。 しかし、AVS プライベート クラウドの vCenter 内の既定の cloudowner ユーザーには、管理者特権を持つ新しいユーザーを作成するための十分な特権がありません。

vRA と SRM をインストールする前に、cloudowner ユーザーの vCenter 特権をエスカレートしてから、vCenter SSO ドメイン内に管理者特権を持つユーザーを作成する必要があります。 既定の AVS プライベート クラウドのユーザーとアクセス許可モデルについて詳しくは、[AVS プライベート クラウドのアクセス許可モデル](learn-private-cloud-permissions.md)に関するページをご覧ください。

インストールは、次の大まかな手順で構成されています。

1. [権限をエスカレートします](escalate-private-cloud-privileges.md)。
2. vSphere Replication と SRM のインストール用のユーザーを AVS プライベート クラウドに作成します。 下の「[vCenter UI:vRA と SRM のインストール用のユーザーを AVS プライベート クラウドに作成する](#vcenter-ui-create-a-user-in-the-avs-private-cloud-for-vra-and-srm-installation)」で説明されています。
3. vRA インストール用の AVS プライベート クラウド環境を準備します。
4. vmware.com の VR ISO の OVF を使用して、AVS プライベート クラウドに vRA をデプロイします。 vRA 6.5 では、[この VMware ブログ](https://blogs.vmware.com/virtualblocks/2017/01/20/vr-65-ovf-choices)に関連情報が記載されています。
5. vRA 用のファイアウォール規則を構成します。 下の「[AVS ポータル: vRA 用のファイアウォール規則を構成する](#avs-portal-configure-firewall-rules-for-vra)」で説明されています。
6. AVS プライベート クラウド サイトで、vCenter シングル サインオンを使用して AVS プライベート クラウド vRA を登録します。
7. 2 つのアプライアンス間の vSphere Replication 接続を構成します。 必要なポートがファイアウォール経由で開かれていることを確認します。 vSphere Replication 6.5 に対して開く必要があるポート番号の一覧については、[この VMware サポート技術情報の記事](https://kb.vmware.com/s/article/2087769)をご覧ください。

vSphere Replication 6.5 の詳細なインストール手順については、[VMware vSphere Replication 6.5 のインストールと構成](https://docs.vmware.com/en/vSphere-Replication/6.5/vsphere-replication-65-install.pdf)に関する VMware のドキュメントをご覧ください。

#### <a name="vcenter-ui-create-a-user-in-the-avs-private-cloud-for-vra-and-srm-installation"></a>vCenter UI:vRA と SRM のインストール用のユーザーを AVS プライベート クラウドに作成する

AVS ポータルから特権をエスカレートした後、cloudowner ユーザーの資格情報を使用して vCenter にサインインします。

VCenter 内で新しいユーザー `srm-soln-admin` を作成し、そのユーザーを vCenter 内の Administrators グループに追加します。
cloudowner ユーザーとして vCenter からサインアウトし、*srm-soln-admin* ユーザーとしてサインインします。

#### <a name="avs-portal-configure-firewall-rules-for-vra"></a>AVS ポータル: vRA 用のファイアウォール規則を構成する

[ファイアウォールのテーブルと規則の設定](firewall.md)に関する記事の説明に従ってファイアウォール規則を構成してポートを開き、以下の間の通信を有効にします。

* SRM ソリューション ネットワーク内の vRA と管理ネットワーク内の vCenter および ESXi ホスト。
* 2 つのサイトにある vRA アプライアンス。

vSphere Replication 6.5 に対して開く必要があるポート番号の一覧については、[この VMware サポート技術情報の記事](https://kb.vmware.com/s/article/2087769)をご覧ください。

### <a name="install-srm-server-in-your-on-premises-environment"></a>オンプレミス環境に SRM サーバーをインストールする

開始する前に、以下を確認します。

* vSphere Replication アプライアンスが、オンプレミスおよび AVS プライベート クラウド環境にインストールされている。
* 両方のサイトの vSphere Replication アプライアンスが相互に接続されている。
* 前提条件とベスト プラクティスについて、VMware の情報を確認した。 SRM 6.5 については、[SRM 6.5 の前提条件とベスト プラクティス](https://docs.vmware.com/en/Site-Recovery-Manager/6.5/com.vmware.srm.install_config.doc/GUID-BB0C03E4-72BE-4C74-96C3-97AC6911B6B8.html)に関する VMware ドキュメントをご覧ください。

この [VMware ドキュメント](https://docs.vmware.com/en/Site-Recovery-Manager/6.5/com.vmware.srm.install_config.doc/GUID-F474543A-88C5-4030-BB86-F7CC51DADE22.html)で説明されているように、VMware のドキュメントに従って、デプロイ モデル "プラットフォーム サービス コントローラーごとに 1 つの vCenter インスタンスをがある 2 サイト トポロジ" で SRM サーバーのインストールを実行します。 SRM 6.5 のインストール手順については、VMware ドキュメント「[Site Recovery Manager のインストール](https://docs.vmware.com/en/Site-Recovery-Manager/6.5/com.vmware.srm.install_config.doc/GUID-437E1B65-A17B-4B4B-BA5B-C667C90FA418.html)」をご覧ください。

### <a name="install-srm-server-in-your-avs-private-cloud"></a>AVS プライベート クラウドに SRM サーバーをインストールする

開始する前に、以下を確認します。

* vSphere Replication アプライアンスが、オンプレミスおよび AVS プライベート クラウド環境にインストールされている。
* 両方のサイトの vSphere Replication アプライアンスが相互に接続されている。
* 前提条件とベスト プラクティスについて、VMware の情報を確認した。 SRM 6.5 については、[Site Recovery Manager 6.5 サーバーのインストールに関する前提条件とベスト プラクティス](https://docs.vmware.com/en/Site-Recovery-Manager/6.5/com.vmware.srm.install_config.doc/GUID-BB0C03E4-72BE-4C74-96C3-97AC6911B6B8.html)に関するページをご覧ください。

次の手順では、AVS プライベート クラウドの SRM のインストールについて説明します。

1. [vCenter UI:SRM をインストールする](#vcenter-ui-install-srm)
2. [AVS ポータル: SRM 用のファイアウォール規則を構成する](#avs-portal-configure-firewall-rules-for-srm)
3. [vCenter UI:SCP を構成する](#vcenter-ui-configure-srm)
4. [AVS ポータル: 特権のエスカレートを解除する](#avs-portal-de-escalate-privileges)

#### <a name="vcenter-ui-install-srm"></a>vCenter UI:SRM をインストールする

srm-soln-admin 資格情報を使用して vCenter にログインした後、この [VMware ドキュメント](https://docs.vmware.com/en/Site-Recovery-Manager/6.5/com.vmware.srm.install_config.doc/GUID-F474543A-88C5-4030-BB86-F7CC51DADE22.html)で説明されているように、VMware のドキュメントに従って、デプロイ モデル "プラットフォーム サービス コントローラーごとに 1 つの vCenter インスタンスをがある 2 サイト トポロジ" で SRM サーバーのインストールを実行します。 SRM 6.5 のインストール手順については、VMware ドキュメント「[Site Recovery Manager のインストール](https://docs.vmware.com/en/Site-Recovery-Manager/6.5/com.vmware.srm.install_config.doc/GUID-437E1B65-A17B-4B4B-BA5B-C667C90FA418.html)」をご覧ください。

#### <a name="avs-portal-configure-firewall-rules-for-srm"></a>AVS ポータル: SRM 用のファイアウォール規則を構成する

[ファイアウォールのテーブルと規則の設定](firewall.md)に関する記事の説明に従ってファイアウォール規則を構成して、以下の間の通信を可能にします。

AVS プライベート クラウド内の SRM サーバーおよび vCenter と PSC。
両方のサイトの SRM サーバー

vSphere Replication 6.5 に対して開く必要があるポート番号の一覧については、[この VMware サポート技術情報の記事](https://kb.vmware.com/s/article/2087769)をご覧ください。

#### <a name="vcenter-ui-configure-srm"></a>vCenter UI:SCP を構成する

AVS プライベート クラウドに SRM がインストールされた後、VMware Site Recovery Manager のインストールおよび構成ガイドのセクションの説明に従って、以下のタスクを実行します。 SRM 6.5 での手順については、VMware ドキュメント「[Site Recovery Manager のインストール](https://docs.vmware.com/en/Site-Recovery-Manager/6.5/com.vmware.srm.install_config.doc/GUID-437E1B65-A17B-4B4B-BA5B-C667C90FA418.html)」をご覧ください。

1. 保護されたサイトと復旧サイト上で Site Recovery Manager Server インスタンスを接続します。
2. リモートの Site Recovery Manager Server インスタンス へのクライアント接続を確立します。
3. Site Recovery Manager ライセンス キーをインストールします。

#### <a name="avs-portal-de-escalate-privileges"></a>AVS ポータル: 特権のエスカレートを解除する

特権のエスカレートを解除するには、「[特権のエスカレートを解除する](escalate-private-cloud-privileges.md#de-escalate-privileges)」をご覧ください。

## <a name="ongoing-management-of-your-srm-solution"></a>SRM ソリューションの継続的な管理

AVS プライベート クラウド環境では、vSphere Replication および SRM ソフトウェアを完全に制御でき、必要なソフトウェア ライフサイクル管理を実行することが期待されています。 vSphere Replication または SRM を更新あるいはアップグレードする前に、新しいバージョンのソフトウェアが AVS プライベート クラウドの vCenter および PSC と互換性があることを確認します。

> [!NOTE]
> AVS では、現在、マネージド DR サービスを提供するためのオプションを調査しています。 

## <a name="multiple-replication-configuration"></a>複数レプリケーションの構成

 [配列ベースのレプリケーションと vSphere レプリケーション テクノロジの両方を同時に SRM と共に使用](https://blogs.vmware.com/virtualblocks/2017/06/22/srm-array-based-replication-vs-vsphere-replication)できます。 ただし、これらは個別の VM セットに適用する必要があります (特定の VM を配列ベースのレプリケーションまたは vSphere レプリケーションによって保護できますが、両方で保護することはできません)。 さらに、AVS サイトは、複数の保護されたサイトの復旧サイトとして構成できます。 マルチサイト構成については、「[SRM マルチサイト オプション](https://blogs.vmware.com/virtualblocks/2016/07/28/srm-multisite/)」をご覧ください。

## <a name="references"></a>References

* [VMware Site Recovery Manager のドキュメント](https://docs.vmware.com/en/Site-Recovery-Manager/index.html)
* [Site Recovery Manager 6.5 の運用上の制限](https://kb.vmware.com/s/article/2147110)
* [vSphere Replication の帯域幅要件の計算](https://docs.vmware.com/en/vSphere-Replication/6.5/com.vmware.vsphere.replication-admin.doc/GUID-4A34D0C9-8CC1-46C4-96FF-3BF7583D3C4F.html)
* [vSphere Replication 6.5 をデプロイする場合の OVF の選択](https://blogs.vmware.com/virtualblocks/2017/01/20/vr-65-ovf-choices/)
* [VMware vSphere Replication 6.5 のインストールと構成](https://docs.vmware.com/en/vSphere-Replication/6.5/vsphere-replication-65-install.pdf)
* [SRM 6.5 の前提条件とベスト プラクティス](https://docs.vmware.com/en/Site-Recovery-Manager/6.5/com.vmware.srm.install_config.doc/GUID-BB0C03E4-72BE-4C74-96C3-97AC6911B6B8.html)
* [プラットフォーム サービス コントローラーごとに 1 つの vCenter Server インスタンスがある 2 サイト トポロジでの Site Recovery Manager](https://docs.vmware.com/en/Site-Recovery-Manager/6.5/com.vmware.srm.install_config.doc/GUID-F474543A-88C5-4030-BB86-F7CC51DADE22.html)
* [VMware Site Recovery Manager 6.5 インストールおよび構成ガイド](https://docs.vmware.com/en/Site-Recovery-Manager/6.5/com.vmware.srm.install_config.doc/GUID-437E1B65-A17B-4B4B-BA5B-C667C90FA418.html)
* [配列ベースのレプリケーションと vSphere レプリケーションでの SRM に関する VMware ブログ](https://blogs.vmware.com/virtualblocks/2017/06/22/srm-array-based-replication-vs-vsphere-replication)
* [SRM マルチサイト オプションに関する VMware ブログ](https://blogs.vmware.com/virtualblocks/2016/07/28/srm-multisite)
* [vSphere Replication 5.8.x、6.x、8 用に開く必要があるポート番号](https://kb.vmware.com/s/article/2147112)

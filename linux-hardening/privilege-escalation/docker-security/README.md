# Docker Security

{% hint style="success" %}
Learn & practice AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Learn & practice GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Check the [**subscription plans**](https://github.com/sponsors/carlospolop)!
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Share hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
Use [**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_content=docker-security) to easily build and **automate workflows** powered by the world's **most advanced** community tools.\
Get Access Today:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=docker-security" %}

## **Basic Docker Engine Security**

**Dockerエンジン**は、Linuxカーネルの**ネームスペース**と**Cgroups**を使用してコンテナを隔離し、基本的なセキュリティ層を提供します。追加の保護は、**Capabilities dropping**、**Seccomp**、および**SELinux/AppArmor**を通じて提供され、コンテナの隔離が強化されます。**authプラグイン**は、ユーザーのアクションをさらに制限できます。

![Docker Security](https://sreeninet.files.wordpress.com/2016/03/dockersec1.png)

### Secure Access to Docker Engine

Dockerエンジンには、Unixソケットを介してローカルでアクセスするか、HTTPを使用してリモートでアクセスできます。リモートアクセスの場合、機密性、整合性、および認証を確保するためにHTTPSと**TLS**を使用することが重要です。

Dockerエンジンは、デフォルトで`unix:///var/run/docker.sock`のUnixソケットでリッスンします。Ubuntuシステムでは、Dockerの起動オプションは`/etc/default/docker`に定義されています。Docker APIとクライアントへのリモートアクセスを有効にするには、次の設定を追加してDockerデーモンをHTTPソケット経由で公開します:
```bash
DOCKER_OPTS="-D -H unix:///var/run/docker.sock -H tcp://192.168.56.101:2376"
sudo service docker restart
```
しかし、HTTP経由でDockerデーモンを公開することはセキュリティ上の懸念から推奨されません。接続をHTTPSで保護することをお勧めします。接続を保護するための主なアプローチは2つあります：

1. クライアントがサーバーのアイデンティティを確認します。
2. クライアントとサーバーが互いのアイデンティティを相互認証します。

証明書はサーバーのアイデンティティを確認するために使用されます。両方の方法の詳細な例については、[**このガイド**](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-3engine-access/)を参照してください。

### コンテナイメージのセキュリティ

コンテナイメージは、プライベートまたはパブリックリポジトリに保存できます。Dockerはコンテナイメージのためのいくつかのストレージオプションを提供しています：

* [**Docker Hub**](https://hub.docker.com): Dockerのパブリックレジストリサービス。
* [**Docker Registry**](https://github.com/docker/distribution): ユーザーが自分のレジストリをホストできるオープンソースプロジェクト。
* [**Docker Trusted Registry**](https://www.docker.com/docker-trusted-registry): ロールベースのユーザー認証とLDAPディレクトリサービスとの統合を特徴とするDockerの商用レジストリオファリング。

### イメージスキャン

コンテナは、ベースイメージのため、またはベースイメージの上にインストールされたソフトウェアのために**セキュリティ脆弱性**を持つ可能性があります。Dockerは、コンテナのセキュリティスキャンを行い、脆弱性をリストする**Nautilus**というプロジェクトに取り組んでいます。Nautilusは、各コンテナイメージレイヤーを脆弱性リポジトリと比較することでセキュリティホールを特定します。

詳細については、[**こちらをお読みください**](https://docs.docker.com/engine/scan/)。

* **`docker scan`**

**`docker scan`**コマンドを使用すると、イメージ名またはIDを使用して既存のDockerイメージをスキャンできます。たとえば、次のコマンドを実行してhello-worldイメージをスキャンします：
```bash
docker scan hello-world

Testing hello-world...

Organization:      docker-desktop-test
Package manager:   linux
Project name:      docker-image|hello-world
Docker image:      hello-world
Licenses:          enabled

✓ Tested 0 dependencies for known issues, no vulnerable paths found.

Note that we do not currently have vulnerability data for your image.
```
* [**`trivy`**](https://github.com/aquasecurity/trivy)
```bash
trivy -q -f json <container_name>:<tag>
```
* [**`snyk`**](https://docs.snyk.io/snyk-cli/getting-started-with-the-cli)
```bash
snyk container test <image> --json-file-output=<output file> --severity-threshold=high
```
* [**`clair-scanner`**](https://github.com/arminc/clair-scanner)
```bash
clair-scanner -w example-alpine.yaml --ip YOUR_LOCAL_IP alpine:3.5
```
### Docker Image Signing

Dockerイメージの署名は、コンテナで使用されるイメージのセキュリティと整合性を確保します。以下は簡潔な説明です：

* **Docker Content Trust** は、Notaryプロジェクトを利用し、The Update Framework (TUF) に基づいてイメージの署名を管理します。詳細については、[Notary](https://github.com/docker/notary) と [TUF](https://theupdateframework.github.io) を参照してください。
* Dockerコンテンツトラストを有効にするには、`export DOCKER_CONTENT_TRUST=1` を設定します。この機能は、Dockerバージョン1.10以降ではデフォルトでオフになっています。
* この機能が有効な場合、署名されたイメージのみがダウンロードできます。最初のイメージプッシュには、ルートおよびタグ付けキーのパスフレーズを設定する必要があり、Dockerはセキュリティを強化するためにYubikeyもサポートしています。詳細は[こちら](https://blog.docker.com/2015/11/docker-content-trust-yubikey/)で確認できます。
* コンテンツトラストが有効な状態で署名されていないイメージをプルしようとすると、「No trust data for latest」というエラーが発生します。
* 最初のイメージプッシュの後、Dockerはイメージに署名するためにリポジトリキーのパスフレーズを要求します。

プライベートキーをバックアップするには、次のコマンドを使用します：
```bash
tar -zcvf private_keys_backup.tar.gz ~/.docker/trust/private
```
Dockerホストを切り替える際は、操作を維持するためにルートおよびリポジトリキーを移動する必要があります。

***

<figure><img src="../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_content=docker-security)を使用して、世界で最も高度なコミュニティツールによって強化された**ワークフローを簡単に構築および自動化**します。\
今すぐアクセスを取得：

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=docker-security" %}

## コンテナのセキュリティ機能

<details>

<summary>コンテナセキュリティ機能の概要</summary>

**主なプロセス隔離機能**

コンテナ化された環境では、プロジェクトとそのプロセスを隔離することがセキュリティとリソース管理のために重要です。以下は、主要な概念の簡略化された説明です：

**ネームスペース**

* **目的**：プロセス、ネットワーク、ファイルシステムなどのリソースの隔離を確保します。特にDockerでは、ネームスペースがコンテナのプロセスをホストや他のコンテナから分離します。
* **`unshare`の使用**：`unshare`コマンド（または基盤となるシステムコール）は、新しいネームスペースを作成するために利用され、追加の隔離層を提供します。ただし、Kubernetesはこれを本質的にブロックしませんが、Dockerはブロックします。
* **制限**：新しいネームスペースを作成することは、プロセスがホストのデフォルトネームスペースに戻ることを許可しません。ホストネームスペースに侵入するには、通常、ホストの`/proc`ディレクトリへのアクセスが必要で、`nsenter`を使用して入ります。

**コントロールグループ（CGroups）**

* **機能**：主にプロセス間でリソースを割り当てるために使用されます。
* **セキュリティの側面**：CGroups自体は隔離セキュリティを提供しませんが、`release_agent`機能が誤って設定されると、無許可のアクセスに悪用される可能性があります。

**能力のドロップ**

* **重要性**：プロセス隔離のための重要なセキュリティ機能です。
* **機能**：特定の能力をドロップすることにより、ルートプロセスが実行できるアクションを制限します。プロセスがルート権限で実行されていても、必要な能力が欠けていると、特権アクションを実行できず、システムコールは権限不足のために失敗します。

これらは、プロセスが他の能力をドロップした後の**残りの能力**です：

{% code overflow="wrap" %}
```
Current: cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap=ep
```
{% endcode %}

**Seccomp**

デフォルトでDockerに有効になっています。これは、プロセスが呼び出すことができる**syscallをさらに制限する**のに役立ちます。\
**デフォルトのDocker Seccompプロファイル**は[https://github.com/moby/moby/blob/master/profiles/seccomp/default.json](https://github.com/moby/moby/blob/master/profiles/seccomp/default.json)で見つけることができます。

**AppArmor**

Dockerには、アクティブ化できるテンプレートがあります：[https://github.com/moby/moby/tree/master/profiles/apparmor](https://github.com/moby/moby/tree/master/profiles/apparmor)

これにより、機能、syscall、ファイルやフォルダへのアクセスを減らすことができます...

</details>

### Namespaces

**Namespaces**は、Linuxカーネルの機能で、**カーネルリソースを分割**し、あるセットの**プロセス**があるセットの**リソース**を**見る**一方で、**別の**セットの**プロセス**が**異なる**セットのリソースを見ることができるようにします。この機能は、リソースとプロセスのセットに同じ名前空間を持たせることによって機能しますが、それらの名前空間は異なるリソースを指します。リソースは複数の空間に存在することがあります。

Dockerは、コンテナの隔離を実現するために以下のLinuxカーネルのNamespacesを利用しています：

* pid namespace
* mount namespace
* network namespace
* ipc namespace
* UTS namespace

**名前空間に関する詳細情報**は、以下のページを確認してください：

{% content-ref url="namespaces/" %}
[namespaces](namespaces/)
{% endcontent-ref %}

### cgroups

Linuxカーネルの機能**cgroups**は、**一連のプロセス間でcpu、memory、io、network bandwidthなどのリソースを制限する**能力を提供します。Dockerは、特定のコンテナのリソース制御を可能にするcgroup機能を使用してコンテナを作成できます。\
以下は、ユーザースペースのメモリが500mに制限され、カーネルメモリが50mに制限され、cpuシェアが512、blkio-weightが400に設定されたコンテナです。CPUシェアは、コンテナのCPU使用量を制御する比率です。デフォルト値は1024で、範囲は0から1024です。3つのコンテナが同じCPUシェア1024を持つ場合、各コンテナはCPUリソースの競合が発生した場合に最大33%のCPUを使用できます。blkio-weightは、コンテナのIOを制御する比率です。デフォルト値は500で、範囲は10から1000です。
```
docker run -it -m 500M --kernel-memory 50M --cpu-shares 512 --blkio-weight 400 --name ubuntu1 ubuntu bash
```
コンテナのcgroupを取得するには、次のようにします:
```bash
docker run -dt --rm denial sleep 1234 #Run a large sleep inside a Debian container
ps -ef | grep 1234 #Get info about the sleep process
ls -l /proc/<PID>/ns #Get the Group and the namespaces (some may be uniq to the hosts and some may be shred with it)
```
For more information check:

{% content-ref url="cgroups.md" %}
[cgroups.md](cgroups.md)
{% endcontent-ref %}

### Capabilities

Capabilities allow **finer control for the capabilities that can be allowed** for root user. Docker uses the Linux kernel capability feature to **limit the operations that can be done inside a Container** irrespective of the type of user.

When a docker container is run, the **process drops sensitive capabilities that the process could use to escape from the isolation**. This try to assure that the process won't be able to perform sensitive actions and escape:

{% content-ref url="../linux-capabilities.md" %}
[linux-capabilities.md](../linux-capabilities.md)
{% endcontent-ref %}

### Seccomp in Docker

This is a security feature that allows Docker to **limit the syscalls** that can be used inside the container:

{% content-ref url="seccomp.md" %}
[seccomp.md](seccomp.md)
{% endcontent-ref %}

### AppArmor in Docker

**AppArmor**は、**コンテナ**を**制限された**リソースの**セット**に**プログラムごとのプロファイル**で制限するためのカーネル拡張です。:

{% content-ref url="apparmor.md" %}
[apparmor.md](apparmor.md)
{% endcontent-ref %}

### SELinux in Docker

* **ラベリングシステム**: SELinuxは、すべてのプロセスとファイルシステムオブジェクトに一意のラベルを割り当てます。
* **ポリシーの強制**: プロセスラベルがシステム内の他のラベルに対してどのようなアクションを実行できるかを定義するセキュリティポリシーを強制します。
* **コンテナプロセスラベル**: コンテナエンジンがコンテナプロセスを開始するとき、通常は制限されたSELinuxラベル、一般的には`container_t`が割り当てられます。
* **コンテナ内のファイルラベリング**: コンテナ内のファイルは通常`container_file_t`としてラベル付けされます。
* **ポリシールール**: SELinuxポリシーは、`container_t`ラベルを持つプロセスが`container_file_t`としてラベル付けされたファイルとのみ相互作用（読み取り、書き込み、実行）できることを主に保証します。

このメカニズムにより、コンテナ内のプロセスが侵害された場合でも、対応するラベルを持つオブジェクトとのみ相互作用するように制限され、そうした侵害からの潜在的な損害が大幅に制限されます。

{% content-ref url="../selinux.md" %}
[selinux.md](../selinux.md)
{% endcontent-ref %}

### AuthZ & AuthN

In Docker, an authorization plugin plays a crucial role in security by deciding whether to allow or block requests to the Docker daemon. This decision is made by examining two key contexts:

* **Authentication Context**: This includes comprehensive information about the user, such as who they are and how they've authenticated themselves.
* **Command Context**: This comprises all pertinent data related to the request being made.

These contexts help ensure that only legitimate requests from authenticated users are processed, enhancing the security of Docker operations.

{% content-ref url="authz-and-authn-docker-access-authorization-plugin.md" %}
[authz-and-authn-docker-access-authorization-plugin.md](authz-and-authn-docker-access-authorization-plugin.md)
{% endcontent-ref %}

## DoS from a container

If you are not properly limiting the resources a container can use, a compromised container could DoS the host where it's running.

* CPU DoS
```bash
# stress-ng
sudo apt-get install -y stress-ng && stress-ng --vm 1 --vm-bytes 1G --verify -t 5m

# While loop
docker run -d --name malicious-container -c 512 busybox sh -c 'while true; do :; done'
```
* 帯域幅DoS
```bash
nc -lvp 4444 >/dev/null & while true; do cat /dev/urandom | nc <target IP> 4444; done
```
## 興味深いDockerフラグ

### --privilegedフラグ

次のページでは**`--privileged`フラグが何を意味するか**を学ぶことができます：

{% content-ref url="docker-privileged.md" %}
[docker-privileged.md](docker-privileged.md)
{% endcontent-ref %}

### --security-opt

#### no-new-privileges

攻撃者が低特権ユーザーとしてアクセスを得ることができるコンテナを実行している場合、**誤って設定されたsuidバイナリ**があると、攻撃者はそれを悪用し、**コンテナ内で特権を昇格させる**可能性があります。これにより、彼はコンテナから脱出できるかもしれません。

**`no-new-privileges`**オプションを有効にしてコンテナを実行すると、この種の特権昇格を**防ぐことができます**。
```
docker run -it --security-opt=no-new-privileges:true nonewpriv
```
#### その他
```bash
#You can manually add/drop capabilities with
--cap-add
--cap-drop

# You can manually disable seccomp in docker with
--security-opt seccomp=unconfined

# You can manually disable seccomp in docker with
--security-opt apparmor=unconfined

# You can manually disable selinux in docker with
--security-opt label:disable
```
For more **`--security-opt`** options check: [https://docs.docker.com/engine/reference/run/#security-configuration](https://docs.docker.com/engine/reference/run/#security-configuration)

## その他のセキュリティ考慮事項

### シークレットの管理: ベストプラクティス

シークレットをDockerイメージに直接埋め込んだり、環境変数を使用したりすることは避けることが重要です。これらの方法は、`docker inspect`や`exec`のようなコマンドを通じてコンテナにアクセスできる誰にでも機密情報を露出させてしまいます。

**Dockerボリューム**は、機密情報にアクセスするためのより安全な代替手段です。これらはメモリ内の一時ファイルシステムとして利用でき、`docker inspect`やログに関連するリスクを軽減します。ただし、ルートユーザーやコンテナへの`exec`アクセスを持つ者は、依然としてシークレットにアクセスできる可能性があります。

**Dockerシークレット**は、機密情報を扱うためのさらに安全な方法を提供します。イメージビルドフェーズ中にシークレットが必要なインスタンスには、**BuildKit**がビルド時のシークレットをサポートし、ビルド速度を向上させ、追加機能を提供する効率的なソリューションを提供します。

BuildKitを活用するには、以下の3つの方法で有効化できます：

1. 環境変数を通じて: `export DOCKER_BUILDKIT=1`
2. コマンドにプレフィックスを付けて: `DOCKER_BUILDKIT=1 docker build .`
3. Docker設定でデフォルトで有効にする: `{ "features": { "buildkit": true } }`、その後Dockerを再起動します。

BuildKitは、`--secret`オプションを使用してビルド時のシークレットを利用でき、これらのシークレットがイメージビルドキャッシュや最終イメージに含まれないようにします。コマンドの例:
```bash
docker build --secret my_key=my_value ,src=path/to/my_secret_file .
```
実行中のコンテナに必要な秘密情報について、**Docker Compose と Kubernetes** は堅牢なソリューションを提供します。Docker Compose は、`docker-compose.yml` の例に示すように、秘密ファイルを指定するためにサービス定義内で `secrets` キーを利用します:
```yaml
version: "3.7"
services:
my_service:
image: centos:7
entrypoint: "cat /run/secrets/my_secret"
secrets:
- my_secret
secrets:
my_secret:
file: ./my_secret_file.txt
```
この設定により、Docker Composeを使用してサービスを起動する際にシークレットを使用することができます。

Kubernetes環境では、シークレットがネイティブにサポートされており、[Helm-Secrets](https://github.com/futuresimple/helm-secrets)のようなツールでさらに管理できます。Kubernetesのロールベースアクセス制御（RBAC）は、Docker Enterpriseと同様にシークレット管理のセキュリティを強化します。

### gVisor

**gVisor**は、Goで書かれたアプリケーションカーネルで、Linuxシステムの表面の大部分を実装しています。これは、アプリケーションとホストカーネルの間に**隔離境界**を提供する`runsc`という[Open Container Initiative (OCI)](https://www.opencontainers.org)ランタイムを含んでいます。`runsc`ランタイムはDockerとKubernetesと統合されており、サンドボックス化されたコンテナを簡単に実行できます。

{% embed url="https://github.com/google/gvisor" %}

### Kata Containers

**Kata Containers**は、軽量の仮想マシンを使用して安全なコンテナランタイムを構築するために活動しているオープンソースコミュニティです。これにより、コンテナのように感じられ、動作しますが、**ハードウェア仮想化**技術を使用して、より強力なワークロードの隔離を提供します。

{% embed url="https://katacontainers.io/" %}

### まとめのヒント

* **`--privileged`フラグを使用したり、** [**Dockerソケットをコンテナ内にマウントしないでください**](https://raesene.github.io/blog/2016/03/06/The-Dangers-Of-Docker.sock/)**。** Dockerソケットはコンテナを生成することを可能にするため、例えば`--privileged`フラグを使用して別のコンテナを実行することでホストを完全に制御する簡単な方法です。
* **コンテナ内でrootとして実行しないでください。** [**異なるユーザーを使用し**](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#user) **、** [**ユーザーネームスペースを使用してください**](https://docs.docker.com/engine/security/userns-remap/)**。** コンテナ内のrootは、ユーザーネームスペースで再マップされない限り、ホストのrootと同じです。主にLinuxネームスペース、能力、およびcgroupsによって軽く制限されています。
* [**すべての能力を削除**](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities) **(`--cap-drop=all`)し、必要なものだけを有効にしてください** (`--cap-add=...`)。多くのワークロードは能力を必要とせず、追加すると潜在的な攻撃の範囲が広がります。
* [**“no-new-privileges”セキュリティオプションを使用してください**](https://raesene.github.io/blog/2019/06/01/docker-capabilities-and-no-new-privs/) **。これにより、プロセスがより多くの特権を取得するのを防ぎます。例えば、suidバイナリを通じて。**
* [**コンテナに利用可能なリソースを制限してください**](https://docs.docker.com/engine/reference/run/#runtime-constraints-on-resources)**。** リソース制限は、サービス拒否攻撃からマシンを保護できます。
* **seccomp** [**、AppArmor**](https://docs.docker.com/engine/security/apparmor/) **（またはSELinux）プロファイルを調整して、コンテナに必要な最小限のアクションとシステムコールを制限してください。**
* **公式のdockerイメージを使用し** [**、署名を要求するか、これに基づいて自分のものを構築してください。**](https://docs.docker.com/docker-hub/official_images/) **バックドア付きのイメージを継承したり使用しないでください。** また、ルートキーやパスフレーズを安全な場所に保管してください。DockerはUCPでキーを管理する計画を持っています。
* **定期的に** **イメージを再構築して、ホストとイメージにセキュリティパッチを適用してください。**
* **シークレットを賢く管理し、攻撃者がアクセスしにくくしてください。**
* **Dockerデーモンを公開する場合は、クライアントとサーバーの認証を使用してHTTPSを使用してください。**
* Dockerfileでは、**ADDの代わりにCOPYを優先してください。** ADDは自動的に圧縮ファイルを抽出し、URLからファイルをコピーできます。COPYにはこれらの機能がありません。可能な限りADDの使用を避け、リモートURLやZipファイルを通じた攻撃に対して脆弱にならないようにしてください。
* **各マイクロサービスに対して別々のコンテナを持ってください。**
* **コンテナ内にsshを置かないでください。** “docker exec”を使用してコンテナにsshできます。
* **より小さな**コンテナ**イメージを持ってください。**

## Dockerブレイクアウト / 特権昇格

もしあなたが**dockerコンテナ内にいる**か、**dockerグループのユーザーにアクセスできる場合**、**脱出して特権を昇格させる**ことを試みることができます：

{% content-ref url="docker-breakout-privilege-escalation/" %}
[docker-breakout-privilege-escalation](docker-breakout-privilege-escalation/)
{% endcontent-ref %}

## Docker認証プラグインバイパス

もしあなたがdockerソケットにアクセスできるか、**dockerグループのユーザーにアクセスできるが、docker認証プラグインによって行動が制限されている場合**、それを**バイパスできるか確認してください：**

{% content-ref url="authz-and-authn-docker-access-authorization-plugin.md" %}
[authz-and-authn-docker-access-authorization-plugin.md](authz-and-authn-docker-access-authorization-plugin.md)
{% endcontent-ref %}

## Dockerの強化

* ツール[**docker-bench-security**](https://github.com/docker/docker-bench-security)は、プロダクションでDockerコンテナを展開する際の一般的なベストプラクティスをチェックするスクリプトです。テストはすべて自動化されており、[CIS Docker Benchmark v1.3.1](https://www.cisecurity.org/benchmark/docker/)に基づいています。\
このツールは、dockerを実行しているホストまたは十分な特権を持つコンテナから実行する必要があります。**READMEでの実行方法を確認してください：** [**https://github.com/docker/docker-bench-security**](https://github.com/docker/docker-bench-security)。

## 参考文献

* [https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
* [https://twitter.com/_fel1x/status/1151487051986087936](https://twitter.com/_fel1x/status/1151487051986087936)
* [https://ajxchapman.github.io/containers/2020/11/19/privileged-container-escape.html](https://ajxchapman.github.io/containers/2020/11/19/privileged-container-escape.html)
* [https://sreeninet.wordpress.com/2016/03/06/docker-security-part-1overview/](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-1overview/)
* [https://sreeninet.wordpress.com/2016/03/06/docker-security-part-2docker-engine/](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-2docker-engine/)
* [https://sreeninet.wordpress.com/2016/03/06/docker-security-part-3engine-access/](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-3engine-access/)
* [https://sreeninet.wordpress.com/2016/03/06/docker-security-part-4container-image/](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-4container-image/)
* [https://en.wikipedia.org/wiki/Linux_namespaces](https://en.wikipedia.org/wiki/Linux_namespaces)
* [https://towardsdatascience.com/top-20-docker-security-tips-81c41dd06f57](https://towardsdatascience.com/top-20-docker-security-tips-81c41dd06f57)
* [https://www.redhat.com/sysadmin/privileged-flag-container-engines](https://www.redhat.com/sysadmin/privileged-flag-container-engines)
* [https://docs.docker.com/engine/extend/plugins_authorization](https://docs.docker.com/engine/extend/plugins_authorization)
* [https://towardsdatascience.com/top-20-docker-security-tips-81c41dd06f57](https://towardsdatascience.com/top-20-docker-security-tips-81c41dd06f57)
* [https://resources.experfy.com/bigdata-cloud/top-20-docker-security-tips/](https://resources.experfy.com/bigdata-cloud/top-20-docker-security-tips/)

<figure><img src="../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_content=docker-security)を使用して、世界で最も高度なコミュニティツールによって駆動される**ワークフローを簡単に構築し、自動化**してください。\
今日アクセスを取得：

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=docker-security" %}

{% hint style="success" %}
AWSハッキングを学び、実践する：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCPハッキングを学び、実践する： <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricksをサポートする</summary>

* [**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)を確認してください！
* **💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**テレグラムグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**をフォローしてください。**
* **ハッキングのトリックを共有するために、[**HackTricks**](https://github.com/carlospolop/hacktricks)と[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のGitHubリポジトリにPRを提出してください。**

</details>
{% endhint %}

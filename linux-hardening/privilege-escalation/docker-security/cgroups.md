# CGroups

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

## 基本情報

**Linuxコントロールグループ**、または**cgroups**は、CPU、メモリ、ディスクI/Oなどのシステムリソースをプロセスグループ間で割り当て、制限し、優先順位を付けることを可能にするLinuxカーネルの機能です。これは、リソース制限、ワークロードの分離、異なるプロセスグループ間のリソース優先順位付けなどの目的に役立つ、プロセスコレクションのリソース使用を**管理および隔離する**ためのメカニズムを提供します。

**cgroupsには2つのバージョン**があります：バージョン1とバージョン2。両方はシステム上で同時に使用できます。主な違いは、**cgroupsバージョン2**が**階層的なツリー構造**を導入し、プロセスグループ間でのリソース配分をより微妙かつ詳細に行えるようにすることです。さらに、バージョン2は、以下のようなさまざまな改善をもたらします：

新しい階層的な組織に加えて、cgroupsバージョン2は**新しいリソースコントローラーのサポート**、レガシーアプリケーションへのより良いサポート、パフォーマンスの向上など、**いくつかの他の変更と改善**も導入しました。

全体として、cgroups **バージョン2はバージョン1よりも多くの機能と優れたパフォーマンスを提供**しますが、後者は古いシステムとの互換性が懸念される特定のシナリオではまだ使用される可能性があります。

任意のプロセスのv1およびv2 cgroupsをリストするには、そのcgroupファイルを/proc/\<pid>で確認します。次のコマンドを使用して、シェルのcgroupsを確認することから始めることができます：
```shell-session
$ cat /proc/self/cgroup
12:rdma:/
11:net_cls,net_prio:/
10:perf_event:/
9:cpuset:/
8:cpu,cpuacct:/user.slice
7:blkio:/user.slice
6:memory:/user.slice 5:pids:/user.slice/user-1000.slice/session-2.scope 4:devices:/user.slice
3:freezer:/
2:hugetlb:/testcgroup
1:name=systemd:/user.slice/user-1000.slice/session-2.scope
0::/user.slice/user-1000.slice/session-2.scope
```
The output structure is as follows:

* **Numbers 2–12**: cgroups v1, 各行は異なるcgroupを表します。これらのコントローラーは、番号の隣に指定されています。
* **Number 1**: これもcgroups v1ですが、管理目的のみに使用され（例えば、systemdによって設定され）、コントローラーがありません。
* **Number 0**: cgroups v2を表します。コントローラーはリストされておらず、この行はcgroups v2のみを実行しているシステムで独占的です。
* **名前は階層的**で、ファイルパスに似ており、異なるcgroups間の構造と関係を示しています。
* **/user.sliceや/system.sliceのような名前**はcgroupsの分類を指定し、user.sliceは通常systemdによって管理されるログインセッション用、system.sliceはシステムサービス用です。

### cgroupsの表示

ファイルシステムは通常、**cgroups**にアクセスするために利用され、カーネルとのインタラクションに伝統的に使用されるUnixシステムコールインターフェースとは異なります。シェルのcgroup設定を調査するには、**/proc/self/cgroup**ファイルを確認し、シェルのcgroupを明らかにします。その後、**/sys/fs/cgroup**（または**`/sys/fs/cgroup/unified`**）ディレクトリに移動し、cgroupの名前を共有するディレクトリを見つけることで、cgroupに関連するさまざまな設定やリソース使用情報を観察できます。

![Cgroup Filesystem](<../../../.gitbook/assets/image (1128).png>)

cgroupsの主要なインターフェースファイルは**cgroup**で始まります。**cgroup.procs**ファイルは、標準コマンド（catなど）で表示でき、cgroup内のプロセスをリストします。別のファイル、**cgroup.threads**にはスレッド情報が含まれています。

![Cgroup Procs](<../../../.gitbook/assets/image (281).png>)

シェルを管理するcgroupsは通常、メモリ使用量とプロセス数を制御する2つのコントローラーを含みます。コントローラーと対話するには、コントローラーのプレフィックスを持つファイルを参照する必要があります。例えば、**pids.current**を参照してcgroup内のスレッド数を確認します。

![Cgroup Memory](<../../../.gitbook/assets/image (677).png>)

値に**max**が示されている場合、cgroupに特定の制限がないことを示唆しています。ただし、cgroupsの階層的な性質により、ディレクトリ階層の下位レベルのcgroupによって制限が課される可能性があります。

### cgroupsの操作と作成

プロセスは**`cgroup.procs`ファイルにそのプロセスID（PID）を書き込むことによって**cgroupsに割り当てられます。これにはroot権限が必要です。例えば、プロセスを追加するには:
```bash
echo [pid] > cgroup.procs
```
同様に、**cgroup属性を変更すること、例えばPID制限を設定すること**は、関連するファイルに希望の値を書き込むことで行われます。cgroupの最大3,000 PIDを設定するには:
```bash
echo 3000 > pids.max
```
**新しいcgroupsの作成**は、cgroup階層内に新しいサブディレクトリを作成することを含み、これによりカーネルは必要なインターフェースファイルを自動的に生成します。アクティブなプロセスのないcgroupsは`rmdir`で削除できますが、いくつかの制約に注意してください：

* **プロセスはリーフcgroupsにのみ配置できます**（つまり、階層内で最もネストされたもの）。
* **cgroupは親に存在しないコントローラーを持つことはできません**。
* **子cgroupsのコントローラーは`cgroup.subtree_control`ファイルで明示的に宣言する必要があります**。たとえば、子cgroupでCPUとPIDコントローラーを有効にするには：
```bash
echo "+cpu +pids" > cgroup.subtree_control
```
The **root cgroup**はこれらのルールの例外であり、プロセスを直接配置することを許可します。これを使用して、systemd管理からプロセスを削除することができます。

**cgroup内のCPU使用量の監視**は、`cpu.stat`ファイルを通じて可能で、消費された総CPU時間を表示し、サービスのサブプロセス全体の使用状況を追跡するのに役立ちます：

<figure><img src="../../../.gitbook/assets/image (908).png" alt=""><figcaption><p>cpu.statファイルに表示されるCPU使用統計</p></figcaption></figure>

## 参考文献

* **書籍: How Linux Works, 3rd Edition: What Every Superuser Should Know By Brian Ward**

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

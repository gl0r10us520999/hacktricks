# macOSプロセスの乱用

{% hint style="success" %}
AWSハッキングの学習と実践：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCPハッキングの学習と実践：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricksをサポートする</summary>

* [**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)をチェック！
* 💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)に参加するか、[**telegramグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**をフォロー**してください。
* [**HackTricks**](https://github.com/carlospolop/hacktricks)と[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のGitHubリポジトリにPRを提出してハッキングテクニックを共有してください。

</details>
{% endhint %}

## プロセスの基本情報

プロセスは実行中の実行可能ファイルのインスタンスですが、プロセスはコードを実行しません。これらはスレッドです。したがって、**プロセスは実行中のスレッドのためのコンテナに過ぎません**。メモリ、ディスクリプタ、ポート、権限を提供します...

従来、プロセスは他のプロセス（PID 1を除く）内で**`fork`**を呼び出すことによって開始され、現在のプロセスの正確なコピーを作成し、その後**子プロセス**は通常、新しい実行可能ファイルをロードして実行するために**`execve`**を呼び出しました。その後、このプロセスをメモリコピーなしでより速くするために**`vfork`**が導入されました。\
その後、**`posix_spawn`**が導入され、**`vfork`**と**`execve`**を1つの呼び出しで組み合わせ、フラグを受け入れるようになりました。

- `POSIX_SPAWN_RESETIDS`：実効IDを実IDにリセット
- `POSIX_SPAWN_SETPGROUP`：プロセスグループ所属を設定
- `POSUX_SPAWN_SETSIGDEF`：シグナルのデフォルト動作を設定
- `POSIX_SPAWN_SETSIGMASK`：シグナルマスクを設定
- `POSIX_SPAWN_SETEXEC`：同じプロセスで実行（より多くのオプションを備えた`execve`のような）
- `POSIX_SPAWN_START_SUSPENDED`：中断して開始
- `_POSIX_SPAWN_DISABLE_ASLR`：ASLRなしで開始
- `_POSIX_SPAWN_NANO_ALLOCATOR:` libmallocのNanoアロケータを使用
- `_POSIX_SPAWN_ALLOW_DATA_EXEC:` データセグメントでの`rwx`を許可
- `POSIX_SPAWN_CLOEXEC_DEFAULT`：`exec(2)`時にすべてのファイル記述子をデフォルトで閉じる
- `_POSIX_SPAWN_HIGH_BITS_ASLR:` ASLRスライドの上位ビットをランダム化

さらに、`posix_spawn`は、生成されたプロセスのいくつかの側面を制御する**`posix_spawnattr`**の配列を指定することを可能にし、ディスクリプタの状態を変更する**`posix_spawn_file_actions`**を提供します。

プロセスが終了すると、親プロセスに**リターンコードを送信**し（親が死んだ場合、新しい親はPID 1になります）、`SIGCHLD`シグナルを使用してこの値を取得する必要があります。それが起こるまで、子プロセスはゾンビ状態にあり、まだリストされていますがリソースを消費しません。

### PID

PID（プロセス識別子）は一意のプロセスを識別します。XNUでは、**PID**は**64ビット**で単調に増加し、**決してラップしません**（乱用を避けるため）。

### プロセスグループ、セッション、Coalation

**プロセス**は、それらをより簡単に処理するために**グループ**に挿入できます。たとえば、シェルスクリプト内のコマンドは同じプロセスグループになるため、例えばkillを使用して**一緒にシグナルを送信**することができます。\
また、**プロセスをセッションにグループ化**することも可能です。プロセスがセッションを開始すると（`setsid(2)`）、子プロセスはセッション内に設定されますが、独自のセッションを開始しない限り。

CoalationはDarwinでプロセスをグループ化する別の方法です。Coalationに参加するプロセスは、プールリソースにアクセスしたり、レジャーを共有したり、Jetsamに直面したりすることができます。Coalationには異なる役割があります：リーダー、XPCサービス、エクステンション。

### 資格情報とペルソナ

各プロセスには、システムでの特権を識別する**資格情報**があります。各プロセスには1つの主要な`uid`と1つの主要な`gid`があります（ただし、複数のグループに属する場合があります）。\
バイナリに`setuid/setgid`ビットがある場合、ユーザーIDとグループIDを変更することも可能です。\
新しいuid/gidを設定するためのいくつかの関数があります。

シスコール**`persona`**は、**別の**セットの**資格情報**を提供します。ペルソナを採用すると、そのuid、gid、およびグループメンバーシップを**一度に**仮定します。[**ソースコード**](https://github.com/apple/darwin-xnu/blob/main/bsd/sys/persona.h)では、構造体を見つけることができます。
```c
struct kpersona_info { uint32_t persona_info_version;
uid_t    persona_id; /* overlaps with UID */
int      persona_type;
gid_t    persona_gid;
uint32_t persona_ngroups;
gid_t    persona_groups[NGROUPS];
uid_t    persona_gmuid;
char     persona_name[MAXLOGNAME + 1];

/* TODO: MAC policies?! */
}
```
## スレッドの基本情報

1. **POSIXスレッド（pthreads）:** macOSはC/C++向けの標準スレッドAPIであるPOSIXスレッド（`pthreads`）をサポートしています。macOSにおけるpthreadの実装は`/usr/lib/system/libsystem_pthread.dylib`にあり、これは一般に利用可能な`libpthread`プロジェクトから来ています。このライブラリはスレッドの作成や管理に必要な関数を提供します。
2. **スレッドの作成:** 新しいスレッドを作成するには`pthread_create()`関数が使用されます。内部的には、この関数は`bsdthread_create()`を呼び出し、これはXNUカーネル（macOSが基づいているカーネル）固有の低レベルシステムコールです。このシステムコールは、スケジューリングポリシーやスタックサイズなどのスレッドの振る舞いを指定する`pthread_attr`（属性）から派生したさまざまなフラグを取ります。
* **デフォルトのスタックサイズ:** 新しいスレッドのデフォルトのスタックサイズは512 KBであり、通常の操作には十分ですが、必要に応じてスレッド属性を介して調整することができます。
3. **スレッドの初期化:** `__pthread_init()`関数はスレッドのセットアップ中に重要であり、`env[]`引数を利用してスタックの場所やサイズなどの環境変数を解析します。

#### macOSにおけるスレッドの終了

1. **スレッドの終了:** 通常、スレッドは`pthread_exit()`を呼び出すことで終了します。この関数により、スレッドはきれいに終了し、必要なクリーンアップが行われ、スレッドは参加者に戻り値を送信することができます。
2. **スレッドのクリーンアップ:** `pthread_exit()`を呼び出すと、関数`pthread_terminate()`が呼び出され、関連するすべてのスレッド構造の削除が処理されます。これにより、Machスレッドポート（MachはXNUカーネルの通信サブシステム）が解放され、カーネルレベルのスレッドに関連する構造が削除される`bsdthread_terminate`が呼び出されます。

#### 同期メカニズム

共有リソースへのアクセスを管理し、競合状態を回避するために、macOSはいくつかの同期プリミティブを提供しています。これらはデータの整合性とシステムの安定性を確保するためにマルチスレッド環境で重要です。

1. **ミューテックス:**
* **通常のミューテックス（シグネチャ: 0x4D555458）:** メモリフットプリントが60バイト（ミューテックス56バイトとシグネチャ4バイト）の標準ミューテックス。
* **高速ミューテックス（シグネチャ: 0x4d55545A）:** 通常のミューテックスに類似していますが、より高速な操作に最適化されており、サイズも60バイトです。
2. **条件変数:**
* 特定の条件が発生するのを待つために使用され、サイズは44バイトです（40バイトと4バイトのシグネチャ）。
* **条件変数属性（シグネチャ: 0x434e4441）:** 条件変数の構成属性で、サイズは12バイトです。
3. **一度だけ変数（シグネチャ: 0x4f4e4345）:**
* 初期化コードが一度だけ実行されることを保証します。サイズは12バイトです。
4. **読み取り書き込みロック:**
* 一度に複数のリーダーまたは1つのライターを許可し、共有データへの効率的なアクセスを可能にします。
* **読み取り書き込みロック（シグネチャ: 0x52574c4b）:** サイズは196バイトです。
* **読み取り書き込みロック属性（シグネチャ: 0x52574c41）:** 読み取り書き込みロックの属性で、サイズは20バイトです。

{% hint style="success" %}
これらのオブジェクトの最後の4バイトはオーバーフローを検出するために使用されます。
{% endhint %}

### スレッドローカル変数（TLV）

**スレッドローカル変数（TLV）**は、Mach-Oファイル（macOSの実行可能ファイルの形式）のコンテキストで、マルチスレッドアプリケーションの**各スレッド**に固有の変数を宣言するために使用されます。これにより、各スレッドが変数の独自のインスタンスを持ち、ミューテックスなどの明示的な同期メカニズムを必要とせずに競合を回避し、データの整合性を維持できます。

C言語および関連言語では、**`__thread`**キーワードを使用してスレッドローカル変数を宣言できます。以下は、例での動作方法です:
```c
cCopy code__thread int tlv_var;

void main (int argc, char **argv){
tlv_var = 10;
}
```
このスニペットでは、`tlv_var`をスレッドローカル変数として定義しています。このコードを実行する各スレッドは、独自の`tlv_var`を持ち、1つのスレッドが`tlv_var`を変更しても、他のスレッドの`tlv_var`に影響を与えません。

Mach-Oバイナリでは、スレッドローカル変数に関連するデータは特定のセクションに整理されています:

* **`__DATA.__thread_vars`**: このセクションには、スレッドローカル変数に関するメタデータが含まれており、そのタイプや初期化状態などが含まれています。
* **`__DATA.__thread_bss`**: このセクションは、明示的に初期化されていないスレッドローカル変数に使用されます。ゼロで初期化されたデータのために確保されたメモリの一部です。

Mach-Oは、スレッドが終了する際にスレッドローカル変数を管理するための特定のAPIである**`tlv_atexit`**を提供しています。このAPIを使用すると、スレッドが終了するときにスレッドローカルデータをクリーンアップするための特別な関数である**デストラクタを登録**することができます。

### スレッドの優先度

スレッドの優先度を理解するには、オペレーティングシステムがどのスレッドを実行するか、いつ実行するかを見る必要があります。この決定は、各スレッドに割り当てられた優先度レベルに影響を受けます。macOSやUnix系システムでは、これは`nice`、`renice`、Quality of Service (QoS) クラスなどの概念を使用して処理されます。

#### NiceとRenice

1. **Nice:**
* プロセスの`nice`値は、その優先度に影響を与える数値です。すべてのプロセスには、-20（最高の優先度）から19（最低の優先度）までの`nice`値があります。プロセスが作成されたときのデフォルトの`nice`値は通常0です。
* より低い`nice`値（-20に近い）はプロセスをより「利己的」にし、他の`nice`値が高いプロセスよりもCPU時間を多く割り当てます。
2. **Renice:**
* `renice`は、既に実行中のプロセスの`nice`値を変更するために使用されるコマンドです。これは、プロセスの優先度を動的に調整するために使用でき、新しい`nice`値に基づいてCPU時間の割り当てを増減させることができます。
* たとえば、プロセスが一時的により多くのCPUリソースが必要な場合、`renice`を使用してその`nice`値を下げることができます。

#### Quality of Service (QoS) クラス

QoSクラスは、特に**Grand Central Dispatch (GCD)**をサポートするmacOSなどのシステムでスレッドの優先度を処理するためのより現代的なアプローチです。QoSクラスを使用すると、開発者は重要度や緊急度に基づいて作業を異なるレベルに分類することができます。macOSは、これらのQoSクラスに基づいてスレッドの優先順位付けを自動的に管理します:

1. **User Interactive:**
* このクラスは、ユーザーと対話しているタスクや即座の結果が必要なタスクに使用されます。これらのタスクは、インターフェースが応答性を維持するために最高の優先度が与えられます（例: アニメーションやイベント処理）。
2. **User Initiated:**
* ユーザーが開始し、即座の結果が期待されるタスク、例えばドキュメントを開くかボタンをクリックして計算が必要なタスクなどに使用されます。これらは高い優先度ですが、ユーザーインタラクティブより下です。
3. **Utility:**
* これらのタスクは長時間実行され、通常は進行状況インジケーターが表示されます（例: ファイルのダウンロード、データのインポート）。ユーザーが開始したタスクよりも優先度が低く、即座に完了する必要はありません。
4. **Background:**
* このクラスは、バックグラウンドで動作し、ユーザーには見えないタスクに使用されます。これらはインデックス作成、同期、バックアップなどのタスクであり、最低の優先度を持ち、システムパフォーマンスに最小限の影響を与えます。

QoSクラスを使用すると、開発者は正確な優先度の数値を管理する必要はなく、代わりにタスクの性質に焦点を当て、システムがCPUリソースを適切に最適化します。

さらに、スケジューリングポリシーには異なる**スレッドスケジューリングポリシー**があり、スケジューラが考慮するスケジューリングパラメータのセットを指定するために使用できます。これは`thread_policy_[set/get]`を使用して行うことができ、競合状態攻撃で役立つかもしれません。
### Python Injection

環境変数**`PYTHONINSPECT`**が設定されている場合、Pythonプロセスは終了時にPython CLIに移行します。対話セッションの開始時に実行するPythonスクリプトを示すために**`PYTHONSTARTUP`**を使用することも可能です。\
ただし、**`PYTHONINSPECT`**が対話セッションを作成するときには、**`PYTHONSTARTUP`**スクリプトは実行されません。

**`PYTHONPATH`**や**`PYTHONHOME`**などの他の環境変数も、Pythonコマンドが任意のコードを実行するのに役立つかもしれません。

**`pyinstaller`**でコンパイルされた実行可能ファイルは、埋め込まれたPythonを使用していても、これらの環境変数を使用しません。

{% hint style="danger" %}
全体的に、環境変数を悪用してPythonが任意のコードを実行する方法を見つけることはできませんでした。\
ただし、ほとんどの人々は**Hombrew**を使用してPythonをインストールします。これにより、デフォルトの管理者ユーザーにとって**書き込み可能な場所**にPythonがインストールされます。次のような方法でそれを乗っ取ることができます:
```bash
mv /opt/homebrew/bin/python3 /opt/homebrew/bin/python3.old
cat > /opt/homebrew/bin/python3 <<EOF
#!/bin/bash
# Extra hijack code
/opt/homebrew/bin/python3.old "$@"
EOF
chmod +x /opt/homebrew/bin/python3
```
## 検出

### シールド

[**Shield**](https://theevilbit.github.io/shield/) ([**Github**](https://github.com/theevilbit/Shield)) は、次のようなプロセスインジェクションアクションを**検出およびブロック**するオープンソースアプリケーションです：

- **環境変数の使用**：次の環境変数の存在を監視します：**`DYLD_INSERT_LIBRARIES`**、**`CFNETWORK_LIBRARY_PATH`**、**`RAWCAMERA_BUNDLE_PATH`**、**`ELECTRON_RUN_AS_NODE`**
- **`task_for_pid`** の呼び出し：1 つのプロセスが別のプロセスの**タスクポートを取得**しようとする場合を見つけ、プロセスにコードをインジェクトすることを可能にします。
- **Electron アプリのパラメータ**：誰かが**`--inspect`**、**`--inspect-brk`**、**`--remote-debugging-port`** のコマンドライン引数を使用して Electron アプリをデバッグモードで起動し、それにコードをインジェクトすることができます。
- **シンボリックリンク**または**ハードリンク**の使用：一般的に最も一般的な悪用は、**ユーザ権限でリンクを配置**し、それを**より高い権限**の場所に向けることです。ハードリンクとシンボリックリンクの検出は非常に簡単です。リンクを作成するプロセスがターゲットファイルよりも**異なる権限レベル**を持っている場合、**アラート**を作成します。残念ながら、シンボリックリンクの場合、ブロックは不可能です。リンクの宛先に関する情報が作成前にないためです。これは Apple の EndpointSecuriy フレームワークの制限です。

### 他のプロセスによる呼び出し

[**このブログ投稿**](https://knight.sc/reverse%20engineering/2019/04/15/detecting-task-modifications.html) では、関数 **`task_name_for_pid`** を使用して、他の**プロセスがプロセスにコードをインジェクト**しようとしている情報を取得し、その他のプロセスに関する情報を取得する方法について説明されています。

その関数を呼び出すには、そのプロセスを実行しているユーザーと**同じ uid**である必要があります。または**root**である必要があります（情報を返すだけであり、コードをインジェクトする方法ではありません）。

## 参考文献

- [https://theevilbit.github.io/shield/](https://theevilbit.github.io/shield/)
- [https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f](https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f)

# FISSURE - The RF Framework

**周波数独立SDRベースの信号理解と逆エンジニアリング**

FISSUREは、信号検出と分類、プロトコル発見、攻撃実行、IQ操作、脆弱性分析、自動化、AI/MLのためのフックを備えた、すべてのスキルレベル向けに設計されたオープンソースのRFおよび逆エンジニアリングフレームワークです。このフレームワークは、ソフトウェアモジュール、ラジオ、プロトコル、信号データ、スクリプト、フローベース、参考資料、サードパーティツールの迅速な統合を促進するために構築されました。FISSUREは、ソフトウェアを1か所にまとめ、チームが特定のLinuxディストリビューションのための同じ実績のあるベースライン構成を共有しながら、スムーズに作業を開始できるようにするワークフローの促進者です。

FISSUREに含まれるフレームワークとツールは、RFエネルギーの存在を検出し、信号の特性を理解し、サンプルを収集・分析し、送信および/または注入技術を開発し、カスタムペイロードやメッセージを作成するために設計されています。FISSUREには、識別、パケット作成、ファジングを支援するためのプロトコルおよび信号情報の成長するライブラリが含まれています。オンラインアーカイブ機能があり、信号ファイルをダウンロードし、トラフィックをシミュレートしてシステムをテストするためのプレイリストを構築できます。

フレンドリーなPythonコードベースとユーザーインターフェースにより、初心者はRFおよび逆エンジニアリングに関する人気のツールや技術を迅速に学ぶことができます。サイバーセキュリティやエンジニアリングの教育者は、組み込みの教材を活用したり、フレームワークを利用して自分の実世界のアプリケーションを示すことができます。開発者や研究者は、日常のタスクにFISSUREを使用したり、最先端のソリューションをより広いオーディエンスに公開することができます。FISSUREの認知度と使用がコミュニティで高まるにつれて、その能力の範囲と包含する技術の幅も広がります。

**追加情報**

* [AISページ](https://www.ainfosec.com/technologies/fissure/)
* [GRCon22スライド](https://events.gnuradio.org/event/18/contributions/246/attachments/84/164/FISSURE\_Poore\_GRCon22.pdf)
* [GRCon22論文](https://events.gnuradio.org/event/18/contributions/246/attachments/84/167/FISSURE\_Paper\_Poore\_GRCon22.pdf)
* [GRCon22ビデオ](https://www.youtube.com/watch?v=1f2umEKhJvE)
* [ハックチャットのトランスクリプト](https://hackaday.io/event/187076-rf-hacking-hack-chat/log/212136-hack-chat-transcript-part-1)

## 始めに

**サポートされている**

FISSURE内には、ファイルナビゲーションを容易にし、コードの冗長性を減らすために3つのブランチがあります。Python2\_maint-3.7ブランチは、Python2、PyQt4、およびGNU Radio 3.7を中心に構築されたコードベースを含み、Python3\_maint-3.8ブランチはPython3、PyQt5、およびGNU Radio 3.8を中心に構築され、Python3\_maint-3.10ブランチはPython3、PyQt5、およびGNU Radio 3.10を中心に構築されています。

|   オペレーティングシステム   |   FISSUREブランチ   |
| :--------------------------: | :----------------: |
|  Ubuntu 18.04 (x64)        | Python2\_maint-3.7 |
| Ubuntu 18.04.5 (x64)       | Python2\_maint-3.7 |
| Ubuntu 18.04.6 (x64)       | Python2\_maint-3.7 |
| Ubuntu 20.04.1 (x64)       | Python3\_maint-3.8 |
| Ubuntu 20.04.4 (x64)       | Python3\_maint-3.8 |
|  KDE neon 5.25 (x64)       | Python3\_maint-3.8 |

**進行中（ベータ）**

これらのオペレーティングシステムはまだベータステータスです。開発中であり、いくつかの機能が欠けていることが知られています。インストーラー内の項目は、既存のプログラムと競合する可能性があるか、ステータスが削除されるまでインストールに失敗することがあります。

|     オペレーティングシステム     |    FISSUREブランチ   |
| :------------------------------: | :-----------------: |
| DragonOS Focal (x86\_64)       |  Python3\_maint-3.8 |
|    Ubuntu 22.04 (x64)          | Python3\_maint-3.10 |

注：特定のソフトウェアツールはすべてのOSで動作しません。[ソフトウェアと競合](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Help/Markdown/SoftwareAndConflicts.md)を参照してください。

**インストール**
```
git clone https://github.com/ainfosec/FISSURE.git
cd FISSURE
git checkout <Python2_maint-3.7> or <Python3_maint-3.8> or <Python3_maint-3.10>
git submodule update --init
./install
```
これは、インストールGUIを起動するために必要なPyQtソフトウェア依存関係をインストールします。

次に、オペレーティングシステムに最も適したオプションを選択します（OSがオプションに一致する場合は自動的に検出されるはずです）。

|                                          Python2\_maint-3.7                                          |                                          Python3\_maint-3.8                                          |                                          Python3\_maint-3.10                                         |
| :--------------------------------------------------------------------------------------------------: | :--------------------------------------------------------------------------------------------------: | :--------------------------------------------------------------------------------------------------: |
| ![install1b](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/install1b.png) | ![install1a](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/install1a.png) | ![install1c](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/install1c.png) |

既存の競合を避けるために、クリーンなオペレーティングシステムにFISSUREをインストールすることをお勧めします。FISSURE内のさまざまなツールを操作する際のエラーを避けるために、すべての推奨チェックボックス（デフォルトボタン）を選択してください。インストール中に複数のプロンプトが表示され、主に昇格された権限とユーザー名を要求されます。項目の最後に「Verify」セクションが含まれている場合、インストーラーはその後のコマンドを実行し、コマンドによってエラーが発生したかどうかに応じてチェックボックス項目を緑または赤で強調表示します。「Verify」セクションのないチェック済み項目は、インストール後も黒のままになります。

![install2](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/install2.png)

**使用法**

ターミナルを開いて、次のコマンドを入力します：
```
fissure
```
Refer to the FISSURE Help menu for more details on usage.

## Details

**Components**

* ダッシュボード
* セントラルハブ (HIPRFISR)
* ターゲット信号識別 (TSI)
* プロトコル発見 (PD)
* フローネットワーク & スクリプトエグゼキュータ (FGE)

![components](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/components.png)

**Capabilities**

| ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/detector.png)_**信号検出器**_ | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/iq.png)_**IQ操作**_      | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/library.png)_**信号ルックアップ**_          | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/pd.png)_**パターン認識**_ |
| --------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/attack.png)_**攻撃**_           | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/fuzzing.png)_**ファジング**_         | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/archive.png)_**信号プレイリスト**_       | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/gallery.png)_**画像ギャラリー**_  |
| ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/packet.png)_**パケット作成**_   | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/scapy.png)_**Scapy統合**_ | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/crc\_calculator.png)_**CRC計算機**_ | ![](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Icons/README/log.png)_**ログ記録**_            |

**Hardware**

The following is a list of "supported" hardware with varying levels of integration:

* USRP: X3xx, B2xx, B20xmini, USRP2, N2xx
* HackRF
* RTL2832U
* 802.11アダプタ
* LimeSDR
* bladeRF, bladeRF 2.0 micro
* Open Sniffer
* PlutoSDR

## Lessons

FISSURE comes with several helpful guides to become familiar with different technologies and techniques. Many include steps for using various tools that are integrated into FISSURE.

* [Lesson1: OpenBTS](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson1\_OpenBTS.md)
* [Lesson2: Luaディセクター](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson2\_LuaDissectors.md)
* [Lesson3: Sound eXchange](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson3\_Sound\_eXchange.md)
* [Lesson4: ESPボード](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson4\_ESP\_Boards.md)
* [Lesson5: ラジオソン追跡](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson5\_Radiosonde\_Tracking.md)
* [Lesson6: RFID](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson6\_RFID.md)
* [Lesson7: データ型](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson7\_Data\_Types.md)
* [Lesson8: カスタムGNUラジオブロック](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson8\_Custom\_GNU\_Radio\_Blocks.md)
* [Lesson9: TPMS](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson9\_TPMS.md)
* [Lesson10: アマチュア無線試験](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson10\_Ham\_Radio\_Exams.md)
* [Lesson11: Wi-Fiツール](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/Lessons/Markdown/Lesson11\_WiFi\_Tools.md)

## Roadmap

* [ ] さらなるハードウェアタイプ、RFプロトコル、信号パラメータ、分析ツールを追加
* [ ] より多くのオペレーティングシステムをサポート
* [ ] FISSUREに関するクラス資料を開発 (RF攻撃、Wi-Fi、GNUラジオ、PyQtなど)
* [ ] 選択可能なAI/ML技術を用いた信号コンディショナー、特徴抽出器、信号分類器を作成
* [ ] 不明な信号からビットストリームを生成するための再帰的復調メカニズムを実装
* [ ] メインのFISSUREコンポーネントを一般的なセンサーノード展開スキームに移行

## Contributing

Suggestions for improving FISSURE are strongly encouraged. Leave a comment in the [Discussions](https://github.com/ainfosec/FISSURE/discussions) page or in the Discord Server if you have any thoughts regarding the following:

* 新機能の提案とデザイン変更
* インストール手順を含むソフトウェアツール
* 新しいレッスンや既存のレッスンの追加資料
* 興味のあるRFプロトコル
* 統合のためのさらなるハードウェアとSDRタイプ
* PythonでのIQ分析スクリプト
* インストールの修正と改善

Contributions to improve FISSURE are crucial to expediting its development. Any contributions you make are greatly appreciated. If you wish to contribute through code development, please fork the repo and create a pull request:

1. プロジェクトをフォーク
2. フィーチャーブランチを作成 (`git checkout -b feature/AmazingFeature`)
3. 変更をコミット (`git commit -m 'Add some AmazingFeature'`)
4. ブランチにプッシュ (`git push origin feature/AmazingFeature`)
5. プルリクエストを開く

Creating [Issues](https://github.com/ainfosec/FISSURE/issues) to bring attention to bugs is also welcomed.

## Collaborating

Contact Assured Information Security, Inc. (AIS) Business Development to propose and formalize any FISSURE collaboration opportunities–whether that is through dedicating time towards integrating your software, having the talented people at AIS develop solutions for your technical challenges, or integrating FISSURE into other platforms/applications.

## License

GPL-3.0

For license details, see LICENSE file.

## Contact

Join the Discord Server: [https://discord.gg/JZDs5sgxcG](https://discord.gg/JZDs5sgxcG)

Follow on Twitter: [@FissureRF](https://twitter.com/fissurerf), [@AinfoSec](https://twitter.com/ainfosec)

Chris Poore - Assured Information Security, Inc. - poorec@ainfosec.com

Business Development - Assured Information Security, Inc. - bd@ainfosec.com

## Credits

We acknowledge and are grateful to these developers:

[Credits](https://github.com/ainfosec/FISSURE/blob/Python3\_maint-3.8/CREDITS.md)

## Acknowledgments

Special thanks to Dr. Samuel Mantravadi and Joseph Reith for their contributions to this project.

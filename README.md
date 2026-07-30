# torabo-tsuki LP 用 ZMK ファームウェア

[torabo-tsuki LP](https://github.com/sekigon-gonnoc/torabo-tsuki-lp) 用の ZMK ファームウェアです。KeyBall39 のキーマップを移植しています。

書き込みの際は、`_central` がついている uf2 をトラックボールがついている側に、`_peripheral` がついている uf2 を反対側に書き込んでください。キーマップは keymap-editor および zmk-studio で編集できます。

## ファームウェアのビルドと書き込み手順

### 1. ビルドする（GitHub Actions による自動ビルド）

このリポジトリは GitHub Actions で自動ビルドされます。keymap-editor や GitHub 上で `config/keymap.keymap` などのファイルを編集してコミット（master ブランチに push）すると、ビルドが自動で開始します。手動で実行したい場合は、リポジトリの Actions タブから Build ワークフローを選び Run workflow を押してください。

### 2. uf2 ファイルをダウンロードする

Actions タブを開き、一番上の（最新の）ビルドをクリックします。ビルドが成功（緑のチェック）していることを確認したら、ページ下部の Artifacts 欄にある firmware をクリックして zip をダウンロードし、展開します。zip の中には次の uf2 が含まれます。`torabo_tsuki_lp_right_central` はトラックボールがある右側（Central）用、`torabo_tsuki_lp_left_peripheral` は左側（Peripheral）用、`settings_reset` は設定初期化用（ペアリング情報が壊れたとき等に使用）です。

### 3. キーボードに書き込む

まずスイッチを OFF にした状態で、片方の基板を USB ケーブルで PC に接続します。`BLEMICROPRO` という名前のストレージが表示されたら、対応する uf2 をそのストレージにコピーします。トラックボールがある側には `_central`、反対側には `_peripheral` を書き込んでください。例えば右手にトラックボールがある場合、右手に `torabo_tsuki_lp_right_central`、左手に `torabo_tsuki_lp_left_peripheral` を書き込みます。書き込み後は一度 USB を抜き、スイッチを ON にしてから接続し直すとキーボードとして動作します。

うまく動かない場合は `settings_reset` を書き込み、一度再起動（USB を抜き差し）してから、改めて通常のファームウェアを書き込んでください。

## キーマップの主な機能

スクロールは、スクロールレイヤー（レイヤー3）でトラックボールがスクロールになります（左右方向は反転済み）。スクロールには慣性（惰性スクロール）がついていて、ボールを弾いて手を離すと iOS のようにゆっくり減衰しながらスクロールが続きます。トラックボール感度は右側 overlay の `zip_xy_scaler` で調整しています。Bluetooth 切り替えは、レイヤー3 の右上に `BT_SEL 0/1/2`（プロファイル 1/2/3 切替）と `BT_CLR`（現在のプロファイルのペアリング解除）を配置しています。Esc は、左 Control とレイヤー1（数字入力レイヤー）キーの同時押しで入力できます（コンボ）。

## 慣性スクロールの調整

慣性スクロールは [zmk-input-processor-scroll-inertia](https://github.com/mjmjm0101/zmk-input-processor-scroll-inertia) で実装しています。設定は `snippets/scroll-inertia/scroll-inertia.overlay` にあり、感触の調整はこのファイルの `scroll_inertia` ノードにプロパティを足して行います。よく使うのは次の3つです。

- **慣性が長すぎる / いつまでも滑る** → `decay-fast` / `decay-slow` / `decay-tail` を `990` から `980` 程度に下げる（3つとも同じ値にすると単一カーブ）。または `friction` を `35` から `100` 程度に上げる。
- **弾いていないのに慣性が発動する** → `start`（既定 `40`）と `move`（既定 `80`）を上げる。
- **弱く弾いたときに慣性が乗らない** → `start` を下げる。

なお慣性プロセッサは ZMK のマウス HID を扱うため central 側でしかビルドできません。そのため shield の overlay ではなく、`build.yaml` の central ビルドにだけ付ける `scroll-inertia` snippet として組み込んでいます。左手側にトラックボールがある構成（`_left_central`）で慣性を使う場合は、`torabo_tsuki_lp_left.overlay` にスクロールレイヤー用の `scroller` ノードを追加したうえで、`build.yaml` の該当ビルドに `scroll-inertia` を足してください。

## キーマップの編集方法

keymap-editor を使う場合は、このリポジトリをフォーク／クローンして読み込んで編集してください。keymap-editor の仕様上 `config/info.json` は L サイズ用になっているため、XS サイズには存在しないキーも表示されます。編集後は上記のビルド手順で新しい uf2 を作成して書き込んでください。

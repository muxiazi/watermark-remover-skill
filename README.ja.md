# Watermark Remover Skill（チーム版インストールガイド）

[🇨🇳 简体中文](./README.md) | [🇬🇧 English](./README.en.md) | **🇯🇵 日本語** | [🇰🇷 한국어](./README.ko.md)


Claude / Claude Code などの AI エージェントから「布衣画像一括ウォーターマーク除去ソフト」を自動呼び出しして画像を処理できるようにします。
ソフト本体がまだない場合は「布衣图片批量去水印软件」で検索して入手してください。

---

## 仕組み(インストールの前に理解してください)

```
[エージェント] ──画像投入──► [入力ディレクトリ] ──監視で自動起動──► [GUI が処理] ──書き出し──► [出力ディレクトリ] ──エージェント読込
```

エージェントは GUI のボタンを直接操作するのではなく、**GUI が監視している入力ディレクトリに画像をコピー**し、**結果が現れるまで出力ディレクトリをポーリング**します。

そのため GUI は**常時起動**しておき、**「📡 ディレクトリ監視」ボタンが ON 状態**である必要があります。

---

## インストール手順

### 会員登録ページ: https://buyitanan.com/bu_yi_tu_pian_pi_liang_qu_shui_yin

### 1. GUI アプリの準備

> **注意:** GUI 本体は現在中国語のみです。以下の日本語ボタン名は参考訳で、アプリ内の実際のラベルは中国語(括弧内に記載)です。

- 「布衣画像一括ウォーターマーク除去ソフト」(布衣图片批量去水印软件)をインストールして起動
- アカウントにログイン(**無料会員ではダメ**、無料会員には監視権限がありません)
- GUI 上で:
  - 「入力フォルダ選択」(选择输入文件夹)をクリックしてディレクトリを指定(専用フォルダを新規作成するのを推奨、例: `D:\watermark_in`)
  - 「出力フォルダ選択」(选择输出文件夹)をクリックして別のディレクトリを指定(例: `D:\watermark_out`)
  - 「📡 ディレクトリ監視: OFF」(📡 监控目录:关闭)ボタンをクリック、緑色の「📡 ディレクトリ監視: ON」(📡 监控目录:开启)に変わるはず
- プログラムを起動したままにする(最小化は OK、ただし閉じてはいけない)

### 2. 本 skill のインストール

```bash
# このディレクトリをチームメンバーのローカル任意の場所にコピー
# 例: ~/skills/watermark-remover/
```

必要なのは Python 3.8+ のみ。**サードパーティ依存なし**。

### 3. 設定ファイルの作成

```bash
cp config.example.json config.json
```

`config.json` を編集し、`input_dir` / `output_dir` を手順 1 で GUI 内で実際に選んだ 2 つのディレクトリに変更します。**両者は完全に一致している必要があります**。

### 4. セルフチェック実行

```bash
python watermark_remover.py check
```

✅ が出力されれば、入力/出力ディレクトリの両方が認識されています。

さらにテスト画像 1 枚で実際の動作確認を行うには:

```bash
python watermark_remover.py check --sample test.png --timeout 60
```

60 秒以内に ✅ が出れば、エージェント → GUI までのパイプライン全体が正常につながっていることを意味します。

---

## Claude 向けデプロイ方法

`watermark-remover-skill/` ディレクトリ全体を Claude プロジェクトのナレッジベース / Skill ライブラリにアップロードするだけです。Claude は `SKILL.md` の frontmatter(`name` / `description`)を読み取り、ユーザーがウォーターマーク除去タスクに言及した際に自動で skill を呼び出します。

Claude Code などの CLI エージェントの場合は、エージェントがアクセスできる場所にディレクトリを置き、設定ファイルのパスをエージェントに伝えるか、環境変数を設定します:

```bash
export WATERMARK_REMOVER_CONFIG=~/skills/watermark-remover/config.json
```

---

## よく使うコマンド早見

```bash
# セルフチェック
python watermark_remover.py check
python watermark_remover.py check --sample test.png

# 1 枚処理
python watermark_remover.py process input.jpg

# 複数枚処理
python watermark_remover.py process a.jpg b.png c.webp

# ディレクトリ全体を処理
python watermark_remover.py process ~/photos_to_clean/

# 大量バッチ + カスタムタイムアウト + 結果エクスポート
python watermark_remover.py process ~/big_batch/ --timeout 1800 --json-out result.json

# タスクサブディレクトリを作らない(入力ルートに直接置く)
python watermark_remover.py process input.jpg --no-subdir

# コピーではなく移動(元ファイルが削除されます!)
python watermark_remover.py process input.jpg --move
```

---

## チーム協業の注意事項

複数のチームメンバーが同じ GUI インスタンス(同じマシン)を共有する可能性があるため、以下を取り決めておきましょう:

1. **必ずタスクサブディレクトリで隔離する**: デフォルトでは毎回 UUID サブディレクトリが自動生成されるので、複数人の並列実行でも衝突しません。**安易に `--no-subdir` を使わないでください**。
2. **お互いの永続化記録を消さない**: GUI の「処理済みリストをクリア」での「Yes to All」は `processed_files.json` をクリアしてしまい、他人のタスクが再処理される可能性があります。**実行前に必ずチームと同期してください**。
3. **タイムアウトは十分に**: 1 枚は通常 3〜30 秒ですが、大きな画像のバッチ処理は数分かかることもあります。エージェントが pending を報告した場合、まず GUI がまだ動いているかを確認してから諦めるか timeout を延長するかを決めます。
4. **「完了後クリーンアップ」のルールを推奨**: 処理完了後、エージェントにタスクサブディレクトリを入力ディレクトリから削除させ、環境を綺麗に保つとよいでしょう。

---

## ファイル一覧

```
watermark-remover-skill/
├── SKILL.md              # Claude/エージェントが読む指示ファイル(frontmatter を変更しないこと)
├── README.md             # 本ファイル、人間向け
├── watermark_remover.py  # コアスクリプト(CLI/ライブラリ両用)
├── config.example.json   # 設定ファイルのテンプレート
└── config.json           # 実際の設定(初回インストール時にテンプレートからコピーして編集)
```

---

## フィードバックと拡張

- 「GUI に依存しない」CLI バージョン(ソフトを介さず直接アルゴリズムを呼び出す)を作りたい場合、元の GUI コードから `InpaintWorker` の処理ロジックを抽出する必要があります。本 skill ではカバーしていません。
- Webhook / HTTP API トリガーを追加したい場合は、`WatermarkRemoverClient` クラスをベースに拡張できます。

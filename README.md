# Zenn Contents

- [📘 How to use](https://zenn.dev/zenn/articles/zenn-cli-guide)
- [📘 Markdown guide](https://zenn.dev/zenn/articles/markdown-guide)

## 📝 Article

- 👇 ファイルの配置ルール
  - 1つの記事の内容は、1つのマークダウンファイル（`◯◯.md`）で管理します。
  - ファイルはarticlesという名前のディレクトリ内に含める必要があります。

```
.
└─ articles
   ├── example-article1.md
   └── example-article2.md
```

- 👇 新しい記事を作成する

```
$ npx zenn new:article
```

- 👇 記事のURLの一部となるslugを指定して作成

```
$ npx zenn new:article --slug my-awesome-article
```

### 🖌 記事の書き方

- ファイルの上部には---に挟まれる形で記事の設定（Front Matter）が含まれています。ここに記事のタイトル（title）やトピックス（topics）などをyaml形式で指定することになります。

```
---
title: ""        # 記事のタイトル
emoji: "😸"      # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech"     # tech: 技術記事 / idea: アイデア記事
topics: []       # タグ。["markdown", "rust", "aws"]のように指定する
published: true  # 公開設定（falseにすると下書き）
---

ここから本文を書く
```

- コマンド実行時に記事のFront Matterをオプションで指定することもできます。

```
$ npx zenn new:article --slug 記事のスラッグ --title タイトル --type idea --emoji ✨
```

**※ slugは`a-z0-9`とハイフン`-`の12〜50字の組み合わせにする必要があります**

## 👁‍🗨 ブラウザでプレビュー表示

- 👇 表示をプレビューする

```
$ npx zenn preview
```

- 👇 別のPORTを指定

```
$ npx zenn preview --port 3333
```

- 👇 ホットリロードのOFF

```
$ npx zenn preview --no-watch
```

## 📚 Book

- 👇 本のディレクトリ構成

```
.
├─ articles
└─ books
   └── my-awesome-book  # slug
       ├── config.yaml  # 本の設定
       ├── cover.png    # カバー画像（JPEGかPNG）
       ├── example1.md  # チャプターのslug
       ├── example2.md  # チャプターのslug
       └── example3.md  # チャプターのslug
```

- 👇 新しい本を作成する

```
$ npx zenn new:book
```

- 👇 本のURLの一部となるslugを指定して作成

```
$ npx zenn new:book --slug my-awesome-book
```

## ⚙️ Zenn CLIのアップデート

- Zenn CLIの表示がzenn.devと異なるときやCLI利用時に更新通知が表示されたときは下記のコマンドでアップデートを行ってください。

```
$ npm install zenn-cli@latest
```

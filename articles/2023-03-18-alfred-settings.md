---
title: "Alfredの有料版を買ったので作業効率を上げるための各種設定メモ"
emoji: "🪬"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["alfred"]
published: true
---

## はじめに

以前は、[Alfred](https://www.alfredapp.com/) の無料版と [Clipy](https://clipy-app.com/) というアプリケーションを併用していましたが、ワークフローを導入したいと思い、Alfred の有料版を購入しました。

今回は有料版を購入したのを機に、環境設定の見直しやワークフローの導入・カスタマイズを行い作業効率の改善をしたので、その内容をメモがてら紹介します。

## 無料版と有料版の違い

Alfred の無料版には使用できない機能がいくつかあります。
その中でも、有料版でしか使用できない強力な機能は以下の通りです。

- クリップボード履歴
- スニペット
- 1Password連携
- ワークフロー

https://webrandum.net/alfred4-comparison-powerpack/

1Password は使用していないため、クリップボードとスニペットは Clipy というアプリで代用することができました。

ただ、Notion から気軽にメモを取るために、ワークフローを使用したかったため、有料版を購入しました。

ワークフローを必要としない場合は、無料版でも十分だと思います。

## 基本設定

### Clipboard の Hotkey の設定

Clipy を使っていたときは `⇧ + ⌘ + V (シフト + コマンド + V)` だったので、それに合わせて Alfred の Hotkey を変更しました。

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/c19acc1ceebf4e86aa354a8bcb05b246/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202023-03-18%2015.59.04.png)

### カンマ付きの数字も電卓で計算できるようにする設定

デフォルトの設定ではカンマ付きの数値は文字列の扱いとなって電卓で計算が出来ません。

カンマ付きの数値をコピペで計算したいケースがちょいちょいあったので、電卓設定の Input にある「Ignore thousands grouping separator」にチェックを付けると、カンマ付きの数値でも計算してくれるようになります。

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/de874889eae64e239e6aa89e838201a2/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202023-03-18%2016.04.13.png)

### Alfred 起動時に自動で英語入力にする設定

Alfred のコマンドは英語となっているため、キーボードが日本語になっていると、 keyword を入力するときに英語に入力切り替えをする手間が増えます。

Advanced の Force Keyboard を `Alphanumeric` に設定しておくと、Alfred 起動時には必ず英語入力になります。

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/3985e50f0a68425587a55260523c5416/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202023-03-18%2016.22.24.png)

### ターミナルアプリを iTerm2 に変更

Alfred のターミナル/シェル機能はターミナルを開くだけではなく、Alfred からターミナルで使えるコマンドを実行することができます。

ただ、iTerm2 は Hotkey があり、直接 iTerm2 を開いて実行するため実際は設定不要な気がします。というか不要ですが、まぁどうせならということで設定します。

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/53cedce004b446eba8122fe1c468e238/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202023-03-18%2016.30.22.png)

デフォルトでは Prefix が `>` となっているので、変えたい人は好みで変えて下さい。

iTerm2 の設定ですが、 Application を `Terminal` から `custom` に変更します。

`custom` の場合はスクリプトを入力する必要があるので、ターミナルから `curl` コマンドを実行してスクリプトを取得します。

```bash
curl --silent 'https://raw.githubusercontent.com/vitorgalvao/custom-alfred-iterm-scripts/master/custom_iterm_script.applescript' | pbcopy
```

https://github.com/vitorgalvao/custom-alfred-iterm-scripts

クリップボードにコピーされているので、 Alfred にペーストします。

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/af145fcffaec483ab3b3dd750cd011f1/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202023-03-18%2016.36.34.png)

これで、開くターミナルアプリが iTerm2 になりました。

### 設定ファイルを iCloud に同期させる設定

PC を変えたときでも環境設定は同じにしたいです。

Alfred は設定ファイルの同期やバックアップの作成ができるので Dropbox や iCloud に同期させる設定しましょう。

Advanced の Syncing にある「Set preferences folder...」を選択してフォルダを選択します。

自分の場合は iCloud の Documents を設定しています。

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/43409e4eaa08461b969d38ae20232482/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202023-03-18%2016.43.06.png)

## ワークフローの設定

Alfred にはワークフローという有料版でしか使用できない機能があり、この機能を使うと色々と作業を効率化させることができます。

Alfred のワークフローは公式が [Alfred Gallery](https://alfred.app/) から提供していたり、誰かが作っていたり、自作で作ったりすることができます。

色々と便利なワークフローを入れたので、気になったのがあったら試してみて下さい。

### FastNotion（自作）

FastNotion は Notion 公式アンバサダーの [Tsuburaya](https://twitter.com/___35d) さんが紹介しているワークフローです。

このワークフローは Alfred で入力したメモを Notion の指定したデータベースに保存することができる自作のワークフローです。

具体的な仕組みとしては、Alfred で入力したテキストを Notion API を使用してデータベースに保存するスクリプトを実行させています。

Notion は好きでいつも使っているのですが、簡易的なメモを取るときは結構めんどくさくて悩みだったので、このワークフローを入れてからメモが楽になりました。

具体的な設定方法については YouTube で紹介してくれているので、そちらを参考にしてみて下さい。

https://www.youtube.com/watch?v=Hoge2RZCrl0

ただ、動画で設定しているときの Notion API のバージョンが古いため、実際のコードの部分は最新の公式のドキュメントを参考に置き換えて下さい。

https://developers.notion.com/reference/post-page

### Colors

RGB, Hex のカラーコードを変換・表示してくれるワークフローです。

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/211a21ceae70436e8947822a32f56b3a/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202023-03-18%2017.25.22.png)

http://www.packal.org/workflow/colors

### Terminal Finder

ターミナルから Finder への移動やその反対の作業をよく行う人には便利です。

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/ffa17f1c34f34835a6c70bb9358a8655/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202023-03-18%2017.27.13.png)

https://github.com/LeEnno/alfred-terminalfinder

### Google Suggest

g + keyword で検索したキーワードの検索候補をサジェスト表示してくれます。

いちいちブラウザから検索する必要がないので楽です。

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/20f8a035a63e45b89bc09f01c1759a0e/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202023-03-18%2018.04.04.png)

https://alfred.app/workflows/alfredapp/google-suggest/

### Deepl-Translate

Alfred で Deepl 翻訳が使えます。

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/77b18588427545ac896a18f2df53810c/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202023-03-18%2018.08.52.png)

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/fd7fc798040c4b9dac3b388a5e12c54a/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202023-03-18%2018.09.03.png)

https://github.com/AlexanderWillner/deepl-alfred-workflow2

デフォルト設定では、優先言語がドイツ語 (DE) と英語 (EN) で、訳文の言語が英語になっています。

そのため、日本語 ⇔ 英語とするための設定を行います。

実際の設定方法はこちらの記事を参考に。

https://tomi-kun.hatenablog.com/entry/2022/10/13/001546

### Notion Search

Notion のページ検索を Alfred 上からできるワークフローです。

Notion のページを開きたいときは、 Notion アプリを開いてから `⌘ + P` でページ検索を行っていましたが、Alfred から一発でページ検索ができるようになります。

ただ、1 つのワークスペースしか設定できないので、複数ワークスペースを検索したい場合は実現出来なそうです。

設定が結構大変ですが、[公式の手順](https://github.com/wrjlewis/notion-search-alfred-workflow#get-your-workflow-variables)を参考に設定ができるので確認してください。

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/c0955f2c8a4b4c5fa41808758d671a30/S__179093509.jpg)

https://github.com/wrjlewis/notion-search-alfred-workflow

### GitHub

Github のリポジトリや PullRequest, issue などを開くことができるワークフローです。

複数リポジトリがある場合や、PR を作成するさいにすぐに画面を開くことができるので便利です。

こちらは公式の Setup 手順に従って設定をして下さい。

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/50c11ae856c44af891a5ef00f8939d3c/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202023-03-18%2018.35.41.png)

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/71ad0ea0265645c4ae12b73c9b675d82/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202023-03-18%2018.34.57.png)

https://github.com/gharlan/alfred-github-workflow

## その他のランチャーアプリ

Alfred の有料版を購入してから知ったのですが、Raycast というランチャーアプリも便利みたいです。

https://www.raycast.com/

しかも最近 Raycast AI が導入されたので、ランチャーから ChatGPT のように AI が動いてくれます。

使ったことはないのですが、便利そうだしかっこいいので Alfred を使ってみようと思ってる人は是非比較して試してみて下さい。

## 最後に

今までは Spotlight より便利なランチャーアプリとしてしか使用していませんでしたが、有料版の購入をきっかけに設定を見直したりワークフローを導入したことで作業効率が上がりました。

他にもいろんなワークフローがあるので、今後も便利なものがあれば導入していこうと思います。

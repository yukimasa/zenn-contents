---
title: "Next.js × microCMSでZennのマークダウンを表示するブログの作り方"
emoji: "👏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nextjs", "microcms", "zenn", "markdown"]
published: false
---

## はじめに

このサイトは Next.js で構築し、記事コンテンツは microCMS で管理しています。

基本的なスタイリングについては chakra-ui というコンポーネントライブラリを使用しているため、細かい部品ごとのスタイリングはライブラリに任せています。

https://chakra-ui.com/

記事は markdown 記法のものを microCMS から取得し、zenn が提供しているライブラリを使用して HTML へのパース及びスタイリングを行っています。

このライブラリを導入することで、zenn 独自のマークダウン記法も使用できる上に、リンクをカードにしてくれたり、Twitter や Youtube の埋め込みも勝手にやってくれます。

今回の記事で自分と同じような構成を考えている人の参考になれば良いなと思います。

サイトの構築自体はこちらを参考に

https://www.yukimasa.net/posts/created-jamstack-personal-site

## 注意事項

- microCMS のリッチエディタを使用している場合は、markdown 記法に変換したものを新たなフィールドに移動する必要がある
  - 記事が多いほど手間がかかる
- 執筆は別のエディタで行い、コンテンツを mircoCMS に貼り付ける方法がやりやすいが、その場合は画像だけ microCMS にアップしておく必要がある
- zenn の独自記法に依存するため、他の箇所でコンテンツ利用する場合は使用できない
  - スタイルの恩恵だけ受けて、独自記法は使用しないようにするなど工夫が必要

## 使用するライブラリ

### zenn-markdown-html

zenn 独自の記法を含む markdown を HTML に変換するためのライブラリ。

https://github.com/zenn-dev/zenn-editor/tree/main/packages/zenn-markdown-html

### zenn-content-css

zenn-markdown-html で markdown から変換された HTML に適用するためのCSS。

https://github.com/zenn-dev/zenn-editor/tree/main/packages/zenn-content-css

### zenn-embed-elements

markdown の zenn 独自の埋め込み要素をHTMLに変換するためのライブラリだが、現在は KaTeX による数式のレンダリングのためにのみ利用している。

https://github.com/zenn-dev/zenn-editor/tree/main/packages/zenn-embed-elements

## バージョン

- Next.js: 13.1.6
- React: 18.2.0
- zenn-content-css: 0.1.137
- zenn-embed-elements: 0.1.137
- zenn-markdown-html: 0.1.137

## microCMS の記事投稿をリッチエディタからテキストエリアに変更する

新規で microCMS を構築する場合は markdown 用にテキストエリアのフィールドを用意すれば良いのですが、すでにリッチエディタで運用している場合は引っ越しが必要です。

方法としては、現在のリッチエディタを生かしたまま、新たにテキストエリアのフィールドを新規追加します。

リッチエディタの内容を markdown 記法に変換し、作成したテキストエリアのフィールドに追加します。

変換方法は何でもいいのですが、自分の場合はリッチエディタのコンテンツを Notion のページにコピペして、そのページを markdown でエクスポートしたものをテキストエリアに貼り付けていました。

そうすることで画像のリンク先も変更することなく引っ越しできるのですが、他に方法があれば何でも良いです。

実装が完了したら不要になったリッチエディタは削除しましょう。

## Zennのマークダウン記法を導入

導入自体はすごい簡単で、こちらの記事を参考に実装してください。

https://zenn.dev/team_zenn/articles/intro-zenn-markdown

ただ、上記の記事で実装したときに、リンクカードやTwitterの埋め込みが動かない事象が発生しました。

`_document.tsx` に script を埋め込む用に記載がありますが、`_document.tsx` は SSR のみの実行なので、 `pages` ごとに以下の `Head` タグを追加しました。

```tsx
import Head from "next/head";

// ..省略..

<Head>
  <script src="https://embed.zenn.studio/js/listen-embed-event.js"></script>
</Head>;
```

## Zennのマークダウン記法

https://zenn.dev/zenn/articles/markdown-guide

zenn は基本的なマークダウン記法に加えて、アコーディオンやメッセージなど独自のマークダウン記法も用意しています。

:::message
メッセージをここに
:::

:::details アコーディオン
表示したい内容
:::

また、コードブロックは言語指定とファイル名の表示も可能です。

```ts:index.ts
const hoge: string = "Hello, world!"
console.log(hoge)
```

シンタックスハイライトには Prism.js を使用しており、対応言語はこちらから確認できます。

https://prismjs.com/#supported-languages

zenn 記法は便利なのですが、 headlessCMS のコンテンツなのでなるべく独自記法には依存させたくは無いです。

が、現状このサイト以外でコンテンツを使用する予定は無いので、ひとまず気にせず使ってます。

## 記事執筆について

もともとは microCMS のリッチエディタを使用していましたが、基本的には Notion で書いた記事を貼り付けて、画像だけ別途アップロードする運用でした。

今は執筆時に markdown プレビューを見ながら書きたいので、Visual Studio Code の zenn 拡張を使用して記事を書いています。

ただ、画像だけは変わらず microCMS にアップロードしてそのリンクを使用しているので、まだ完全ではないなといったところです。

https://zenn.dev/negokaz/articles/aa4e12b76d516597a00e

## まとめ

もともとは拾ってきた適当な css を貼り付けて、シンタックスハイライトは highlight.js というライブラリを使用しており、スタイルの改善をしたいなといったと思ったところから始まりました。

結果的に執筆も markdown に代わり、スタイリングも zenn と同じで見やすくなったので良かったです。

まだ改善箇所はありますが、このサイトを遊び場として色々と実装していきたいなと思います。

---
title: "Tocbot を使用し Next.js で構築したブログに目次を追加する"
emoji: "📋"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nextjs"]
published: false
---

この記事は [ミライトデザイン Advent Calendar 2023](https://qiita.com/advent-calendar/2023/miraito-inc) の 3 日目の記事です。

2 日目に書いたこちらの記事に続いて今回も書いていきます。

https://www.yukimasa.net/posts/auto-switch-git-user-per-repo

## はじめに

この記事では、 Next.js で構築したブログに Tocbot というライブラリを使用して、目次を追加する方法について紹介します。

[zenn](https://zenn.dev/) の目次に使用しているライブラリで、このサイトでも実際に Tocbot を使用して作成しています。

https://zenn.dev/link/comments/4d10c8dd8315e8

なお、 Next.js でのブログ構築についてはこの記事では紹介しません。

このサイトを構築したときの記事が別でありますので、気になる方はこちらを見てみてください。

https://www.yukimasa.net/posts/created-jamstack-personal-site/

## Tocbotとは

https://tscanlin.github.io/tocbot/

Tocbot は html の見出しタグ ( h1, h2 など ) から、自動的に目次 ( Table of Contents ) を生成するライブラリです。

また、目次はリンクになっているため任意の見出しにジャンプしたりすることができますし、スタイルも自由につけることができます。

## 使用方法

使い方はとても簡単なので、以下の手順を見て試してみてください。

### Tocbot のインストール

まずは以下のコマンドを実行してインストールしましょう。

```sh-session
❯ yarn add tocbot
```

or

```sh-session
❯ npm install --save tocbot
```

### Toc コンポーネントの作成

```tsx
import { useEffect } from "react";
import tocbot from "tocbot";

export const Toc = () => {
  useEffect(() => {
    tocbot.init({
      tocSelector: ".toc", //　目次を追加する class 名
      contentSelector: ".content", // 目次を取得するコンテンツの class 名
      headingSelector: "h1, h2, h3, h4", // 目次として取得する見出しタグ
    });

    // 不要となったtocbotインスタンスを削除
    return () => tocbot.destroy();
  }, []);

  return <nav className="toc" />;
};
```

使い方としては、 useEffect 内で `tocbot.init()` を実行して初期化処理を実行します。

init の引数に設定値を入れることで目次の各設定ができます。

色々なオプションがあるので、公式の [API > Options](https://tscanlin.github.io/tocbot/#options) から確認してみてください。

なお、このサイトではヘッダーを固定しているため、目次のアンカー位置がズレてしまいます。
そのため以下の設定を追記してヘッダーの高さ分を調整します。

また、 必要な見出しタグのみを目次として設定したいため、 `headingSelector` も修正します。

```diff tsx
export const Toc = () => {
  useEffect(() => {
    tocbot.init({
      tocSelector: ".toc", //　目次を追加する class 名
      contentSelector: ".content", // 目次を取得するコンテンツの class 名
-     headingSelector: "h1, h2, h3, h4", // 目次として取得する見出しタグ
+     headingSelector: "h2, h3", // 目次として取得する見出しタグ
+     // ヘッダーの高さ分ズレてしまうため調整
+     headingsOffset: 70,
+     scrollSmoothOffset: -70,
    });

    // 不要となったtocbotインスタンスを削除
    return () => tocbot.destroy();
  }, []);

  return <nav className="toc" />;
};
```

### スタイルの調整

目次のスタイルは CDN でスタイルを当てるか、独自で CSS を当てるかになります。

CDN を利用する場合は Next.js が用意している next/head の `<Head>` タグへ追加すれば適用されます。

```tsx
<Head>
  <link
    rel="stylesheet"
    href="https://cdnjs.cloudflare.com/ajax/libs/tocbot/4.11.1/tocbot.css"
  />
</Head>;
```

自分でスタイリングする場合は `tocSelector` で指定した class に対して CSS を書けば適用されます。

アクティブになっているリンクには `.is-active-link` という class が付与されるので、アクティブになっているリンクのスタイルも当てることができます。

参考にこのサイトで使用しているコードを紹介します。
※ コンポーネントライブラリには [Chakra UI](https://chakra-ui.com/) を使用しています。

```tsx
import { Box, useColorModeValue } from "@chakra-ui/react";
import { useEffect } from "react";
import tocbot from "tocbot";

export const Toc = () => {
  useEffect(() => {
    tocbot.init({
      tocSelector: ".toc",
      contentSelector: ".postArticle",
      headingSelector: "h2, h3",
      headingsOffset: 70,
      scrollSmoothOffset: -70,
    });

    // 不要となったtocbotインスタンスを削除
    return () => tocbot.destroy();
  }, []);
  const activeLinkColor = useColorModeValue("gray.900", "gray.300");
  const activeListIconColor = useColorModeValue("purple.500", "purple.700");

  return (
    <Box
      sx={{
        ".toc": {
          borderRadius: "0.25rem",
          fontSize: "0.875rem",
          position: "relative",

          ".toc-list": {
            paddingTop: "0.5rem",

            ":first-child": {
              paddingTop: 0,
            },
            ".toc-list-item": {
              position: "relative",
              listStyle: "none",
              paddingLeft: 5,
              paddingBottom: "0.5rem",

              ":last-child": {
                paddingBottom: 0,
              },
              "::before": {
                position: "absolute",
                top: "4px",
                left: 0,
                width: "12px",
                height: "12px",
                content: '""',
                background: "purple.300",
                border: "2px solid #fff",
                borderRadius: "full",
              },
              ".toc-link": {
                color: "gray.500",
                fontWeight: 700,
              },
              ".toc-list .toc-link": {
                fontWeight: 400,
              },
            },

            ".toc-list-item.is-active-li": {
              ".toc-link": {
                color: activeLinkColor,
              },
              "::before": {
                background: activeListIconColor,
                borderColor: "purple.100",
              },
            },
          },
        },
      }}
    >
      <nav className="toc" />
    </Box>
  );
};
```

## まとめ

Next.js で構築したブログに Tocbot を使用して目次を追加する方法を紹介しました。

Tocbot のオプションは他にもあるので、詳しい情報は Tocbot の公式ドキュメントを確認してください。

zenn でも使用されていて、簡単に目次を作成することができ、スタイルも自由にカスタマイズができるので是非試してみてください。

このサイトの目次スタイルは zenn を参考にしました。

記事のスタイルについても zenn のマークダウン記法を表示させるようにしているので、よければこちらも参考にしてみてください。

https://www.yukimasa.net/posts/zenn-markdown-with-nextjs

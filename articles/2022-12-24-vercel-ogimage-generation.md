---
title: "vercelのog image generationを使用して、Next.jsで動的にOG画像を自動生成する"
emoji: "🐕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nextjs", "vercel"]
published: false
---

この記事は [ミライトデザイン Advent Calendar 2022](https://qiita.com/advent-calendar/2022/miraito-inc) の23日目の記事です。

https://qiita.com/advent-calendar/2022/miraito-inc

前日の [kakiuchi](https://qiita.com/tkek321) の記事に続いて書いていきます。

https://qiita.com/tkek321/items/2aa7757514532dbda2c4

## はじめに

2022/10/10にvercelからOG Imageを自動生成するライブラリの発表がありました。

https://vercel.com/blog/introducing-vercel-og-image-generation-fast-dynamic-social-card-images

このライブラリを使ってNext.jsで作ったページのOG画像を動的に生成する方法を紹介します。

前回の記事を例にこんな感じで表示させることができます。

https://twitter.com/yyykms123/status/1601279551430328320

## 環境

- react: 18.2
- next: 12.3.1
- vercel/og: 0.0.20

## OG画像を生成するライブラリ

vercelが発表した `@vercel/og` というライブラリは、[Vercel Edge Functions](https://vercel.com/docs/concepts/functions/edge-functions) でHTML/CSS（JSX）でマークアップしたものを動にOG画像として生成することができるライブラリです。

[Vercel Edge Functions](https://vercel.com/docs/concepts/functions/edge-functions) とは、エッジで関数を実行できる環境となるので、`@vercel/og` は `runtime: 'edge'` という設定を行ってEdge Runtimeで使用する必要があります。

また、Next.js は v12.2.3 以降を使用する必要があります。

コアエンジンでは [Satori](https://github.com/vercel/satori) と [Resvg](https://github.com/yisibl/resvg-js) という2つのライブラリを使用して、最終的にPNGに変換しています。

具体的には、SatoriというライブラリがHTML/CSS（JSX）からSVGを生成し、Resvgが生成されたSVGをPNGに変換しているような流れで、これをEdge環境で行っています。（JSX → SVG → PNG）

https://vercel.com/docs/concepts/functions/edge-functions/og-image-generation

## 使用方法

基本的には公式の通り進めてけば問題ないです。
サンプルリポジトリを作成しているので参考に見てみてください。

https://github.com/yukimasa/vercel-og-sample

### 画像生成の設定

まずはライブラリのインストール

```
$ yarn add @vercel/og
```

ライブラリをインストールしたら `pages/api` 配下に `og.tsx` を作成して、og画像を生成するためのAPIエンドポイントを作成します。

config に `runtime: 'edge'` を指定して `ImageResponse` を return するだけです。

今回は公式の「Hello world!」の画像で設定をしています。

```tsx:pages/api/og.tsx
import { ImageResponse } from '@vercel/og';

export const config = {
  runtime: 'edge',
};

export default function handler() {
  return new ImageResponse(
    (
      <div
        style={{
          fontSize: 128,
          background: 'white',
          width: '100%',
          height: '100%',
          display: 'flex',
          textAlign: 'center',
          alignItems: 'center',
          justifyContent: 'center',
        }}
      >
        Hello world!
      </div>
    ),
    {
      width: 1200,
      height: 600,
    },
  );
}
```

作成したら `yarn dev` でサーバーを立ち上げ [http://localhost:3000/api/og](http://localhost:3000/api/og) を確認すると画像が生成されていることが確認できます。

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/11bf6475eee1476eb2c535f645bb52f0/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202022-12-22%2023.20.56.png)

画像の生成はこれだけです。

### OGPの設定

ただ、このままだと画像が生成されただけなので、OG画像として表示されるようにするために、pageの `<Head>` に `<meta>` タグを追加します。

ここでは、試しにトップページで確認できるように `pages/index.tsx` で設定します。

実際に確認できるようにvercelにデプロイして絶対パスを指定しておきましょう。

```tsx:pages/index.tsx
<meta
  property="og:image"
  content="<https://vercel-og-sample.vercel.app/api/og>"
/>
```

デプロイできたらOGPを確認できるツールで確認してみましょう。

https://rakko.tools/tools/9/

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/b488b00841d941959682185ec9332840/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202022-12-23%2016.01.43.png)

先程確認した画像でOG画像が設定されていることが確認できます。

### ページごとで動的に画像を生成する

画像を生成してOG画像として設定できたことが確認できましたが、このままでは全ページ「Hello world!」の画像となってしまいます。

そのため、次はページごとに動的に画像が変わるように設定していきます。

ページごとにOG画像を生成するためには、queryパラメーターを使用して動的にページタイトルやその他要素を画像に挿入することができます。

```
<https://vercel-og-sample.vercel.app/api/og?title=hogehoge&publishedAt=2020-12-23>
```

まずは、先程作成した `og.tsx` を下記のように修正します。

```tsx:pages/api/og.tsx
import { ImageResponse } from "@vercel/og";
import { NextRequest } from "next/server";

export const config = {
  runtime: "edge",
};

export default function (req: NextRequest) {
  const { searchParams } = new URL(req.url);
  const hasTitle = searchParams.has("title");
  const title = hasTitle
    ? searchParams.get("title")?.slice(0, 100)
    : "My default title";

  return new ImageResponse(
    (
      <div
        style={{
          fontSize: 128,
          background: "white",
          width: "100%",
          height: "100%",
          display: "flex",
          textAlign: "center",
          alignItems: "center",
          justifyContent: "center",
        }}
      >
        {title}
      </div>
    ),
    {
      width: 1200,
      height: 600,
    }
  );
}
```

最初と違う点としては、 `searchParams` でパラメーターから `title` のデータを取得して、画像のテキストに挿入しています。

試しに先程トップページで指定したurlに `title` のパラメーターを付与して確認してみます。（画像の確認なので、localhostで問題なし）

開発環境で、apiのパスにアクセスしてみます。

[http://localhost:3000/api/og?title=OG画像を動的に生成](http://localhost:3000/api/og?title=OG%E7%94%BB%E5%83%8F%E3%82%92%E5%8B%95%E7%9A%84%E3%81%AB%E7%94%9F%E6%88%90)

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/517cebf179af4d0eb3bae2a1197cb0d7/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202022-12-23%2016.30.09.png)

するとパラメーターで渡したtitleが画像で表示されていることを確認できました。

### サンプルブログを作成し、ページごとにパラメーターを変更してOG画像を設定する

あとは、トップページではなくページごとの `<meta>` タグにクエリーパラメーターを設定するようにします。

サンプルとして、ブログのように一覧ページと詳細ページを作成して各ページで動的にOG画像が生成されるようにします。

### APIの作成

まずはブログのダミーデータを取得できるようにAPIを作成します。

Postの型定義

```ts:types/post.ts
export type Post = {
  id: string;
  title: string;
};
```

ブログのダミーデータ

```ts:mock/posts.ts
import { Post } from "../types/post";

export const postsData: Post[] = [
  {
    id: "4d20c87f-3497-125b-209d-6aa0c44333ac",
    title: "OG画像を動的に生成する",
  },
  {
    id: "46ab05ad-f790-574e-5367-a88a21bf6916",
    title: "Next.jsでJamstackな個人サイトを作った",
  },
];
```

post一覧取得API

```ts:pages/api/posts/index.ts
import type { NextApiRequest, NextApiResponse } from "next";
import { postsData } from "../../../mock/posts";
import { Post } from "../../../types/post";

export default function handler(
  req: NextApiRequest,
  res: NextApiResponse<Post[]>
) {
  res.status(200).json(postsData);
}
```

post詳細API

```tsx:pages/api/posts/[id].ts
import type { NextApiRequest, NextApiResponse } from "next";
import { postsData } from "../../../mock/posts";
import { Post } from "../../../types/post";

export default function handler(
  req: NextApiRequest,
  res: NextApiResponse<Post>
) {
  const posts = postsData;
  const { id } = req.query;
  const post = posts.filter((post) => post.id === id)[0];

  res.status(200).json(post);
}
```

これで、ブログのデータを取得できるAPIができました。

試しに確認してみます。

[http://localhost:3000/api/posts](http://localhost:3000/api/posts)

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/6f1f00312d8d495bb25ed36654e72385/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202022-12-23%2017.14.55.png)

[http://localhost:3000/api/posts/4d20c87f-3497-125b-209d-6aa0c44333ac](http://localhost:3000/api/posts/4d20c87f-3497-125b-209d-6aa0c44333ac)

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/f96020b945184a32a4c7711f4f59659d/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202022-12-23%2018.07.08.png)

これで、一覧及び詳細データが取得できました。

APIができたら、この時点でデプロイをしておきます。

この後ページを作成しますが、APIからダミーデータを取得して事前ビルドをする必要があるので、本番環境でAPIが無いと事前ビルドでデータが取得できずにエラーとなってしまいます。

### ページの作成

あとは一覧と詳細のページファイルを作成しますが、開発環境と本番環境でurlが変わるようにenvの設定をしておきます。

```:.env.local
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

本番環境では、自身がデプロイした環境で設定してください。

vercel の場合だと、ダッシュボードの `Settings > Environment Variables` で設定ができます。

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/b80abfe456594d7192212951fc5228ef/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202022-12-23%2018.45.58.png)

では、ダミーデータをAPIから取得して、一覧画面に表示させるためのページファイルを作成します。

```tsx:pages/posts/index.tsx
import { GetStaticProps, NextPage } from "next";
import Head from "next/head";
import Link from "next/link";
import { Post } from "../../types/post";

type DataType = {
  contents: Post[];
};

export default function Posts({ contents }: DataType) {
  return (
    <>
      <Head>
        <title>posts</title>
        <meta name="description" content="記事一覧" />
        <meta name="viewport" content="width=device-width, initial-scale=1" />
      </Head>

      <main
        style={{
          display: "flex",
          flexDirection: "column",
          justifyContent: "space-between",
          alignItems: "center",
          padding: "6rem",
        }}>
        <ul>
          {contents.map((post) => (
            <Link key={post.id} href={`/posts/${post.id}`}>
              <li>{post.title}</li>
            </Link>
          ))}
        </ul>
      </main>
    </>
  );
}

export const getStaticProps: GetStaticProps<DataType> = async () => {
  const data = await fetch(`${process.env.NEXT_PUBLIC_APP_URL}/api/posts`).then(
    (data) => data.json()
  );

  return {
    props: {
      contents: data,
    },
  };
};
```

一覧ページを確認するとダミーデータのリストが表示されたことが確認できます。

[http://localhost:3000/posts](http://localhost:3000/posts)

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/5c3c5c8b7b944488aae093d957c756d8/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202022-12-23%2018.10.21.png)

次に詳細画面のページファイルを作成します。

```tsx:pages/posts/[id].tsx
import {
  GetStaticPaths,
  GetStaticPathsResult,
  GetStaticProps,
  InferGetStaticPropsType,
  NextPage,
} from "next";
import Head from "next/head";
import { Post } from "../../types/post";
import { ParsedUrlQuery } from "querystring";

type Params = {
  id: string;
} & ParsedUrlQuery;

type DataType = { post: Post };

type Props = InferGetStaticPropsType<typeof getStaticProps>;

export default function Posts({ post }: Props) {
  return (
    <>
      <Head>
        <title>{post.title}</title>
        <meta name="description" content={post.title} />
        <meta name="viewport" content="width=device-width, initial-scale=1" />
        <meta
          property="og:image"
          content={`${process.env.NEXT_PUBLIC_APP_URL}/api/og?title=${post.title}`}
        />
      </Head>

      <main
        style={{
          display: "flex",
          flexDirection: "column",
          justifyContent: "space-between",
          alignItems: "center",
          padding: "6rem",
        }}>
        <p>id: {post.id}</p>
        <h2>{post.title}</h2>
      </main>
    </>
  );
}

export const getStaticPaths: GetStaticPaths = async (): Promise<
  GetStaticPathsResult<Params>
> => {
  const data = await fetch(`${process.env.NEXT_PUBLIC_APP_URL}/api/posts`).then(
    (data) => data.json()
  );

  const paths = data.map((post: Post) => {
    return {
      params: {
        id: post.id,
      },
    };
  });

  return { paths, fallback: false };
};

export const getStaticProps: GetStaticProps<DataType, Params> = async ({
  params,
}) => {
  if (!params?.id) {
    throw new Error("Error: ID not found");
  }

  const post = await fetch(
    `${process.env.NEXT_PUBLIC_APP_URL}/api/posts/${params.id}`
  ).then((data) => data.json());

  return {
    props: { post },
  };
};
```

一覧からブログを選択すると詳細画面に遷移され、idとtitleが表示されることが確認できました。

[http://localhost:3000/posts/46ab05ad-f790-574e-5367-a88a21bf6916](http://localhost:3000/posts/46ab05ad-f790-574e-5367-a88a21bf6916)

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/1231fa11cd944264bb1f56bc4ab243de/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202022-12-23%2018.10.37.png)

また、詳細ページの meta タグを確認してみてください。

```tsx
<meta
  property="og:image"
  content={`${process.env.APP_URL}/api/og?title=${post.title}`}
/>;
```

上記のように記述していますが、ビルドされたあとはAPIから取得したデータのタイトルが表示されていることが確認できます。

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/ee104e9b8f0d44589b1fe01f768befce/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202022-12-23%2018.32.41.png)

これで、ページごとにOG画像が生成されるようになったので、最後にデプロイしてOGP確認ツールで見てみます。

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/75a32e58be1c41ea84203d2cce0bcf4f/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202022-12-24%2012.47.05.png)

ページのタイトルを取得してOG画像が設定されていることが確認できました。

## その他カスタマイズ

OG画像の設定はできるようになったので、あとはスタイルをカスタマイズして完成させます。

公式の方でサンプルを用意していたり、play groundで確認できたりするので、後は自分好みにカスタマイズしてスタイルを整えていきましょう。

https://vercel.com/docs/concepts/functions/edge-functions/og-image-examples

https://og-playground.vercel.app/

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/e78d5bace84846ffbe8078f83ceae062/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202022-12-24%2012.57.19.png)

## まとめ

もともとOG画像は自分で作成して設定していたりしたので、自動で生成してくれると楽ですし、デザインも統一されるのでとてもいいなと思いました。

Vercelが用意してくれているライブラリで簡単にOG画像が生成できるので、よかったら試してみてください。

次は「NFTと仮想通貨とブロックチェーン関連」について書いてくれる [FrozenVoice](https://qiita.com/FrozenVoice) さんの記事です。

https://qiita.com/FrozenVoice/items/9b5a607ec6c8d8ef9d54

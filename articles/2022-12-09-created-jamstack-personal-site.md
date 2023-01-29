---
title: "Next.jsでJamstackな個人サイトを作った"
emoji: "💭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nextjs", "microcms", "vercel"]
published: false
---

この記事は [ミライトデザイン Advent Calendar 2022](https://qiita.com/advent-calendar/2022/miraito-inc) の9日目の記事です。

https://qiita.com/advent-calendar/2022/miraito-inc

前日の [nyokkiさん](https://qiita.com/Nyokki) の記事に続いて書いていきます。

https://qiita.com/Nyokki/items/2a9b9683fbc5fb70d8ee

## はじめに

個人のブログは Wordpress を使用したものがあるのですが、 Next.js を使用してなにか作りたいと思ったので、 Next.js × microCMS というみんながよくやる Jamstack 構成で個人サイトを作成することにしました。

今回は作成した個人サイトの構成や軽いデモを行いたいと思います。

## 構成

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/98af3ef134c940a78e54b71ea919db79/%E3%82%B7%E3%82%B9%E3%83%86%E3%83%A0%E6%A7%8B%E6%88%90%E5%9B%B3.drawio.png)

### ヘッドレスCMS

コンテンツを管理するために、ヘッドレスCMSの microCMS というサービスを使用しています。

Wordpress などの CMS はコンテンツを管理する部分（バックエンド部分）と表示する部分（フロント部分）が統合されていますが、ヘッドレスCMSはコンテンツを管理する部分（バックエンド）だけの CMS となります。

フロント部分は自分で用意する必要がある分自由度が高いので、Next.js を使用して作成します。

ヘッドレスCMSは Spotify などで使われている [Contentful](https://www.contentful.com/) や、Meta社が開発した GraphQL の活用に特化した [GraphCMS](https://hygraph.com/) などがありますが、今回は日本の会社で日本語対応していることから microCMS を選択しました。

https://microcms.io/

### 静的サイトジェネレーター

HTML を作成するものとしては Next.js を使用します。

その他には [Gatsby](https://www.gatsbyjs.com/) などが有名ですが、一番の目的は Next.js で作るということだったので、今回は対象外となりました。

https://nextjs.org/

### ホスティングサービス

Next.js の開発元でもあるVercel社が運営している [Vercel](https://vercel.com/) というホスティングサービスを使用。

個人使用（Hobbyプラン）は無料で、 Next.js と開発元が同じなので相性が良く、 Next.js の新しい機能を使用できたりするため採用しています。

社名と同じでややこしいです。

https://vercel.com/

### CI/CD

基本的には Vercel の設定ですべて完結します。

コンテンツ管理するのは現状は投稿記事のみのため、 SSG を採用しソースコード修正時とコンテンツ作成・更新時にデプロイされるようにします。

ソースコード修正時は Vercel の設定で指定したブランチ（ `main` ）を `push` したときに自動でビルド・デプロイするようにしています。

コンテンツ作成・更新時は microCMS で Vercel の deploy用の Webhook を設定して自動でビルド・デプロイされます。

### スタイル

ほんとはfigmaなどでデザインを作成してからコーディングしたいのですが、デザインスキルは皆無なので、コンポーネントライブラリの chakra-ui を使ってよしなにスタイリングしてます。

提供されているテンプレートのデザインや、その他のサービスなどを参考に作成しています。

https://chakra-ui.com/

## サイト作成手順

今回は作成した個人サイトと同じ様に、 microCMS のコンテンツを取得して表示させる所までをデモしたいと思います。

個人サイトでは chakra-ui を導入してスタイリングしていますが、デモではAPIの取得及び静的ファイルの生成・デプロイまでとしたいと思います。

githubのサンプルはこちら

https://github.com/yukimasa/homepage-sample

### 環境

- Next.js: 13.0.6
- React: 18.2.0
- TypeScript: 4.9.3

### Next.js でプロジェクト作成

公式 [Getting Started](https://nextjs.org/docs) のコマンドを実行して、プロジェクト名の設定と ESLint の設定を行います。

ESLintは `Yes` を選択しましょう。

```sh
❯ yarn create next-app --typescript

? What is your project named? › homepage-sample
? Would you like to use ESLint with this project? … No / Yes
```

プロジェクト作成したら `yarn dev` で [http://localhost:3000/](http://localhost:3000/) が立ち上がることを確認します。

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/d42a645db2714d6e8c020cc1f5b92544/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202022-12-05%2020.25.33.png)

### microCMS の設定

[https://microcms.io/](https://microcms.io/) でアカウント作成したら、サービス名、サービスIDを入力してサービスを作成します。

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/ea1c9e9f831541fa991af0a4908b972b/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202022-12-05%2020.45.11.png)

API を作成は「自分で決める」を選択し、API名を「投稿」とし、エンドポイントを `posts` に設定して次へ進みます。

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/226075dae64147c1ae595d2562dfe00e/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202022-12-05%2020.46.56.png)

リスト形式を選択し、以下の内容でAPIスキーマを定義します。

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/896ff587ec4d4bf6a0f2ddc72e60927f/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202022-12-05%2020.49.30.png)

テスト用に最低限内容がわかるぐらいで適当にコンテンツを作成しておきます。

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/7db57b1f0abc44f6a0adc5438b80509e/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202022-12-05%2021.17.15.png)

### API で microCMSからコンテンツを取得

microCMSからAPIでコンテンツを取得するために、公式が提供しているライブラリ（ `microcms-js-sdk` ）をインストールしましょう。

https://github.com/microcmsio/microcms-js-sdk

```sh
❯ yarn add microcms-js-sdk
```

`.env.local` を作成して、microCMS のAPIキーとドメインを追加します。

こちらの情報は外部公開しないシークレットなデータなので git 管理はしないように注意してください。

```env:.env.local
MICROCMS_SERVICE_DOMAIN=homepage-sample
MICROCMS_API_KEY=hogehogehoge
```

`libs` フォルダに `client.ts` を作成してsdkの設定を行います。

```ts:libs/client.ts
import { createClient } from "microcms-js-sdk";

export const microcmsClient = createClient({
  serviceDomain: process.env.MICROCMS_SERVICE_DOMAIN ?? "",
  apiKey: process.env.MICROCMS_API_KEY ?? "",
});
```

次に `types/post.ts` を作成し記事の型を定義します。

`id` や作成日などは microCMS の型が用意してくれているので、交差型を利用して記事の型に追加します。

```ts:types/post.ts
import { MicroCMSListContent } from "microcms-js-sdk";

export type Post = {
  title: string;
  content: string;
} & MicroCMSListContent;
```

型定義をしたら `pages/index.tsx` を作成して記事一覧を取得するコードを記述します。

```tsx:pages/index.tsx
import { GetStaticProps, InferGetStaticPropsType, NextPage } from "next";
import Link from "next/link";
import { microcmsClient } from "../libs/client";
import { Post } from "../types/post";

type DataType = {
  contents: Post[];
};

type Props = InferGetStaticPropsType<typeof getStaticProps>;

const Home: NextPage<Props> = ({ contents }) => {
  return (
    <div style={{ width: "1170px", margin: "100px auto" }}>
      <h2>投稿一覧</h2>
      <ul>
        {contents.map((post) => (
          <li key={post.id}>
            <Link href={`/posts/${post.id}`}>{post.title}</Link>
          </li>
        ))}
      </ul>
    </div>
  );
};

export default Home;

export const getStaticProps: GetStaticProps<DataType> = async () => {
  const data = await microcmsClient.get<DataType>({ endpoint: "posts" });

  return {
    props: {
      contents: data.contents,
    },
  };
};
```

上記のコードは `getStaticProps` でビルド前に API でコンテンツを取得し、コンポーネントに `props` として渡しています。

トップページを表示すると作成した投稿一覧が表示されていることが確認できます。 （システムのテーマがダークなので暗い。。）

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/d3344ef8e71a4f4b8885ffdcf61c1310/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202022-12-05%2021.17.58.png)

これで microCMS からコンテンツを取得できたことは確認できましたが、ついでに詳細画面も作成します。

詳細画面は `query` （記事の `id` ）によって動的に変わるページとなるので、[Dynamic Routes](https://nextjs.org/docs/routing/dynamic-routes)を使用します。（取得した `id` ごとに1つの HTML が生成される。）

https://nextjs.org/docs/routing/dynamic-routes

Dynamic Routes はファイル名にブラケット釣るので、`pages/posts/[id].tsx` のようにして下記のコードを記述します。

```tsx:pages/posts/[id].tsx
import type {
  GetStaticPaths,
  GetStaticPathsResult,
  GetStaticProps,
  InferGetStaticPropsType,
  NextPage,
} from "next";
import { Post } from "../../types/post";
import { microcmsClient } from "../../libs/client";
import { ParsedUrlQuery } from "querystring";

type Params = {
  contentId: string;
} & ParsedUrlQuery;

type DataType = { post: Post };

type Props = InferGetStaticPropsType<typeof getStaticProps>;

const PostId: NextPage<Props> = ({ post }) => {
  return (
    <main style={{ width: "1170px", margin: "100px auto" }}>
      <h2>{post.title}</h2>
      <p>{post.publishedAt}</p>
      <div
        dangerouslySetInnerHTML={{
          __html: `${post.content}`,
        }}
      />
    </main>
  );
};

export default PostId;

// 静的生成のためのパスを指定
export const getStaticPaths: GetStaticPaths = async (): Promise<
  GetStaticPathsResult<Params>
> => {
  const data = await microcmsClient.get({ endpoint: "posts" });
  const paths = data.contents.map((content: Post) => {
    return {
      params: {
        id: content.id,
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

  const data = await microcmsClient.get<Post>({
    endpoint: "posts",
    contentId: params.id as string | undefined,
  });

  return {
    props: { post: { ...data } },
  };
};

```

上記のコードは `getStaticPaths` で投稿一覧を取得し、 `params` としてその `id` を `return` させています。

`getStaticProps` でその `params` を受け取り `id` から詳細 API で投稿の詳細情報を取得しています。

その後は一覧と同じでコンポーネントにpropsで渡してデータを表示させています。

以上から `getStaticPaths` 及び `getStaticProps` は、いずれもビルドを行う前に外部のデータが必要な際に使用する関数ということがわかります。

詳細画面を確認するためにトップページの記事リストのリンクを選択します。

すると詳細画面に遷移し、 microCMS で作成した記事が表示されていることが確認できます。

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/a57c724208ae4e798af85f86206357f9/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202022-12-05%2021.31.52.png)

> _※ dangerouslySetInnerHTML はXSSを引き起こす可能性があるため非推奨とされています。今回の構成においては、入力はmicroCMSのリッチエディタからのみであり、ユーザーからの自由入力箇所ではないため安全であるとして利用しています。_
>
> _microCMS の [公式ブログ](https://blog.microcms.io/microcms-next-jamstack-blog/) からの引用_

なお、開発環境ではリクエストの度に `getStaticProps` が呼び出されるので、`yarn build` を実行して実際に静的ファイルが生成されていることを確認しましょう。

```sh
❯ yarn build
yarn run v1.22.17
$ next build
info  - Loaded env from /Users/yukimasa/works/myApp/homepage-sample/.env.local
info  - Linting and checking validity of types
info  - Creating an optimized production build
info  - Compiled successfully
info  - Collecting page data
info  - Generating static pages (6/6)
info  - Finalizing page optimization

Route (pages)                              Size     First Load JS
┌ ● /                                      2.34 kB        75.5 kB
├   /_app                                  0 B            73.2 kB
├ ○ /404                                   181 B          73.4 kB
├ λ /api/hello                             0 B            73.2 kB
└ ● /posts/[id] (1576 ms)                  399 B          73.6 kB
    ├ /posts/28l3ltjqm (666 ms)
    ├ /posts/g_gpm2rryitr (663 ms)
    └ /posts/ufcpmp8hc
+ First Load JS shared by all              73.4 kB
  ├ chunks/framework-8c5acb0054140387.js   45.4 kB
  ├ chunks/main-f2e125da23ccdc4a.js        26.7 kB
  ├ chunks/pages/_app-3893aca8cac41098.js  296 B
  ├ chunks/webpack-8fa1640cc84ba8fe.js     750 B
  └ css/ab44ce7add5c3d11.css               247 B

λ  (Server)  server-side renders at runtime (uses getInitialProps or getServerSideProps)
○  (Static)  automatically rendered as static HTML (uses no initial props)
●  (SSG)     automatically generated as static HTML + JSON (uses getStaticProps)

✨  Done in 7.34s.
```

上記ログを見ると、`/` と `/posts/[id]` が SSG となっていることが確認できます。（ ● マーク）

実際のビルドされたファイルは `.next/` フォルダに出力されており、 `yarn start` を実行すると production mode で開発環境が実行されます。

本番の様に事前ビルドした静的ファイルを使用して確認する場合は `yarn build` してから `yarn start` して確認しましょう。

### Vercel にデプロイ

API からコンテンツを取得することが確認できたので、静的ページのビルドを行い Vercel にデプロイします。

Vercel へのデプロイは Github との連携で簡単にできるので、 Github にリポジトリを作成して `push` しておきます。

[Vercel](https://www.notion.so/headlessCMS-Next-js-Vercel-11e2acefd23a4c669081e1d6ddbaf02b) に Github アカウントでログインして 「Add new project」 を選択し、 Github のリポジトリと連携を行ってください。

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/adad8caa179949bca706fdd464d4fbc4/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202022-12-06%2010.48.46.png)

Github に Vercel がない場合はインストールし、その後 Github 上で Repository access の設定を行う必要があります。

連携ができたら import を選択するとプロジェクトの情報が表示されます。

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/14a20ae6bc584684ba80eba68c979f18/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202022-12-06%2010.54.19.png)

「Build and Output Settings」は何もせず、「Environment Variables」に環境変数を設定していきます。

現在は microcms-sdk の createClient で使用している `MICROCMS*SERVICE*DOMAIN` と `MICROCMS_API_KEY` の設定が必要なので追加しましょう。

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/3c06ae7328374aba940c6d01d9e48dd6/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202022-12-06%2010.57.41.png)

現状の設定はこれだけなのでこのままデプロイを実行します。

デプロイが完了すると下記の画面が表示されるので、左の画面プレビューをクリックして実際にホスティングされていることを確認しましょう。

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/e1c8c689f0974ea5b00d4f2a8854da08/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202022-12-06%2011.21.40.png)

vercel に割り振られたドメインでプロジェクトを確認することができます。

### Vercel と microCMS の連携

このままだと microCMS のコンテンツを更新してもホスティングされたファイルには反映されないので、記事更新のタイミングでもビルド・デプロイされるように Webhook を使用して Vercel と microCMS の連携を行います。

Vercel の Dashboard から Settings タブを選択し、Git > Deploy Hooks で hook を作成します。

hookの名前は何でもいいですが、「microCMS」とし、対象ブランチを「main」としておきます。

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/ba5fde8b16994e3c8c615c3fea450720/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202022-12-06%2011.28.17.png)

「Create Hook」で作成し、表示された URL をコピーしておきます。

次に microCMS の管理画面で、「投稿」の API 設定を開きます。

Webhook のメニューを開き、追加を選択するとサービスの選択肢が表示されるので Vercel を選択します。

Webhook の識別名と先ほどコピーした Webhook の URL を入力し設定を完了します。

トリガーの細かい設定は好みで構いません。

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/dcf7a5241b4d4944ab8357c7a5a783b8/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202022-12-06%2011.36.21.png)

設定が完了したあとは記事を更新するとビルド・デプロイが自動で行われます。（Vercel の Deployments でログが確認できる）

※ Webhook の設定はコンテンツごとのAPI設定で行うため、別のコンテンツを追加し「更新したらビルド・デプロイ」と同じ様に行いたい場合は、別で Webhook の設定が必要です。

### ドメインの設定

ドメインは Vercel によって作られたものが設定されているのでこのままでも問題ないですが、独自ドメインを設定したい場合は Vercel の管理画面から設定を行います。

Dashboard の Settings タブから Domains を選択し、取得済みのドメインを追加します。

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/1c5fbe430f79449c85a3f022f05e619f/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202022-12-06%2011.42.47.png)

追加方法は何でも良いのですが、一番上が Recommended されているのでこちらで追加します。

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/79d57dfe8b5b40be92532d818e90576f/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202022-12-06%2011.43.17.png)

Aレコードや CNAME が作成されるので、取得したドメインのサーバーでレコードの設定を行ってください。

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/bf7d8033e9424c56ba1bb65cd56ce029/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88%202022-12-06%2011.45.00.png)

しばらく時間がかかりますが、設定が完了すると **Invalid** から **Valid** にステータスが変わり、設定したドメインでのアクセスが可能となります。

## おわりに

Next.js での SSR や API 、Dynamic Routes などの機能を使用したことが無かったので、勉強がてらサイトを作ってみました。

今後も色々アップデートできていければと思います。

明日は CSS in JS と Stitches について書いてくれる [polidog](https://qiita.com/polidog) さんの記事です。

https://qiita.com/polidog/items/63eb8bce10fd42b30f79

---
title: "Next.jsで画面遷移時にプログレスバーを表示させる"
emoji: "🪬"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nextjs"]
published: true
---

## はじめに

今回は Next.js で構築したサイトでページ遷移するときに、画面上部にプログレスバーを表示させ読み込み状態がわかるようにする方法を紹介します。

プログレスバーは Zenn などでも使用されている NProgress というライブラリを使用することで簡単に表示させることができます。

https://ricostacruz.com/nprogress

![](https://images.microcms-assets.io/assets/b26b0b6364c64a6191344b84bc3f3136/bae6afc112c94202be6fedecd3a865b1/nprogress.gif)

## 環境

- Next.js: 13.0.6
- React: 18.2.0
- NProgress: 0.2.0

## 実装手順

1. ライブラリのインストール
2. プログレスバーを表示させる hooks の作成
3. `_app.tsx` に hooks の呼び出し処理を実装する

### ライブラリのインストール

https://github.com/rstacruz/nprogress

ライブラリをインストールする

```sh
❯ yarn add nprogress
```

型定義をインストール

```sh
❯ yarn add -D @types/nprogress
```

### プログレスバーを表示させる hooks の作成

ページ遷移時にプログレスバーを表示させる hooks を作成します。

Next.js の `useRouter` でページ遷移時のイベントを取得できるので、イベントごとにプログレスバーの表示/非表示を切り替えます。

プログレスバーの制御は `NProgress.start()` と `NProgress.done()` を実行するだけです。

まずは `src/hooks/useProgressBarAtTransition.ts` を作成します。

```ts:src/hooks/useProgressBarAtTransition.ts
import 'nprogress/nprogress.css'

import { useRouter } from 'next/router'
import NProgress from 'nprogress'
import { useEffect } from 'react'

export const useProgressBarAtTransition = (): void => {
  const router = useRouter()

  useEffect(() => {
    const handleStart = () => {
      NProgress.start()
    }

    const handleStop = () => {
      NProgress.done()
    }

    router.events.on('routeChangeStart', handleStart)
    router.events.on('routeChangeComplete', handleStop)
    router.events.on('routeChangeError', handleStop)

    return () => {
      router.events.off('routeChangeStart', handleStart)
      router.events.off('routeChangeComplete', handleStop)
      router.events.off('routeChangeError', handleStop)
    }
  }, [router])
}
```

プログレスバーのスタイルは `import 'nprogress/nprogress.css'` でライブラリのデフォルトスタイルを適用します。

イベントごとの処理は `router.events.on` でイベント名とイベント発生時の処理を指定することで実行できます。

各イベントについては以下のとおりです。

- `routeChangeStart` : 画面遷移が変更され始めたときに発生
- `routeChangeComplete` : 画面遷移が完全に変更されたときに発生
- `routeChangeError` : 画面遷移時にエラーが発生した場合、または画面の読み込みがキャンセルされた場合に発生

useEffect でイベントごとの処理を指定し、コンポーネントが破棄された際のクリーンアップ処理は `router.events.off` で行います。

その他の router の情報については [公式ページ](https://nextjs.org/docs/api-reference/next/router#routerreload) で確認できるので、時間があるときにでも覗いてみてください。

https://nextjs.org/docs/api-reference/next/router#routerreload

### `_app.tsx` に hooks の呼び出し処理を実装する

全ページに適用させるために `_app.tsx` で作成した hooks を実行させます。

```tsx:_app.tsx
import type { AppProps } from 'next/app'

import { useProgressBarAtTransition } from '@/hooks/useProgressBarAtTransition'

const MyApp = ({ Component, pageProps }: AppProps) => {
  useProgressBarAtTransition()

  return <Component {...pageProps} />
}

export default MyApp
```

これで画面遷移を行なうと画面上部にプログレスバーとスピナーが表示されていることが確認できるかと思います。

## カスタマイズ

実装としては完了となりますが、カスタマイズもいくつかできるので試したいと思います。

詳細については公式の Github に記載があるので確認してみてください。

https://github.com/rstacruz/nprogress#advanced-usage

### スピナーを非表示にする

画面右上に表示されるスピナーはプログレスバーが表示されているので非表示にしたいと思います。

```ts:src/hooks/useProgressBarAtTransition.ts
// ..省略..

export const useProgressBarAtTransition = (): void => {
  const router = useRouter()

  // スピナーを非表示にする
  NProgress.configure({ showSpinner: false })

  useEffect(() => {

    // ..省略..

  }
}
```

`NProgress.configure({ showSpinner: false })` を追加するだけでスピナーが非表示となります。

スピナーの設定以外にも速度やアニメーションなども設定できるみたいなので、気になる人は試してみてください。

### スタイルを修正する

スタイルはライブラリデフォルトのものを指定していましたが、プログレスバーの色などを変更したい場合は css を修正します。

まずは、公式の css をプロジェクトに作成するので、 Github からコピーしてきます。

https://github.com/rstacruz/nprogress/blob/master/nprogress.css

コピーしてきたら `#29d` が指定されている箇所を変更したい色に置換してみます。

変更したら `_app.tsx` でcssをインポートします。

```tsx:styles/nprogress.css
import '../../styles/nprogress.css'

// ..省略..
```

そうするとプログレスバーのカラーが変更されたことが確認できると思います。

## まとめ

NProgress を使うだけで簡単に画面遷移時のプログレスバーが表示できたかと思います。

細かいカスタマイズもできるので色々いじってみてください。

## 参考

https://caspertheghost.me/blog/nprogress-next-js

https://fumidzuki.com/knowledge/5013/

https://learnjsx.com/category/4/posts/nextjs-nprogress

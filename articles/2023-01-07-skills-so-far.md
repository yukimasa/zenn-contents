---
title: "2023を迎えたということで、今まで触れた技術などをまとめとく"
emoji: "🪬"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["振り返り"]
published: true
---

## はじめに

年が明けてから少し時間が経ってしまいましたが、2023年6月にはエンジニアとして4周年になるので、年の境目をきっかけに今まで触れたことのある技術や経験したことについて振り返ってまとめたいと思います。

## 言語

- Ruby（1年8ヶ月）
- Swift（1年8ヶ月）
- Java（1年8ヶ月）
- PHP（2年4ヶ月〜）
- TypeScript（2年4ヶ月〜）

### コメント

Ruby, Swift, Java は同じ案件で使用。

Ruby on Rails で構築された管理システムの開発と、iOS, Android の開発で Swift, Java を使用したモバイルアプリの開発を行っていました。

エンジニアなりたてほやほやだったので、最初は開発より Git やその他周辺知識のキャッチアップで忙しかった気がします。

管理システム開発したりアプリのリリースまで行ったりしたので、だいぶ良い経験になったと思います。

PHP, TypeScript は今の案件で使っている言語で、Ruby とかよりはちゃんと身についている感じです。

## フレームワーク・ライブラリ

### フロントエンド

- React/Next.js
  - [Redux Toolkit](https://redux-toolkit.js.org/)
    - 状態管理ライブラリ
  - [Redux Persist](https://github.com/rt2zz/redux-persist)
    - 管理している状態をブラウザの storage と同期させるライブラリ
  - [React Hook Form](https://react-hook-form.com/)
    - フォームバリデーションライブラリ
  - [Material UI](https://mui.com/)（現: MUI）
    - UIコンポーネントを提供しているライブラリ
    - v5でブランド名が MUI に変更された（破壊的な変更もあって、乗り換えがまだできていない…）
  - [Chakra UI](https://chakra-ui.com/)
    - 同じくUIコンポーネントを提供しているライブラリ
    - 個人的にはこっちのほうがUtility-Firstで好み
    - → MUI より Utility-First で style が当てやすく、TailwindCSS はコンポーネントではなく class を提供している感じなので、結局は自分でUIを組み立てなきゃいけない
    - → ただ、MUI より提供しているコンポーネントが少ないのがデメリットかも

### バックエンド

- Ruby on Rails
- Laravel
  - [Laravel Sanctum](https://readouble.com/laravel/9.x/ja/sanctum.html)
    - SPA での認証に使用
  - [Laravel-permission](https://spatie.be/docs/laravel-permission/v5/introduction)
    - ロールを使用したアクセス制御を実現するために使用
    - **RBAC方式（Role Base Access Control）**
    - → User ⇔ Role ⇔ Permission
  - [Laravel-Dacapo](https://github.com/ucan-lab/laravel-dacapo)
    - マイグレーションファイルをスキーマで管理し生成するライブラリ
    - **※ プロジェクト構築時のみ使用し、運用フェーズでは通常のマイグレーション運用とするので、使用後は削除する**
  - [Laravel Enum](https://github.com/BenSampo/laravel-enum)
    - Laravel で Enum をいい感じに使えるようにするライブラリ
    - PHP8.1 からは Enum が追加されたので不要になりそう
  - [L5 Swagger](https://github.com/DarkaOnLine/L5-Swagger)
    - PHPdoc みたいな形で Swagger のapi仕様書を作成するために使用

#### 参考

https://qiita.com/ucan-lab/items/e75ef2bba744d9dcd4d0

### コメント

Rails を使っていたときは テンプレートエンジン で View を作成していましたが、現在担当している案件では、フロントを Next.js/React で構築し、 Laravel でAPIサーバーを立てています。

認証に Laravel Sanctum を使用していたが、最近 AWS Cognito を導入して実装内容が変わった。

AWS Cogntio は Laravel から sdk を使用する方法での認証となっているので、調べても情報が少なく苦労したがかなりの経験となった。

## プラットフォーム

### GCP

https://console.cloud.google.com/?hl=ja

- コンピューティング
  - GAE（Google App Engine）
  - GCE（Google Compute Engine）
  - GCF（Google Cloud Functions）
- ストレージ・データベース
  - GCS（Google Cloud Storage）
  - Cloud SQL
- 管理ツール
  - Cloud Monitoring
  - Logging
- ネットワーキング
  - Cloud Load Balancing
  - Cloud NAT
  - Cloud VPN
- デベロッパーツール
  - Cloud SDK
  - Cloud Scheduler
  - Cloud Tasks

### AWS

https://aws.amazon.com/jp/

- Amazon Cognito
- Amazon SES（Simple Email Service）

### コメント

基本的には GCP を使用していたが、 認証に Cognito を使用するとのことから AWS も触るようになった。

スケーラビリティがどうとか細かい設計や設定はまだ怪しいが、基本的な操作などは一通りできるようにはなっている感じ

## ツール・サービス

### Docker

- 主に開発環境で使用
- 今はほとんどがDockerで最初の頃は vagrant で構築したりしていた
- Dockerの細かいコマンドとかは忘れてたりするが基本概念はある程度理解
- このの記事が理解できたら運用する上では十分そう

https://zenn.dev/suzuki_hoge/books/2022-03-docker-practice-8ae36c33424b59

### Vercel

- OG Image Generation
  - OG画像を動的に生成するライブラリ
  - 実際にこのサイトで使用しているので、こちらを参考に

https://www.yukimasa.net/posts/vercel-ogimage-generation

- Vercel Analytics
  - アプリケーションのパフォーマンスやぺーじアクセスなどを計測できる
  - Google Analitics より自由度は低そうだが試しに入れてみた

### **[PAY.JP](http://pay.jp/)**

- クレジットカード決済の代行サービス
- 都度課金や継続課金に対応しているが、最近だと Stripe のほうがいいぽい？

### [SendGrid](https://sendgrid.kke.co.jp/)

- メールサーバーとして使用
- 無料枠があるし使い勝手はいい感じ

## おわりに

2022年は案件で忙しかったこともあり、キャッチアップやアウトプットが少なかったなと感じた年だった。（案件での経験はかなり得たが）

Next.js でこのサイトを作ったりした分、2023年はもうちょっとキャッチアップやアウトプットを増やしていこうと思いました。

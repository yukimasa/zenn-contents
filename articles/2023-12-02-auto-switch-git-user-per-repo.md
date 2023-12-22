---
title: "リポジトリ毎の Git ユーザー情報を自動で切り替える"
emoji: "🪬"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["git"]
published: true
---

この記事は [ミライトデザイン Advent Calendar 2023](https://qiita.com/advent-calendar/2023/miraito-inc) の 2 日目の記事です。

## はじめに

Git を使っていると個人や会社、その他クライアントなどでリポジトリのユーザー名・ユーザーアドレスを切り替えたいことがよくあります。

リポジトリ毎に config 設定をすれば良いのですが、設定をし忘れてグローバルに設定している自分のアドレスでコミットしてしまったということが何回かありませんか？

今回はリポジトリ毎に設定を自動で切り替える方法を紹介します。

## リポジトリごとの設定を強制する

`useConfigOnly` という設定を `true` にしておくことで、グローバルも含め `user.name` と `user.email` を指定していないとコミットできなくなります。

```sh-session
❯ git config --global user.useConfigOnly true
```

ただし、グローバルの設定があるとそれを使用するようになるため、リポジトリ毎で明示的に指定したい場合は、グローバルの設定を削除してリポジトリ毎に都度設定させるようにします。

```sh-session
❯ git config --global --unset user.email
❯ git config --global --unset user.name
```

これらの設定をした上で、ユーザー設定をしていないリポジトリでコミットをしようとすると下記のエラーが表示されるようになり、設定し忘れに気づくことができるようになりました。

```sh-session
❯ git commit -m "first commit"
Author identity unknown

*** Please tell me who you are.

Run

  git config --global user.email "you@example.com"
  git config --global user.name "Your Name"

to set your account's default identity.
Omit --global to set the identity only in this repository.

fatal: no email was given and auto-detection is disabled
```

:::message
`user.useConfigOnly` は、Git バージョン 2.8.0 以降から設定可能になります。
:::

## ディレクトリ単位で config を自動で切り替える

設定を強制させることでリポジトリ毎のユーザー設定を間違えることはなくなりましたが、都度リポジトリに設定することになるので少々面倒です。

そこで gitconfig で設定できる `includeIf` というのを使って、ディレクトリ単位でアカウントを切り替えるようにします。

`includeIf` は、gitconfig 内で使用できる機能で、特定の条件が満たされたときに特定の設定を適用することができます。

例えば、個人用（ private ）と仕事用（ works ）が以下のようなディレクトリで別れているとします。

```text
~/
├── private
└── works
```

private には個人用の情報で、 works には仕事用の情報を設定するとした場合、下記の config ファイルを作成します。

```text
~/
├── .gitconfig           # グローバルの config ファイル
├── .gitconfig-private   # 個人用の config ファイル
└── .gitconfig-works     # 仕事用の config ファイル
```

それぞれ設定したい情報を各設定ファイルに追記します。

```text:.gitconfig-private
[user]
  name  = private-user
  email = private-user@example.com
```

```text:.gitconfig-works
[user]
  name  = company-user
  email = company-user@company.example.com
```

そして、 `.gitconfig` で `includeIf` を使って作業ディレクトリ毎に読み込むファイルを設定します。

```text:.gitconfig
[user]
  [includeIf "gitdir:~/private/"]
    path = ~/.gitconfig-private
  [includeIf "gitdir:~/works/"]
    path = ~/.gitconfig-works
```

これで、指定したディレクトリ配下のリポジトリは、適用された config のユーザー情報で設定されるようになりました。

private 配下のリポジトリで user.name と user.email を確認すると意図した設定となっていることが確認できます。

```sh-session
❯ cd ~/private/private-test/
❯ git config user.name
private-user
❯ git config user.email
private-user@example.com
```

works 配下のリポジトリも同様に確認できました。

```sh-session
❯ cd ~/works/works-test/
❯ git config user.name
company-user
❯ git config user.email
company-user@company.example.com
```

例外的に特定のリポジトリだけ情報を変えたい場合は、通常通り local の gitconfig を設定することで上書きされるため問題ありません。

```sh-session
❯ cd ~/works/works-test/
❯ git config user.name "company-user-1"
❯ git config user.email
company-user-1
```

これで、リポジトリ毎に設定を行わずに、指定したディレクトリ配下のユーザーが自動切り替えされるようになりました。

:::message
`includeIf` は、Git バージョン 2.13.0 以降から設定可能になります。
:::

## まとめ

今回は、 `useConfigOnly` 設定を使用してユーザー設定の強制を行い、 `includeIf` 設定を使用してディレクトリ単位でユーザー情報を自動で切り替える方法について紹介しました。

これにより、リポジトリ毎にユーザー設定を行う手間を省き、設定のミスを防ぐことができます。

Git は開発をしていれば使用することが多いと思うので、是非この設定を参考にストレスフリーになってみてください。

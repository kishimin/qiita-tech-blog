---
title: カスタムフックは再利用できるべきなのか？
tags:
  - "React"
  - "フロントエンド"
  - "Hooks"
private: false
updated_at: ""
id: null
organization_url_name: null
slide: false
ignorePublish: false
posting_campaign_uuid: null
agreed_posting_campaign_term: false
---

# はじめに

カスタムフックに関するコードレビューで、こんな言葉を耳にしました。

「それはカスタムフックにするべきではない」  
「カスタムフックは使い回せるべき」

この意見を聞いて、カスタムフックは本当に使い回せるべきなのか？と疑問に思いました。

自分としては、カスタムフックには再利用以外にも意味があるのではないかと考えています。

そこで今回は、React公式ドキュメントや2人のエンジニアのブログを参考に、カスタムフックの再利用について調べてみようと思います。

# 自分の考え

自分としては、カスタムフックには再利用するだけでなく、ロジックを分離する意味もあると思っています。

例えば、以下の記事で紹介されているようなデータ取得処理です。

https://zenn.dev/juth/articles/3fd066b1cceab4#2.-hooks%2F-%E2%80%95-%E3%83%87%E3%83%BC%E3%82%BF%E5%8F%96%E5%BE%97%E3%83%AD%E3%82%B8%E3%83%83%E3%82%AF%E3%81%AE%E5%88%87%E3%82%8A%E5%87%BA%E3%81%97s

実際、[Tanstack Query](https://tanstack.com/query/latest/docs/framework/react/overview)を使用したときも、API通信の処理と、エラーやローディングなどの表示を分けて考えられる点がよいと感じました。

このようなデータ取得処理は、特定のコンポーネントでしか使わない場合でも、カスタムフックに切り出す意味があるのではないかと考えています。

そのため、カスタムフックは必ずしも使い回せるべきだとは考えていません。

また、コンポーネントはできるだけ表示に関する処理に集中させた方がよいと思っています。

ロジックを通常の関数やカスタムフックなどに切り出すことで、コンポーネントから分離できます。

その方がコンポーネントのテストもしやすくなると思います。

# カスタムフックの考えを調べてみる

カスタムフックは再利用できるべきなのか、Reactの公式ドキュメントや2人のエンジニアのブログを確認してみます。

今回は、以下の3つを参考にしました。

- Reactの公式ドキュメント
- Max Rozenさんのブログ
- TkDodoさんのブログ

## React公式ドキュメント

まずはReact公式ドキュメントを確認してみます。

https://ja.react.dev/learn/reusing-logic-with-custom-hooks

公式ドキュメントでは、カスタムフックを使って複数のコンポーネント間でロジックを再利用する方法が紹介されています。

一方で、ロジックを切り出すことについて、以下のようにも説明されています。

> ロジックをカスタムフックに抽出することで、外部システムやブラウザ API とのやり取りに関する面倒な詳細を隠蔽することができます。あなたのコンポーネントのコードは、実装方法ではなく意図を表現するようになるのです。

また、カスタムフックに切り出すタイミングについても説明されています。

> あらゆる小さなコードの重複に対してカスタムフックを抽出する必要はありません。多少の重複は問題ありません。

一方で、エフェクトについては、以下のように述べられています。

> ただし、エフェクトを書くときは常に、更にそのエフェクトをカスタムフックにラップすることでより分かりやすくならないか、検討するようにしてください。エフェクトは頻繁には必要とされないものです。エフェクトを書くということは、外部システムと同期するために「React の外に踏み出す」必要がある、もしくは React に組み込みの API がない何かを行う必要があるということです。カスタムフックにラップすることで、あなたの意図とデータの流れを正確に表現することができます。

ただし、何でもカスタムフックに切り出せばよいわけではありません。

> カスタムフックは具体的かつ高レベルのユースケースに対して使うようにしてください。useEffect API 自体の代替物ないし便利なラッパとして機能させるための、カスタム「ライフサイクル」フックを作ったり使ったりしないようにしてください。

### 思ったこと

カスタムフックは基本的にロジックを再利用するためのものだと思いました。

ただし、ライフサイクルを置き換えるためではなく、`useEffect` が必要な処理をカスタムフックにラップするのはよさそうだと感じました。

## Max Rozenさんのブログ

まずは、RMax Rozenさんです。

公式サイトでは、以下のように自己紹介されています。

> Hi! 👋 I'm Max Rozen.
>
> I'm also known as rozenmd around the internet. I'm an ex-Atlassian, and these days I work at Cloudflare, helping build D1.
>
> I run OnlineOrNot, a service that provides status pages and uptime monitoring for websites, APIs, and cron jobs.
>
> I also run this blog on the side, where I write about React, and provide advice that hopefully helps improve your career as a frontend developer.

日本語訳：

> こんにちは！👋 Max Rozenです。
>
> インターネット上ではrozenmdという名前でも知られています。以前はAtlassianで働いており、現在はCloudflareでD1の開発に携わっています。
>
> また、Webサイト、API、cronジョブのステータスページや稼働監視を提供するサービス「OnlineOrNot」を運営しています。
>
> このブログでは、Reactに関する記事や、フロントエンドエンジニアとしてのキャリアに役立つアドバイスを書いています。

https://maxrozen.com/

今回参考にしたのは、カスタムフックの開発ガイドラインについて書かれた記事です。

https://maxrozen.com/guidelines-for-developing-custom-hooks-react

この記事では、カスタムフックを使う理由として、以下のようなメリットが挙げられています。

> Why custom hooks?
>
> Since hooks can be treated as functions in a lot of ways, what we know about functions still applies.
>
> - They're reusable
> - We can break our larger hooks into smaller hooks
> - We can compose our smaller hooks together, like lego bricks
> - They're testable

日本語訳：

> なぜカスタムフックなのか？
>
> Hookは多くの点で関数として扱えるため、関数について知られていることはHookにも当てはまります。
>
> - 再利用できる
> - 大きなHookを小さなHookに分割できる
> - レゴブロックのように、小さなHookを組み合わせられる
> - テストできる

### 思ったこと

カスタムフックのメリットとして、再利用できることが挙げられていました。

そのため、カスタムフックは再利用できるように作るべきなのかなと思いました。

## TkDodoさんのブログ

続いて、TkDodoさんです。

公式ブログでは、以下のように自己紹介されています。

> 👋 I'm Dominik
>
> 🛠️ I build things for the web, work at Sentry, and help maintain TanStack Query and TanStack Router . I spend a lot of my time thinking about React, TypeScript, and how to make frontend code simpler and more reliable.

日本語訳：

> 👋 Dominikです。
>
> 🛠️ Web向けの開発を行い、Sentryで働いています。また、TanStack QueryとTanStack Routerのメンテナンスにも携わっています。ReactやTypeScript、フロントエンドのコードをよりシンプルで信頼性の高いものにする方法について、多くの時間を費やして考えています。

https://tkdodo.eu/blog

今回参考にしたのは、`useEffect` をより扱いやすくするための原則について書かれた記事です。

https://tkdodo.eu/blog/simplifying-use-effect

この記事では、3つ目の原則として、カスタムフックを書くことが紹介されています。

> 3. Write custom hooks
>
> I really don’t like components where 50% of the code is hook calls. It usually shows that we are mixing our logic with our markup. Tucking them away in a custom hook has multiple advantages, apart from the obvious “you can reuse them”:

日本語訳：

> 3. カスタムフックを書く
>
> コードの50%がHookの呼び出しで占められているようなコンポーネントは、私はあまり好きではありません。通常、これはロジックとマークアップが混在していることを示しています。これらをカスタムフックに切り出すことには、明らかな「再利用できる」というメリット以外にも、複数の利点があります。

そのメリットとして、以下の3つが紹介されています。

> You can name them

日本語訳：

> 名前を付けられる

> You can encapsulate logic

日本語訳：

> ロジックをカプセル化できる

> You can test them in isolation

日本語訳：

> 個別にテストできる

4つ目の原則では、名前を付けることが紹介されています。その中で、カスタムフックを1回しか使用しない場合についても述べられています。

> 4.Give them names
>
> All the above reasons make me want to write custom hooks even if I only use them once. But if you cannot or don’t want to extract it to a custom hook for whatever reason, the function passed to useEffect can still have a name, so consider naming your effects:

日本語訳：

> 4.名前を付ける
>
> ここまでに挙げた理由から、私はカスタムフックを1回しか使用しない場合でも、カスタムフックとして書きたいと思っています。ただし、何らかの理由でカスタムフックに切り出せない、あるいは切り出したくない場合でも、`useEffect` に渡す関数には名前を付けられます。そのため、Effectに名前を付けることを検討してみてください。

### 思ったこと

`useEffect` を持つカスタムフックは、1回しか使用しない場合でもカスタムフックとして書いても良さそうだと思いました。

# まとめ

カスタムフックは、基本的にロジックを再利用するために使用する。

ただし、ライフサイクルを置き換える目的でなければ、再利用しない場合でも、`useEffect` をラップするために使用してもよい。

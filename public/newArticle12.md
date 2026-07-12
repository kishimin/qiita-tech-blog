---
title: playwright.config.ts が include されておらず「Cannot find name 'process'」が発生した
tags:
  - Playwright
private: false
updated_at: ""
id: null
organization_url_name: null
slide: false
ignorePublish: false
posting_campaign_uuid: null
agreed_posting_campaign_term: false
---

## やりたいこと

`playwright.config.ts` で発生した型エラーを解消する。

## 実行したこと

`playwright.config.ts` を開きました。

## 実行結果

`process` に型エラーが表示されました。

## エラー内容

```
Cannot find name 'process'. Do you need to install type definitions for node? Try `npm i --save-dev @types/node` and then add 'node' to the types field in your tsconfig.
```

## 原因調査

エラー文の以下の内容を確認しました。

```txt
then add 'node' to the types field in your tsconfig.
```

## 原因

`playwright.config.ts` が `tsconfig.node.json` の対象に含まれていなかったため、型定義が適用されていませんでした。

## 解決策

`tsconfig.node.json` の `include` に `playwright.config.ts` を追加しました。

```json
{
  "include": ["vite.config.ts", "playwright.config.ts"]
}
```

`playwright.config.ts` が TypeScript の設定対象となり、`process` の型エラーが解消しました。

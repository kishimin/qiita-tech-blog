---
title: 「Do not use a triple slash reference for vitest/config」が発生した
tags:
  - Vitest
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

Vitest の除外対象にデフォルトの除外設定を含めたい。

## エラー内容

```txt
Do not use a triple slash reference for vitest/config, use `import` style instead.
```

## 原因

`vitest.config.ts` で triple-slash reference を使用していました。

```ts
/// <reference types="vitest/config" />
```

`vitest/config` を `import` するように変更したため、triple-slash reference が不要になっていました。

## 解決策

`vitest.config.ts` から以下を削除しました。

```ts
/// <reference types="vitest/config" />
```

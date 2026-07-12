---
title: Zod のスキーマ定義が原因で"Type 'unknown' is not assignable to type ～"が発生した
tags:
  - React
  - フロントエンド
  - Zod
  - Typescript
private: false
updated_at: ""
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

## やりたいこと

API 通信を行うテストを成功させる

## 実行結果

テスト実行時に型エラーが発生しました。

## エラー内容

```ts
Type 'unknown' is not assignable to type 'ItemResponseDto'.
```

## 原因調査

Zod のスキーマ定義を確認しました。

## 原因

`z.object()` を使用せず、通常のオブジェクトとして定義していました。

```ts
export const ItemSchema = {
  id: z.uuid(),
  name: z.string(),
  value: z.int().nullable(),
};
```

## 解決策

`z.object()` を使用して、Zod のオブジェクトスキーマとして定義しました。

```ts
export const ItemSchema = z.object({
  id: z.uuid(),
  name: z.string(),
  value: z.int().nullable(),
});
```

## 学んだこと

Zod でオブジェクトスキーマを定義する場合は、`z.object()` を使用します。

## 参考

https://zod.dev/api

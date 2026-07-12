---
title: Testing Library で Unable to find role="listitem" が発生した原因が Zod の UUID 検証だった
tags:
  - React
  - テスト
  - フロントエンド
  - Zod
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

API 通信後に取得した一覧データのタイトルが表示されることをテストする。

## 実行したコード

```ts
test("タイトルが表示される", async () => {
  setup();

  const items = await screen.findAllByRole("listitem");

  expect(items[0]).toHaveTextContent(sampleItems[0].name);
});
```

```bash
npm run test
```

## 実行結果

```txt
TestingLibraryElementError: Unable to find role="listitem"

Ignored nodes: comments, script, style
```

## エラー内容

`listitem` を取得できず、テストが失敗していました。

## 原因調査

Zod でデータをパースする処理を追加する前は、テストが成功していました。

```ts
data.map((item) => ({
  id: item.id ?? crypto.randomUUID(),
  name: item.name ?? "",
  value: item.value === undefined ? null : item.value,
}));
```

Zod の `safeParse()` を使用するように変更した後、テストが失敗するようになりました。

```ts
data.map((item) => {
  const result = ItemSchema.safeParse({
    id: item.id,
    name: item.name,
    value: item.value,
  });

  if (result.success) {
    return result.data;
  }

  throw new Error("");
});
```

## 原因

テストデータの `id` に空文字を設定していたため、Zod のパースに失敗していました。

```ts
export const sampleItems = [
  {
    id: "",
    value: 100,
    name: "サンプル",
  },
];
```

スキーマでは `id` を UUID として定義しています。

```ts
export const ItemSchema = z.object({
  id: z.uuid(),
  name: z.string(),
  value: z.int().nullable(),
});
```

そのため、空文字の `id` は `safeParse()` に失敗し、一覧データが表示されていませんでした。

## 解決策

テストデータの `id` を UUID として有効な値に変更しました。

```ts
export const sampleItems = [
  {
    id: crypto.randomUUID(),
    value: 100,
    name: "サンプル",
  },
];
```

`crypto.randomUUID()` を使用して UUID を生成することで、Zod のパースに成功し、テストも成功しました。

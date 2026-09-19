---
title: StorybookのInteraction Testでフォーム送信後に「No Preview」が表示された
tags:
  - Storybook
  - React
  - TypeScript
private: false
updated_at: ""
id: null
organization_url_name: null
slide: false
ignorePublish: false
posting_campaign_uuid: null
agreed_posting_campaign_term: false
---

# 概要

StorybookのInteraction Testでフォームの送信ボタンをクリックしたところ、Storyが表示されなくなり「No Preview」が表示されました。

## やりたいこと

StorybookのInteraction Testで、フォームの送信ボタンをクリックした後の表示を確認します。

## 実行したコード

プロジェクト固有の内容を省略したサンプルです。

```ts
export const Progress: Story = {
  play: async ({ step, canvas, userEvent }) => {
    await step("送信ボタンをクリックすると待機状態になる", async () => {
      await userEvent.type(
        canvas.getByRole("textbox", { name: "メッセージ" }),
        "sample",
      );

      await userEvent.click(canvas.getByRole("button", { name: "送信" }));

      await expect(
        canvas.getByRole("heading", {
          name: "処理中です",
        }),
      ).toBeVisible();
    });
  },
};
```

フォームでは、以下のように`onSubmit`を実行していました。

```tsx
<form
  onSubmit={() => {
    void handleSubmit(onSubmit);
  }}
>
  <label htmlFor="message">メッセージ</label>
  <input id="message" {...register("message")} />
  <p>{errors.message?.message}</p>
  <button>送信</button>
</form>
```

## エラー内容

Storybook上で以下のメッセージが表示されました。

```txt
No Preview

Sorry, but you either have no stories or none are selected somehow.

Please check the Storybook config.

Try reloading the page.

If the problem persists, check the browser console, or the terminal you've run Storybook from.
```

## 原因調査

Interaction Testを追加してStorybookを起動するとエラーが発生しました。

Storyのみの場合は正常に表示できました。

調べたところ、以下のボタンクリック時に問題が発生していました。

```ts
await userEvent.click(canvas.getByRole("button", { name: "送信" }));
```

## 原因

フォーム内の`button`をクリックしたことでフォームが送信され、ブラウザのデフォルト動作が発生していました。

その結果、ページの再読み込みが発生し、Storybookで「No Preview」が表示されていました。

## 解決策

`onSubmit`で`preventDefault()`を実行し、フォーム送信時のデフォルト動作をキャンセルしました。

```tsx
<form
  onSubmit={(e) => {
    e.preventDefault();
    void handleSubmit(onSubmit);
  }}
>
  <label htmlFor="message">メッセージ</label>
  <input id="message" {...register("message")} />
  <p>{errors.message?.message}</p>
  <button>送信</button>
</form>
```

追加したのは以下です。

```ts
e.preventDefault();
```

これによってページの再読み込みを防ぐことができました。

## 学んだこと

フォームを送信すると、ブラウザによるデフォルトの送信処理が行われます。

今回のようにJavaScript側でフォーム送信を処理する場合は、`event.preventDefault()`を使うことで、このデフォルト動作をキャンセルできます。

`preventDefault()`を実行しない場合、フォーム送信によってページが再読み込みされ、Storybookの表示にも影響することがあると分かりました。

## 参考

https://github.com/storybookjs/storybook/issues/128

https://qiita.com/yokoto/items/27c56ebc4b818167ef9e

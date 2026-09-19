---
title: React Hook FormでhandleSubmitを呼んでもバリデーションエラーが表示されなかった
tags:
  * React
  * ReactHookForm
  * TypeScript
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

React Hook Formでフォームを送信したときに、バリデーションエラーが表示されませんでした。

## やりたいこと

フォームを送信したときに、入力内容に問題があればバリデーションエラーを表示したいです。

## 実行したコード

StorybookのInteraction Testで、何も入力せずに送信ボタンをクリックします。

```ts
export const SubmitFormError: Story = {
  play: async ({ step, canvas, userEvent }) => {
    await step("入力せずに送信するとエラーが表示される", async () => {
      await userEvent.click(canvas.getByRole("button", { name: "送信" }));

      await expect(canvas.getByText("入力は必須です")).toBeVisible();
    });
  },
};
```

フォームでは以下のように実装していました。

```tsx
export const SampleForm = () => {
  const {
    register,
    formState: { errors },
    handleSubmit,
  } = useForm<SampleSchema>({
    resolver: zodResolver(sampleSchema),
    mode: "onChange",
  });

  const onSubmit = () => console.log("submit");

  return (
    <form
      noValidate
      onSubmit={(e) => {
        e.preventDefault();
        void handleSubmit(onSubmit);
      }}
    >
      <label htmlFor="message">入力</label>
      <input id="message" {...register("message")} />
      <p>{errors.message?.message}</p>
      <button>送信</button>
    </form>
  );
};
```

## 問題

送信ボタンをクリックしても、期待していたバリデーションエラーが表示されませんでした。

## 原因調査

他で実装していたフォームと比較しました。

```tsx
<form
  onSubmit={(e) => {
    return void handleSubmit(onSubmit)(e);
  }}
  noValidate
>
```

比較すると、問題が発生したコードでは以下のようになっていました。

```ts
void handleSubmit(onSubmit);
```

## 原因

`handleSubmit(onSubmit)`が返す関数を実行していなかったことが原因でした。

`handleSubmit(onSubmit)`を呼び出すだけではなく、返された関数を実行する必要があります。

```ts
handleSubmit(onSubmit)(e);
```

## 解決策

`handleSubmit(onSubmit)`が返した関数を、submitイベントを渡して実行するように変更しました。

```tsx id="kfn3mi"
export const SampleForm = () => {
  const {
    register,
    formState: { errors },
    handleSubmit,
  } = useForm<SampleSchema>({
    resolver: zodResolver(sampleSchema),
    mode: "onChange",
  });

  const onSubmit = () => console.log("submit");

  return (
    <form
      noValidate
      onSubmit={(e) => {
        e.preventDefault();
        void handleSubmit(onSubmit)(e);
      }}
    >
      <label htmlFor="message">入力</label>
      <input id="message" {...register("message")} />
      <p>{errors.message?.message}</p>
      <button>送信</button>
    </form>
  );
};
```

修正した箇所は以下です。

```ts
void handleSubmit(onSubmit)(e);
```

## 学んだこと

`handleSubmit(onSubmit)`は、その場で`onSubmit`を実行するものではなく、フォームのsubmitイベントを処理するための関数を返します。

そのため、今回のように`onSubmit`内で呼び出す場合は、返された関数を実行する必要があります。

## 参考

https://react-hook-form.com/docs/useform/handlesubmit

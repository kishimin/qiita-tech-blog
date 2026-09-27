---
title: Parameter cannot have question mark and initializer. の解決方法
tags:
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

E2Eテストでログインを行うヘルパー関数を作成した際に、`Parameter cannot have question mark and initializer.` というエラーが発生したため、解決方法をまとめます。

## やりたいこと

E2Eテストでログインを行うヘルパー関数を作成する。

## 実行したコード

```ts
export const login = async (
  page: Page,
  userName?: string = loginUser.userName,
  password?: string = loginUser.password,
) => {
  await page
    .getByRole("textbox", { name: loginTexts.userNameInput })
    .fill(userName);
  await page.getByLabel(loginTexts.passwordInput).fill(password);
  await page.getByRole("button", { name: loginTexts.loginButton }).click();
};
```

## エラー内容

`userName` と `password` の引数で以下のエラーが発生しました。

```text
Parameter cannot have question mark and initializer.
```

## 原因

TypeScriptでは、デフォルト値を指定した引数に `?` は使用できません。

## 解決策

デフォルト値を指定した引数から `?` を削除しました。

```ts
export const login = async (
  page: Page,
  userName: string = loginUser.userName,
  password: string = loginUser.password,
) => {
  await page
    .getByRole("textbox", { name: loginTexts.userNameInput })
    .fill(userName);
  await page.getByLabel(loginTexts.passwordInput).fill(password);
  await page.getByRole("button", { name: loginTexts.loginButton }).click();
};
```

## 学んだこと

TypeScriptでは、デフォルト値を指定した引数に `?` を付けることはできません。

## 参考

- [TypeScript公式：More on Functions - Optional Parameters](https://www.typescriptlang.org/docs/handbook/2/functions.html#optional-parameters)
- [TypeScript公式：Functions - Optional and Default Parameters](https://www.typescriptlang.org/docs/handbook/functions#optional-and-default-parameters)

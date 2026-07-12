---
title: Storybook のすべての Story に MUI のテーマを適用したい
tags:
  - フロントエンド
  - Storybook
  - mui
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

Storybook のすべての Story に MUI のテーマを適用したい。

## 実行結果

Story ごとに `ThemeProvider` を設定するとテーマが適用されました。

## 問題

`ThemeProvider` を設定していない Story には、MUI のテーマが適用されていませんでした。

## 原因調査

Story ごとに `ThemeProvider` を設定するとテーマが適用されることを確認しました。

Storybook 全体の設定を行う `preview.tsx` を確認しました。

## 原因

`preview.tsx` で MUI の `ThemeProvider` を設定していませんでした。

そのため、Story 全体にテーマが適用されていませんでした。

## 解決策

`preview.tsx` の `decorators` で Story 全体を `ThemeProvider` で囲みました。

```tsx
import { ThemeProvider } from "@mui/material/styles";
import { theme } from "../src/theme";

const preview: Preview = {
  decorators: [
    (Story) => (
      <ThemeProvider theme={theme}>
        <Story />
      </ThemeProvider>
    ),
  ],
};
```

Story 全体を `ThemeProvider` で囲むことで、MUI のテーマが適用されました。

## 学んだこと

Storybook で全 Story に共通の Provider を適用する場合は、`preview.tsx` の `decorators` を使用できます。

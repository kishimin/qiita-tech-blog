---
title: 【MUI v6】Checkbox・Radio・Autocompleteのテスト方法
tags:
  - Frontend
  - test
  - MUI
  - Jest
  - RTL
private: false
updated_at: ""
id: null
organization_url_name: null
slide: false
ignorePublish: true
posting_campaign_uuid: null
agreed_posting_campaign_term: false
---

## はじめに

Material UI v6・Jest・React Testing Libraryを使用して、CheckboxのDOM取得から入力、検証までの方法をまとめます。

RadioとAutocompleteのテストは今後追記します。

## 対象コンポーネント

- Checkbox
- Radio（未完成）
- Autocomplete（未完成）

## Checkbox

### 実装コード

```ts: checkbox.tsx
import {
  Checkbox,
  FormControl,
  FormControlLabel,
  FormHelperText,
} from "@mui/material";
import { useState } from "react";

export const CheckboxForm = () => {
  const [isNonbiri, setIsNonbiri] = useState<boolean>(false);

  return (
    <FormControl>
      <FormControlLabel
        label={"のんびり"}
        control={
          <Checkbox
            checked={isNonbiri}
            onChange={(_, checked) => setIsNonbiri(checked)}
            slotProps={{ input: { "aria-describedby": "check-description" } }}
          />
        }
      />

      <FormHelperText id={"check-description"}>{"のんびり"}</FormHelperText>
    </FormControl>
  );
};

```

### テスト準備

```ts
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { CheckboxForm } from "./checkbox";

const setup = () => {
  const user = userEvent.setup();

  render(<CheckboxForm />);

  return { user };
};
```

### DOMの表示の検証

```ts
test("のんびりというチェックボックスが表示される", () => {
  setup();

  expect(
    screen.getByRole("checkbox", { name: "のんびり" }),
  ).toBeInTheDocument();
});
```

### 入力の検証

```ts
test("のんびりというチェックボックスを切り替えることができる", async () => {
  const { user } = setup();

  const checkbox = screen.getByRole("checkbox", { name: "のんびり" });

  expect(checkbox).not.toBeChecked();

  await user.click(checkbox);

  expect(checkbox).toBeChecked();
});
```

### ヘルパーテキスト

```ts
test("のんびりというチェックボックスのヘルパーテキストが表示される", () => {
  setup();

  expect(
    screen.getByRole("checkbox", { name: "のんびり" }),
  ).toHaveAccessibleDescription("のんびり");
});
```

### 注意点

#### 1. input の表示の検証

MUI v6.4.12 の Checkbox は、内部で `SwitchBase` を使っています。`SwitchBase` は、操作領域の土台となる root と、実際の入力を担う input の両方を持ち、チェック状態に応じたアイコンも表示します。

```text
Checkbox
└─ CheckboxRoot（SwitchBase をスタイルしたコンポーネント）
   └─ SwitchBase
      └─ RootSlot（既定: SwitchBaseRoot）
         ├─ InputSlot（既定: SwitchBaseInput）
         │  └─ input type="checkbox"（opacity: 0）
         └─ チェック状態に応じたアイコン
```

`opacity: 0` は、`SwitchBaseInput` のスタイルとして input に指定されています。見た目はアイコンで表現する一方、input はチェック状態や入力イベントを扱います。

`getByRole("checkbox")` が取得するのは、`checkbox` ロールを持つ input です。jest-dom の `toBeVisible()` は `opacity: 0` の要素を非表示と判定するため、この input に対する `toBeVisible()` は失敗します。

そのため、`toBeVisible()` の代わりに `toBeInTheDocument()` を使い、input が DOM に存在することを確認しています。これはアイコンの視覚的な表示を保証するものではありません。

参考（MUI v6.4.12 の実装）：

https://github.com/mui/material-ui/blob/v6.4.12/packages/mui-material/src/internal/SwitchBase.js

https://github.com/mui/material-ui/blob/v6.4.12/packages/mui-material/src/Checkbox/Checkbox.js

参考（jest-dom の各 matcher の仕様）：

https://github.com/testing-library/jest-dom#tobevisible

https://github.com/testing-library/jest-dom#tobeinthedocument

#### 2. ヘルパーテキストの紐づけ

`slotProps` の `input` に指定した props は、内部の `<input type="checkbox">` に指定されます。

そこで、input とヘルパーテキストを紐づけるために、`slotProps` の `input` に `aria-describedby` を指定しています。値にはヘルパーテキストの `id` を指定しています。

https://mui.com/material-ui/api/checkbox/

## Radio

### テスト準備

### DOMの表示の検証

### 入力の検証

### ヘルパーテキスト

## Autocomplete

### テスト準備

### DOMの表示の検証

### 入力の検証

### ヘルパーテキスト

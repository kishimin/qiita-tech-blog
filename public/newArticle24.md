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
ignorePublish: false
posting_campaign_uuid: null
agreed_posting_campaign_term: false
---

## はじめに

Material UI v6・Jest・React Testing Libraryを使用して、Checkbox・Radio・AutocompleteのDOM取得から入力、検証までの方法をまとめます。

## 対象コンポーネント

- Checkbox
- Radio
- Autocomplete

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

MUI v6.4.12 の Checkbox は、内部で `SwitchBase` を使っている。`SwitchBase` は、操作領域の土台となる root と、実際の入力を担う input の両方を持ち、チェック状態に応じたアイコンも表示している。

```text
Checkbox
└─ CheckboxRoot（SwitchBase をスタイルしたコンポーネント）
   └─ SwitchBase
      └─ RootSlot（既定: SwitchBaseRoot）
         ├─ InputSlot（既定: SwitchBaseInput）
         │  └─ input type="checkbox"（opacity: 0）
         └─ チェック状態に応じたアイコン
```

`opacity: 0` は、`SwitchBaseInput` のスタイルとして input に指定されている。見た目はアイコンで表現する一方、input はチェック状態や入力イベントを扱っている。

`getByRole("checkbox")` が取得するのは、`checkbox` ロールを持つ input である。jest-dom の `toBeVisible()` は `opacity: 0` の要素を非表示と判定するため、この input に対する `toBeVisible()` は失敗する。

そのため、`toBeVisible()` ではなく `toBeInTheDocument()` で input が DOM に存在することを確認する。

ただし、アイコンが表示されていることは確認できない。

参考（MUI v6.4.12 の実装）：

https://github.com/mui/material-ui/blob/v6.4.12/packages/mui-material/src/internal/SwitchBase.js

https://github.com/mui/material-ui/blob/v6.4.12/packages/mui-material/src/Checkbox/Checkbox.js

参考（jest-dom の各 matcher の仕様）：

https://github.com/testing-library/jest-dom#tobevisible

https://github.com/testing-library/jest-dom#tobeinthedocument

#### 2. ヘルパーテキストの紐づけ

`slotProps` の `input` に指定した props は、内部の `<input type="checkbox">` に指定される。

`slotProps` の `input` に指定した `aria-describedby` とヘルパーテキストの `id` を紐づけないと、input とヘルパーテキストが紐づかない。

https://mui.com/material-ui/api/checkbox/

## Radio

### 実装コード

```ts: radio.tsx
import {
  FormControl,
  FormControlLabel,
  FormHelperText,
  FormLabel,
  Radio,
  RadioGroup,
} from "@mui/material";
import { useState } from "react";

const PACETYPE = {
  NOT: "未設定",
  NONBIRI: "のんびり",
  MYPACE: "マイペース",
} as const;

type PaceType = (typeof PACETYPE)[keyof typeof PACETYPE];

export const RadioForm = () => {
  const [type, setType] = useState<PaceType>(PACETYPE.NOT);

  return (
    <FormControl>
      <FormLabel id={"radio"}>{"のんびり"}</FormLabel>

      <RadioGroup
        aria-labelledby={"radio"}
        aria-describedby={"radio-description"}
        value={type}
        onChange={(e) => setType(e.target.value as PaceType)}
      >
        <FormControlLabel
          value={PACETYPE.NOT}
          control={<Radio />}
          label={PACETYPE.NOT}
        />
        <FormControlLabel
          value={PACETYPE.NONBIRI}
          control={<Radio />}
          label={PACETYPE.NONBIRI}
        />
        <FormControlLabel
          value={PACETYPE.MYPACE}
          control={<Radio />}
          label={PACETYPE.MYPACE}
        />
      </RadioGroup>

      <FormHelperText id={"radio-description"}>{"のんびり"}</FormHelperText>
    </FormControl>
  );
};
```

### テスト準備

```ts
import { render, screen, within } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { RadioForm } from "./radio";

const setup = () => {
  const user = userEvent.setup();

  render(<RadioForm />);

  return { user };
};
```

### DOMの表示の検証

```ts
test("のんびりというラジオが表示される", () => {
  setup();

  expect(screen.getByRole("radiogroup", { name: "のんびり" })).toBeVisible();
});
```

### 選択肢の検証

```ts
test("のんびりというラジオの選択肢が仕様通りである", () => {
  setup();

  const group = screen.getByRole("radiogroup", { name: "のんびり" });
  const items = within(group).getAllByRole("radio");
  const expected = ["未設定", "のんびり", "マイペース"];

  expect(items).toHaveLength(expected.length);
  items.forEach((item, index) => {
    expect(item).toHaveAccessibleName(expected[index]);
  });
});
```

### 入力の検証

```ts
test("のんびりというラジオの初期値は未設定である", () => {
  setup();

  expect(screen.getByRole("radio", { name: "未設定" })).toBeChecked();
});

test("のんびりというラジオでのんびりを選択できる", async () => {
  const { user } = setup();

  await user.click(screen.getByRole("radio", { name: "のんびり" }));

  expect(screen.getByRole("radio", { name: "のんびり" })).toBeChecked();
  expect(screen.getByRole("radio", { name: "未設定" })).not.toBeChecked();
});
```

### ヘルパーテキスト

```ts
test("のんびりというラジオのヘルパーテキストが表示される", () => {
  setup();

  expect(
    screen.getByRole("radiogroup", { name: "のんびり" }),
  ).toHaveAccessibleDescription("のんびり");
});
```

### 注意点

ラベルを `id` と `aria-labelledby` で紐づけないと、グループを role と name で取得できない。

また、`id` と `aria-describedby` で紐づけないと、グループとヘルパーテキストが紐づかない。

https://v6.mui.com/material-ui/react-radio-button/

## Autocomplete

### 実装コード

```ts: autocomplete.tsx
import {
  Autocomplete,
  FormControl,
  FormHelperText,
  FormLabel,
  OutlinedInput,
} from "@mui/material";
import { useState } from "react";

const PACETYPE = {
  NOT: "未設定",
  NONBIRI: "のんびり",
  MYPACE: "マイペース",
} as const;

type PaceType = (typeof PACETYPE)[keyof typeof PACETYPE];

export const AutocompleteForm = () => {
  const [type, setType] = useState<PaceType | null>(null);

  return (
    <FormControl>
      <FormLabel htmlFor={"autocomplete"}>{"のんびり"}</FormLabel>

      <Autocomplete
        id={"autocomplete"}
        options={[PACETYPE.NOT, PACETYPE.NONBIRI, PACETYPE.MYPACE]}
        value={type}
        onChange={(_, value) => setType(value)}
        renderInput={(params) => (
          <OutlinedInput
            {...params.InputProps}
            id={params.id}
            inputProps={{
              ...params.inputProps,
              "aria-describedby": "autocomplete-description",
            }}
          />
        )}
      />

      <FormHelperText id={"autocomplete-description"}>{"のんびり"}</FormHelperText>
    </FormControl>
  );
};
```

### テスト準備

```ts
import { render, screen, within } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { AutocompleteForm } from "./autocomplete";

const setup = () => {
  const user = userEvent.setup();

  render(<AutocompleteForm />);

  return { user };
};
```

### DOMの表示の検証

```ts
test("のんびりというオートコンプリートが表示される", () => {
  setup();

  expect(screen.getByRole("combobox", { name: "のんびり" })).toBeVisible();
});
```

### 選択肢の検証

```ts
test("のんびりというオートコンプリートの選択肢が仕様通りである", async () => {
  const { user } = setup();

  const input = screen.getByRole("combobox", { name: "のんびり" });

  await user.click(input);

  const list = await screen.findByRole("listbox");
  expect(list).toBeVisible();

  const items = within(list).getAllByRole("option");
  const expected = ["未設定", "のんびり", "マイペース"];

  expect(items).toHaveLength(expected.length);
  items.forEach((item, index) => {
    expect(item).toHaveTextContent(expected[index]);
  });
});
```

### 入力の検証

```ts
test("のんびりというオートコンプリートの初期値は空である", () => {
  setup();

  expect(screen.getByRole("combobox", { name: "のんびり" })).toHaveValue("");
});

test("のんびりというオートコンプリートでのんびりを選択できる", async () => {
  const { user } = setup();

  const input = screen.getByRole("combobox", { name: "のんびり" });

  await user.click(input);
  await user.click(screen.getByRole("option", { name: "のんびり" }));

  expect(input).toHaveValue("のんびり");
});

test("のんびりというオートコンプリートで候補を絞り込んで選択できる", async () => {
  const { user } = setup();
  const input = screen.getByRole("combobox", { name: "のんびり" });

  await user.clear(input);
  await user.type(input, "のん");

  expect(screen.getByRole("option", { name: "のんびり" })).toBeVisible();
  expect(
    screen.queryByRole("option", { name: "未設定" }),
  ).not.toBeInTheDocument();

  await user.click(screen.getByRole("option", { name: "のんびり" }));

  expect(input).toHaveValue("のんびり");
});
```

### ヘルパーテキスト

```ts
test("のんびりというオートコンプリートのヘルパーテキストが表示される", () => {
  setup();

  expect(
    screen.getByRole("combobox", { name: "のんびり" }),
  ).toHaveAccessibleDescription("のんびり");
});
```

### 注意点

#### 値の検証

Autocomplete の `combobox` は入力欄なので、選択した値の確認には `toHaveValue` を使う。

#### 入力欄への props の引き継ぎ

`params.inputProps` を `OutlinedInput` の `inputProps` に渡すのは、Autocomplete が入力欄の操作に必要とする props を、実際の HTML `<input>` に届けるためである。

```tsx
<OutlinedInput
  {...params.InputProps}
  id={params.id}
  inputProps={{
    ...params.inputProps,
    "aria-describedby": "autocomplete-description",
  }}
/>
```

`params.inputProps` には、Autocomplete が入力値やイベント、role・ARIA 属性などを扱うための props が含まれる。これらを引き継ぐことで、入力による候補の絞り込みやキーボード操作、候補リストとの関連付けが動作する。

https://v4.mui.com/ja/components/autocomplete/#custom-input

- `params.InputProps`：`OutlinedInput` 側に渡す props。ref や候補を開くボタンなどを含む。
- `params.inputProps`：その内側の HTML `<input>` に渡す props。

`aria-describedby` を指定して、この入力欄と「のんびり」というヘルパーテキストが関連付ける。

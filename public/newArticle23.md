---
title: 【MUI v6】TextField・OutlinedInput・Selectのテスト方法
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

Material UIを使用した画面のテストを書く際、各コンポーネントのDOM取得方法や操作方法を調べても、まとまった記事が見つかりませんでした。

そこで本記事では、Material UI v6・Jest・React Testing Libraryを使用して、TextField・OutlinedInput・SelectのDOM取得から入力、検証までの方法をまとめます。

## 対象コンポーネント

- TextField
- OutlinedInput
- Select

## TextField

### 実装コード

```ts: text-field.tsx
import { TextField } from "@mui/material";
import { useState } from "react";

export const TextFieldForm = () => {
  const [text, setText] = useState<string>("");

  return (
    <TextField
      label={"のんびり"}
      onChange={(e) => setText(e.target.value)}
      value={text}
      helperText={"のんびり"}
    />
  );
};

```

### テスト準備

```ts
import { render, screen } from "@testing-library/react";
import { TextFieldForm } from "./text-field";
import userEvent from "@testing-library/user-event";

const setup = () => {
  const user = userEvent.setup();

  render(<TextFieldForm />);

  return { user };
};

```

### DOMの表示の検証

`role`が`textbox`

```ts
test("のんびりというテキスト入力が表示される", () => {
  setup();

  expect(screen.getByRole("textbox", { name: "のんびり" })).toBeVisible();
});
```

### 入力の検証

`type`で文字を入力

```ts
test("のんびりというテキスト入力にのんびりと入力するとのんびりという値が表示される", async () => {
  const { user } = setup();

  await user.type(
    screen.getByRole("textbox", { name: "のんびり" }),
    "のんびり",
  );

  expect(screen.getByRole("textbox", { name: "のんびり" })).toHaveValue(
    "のんびり",
  );
});
```

### ヘルパーテキスト

```ts
test("のんびりというテキスト入力のヘルパーテキストが表示される", () => {
  setup();

  expect(
    screen.getByRole("textbox", { name: "のんびり" }),
  ).toHaveAccessibleDescription("のんびり");
});
```

## OutlinedInput

### 実装コード

```ts: outlined-input.tsx
import {
  FormControl,
  FormHelperText,
  FormLabel,
  OutlinedInput,
} from "@mui/material";
import { useState } from "react";

export const OutlinedInputForm = () => {
  const [text, setText] = useState<string>("");

  return (
    <FormControl>
      <FormLabel htmlFor={"text"}>{"のんびり"}</FormLabel>

      <OutlinedInput
        id={"text"}
        value={text}
        onChange={(e) => setText(e.target.value)}
        aria-describedby={"text-description"}
      />

      <FormHelperText id={"text-description"}>{"のんびり"}</FormHelperText>
    </FormControl>
  );
};

```

### テスト準備

```ts
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { OutlinedInputForm } from "./outlined-input";

const setup = () => {
  const user = userEvent.setup();

  render(<OutlinedInputForm />);

  return { user };
};
```

### DOMの表示の検証

```ts
test("のんびりというテキスト入力が表示される", () => {
  setup();

  expect(screen.getByRole("textbox", { name: "のんびり" })).toBeVisible();
});
```

### 入力の検証

```ts
test("のんびりというテキスト入力にのんびりと入力するとのんびりという値が表示される", async () => {
  const { user } = setup();

  await user.type(
    screen.getByRole("textbox", { name: "のんびり" }),
    "のんびり",
  );

  expect(screen.getByRole("textbox", { name: "のんびり" })).toHaveValue(
    "のんびり",
  );
});
```

### ヘルパーテキスト

```ts
test("のんびりというテキスト入力のヘルパーテキストが表示される", () => {
  setup();

  expect(
    screen.getByRole("textbox", { name: "のんびり" }),
  ).toHaveAccessibleDescription("のんびり");
});
```

### 注意点

ラベルをidとhtmlForで紐づけないとroleとnameで取得できない。

また、idとaria-describedbyで紐づけないと入力とヘルパーテキストが紐づかない

## Select

### 実装コード

```ts: select-field.tsx
import {
  FormControl,
  FormHelperText,
  FormLabel,
  MenuItem,
  Select,
} from "@mui/material";
import { useState } from "react";

export const PACETYPE = {
  NOT: "未設定",
  NONBIRI: "のんびり",
  MYPACE: "マイペース",
} as const;

export type PaceType = (typeof PACETYPE)[keyof typeof PACETYPE];

export const SelectForm = () => {
  const [type, setType] = useState<PaceType>(PACETYPE.NOT);

  return (
    <FormControl>
      <FormLabel id={"select"}>{"のんびり"}</FormLabel>

      <Select
        labelId={"select"}
        value={type}
        onChange={(e) => setType(e.target.value)}
        aria-describedby={"select-description"}
      >
        <MenuItem value={PACETYPE.NOT}>{PACETYPE.NOT}</MenuItem>
        <MenuItem value={PACETYPE.NONBIRI}>{PACETYPE.NONBIRI}</MenuItem>
        <MenuItem value={PACETYPE.MYPACE}>{PACETYPE.MYPACE}</MenuItem>
      </Select>

      <FormHelperText id={"select-description"}>{"のんびり"}</FormHelperText>
    </FormControl>
  );
};

```

### テスト準備

```ts
import { render, screen, within } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { SelectForm } from "./select-field";

const setup = () => {
  const user = userEvent.setup();

  render(<SelectForm />);

  return { user };
};

```

### DOMの表示の検証

```ts
test("のんびりというセレクトが表示される", () => {
  setup();

  expect(screen.getByRole("combobox", { name: "のんびり" })).toBeVisible();
});
```

### 選択肢の検証

```ts
test("のんびりというセレクトの選択肢が仕様通りである", async () => {
  const { user } = setup();
  const select = screen.getByRole("combobox", { name: "のんびり" });

  await user.click(select);

  const list = await screen.findByRole("listbox");
  expect(list).toBeVisible();

  const items = within(list).getAllByRole("option");
  const expected = ["未設定", "のんびり", "マイペース"];
  items.forEach((item, index) => {
    expect(item).toHaveTextContent(expected[index]);
  });
});
```

### 入力の検証

```ts
test("のんびりというセレクトでのんびりを選択できる", async () => {
  const { user } = setup();
  const select = screen.getByRole("combobox", { name: "のんびり" });
  await user.click(select);

  await user.click(screen.getByRole("option", { name: "のんびり" }));

  expect(select).toHaveTextContent("のんびり");
});
```

### ヘルパーテキスト

```ts
test("のんびりというセレクトのヘルパーテキストが表示される", () => {
  setup();

  expect(
    screen.getByRole("combobox", { name: "のんびり" }),
  ).toHaveAccessibleDescription("のんびり");
});
```

### 注意点

ラベルをidとlabelIdで紐づけないとroleとnameで取得できない。

また、idとaria-describedbyで紐づけないと入力とヘルパーテキストが紐づかない

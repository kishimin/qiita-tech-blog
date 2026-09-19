---
title: StorybookでMemoryRouterを使用した際に「basename」がnullになるエラーが発生した
tags:
  - Storybook
  - React
  - ReactRouter
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

StorybookでReact Routerの`Link`を使用しているコンポーネントを表示したところ、以下のエラーが発生しました。

```txt
Cannot destructure property 'basename' of 'import_react.useContext(...)' as it is null.
```

## やりたいこと

React Routerの`Link`を使用しているコンポーネントのStoryを表示したい。

## 実行したコード

`preview.tsx`

```tsx
import type { Preview } from "@storybook/react-vite";
import { MemoryRouter } from "storybook/internal/router";

const preview: Preview = {
  parameters: {
    controls: {
      matchers: {
        color: /(background|color)$/i,
        date: /Date$/i,
      },
    },
    a11y: {
      test: "todo",
    },
  },
  tags: ["autodocs"],
  decorators: [
    (Story) => (
      <MemoryRouter>
        <Story />
      </MemoryRouter>
    ),
  ],
};

export default preview;
```

表示するコンポーネントでは、React Routerの`Link`を使用しています。

```tsx
import { Link } from "react-router";

export const Links = () => {
  return (
    <ul>
      <li>
        <Link to="/sample">サンプル</Link>
      </li>
    </ul>
  );
};
```

Interaction Testでは、リンクが表示されることを確認しています。

```ts
await step("リンクが表示される", async () => {
  const expected = ["サンプル"];

  for (let index = 0; index < listItems.length; index++) {
    await expect(
      within(listItems[index]).getByRole("link", {
        name: expected[index],
      }),
    ).toBeVisible();
  }
});
```

## エラー内容

```txt
Cannot destructure property 'basename' of 'import_react.useContext(...)' as it is null.
```

## 原因調査

エラーについて検索しました。

調べると、`MemoryRouter`で囲っていないことが原因という情報が出てきました。

しかし、`preview.tsx`ではすでに`MemoryRouter`で囲っていました。

```tsx
<MemoryRouter>
  <Story />
</MemoryRouter>
```

そこで`MemoryRouter`のimport元を確認しました。

## 原因

`MemoryRouter`をStorybook内部のモジュールからimportしていたことが原因でした。

```ts
import { MemoryRouter } from "storybook/internal/router";
```

## 解決策

`MemoryRouter`を`react-router`からimportするように変更しました。

```ts
import { MemoryRouter } from "react-router";
```

## 参考

https://qiita.com/kikotkk/items/9d972681315d48e9fa17

https://qiita.com/pumarukkk/items/bf131b7a681a24aec2e6

---
title: Vitest が Playwright の E2E テストを実行して CI 上で失敗した
tags:
  - Playwright
  - Vitest
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

CI 上でテストを成功させる。

## 実行したこと

Playwright を導入しました。

```bash
npm init playwright@latest
```

その後、変更を push しました。

## 実行結果

CI 上でテストが失敗しました。

## エラー内容

```
FAIL   0  e2e/example.spec.ts [ e2e/example.spec.ts ]
Error: Playwright Test did not expect test() to be called here.
Most common reasons include:
- You are calling test() in a configuration file.
- You are calling test() in a file that is imported by the configuration file.
- You have two different versions of @playwright/test. This usually happens
  when one of the dependencies in your package.json depends on @playwright/test.
- You are calling test() from an async test.describe() block. Only sync ones are supported.
 ❯ _TestTypeImpl._currentSuite node_modules/playwright/lib/common/index.js:2257:13
 ❯ _TestTypeImpl._createTest node_modules/playwright/lib/common/index.js:2271:24
 ❯ node_modules/playwright/lib/common/index.js:1220:12
 ❯ e2e/example.spec.ts:3:1
      1| import { test, expect } from '@playwright/test';
      2|
      3| test('has title', async ({ page }) => {
       | ^
      4|   await page.goto('https://playwright.dev/');
      5|
```

## 原因調査

エラー内容を日本語に訳して確認しました。

Playwright Test では、ここで test() が呼び出されることは想定されていませんでした。

以下のエラー内容で検索しました。

Error: Playwright Test did not expect test() to be called here.

## 原因

Vitest が `e2e` ディレクトリ内の Playwright のテストをテスト対象として実行していました。

そのため、Playwright のテストを Vitest で実行しようとしてエラーが発生していました。

## 解決策

Vitest のテスト対象から `e2e` ディレクトリを除外しました。

```ts
import { configDefaults } from "vitest/config";

export default defineConfig({
  test: {
    exclude: [...configDefaults.exclude, "e2e/"],
  },
});
```

`e2e` ディレクトリを除外することで、Playwright のテストが Vitest から実行されなくなり、CI 上のテストが成功しました。

## 参考

https://github.com/vitalets/playwright-bdd/issues/171#issuecomment-2909298414

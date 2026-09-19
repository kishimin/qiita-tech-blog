---
title: Playwrightでリンクをクリックしても画面遷移しなかった原因
tags:
  - Playwright
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

Playwrightでリンクをクリックして画面遷移するE2Eテストを作成したところ、期待したURLへ遷移せずテストが失敗しました。

原因は、リンクではなく`listitem`をクリックしていたことでした。

## やりたいこと

Playwrightでリンクをクリックし、別の画面へ遷移することを確認したい。

## 実行したコード

```ts
test("リンクをクリックすると別の画面に遷移する", async ({ page }) => {
  const samplePage = new SamplePage(page);
  const destinationPage = new DestinationPage(page);

  await samplePage.goto();

  await samplePage.gotoDestination();

  await expect(page).toHaveURL("/sample");

  await expect(destinationPage.getPageTitle).toBeVisible();
});
```

Page Objectでは、以下のようにリンクを取得していました。

```ts
export class SamplePage extends BasePage {
  readonly getList: Locator;
  readonly getSampleLink: Locator;

  constructor(page: Page) {
    super(page);

    this.getList = page.getByRole("list");

    this.getSampleLink = this.getList.getByRole("listitem").filter({
      has: page.getByRole("link", { name: /サンプル/ }),
    });
  }

  async goto() {
    await this.page.goto("/");
  }

  async gotoDestination() {
    await this.getSampleLink.click();
  }
}
```

## 実行結果

テストが失敗しました。

## エラー内容

期待していたURLへ遷移せず、元のURLのままでした。

```txt
Error: expect(page).toHaveURL(expected) failed

Expected: "http://localhost:5173/sample"
Received: "http://localhost:5173/"
Timeout: 5000ms
```

## 原因調査

ローカルで実際にリンクをクリックすると、正常に画面遷移しました。

また、Playwrightで以下のようにリンクを直接取得してクリックした場合も成功しました。

```ts
await page.getByRole("link", { name: /サンプル/ }).click();
```

Page Objectで定義していたLocatorを確認しました。

```ts
this.getSampleLink = this.getList.getByRole("listitem").filter({
  has: page.getByRole("link", { name: /サンプル/ }),
});
```

## 原因

リンクではなく`listitem`をクリックしていたことが原因でした。

`filter()`は、指定した条件に一致する要素に絞り込むためのものです。

以下のコードでは、リンクを持つ`listitem`に絞り込んでいるだけなので、取得されるLocatorは`listitem`のままでした。

```ts
this.getList.getByRole("listitem").filter({
  has: page.getByRole("link", { name: /サンプル/ }),
});
```

そのため、`click()`ではリンクではなく`listitem`をクリックしていました。

## 解決策

`listitem`から、その中にあるリンクを取得するように変更しました。

```ts
this.getSampleLink = this.getList
  .getByRole("listitem")
  .getByRole("link", { name: /サンプル/ });
```

## 学んだこと

Playwrightの`filter()`は子要素を取得するものではなく、現在のLocatorを条件によって絞り込むためのものだと分かりました。

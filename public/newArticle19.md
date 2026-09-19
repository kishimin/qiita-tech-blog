---
title: PlaywrightのE2Eテストがモバイル環境でのみタイムアウトした
tags:
  - Playwright
  - TypeScript
  - E2E
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

PlaywrightでE2Eテストを実行したところ、デスクトップ環境では成功しましたが、モバイル環境でのみリンクのクリックがタイムアウトしました。

原因は、モバイル表示時に大きな画像がリンクに重なり、クリックを妨げていたことでした。

## やりたいこと

Playwrightでリンクをクリックし、別の画面へ遷移するE2Eテストを成功させたい。

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

Page Objectでは、リンクを取得してクリックしていました。

```ts
export class SamplePage extends BasePage {
  readonly getList: Locator;
  readonly getSampleLink: Locator;

  constructor(page: Page) {
    super(page);

    this.getList = page.getByRole("list");
    this.getSampleLink = this.getList
      .getByRole("listitem")
      .getByRole("link", { name: /サンプル/ });
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

デスクトップ環境では成功しましたが、モバイル環境（Pixel 5）でのみテストが失敗しました。

## エラー内容

リンクのクリックでタイムアウトしました。

```txt
Test timeout of 30000ms exceeded.

Error: locator.click: Test timeout of 30000ms exceeded.
```

Call logを確認すると、リンク自体は表示されていましたが、画像がクリックを妨げていました。

```txt
element is visible, enabled and stable
scrolling into view if needed
done scrolling
<img ... /> from <header>…</header> subtree intercepts pointer events
```

## 原因調査

モバイル環境で画面を確認したところ、画像が大きく表示されていました。

PlaywrightのCall logにも、画像がpointer eventsを妨げていることが表示されていました。

```txt
<img ... /> from <header>…</header> subtree intercepts pointer events
```

## 原因

モバイル表示時に画像が大きく表示され、リンクに重なっていたことが原因でした。

そのため、Playwrightがリンクをクリックしようとしても画像がクリックを妨げ、クリックできないままタイムアウトしていました。

## 解決策

モバイル表示でもリンクに重ならないように、画像のサイズを小さくしました。

画像のサイズを修正したことで、リンクをクリックできるようになり、E2Eテストも成功しました。

## 学んだこと

Playwrightのテストが特定の画面サイズでのみ失敗する場合は、テストコードだけでなく、レスポンシブ表示による要素の重なりも確認する必要があると分かりました。

また、Call logの`intercepts pointer events`から、別の要素がクリックを妨げていることを確認できました。

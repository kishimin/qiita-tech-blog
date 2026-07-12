---
title: "Assets in public directory cannot be imported from JavaScript." のエラーを解消する
tags:
  - テスト
  - フロントエンド
  - React
private: false
updated_at: '2026-07-12T17:44:24+09:00'
id: 828de84417f2d78dd31e
organization_url_name: null
slide: false
ignorePublish: false
posting_campaign_uuid: null
agreed_posting_campaign_term: false
---

## やりたいこと

テスト実行時にコマンドラインへ表示されるエラーを解消する。

## 実行したコード

```bash
npm run test
```

## 実行結果

テスト実行時に、コマンドラインへエラーが表示されました。

## エラー内容

```
Assets in public directory cannot be imported from JavaScript. If you intend to import that asset, put the file in the src directory, and use /src/images/SampleLogoImage.png instead of /public/images/SampleLogoImage.png. If you intend to use the URL of that asset, use /images/SampleLogoImage.png?url. Files in the public directory are served at the root path. Instead of /public/images/SampleLogoImage.png, use /images/SampleLogoImage.png.
```

## 原因調査

エラー内容を日本語に訳して確認しました。

```
public ディレクトリ内のアセットは、JavaScript からインポートすることはできません。そのアセットをインポートしたい場合は、ファイルを src ディレクトリに配置し、/public/images/SampleLogoImage.png の代わりに /src/images/SampleLogoImage.png を使用してください。そのアセットの URL を使用したい場合は、/images/SampleLogoImage.png?url を使用してください。public ディレクトリ内のファイルは、ルートパスから配信されます。/public/images/SampleLogoImage.png の代わりに、/images/SampleLogoImage.png を使用してください。
```

## 原因

`public` ディレクトリに配置した画像を import していました。

`public` ディレクトリ内のファイルはルートパスから参照する必要があります。

## 解決策

画像を `public/images` から `src/images` に移動しました。

```txt
public/images/SampleLogoImage.png
↓
src/images/SampleLogoImage.png
```

`src` ディレクトリに移動することで、 import できるようになり、エラーが解消しました。

---
title: Bun 経由の TypeScript 型チェックと依存先の型宣言
tags:
  - Bun
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

Bun 経由で TypeScript の型チェックを実行したところ、依存ライブラリの型宣言ファイルでエラーが発生した。この記事では、一時的に型宣言の検査を省く方法をまとめる。

## やりたいこと

プロジェクトのディレクトリでアプリの TypeScript コードを型チェックする。

## 実行したコード

```powershell
bunx tsc --noEmit
```

## 実行結果

型宣言のエラーにより失敗した。同じ環境で確認した結果は次のとおり。

| コマンド | 結果 |
| --- | --- |
| `bunx tsc --noEmit` | 型宣言のエラーで失敗 |
| `bunx tsc --noEmit --skipLibCheck` | 成功 |
| `bunx tsc --version` | Version 5.6.2 |

## エラー内容

`bun-types`、Zod、および TypeScript の標準ライブラリの型宣言でエラーが発生した。例として、`Type 'Uint8Array' is not generic` や `esModuleInterop` に関するエラーが出た。

## 原因調査

`--skipLibCheck` を付けた実行は成功した。また、この環境では、プロジェクト直下ではなく親ディレクトリの `node_modules/typescript` が使われていた。プロジェクトの `package.json` に TypeScript の依存宣言がないことだけでは、`bunx tsc` が実行できないとは判断できない。

## 原因

型チェックには、アプリから参照される依存ライブラリの型宣言も含まれる。そのため、エラーの場所が `node_modules` になる。これは `node_modules` 内の全ソースを無条件に検査しているという意味ではない。

型宣言でエラーが発生した個々の根本原因は、この調査では特定していない。

## 解決策

一時的に型宣言ファイルの検査を省く場合は、プロジェクトのディレクトリで次を実行する。

```powershell
bunx tsc --noEmit --skipLibCheck
```

継続的に適用する場合は、`tsconfig.json` の `compilerOptions` に以下を追加する。

```json
"skipLibCheck": true
```

`skipLibCheck` は型宣言ファイル自体の検査を省く。アプリ側で依存ライブラリの型を使用するコードの型チェックは継続される。ただし、型宣言同士の不整合を検出できなくなるため、依存関係の互換性を修復する代わりになるものではない。

今回の調査では `tsconfig.json` を変更していない。上記の設定は適用方法の説明である。

## 参考

- [TypeScript 公式: skipLibCheck](https://www.typescriptlang.org/tsconfig/skipLibCheck.html)

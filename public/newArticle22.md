---
title: Bun で TypeScript を依存に宣言して型チェックする
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

Bun 経由の TypeScript 型チェックでコンパイラを依存として宣言する方法を整理する。パッケージ名の設定についても扱う。

## やりたいこと

ローカルとクリーンな CI で、プロジェクトが指定した TypeScript を使って型チェックする。

## 実行したコード

プロジェクトの `typecheck` は次のコマンドを実行する。

```powershell
bunx tsc --noEmit
```

## 実行結果

以前の調査では、親ディレクトリの TypeScript 5.6.2 を使い、`bunx tsc --noEmit --skipLibCheck` が成功した。クリーンな CI での実行結果は、このメモでは確認していない。

## エラー内容

TypeScript がプロジェクトの依存として宣言されていなかった。クリーンな CI における具体的なエラーメッセージは、このメモには記録していない。

## 原因調査

2026-10-08 の調査では、ローカルで親ディレクトリのコンパイラが使用されていた。一方、TypeScript をプロジェクトの依存に宣言していなければ、クリーンな CI では同じ環境を再現できない。

## 原因

`bunx` はローカルに実行ファイルがなければ指定名のパッケージを取得する。`tsc` という実行ファイルを提供するパッケージは `typescript` であり、未導入時の `bunx tsc` は別の `tsc` パッケージを取得する問題がある。

## 解決策

プロジェクトのディレクトリで TypeScript を開発依存として追加する。

```powershell
bun add --dev typescript
```

`package.json` と `bun.lock` をセットでコミットし、CI はロック済みの依存を導入する。

```yaml
run: bun install --frozen-lockfile
```

TypeScript が導入された状態なら、既存の `bunx tsc --noEmit` を使える。

`--package` は `bunx` で実行パッケージを選ぶオプションであり、`bun install --package` を追加しても TypeScript の依存宣言の代わりにはならない。依存を宣言せず明示的に実行する形なら `bunx --package typescript tsc --noEmit` だが、プロジェクト内でバージョンを固定するため開発依存への追加を推奨する。

### 型宣言の検査

`skipLibCheck: true` は依存先を含む型宣言ファイルの検査を省く。アプリ側の型チェックは継続する。これはコンパイラの導入とは別の設定であり、型宣言同士の不整合を修復するものではない。

以前の調査では `bunx tsc --noEmit --skipLibCheck` は成功し、使用された TypeScript は親ディレクトリの 5.6.2 だった。依存追加後のバージョンや検証結果は確認していない。

### パッケージ名について

アプリの表示名と、`package.json` のパッケージ名は別である。npm の新規パッケージ名は大文字を含められないため、`package.json` では `"name": "my-app"` のように小文字を使う。

非公開アプリなら `"private": true` を付けることで npm への公開を防止できる。名前を変更した場合は `bun install` でロックファイルのメタデータも更新する。

## 参考

- [Bun: bunx と --package](https://bun.sh/docs/pm/bunx)
- [npm: package.json の name](https://docs.npmjs.com/cli/v11/configuring-npm/package-json/#name)
- [TypeScript: skipLibCheck](https://www.typescriptlang.org/tsconfig/skipLibCheck.html)

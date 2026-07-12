---
title: Pillow でフォント読み込み時に「OSError: cannot open resource」が発生した
tags:
  - Python
  - Pillow
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

FastAPI の API を実行し、PNG 画像を返せるようにする。

## 実行したこと

OpenAPI から画像生成 API を実行しました。

## 実行結果

API の実行に失敗し、ステータスコード `500` が返されました。

## エラー内容

```txt
INFO: "POST /images HTTP/1.1" 500
ERROR: Exception in ASGI application

OSError: cannot open resource
```

## 原因調査

エラーが発生している箇所を確認しました。

```python
font = ImageFont.truetype(
    font=font_path,
    size=text_size,
)
```

`ImageFont.truetype()` でフォントファイルを読み込む際に、`cannot open resource` が発生していました。

フォントファイルのパスを確認しました。

## 原因

フォントファイルのパスを正しく解決できず、フォントを読み込めていませんでした。

## 解決策

プロジェクト内のフォントファイルを、Python ファイルの位置を基準に読み込むように変更しました。

```python
from pathlib import Path

PACKAGE_ROOT = Path(__file__).resolve().parent.parent

FONT_PATH = PACKAGE_ROOT / "fonts" / "sample.ttf"
```

実行時のカレントディレクトリではなく、Python ファイルの位置を基準にパスを生成することで、フォントファイルを読み込めるようになりました。

## 学んだこと

ファイルの相対パスは、実行時のカレントディレクトリによって参照先が変わる場合があります。

`Path(__file__).resolve()` を使用すると、Python ファイルの位置を基準にパスを生成できます。

## 参考

https://qiita.com/neko_the_shadow/items/09ff3a423954a2adfe18

https://docs.python.org/ja/3/library/pathlib.html#pathlib.Path.resolve

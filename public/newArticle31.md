---
title: 指定した坂道メンバーのブログを形態素解析してワードクラウド画像を返すFastAPIの話
tags:
  - "Python"
  - "形態素解析"
  - "wordcloud"
private: false
updated_at: ""
id: null
organization_url_name: null
slide: false
ignorePublish: true
posting_campaign_uuid: null
agreed_posting_campaign_term: false
---

# はじめに

Kotogumoというアプリを開発しています。

https://github.com/kishimin/blog-wordcloud

Kotogumoは指定した坂道メンバーのブログを形態素解析し、その形態素解析した結果を元にワードクラウド画像を作成します。その後そのワードクラウド画像を表示し、PNG画像としてダウンロードできるアプリを想定しています。

その中でスクレイピングで取得した坂道メンバーのブログ記事本文とタイトルを`SudachiPy`で形態素解析し、`wordcloud`を用いて単語の頻度を可視化した画像を作成し、`FastAPI`で返すAPIを作成しました。

使用した技術は以下になります。

| 技術              | バージョン  |
| ----------------- | ----------- |
| SudachiPy         | 0.7.0       |
| wordcloud         | 1.9.6       |
| fastapi[standard] | `>=0.142.2` |

`SudachiPy`は以下を参考にしました。

https://github.com/WorksApplications/SudachiPy

また、`wordcloud`は以下を参考にしました。

https://amueller.github.io/word_cloud/generated/wordcloud.WordCloud.html

https://note.com/noa813/n/n777ba133e62f

https://youtu.be/2QhGXFiFYcY?si=S-jUvmMfM0f-570l

スクレイピングは、各坂道グループの公式サイトと以下のサイトを利用しているので、データとしては、公式サイトと以下のサイトから取得できる分だけになります。

https://www.sakamichi.co/

何を含めるかや除外したほうがメンバーが良く使用する単語を特徴として抽出できるようにするために、やったことを記載します。

以下の技術記事で思考の記録を残されているのが興味深いと思ったので記事にしてみました。

https://zenn.dev/hikozuma/articles/b410983bd7aadc

また、反省と改善や次に試したいこともまとめたいと思います。

スクレイピングとブログ記事の内容を取得するためのAPIは別のAPIです。

Kotogumoの`FastAPI`はまず、別APIを叩き、「ブログ記事の内容を取得」します。
その後、Kotogumoの`FastAPI`内で、「形態素解析」、「ワードクラウドの作成」、「PNG画像を返す」ことを行います。

Kotogumoを開発しようと思ったのは、メンバーごとのブログの本文とタイトルで使用頻度の高い単語から、そのメンバーの特徴がある程度出るのではないかと思い、それを考察するのが面白いのではないかと思ったからです。
また、単語とメンバーごとに使用頻度を比較することができたら、面白いのではないかと思ったからです。

実際に、複数のメンバーで見てみると、使用頻度の高い単語が異なり、面白かったです。
また、この単語が使用頻度が高いのかと発見することができて面白かったです。

# やったこと

まずはやったことについてです。

以下のことを行いました。

- 基本的な設定
- 不要な文字列を除外する
- 形態素解析で分離されたくない単語を保護する

それぞれ詳細に見ていきたいと思います。対応した順番どおりに記載していません。ブログ記事の取得期間は記載がないものは、メモしていなかったので不明です。

## 基本的な設定

まずは、基本的な設定についてです。

基本的な設定として、含める品詞を決めること、正規化をやめたこと、100語に制限したこと、カラーマップを決めることについて行いました。

### 含める品詞

「名詞」、「動詞」、「形容詞」、「形状詞」、「副詞」、「感動詞」を含めました。

```py
INCLUDED_PARTS_OF_SPEECH = ("名詞", "動詞", "形容詞", "形状詞", "副詞", "感動詞")
```

immutableとなるようにタプルで定義しました。

感動詞はメンバーごとの感情が出やすそうと思い、追加しました。

### 正規化をやめた

動詞、形容詞を正規化しようと思いましたが、メンバーが書いた言葉どおりに出力したほうが言葉の傾向がわかる、かつ、動詞が漢字になってしまうので、違和感を感じたので正規化をやめました。

違和感は、メンバーが書いた言葉どおりになってないことから起因すると思われます。

以下が蔵盛妃那乃さんのブログ記事をもとに生成したワードクラウド画像での例になります。

![御座るや成る、為るなどの単語があったために正規化をやめた例.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2284906/2461533d-018d-4fe6-80be-a52d6374b3e0.png)

「御座る」、「為る」のような単独で使わないような単語が漢字で表示されて、特徴としてわかりにくいし、違和感がありました。

正規化しようと思ったのは、似たような動詞や形容詞がたくさん出るのを防いだほうが様々な単語を見れそうだと思ったからです。
また、`SudachiPy`で切断された品詞ごとの単語の語尾を整えようと思ったからです。

正規化に関して、`SudachiPy`での例がREADMEに記載されています。

```
# Normalization

tokenizer_obj.tokenize("附属", mode)[0].normalized_form()
# => '付属'
tokenizer_obj.tokenize("SUMMER", mode)[0].normalized_form()
# => 'サマー'
tokenizer_obj.tokenize("シュミレーション", mode)[0].normalized_form()
# => 'シミュレーション'
```

https://github.com/WorksApplications/SudachiPy#usage-as-a-python-package

### 100語に制限

単語数が多く、特徴を判断しにくいと思ったため、100語に制限しました。

参考にしたサイトが100語に制限していたので、まずは100語に制限してみようと思い、制限しました。

https://note.com/noa813/n/n777ba133e62f

制限したほうが大きく表示されるものに目がいきやすく、単語数が少ないとも感じないので、最終的に100語で確定しました。

蔵盛妃那乃さんのブログ記事をもとに生成したワードクラウド画像での変更前の画像と変更後の画像です。

変更前

![100語制限前.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2284906/2a57f2d0-e977-44d6-ab48-22d385e79fca.png)

変更後

![100語制限後.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2284906/657bee22-fa20-47f4-8653-c995c7457716.png)

### カラーマップ

ワードクラウドに使用するカラーマップを以下の公式ドキュメントから調査しました。

https://amueller.github.io/word_cloud/generated/wordcloud.WordCloud.html

> colormap : string or matplotlib colormap, default=”viridis”
> Matplotlib colormap to randomly draw colors from for each word. Ignored if “color_func” is specified.

`Matplotlib colormap`で指定できるとのことで、以下のサイトを調べました。

https://matplotlib.org/stable/users/explain/colors/colormaps.html

様々な種類がありましたが、その中で、「Cool」を採用しました。

根拠としては、櫻坂46に『Cool』という楽曲があり、同じ名前のものがあるならということで採用しました。

https://youtu.be/XEKPn3WbksE?si=gK_irKbmUWZFFhP6

## 不要な文字列を除外する

次は不要な文字列を除外したことについてです。

不要な文字列を除外することとして、HTMLタグの除外、ひらがな1文字の除外、`amp;`の除外、リンクの除外についてを取り上げます。

### HTMLタグの除外

別APIは、HTMLタグを含んだままのブログ本文で返していました。しかし、形態素解析するには不要なため、HTMLタグを除去する処理を加えました。

別APIの方を変更しなかったのは、別APIとしては、ブログ本文を返すことが責務だと思ったからです。

HTMLタグがいらないのはKotogumoの`FastAPI`側なので、別APIを叩くKotogumoの`FastAPI`側でHTMLタグの除外を行いました。

以下を参考にしました。

https://mebee.info/2022/09/09/post-78457/

HTMLタグの正規表現にマッチしたものを空文字に変換しました。

```py
import re


def remove_html_tag(text: str):
    return re.sub(re.compile("<.*?>"), "", text)
```

### ひらがな1文字の除外

ひらがな1文字は他の文字と結合して意味をなすものだと思い、除外しました。

以下の蔵盛妃那乃さんのブログ記事をもとに生成したワードクラウド画像では、「い」が大きく表示されています。

![いが大きく表示されている画像.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2284906/36dfe180-33e5-4548-aa0b-a011cf44ee61.png)

形態素解析により、文字が分割されたことで生じていると思いました。
なので、形態素解析後の結果でひらがな1文字の正規表現にマッチしたものを形態素解析の結果から含めないようにしました。

```py
single_hiragana_pattern = re.compile("^[\u3040-\u309F]$")
word_cloud_terms = [
    term for term in word_cloud_terms if not single_hiragana_pattern.match(term)
]
```

正規表現は以下を参考にしました。

https://www.javadrive.jp/regex-basic/sample/index8.html

### `amp;`の除外

「ミート&グリート」のまま、ワードクラウドで単語が表示されるようにしたいと思ったときに調べたところ、必要になったことです。

まず、CodexにGit管理しない使い捨てのスクリプトを作成させました。そのスクリプトを用いて、別APIを叩いたときのブログ記事の内容を見てみました。

その時、「ミート&グリート」に注目したところ、「`ミート&amp;グリート`」となっていました。

以下が画像の例になります。

![amp;が混入している根拠画像.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2284906/f7bfc10d-cc85-443d-983a-7f325b75ec94.png)

`&`はHTMLコードで意味を持つ文字を別の文字にするときに、開始を示す記号のため、`&`は特殊文字として`&amp;`という別の文字で記述する必要があるようです。

以下が参考になります。

https://zenn.dev/kagan/articles/4038e7a9a5ff5e

https://qiita.com/coding1/items/263aec24eb998ea704e2

https://www.cis.twcu.ac.jp/~asakawa/comp2d-2008/special_chars.html

しかし、`amp;`は形態素解析をするには不要な文字列なので、解析前に除外しました。

こちらも、HTMLタグを除外したのと同様に、Kotogumoの`FastAPI`側が除外する責務だと思いました。

除外した方法としては、`amp;`を空文字に置き換えました。

```py
analysis_text = analysis_text.replace("amp;", "")
```

除外後は以下のように解析、表示されました。

```txt
00482	surface='ミート'	part_of_speech=('名詞', '普通名詞', '一般', '*', '*', '*')
00483	surface='＆'	part_of_speech=('補助記号', '一般', '*', '*', '*', '*')
00484	surface='グリート'	part_of_speech=('名詞', '普通名詞', '一般', '*', '*', '*')
```

```txt
ミート&グリート
```

### リンクの除外

httpsなどのリンクとして意味のない文字列が混入するのを防ぐために行いました。
一律除外だと、すぐに実装できそうだと思ったので行いました。

以下の蔵盛妃那乃さんのブログ記事をもとに生成したワードクラウド画像では、「https youtu」が含まれています。

![ワードクラウドにリンクの一部が含まれている画像.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2284906/f09cdd20-f3de-4f38-b725-d56155c519fb.png)

除外した方法として、URLの正規表現でマッチした文字列を空文字に置換しました。

```py
# Do not keep URLs: tokenization splits them into unrelated word cloud terms.
url_pattern = re.compile(
    r"https?:\/\/(?:www\.)?[a-zA-Z0-9:?#/@\-._~%!$&'()*+,;=]{1,256}\.[a-zA-Z0-9()]{1,6}\b(?:[a-zA-Z0-9:?#/@\-._~%!$&'()*+,;=]*)"
)
analysis_text = re.sub(url_pattern, "", analysis_text)
```

以下を参考にしました。

https://pote-chil.com/posts/javascript-url-regex

https://qiita.com/str32/items/a692073af32757618042

## 形態素解析で分離されたくない単語を保護する

最後に、分割して表示されたくない言葉がそのままの形で保持されるようにしました。

まず、保護する単語のリストを用意し、そこに該当の単語を一つずつ格納しました。その後、形態素解析から外し、最後に、ワードクラウド用の単語のリストに格納するという仕組みを取りました。

以下が擬似的なコード例になります。

```py
objects_response = requests.get(url=f"{EXAMPLE_COLLECTOR_URL}/objects")
object_names = []
for objects in objects_response.json().values():
    for object in objects:
        object_name = object["name"]
        object_names.append(object_name)

analysis_text = ""

protected_names = []
for protected_name in object_names:
    if not protected_name:
        continue
    protected_names.extend(re.findall(re.escape(protected_name), analysis_text))
    analysis_text = analysis_text.replace(protected_name, "")

word_cloud_terms = []

"""
ここで形態素解析して、word_cloud_termsに解析結果を格納(省略)
"""

word_cloud_terms.extend(protected_names)
word_cloud_text = " ".join(word_cloud_terms)
```

### グループ名

以下の渡辺莉奈さんのブログ記事をもとに生成したワードクラウド画像では、「日向」と「坂」が同じくらいの大きさで別々に表示されています。

![日向と坂が分離された画像.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2284906/207bbbd4-9bbe-4e7f-b131-1678bbb072f5.png)

形態素解析で、分離した言葉として解釈されていると思い、グループ名を保護することにしました。

### メンバー名

上村ひなのさんが「上村ひな」まででしかなく、「の」が含まれていませんでした。
形態素解析で、以下の上村ひなのさんのブログ記事をもとに生成したワードクラウド画像のように、「上村ひな(名詞)」と「の(助詞)」と判断されて、助詞を含めていないので、「上村ひな」までしか表示されなかったのだろうと思いました。

![メンバー名の保護前例1の画像.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2284906/f9556056-4d99-4ba5-8996-70e1ea3ac125.png)

また、以下の蔵盛妃那乃さんのブログ記事をもとに生成したワードクラウド画像のように、蔵盛妃那乃さんが、「蔵盛妃」や「那乃」とかに分離されていると思いました。

![メンバー名保護前例2の画像.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2284906/27180bb7-3b87-4012-a7d5-11da18b3eb0e.png)

このように名前が形態素解析により、分離されたと思いました。そこで、メンバー名のフルネームを保護することにしました。

この対応を行った後、以下の蔵盛妃那乃さんのブログ記事をもとに生成したワードクラウド画像のように、メンバー名が2回繰り返し表示されるという問題点が生じました。

![メンバー名が2回繰り返し表示される画像.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2284906/98c85e66-4b61-4baf-8ee7-ced207444983.png)

メンバー名を保護する単語に加えて、形態素解析から外し、その後、ワードクラウド用の単語のリストに格納するという仕組みを取っていました。参考にした以下の動画と同様に単語のリストを`join`を用いて空白をいれながら、1つの文字列にしています。

https://youtu.be/2QhGXFiFYcY?si=S-jUvmMfM0f-570l

wordcloudの公式ドキュメントを調べてみました。

https://amueller.github.io/word_cloud/generated/wordcloud.WordCloud.html

すると、以下の記載が見つかりました。

> collocations : bool, default=True
> Whether to include collocations (bigrams) of two words. Ignored if using generate_from_frequencies.

デフォルトの挙動で単語が続いたときに、一つの単語として見てしまうようです。メンバー名がリスト内に集中する区間が存在し、メンバー名 + 空白 + メンバー名を一つの単語としてワードクラウド側が判断してしまったことが原因でした。

そこで、`collocations=False`にしました。

最終結果として以下の蔵盛妃那乃さんのブログ記事をもとに生成したワードクラウド画像のようになりました。

![メンバー名保護成功後の画像.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2284906/b355d9ff-31af-44e8-861d-a5c9695a71ff.png)

### 種類別のミート&グリートの言葉

「ミート&グリート」のまま、ワードクラウドで単語が表示されるようにしたいと思いました。そのまま、形態素解析すると、「ミート」、「&」、「グリート」に分割され、かつ、「&」が補助記号で品詞の対象外となっているのでワードクラウドには含まれないようになってしまいます。

また、「オンラインミート&グリート」、「リアルミート&グリート」、「ミート&グリート」を区別するため、「オンラインミート&グリート」、「リアルミート&グリート」、「ミート&グリート」の順で保護する単語のリストに加えました。「ミート&グリート」を他の2つよりもあとにしないと、他の2つを保護対象の単語のリストに追加する前で、「ミート&グリート」を追加してしてしまい、「オンラインミート&グリート」、「リアルミート&グリート」が保護対象の単語のリストに追加されなくなるからです。

# 反省と改善

形態素解析の結果を分析せずに、推測して画像の出力結果から推測で判断していたのが良くなかったと思いました。

グループ名やメンバー名以外で、単語が分離されて同じ大きさに表示されているのかなと思い、それを保護対象の単語のリストに追加しました。しかし、単語の分離が解消されませんでした。

グループ名やメンバー名の例は形態素解析の結果を見ずに成功しましたが、形態素解析の結果を分析したほうが根拠になると思いました。グループ名やメンバー名の判断に成功した原因は、複数のメンバーで同様の現象が見られたことで成功したと思います。

出力で見るよりも、形態素解析の出力を見て、保護する単語や除外する単語を決められるといいと思いました。

そこで、改善として、Codexを用いて、Git管理しない使い捨てのスクリプトを作らてみました。解析前のテキスト、解析後のテキスト、ワードクラウドにいれるテキストをtextファイルに出力するものです。

上記のスクリプトを用いて、見つかった分離された単語はありませんでしたが、「ミート&グリート」の例に代表されるように、原因を分析するのに役に立ったので、解析したテキストの出力を見るのはいいと思いました。

# 次に試したいこと

記号、絵文字を含めてメンバーごとの特徴が抽出されるようにしたいと思いました。

また、`SudachiPy`にはユーザー辞書というものがあるようです。これを利用して、デフォルトの形態素解析を行ったときには、分離されてしまう単語を追加したいと思いました。

https://github.com/WorksApplications/SudachiPy/blob/develop/docs/tutorial.md#%E3%83%A6%E3%83%BC%E3%82%B6%E3%83%BC%E8%BE%9E%E6%9B%B8

グループ名やメンバー名を取得するときに、別のAPIを叩いています。叩くこともなくなるので、不要な処理を減らすことが見込めるし、コードも少なくすることができるので、試してみたいと思いました。

また、1名しか確認が取れていませんが、佐藤優羽さんの下の名前が「優」と「羽」で形態素解析の結果で、分離されているのを見つけました。このように、下の名前や名字しかかかれていないときにも、分離がされないように、こちらも`SudachiPy`のユーザー辞書に追加したいです。

さらに、各メンバーごとに投稿された期間を指定できるようにしたいです。これは、期間ごとのメンバーのブログの特徴を見ることができるので面白いのではと思っています。

また、技術的には、テストを書けていないので、テストを書けそうなところは、テストを書きたいです。テスト駆動開発ができそうなところでは、テスト駆動開発をしたいです。コードの保守性やテスタビリティを意識してないので、コードを整えたいです。

# 最後に

Kotogumoを完成させたいです。

以下に現時点の完成後の蔵盛妃那乃さんのブログ記事をもとに生成したワードクラウド画像を掲載します。ブログ記事の取得期間は不明です。

![現時点のワードクラウド画像.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2284906/9628758f-df7c-4e2a-a5bb-6b93cd7e2730.png)

# 補足

## `SudachiPy`

`SudachiPy`のREADMEに以下のように記載されています。

> SudachiPyは日本語形態素解析器Sudachiのpython版です。

https://github.com/WorksApplications/SudachiPy/blob/develop/docs/tutorial.md

さらに、`Sudachi`のREADMEには以下のように記載されています。

> Sudachi は日本語形態素解析器です。形態素解析はおもに以下の3つの処理を おこないます。
>
> - テキスト分割
> - 品詞付与
> - 正規化処理

Pythonでの日本語の形態素解析器です。

## 形態素解析

ウィキペディアに以下のように記載されています。

> 形態素解析（けいたいそかいせき、（英: morphological analysis）は自然言語の文字列を意味に基づく最小単位へ分割しその品詞を特定する処理である。

> 形態素解析とは、対象言語の文法や単語の品詞等の情報にもとづき、文法的な情報の注記の無い自然言語のテキストデータ（文）を単語の列に分割し、各単語の品詞や活用などを判別することで形態素（おおまかにいえば、言語で意味を持つ最小単位）の列を得る作業である。

https://ja.wikipedia.org/wiki/%E5%BD%A2%E6%85%8B%E7%B4%A0%E8%A7%A3%E6%9E%90

形態素解析は、テキストデータをある言語の文法、品詞に基づいて分割し、各分割後の単語の情報を得ることです。

## 形状詞

国立国語研究所によると、以下のように記載されています。

> 「形状詞」は形容動詞語幹相当の品詞です。

https://clrd.ninjal.ac.jp/unidic/glossary.html

形状詞は、形容動詞の「～だ」みたいなのがないものです。

## ワードクラウド

文部科学省が出している資料に以下の記載がありました。

> ワードクラウドは、指定した期間における単語の出現回数の多さを、単語の文字列の大きさで示したものである。
> 当該分析手法は、対象となる文章に含まれる各単語の出現回数を計測し、単語の文字列の大きさを用いて、直感的に文章における特徴的な単語を把握するために活用する手法である。

https://www.mext.go.jp/content/20250722-mxt_chousei01-100000404_1.pdf

ワードクラウドは、単語の出現回数の多さを、単語の文字列の大きさで示したものです。

## wordcloud

`word_cloud.Wordcloud`についてドキュメントに以下のように記載されています。

> Word cloud object for generating and drawing.

[DeepL](https://www.deepl.com/ja/translator)で翻訳しました。

> ワードクラウドオブジェクト（生成・描画用）。

https://amueller.github.io/word_cloud/generated/wordcloud.WordCloud.html

wordcloudは、ワードクラウドを作成するためのライブラリです。

## 文字参照

MDNリファレンスに以下のように記載されています。

> HTML の文字参照 (character reference) とは、レンダリングされたウェブページ上で別の文字を表すために使用されている、文字のエスケープシーケンスのことです。

https://developer.mozilla.org/ja/docs/Glossary/Character_reference

文字参照別は、別の文字を表すために使用される文字の仕組みです。

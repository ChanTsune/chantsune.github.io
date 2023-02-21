---
title: "[Django] 記事の一覧ページと詳細ページの追加"
date: 2018-11-17T16:05:35+00:00
lastmod: 2018-11-25T07:27:56+00:00
draft: false
categories: []
tags: ["django","Djangoでブログを作ろう"]
# weight: 6331
---
前回、記事を投稿できるようになったので、今回は投稿した記事を見られるようにしたいと思います。  

### シリーズ一覧  
> 1. [プロジェクトの作成とアプリケーションの作成](/articles/68/)  
> 1. [view関数の書き方](/articles/69/)  
> 1. [URLの指定の仕方(URLディスパッチャ)](/articles/70/)  
> 1. [データベース(モデル)の設定](/articles/71/)  
> 1. [記事の投稿ページの作成](/articles/72/)
> 1. [記事の一覧ページと詳細ページの追加](/articles/73/) <- 今回  
> 1. [記事の編集ページと記事の削除機能の追加](/articles/74/)  
> 1. [いいね機能の追加](/articles/75/)
> 1. [コメント機能の追加](/articles/77/)
> 1. [Bootstrapを利用したwebデザイン](/articles/78/)


## はじめに
前回までに記事の投稿ができるようになっていることを想定しているので、まだできていない人は出来るようにしてからご覧ください。  


## Djangoテンプレート言語
### 変数  
Djangoでは変数という形で、pythonプログラムの中のデータをテンプレート(HTMLファイル)に渡すことができます。  

テンプレートに変数を渡すにはviews.pyの中でrender関数の第三引数に辞書を渡すことで実現できます。  
具体的には以下のように書きます。  
```py
def sample_view(request):
  context = {"text":"hello!"}
  return render(request, template_name, context)
```

変数を受け取ったテンプレート(HTMLファイル)側は  
```django
{{ text }}
```
とすることで受け取った変数の中身を表示できます。  

ちょうど辞書の中のキーがテンプレート側での変数名になります。  

上の例だと`{{ text }}`は`hello!`に置き換えられます。  


### テンプレートタグ  
テンプレートタグと呼ばれる特別な機能を持ったタグの紹介です。  
テンプレートタグは両端を`{%`と`%}`で囲みます。  

#### 繰り返し
テンプレート(HTMLファイル)内に  
```html
{% for v in values %}
  <p>{{ v }}</p>
{% endfor %}
```
というように書くとpythonの`for文`と同じように繰り返しをさせることができます。  
`values`の中身が次々と`v`に代入されて繰り返します。  

上の例だと`values`の中身を一つずつ pタグ でくくって表示させています。  

`values`は変数名でviewから渡した変数なら何でも構いません。  
ただし、変数の中身はpythonのforにも使えるイテレータブルなオブジェクトに限ります。  

注意点は繰り返しの終了地点に`{% endfor %}`を書いてあげることです。  

もう少し詳しく知りたい場合は  
> [これだけは知っておきたいDjangoテンプレートの基本文法](/articles/67/)  


とか  

> [[Django]知ってると便利なforで使える小技](/articles/20/)  

に詳しくまとめてます。  

#### リンクの自動生成
```django
{% url 'index' %}
```
のようにすると自動的にURLの文字列を生成してくれます。  

シングルクォートで囲ってある部分には`urls.py`で指定した**pathのname**を入れます。  
```py
urlpatterns = [
    path("", views.index, name="index"), 
    path("new/", views.new, name="new"),
    # path関数で指定したnameの部分
    # この例だと "index"と"new"
]
```

実際に利用するときは以下のように使うことが多いと思います。  
```html
<a href="{% url 'index' %}">トップページへ</a>
```

リンクが深くなると自分でURL書くのが面倒になるので便利な機能です。  


## 一覧ページの追加
先ほどのテンプレート言語を踏まえて、投稿した記事の一覧が見れるページを作成します。  
### URLの設定
`http://localhost:8000/article/all/`が一覧ページになるようにURLを設定します。  

`myblog/blog/urls.py`
```py
from django.urls import path
from . import views

urlpatterns = [
    path("", views.index, name="index"),
    path("new/", views.new, name="new"),
    path("article/all/", views.article_all, name="article_all"), # 追記
]
```

### viewの設定
`views.py`
```py
def article_all(request):
    template_name = "blog/article_all.html"
    context = {"articles": models.Article.objects.all()}
    return render(request, template_name, context)
```

`models.Article.objects.all()`でデータベースの`Article`のテーブルにあるすべてのデータを取得しています。  
取得したデータに`articles`という変数名を付けてテンプレート側に渡しています。  

`models.Article.objects.all()`で取得したデータはリスト同様、`for文`で回すことが可能です。  

### テンプレートの記述
新しくテンプレートファイル`templates/blog/article_all.html`(blogはアプリケーション名)を作りましょう。  
中身は以下のようにします。  
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <title>article all</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
</head>
<body>
    <h3>投稿された記事一覧</h3>
    {% for article in articles %}
        <p>{{ article.title }} - {{ article.posted_at }}</p>
    {% endfor %}
</body>
</html>
```

```html
{% for article in articles %}
    <p>{{ article.title }} - {{ article.posted_at }}</p>
{% endfor %}
```
`articles`が先ほどテンプレートに渡した変数です。  
中身を`for文`で回しています。  
`{{ article.title }}`で記事のタイトルを、`{{ article.posted_at }}`で記事の投稿時間をそれぞれ展開しています。  

では、表示させてみましょう。  
```
python manage.py runserver
```
でサーバーを起動した後に`http://localhost:8000/article/all/`にアクセス  

![記事の一覧ページ](./list_page.png)

たぶんこんな感じになるはずです。  



## 詳細ページの追加
それぞれの記事の詳細ページを追加します。  
### URL 
`http://localhost:8000/article/数字/`がそれぞれのページのURLになるように設定します。  

と、ここで疑問です。  
今までは、それぞれ個別のページに固有のURLを設定しました。  

では、`http://localhost:8000/article/数字`のように特定の規則を持ったURLを指定するにはどうすればよいのでしょうか？  

答えは、正規表現を用いることです。  

数字にマッチする正規表現はpythonでは`r"\d+"`又は、`r"[0-9]+"`と表すことができます。  

それを適用すると  
```py
re_path(r"article/(\d+)/", views.article, name="article")
```
とすることができます。  

が、  

ちょっと見づらいですよね。  

そこで、Django2系から使えるようになった機能、`パスコンバータ`を利用します。  
簡単に紹介すると正規表現を簡単に表現できるようにしたものです。  

パスコンバータを利用して書くと以下のように書くことができます。  
```py
path("article/<int>/", views.article, name="article")
```

`<int>`で数字を受け取ることを表現できます。  
受け取る値が数字であることがものすごくわかり易くなりました。  

パスコンバータでは数字意外にも任意の文字列を受け取れる`<str>`や数字とアルファベットの組み合わせを受け取れる`<slug>`等があります。  

詳しくは
> [[Django] 2系から導入されたパスコンバータとは](/articles/76/)  


`urls.py`にはパスコンバータを利用したほうを書いておきましょう。  
```py
from django.urls import path
from . import views

urlpatterns = [
    path("", views.index, name="index"),
    path("new/", views.new, name="new"),
    path("article/all/", views.article_all, name="article_all"),
    path("article/<int:pk>/", views.view_article, name="view_article"), # 追記
]
```


### view
`views.py`に`view_article`を書き足します。  
```py
from django.shortcuts import render
from django.http import Http404 # 追記
from . import models

# 略

def view_article(request, pk):
    template_name = "blog/view_article.html"
    try:
        article = models.Article.objects.get(pk=pk)
    except models.Article.DoesNotExist:
        raise Http404
    context = {"article": article}
    return render(request, template_name, context)
```

今までと違い二つ引数を受け取るようになっていますね。  
第一引数は今までと変わりありませんが、第二引数で新たにpkを受け取るようになっています。  
このpkには先ほど設定したURLの`<int:pk>`にマッチした数字が入ります。  
例えば`http://localhost:8000/articles/1/`にアクセスするとpkには`1`が入ります。  

ちなみに`<int:pk>`のpkと引数で受け取るpkは対応しているので名前を揃えておきましょう。  
違う名前にしているとエラーを吐かれる可能性があります。  

関数の中では、ここで受け取った数字を利用してデーターベースから該当するデータを取り出しています。  
`models.Article.objects.get(pk=pk)`
の部分がそれにあたります。  
`get(pk=pk)`でデータベースの主ーキーを利用して当該の記事のデータを取得しています。  

`except models.Article.DoesNotExist:`で記事が見つからなかったときに、ページがない`404エラー`を出すようにしています。  


### テンプレート
新しくテンプレートファイル`templates/blog/view_article.html`(blogはアプリケーション名)を作成します。  

中身はこんな感じで  
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <title>{{ article.title }}</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
</head>
<body>
    <h3>{{ article.title }}</h3>
    <small>投稿日時 : {{ article.posted_at }}</small>
    <small>最終更新 : {{ article.last_modify }}</small>
    <p>{{ article.text }}</p>
</body>
</html>
```


### リンクの設定
詳細ページを追加したので一覧ページから詳細ページへ遷移できるリンクを設置しましょう。  
利便性向上のためにトップページに戻れるリンクも設置します。  

`article_all.html`  
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <title>article all</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
</head>
<body>
    <a href="{% url 'index' %}">トップページへ</a>
    <h3>投稿された記事一覧</h3>
    {% for article in articles %}
    <p><a href="{% url 'view_article' article.pk %}">{{ article.title }} - {{ article.posted_at }}</a></p>
    {% endfor %}
</body>
</html>
```

```html
<a href="{% url 'view_article' article.pk %}">
```

またしても、見慣れない書き方が出てきました。  

`{% url 'view_article' %}`ならまだわかります。  
ですがそれに加えて`article.pk`なるものがくっついています。  

規則を持ったURLを設定した(正規表現、又はパスコンバータを利用した)場合、urlテンプレートタグに追加で規則にマッチする引数を与えてあげる必要があります。  

詳細ページには
```py
path("article/<int:pk>/", views.view_article, name="view_article")
```
のようにパスコンバータを利用して規則を持ったURLを設定しているので、引数を与えてやる必要があります。  

intを使っているので数字の引数です。  


続いて、トップページに「新規記事の投稿ページ」へのリンクと「一覧ページ」へのリンクを設定しましょう。  
こちらにもトップページに戻れるリンクを設置しておきます。  
`index.html`  
```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8" />
    <title>my-blog-index</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
</head>

<body>
    <p>まい ぶろぐ とっぷぺーじ</p>
    <a href="{% url 'new' %}">新規記事の投稿</a>
    <a href="{% url 'article_all' %}">投稿された記事一覧</a>
</body>

</html>
```

残りのページにも適宜リンクを設置していきます。  

`new.html`
```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8" />
    <title>my-blog-new</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
</head>

<body>
    <a href="{% url 'index' %}">トップページへ</a>
    
    <h3>まい ぶろぐ 新規記事の投稿ページ</h3>
    <form action="" method="post">
        {% csrf_token %}
        <p><label for="title_input">記事のタイトル</label></p>
        <p><input type="text" name="title" id="title_input"></p>
        <p><label for="text_area">記事の本文</label></p>
        <p><textarea name="text" id="text_area" cols="30" rows="10"></textarea></p>
        <p><button type="submit">投稿</button></p>
    </form>
</body>

</html>
```

`view_article.html`
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8" />
  <title>{{ article.title }}</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
</head>
<body>
  <p><a href="{% url 'index' %}">トップページへ</a></p>
  <p><a href="{% url 'article_all' %}">一覧へ戻る</a></p>
  <h3>{{ article.title }}</h3>
  <small>投稿日時 : {{ article.posted_at }}</small>
  <small>最終更新 : {{ article.last_modify }}</small>
  <p>{{ article.text }}</p>
</body>
</html>
```

リンクを設置したのでちゃんと移動できるかどうか確認してみましょう。  

Djangoサーバを起動して
```
python manage.py runserver
```

適当なページへアクセス  
http://localhost:8000/

ページ内にあるリンクを踏みまくって遊んでみましょう。  

新しく記事を投稿して一覧ページに要素が増えるかもチェックしておきましょう。  

飽きたらやめてもらって大丈夫です。  


## まとめ
データーベースからのデータの全件取得は  
```py
models.Model.objects.all()
```

データベースからの条件に一致するデータ一件の取り出しは  
```py
models.Model.objects.get(フィールド名=値)
```

テンプレートに変数を渡すにはrender関数の第三引数に辞書の形にして渡す(キーがテンプレート内での変数名)。  

`{% for v in values %}`でテンプレート内で繰り返しができる(終了点には`{% endfor %}`をお忘れなく)。

テンプレート内での変数の展開は`{{ 変数名 }}`。

`{% url 'path関数で指定したname' %}`でURLを自動生成できる。  

## 最後に
今回はちょっと長めでしたが、基本的にはこれでブログとして機能しうるものは出来ました。  
ただ、このままだと寂しいので次回は、投稿した記事を編集できる機能と削除機能を追加しようと思います。  

デザインはやらへんのかいっ！  

...最終回あたりにBootstrap使ってみるに堪えるようなものにはします。  
お楽しみに  

次回  
> [[Django] 記事の編集ページと記事の削除機能の追加](/articles/74/)  

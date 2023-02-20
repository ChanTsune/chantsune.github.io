---
title: "[Django] プロジェクトの作成とアプリケーションの作成"
date: 2018-11-09T09:30:03+00:00
lastmod: 2018-11-25T07:20:48+00:00
draft: false
categories: []
tags: ["django","Djangoでブログを作ろう"]
# weight: 5972
---
Djangoで新しくプロジェクトを作るときに必要な操作をまとめました。  
  
数回に分けて実際に動くブログを作る事を目標にします。  

### シリーズ一覧  
> 1. [プロジェクトの作成とアプリケーションの作成](/articles/68/) <- 今回  
> 1. [view関数の書き方](/articles/69/)  
> 1. [URLの指定の仕方(URLディスパッチャ)](/articles/70/)  
> 1. [データベース(モデル)の設定](/articles/71/)  
> 1. [記事の投稿ページの作成](/articles/72/)  
> 1. [記事の一覧ページと詳細ページの追加](/articles/73/)  
> 1. [記事の編集ページと記事の削除機能の追加](/articles/74/)  
> 1. [いいね機能の追加](/articles/75/)
> 1. [コメント機能の追加](/articles/77/)
> 1. [Bootstrapを利用したwebデザイン](/articles/78/)


## はじめに
※このメモはpython3,Django2系がインストール済みなことを想定しています。  
インストールしていない場合は各自でインストールして下さい。  
pythonがインストールしてある場合、Djangoのインストールは
```
pip install django
```
でできます。  

## プロジェクトの作成
まずは新しくDjangoのプロジェクトを作成します。  
プロジェクトを作成したいディレクトリで以下のコマンドを実行します。  
```
django-admin.py startproject プロジェクト名
```
プロジェクト名のところはご自由にどうぞ、今回は`myblog`という名前にしようと思います。  

実行が完了すると以下のようなファイル群が出来上がります。  
```
./myblog
├── manage.py
└── myblog
    ├── __init__.py
    ├── settings.py
    ├── urls.py
    └── wsgi.py
```

続いてアプリケーションの作成です。  

## アプリケーションの作成
先程作成したプロジェクトの名前のフォルダ(myblog)に移動します。  
```
cd myblog
```
移動したら移動したディレクトリにmanage.pyがある事を確認してください。  
確認できたら以下のコマンドを実行してください。  

```
python manage.py startapp アプリ名
```
アプリ名は自由な名前に置き換えてください。  
今回は`blog`とします。  
コマンドが実行されると以下のファイル群が出来上がります。  
```
./blog
├── admin.py
├── apps.py
├── __init__.py
├── migrations
│   └── __init__.py
├── models.py
├── tests.py
└── views.py
```

プロジェクト全体では以下のファイル構成になっているはずです。  

```
myblog
├── blog
│   ├── admin.py
│   ├── apps.py
│   ├── __init__.py
│   ├── migrations
│   │   └── __init__.py
│   ├── models.py
│   ├── tests.py
│   └── views.py
├── manage.py
└── myblog
    ├── __init__.py
    ├── settings.py
    ├── urls.py
    └── wsgi.py
```

## アプリケーションの登録
先程アプリケーションを作成しましたが、これだけでは**Djangoのプロジェクトにアプリケーションが認識されていません。**  
なのでアプリケーションを登録してあげる必要があります。  
具体的には`myblog/myblog/setting.py`の`INSTALLED_APPS`に以下のように追記しましょう。  
```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'blog', # ここに作成したアプリケーションの名前を追記
    # 今回の場合はblog
]
```

何度も言いますがこの操作はプロジェクトにアプリケーションを登録するために必要です。  
**これを忘れると今後の作業に影響が出るので忘れずにやりましょう！**  

Djangoを使い始めた頃はよくここで引っかかったものです。  

最後にコンソールで
```
python manage.py runserver
```
を実行してエラーがでないことを確認してブラウザで  

http://localhost:8000

にアクセスしてみましょう。  

![Djangoのwelcomeページ](./welcom_page.png) 

上記のようページが表示されれば今回はOKです。  


## まとめ
プロジェクトの作成は
```
django-admin.py startproject プロジェクト名
```

アプリケーションの作成は
```
python manage.py startapp アプリ名
```

新しくアプリケーションを作成したら`setting.py`の`INSTALLED_APPS`に忘れずにアプリケーションを登録すること。



次回はview関数(コントローラー)の設定の仕方を解説しようと思います。  
> [[Django] view関数の書き方](/articles/69/)  

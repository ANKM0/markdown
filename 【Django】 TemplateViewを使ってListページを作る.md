---
title: 【Django】 TemplateViewを使ってListページを作る
topics: ["GitHubActions", "Ruby", "YAML"]
published: true
---

Templateviewを使ってListページを作ろうとしたときに
良い感じの記事と巡り会えず、思ったよりハマったのでTemplateviewを使ってListページを作る方法をまとめました

# 概要
TemplateViewを使ってListページを作成する方法です


# この記事で伝えたいこと
- Templateviewを使ったListページの作り方

- context_dataにデータを詰めれば上手くいく

# 結論

`[モデル].objects.all()`で取得したデータをcontext_dataに代入する

```app1_views.py
from django.views.generic import TemplateView

from .models import TestData


class IndexView(TemplateView):
    template_name: str = "app1/index.html"

    def get_context_data(self, **kwargs):
        ctx = super().get_context_data(**kwargs)
        ctx['object_list'] = TestData.objects.all()
        return ctx
```

# ソースコード
ソースコードをGitHubに公開しています <br>
https://github.com/ANKM0/django_sample_make_list_with_templateview.git


# 環境
- django4.1.3

## フォルダ構成

フォルダ構成はこんな感じになっています <br>
プロジェクト名はconfig,アプリ名はapp1です

<pre>
DJANGO_SAMPLE(root)
│  db.sqlite3
│  gen_secrets.py
│  manage.py
│  secrets.json
│
├─app1
│  │  admin.py
│  │  apps.py
│  │  forms.py
│  │  models.py
│  │  tests.py
│  │  urls.py
│  │  views.py
│  │  __init__.py
│  │
│  ├─migrations
│  │  │  0001_initial.py
│  │  │  __init__.py
│  │  │
│  │  └─__pycache__
│  │          0001_initial.cpython-310.pyc
│  │          __init__.cpython-310.pyc
│  │
│  └─__pycache__
│          略
│
├─config
│  │  asgi.py
│  │  settings.py
│  │  urls.py
│  │  wsgi.py
│  │  __init__.py
│  │
│  └─__pycache__
│          略
│
└─templates
    └─app1
            base.html
            index.html
</pre>









# 下準備

## リストで表示するためのデータを登録
Listページの作り方を説明する前にリストで表示するためのデータを登録します

number, name, priceカラムを持つTestDataモデルを作成します
```app1_models.py
from django.db import models


class TestData(models.Model):
    number = models.PositiveIntegerField()
    name = models.CharField(max_length=200, blank=False, null=False)
    price = models.PositiveIntegerField()

    def __str__(self):
        return self.name

    class Meta:
        verbose_name_plural = "テストデータ"
```

<br>

3件のデータを登録しました

![Alt text](https://i.gyazo.com/022a7a00a68928454b94df6f9efa5c99.png)


# 本題

普段,リストを作成する場合はListViewを使っていると思います

```app1_views.py
from django.views.generic import ListView
from .models import TestData


class IndexView(ListView):
    template_name: str = "app1/index.html"
    model = TestData

```

<br>

このとき,ListViewは
1. モデルからデータを取得

1. templateにデータをobject_listという名前で渡す

という処理を行っています

<br>

これらの処理をTemplateViewに追加していきます

<br>

## 方法その1 get関数を使う

ListViewと同じ処理をする方法その１はget関数をオーバーライドする方法です <br>

<br>

コードで書くとこんな感じになります

```app1_views.py
from django.views.generic import TemplateView
from django.shortcuts import redirect, render

from .models import TestData


class IndexView(TemplateView):
    template_name: str = "app1/index.html"
    object_list = TestData.objects.all()

    def get(self, request, *args, **kwargs):
        object_list = self.object_list
        context = {
            "object_list": object_list,
        }
        return render(request, self.template_name, context)

```

<br>

`object_list = TestData.objects.all()` でモデルからデータを取得しています<br>

contextにデータを渡して
```
context = {
    "object_list": object_list,
}
return render(request, self.template_name, context)
```

`return render(request, self.template_name, context)` でデータをテンプレートに渡しています


<br>

データを渡すテンプレートはこのようになります

```index.html
{% extends "app1/base.html" %}
{% block title %}index{% endblock %}

{% block content %}
<div class="container">
    <h1>index page</h1>

    {{ object_list }}  <!-- querysetになので取り出す必要があります -->

    {% for item in object_list %}
    <li>{{ item.number }}</li>
    <li>{{ item.name }}</li>
    <li>{{ item.price }}</li>
    <br>
    {% endfor %}

</div>
{% endblock %}
```


サーバーではこのように表示されます

![Alt text](https://i.gyazo.com/3958bdc7b2a97e69724640993a6bdbce.png)

<br>

## 方法その2 get_context_dataを使う


get関数をオーバーライドする場合、処理を都度書く必要があるのでコードが汚く(冗長に)なりがちです

``` app1_views.py
class IndexView(TemplateView):
    template_name: str = "app1/index.html"
    object_list = TestData.objects.all()

    def get(self, request, *args, **kwargs):
        object_list = self.object_list
        context = {
            "object_list": object_list,
            # たくさんの処理
            # ︙
        }
        return render(request, self.template_name, context)

    def post(self, request, *args, **kwargs):
        object_list = self.object_list
        context = {
            "object_list": object_list,
            # たくさんの処理
            # ︙
        }
        return render(request, self.template_name, context)

```

<br>

そこで簡潔に書くことのできる <br>
get_context_data をオーバーライドする方法を使うことでコードが汚くなるのを防ぐことができます


```app1_views.py
from django.views.generic import TemplateView

from .models import TestData


class IndexView(TemplateView):
    template_name: str = "app1/index.html"

    def get_context_data(self, **kwargs):
        ctx = super().get_context_data(**kwargs) # ctxにget_context_dataの中身(**kwargs)を代入
        ctx['object_list'] = TestData.objects.all() # ctxにデータを追加
        return ctx
```


`ctx = super().get_context_data(**kwargs)` でget_context_dataを継承してget_context_dataの中身をctxに代入しています<br>

<br>

get_context_dataの中身は辞書形式なので `ctx['object_list'] = TestData.objects.all()` でデータを渡しています

<br>

サーバーではこのように表示されます

![Alt text](https://i.gyazo.com/3958bdc7b2a97e69724640993a6bdbce.png)




# まとめ
TemplateViewでリストを作成するにはcontext_dataにデータを代入する


# 参考文献
- [ListViewの使い方と出来ること](https://qiita.com/aqmr-kino/items/d536e08a715a9fad5720)
- [Django でまず覚えたい TemplateView のパターン](https://qiita.com/ytyng/items/7cb3c3a5605974151678)
- [Djangoにおけるクラスベース汎用ビューの入門と使い方サンプル](https://qiita.com/dai-takahashi/items/7d0187485cad4418c073)

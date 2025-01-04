---
title: 【Django】 TemplateViewを使ってFormページを作る
topics: ["GitHubActions", "Ruby", "YAML"]
published: true
---


Templateviewを使ってFormページを作ろうとしたときに
思ったよりハマったのでTemplateviewを使ってListページを作る方法をまとめました

# 概要

TemplateViewを使ってFromページを作成する方法です

# この記事で伝えたいこと

- Templateviewを使ったFormページの作り方

# 結論

ModelFormMixinを継承する

```app1_views.py
from django.views.generic.edit import ModelFormMixin
from django.views.generic import TemplateView, ListView
from django.urls import reverse_lazy

from .forms import TestDataModelForm
from .models import TestData


class IndexView(TemplateView, ModelFormMixin):
    template_name: str = "app1/index.html"
    form_class = TestDataModelForm
    success_url = reverse_lazy('app1:data_list')
    model = TestData

    def get(self, request, *args, **kwargs):
        self.object = None
        return super().get(request, *args, **kwargs)

    def post(self, request, *args, **kwargs):
        self.object = None
        self.object_list = self.get_queryset()
        form = self.get_form()
        if form.is_valid():
            return self.form_valid(form)
        else:
            return self.form_invalid(form)
```

# ソースコード

ソースコードをGitHubに公開しています

<https://github.com/ANKM0/django_sample_make_form_with_templateview.git>

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

## models,formsの作成

Formを作成する際に使うmodelsとFormsを作成しました

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

```app1_forms.py
from django import forms
from .models import TestData


class TestDataModelForm(forms.ModelForm):

    class Meta:
        model = TestData
        fields = ("number", "name", "price")
```

# Formに必要な処理

Formに必要でTemplateViewにない機能は

- form画面
- バリデーションの結果から遷移画面を判断する処理

の2つです(他にもあるかもしれませんが)

これらの処理を追加することで　<br>
FormをTemplateViewで使えるようになります

Formに必要な処理をTemplateViewに追加する方法を説明していきます

## 方法その1 From画面とロジックを自作する方法

ないなら作ればいいということで <br>
From画面とロジックを自作する方法です

```index.html
{% extends "app1/base.html" %}
{% block title %}index{% endblock %}

{% block content %}
<div class="container">
    <h1>index page</h1>
    <br>
    <br>
    {{ error_list }}

    <form method="POST">
        {% csrf_token %}

        <span>
            <p>Number:
            <input type="number" name="number" min="0">
            </p>
        </span>
        <span>
            <p>Name:
            <input type="text" name="name" maxlength="200">
            </p>
        </span>
        <span>
            <p>Price:
            <input type="number" name="price" min="0">
            </p>
        </span>

        <button type="submit" name="submit">送信</button>
    </form>

</div>
{% endblock %}
```

inputタグでFormを作成して、min,maxlengthで入力出来る値を制限しています

ロジックはこんな感じで作成します

```app1_views.py
from django.views.generic import TemplateView, ListView
from django.http import HttpResponse
from django.shortcuts import render

from .models import TestData
from .forms import TestDataModelForm


class IndexView(TemplateView):
    template_name: str = "app1/index.html"
    form_class = TestDataModelForm

    def post(self, request, *args, **kwargs) -> HttpResponse:
        number = request.POST.get("number")
        name = request.POST.get("name")
        price = request.POST.get("price")

        default_data = {
            "number": number,
            "name": name,
            "price": price,
        }
        form = self.form_class(default_data)

        if form.is_valid():
            form.save()
        else:
            print(f"error:{form.errors}")

        context = {
            "error_list": form.errors,
        }

        return render(request, self.template_name, context)

```

`request.POST.get("[inputタグのname]")` でフォームに入力された値を取得して

`self.form_class(default_data)`　でバリデーション(値の検証)を実行して

`form.save()` でデータを保存しています

## 方法その2 djangoのformを使う

From画面を自作する場合、inputタグの数だけ処理を書く必要があります

``` app1_views.py
class IndexView(TemplateView):
    template_name: str = "app1/index.html"
    form_class = TestDataForm

    def post(self, request, *args, **kwargs) -> HttpResponse:
        number = request.POST.get("number")
        name = request.POST.get("name")
        price = request.POST.get("price")
        # inputタグの数だけ処理を書く
        # ︙

        default_data = {
            "number": number,
            "name": name,
            "price": price,
            # inputタグの数だけ処理を書く
            # ︙
        }
```

この書き方だとコードが汚く(冗長に)なるので

djangoが用意しているformを使います

```app1_views.py
from django.views.generic import TemplateView, ListView
from django.http import HttpResponse
from django.shortcuts import render, redirect

from .forms import TestDataModelForm
from .models import TestData


class IndexView(TemplateView):
    template_name: str = "app1/index.html"
    form_class = TestDataModelForm

    def get(self, request, *args, **kwargs) -> HttpResponse:
        form = self.form_class()
        context = {
            "form": form
        }
        return render(request, self.template_name, context)

    def post(self, request, *args, **kwargs) -> HttpResponse:
        form = self.form_class(request.POST)
        if form.is_valid():
            form.save()
            return redirect("app1:index")
        else:
            context = {"form": form}
            return render(request, self.template_name, context)
```

`return render(request, self.template_name, context)` でformをtemplateに渡しています

contextは辞書型にする必要があるので注意です

templateはこのようになります

```index.html
{% extends "app1/base.html" %}
{% block title %}index{% endblock %}

{% block content %}
<div class="container">
    <h1>index page</h1>
    <br>
    <br>

    <form method="POST">
        {% csrf_token %}
        {{ form.as_p }}
        <button type="submit">送信</button>
    </form>

</div>
{% endblock %}
```

## 方法その3 get_context_dataを使う

方法2では getとpostにそれぞれ処理を書いていましたが

getはデータの受け渡ししかしていないので 副作用を避けるために使わない方が良いです

そこでgetメソッドをget_context_dataメソッドに置き換えます

```app1_views.py
from django.views.generic import TemplateView, ListView
from django.http import HttpResponse
from django.shortcuts import redirect

from .forms import TestDataModelForm
from .models import TestData


class IndexView(TemplateView):
    template_name: str = "app1/index.html"
    form_class = TestDataModelForm

    def get_context_data(self, **kwargs):
        ctx = super().get_context_data(**kwargs)
        ctx["form"] = self.form_class()
        return ctx

    def post(self, request, *args, **kwargs) -> HttpResponse:
        form = self.form_class(request.POST)
        if form.is_valid():
            form.save()
            return redirect("app1:index")
```

`ctx = super().get_context_data(**kwargs)` でget_context_dataを継承してget_context_dataの中身をctxに代入

get_context_dataの中身は辞書形式なので `ctx["form"] = self.form_class()` でデータを渡しています

## 方法その4 ModelFormMixinを使う

実はフォーム表示に必要な属性やメソッドがModelFormMixinに用意されています

今までformの機能をメソッドを上書き(オーバーライド)することで追加してきましたが

Mixinを継承した方が綺麗に書けるのでMixinを使います

```app1_views.py
from django.views.generic.edit import ModelFormMixin
from django.views.generic import TemplateView, ListView
from django.urls import reverse_lazy

from .forms import TestDataModelForm
from .models import TestData


class IndexView(TemplateView, ModelFormMixin):
    # https://docs.djangoproject.com/en/4.1/ref/class-based-views/mixins-single-object/
    template_name: str = "app1/index.html"
    form_class = TestDataModelForm
    success_url = reverse_lazy('app1:data_list')
    model = TestData

    def get(self, request, *args, **kwargs):
        self.object = None
        return super().get(request, *args, **kwargs)

    def post(self, request, *args, **kwargs):
        self.object = None
        self.object_list = self.get_queryset()  # modelからformを作成するために必要
        form = self.get_form()
        if form.is_valid():
            return self.form_valid(form)
        else:
            return self.form_invalid(form)
```

FormViewを使用するときと同じで

- template_name <br>
使用するtemplateのパス

- form_class <br>
使用するForm

- success_url <br>
遷移先のurl

- model <br>
Formに使うmodel

をそれぞれ指定します

また、`self.object = None`をget/postで指定する必要があります

これは`ModelFormMixin`の親の`SingleObjectMixin`が`self.object`を持っているため <br>
`self.object`を使わないView(TemplateView)でも定義する必要があるからです

使わないときは`self.object = None`にする必要があります

# まとめ

TemplateViewでFormを作成するには　get_context_data か Mixinを使う

# 参考文献

- [Djangoの 汎用クラスビューをまとめて、実装について言及する](https://qiita.com/renjikari/items/af3e8958d2653e6f8d46)
- [Djangoのクラスベースビューを完全に理解する](https://www.membersedge.co.jp/blog/completely-guide-for-django-class-based-views/)
- [みかん箱でプログラミング FormMixinクラス](https://en-junior.com/formview/#form-mixin)
- [mixinの使い方](https://man.plustar.jp/django/ref/class-based-views/mixins-editing.html)
- [django公式ドキュメント SingleObjectMixin self.object = Noneにする必要がある](https://docs.djangoproject.com/en/4.1/ref/class-based-views/mixins-single-object/#django.views.generic.detail.SingleObjectMixin.get_context_data:~:text=requires%20that%20the-,self.object,-attribute%20be%20set)
- [メモ〜djangoでのself.objectの注意点](https://qiita.com/keishi04hrikzira/items/3e07cffa247895f841a0)

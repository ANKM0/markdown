---
title: 【Django】 フォームをテーブルの中に入れる
topics: ["GitHubActions", "Ruby", "YAML"]
published: true
---

# この記事で伝えたいこと

- 画像のような画面の作り方

![Alt text](https://i.gyazo.com/e9182634b076b68ce734980a6c7b3f08.png)

# 結論

Formsetとget_context_dataメソッドを使う

```app1/views.py
from django.views.generic import TemplateView
from django.http import HttpResponse
from django import forms
from django.shortcuts import render, redirect

from .models import TestData


class IndexView(TemplateView):
    template_name: str = "app1/index.html"
    object_list = TestData.objects.all().order_by("number")
    max_num = object_list.count()

    # モデルフォームセット
    test_data_formset = forms.modelformset_factory(
        model=TestData,
        fields=["quantity"],
        extra=3,  # default-> 1
        max_num=max_num
    )

    def get_context_data(self, **kwargs):
        # get処理だけ書く
        ctx = super().get_context_data(**kwargs)
        # 新規作成formを作る場合は
        # formset = self.test_data_formset(queryset=TestData.objects.none())
        ctx["formset"] = self.test_data_formset()
        ctx["object_list"] = self.object_list
        return ctx

    def post(self, request, *args, **kwargs) -> HttpResponse:
        formset = self.test_data_formset(request.POST)
        if formset.is_valid():
            formset.save()
            return redirect("app1:index")
        context = {
            "object_list": self.object_list,
            "formset": formset,
        }
        return render(request, self.template_name, context)

```

# ソースコード

ソースコードをGitHubに公開しています
<https://github.com/ANKM0/django_sample_make_form_and_list_with_templateview.git>

# 環境

- django4.1.3
- クラスベースビューを使います

## フォルダ構成

フォルダ構成はこんな感じになっています
プロジェクト名はconfig,アプリ名はapp1です

``` markdown
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
```

# 下準備

## models,formsの作成

Formを作成する際に使うmodelsとFormsを作成します

```app1/models.py
from django.db import models


class TestData(models.Model):
    number = models.PositiveIntegerField()
    name = models.CharField(max_length=200)
    price = models.PositiveIntegerField()
    quantity = models.PositiveIntegerField(blank=True, null=True)

    def __str__(self):
        return self.name

    class Meta:
        verbose_name_plural = "テストデータ"
```

```app1/forms.py
from django import forms
from .models import TestData


class QuantityForm(forms.Form):
    quantity = forms.IntegerField(
        label="price",
        min_value=0,
        required=False,
    )
```

## ルーティング設定

ルーティングも設定しておきます

```cofig/urls.py
from django.contrib import admin
from django.urls import path, include


urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('app1.urls')),
]
```

```app1/urls.py
from django.urls import path
from . import views


app/name = "app1"
urlpatterns = [
    path('', views.IndexView.as_view(), name='index'),
]
```

## templateのbaseを作成

templateに使うbase.htmlを作成します

```templates/app1/base.html
<!DOCTYPE html>
{% load static %}

<head>
    <!-- Required meta tags -->
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    {% block meta %}{% endblock %}

    <!-- Bootstrap CSS -->
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css" integrity="sha384-ggOyR0iXCbMQv3Xipma34MD+dH/1fQ784/j6cY/iJTQUOhcWr7x9JvoRxT2MZw1T" crossorigin="anonymous">

    <!-- Optional JavaScript -->
    <!-- jQuery first, then Popper.js, then Bootstrap JS -->
    <script src="https://code.jquery.com/jquery-3.3.1.slim.min.js" integrity="sha384-q8i/X+965DzO0rT7abK41JStQIAqVgRVzpbzo5smXKp4YfRvH+8abtTE1Pi6jizo" crossorigin="anonymous"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.7/umd/popper.min.js" integrity="sha384-UO2eT0CpHqdSJQ6hJty5KVphtPhzWj9WO1clHTMGa3JDZwrnQq4sF86dIHNDz0W1" crossorigin="anonymous"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js" integrity="sha384-JjSmVgyd0p3pXB1rRibZUAYoIIy6OrQ6VrjIEaFf/nJGzIxFDsf4x0xIM+B07jRM" crossorigin="anonymous"></script>

    <!-- FontAwesome-->
    <link rel="stylesheet" href="https://pro.fontawesome.com/releases/v5.10.0/css/all.css" integrity="sha384-AYmEC3Yw5cVb3ZcuHtOA93w35dYTsvhLPVnYs9eStHfGJvOvKxVfELGroGkvsg+p" crossorigin="anonymous" />

    <!-- https://github.com/yubinbango/yubinbango -->
    <script src="https://yubinbango.github.io/yubinbango/yubinbango.js" charset="UTF-8"></script>

    <link rel="stylesheet" href="{% static 'css/app1/main.css' %}">
    {% block csslink %}{% endblock %}

    <title>{% block title %}{% endblock %}</title>
</head>



<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <header class="mb-4">
        <nav class="navbar navbar-expand-sm navbar-dark bg-dark">
            <div><p class="navbar-brand">ヘッダー</p></div>
        </nav>
    </header>

    <main class="mb-5">
        {% block content %}{% endblock %}
    </main>

    <footer class="fixed-bottom">
        <div class="container">
        </div>
    </footer>

</body>
</html>
```

# 本題　フォームinテーブルの作り方

フォームをテーブルの中に入れる方法を説明していきます

## 方法その1 フォーム画面とロジックを自作する

Form画面とロジックのコードを書く方法です

画面

```templates/app1/index.html
{% extends "app1/base.html" %}
{% block title %}index{% endblock %}

{% block content %}
<div class="container">
    <h1>index page</h1>
    <br>
    <br>
    {{error_list.0}}

    <form class="form" action="" method="post" enctype="multipart/form-data">
        {% csrf_token %}

        <div class="bg-light">
            <table class="table table-bordered ">
                <thead>
                    <tr class="table-light">
                        <th scope="col">商品番号</th>
                        <th scope="col">商品名</th>
                        <th scope="col">価格</th>
                        <th scope="col">数</th>
                    </tr>
                </thead>
                <tbody>
                    {% for item in object_list %}
                        <tr class="bg-light">
                            <td>
                                {{ item.number }}
                            </td>
                            <td>
                                {{ item.name }}
                            </td>
                            <td>
                                {{ item.price }}
                            </td>
                            <td>
                                <div class="form-group">
                                    <input class="form-control" type="number" name="quantity_{{forloop.counter}}"
                                    {% if item.quantity == 0 %}
                                        value=""
                                    {% else %}
                                        value="{{item.quantity}}"
                                    {% endif %}
                                    min="0">
                                </div>
                            </td>
                        </tr>

                    {% endfor %}
                </tbody>
            </table>
        </div>
        <div class="text-right">
            <button type="submit" value="送信">送信</button>
        </div>

    </form>
</div>
{% endblock %}
```

ロジック

```app1/urls.py
from django.views.generic import TemplateView
from django.http import HttpResponse
from django.shortcuts import render, redirect


from .models import TestData
from .forms import QuantityForm


class IndexView(TemplateView):
    template_name: str = "app1/index.html"
    form_class = QuantityForm

    def get(self, request, *args, **kwargs) -> HttpResponse:
        object_list = TestData.objects.all().order_by("number")
        context = {
            "object_list": object_list,
        }
        return render(request, self.template_name, context)

    def post(self, request, *args, **kwargs) -> HttpResponse:
        quantity_list: list[str] = []
        error_list: int[str] = []

        object_list = TestData.objects.all().order_by("number")
        for quantity_id in range(1, object_list.count()+1):
            raw_quantity: str = request.POST.get(f"quantity_{quantity_id}")
            if raw_quantity == "":
                quantity_list += [-404]
            elif int(raw_quantity) < 0:
                quantity_list += [-403]
            else:
                quantity_list += [int(raw_quantity)]

        if len([i for i, x in enumerate(quantity_list) if x == -404]) == len(quantity_list):
            error_list += ["数を入力してください"]
        elif -403 in quantity_list:
            error_list += ["0以下の数が入力されています"]

        if len(error_list) > 0:
            object_list = TestData.objects.all().order_by("number")
            context = {
                "object_list": object_list,
                "error_list": error_list,
            }
            return render(request, self.template_name, context)

        for counter in range(0, len(quantity_list)):
            print("---------------")
            quantity: int = quantity_list[counter]
            print(f"quantity_list:{quantity_list}")
            print(f"quantity:{quantity}")

            if not quantity == -404 or quantity == 0:
                object = TestData.objects.get(number=counter+1)
                object.quantity = quantity
                object.save()
        return redirect("app1:index")
```

formで必要な処理
入力画面を表示->バリデーション->DBに登録

のうち、

- 入力画面
- DBに登録する処理

の2つを実装しています

この方法だと、処理を都度書く必要があるのでコードが汚く(冗長に)なります
そこで、Formsetを使います

## 方法その2 Formsetを使う

画面
※Django-Boostを使っています

```templates/app1/base.html
{% extends 'app1/base.html' %}
{% load boost %}
{% block title %}index{% endblock %}

{% block content %}
<div class="container">
    <h1>index page</h1>
    <br>
    <br>
    {{error_list.0}}

    <form class="form" action="" method="post" enctype="multipart/form-data">
        {% csrf_token %}
        {{ formset.management_form }}

        <div class="bg-light">
            <table class="table table-bordered ">
                <thead>
                    <tr class="table-light">
                        <th scope="col">商品番号</th>
                        <th scope="col">商品名</th>
                        <th scope="col">価格</th>
                        <th scope="col">数</th>
                    </tr>
                </thead>
                <tbody>
                    {% for item, form in object_list|zip:formset %}
                        <tr class="bg-light">
                            <td>
                                {{ item.number }}
                            </td>
                            <td>
                                {{ item.name }}
                            </td>
                            <td>
                                {{ item.price }}
                            </td>
                            <td>
                                {{ form.non_field_errors }}
                                <div class="form-group">
                                    <label for="{{ form.quantity.id_for_label }}">{{ form.quantity.label_tag }}</label>
                                {{ form.quantity }}
                                {{ form.quantity.errors }}

                                {% for field in form.hidden_fields %}
                                    {{ field }}
                                {% endfor %}
                                </div>
                            </td>
                        </tr>
                    {% endfor %}
                    {% for field in form.hidden_fields %}
                        {{ field }}
                    {% endfor %}
                </tbody>
            </table>
        </div>
        <div class="text-right">
            <button type="submit" class="btn btn-primary">送信</button>
        </div>

    </form>
</div>
{% endblock %}
```

ロジック

```app1/urls.py

from django.views.generic import TemplateView
from django.http import HttpResponse
from django import forms
from django.shortcuts import render, redirect

from .models import TestData


class IndexView(TemplateView):
    template_name: str = "app1/index.html"
    object_list = TestData.objects.all().order_by("number")
    max_num = object_list.count()

    # フォームセット(モデルと紐づけたいので、モデルフォームセットを使用)
    test_data_formset = forms.modelformset_factory(
        model=TestData,
        fields=["quantity"],
        extra=3,  # default-> 1
        max_num=max_num    # object_list.count()が3を返すとき、initial含めformは最大4となる
    )

    def get(self, request, *args, **kwargs) -> HttpResponse:
        # 新規作成formを作る場合は下のコメントアウトのように記述
        # formset = self.test_data_formset(queryset=TestData.objects.none())
        formset = self.test_data_formset()
        context = {
            "object_list": self.object_list,
            "formset": formset,
        }
        return render(request, self.template_name, context)

    def post(self, request, *args, **kwargs) -> HttpResponse:
        formset = self.test_data_formset(request.POST)
        if formset.is_valid():
            formset.save()
            return redirect("app1:index")
        context = {
            "object_list": self.object_list,
            "formset": formset,
        }
        return render(request, self.template_name, context)
```

Formsetを使用することで、必要なコードを少なくできました

しかし、getメソッドとpostメソッドで処理が重複しています
そこで、get_context_dataメソッドを使い、更にきれいなコードにします

## 方法3 Formset+get_context_dataメソッド

```app1/urls.py
from django.views.generic import TemplateView
from django.http import HttpResponse
from django import forms
from django.shortcuts import render, redirect

from .models import TestData


class IndexView(TemplateView):
    template_name: str = "app1/index.html"
    object_list = TestData.objects.all().order_by("number")
    max_num = object_list.count()

    test_data_formset = forms.modelformset_factory(
        model=TestData,
        fields=["quantity"],
        extra=3,
        max_num=max_num
    )

    def get_context_data(self, **kwargs):
        # get処理だけ書く
        ctx = super().get_context_data(**kwargs)
        # 新規作成formを作る場合は
        # formset = self.test_data_formset(queryset=TestData.objects.none())
        ctx["formset"] = self.test_data_formset()
        ctx["object_list"] = self.object_list
        return ctx

    def post(self, request, *args, **kwargs) -> HttpResponse:
        formset = self.test_data_formset(request.POST)
        if formset.is_valid():
            formset.save()
            return redirect("app1:index")
        context = {
            "object_list": self.object_list,
            "formset": formset,
        }
        return render(request, self.template_name, context)
```

```ctx["キー"] = self.データ```と書くことでIndexViewクラスにデータを登録しています

get_context_dataはgetのときだけ呼び出されるので
```formset = self.test_data_formset(request.POST)```とすることでpostされたデータを渡しています

# まとめ

フォームをテーブルの中に入れるには、Formsetとget_context_dataメソッドを使う

# 参考文献

- [Formsets | Django documentation | Django](https://docs.djangoproject.com/en/4.1/topics/forms/formsets/)
- [Creating forms from models | Django documentation | Django](https://docs.djangoproject.com/en/4.1/topics/forms/modelforms/#model-formsets:~:text=%22birth_date%22%2C)
- [Django Formsetを使ってFormをたくさん並べよう!](https://qiita.com/qtatsunishiura/items/a6cc11e025aca1c16ed1)

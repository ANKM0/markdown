## Hello, world!まで
### パッケージ導入
pip install pipenv <br>
pipenv shell <br>
pipenv install --dev autopep8 flake8 <br>
pipenv install django==4.1.3 <br>
### プロジェクト、アプリ作成
mkdir ルートファイル (ルートファイルを作成) <br>
cd ルートファイル <br>
django-admin startproject config . <br>
python manage.py startapp app1 <br>

### 設定
(消すかも)ーーーーー <br>
mkdir templates/app1 <br>
cd  <br>
D:\programs\django_sample_make_form_or_list_with_templateview\django_sample\app1 <br>
type nul > forms.py  <br>
type nul > urls.py  <br>
mkdir templates  <br>
cd templates <br>
mkdir app1 <br>
cd app1 <br>
type nul > base.html <br>
cd  <br>D:\programs\django_sample_make_form_or_list_with_templateview\django_sample <br>
ーーーーー <br>
INSTALLED_APPS に app1 を追加 <br>
TIME_ZONE = 'Asia/Tokyo' に変更 <br>
LANGUAGE_CODE = 'ja' に変更 <br>
データベース 'NAME': str(BASE_DIR / 'db.sqlite3'), に変更 <br>
TEMPLATESのパスを os.path.join(BASE_DIR, 'templates'), に設定 <br>

static,media を設定 <br>
```
STATIC_URL = "static/"
STATICFILES_DIRS = [
    os.path.join(BASE_DIR, "local_static")
]
STATIC_ROOT = os.path.join(BASE_DIR, 'static')

MEDIA_URL = '/media/'
# MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
MEDIA_ROOT = os.path.join(BASE_DIR.parent.parent, 'media')
```
<br>

python manage.py makemigrations <br>
python manage.py migrate <br>

https://hiroki-sawano.hatenablog.com/entry/django-secrets 参考に  <br>
SECRET_KEYを変更  <br>
SECRET_KEY = 'django-insecure-j05_(nt-x9i^w%l$kcz+-uo909t9&(fx_!1=bz%6p_zaa2&9^b' <br>

```
if os.path.exists(os.path.join(BASE_DIR, 'secrets.json')):
    with open(os.path.join(BASE_DIR, 'secrets.json')) as secrets_file:
        secrets = json.load(secrets_file)
else:
    print('secrets.json could not be found.')
    quit()


def get_secret(setting, secrets=secrets, is_optional=False):  # 設定が見つからない場合にNoneを返したい場合にはis_optionalにTrueを設定
    try:
        secret = secrets[setting]
        if secret:
            return secret
        else:
            if is_optional:
                return None
            else:
                print(f'Please set {setting} in secrets.json')
                quit()
    except KeyError:
        print(f' Please set {setting} in secrets.json')
        quit()
SECRET_KEY = get_secret('SECRET_KEY')
```


cd D:\programs\django_sample_make_form_or_list_with_templateview\django_sample <br>
type nul > gen_secrets.py <br>

```
import json
import os

from django.core.management.utils import get_random_secret_key

BASE_DIR = os.path.dirname(
    os.path.dirname(
        os.path.abspath(__file__)
    )
)

secrets = {
    'SECRET_KEY': get_random_secret_key(),
}

with open(os.path.join(BASE_DIR, 'secrets.json'), 'w') as outfile:
    json.dump(secrets, outfile)
```

python django_sample\gen_secrets.py <br>

cd D:\programs\django_sample_make_form_or_list_with_templateview\django_sample <br>
type nul > .gitignore <br>
secrets.json (と gen_secrets.py) をgitignoreに追加 <br>



## indexページ、データ作成
### indexを表示できるようにする

``` python
# config/views
from django.contrib import admin
from django.urls import path, include


urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('app1.urls')),
]
```

``` python
# app1/urls
from django.urls import path
from . import views


app_name = "app1"
urlpatterns = [
    path('', views.IndexView.as_view(), name='index'),
]
```

``` python
# app1/views
from django.views.generic import TemplateView, ListView, DetailView


class IndexView(TemplateView):
    template_name: str = "app1/index.html"
```

``` django
<!-- app1/base.html -->
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

``` django
<!-- app1/index.html -->
{% extends "app1/base.html" %}
{% block title %}index{% endblock %}

{% block content %}
<div class="container">
    <h1>index page</h1>
</div>
{% endblock %}
```

### データを追加する
``` python
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

python manage.py makemigrations <br>
python manage.py migrate <br>

python manage.py createsuperuser <br>
xings <br>
test@test.gmail.com <br>
e9d9f16 <br>

adminにモデルを登録する
``` python
from django.contrib import admin
from .models import TestData


@admin.register(TestData)
class ProductDataAdmin(admin.ModelAdmin):
    model = TestData
    list_display = ["number", "name", "price"]
    ordering = ["number"]
```

adminページからデータを登録する







html変更
admin追加




python manage.py createsuperuser

a@a.gmail.com
gllilwe9d9f16



git checkout -b feature/#5















from django.views.generic import TemplateView
from django.http import HttpResponse
from django.shortcuts import render, redirect

from .forms import TestDataModelForm


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










pipenv shell
pipenv install
pipenv install --dev
python "django_sample\gen_secrets.py"
django_sample直下にsecrets.json移動
cd django_sample
python manage.py runserver
python manage.py createsuperuser
xings
test@test.gmail.com
gllilwe9d9f16

python manage.py makemigrations
python manage.py migrate










git ロールバック
git reset --hard e4997212bcb71493b7664d1a96d114117b7bb116
git push -f origin HEAD^:feature/#23



git tag v1.1.1
git tag -a タグ -m 'タグのコメント'
git push origin v1.1.1

git tag -d v1.1.1














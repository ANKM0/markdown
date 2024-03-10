---
title: 【Django】 manage.pyを介さずに外部からDjangoのORMを使ってDBを操作したい
topics: ["GitHubActions", "Ruby", "YAML"]
published: true
---

## manage.pyを介さずにDjangoのORMを呼びたい

外部のcsvから値をDBに入れる際に
Djangoの処理を呼ばずに、ORMだけを呼び出して、処理を実行したかった

ORMを呼び出すときは　BaseCommandを継承して、新規コマンドを作成、 `python manage.py [新規コマンド]`ってすればできる
でも、今回はアプリで何かするわけじゃなくて、値を入れるだけ、
つまり、ORMだけ使いたい

というわけで、ORMを外部から使える方法を探しました

## ファイルツリー

```bash
website_with_wix # ルートdir
|--Pipfile
|--Pipfile.lock
|--__init__.py # <-追加
|--account # アプリ
|  |--__init__.py
|  |--admin.py
|  |--apps.py
|  |--forms.py
|  |--migrations
|  |  |--略
|  |--models.py
|  |--tests.py
|  |--urls.py
|  |--views.py
|--app1 # アプリ
|  |--__init__.py
|  |--__pycache__
|  |  |--略
|  |--admin.py
|  |--apps.py
|  |--forms.py
|  |--migrations
|  |  |--略
|  |--models.py
|  |--templatetags
|  |  |--__init__.py
|  |  |--__pycache__
|  |  |  |--略
|  |  |--mytag.py
|  |--tests.py
|  |--urls.py
|  |--views.py
|--config # プロジェクトdir
|  |--__init__.py # <-修正 (必要ないかも)
|  |--__pycache__
|  |  |  |--略
|  |--asgi.py
|  |--settings.py
|  |--urls.py
|  |--wsgi.py
|--manage.py
|--orm # <-追加
|  |--orm_sample.py # <-追加
```

## 解決策

下記の手順で、ORMだけを利用できるようになります

- sys.pathにDjangoのプロジェクトを指定
- Djangoの環境変数をsettings.configure()の引数に指定
- modelsのimport

### 手順

`website_with_wix/__init__.py`を作成

`website_with_wix/orm/orm_sample.py`を作成
下記コードを追加する

```py
import sys
sys.path.append("ルートdirのパス")
from django.conf import settings
import django
from  config.settings import DATABASES, INSTALLED_APPS, PREFECTURES
# settingsの中身をimportする
settings.configure(DATABASES=DATABASES, INSTALLED_APPS=INSTALLED_APPS)
django.setup()

from "アプリ名".models import "モデル名"
print("モデル名".objects.all())
# >> <QuerySet []>

# コード例
#
# import sys
# sys.path.append("public_html/project_dir/neoseruko_djnago_projects/website_with_wix/")
# from django.conf import settings
# import django
# from  config.settings import DATABASES, INSTALLED_APPS, PREFECTURES
# # 筆者の環境では、PREFECTURES定数をsettingsに追加しているので、orm_sample.pyでも`PREFECTURES=PREFECTURES`を追加している
# settings.configure(DATABASES=DATABASES, INSTALLED_APPS=INSTALLED_APPS, PREFECTURES=PREFECTURES)
# django.setup()

# from app1.models import UserInfo
# print(UserInfo.objects.all())
```

`website_with_wix/config/__init__.py`に下記コードを追加

```python
import pymysql

pymysql.install_as_MySQLdb()
```

実行結果

```bash
(.venv) [website_with_wix]$ pwd
./website_with_wix
python orm/orm_sample.py
<QuerySet []>
```

## 補足

この作業を行っている際に`django.core.exceptions.ImproperlyConfigured: Error loading MySQLdb module.
Did you install mysqlclient?`が発生した

恐らくconfigを設定した際にDBの設定を上手く読み込めなくなったことが原因(未調査)

対象法としては、
`pip install pymysql`コマンドでpymysqlをインストールした上で
`settings.pyのある階層の__init__.py`に下記のように記述する

```py
import pymysql

pymysql.install_as_MySQLdb()
```

## 参考

- Can one use the Django database layer outside of Django?
  - <https://stackoverflow.com/questions/2180415/can-one-use-the-django-database-layer-outside-of-django>
- Importing models from outside applications
  - <https://groups.google.com/g/django-users/c/vcQo6ybbJyg/m/5CW-D64oVe4J>
- 外部アプリからもDjangoのORMを使ってDBを操作したい
  -<https://qiita.com/Mabuchin/items/2c3cb1dc9f92f98fa22d>
- Djangoのtest実行時に発生するデータベースエラー Got an error creating the test database: permission denied to create database
  - <https://qiita.com/yutoun/items/9b1638f41d7c50341601>

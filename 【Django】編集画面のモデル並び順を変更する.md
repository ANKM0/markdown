# 【Django】編集画面のモデル並び順を変更する

## やりたいこと

- Djangoの管理画面のモデル並び順
- デフォルトで、アルファベット順になっているこれを
- 任意の順番に変更すること

## 解決策

- 管理画面のモデル並び順は`django\contrib\admin\sites.py`の`AdminSite`クラスの`get_app_list`メソッドで、定義されている
- これをオーバーライドする

```python
class AdminSite:
    # 略
    def get_app_list(self, request, app_label=None):
        """
        Return a sorted list of all the installed apps that have been
        registered in this site.
        """
        app_dict = self._build_app_dict(request, app_label)

        # Sort the apps alphabetically.
        app_list = sorted(app_dict.values(), key=lambda x: x["name"].lower())

        # Sort the models alphabetically within each app.
        for app in app_list:
            app["models"].sort(key=lambda x: x["name"])

        return app_list
```

具体的には、settings.pyを以下のように編集する

```python
# admin画面の並び順を変更する
ADMIN_ORDERING = [
    ('app1', [
        'ProtectUsage',
        'ProtectKind',
        'VersionUpDownloadFile',
        'Cart',
        'Prefecture',
        'Series',
        'Category',
        'ProductData',
        'SPData',
        'BuyingHistory',
        'UserInfo',
        'CartItem',
    ]),
]
def get_app_list(self, request):
    # app1の部分の設定
    app_dict = self._build_app_dict(request)
    for app_name, object_list in ADMIN_ORDERING:
        app = app_dict[app_name]
        app['models'].sort(key=lambda x: object_list.index(x['object_name']))
        yield app

        # それ以外の部分の設定
        explicit_app_names = [app_name for app_name, object_list in ADMIN_ORDERING]
        for app_name in app_dict:
            if app_name not in explicit_app_names:
                yield app_dict[app_name]
admin.AdminSite.get_app_list = get_app_list
```

## 補足

- メソッドをオーバーライドする今回の方法以外にも
- [テンプレートをカスタマイズ方法](https://docs.djangoproject.com/en/4.2/intro/tutorial07/#customize-the-admin-index-page)でも、同じことができる
- こちらの方がスコープが狭いので、副作用を気にする場合はこちらがおすすめ

## 参考

- <https://stackoverflow.com/questions/398163/ordering-admin-modeladmin-objects-in-django-admin>
- <https://stackoverflow.com/questions/398163/ordering-admin-modeladmin-objects-in-django-admin>

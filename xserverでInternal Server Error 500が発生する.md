---
title: xserverでInternal Server Error 500が発生する
topics: ["GitHubActions", "Ruby", "YAML"]
published: true
---



## 問題

Xserverでdjangoをデプロイすると、`Internal Server Error 500`が発生した

## 原因

- `pip` に`PyMySQL`がインストールされていない
- `index.cgi`の設定が間違っている

## 解決策

- Xserverのエラーログを確認する

  - サーバー管理>アクセス解析>エラーログ

    ```log
    xsrv.jp.error_log

    [Tue Oct 31 22:32:35.117072 2023] [cgid:error] [pid 3012133:tid 3012253] [client 106.154.147.88:55900] End of script output before headers: index.cgi
    [Tue Oct 31 22:32:47.929607 2023] [cgid:error] [pid 3012133:tid 3012278] [client 106.154.147.88:42784] End of script output before headers: index.cgi
    [Tue Oct 31 22:32:49.228605 2023] [cgid:error] [pid 3012135:tid 3012633] [client 106.154.147.88:42790] End of script output before headers: index.cgi
    [Tue Oct 31 22:32:56.847652 2023] [cgid:error] [pid 3012135:tid 3012561] [client 106.154.147.88:35092] End of script output before headers: index.cgi
    [Tue Oct 31 22:53:52.240274 2023] [cgid:error] [pid 3012135:tid 3012679] [client 106.154.147.88:46488] End of script output before headers: index.cgi
    [Tue Oct 31 22:53:57.136246 2023] [cgid:error] [pid 3012135:tid 3012667] [client 106.154.147.88:41166] End of script output before headers: index.cgi
    [Tue Oct 31 22:54:05.762775 2023] [cgid:error] [pid 3012133:tid 3012173] [client 106.154.147.88:45378] End of script output before headers: index.cgi
    [Tue Oct 31 22:54:29.292636 2023] [cgid:error] [pid 3012134:tid 3012529] [client 106.154.147.88:47134] End of script output before headers: index.cgi
    ```

- `End of script output before headers`なので、cgiが動いていないことが分かる
- index.cgiを直接実行してエラー箇所を確認する

  ```cmd
  cd /home/xs797670/xs797670.xsrv.jp/public_html/project_dir/
  ./index.cgi

  >File "/home/xs797670/xs797670.xsrv.jp/public_html/project_dir/.venv/lib/python3.12/site-packages/django/db/backends/mysql/base.py", line 17, in <module>
  >    raise ImproperlyConfigured(
  >django.core.exceptions.ImproperlyConfigured: Error loading MySQLdb module.
  >Did you install mysqlclient?
  ```

- `PyMySQL`が入っていないので、インストールする

  ```cmd
  pip3 show PyMySQL
  >WARNING: Package(s) not found: PyMySQL

  pip3.12 install pymysql
  pip3.12 show pymysql
  >Name: PyMySQL
  ```

- もう一度`index.cgi`を実行

    ```cmd
    cd /home/xs797670/xs797670.xsrv.jp/public_html/project_dir/
    ./index.cgi

    >File "/home/xs797670/xs797670.xsrv.jp/public_html/project_dir/.venv/lib/python3.12/site-packages/django/core/handlers/wsgi.py", line 71, in __init__
    >self.method = environ["REQUEST_METHOD"].upper()
    >              ~~~~~~~^^^^^^^^^^^^^^^^^^
    >KeyError: 'REQUEST_METHOD'
    >Status: 500 Internal Server Error
    >Content-Type: text/plain
    >Content-Length: 59
    ```

- `environ`の値が足りない?ので、追加する

    ```index.cgi
    import sys, os
    import pymysql
    from wsgiref.handlers import CGIHandler
    from django.core.wsgi import get_wsgi_application

    sys.path.insert(0, "/home/xs797670/xs797670.xsrv.jp/public_html/project_dir/.venv/bin")
    sys.path.append('/home/xs797670/xs797670.xsrv.jp/public_html/project_dir/neoseruko_djnago_projects/website_with_wix')
    sys.path.append('/home/xs797670/xs797670.xsrv.jp/public_html/project_dir/neoseruko_djnago_projects/website_with_wix/config')


    os.environ.setdefault("DJANGO_SETTINGS_MODULE", "config.settings")

    os.environ['DJANGO_SETTINGS_MODULE'] = 'config.settings'

    # ここを追加
    os.environ["SERVER_NAME"] = "http://xs797670.xsrv.jp/"
    os.environ["SERVER_PORT"] = "443"
    os.environ["REQUEST_METHOD"] = "GET"
    os.environ['OPENBLAS_NUM_THREADS'] = "1"

    pymysql.install_as_MySQLdb()


    application = get_wsgi_application()
    CGIHandler().run(application)
    ```

- `index.cgi`を実行して確認する

    ```bash
    ./index.cgi
    >Status: 400 Bad Request
    >Content-Type: text/html; charset=utf-8
    >X-Content-Type-Options: nosniff
    >Referrer-Policy: same-origin
    >Cross-Origin-Opener-Policy: same-origin
    >
    >
    ><!doctype html>
    ><html lang="en">
    ><head>
    ><title>Bad Request (400)</title>
    ></head>
    ><body>
    ><h1>Bad Request (400)</h1><p></p>
    ></body>
    ></html>
    ```

## 参考にしたWebページ

### アクセスログ

- [【Python × Django × エックスサーバー】ウェブアプリをデプロイするまで](https://techtech-step.com/tech/2022/896/#toc13)

### cgi実行

- [【Django】エックスサーバーの仮想環境をcondaに変更したら500エラーになった](https://freemas.org/python/django/error-xserver-conda)

### `index.cgi` 設定値の追加

- [【レンタルサーバー】バリューサーバー新仕様マイグレーションへの対応メモ【Python/Flask】](https://karupoimou.hatenablog.com/entry/20211213/1639405861)

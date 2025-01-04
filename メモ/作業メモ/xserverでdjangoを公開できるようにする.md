# xserverでdjangoを公開できるように設定した際のメモ <!-- omit in toc -->

## 目次 <!-- omit in toc -->

- [SSH接続設定](#ssh接続設定)
- [Gitインストール](#gitインストール)
- [Vimインストール](#vimインストール)
- [Homebrewインストール](#homebrewインストール)
- [Pythonのインストール](#pythonのインストール)
- [neoseruko\_djnago\_projectsの導入](#neoseruko_djnago_projectsの導入)
- [DBの設定](#dbの設定)
- [CGI の設定](#cgi-の設定)
- [FCGI への対応](#fcgi-への対応)
- [参考URL](#参考url)
  - [SSH接続](#ssh接続)
  - [Git インストール](#git-インストール)
  - [Vim インストール](#vim-インストール)
  - [Homebrewインストール](#homebrewインストール-1)
  - [mysql接続](#mysql接続)
  - [CGI の設定](#cgi-の設定-1)
  - [FCGIの設定](#fcgiの設定)

## SSH接続設定

- 公開鍵の作成

  以下のコマンドを実行して公開鍵を作成する

  ```cmd
  ssh-keygen -t rsa -f xs797670.key
  ```

  `xs797670.pub`が公開鍵

  ```chmod 600 ~/.ssh/keys/xs797670.key```でパーミッションを変更

- ssh設定をONに変更

  - xserverサーバーパネル
  - アカウント>ssh設定
  - 「ONにする」クリック

- 公開鍵の登録

  - xserverサーバーパネル
  - アカウント>ssh設定
  - \>公開鍵登録・更新
  - `xs797670.pub`の中身を登録する

- 接続する

  sshコンフィグに下記の内容を設定する

  ```SSH Cofig
  Host XSERVER
    HostName [サーバー番号(sv***)].xserver.jp
    User xs797670 # サーバーID
    Port 10022
    IdentityFile "\.ssh\keys\xs797670.key"
    IdentitiesOnly yes
  ```

  > [!Notion]
  >
  > `Permission denied`が出て接続できないときは<br>
  > `chmod 600 ~/.ssh/keys/xs797670`を実行する

## Gitインストール

パスを通す

1. `vi ~/.bashrc`で~/.bashrceを開く
2. ~/.bashrcに書き込む

    ```cmd
    # .bashrc

    # Source global definitions
    if [ -f /etc/bashrc ]; then
            . /etc/bashrc
    fi

    # User specific aliases and functions

    export PATH=$HOME/opt/bin:$HOME/bin:$PATH:$HOME/opt/curl/bin:$HOME/opt/git/bin:$HOME/opt/gettext/bin:$HOME/opt/ssl/bin # ここを追加
    ```

3. `source ~/.bashrc` で設定を更新する

下記コマンドを実行して、ビルド時のコンパイラフラグを追加

```cmd
export CFLAGS=-fPIC
```

- opensslのインストール

  ```cmd
  cd /tmp/
  wget http://www.openssl.org/source/openssl-1.0.2k.tar.gz
  tar xvfz openssl-1.0.2k.tar.gz
  cd openssl-1.0.2k
  ./config shared --prefix=$HOME/opt/ssl --openssldir=$HOME/opt/ssl
  make
  make install
  ```

  ダウンロードできているか確認する

  ```cmd
  which openssl
  > ~/opt/ssl/bin/openssl
  ```

  find / grep | .bash_profile

- curlのインストール

  ```cmd
  cd /tmp/
  wget https://curl.haxx.se/download/curl-7.65.3.tar.gz
  tar zxvf curl-7.65.3.tar.gz
  cd curl-7.65.3
  ./configure --with-ssl=$HOME/opt/ssl --enable-libcurl-option --prefix=$HOME/opt/curl --enable-shared
  make
  make install
  pushd ~/opt/curl/lib
  ln -s $HOME/opt/ssl/lib/libssl.so.1.0.0 libssl.so.1.0.0
  ln -s $HOME/opt/ssl/lib/libcrypto.so.1.0.0 libcrypto.so.1.0.0
  popd
  ```

  確認

  ```cmd
  curl -V
  >curl 7.65.3 (x86_64-pc-linux-gnu) libcurl/7.65.3 OpenSSL/1.0.2k zlib/1.2.7

  which curl
  > ~/opt/bin/curl
  ```

- gettextのインストール

  ```cmd
  cd /tmp/
  wget https://ftp.gnu.org/pub/gnu/gettext/gettext-0.21.tar.gz
  tar zxvf gettext-0.21.tar.gz
  cd gettext-0.21
  ./configure --prefix=${HOME}/opt/gettext
  make
  make install
  cd -
  ```

- gitのインストール

  ```cmd
  cd /tmp/
  wget https://www.kernel.org/pub/software/scm/git/git-2.17.0.tar.gz
  tar zxvf git-2.17.0.tar.gz
  cd git-2.17.0
  ./configure --prefix=$HOME/opt/git --with-openssl=$HOME/opt/ssl  --with-curl=$HOME/opt/curl --enable-shared
  make
  make install
  ```

  確認

  ```cmd
  git --version
  >git version 2.17.0

  which git
  > ~/opt/git/bin/git
  ```

  `git clone https://github.com/vim/vim.git`で`git: 'remote-https' is not a git command. See 'git --help'.`が発生する時は<br>
  `git-git-remote-https`がビルドされていないので、下記コマンドを実行する

  ```cmd
  cp /usr/libexec/git-core/git-remote-https ~/opt/git/libexec/git-core/git-remote-https
  ```

## Vimインストール

- ncurses のインストール

  ```cmd
  cd /tmp/
  wget https://invisible-island.net/datafiles/release/ncurses.tar.gz
  tar zxvf ncurses.tar.gz
  cd ncurses-6.3/
  ./configure --prefix=${HOME}/opt/ncurses
  make
  make install
  ```

- Vimのインストール

  ```cmd
  cd /tmp/
  git clone https://github.com/vim/vim.git
  cd vim
  ./configure --prefix=${HOME}/opt/git --with-local-dir=${HOME}/opt/ncurses
  make
  make install
  ```

  確認

  ```cmd
  which vim
  > ~/opt/git/bin/vim

  vim --version
  > VIM - Vi IMproved 9.0 (2022 Jun 28, compiled Oct 23 2023 18:20:40)
  ```

  インサートモードでBS,Delを有効に設定する

  ```cmd
  # vim ~/.vimrc
  set backspace=indent,eol,start
  ```

`git clone https://github.com/vim/vim.git`実行

## Homebrewインストール

- ~/.bashrcを更新する

  `vim ~/.bashrc`

  ```cmd
  export HOMEBREW_FORCE_BREWED_GIT=
  export HOMEBREW_FORCE_BREWED_CURL=
  export HOMEBREW_CURL_PATH=${HOME}/opt/curl/bin/curl
  export HOMEBREW_GIT_PATH=${HOME}/opt/git/bin/git
  export HOMEBREW_PREFIX=${HOME}/.linuxbrew
  export HOMEBREW_CELLAR=${HOME}/.linuxbrew/Cellar
  export HOMEBREW_REPOSITORY=${HOME}/.linuxbrew/Homebrew
  export MANPATH=${HOME}/.linuxbrew/share/man${MANPATH+:$MANPATH}
  export INFOPATH=${HOME}/.linuxbrew/share/info:${INFOPATH:-}
  export PATH=$HOME/.linuxbrew/opt/binutils/bin:$HOME/.linuxbrew/bin:$HOME/.linuxbrew/sbin:$HOME/opt/bin:$HOME/bin:$HOME/opt/curl/bin:$HOME/opt/git/bin:$HOME/opt/gettext/bin:$HOME/opt/ssl/bin:$PATH
  ```

- 下記のコマンドを実行し、Homebrewをインストールします。

  ```cmd
  mkdir ~/.linuxbrew
  git clone https://github.com/Homebrew/brew ~/.linuxbrew/Homebrew
  mkdir ~/.linuxbrew/bin
  ln -s ~/.linuxbrew/Homebrew/bin/brew ~/.linuxbrew/bin
  ```

  確認

  ```cmd
  brew help
  >Example usage:
    brew search TEXT|/REGEX/
    brew info [FORMULA|CASK...]
    brew install FORMULA|CASK...
    brew update
    brew upgrade [FORMULA|CASK...]
    brew uninstall FORMULA|CASK...
    brew list [FORMULA|CASK...]
  ```

## Pythonのインストール

- 下記コマンドを実行して、依存ライブラリbinutilsのインストール

  ```cmd
  brew install binutils
  ```

- 環境変数を追加

  ```cmd
  echo 'export PATH="$HOME/.linuxbrew/opt/binutils/bin:$PATH"' >> $HOME/.bash_profile
  export LDFLAGS="-L${HOME}/.linuxbrew/opt/binutils/lib"
  export CPPFLAGS="-I${HOME}/.linuxbrew/opt/binutils/include"
  ```

- 下記コマンドを実行して、Pythonをインストール

  ```cmd
  brew install python@3.12
  ```

  ```cmd
  $HOME/.linuxbrew/bin/python3.12 -V
  >Python 3.12.0
  $HOME/.linuxbrew/bin/pip3.12 -V
  >pip 23.2.1 from $HOME/.linuxbrew/opt/python@3.12/lib/python3.12/site-packages/pip (python 3.12)
  ```

- `~/.bashrc`に`alias`を追加

  - `vim ~/.bashrc`
  - `PATH`を下記の内容に変更

  ```cmd
  alias python3='$HOME/.linuxbrew/bin/python3.12'
  alias pip3='$HOME/.linuxbrew/bin/pip3.12 -V'

  export PATH=$HOME/.linuxbrew/opt/binutils/bin:$HOME.linuxbrew/opt/python@3.12/libexec/bin/:$HOME/.linuxbrew/bin:$HOME/.linuxbrew/sbin:$HOME/opt/curl/bin:$HOME/opt/git/bin:$HOME/opt/gettext/bin:$HOME/opt/ssl/bin:$PATH
  ```

  確認

  ```cmd
  python3 -V
  >Python 3.12.0
  pip3 -V
  >pip 23.2.1 from $HOME/.linuxbrew/opt/python@3.12/lib/python3.12/site-packages/pip (python 3.12)
  ```

## neoseruko_djnago_projectsの導入

- プロジェクトのクローン

  - 下記のコマンドを実行して、プロジェクトをクローンする

  ```cmd
  cd /home/xs797670/xs797670.xsrv.jp
  mkdir project_dir
  cd project_dir
  git clone https://github.com/ANKM0/neoseruko_djnago_projects.git
  ```

- 仮想環境の作成

  - 下記のコマンドを実行して、仮想環境を作成

  ```cmd
  pwd
  >/home/xs797670/xs797670.xsrv.jp/project_dir
  python3 -m venv .venv
  source .venv/bin/activate
  ```

  - requirements.txtからモジュールをインポートする

  ```cmd
  cd neoseruko_djnago_projects/
  pip insrtall -r requirements.txt
  ```

  ```requirements.txt
  asgiref==3.7.2
  autopep8==2.0.4
  certifi==2023.7.22
  cffi==1.15.1
  charset-normalizer==3.2.0
  cryptography==41.0.3
  defusedxml==0.7.1
  Django==4.2.4
  django-admin-rangefilter==0.11.1
  django-allauth==0.55.2
  django-boost==2.1
  django-cleanup==8.0.0
  django-extensions==3.2.3
  django-hint==0.3.2
  django-widget-tweaks==1.5.0
  flake8==6.1.0
  idna==3.4
  mccabe==0.7.0
  oauthlib==3.2.2
  pycodestyle==2.11.0
  pycparser==2.21
  pyflakes==3.1.0
  PyJWT==2.8.0
  PyMySQL==1.1.0
  python3-openid==3.2.0
  requests==2.31.0
  requests-oauthlib==1.3.1
  sqlparse==0.4.4
  tomli==2.0.1
  typing_extensions==4.7.1
  tzdata==2023.3
  ua-parser==0.18.0
  urllib3==2.0.4
  user-agents==2.2.0
  ```

## DBの設定

- DBを作成する

  - DBを作成する

    - xserverサーバーパネル
    - データベース>MySQL設定
    - \>MySQL追加
    - MySQLデータベース名、文字コードを入力して、DBを作成する

  - ユーザーを作成する
    - xserverサーバーパネル
    - データベース>MySQL設定
    - \>MySQLユーザ追加
    - MySQLユーザID、パスワードを入力して、ユーザーを作成する

  - DBにユーザーを登録する
    - xserverサーバーパネル
    - データベース>MySQL設定
    - \>MySQL一覧
    - アクセス権未所有ユーザの「追加」をクリックして、ユーザーを登録する

- mysql設定

  - PyMySQLがpipに入っているか確認する

  ```cmd
  (.venv) [xs797670@sv14769 build]$ pip show PyMySQL
  Name: PyMySQL
  ```

  - `PyMySQL` を利用できるように`manage.py`を編集する

  ```python
  #!/usr/bin/env python
  """Django's command-line utility for administrative tasks."""
  import os
  import sys
  import pymysql #ここを追加

  pymysql.install_as_MySQLdb() #ここを追加


  def main():
      """Run administrative tasks."""
      os.environ.setdefault("DJANGO_SETTINGS_MODULE", "config.settings")
      try:
          from django.core.management import execute_from_command_line
      except ImportError as exc:
          raise ImportError(
              "Couldn't import Django. Are you sure it's installed and "
              "available on your PYTHONPATH environment variable? Did you "
              "forget to activate a virtual environment?"
          ) from exc
      execute_from_command_line(sys.argv)


  if __name__ == "__main__":
      main()
  ```

  - `DATABASES`の情報を設定

  ```python
  # setting.py
  DATABASES = {
      'default': {
          'ENGINE': 'django.db.backends.mysql',
          'NAME': get_secret('DB_NAME'),
          'USER': get_secret('DB_USER'),
          'PASSWORD': get_secret('DB_PASSWORD'),
          'HOST': get_secret('DB_HOST'),
          'PORT': get_secret('DB_PORT'),
          'OPTIONS': {
              'charset': 'utf8mb4',
          }
      }
  }
  ```

  ```json
  // secrets.json
  {
      "SECRET_KEY": "kjn&eav$*pdn5$v+%0js%1%=3@cdwu3s$=^4k7b1^-ahr=^6i2",
      "DB_NAME": "xs797670_neoseruko",
      "DB_USER": "xs797670_user",
      "DB_PASSWORD": "gllilwe9d9f16",
      "DB_HOST": "localhost",
      "DB_PORT": "3306"
  }
  ```

  - 接続できているか確認する

  ```cmd
  python3.12 manage.py runserver
  Watching for file changes with StatReloader
  Performing system checks...

  System check identified no issues (0 silenced).

  You have 22 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, app1, auth, contenttypes, django_boost, sessions, user.
  Run 'python manage.py migrate' to apply them.
  October 31, 2023 - 20:58:49
  Django version 4.2.4, using settings 'config.settings'
  Starting development server at http://127.0.0.1:8000/
  Quit the server with CONTROL-C.
  ```

## CGI の設定

- `.htaccess`の設定

  - `/home/xs797670/xs797670.xsrv.jp/public_html/.htaccess`

  ```.htaccess
  SetEnvIf Request_URI ".*" Ngx_Cache_NoCacheMode=off
  SetEnvIf Request_URI ".*" Ngx_Cache_AllCacheMode

  RewriteEngine On

  RewriteCond %{REQUEST_URI} !index\.cgi
  RewriteCond %{REQUEST_URI} !(^/static/)
  RewriteCond %{REQUEST_URI} !(^/media/)
  RewriteRule ^(.*)$ /project_dir/index.cgi/$1 [QSA,L]
  ```

- `index.cgi`の設定

  - `index.cgi`を作成する
  - `/home/xs797670/xs797670.xsrv.jp/public_html/project_dir/index.cgi`

  ```cmd
  cd /home/xs797670/xs797670.xsrv.jp/public_html/project_dir
  touch index.cgi
  chmod 755 index.cgi
  ```

  - `cgi`を編集する

  ```cgi
  #!/home/xs797670/xs797670.xsrv.jp/public_html/project_dir/.venv/bin/python3.12
  # coding: utf-8

  import sys, os
  import pymysql
  from wsgiref.handlers import CGIHandler
  from django.core.wsgi import get_wsgi_application

  sys.path.insert(0, "/home/xs797670/xs797670.xsrv.jp/public_html/project_dir/.venv/bin")
  sys.path.append('/home/xs797670/xs797670.xsrv.jp/public_html/project_dir/neoseruko_djnago_projects/website_with_wix')
  sys.path.append('/home/xs797670/xs797670.xsrv.jp/public_html/project_dir/neoseruko_djnago_projects/website_with_wix/config')


  os.environ.setdefault("DJANGO_SETTINGS_MODULE", "config.settings")
  os.environ['DJANGO_SETTINGS_MODULE'] = 'config.settings'

  os.environ["SERVER_NAME"] = "http://xs797670.xsrv.jp/"
  os.environ["SERVER_PORT"] = "443"
  os.environ["REQUEST_METHOD"] = "GET"
  os.environ['OPENBLAS_NUM_THREADS'] = "1"

  pymysql.install_as_MySQLdb()

  application = get_wsgi_application()
  CGIHandler().run(application)
  ```

## FCGI への対応

- FCGI を利用するために，以下のコマンドを実行し，flup ライブラリをインストールする．

  `pip install flup`

- `index.fcgi`を作成する

  - `/home/xs797670/xs797670.xsrv.jp/public_html/project_dir/index.fcgi`

  ```cmd
  #!/home/xs797670/xs797670.xsrv.jp/public_html/project_dir/.venv/bin/python3.12
  # coding: utf-8

  import sys, os
  import pymysql
  from wsgiref.handlers import CGIHandler
  from django.core.wsgi import get_wsgi_application
  from flup.server.fcgi import WSGIServer

  sys.path.insert(0, "/home/xs797670/xs797670.xsrv.jp/public_html/project_dir/.venv/bin")
  sys.path.append('/home/xs797670/xs797670.xsrv.jp/public_html/project_dir/neoseruko_djnago_projects/website_with_wix')
  sys.path.append('/home/xs797670/xs797670.xsrv.jp/public_html/project_dir/neoseruko_djnago_projects/website_with_wix/config')


  os.environ.setdefault("DJANGO_SETTINGS_MODULE", "config.settings")

  os.environ['DJANGO_SETTINGS_MODULE'] = 'config.settings'

  os.environ["SERVER_NAME"] = "xs797670.xsrv.jp"
  os.environ["SERVER_PORT"] = "8000"
  os.environ["REQUEST_METHOD"] = "GET"
  os.environ['OPENBLAS_NUM_THREADS'] = "1"

  pymysql.install_as_MySQLdb()

  application = get_wsgi_application()
  WSGIServer(application).run()
  ```

  - 権限を変更する

    `chmod 755 index.fcgi`

- `.htaccess`を編集

  ```.htaccess
  SetEnvIf Request_URI ".*" Ngx_Cache_NoCacheMode=off
  SetEnvIf Request_URI ".*" Ngx_Cache_AllCacheMode

  RewriteEngine On

  RewriteCond %{REQUEST_URI} !index\.cgi
  RewriteCond %{REQUEST_URI} !(^/static/)
  RewriteCond %{REQUEST_URI} !(^/media/)
  RewriteRule ^(.*)$ /project_dir/index.fcgi/$1 [QSA,L]
  ```

## 参考URL

### SSH接続

- [エックスサーバーにSSHで接続してみよう！](http://vdeep.net/xserver-ssh#PC)

### Git インストール

- [Xserver（エックスサーバー）にGitをインストールする](https://brainlog.jp/server/post-1132/)
- [XSERVERにgitを入れる (https使えるように)](https://couger.hatenablog.com/entry/2015/06/20/235118)
- [エックスサーバー(xserver.ne.jp)にpipenvでPython3.9をインストール](https://qiita.com/joshnn/items/244ca701c1bf9be8f350)
- [【解説】Homebrewを使ってXserverにPython＋pipenvをインストールする](https://yururi-do.com/install-python-pipenv-with-homebrew-on-xserver/)

### Vim インストール

- [Xserver に Vim をインストールする](https://ityorozu.net/itblog-xserver-vim/)
- [エックスサーバーに独自に Vim を導入する](https://laboradian.com/install-vim-on-xserver/)

### Homebrewインストール

- [【解説】Homebrewを使ってXserverにPython＋pipenvをインストールする](https://yururi-do.com/install-python-pipenv-with-homebrew-on-xserver/)

### mysql接続

- [Django】エックスサーバーで公開する](https://yunifusu.com/programming/django/django_xserver_deploy/)
- [Djangoでmysqlに接続](https://qiita.com/malvageee/items/2713d062971d528ab211)

### CGI の設定

- [Django プロジェクトを XServer で立ち上げる](https://workspacememory.hatenablog.com/entry/2021/05/07/015923)
- [Django で作ったウェブサイトをエックスサーバーで公開する方法](https://note.com/ka_disco/n/na2235b299224#8b675833-e7a4-4929-8a60-6e78fd855438)

### FCGIの設定

- [Django プロジェクトを XServer で立ち上げる](https://workspacememory.hatenablog.com/entry/2021/05/07/015923)

---
title: git clone で 'remote-https' is not a git command が発生する
topics: []
published: true
---



## 環境

- Xserver
- Linux sv14769.xserver.jp 5.4.0-164-generic #181-Ubuntu SMP Fri Sep 1 13:41:22 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux

## 問題

`git clone https://github.com/vim/vim.git`実行時に下記エラーが発生する

```cmd
git: 'remote-https' is not a git command. See 'git --help'.
```

## 原因

git-remote-httpsがビルドされていない

```cmd
ls -l ~/opt/git/libexec/git-core/git-remote-https
>ls: /home/xs797670/opt/git/libexec/git-core/git-remote-https にアクセスできません: そのようなファイルやディレクトリはありません
```

## 解決策

古い方のgitからgit-remote-httpsをコピーする

````cmd
cp /usr/libexec/git-core/git-remote-https ~/opt/git/libexec/git-core/git-remote-https
````

## 補足

`openssl->curl->gettext->git`の順にビルドして、`--with-openssl`にsslディレクトリを指定する方法でも解決できる

参考:<br>
<https://qiita.com/joshnn/items/244ca701c1bf9be8f350>

### 参考にしたWebページ

- [Xserver（エックスサーバー）にGitをインストールする](https://brainlog.jp/server/post-1132/)
- [XSERVERにgitを入れる (https使えるように)](https://couger.hatenablog.com/entry/2015/06/20/235118)
- [エックスサーバー(xserver.ne.jp)にpipenvでPython3.9をインストール](https://qiita.com/joshnn/items/244ca701c1bf9be8f350)
- [【解説】Homebrewを使ってXserverにPython＋pipenvをインストールする](https://yururi-do.com/install-python-pipenv-with-homebrew-on-xserver/)

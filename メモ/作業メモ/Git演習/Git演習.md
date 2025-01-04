# Git 演習のメモ

リポジトリを作成
ファイルを編集、追加
変更履歴を追跡する
ブランチの生成、使い方
過去のVerに遡る
比較する
ブランチをマージする
リモートリポジトリで管理する
複数人でバージョン管理する

## リポジトリを作成

- ファイルを作成する
- git init .

## ファイルを編集、追加

- ファイルを編集する
- (git statusで確認)
- git add ファイル名 でファイルをステージングエリアに移動
- git commit -m "メッセージ" でコミット(登録)を行う

## Githubにリポジトリを作成する

### ローカルでgitリポジトリを作成していない場合

- git init
- git add xxx
- git commti xxx
- git branch -M main
- git remote add origin git@github.com:xxx
- git poush -u origin main

### ローカルでgitリポジトリを作成済みの場合

- git remote add origin git@github.com:xxx でリモートブランチのURLを追加
- git branch -M main　デフォルトブランチをmainにする
- git push -u origin master

## ブランチを作成する

- git branch で現在のブランチを確認
- git branch ブランチ名 でブランチ名を作成する
- git checkout ブランチ名　でブランチに移動

git checkout -b "bnrach"

## 過去のVerに遡る

### タグを作成する

- git tag -a <tag-name> <commit-hash> -m "message"
- ex) git tag -a v0.1.0 9fceb02 -m "Initial release"
- git tag で作成したタグを確認

### タグをもとにブランチを作成

- git branch ブランチ名 タグ名
- git checkout ブランチ名
- もしくは git checkout -B ブランチ名 タグ名

## 比較する

Githubの機能を使う

### コマンドでやるなら

- git diff 変更前のSHA..変更後のSHA でコミット同士を比較する
- git diff HEAD^ で今回コミットした変更点を見る

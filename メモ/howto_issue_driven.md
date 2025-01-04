# Issues作成

new issue <br>
タイトル、内容(todo)を入力 <br>
submit new issue <br>

# ローカルでブランチを切る

## 命名規則

1. 新規開発の場合 <br>
feature/#番号 <br>
2. バグフィックスの場合 <br>
bugfix/#番号 <br>

命名規則を元にローカルでブランチを切る <br>
git checkout -b feature/#3

# コードを書く

# コミット＆プッシュ

git status <br>
git dif <br>
git add . <br>
git commit -m "コメント #番号" <br>
git push origin ブランチ名#番号

# プルリクエスト

compare & pullrequestから <br>
コミットが issue と関連付けられていることを確認した上でプルリクエストを作成 <br>

## 入力すること

## why

理由

## what

何をしたか

# マージする

マージする時コメントに "close #番号" を入れる

# ローカルのメインに反映

git checkout main <br>
git pull origin main  <br>

# ローカルのブランチを削除する

git branch -d ブランチ名#番号

# リモートのブランチを削除する

git push origin :ブランチ名#番号
(git push --delete origin ブランチ名#番号)

# 削除したリモートブランチがローカルに残ってる時

git fetch -p

# メモ

feat: 新しい機能
fix: バグの修正
docs: ドキュメントのみの変更
style: 空白、フォーマット、セミコロン追加など
refactor: 仕様に影響がないコード改善(リファクタ)
perf: パフォーマンス向上関連
test: テスト関連
chore: ビルド、補助ツール、ライブラリ関連

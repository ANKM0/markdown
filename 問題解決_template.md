---
title: 【プログラミングメモ(問題解決系)のテンプレート
topics: ["GitHubActions", "Ruby", "YAML"]
published: true
---



## 問題
Pow + rbenvでRailsを開発している場合、通常Powはプロジェクトのルートディレクトリにある.ruby-versionを参照します。

````
[your-app] cat .ruby-version
1.9.3-p392
[your-app] rbenv local 1.9.3-p429
[your-app] cat .ruby-version
1.9.3-p429
````

しかしPowを再起動しても、なぜかRailsが以前のバージョンで動きつづけたりする場合があります。

## 原因
プロジェクトの上位ディレクトリに別の.ruby-versionがあると、Powがそのバージョンを参照してしまうようです。

## 解決策
プロジェクトの上位ディレクトリに一つずつ移動して、.ruby-versionがないか確認します。
上位ディレクトリに不要な.ruby-versionがあった場合は削除します。

````
[your-app] cd ..
[projects] cat .ruby-version
1.9.3-p392
[projects] rm .ruby-version
````

### 確認時のPowのバージョン
- 0.4.0

### Railsで使用中のRubyのバージョンを確認する
Railsの内部から使用中のRubyのバージョンを確認する場合は以下の定数が使えます。
怪しい場合はlogger等で確認してみましょう。

````ruby
RUBY_VERSION # => "1.9.3"
RUBY_PATCHLEVEL # => 429
````

### 参考にしたWebページ
- この現象はPowの[Issue](https://github.com/37signals/pow/issues/363)に挙げられています。

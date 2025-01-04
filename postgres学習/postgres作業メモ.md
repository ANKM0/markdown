# postgresの作業メモ

## 接続時のコマンド

password: e9d9f16

<!-- C:\Program Files\PostgreSQL\17\bin -->
cd '.\Program Files\PostgreSQL\17\bin\'
 .\psql.exe -U postgres -p 5433

## 作成したテーブル(shop)に接続

 .\psql.exe -U postgres -p 5433 -d shop

## 1章

### テーブル作成

CREATE TABLE Shoin
 (shoin_id CHAR(4) NOT NULL PRIMARY KEY,
 shoin_mei VARCHAR(100) NOT NULL,
 shoin_bunrui VARCHAR(32) NOT NULL,
 hanbai_tanka INTEGER ,
 siire_tanka INTEGER ,
 torokubi DATE);

### テーブル定義の変更

ALTER TABLE shoin ADD COLUMN shoin_mai_kana VARCHAR(100);

<!-- 削除する場合、実行していない -->
ALTER TABLE DROP shoin ADD COLUMN shoin_mai_kana VARCHAR(100);

### データ登録

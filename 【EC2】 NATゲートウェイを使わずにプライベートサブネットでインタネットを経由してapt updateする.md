---
title: 【EC2】 NATゲートウェイを使わずにプライベートサブネットでインタネットを経由してapt updateする
topics: ["GitHubActions", "Ruby", "YAML"]
published: true
---

# aa

# 結論

ポートフォワーディングの設定をして、proxy経由で通信するように設定する

# 前提

以下のような環境を想定しています

![Alt text](https://i.gyazo.com/36029254292ec7a30665d02e187082db.png)

プライベートサブネットとパブリックサブネットに一つずつインスタンスがある構成です

パブリックサブネットにはElasticIPが紐づけられています

# プライベートサブネットでインタネットを経由してapt updateを行う方法

プライベートサブネットからapt updateするには、

1. ポートフォワーディングの設定をした後で
2. apt がproxy経由で通信できるように設定します

## ポートフォワーディングの設定

プライベートサブネットからインタネットに通信できるようにするためにポートフォワーディングの設定を行います

- PC->パブリック　(パブリックインスタンスにElasticIPが紐づけられているため)
- パブリック->プライベート　(プライベートインスタンスはパブリックインスタンスからssh接続できる仕様)

2つがssh接続できているので

```PC->パブリック->(転送)->パブリック->プライベート```のようにポートフォワーディングを行うことでプライベートサブネットからインタネットに通信できるようになります

### ポートフォワーディングの手順

1. 公開鍵をパブリックインスタンスにコピーします

    ```scp -i "C:\Users\xings\.ssh\keys\aws_ax1_project.pem" -r "C:\Users\xings\.ssh\keys\aws_ax1_project.pem" ubuntu@52.194.215.187:/home/ubuntu/.ssh```
2. パブリックインスタンスにssh接続します

    ```ssh -i "C:\Users\xings\.ssh\keys\aws_ax1_project.pem" ubuntu@52.194.215.187```
3. 公開鍵の権限を変更します

    ```chmod 600 ~/.ssh/aws_ax1_project.pem```
4. パブリックインスタンスからパブリックインスタンスにssh接続します

    -Dオプションを使うことでパブリックインスタンスの1080番ポートを、リスニングポートとして有効にしています

    ```ssh localhost -D 1080 -i ~/.ssh/aws_ax1_project.pem```
5. パブリックインスタンスからプライベートインスタンスにssh接続します

    -Rオプションを使うことでプライベートインスタンスの2080番ポートをパブリックインスタンスの1080番ポートに転送しています

    ```ssh -R 2080:localhost:1080 ubuntu@10.0.131.114 -i ~/.ssh/aws_ax1_project.pem```
6. 確認用コマンドを実行して、成功しているかテストします

    ```curl http://ifconfig.io --proxy 'socks5h://localhost:2080'```

    IPアドレスが出力されていればokです

# apt がproxy経由で通信できるように設定

通信できるようになったのでapt がproxy経由で通信できるように設定します

具体的には

sudo vim /etc/apt/apt.conf
Acquire::http::Proxy "socks5h://127.0.0.1:2080";
Acquire::https::Proxy "socks5h://127.0.0.1:2080";
Acquire::socks::Proxy "socks5h://127.0.0.1:2080";

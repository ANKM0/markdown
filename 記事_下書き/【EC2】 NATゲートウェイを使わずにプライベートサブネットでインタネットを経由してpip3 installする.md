---
title: EC2】 NATゲートウェイを使わずにプライベートサブネットでインタネットを経由してpip3 installする
topics: ["GitHubActions", "Ruby", "YAML"]
published: true
---

# 概要

プライベートサブネットでモジュールを入れようとして、pip3 install <xxx> とすると、以下のようなエラーが出ると思います

``` cmd
WARNING: Retrying (Retry(total=4, connect=None, read=None, redirect=None, status=None)) after connection broken by 'NewConnectionError('<pip._vendor.urllib3.connection.HTTPSConnection object at 0x7fa895d65890>: Failed to establish a new connection: [Errno 101] Network is unreachable')': /simple/pandas/
```

これはプライベートサブネットがインターネットに繋がっていない(仕様)ことが原因です

NATゲートウェイを使うことで解決できるのですが、いかんせん高いので使いたくない

そこで、NATゲートウェイを使わずにプライベートサブネットでpip3 installする方法を紹介します

# 結論

ポートフォワーディングして、proxychains-ngを導入することで解決します

![Alt text](https://i.gyazo.com/d03b6fb7ae47523cf7b42530b4e7792f.png)

# 前提

以下のような環境を想定しています

![Alt text](https://i.gyazo.com/36029254292ec7a30665d02e187082db.png)

プライベートサブネットとパブリックサブネットに一つずつインスタンスがある構成です

パブリックサブネットにはElasticIPが紐づけられています

# プライベートサブネットでインタネットを経由してpip3 installを行う方法

プライベートサブネットでインタネットを経由してpip3 installを行うには

1. ポートフォワーディングの設定をした後で
2. proxychainsでproxy経由で通信するように設定します

## ポートフォワーディングの設定

プライベートサブネットからインタネットに通信できるようにするにはポートフォワーディングを行います

今、次の2つがssh接続できるようになっています

- PC->パブリック　(パブリックインスタンスにElasticIPが紐づけられているため)
- パブリック->プライベート　(プライベートインスタンスはパブリックインスタンスからssh接続できる仕様)

```PC->パブリック->(転送)->パブリック->プライベート```のようにポートフォワーディングを行うことでプライベートサブネットからインタネットに通信できるようになります

### ポートフォワーディングの手順

1. 公開鍵をパブリックインスタンスにコピーします

    ```scp -i "C:\Users\xings\.ssh\keys\aws_ax1_project.pem" -r "C:\Users\xings\.ssh\keys\aws_ax1_project.pem" ec2-user@43.207.191.40:/home/ec2-user/.ssh```

1. pcからパブリックインスタンスにssh接続を行いログインします

    ```ssh -i "C:\Users\xings\.ssh\keys\aws_ax1_project.pem" ec2-user@43.207.191.40```

1. 公開鍵の権限を変更します

    ```chmod 600 ~/.ssh/aws_ax1_project.pem```

1. パブリックインスタンスからパブリックインスタンスにssh接続します

   -Dオプションを使うことでパブリックインスタンスの1080番ポートを、リスニングポートとして有効にしています

    ```ssh localhost -D 1080 -i ~/.ssh/aws_ax1_project.pem```

1. パブリックインスタンスからプライベートインスタンスにssh接続します

    -Rオプションを使うことでプライベートインスタンスの2080番ポートをパブリックインスタンスの1080番ポートに転送しています

    ```ssh -R 2080:localhost:1080 ec2-user@10.0.139.136 -i ~/.ssh/aws_ax1_project.pem```

1. 確認用コマンドを実行して、成功しているかテストします

    ```curl http://ifconfig.io --proxy 'socks5h://localhost:2080'```

    IPアドレスが出力されていればokです

## proxychains-ngの設定

```socks5://127.0.0.1:2080```を経由することで外部と通信できるようになりました

ただ、コマンド実行時に一々proxyを指定するのは面倒です

そこで、proxychains-ngを導入して、ネットワークを利用するコマンドの通信がSOCKS/HTTPプロキシを経由するように設定します

### proxychains-ngの手順

※プライベートサブネットにログインした状態で作業します

1. 以下のコマンドを実行します(gccなどコンパイラーが必要です)

    ```cmd
    cd /opt
    sudo curl --proxy socks5://127.0.0.1:2080 -O http://ftp.barfooze.de/pub/sabotage/tarballs/proxychains-ng-4.16.tar.xz
    sudo tar Jxvf ./proxychains-ng-4.16.tar.xz
    cd proxychains-ng-4.16
    sudo ./configure --prefix=/usr --sysconfdir=/etc
    make
    sudo make install
    ```

2. ```sudo vim /etc/proxychains.conf```コマンドで/etc/proxychains.confを編集します

   以下の内容を書き込みます(:wqで保存)

    ```cmd
    [ProxyList]
    socks5 127.0.0.1 2080
    ```

3. ```sudo proxychains4 bash```コマンドを実行します
4. ```curl http://ifconfig.io```コマンドを実行して上手くできているかテストします

   IPアドレスが出力されていればokです

## 補足 次から接続する時の手順

この記事の設定をした後にプライベートサブネットに接続するときは以下の手順を実行する必要があります

1. ```ssh localhost -D 1080 -i ~/.ssh/aws_ax1_project.pem```
2. ```ssh -R 2080:localhost:1080 ec2-user@10.0.139.136 -i ~/.ssh/aws_ax1_project.pem```
3. ```sudo proxychains4 bash```

# 参考文献

- [LANの内壁を抜けて外へ通信させる ～SSHとproxychains-ng～](https://remoteroom.jp/diary/2021-02-07/)
- [プライベートサブネット内のLinuxでオンラインパッケージアップデートする方法を考えてみた](https://blog.serverworks.co.jp/tech/2016/03/08/ssh-portforward/)
- [sshポートフォワーディングについて図示してみる](https://qiita.com/fkshom/items/844a1be031ae3eb4bcea)
- [NATゲートウェイを使わずにプライベートサブネットをインターネットにつなぐ](https://qiita.com/SSMU3/items/5ed5792e74266b54ff8b)

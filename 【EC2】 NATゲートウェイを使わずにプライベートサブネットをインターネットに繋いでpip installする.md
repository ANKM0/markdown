---
title: 【EC2】 NATゲートウェイを使わずにプライベートサブネットをインターネットに繋いでpip installする
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

![Alt text](https://i.gyazo.com/312c8778b915c23434459afca51a4edb.png)

プライベートサブネットとパブリックサブネットに一つずつインスタンスがある構成です

パブリックサブネットにはElasticIPが紐づけられています

# ポートフォワーディングの設定

パブリックインスタンスにElasticIPが紐づけられているので、PCから、パブリックインスタンスへのssh接続ができる状態です

また、プライベートインスタンスはパブリックインスタンスからssh接続できる仕様です

つまり、

- PC->パブリック
- パブリック->プライベート

の2つがsshできているので、ポートフォワーディングを使うことで

PC->パブリック->プライベートと接続できます




操作端末から、Instance-publicにssh接続が可能であること
Instance-publicからInstance-privateへssh接続が可能であること


jumpにログイン
ssh -i "C:\Users\xings\.ssh\keys\aws_ax1_project.pem" ec2-user@43.207.191.40

targetにログイン
ssh -o ProxyCommand='ssh -i "C:\Users\xings\.ssh\keys\aws_ax1_project.pem" -W %h:%p ec2-user@43.207.191.40' -i "C:\Users\xings\.ssh\keys\aws_ax1_project.pem" ec2-user@10.0.139.136
ssh -L8888:192.168.33.10:80 username@example.jp









curl https://example.com/
sudo curl --proxy socks5://127.0.0.1:22 https://example.com/
# まとめ

フォームをテーブルの中に入れるには、Formsetとget_context_dataメソッドを使う

# 参考文献

- [Formsets | Django documentation | Django](https://docs.djangoproject.com/en/4.1/topics/forms/formsets/)
- [Creating forms from models | Django documentation | Django](https://docs.djangoproject.com/en/4.1/topics/forms/modelforms/#model-formsets:~:text=%22birth_date%22%2C)
- [Django Formsetを使ってFormをたくさん並べよう!](https://qiita.com/qtatsunishiura/items/a6cc11e025aca1c16ed1)



プライベートサブネットでyum updateしたい

->[プライベートサブネットをインターネットGWにつなぎ、ネットワークACLで制御](https://qiita.com/SSMU3/items/5ed5792e74266b54ff8b)


出口が常に空いているのでDBが置かれるサブネットには使いたくない...



![Alt text](https://i.gyazo.com/91894cd2f11952b8106ee12441d214d4.png)

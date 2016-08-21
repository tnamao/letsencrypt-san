# let's encrypt 証明書を取得する ansible playbook

## 概要

リモートホスト上で nginx を起動し、let's encrypt のサーバ証明書を取得するための playbook です。

## 必要要件

* リモートホスト
  * OS: Ubuntu 16.04 (xenial)

* ローカルマシン (ansible 実行環境)
  * python 2.7+
  * ansible 2.1.0.0+


## Howto

### 事前準備

* グローバルに公開されたサーバを用意する
* サーバの IP アドレスをサーバ証明書で使用するドメイン名で DNS 登録する
* グローバルサーバは root で ssh ログイン出来るようにする


### 設定ファイルの変更

* inventory/myhost

```
[webserver]
your.hostname.or.ipaddress <== nginx を起動するサーバ

[webserver:vars]
ansible_python_interpreter=/usr/bin/python2.7
```

* var/commn.yml

```
---
cert_email: admin@your.domain <== ホスト管理者のメールアドレス
cert_hostname:
  - your.common.name <== 証明書のコモンネームに相当するドメイン名
  - alias.common.name <== 2つ目以降は SAN に指定されるコモンネームの代替名、不要なら指定しない

letsencrypt_root: /var/www/letsencrypt_root
certbot_path: /opt/certbot
shellscript_path: /opt/shellscripts
```

### ansible-playbook の実行

最初に nginx をインストール設定します

    $ ansible-playbook -i inventory/myhost provision.yml

このコマンドが実行された時点で、ブラウザで対象のドメインにアクセス可能か確認してください。

続いて、let's encrypt の証明書を取得し、nginx を再読込します。

    $ ansible-playbook -i inventory/myhost certificate.yml

このコマンドが終わったら、 https で対象のドメインにアクセスが可能になります。


### 証明書更新ジョブの登録

crontab に更新処理を登録します。証明書の更新は2ヶ月に1回、自動的に実行します。

    $ ansible-playbook -i inventory/myhost cron.yml


### root 以外でリモートホストにアクセスしたい場合

root で ssh ログインさせたい場合は、アカウントに sudo 権限を付けて

* provision.yml
* certificate.yml
* cron.yml

上記のファイルを以下の行を探します

```
  remote_user: root
```

上記の行を以下のように書き換えします

```
  remote_user: work-account <== ssh ログインするアカウント名
  become: true <== sudo を使う設定
```

もし、sudo 権限にパスワードが必要な場合は、以下のように ansible-playbook コマンドの --ask-sudo-pass オプションを追加します

    $ ansible-playbook -i inventory/myhost provision.yml --ask-sudo-pass

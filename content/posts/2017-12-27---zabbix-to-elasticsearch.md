---
title: ZabbixとElasticsearchを連携してみた！
date: "2017-12-27T20:30:00+09:00"
template: "post"
draft: false
slag: "zabbix-to-elasticsearch"
category: "elasticsearch"
tags:
    - "Elasticsearch"
    - "Zabbix"
    - "Tech"
description: "割と最近Elasticsearchを使ったストリームデータ解析が流行っているみたいなので、うちでもできないかなと思って私が仕事で使っているPCにZabbixを導入し、そのデータをElasticsearchに投入してみました"
---
　割と最近Elasticsearchを使ったストリームデータ解析が流行っているみたいなので、うちでもできないかなと思って私が仕事で使っているPCにZabbixを導入し、そのデータをElasticsearchに投入してみました。
今回は、私のPCにZabbixを導入する過程を記します。(Elasticsearchやlogstashの導入は別途書こうかなと)
導入方法は以下のリンクを参考にさせていただきました。
(というかほとんどやってることは変わりませんm(_ _)m)
それでは早速いきます。

[[Zabbix 3.2][Ubuntu 16.04] Zabbix環境構築手順まとめ](https://qiita.com/koara-local/items/26475b321030140d2048)

## 導入環境

- OS : Ubuntu 16.04 LTS / 64bit
- Zabbix : 3.2
- PostgreSQL : 9.5

## 日本語化対応
ZabbixのWebUIの日本語化の際に、localeの選択ができるようになっている必要があります。

`$ sudo apt-get install language-pack-ja`

## Zabbixのリポジトリ設定
Ubuntu 16.04のデフォルトでは古いバージョンのZabbixをインストールしてしまうので、3.2をインストールするための設定

[1 Repository installation [Zabbix Documentation 3.2]](https://www.zabbix.com/documentation/3.2/manual/installation/install_from_packages/repository_installation#for_ubuntu)

`$ wget http://repo.zabbix.com/zabbix/3.2/ubuntu/pool/main/z/zabbix-release/zabbix-release_3.2-1+xenial_all.deb`

`$ sudo dpkg -i zabbix-release_3.2-1+xenial_all.deb`

`$ sudo apt-get update`

## Zabbix(PostgreSQL)向けパッケージのインストール
データの保存先として、MySQLまたはPostgreSQLを選択できます。

[2 Server installation with MySQL database [Zabbix Documentation 3.2]](https://www.zabbix.com/documentation/3.2/manual/installation/install_from_packages/server_installation_with_mysql#debianubuntu)

[3 Server installation with PostgreSQL database [Zabbix Documentation 3.2]](https://www.zabbix.com/documentation/3.2/manual/installation/install_from_packages/server_installation_with_postgresql#debianubuntu)

本記事では3のPostgreSQLを利用した環境を作成します。

`$ sudo apt-get install zabbix-server-pgsql zabbix-frontend-php`

Zabbixが稼働しているサーバーも監視対象にするので、zabbix-agentもインストールします。

`$ sudo apt-get install zabbix-agent`

## 依存パッケージのインストール
後の手順でパッケージが不足していると言われたので、以下をインストールします。

`$ sudo apt-get install php7.0-bcmath php7.0-xml php7.0-mbstring php7.0-pgsql fonts-vlgothic`

php7.0-pgsql => これがないと初期設定でPostgreSQLが認識されない。

fonts-vlgothic => 日本語フォント。グラフのレンダリング時に日本語を使用する場合に必要。

## Zabbix用のDBの作成
以下は作成例です。

`$ sudo sed -i -e '/^local.*all.*all.*/s/peer/trust/' /etc/postgresql/9.5/main/pg_hba.conf`

`$ sudo -u postgres createuser -a -U postgres -P zabbix`

`Enter password for new role:`

`Enter it again:`

`$ sudo -u postgres psql`

`postgres=# create database zabbix owner zabbix encoding UTF8;`

`postgres=# \q`

## 作成したDBの初期化
DBの作成後、作成したDBをZabbix用に初期化します。

`$ zcat /usr/share/doc/zabbix-server-pgsql/create.sql.gz | psql -U postgres zabbix`

## /etc/zabbix/zabbix_server.confの変更
作成したDBに合わせてzabbix-server設定ファイルを修正します。

`$ vi /etc/zabbix/zabbix_server.conf`

`DBHost=localhosts`

`DBName=zabbix`

`DBUser=zabbix`

`DBPassword=<zabbix db user password>`

`DBPort=5432`

DBName, DBUser, DBPasswordは前段で作成したDBの設定に合わせて設定して下さい。

DBPortの5432はPostgreSQLのデフォルト値になります。

こちらの設定はZabbixのWebからの初期設定時にデフォルト値にもなりますが、Web画面で設定する内容はあくまでも zabbix-frontend 向けの設定のため、zabbix_server.conf 自体もちゃんと設定しておかないと後々エラーになりますので必ず設定して下さい。

## Zabbixサービスの起動
systemdからzabbix-serverを起動します。

`$ sudo service zabbix-server start`

`$ sudo update-rc.d zabbix-server enable`

## /etc/zabbix/apach.confのdate.timezoneを変更する
date.timezoneの箇所のみコメントアウトしてありますので、環境に合わせて変更します。

基本的には Asia/Tokyo にするかとは思うんですが、その他にする必要がある場合の設定値は以下で確認できます。

[PHP: List of Supported Timezones - Manual](http://php.net/manual/en/timezones.php)

```
<IfModule mod_php5.c>
    php_value max_execution_time 300
    php_value memory_limit 128M
    php_value post_max_size 16M
    php_value upload_max_filesize 2M
    php_value max_input_time 300
    php_value always_populate_raw_post_data -1
    php_value date.timezone Asia/Tokyo
</IfModule>
<IfModule mod_php7.c>
    php_value max_execution_time 300
    php_value memory_limit 128M
    php_value post_max_size 16M
    php_value upload_max_filesize 2M
    php_value max_input_time 300
    php_value always_populate_raw_post_data -1
    php_value date.timezone Asia/Tokyo
</IfModule>
```

## apacheのサービスの再起動
`$ sudo service apache2 restart`

## サービスの起動確認
ブラウザからhttp://<serverのip>/zabbixにアクセス。

![zabbix1.png](/zabbix/zabbix1.png)

残りの手順は以下のページを参照

[3 Installation from sources [Zabbix Documentation 3.2]](https://www.zabbix.com/documentation/3.2/manual/installation/install#installing_frontend)

初期パスワードでログインします。
- Username : Admin
- Password : zabbix

## WebUIの日本語化
WebUIを日本語化する場合は以下のようにします。

Administration => Users => Admin => LanguageでJapanese(ja_JP)を選択

![zabbix2.png](/zabbix/zabbix2.png)

## Zabbix Agentの設定
Zabbixが稼働しているサーバー自体を監視対象にする。

基本的には zabbix-agent をインストール後、zabbix-serverと同様にserviceを立ち上げれば良いです。

初期状態だとZabbix Serverからのアクセスが無効化されているため、有効化する必要があります。

Configuration => Hosts => 「Zabbix server」のStatusがDisabledになっているのでクリックして有効化

## 次回
次回はPostgreSQLに格納されているデータをlogstash経由でElasticsearchに持っていき、Kibanaで可視化までを書こうかなと思います。

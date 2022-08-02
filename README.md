## 参考URL
https://codelabo.com/posts/20201010192152

## 事前準備

自分の端末の公開鍵をgithubの個人設定に登録してあること。

## 環境構築手順

1.不要なdockerイメージ,ボリューム,コンテナを削除する。

2.本レポジトリをクロンする。

```
$ git clone git@github.com:naritomo08/railsmysql.git railsmysql
$ cd railsmysql
$ git config --local user.name "naritomo"
$ git config --local user.email naritomo08@gmail.com
```

2.1 railsアプリの新規作成からする場合

```bash
$ mkdir src
$ cp Gemfile src/
$ cp Gemfile.lock src/
```
手順3以降を実施する。

2.2　railsアプリがすでにある場合

```bash
$ git clone git@github.com:naritomo08/railsmysqlapp.git src
$ docker-compose build
$ cd src
$ git config --local user.name "naritomo"
$ git config --local user.email naritomo08@gmail.com

```

手順9に飛んでサービスが立ち上がるか確認する。

3.rails newコマンドをrailmapp上で実行

```
$ docker-compose build
$ docker-compose run railmapp rails new . --force --no-deps --database=mysql --skip-test --webpacker
```

4.railsのディレクトリができているかチェック

```
$ ls -l
```

5.所有者がrootになっているファイルの所有者を現在のユーザに書き換え
*状況によってはいらない。

```
$ sudo chown -R $USER:$USER .
```

6.rails new で新しいGemfileができたので再ビルド

```
$ docker-compose build
```

7.webpackerのインストール

```
$ docker-compose run railmapp rails webpacker:install
```

8.DBの設定を変更

```
$ vi src/config/database.yml

default: &default
  adapter: mysql2
  encoding: utf8mb4
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  username: <%= ENV.fetch("MYSQL_USERNAME", "root") %>
  password: <%= ENV.fetch("MYSQL_PASSWORD", "password") %>
  host: <%= ENV.fetch("MYSQL_HOST", "mysqldb") %>

development:
  <<: *default
  database: app_development

test:
  <<: *default
  database: app_test

production:
  <<: *default
  database: app_production
  username: app
  password: <%= ENV['APP_DATABASE_PASSWORD'] %>
```

9.コンテナ立ち上げ

```
$ docker-compose up
途中で立ち上がらないなどのエラーが出ないこと。
```

10.別タブを開き下記のコマンドを実行してDBを作成

```
$ docker exec -ti railsmysql_railmapp_1 bash
$ rake db:create
```

本手順でアプリ新規作り直しから実施した場合、
すべてのコンソールを立ち上げなおし、
すべてのコンテナを削除後立ち上げなおすこと。

既存のアプリの場合、すべてのコンソールを立ち上げなおし
下記手順を使用し立ち上げなおしを実施すること。

## ログインURL

```
1. Rubyサイト
http://localhost:3000

2. adminer

http://127.0.0.1:8081


* ログイン情報
  - データベース種類: MySQL
  - サーバ: mysqldb
  - ユーザ名: root
  - パスワード:password

3. mailhog

http://127.0.0.1:8025

```

## コンテナ起動

```
docker-compose up -d
```

## コンテナ停止

```
docker-compose stop
```

## コンテナ削除

```
docker-compose down
```

## 起動中のコンテナに入る

1. appコンテナ

```bash
$ docker-compose exec railmapp bash
```

2. DBコンテナ

```bash
$ docker-compose exec mysqldb bash
```

## Gemfileを更新した場合

以下のコマンドでコンテナ再ビルド/作り直しを行うこと。

```bash
docker-compose down
docker-compose build
docker-compose up -d
```

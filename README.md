## 参考URL
[Windowsで Docker を用いて Rails 6.0 + MySQL の環境構築](https://codelabo.com/posts/20201010192152)

[Elixirをdocker環境で立ち上げてみる。](https://qiita.com/naritomo08/items/fecf4ace7b9ca9078102)

## 事前準備

windows11+wsl2+Ubuntu22+DockerCompose+vscodeでの環境を構築してること。

## 環境構築手順

### 不要なdockerイメージ,ボリューム,コンテナを削除する。

2.本レポジトリをクロンする。

```bash
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

```bash
$ docker-compose build
$ docker-compose run railmapp rails new . --force --no-deps --database=mysql --skip-test --webpacker
```

4.railsのディレクトリができているかチェック

```bash
$ ls -l
```

5.所有者がrootになっているファイルの所有者を現在のユーザに書き換え
*状況によってはいらない。

```bash
$ sudo chown -R $USER:$USER .
```

6.rails new で新しいGemfileができたので再ビルド

```bash
$ docker-compose build
```

7.webpackerのインストール

```bash
$ docker-compose run railmapp rails webpacker:install
```

8.DBの設定を変更

```bash
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

```bash
$ docker-compose up
途中で立ち上がらないなどのエラーが出ないこと。
```

10.別タブを開き下記のコマンドを実行してDBを作成

```bash
$ docker exec -ti railsmysql_railmapp_1 bash
$ rake db:create
```

本手順でアプリ新規作り直しから実施した場合、
すべてのコンソールを立ち上げなおし、
すべてのコンテナを削除後立ち上げなおすこと。

既存のアプリの場合、すべてのコンソールを立ち上げなおし
下記手順を使用し立ち上げなおしを実施すること。

## ログインURL

### Rubyサイト

http://localhost:3000

### adminer

http://127.0.0.1:8081


* ログイン情報
  - データベース種類: MySQL
  - サーバ: mysqldb
  - ユーザ名: root
  - パスワード:password

### mailhog

http://127.0.0.1:8025

## コンテナ起動

```bash
docker-compose up -d
```

## コンテナ停止

```bash
docker-compose stop
```

## コンテナ削除

```bash
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

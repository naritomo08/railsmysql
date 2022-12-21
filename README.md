## 参考URL
[Windowsで Docker を用いて Rails 6.0 + MySQL の環境構築](https://codelabo.com/posts/20201010192152)

[Elixirをdocker環境で立ち上げてみる。](https://qiita.com/naritomo08/items/fecf4ace7b9ca9078102)

## 事前準備

windows11+wsl2+Ubuntu22+DockerCompose+vscodeでの環境を構築してること。

## 環境構築手順

### 不要なdockerイメージ,ボリューム,コンテナを削除する。

### 本リポジトリをクローンする。

```bash
$ git clone -b rails7 https://github.com/naritomo08/railsmysql.git railsmysql
$ cd railsmysql
```

後にファイル編集などをして、git通知が煩わしいときは
作成したフォルダで以下のコマンドを入れる。

```bash
 rm -rf .git
```

### railsアプリの新規作成準備

```bash
$ mkdir src
$ cp Gemfile src/
$ cp Gemfile.lock src/
```

### rails newコマンドをrailmapp上で実行

```bash
$ docker-compose build
$ docker-compose run --no-deps railmapp rails new . --webpack --force --database=mysql
```

### railsのディレクトリができているかチェック

```bash
$ ls -l src
→複数のファイル/フォルダができていること。
```

### 所有者がrootになっているファイルの所有者を現在のユーザに書き換え

```bash
$ sudo chown -R $USER:$USER .
```

### rails new で新しいGemfileができたので再ビルド

```bash
$ docker-compose build
```

### DBの設定を変更

```bash
$ vi src/config/database.yml

以下の内容に上書きする。

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

### コンテナ立ち上げ

```bash
$ docker-compose up -d
立ち上がっていないコンテナは不要なので消しておくこと。
コンテナ確認/削除方法は割愛。
```

### 下記のコマンドを実行してDBを作成

```bash
$ docker-compose exec railmapp bash
$ rake db:create
```

## ログインURL

### Rubyサイト

http://localhost:3000

### adminer(DB管理ツール)

http://127.0.0.1:8081


* ログイン情報
  - データベース種類: MySQL
  - サーバ: mysqldb
  - ユーザ名: root
  - パスワード:password

### mailhog(メールサーバ)

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

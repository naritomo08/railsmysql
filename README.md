## 参考URL
https://codelabo.com/posts/20201010192152

## 事前準備

自分の端末の公開鍵をgithubの個人設定に登録してあること。

## 環境構築手順

1.本レポジトリをクロンする。
```
$ git clone git@github.com:naritomo08/railsmysql.git railsmysql
$ cd railsmysql
```

2.関連するdockerイメージ,コンテナを削除する。

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
```

10.別タブを開き下記のコマンドを実行してDBを作成
```
$ docker-compose run railmapp rake db:create
```

## ログインURL

```
http://localhost:3000
```

## 以降の起動・停止

```
docker-compose up -d
docker-compose down
```
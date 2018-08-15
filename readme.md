`Heroku` へ `Laravel 5.6` を deploy したサンプルコード

## 前提条件

- Heroku アカウント取得済み
- [heroku toolbelt](https://devcenter.heroku.com/articles/heroku-cli) を導入済み
- Composer を導入済み
- git 導入済み

## composer で Laravel Project を作成

```bash:create-project
$ composer create-project laravel/laravel
Installing laravel/laravel (v5.6.33)
  - Installing laravel/laravel (v5.6.33): Downloading (100%)
Created project in /Users/tabe/temp/laravel
> @php -r "file_exists('.env') || copy('.env.example', '.env');"
Loading composer repositories with package information
Updating dependencies (including require-dev)
(中略)
Writing lock file
Generating optimized autoload files
> Illuminate\Foundation\ComposerScripts::postAutoloadDump
> @php artisan package:discover
Discovered Package: fideloper/proxy
Discovered Package: laravel/tinker
Discovered Package: nunomaduro/collision
Package manifest generated successfully.
> @php artisan key:generate
Application key [base64:*******] set successfully.
```

## Procfile を準備
Heroku の php標準では、apache2 が起動するだけなので、ディレクトリを指定して起動させるために、`Procfile` を準備する。

```bash:Procfile
$ cd laravel
$ ls
app             composer.json   database        public          routes          tests
artisan         composer.lock   package.json    readme.md       server.php      vendor
bootstrap       config          phpunit.xml     resources       storage         webpack.mix.js
$ echo 'web: vendor/bin/heroku-php-apache2 public' >> Procfile
```

## git 初期化

```bash:git-init
$ git init
Initialized empty Git repository in /****/laravel/.git/
```

## .gitignore 作成

```bash:gitignore
$ gibo dump Composer Laravel >> .gitignore
$ echo .env.example >> .gitignore
```

## config/database.php を修正して、PostgreSQL への接続を default とする。

ファイルの先頭で、Heroku で定義される環境変数を読み込むようにしておく
```bash:config/database.php
$url = parse_url(getenv("DATABASE_URL"));
```

`return [` 直後にある `default` を `PostgreSQL` 向けに変更しておく。
```bash:config/database.php
    'default' => env('DB_CONNECTION', 'pgsql'),
```

`'pgsql'` 項目を、`DATABASE_URL` をパースしてきた内容から取得できるよう、変更する。
```bash:config/database.php
        'pgsql' => [
            'driver'   => 'pgsql',
            'host'     => $url["host"],
            'database' => substr($url["path"], 1),
            'username' => $url['user'],
            'password' => $url['pass'],
            'charset'  => 'utf8',
            'prefix'   => '',
            'schema'   => 'public',
        ],
```

ソースをここまで修正すれば、最低限、Heroku 上で動作させることは可能です。

他、キャッシュやセッションを利用する場合には、`Redis` や `memcachier` へ保持するように変更する必要もあります。

## Heroku App の準備

実際に Heroku 上で実行させるには、Herokuアプリを準備した上で、`APP_KEY` 環境変数の定義が必要です。
まずは、Herokuアプリを作ります。

```bash:heroku-create
$ heroku create
Creating app... done, ⬢ afternoon-shelf-58335
https://afternoon-shelf-58335.herokuapp.com/ | https://git.heroku.com/afternoon-shelf-58335.git
```

Heroku Postgres も必要です。次のコマンドか、Heroku ダッシュボードから導入ください。

```bash:heroku
heroku addons:create heroku-postgresql
```

## APP_KEY を定義

次に、`heroku config:set` コマンドを使って、環境変数を設定します。
```bash:config
$ heroku config:set APP_KEY=$(php artisan --no-ansi key:generate --show)
Setting APP_KEY and restarting ⬢ sheltered-lake-94045... done, v9
APP_KEY: base64:NlWvRWrMlKFwA+EkX6fuE+njWyMIR7jwwe9uRP34LKw=
```

## 起動確認

手動で、Heroku へソースコードをデプロイする場合には、`git push heroku master` でどうぞ。

ここまで準備できれば、次のコマンドでアプリケーションを開けます。
```bash:open
$ heroku open
```

## # Heroku Button

次のボタンをクリックすれば、git を介さず、直接 Heroku へデプロイ可能です。

[![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy)


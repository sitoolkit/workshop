---
title: Docker入門ハンズオン
---

# Docker入門

# ハンズオン

---

### 目次

- はじめに
- 環境の準備
- Dockerの構成要素
- Dockerの基本操作
- コンテナの用途
- イメージ・composeの作成
- まとめ

---

### はじめに

**このワークショップでは**

Dockerを使用したことがない方へ、Dockerの基本的な使い方から、
Dockerアプリケーションの作成方法まで、ハンズオン形式で解説します。

**前提知識**

Windows 又は MacでCLI操作ができること

---

## 環境の準備

---

### 環境の準備

ハンズオンでは以下のソフトウェアが必要です。

* Docker
 * Windows  
   [Docker Desktop for Windows](https://docs.docker.com/docker-for-windows/install/)  
 * Mac  
   [Docker Desktop for Mac](https://docs.docker.com/docker-for-mac/install/)  

* 高機能エディタ
 * [Visual Studio Code](https://code.visualstudio.com/) 等

---

## Dockerの構成要素

---

### コンテナを動かす

コマンドプロンプト(Windows) / ターミナル(macOS)で以下のコマンドを実行してください。

```sh
docker run -d --name red -p 3000:3000 redmine
```

コマンドが終了したら、ブラウザで<a href="http://localhost:3000" target="redmine">http://localhost:3000</a>にアクセスしてください。
Redmineのトップページが表示されることを確認します。

---

### コンテナの中に入る

以下のコマンドを実行してください。

```sh
docker exec -it red bash
# コンテナのOSのプロンプト
root@3afe85b72ca6:/usr/src/redmine#
```

新しいプロンプトが始まります。これはコンテナのOS(Linux)のプロンプト(bash)です。
lsコマンドを実行すると、コンテナ内のファイルの一覧が表示されます。

```sh
ls
# 出力
CONTRIBUTING.md  Gemfile.lock.mysql2	  Gemfile.lock.sqlserver  app		config	   doc	  lib	   public  tmp
Gemfile		 Gemfile.lock.postgresql  README.rdoc		  appveyor.yml	config.ru  extra  log	   sqlite  vendor
Gemfile.lock	 Gemfile.lock.sqlite3	  Rakefile		  bin		db	   files  plugins  test
```
---

### コンテナとは

**コンテナ**とは、要するに**サーバー**です。
**docker run**コマンドを使うと、PC内にサーバーを起動することができます。
以下のコマンドで実行中のコンテナの一覧が確認できます。

```sh
docker ps
# 出力
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
4c1fac037c7d        redmine             "/docker-entrypoint.…"   56 seconds ago      Up 56 seconds       0.0.0.0:3000->3000/tcp   red
```


---

### イメージとは

コンテナは**イメージ**から作成されます。
コンテナをサーバーとすると、OS、又はOS+アプリケーションのアセットです。
以下のコマンドでDockerにインストールしているイメージの一覧を表示できます。

```sh
docker images
# 出力
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
redmine             latest              8023a425328b        7 days ago          511MB
postgres            latest              f88dfa384cc4        12 days ago         348MB
```

---


### コンテナの停止

以下のコマンドを実行してください。

```sh
docker stop red
```

コンテナが停止して、ブラウザで<a href="http://localhost:3000" target="redmine">http://localhost:3000</a>にアクセスしてもページが表示されません。
**-a**オプションでコンテナ一覧に停止コンテナを表示することができます。

```sh
docker ps
# 停止中のコンテナは表示されない
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

docker ps -a
# 停止中含む全てのコンテナを表示
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS               NAMES
4c1fac037c7d        redmine             "/docker-entrypoint.…"   25 minutes ago      Exited (1) 2 minutes ago                       red
```

---

### 作成済みコンテナの起動

以下のコマンドを実行してください。

```sh
docker start red
```

作成済みのコンテナを起動することができます。

```sh
docker ps
# 出力
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
4c1fac037c7d        redmine             "/docker-entrypoint.…"   32 minutes ago      Up 3 seconds        0.0.0.0:3000->3000/tcp   red
```

---

### コンテナの削除(1/2)

ブラウザで<a href="http://localhost:3000" target="redmine">http://localhost:3000</a>にアクセスしてください。
画面右上の**ログイン**からID:admin/PW:adminでログインして、パスワード変更してください。

パスワード変更が完了したら、以下のコマンドを実行してください。

```sh
docker stop red
docker rm red
```

**docker rm**コマンドでコンテナが削除され、コンテナの一覧でも表示されなくなります。

```sh
docker ps -a
# 出力
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

---

### コンテナの削除(2/2)

以下のコマンドを再実行して、コンテナを作成します。

```sh
docker run -d --name red -p 3000:3000 redmine
```

ブラウザで<a href="http://localhost:3000" target="redmine">http://localhost:3000</a>にアクセスして、
画面右上のログインからID:admin/PW:adminでログインしてください。
再度、パスワード変更を求められます。
コンテナを削除したことによって、コンテナに保存されたデータも削除されます。

---

### ボリュームのマウント(1/2)

コンテナを運用するため、**ボリューム**を使用してデータを永続化します。
以下のコマンドを実行してください。

```sh
docker stop red
docker rm red
docker run -d --name red -p 3000:3000 -v redmine_data:/usr/src/redmine redmine
```

**-v**オプションで、ボリュームをマウントすることができます。
ボリュームが存在しない場合は自動で作成されます。
以下コマンドで作成したボリュームを表示できます。

```sh
docker volume ls
# 出力
DRIVER              VOLUME NAME
…
local               redmine_data
```

---

### ボリュームのマウント(2/2)

ブラウザで<a href="http://localhost:3000" target="redmine">http://localhost:3000</a>にアクセス、
画面右上のログインからID:admin/PW:adminでログインして、パスワードを変更してください。  
パスワード変更後、以下のコマンドでコンテナ削除、作成済みのボリュームをマウントしてコンテナを再作成してください。

```sh
docker stop red
docker rm red
docker run -d --name red -p 3000:3000 -v redmine_data:/usr/src/redmine redmine
```

ブラウザで<a href="http://localhost:3000" target="redmine">http://localhost:3000</a>にアクセスしてください。
画面右上のログインからID:admin/PW:**変更後パスワード**でログインできます。

---

### ボリュームとは

コンテナに**ボリューム**をマウントすることで、マウントしたディレクトリのデータを保存できます。
コンテナをサーバーとするとボリュームはHDDのようなものです。
コンテナ作成時にマウントすることで、コンテナの作業データをボリュームに保存、使用することができます。

---

## Dockerの基本操作

---

### ログの確認

以下のコマンドでDockerのログを表示できます。

```sh
docker logs -f red
```

ログを表示したまま、<a href="http://localhost:3000" target="redmine">http://localhost:3000</a>にアクセスしてログの出力を確認してください。
ログの停止は**Ctrl+c**をタイプしてください。

---

### コマンドの実行

コンテナの中に入らなくても、コンテナでコマンドを実行することができます。
以下のコマンドを実行してください。

```sh
docker exec red ls
```

コンテナに入ってlsを実行した場合と同じファイルの一覧が表示されます。

---

### ファイルコピー

コンテナ・ホストOSそれぞれ相互にファイルをコピーすることができます。
以下のコマンドを実行してください。

```sh
docker cp red:/usr/src/redmine/CONTRIBUTING.md .
# Windows
dir CONTRIBUTING.md
# Mac
ls CONTRIBUTING.md
```

コンテナのCONTRIBUTING.mdをホストOSにコピーすることができます。

コンテナ・ホストOSのパスを逆にすることで、ホストOSのファイルをコンテナにコピーすることもできます。

```sh
docker cp CONTRIBUTING.md red:/tmp/
docker exec red ls /tmp
```

---

### ボリュームの削除

不要になったボリュームは削除できます。
以下のコマンドを実行してください。

```sh
docker stop red
docker rm red
docker volume rm redmine_data
docker volume ls
```

**docker volume rm**でボリュームが削除され、**docker volume ls**で表示されなくなります。

---

## コンテナの用途

---

### サーバー

ここまでで触ってきたようにコンテナをサーバーとして運用することができます。  
コンテナを削除すると保存したデータも削除されますが、ボリュームを使用することでデータを永続化できます。

---

### 単独処理

以下のコマンドを実行してください。

```sh
docker run --rm redmine ls
docker ps -a
```

**docker exec red ls**と同様、redmineのファイルが表示されますが、コンテナ一覧にコンテナは存在しません。
dokcer runに**--rm**オプションを追加すると、コンテナ終了時にコンテナを削除することができます。
このオプションを利用することで、コンテナ単独での処理が可能です。

---

## イメージ・Composeの作成

---

### イメージの作成

Dockerは独自のイメージを作成することができます。
Redmine-PostgreSQL連携環境の構築を通してイメージの作成方法を理解します。

1. PostgreSQLを使用する設定のRedmineイメージを作成
1. Redmine-PostgreSQL連携環境を起動

---

### カスタムRedmineイメージの作成

1. redディレクトリを作成
1. redディレクトリに<a href="./red/database.yml">database.yml</a>(PostgreSQL接続設定ファイル)を作成  
参考：http://guide.redmine.jp/RedmineInstall/#step-3-
1. redディレクトリに以下内容の**Dockerfile**を作成
```docker
FROM redmine
ADD database.yml /usr/src/redmine/config
```
1. 以下コマンドを実行してイメージを作成
```sh
docker build -t red red/
docker images
# 出力
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
red                 latest              21d9ee9e836b        17 seconds ago      511MB
```

---

### 連携環境の起動

以下のコマンドを実行すると、連携環境が起動できます。

```sh
# postgresの起動
docker run -d --name psql postgres
# redmineの起動
docker run -d --name red --link psql:psql -p 3000:3000 red
```

起動後、ブラウザで<a href="http://localhost:3000" target="redmine">http://localhost:3000</a>にアクセス、
画面右上のログインからID:admin/PW:adminでログインして、パスワードを変更してください。  
パスワード変更後、<a href="http://localhost:3000/admin/info">http://localhost:3000/admin/info</a>にアクセスして、
**Database adapter**が**PostgreSQL**となっていることを確認してください。(デフォルトはSQLite)

---

### Composeとは

Redmine-PostgreSQL連携環境を構築しましたが、dockerコマンドではそれぞれ個別に起動する必要があります。
**Compose**を使用すると、Redmine,PostgreSQLコンテナを一括で管理することができます。
Redmine-PostgreSQL連携環境をComposeで構築します。

以下コマンドでRedmine, PostgreSQLのコンテナと、カスタムRedmineイメージを削除してください。

```sh
docker stop psql red
docker rm psql red
docker rmi red
```

---

### Composeの作成(1/2)

<div style="font-size:0.7em;">

Composeは**docker-compose.yml**ファイルで管理します。  
以下内容をコピーして、<a href="./docker-compose.yml">**docker-compose.yml**</a>という名前で保存してください。

```yml
version: "3.7"
services:
  red:
    build:
      context: ./red
    ports:
      - 3000:3000
    volumes:
      - redmine_data:/usr/src/redmine
    depends_on:
      - psql

  psql:
    image: postgres
    volumes: 
      - psql_data:/var/lib/postgresql/data

volumes:
  redmine_data:
  psql_data:
```

</div>

---

### Composeの作成(2/2)

ここまでで、以下ディレクトリ構成となっている状態です。

```sh
[WorkDirectory]
 |- docker-compose.yml
 `- red/
     |- database.yml
     `- Dockerfile
```

---

### Composeの起動

以下のコマンドでComposeの起動ができます。

```sh
docker-compose up -d
```

起動後、ブラウザで<a href="http://localhost:3000" target="redmine">http://localhost:3000</a>にアクセス、
画面右上のログインからID:admin/PW:adminでログインして、パスワードを変更してください。  
パスワード変更後、<a href="http://localhost:3000/admin/info">http://localhost:3000/admin/info</a>にアクセスして、
**Database adapter**が**PostgreSQL**となっていることを確認してください。(デフォルトはSQLite)

---

### Composeのコンテナ確認

以下のコマンドでdocker-compose.ymlで管理しているコンテナが確認できます。

```sh
docker-compose ps
# 出力
    Name                   Command               State           Ports
-------------------------------------------------------------------------------
docker_psql_1   docker-entrypoint.sh postgres    Up      5432/tcp
docker_red_1    /docker-entrypoint.sh rail ...   Up      0.0.0.0:3000->3000/tcp
```

---

### Composeの停止

以下のコマンドでComposeを停止できます。
docker-compose psでは、オプション無しで停止中のコンテナを表示できます。

```sh
docker-compose stop
docker-compose ps
# 出力
    Name                   Command               State    Ports
---------------------------------------------------------------
docker_psql_1   docker-entrypoint.sh postgres    Exit 0
docker_red_1    /docker-entrypoint.sh rail ...   Exit 1
```

---

### 作成済みComposeの起動

以下のコマンドで作成済みComposeの起動ができます。

```sh
docker-compose start
docker-compose ps
# 出力
    Name                   Command               State           Ports
-------------------------------------------------------------------------------
docker_psql_1   docker-entrypoint.sh postgres    Up      5432/tcp
docker_red_1    /docker-entrypoint.sh rail ...   Up      0.0.0.0:3000->3000/tcp
```

---

### Composeのログ確認

Composeのログ確認は、全コンテナのログの一括確認、コンテナ単位の確認ができます。

```
# 全コンテナのログを表示
docker-compose logs -f
# redmineのログを表示
dokcer-compose logs -f red
```

---

## まとめ

---

### まとめ

* Dockerとは
 * イメージからコンテナを作成してサーバーの運用ができる
 * コンテナにボリュームをマウントすることで、データの永続化が可能
* コンテナでできること
 * サーバーとして運用可能
 * オプションの指定で単独処理が可能
* Dockerfileを用意することで独自のイメージを作成できる
* Composeにより、複数コンテナが連携するDockerアプリケーションを作成できる
 * 弊社プロダクト[sit-ds](https://github.com/sitoolkit/sit-ds)もComposeで実装
 * Jenkins, Redmine, Gitbucket, Sonarqubeのアセット
 * 各種アプリケーションサーバーはLDAP認証 + SelfServicePasswordでPW一括管理
 * バックアップ・リストアをDocker単独処理で実現

---

### 付録：各種リファレンスへのリンク

<a href="http://docs.docker.jp/engine/reference/commandline/index.html">Dockerコマンド</a>

<a href="http://docs.docker.jp/engine/reference/builder.html">Dockerfileリファレンス</a>

<a href="http://docs.docker.jp/compose/compose-file.html">Composeファイル・リファレンス</a>

<a href="http://docs.docker.jp/compose/reference/overview.html">docker-composeコマンド概要</a>

---

### ご清聴ありがとうございました。

### ハンズオンお疲れさまでした。
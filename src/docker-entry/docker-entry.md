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
 - 仮想環境を動かす
 - イメージとコンテナ
 - ボリューム
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

CLI操作の基礎

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
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
4c1fac037c7d        redmine             "/docker-entrypoint.…"   32 minutes ago      Up 3 seconds        0.0.0.0:3000->3000/tcp   red
```

ブラウザで<a href="http://localhost:3000" target="redmine">http://localhost:3000</a>にアクセスしてください。
画面右上の**ログイン**からID:admin/PW:adminでログインして、パスワード変更してください。

---

### コンテナの削除

以下のコマンドを実行してください。

```sh
docker stop red
docker rm red
```

**docker rm**コマンドでコンテナが削除され、コンテナの一覧でも表示されなくなります。

```sh
docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

---

### コンテナの再作成

以下のコマンドを再実行してください。

```sh
docker run -d --name red -p 3000:3000 redmine
```

ブラウザで<a href="http://localhost:3000" target="redmine">http://localhost:3000</a>にアクセスして、
画面右上のログインからID:admin/PW:adminでログインしてください。
再度、パスワード変更を求められます。

本コマンドはredmineの**イメージ**からコンテナを作成・起動するコマンドです。

---

### イメージとは

**イメージ**とはコンテナのベースになるものです。
コンテナの変更内容はコンテナだけに保存されて、同じイメージからコンテナを構築しても、過去に存在したコンテナとは別物になります。

以下のコマンドでDockerにインストールしているイメージの一覧を表示できます。

```sh
docker images
# 出力
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
redmine             latest              8023a425328b        6 days ago          511MB
```

---

### ボリュームを作成してマウント

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

### 作成済みボリュームのマウント

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

**ボリューム**とは、外付けHDDのような外部記憶領域です。
ボリュームを使用することで、コンテナを削除した場合でも変更データを永続化することができます。

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

### 仮想サーバー

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
このオプションを利用することで、サーバーとしての用途ではなく、単独の処理が可能です。

---

## イメージ・Composeの作成

---

### イメージの作成

Dockerは独自のイメージを作成することができます。
Redmine-PostgreSQL連携環境の構築を通してイメージの作成方法を理解します。

1. Redmine用初期化済みPostgreSQLイメージを作成
1. DBにPostgreSQLを使用する設定のRedmineイメージを作成

---

### カスタムPostgreSQLイメージの作成

1. redpostgresディレクトリを作成
1. redpostgresディレクトリに<a href="./redpostgres/init-redmine.sql">init-redmine.sql</a>(Redmine用初期化SQLファイル)を作成  
参考：http://guide.redmine.jp/RedmineInstall/#step-2-
1. redpostgresディレクトリに以下内容のDockerfileを作成
```docker
FROM postgres
ADD init-redmine.sql /docker-entrypoint-initdb.d 
```
1. 以下コマンドを実行
```sh
docker build -t redpostgres redpostgres/
docker images
# 出力
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
redpostgres         latest              0ed2592243c1        11 minutes ago      348MB
```

---

### カスタムRedmineイメージの作成

1. redディレクトリを作成
1. redディレクトリに<a href="./red/database.yml">database.yml</a>(PostgreSQL接続設定ファイル)を作成  
参考：http://guide.redmine.jp/RedmineInstall/#step-3-
1. redディレクトリに以下内容のDockerfileを作成
```docker
FROM redmine
ADD database.yml /usr/src/redmine/config
```
1. 以下コマンドを実行
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
docker run -d --name redpostgres redpostgres
# redmineの起動
docker run -d --name red --link redpostgres:redpostgres -p 3000:3000 red
```

起動後、ブラウザで<a href="http://localhost:3000" target="redmine">http://localhost:3000</a>にアクセス、
画面右上のログインからID:admin/PW:adminでログインして、パスワードを変更してください。  
パスワード変更後、<a href="http://localhost:3000/admin/info">http://localhost:3000/admin/info</a>にアクセスして、
**Database adapter**が**PostgreSQL**となっていることを確認してください。(デフォルトはSQLite)

---

# OLD

---

### Composeの作成(1/4)

Composeとは、複数のDockerコンテナを使用するDockerアプリケーションを実行するツールです。  
ここまでで触ってきたRedmine, Jenkinsを運用して開発を行う場合、それぞれのサーバーを個別に起動・停止するのではなく、
一括で起動・停止・状態確認等ができます。

myjenkinsコンテナを停止・削除して、myjenkinsイメージを削除してください。

---

### Composeの作成(2/4)

<div style="font-size:0.7em;">

Composeは**docker-compose.yml**ファイルで管理します。  
以下内容をコピーして、**docker-compose.yml**という名前で保存してください。

```
version: "3.7"
services:
  redmine:
    image: redmine:alpine
    ports:
      - 3000:3000
    volumes:
      - redmine_data:/usr/src/redmine

  myjenkins:
    build:
      context: ./myjenkins
    ports:
      - 8080:8080
    volumes:
      - myjenkins_data:/var/jenkins_home

volumes:
  redmine_data:
  myjenkins_data:
```

上記docker-compose.ymlの内容の詳細は
[Composeファイルリファレンス](http://docs.docker.jp/compose/compose-file.html)
を参照してください。  

</div>

---

### Composeの作成(3/4)

ここまでで、以下ディレクトリ構成となっている状態です。

```
[WorkDirectory]
 |- docker-compose.yml
 `- myjenkins/
     |- create-admin.groovy
     `- Dockerfile
```

Composeは以下のコマンドで起動します。

```
$ docker-compose up -d
```

[http://localhost:8080](http://localhost:8080)でJenkins、
[http://localhost:3000](http://localhost:3000)でRedmineが起動していることを確認してください。

---

### Composeの作成(4/4)

<div style="font-size:0.9em;">

Composeで起動したコンテナは起動時と同じくdocker-composeコマンドで停止できます。

```
$ docker-compose stop
```

サービス名を指定することで、サービス単位での起動・停止もできます。  
以下のコマンドを実行してください。

```
$ docker-compose up -d myjenkins
```

**docker-compose ps**の実行と、
[http://localhost:8080](http://localhost:8080)、
[http://localhost:3000](http://localhost:3000)
それぞれにアクセスして稼働状態を確認してください。

その他、実行可能なコマンドは
[docker-composeコマンド概要](http://docs.docker.jp/compose/reference/overview.html)
を参照してください。

</div>

---

## まとめ

---

### まとめ

* Dockerとはイメージからコンテナを起動する仮想環境プラットフォーム  
ボリュームを使用してデータの永続化が可能
* 単独処理/サーバー
 * オプションの指定で、単独処理の実行後にコンテナ自動削除
 * 永続化によりアプリケーションサーバーの提供が可能
* Dockerfileによる独自イメージの作成が可能
* Composeによる複数コンテナが連携するDockerアプリケーションの作成が可能
 * 弊社プロダクト[sit-ds](https://github.com/sitoolkit/sit-ds)もComposeで実装
 * Jenkins, Redmine, Gitbucket, Sonarqubeのアセット
 * LDAPによる認証、SelfServicePasswordによるPW管理
 * バックアップ・リストアをDocker単独処理で実施

---

### 付録：各種リファレンスへのリンク

<a href="http://docs.docker.jp/engine/reference/commandline/index.html">Dockerコマンド</a>

<a href="http://docs.docker.jp/engine/reference/builder.html">Dockerfileリファレンス</a>

<a href="http://docs.docker.jp/compose/compose-file.html">Composeファイル・リファレンス</a>

<a href="http://docs.docker.jp/compose/reference/overview.html">docker-composeコマンド概要</a>

---

### ご清聴ありがとうございました。

### ハンズオンお疲れさまでした。
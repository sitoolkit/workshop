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
- ハンズオン：Dockerの基本操作
- コンテナの用途
- 連携環境の作成
- Composeとは
- ハンズオン：Composeの作成
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

* [Visual Studio Code](https://code.visualstudio.com/)

---

## Dockerの構成要素

---

### コンテナを動かす

コマンドプロンプト(Windows) / ターミナル(macOS)で以下のコマンドを実行してください。

```sh
docker run -d --name psql postgres
```

これで、DockerでPostgreSQLサーバーが起動しました。

---

### コンテナの中に入る

以下のコマンドを実行してください。

```sh
docker exec -it psql bash
# コンテナのOSのプロンプト
root@21321a7c78f9:/# 
```

新しいプロンプトが始まります。これはコンテナのOS(Linux)のプロンプト(bash)です。
psqlコマンドを実行すると、PostgreSQLに接続することができます。

```sh
psql -U postgres
# 出力
psql (12.0 (Debian 12.0-2.pgdg100+1))
Type "help" for help.

postgres=#
```
---

### コンテナとは

**コンテナ**とは、要するに**サーバー**です。
**docker run**コマンドを使うと、PC内にサーバーを起動することができます。
以下のコマンドで実行中のコンテナの一覧が確認できます。

```sh
docker ps
# 出力
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
21321a7c78f9        postgres            "docker-entrypoint.s…"   3 minutes ago       Up 2 minutes        5432/tcp            psql
```

---

### イメージの作成(1/3)

前述の**docker run**コマンドは<a href="https://hub.docker.com/_/postgres">Docker Hub</a>
で公開されているPostgreSQLの**Dockerイメージ**でコンテナを起動しています。

**Dockerイメージ**は公開されているイメージをカスタマイズして作ることもできます。

以下のコマンドを実行し、jp-psqlディレクトリとDockerfileファイルを作成してください。

```sh
mkdir jp-psql
cd jp-psql
code Dockerfile
```

---

### イメージの作成 (2/3)

VSCodeがDockerfileを開いたら以下の内容を貼り付けてください。

```docker
FROM postgres
RUN localedef -i ja_JP -c -f UTF-8 -A /usr/share/locale/locale.alias ja_JP.UTF-8
ENV LANG ja_JP.utf8
```

---

### イメージの作成 (3/3)

Dockerfileを保存し、以下のコマンドを実行してください。

```sh
docker build -t jp-psql .
```

これで、**ロケールを日本に変更したPostgreSQL**のイメージが作成できました。

---

### 作成したイメージでコンテナを起動

以下のコマンドでコンテナを起動してください。

```sh
docker run -d --name jp-psql jp-psql
```

コンテナが起動したら、以下のコマンドを実行してください。

```sh
docker exec -it jp-psql bash
# コンテナのプロンプトで
psql -U postgres
# 出力
psql (12.0 (Debian 12.0-2.pgdg100+1))
"help"でヘルプを表示します。

postgres=#
```

ログインメッセージが日本語になっています。

---

### イメージとは

**イメージ**は、要するに**サーバーのインストールメディア**です。

イメージを作っておけば、OSやアプリケーションがセットアップされた状態のコンテナを即座に作成することができます。

```sh
docker images
# 出力
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
jp-psql             latest              cad33107446c        17 seconds ago      352MB
postgres            latest              f88dfa384cc4        4 weeks ago         348MB
```

---

### ボリュームのマウント(1/4)

コンテナの起動時に外部ファイルを参照する、コンテナのデータを外部に保存するためには、**ボリューム**を使用します。

以下のコマンドを実行し、initdbディレクトリとinitilize.shを作成してください。


```sh
cd ..
mkdir initdb
code initdb/initilize.sh
```

---

### ボリュームのマウント(2/4)

VSCodeがinitilize.shを開いたら以下の内容を貼り付けて、改行コードをLFにしてください。

・initilize.sh

```sh
#!/bin/bash
set -e

psql -v ON_ERROR_STOP=1 -U postgres <<-EOSQL
  CREATE USER docker;
  CREATE DATABASE docker;
  GRANT ALL PRIVILEGES ON DATABASE docker TO docker;
EOSQL
```

・VSCode：改行コードの指定  
<img src="vscode-bottom.png"/>

---

### ボリュームのマウント(3/4)

initilize.shを保存し、以下のコマンドでコンテナを起動してください。

```sh
# Windows
for /f "usebackq tokens=*" %i IN (`cd`) DO @set PWD=%i
docker run -d --name mnt-psql -v %PWD%/initdb:/docker-entrypoint-initdb.d -v psql_data:/var/lib/postgresql/data jp-psql

# Mac
docker run -d --name mnt-psql -v ${PWD}/initdb:/docker-entrypoint-initdb.d -v psql_data:/var/lib/postgresql/data jp-psql
```

---

### ボリュームのマウント(4/4)

以下のコマンドを実行してください。

```sh
docker exec -it mnt-psql bash
# コンテナのプロンプトで
psql -U docker
# 出力
psql (12.0 (Debian 12.0-2.pgdg100+1))
"help"でヘルプを表示します。

docker=>
```

dockerユーザーでdocker DBにログインできます。

---

### ボリュームとは

**ボリューム**とは、要するに**外付け記憶媒体**です。

ボリュームをコンテナにマウント(接続)することで、
コンテナ内に外部からデータを持ち込んだり、コンテナ内のデータを外部に持ち出すことができます。

---

### ボリュームの種類(1/2)

* 共有ディレクトリ  
オプションを**-v [ホストOSのパス]:[ディレクトリ]**と指定すると、
ホストOSとコンテナの共有ディレクトリとして扱うことができます。

---

### ボリュームの種類(2/2)

* NamedVolume  
オプションを**-v [任意の名前]:[ディレクトリ]**と指定すると、**NamedVolume**と呼ばれるボリュームをマウントします。
ホストOSからボリューム内のファイル操作ができないので、データを安全に保存することができます。

```sh
docker volume ls
# 出力
DRIVER              VOLUME NAME
local               psql_data
```

---

## ハンズオン
## Dockerの基本操作

---

### 課題

以下の課題を通してDockerの基本操作を学びます。

1. psqlコンテナのログを表示する。

2. mnt-psqlの/docker-entrypoint-initdb.d/initilize.shをホストOSにコピーする。

3. initilize.shをpsqlの/tmpにコピーして、/tmp/initilize.shを実行する。

4. psqlコンテナのbashを経由せずに、PostgreSQLにdockerユーザーでログインする。
(psqlログインコマンド：psql -U docker)

5. psql、jp-psql、mnt-psqlコンテナを停止、削除する。

6. jp-psqlイメージを削除する。

---

### 基本操作(1/4)

・ログの表示

```sh
# ログを表示してプロンプトに戻る
docker logs [コンテナ名]
# プロンプトに戻らずログを表示し続ける(Ctrl+cで停止)
dokcer logs -f [コンテナ名]
```

・コンテナでのコマンド実行

```sh
# コマンドを実行してプロンプトに戻る
docker exec [コンテナ名] [実行コマンド]
# bash/psql等のターミナルを操作
docker exec -it [コンテナ名] [ターミナル]
```

---

### 基本操作(2/4)

・作成済みコンテナの起動

```sh
docker start [コンテナ名]
```

・起動中コンテナの停止

```sh
docker stop [コンテナ名]
```

・停止中コンテナの削除

```sh
docker rm [コンテナ名]
```

---

### 基本操作(3/4)

・ファイルコピー

```sh
# ホストOSからコンテナへコピー
docker cp [ファイルパス] [コンテナ名]:[コピー先ディレクトリ]
# コンテナからホストOSへコピー
docker cp [コンテナ名]:[ファイルパス] [コピー先ディレクトリ]
```

・イメージの確認

```sh
docker images
```

・イメージ削除

```sh
docker rmi [イメージ名]
```

---

### 基本操作(4/4)

・ボリューム確認

```sh
docker volume ls
```

・ボリュームを指定して削除

```sh
docker volume rm [ボリューム名]
```

---

## それでは

## 作業を始めてください

---

## コンテナの用途

---

### サーバー

ここまでで触ってきたようにコンテナをサーバーとして運用することができます。  
コンテナを削除すると保存したデータも削除されますが、ボリュームを使用することでデータを永続化できます。

---

### 単独処理(1/2)

以下のコマンドを実行してください。

```sh
# Windows
for /f "usebackq tokens=*" %i IN (`cd`) DO @set PWD=%i
docker run --rm -v psql_data:/target -v %PWD%:/backup postgres tar cfz /backup/psql_data.tar.gz -C /target .
dir psql_data.tar.gz

# Mac
docker run --rm -v psql_data:/target -v ${PWD}:/backup postgres tar cfz /backup/psql_data.tar.gz -C /target .
ls psql_data.tar.gz

# 共通：コマンド完了後
docker ps -a
```

psql_data.tar.gzが作成できますが、コンテナ一覧にコンテナは存在しません。

---

### 単独処理(2/2)

dokcer runに**--rm**オプションを追加すると、コンテナ終了時にコンテナを削除することができます。
このオプションを利用することで、コンテナ単独での処理が可能です。

先のコマンド例では、psql_dataボリュームのデータをバックアップしました。
別の端末にバックアップをコピーしてコンテナでマウントすることで、同一の環境を構築することができます。

---

## 連携環境の作成

---

### 連携環境(1/5)

Dockerでは複数のコンテナで構成される連携環境を構築することができます。
例として、PostgreSQLとphpPgAdminの連携環境構築します。

---

### 連携環境(2/5)

以下のコマンドを実行して、phpPgAdminのDockerfileを作成します。

```sh
mkdir pgadmin
code pgadmin/Dockerfile
```

VSCodeが開いたら、以下の内容を貼り付けて保存してください。

```docker
FROM dockage/phppgadmin

ENV PHP_PG_ADMIN_SERVER_HOST="jp-psql" \
    PHP_PG_ADMIN_SERVER_PORT=5432
```

---

### 連携環境(3/5)

以下のコマンドを実行して、Dockerfileからイメージを作成します。

```sh
docker build -t pgadmin pgadmin/
```

---

### 連携環境(4/5)

以下のコマンドで、PostgreSQL、phpPgAdminを起動します。

```sh
docker run -d --name jp-psql jp-psql
docker run -d --name pgadmin -p 80:80 --link jp-psql:jp-psql pgadmin
```

ブラウザで<a href="http://localhost">http://localhost</a>にアクセスすると、phpPgAdminが表示できます。

---

### 連携環境(5/5)

phpPgAdminが開いたら、"サーバー > PosgreSQL"をクリックして、"ユーザー名:postgres/パスワード:[空欄]"でログインしてください。

jp-psqlコンテナにログインすることができます。

<img src="pgadmin.png"/>

---

## Composeとは

---

### Composeとは

前述の連携環境はPostgreSQL、phpPgAdminのコンテナを個別に起動しました。

**Compose**を使用することで複数のコンテナを一括で起動することができます。
同様の連携環境をComposeで構築することで、Composeの構築方法を学びます。

---

### Composeファイル(1/4)

Composeで起動するコンテナは**docker-compose.yml**で管理します。
docker-compose.ymlの構成について説明します。

* version  
Composeファイルのバージョンを指定します。
最新は3.7です。  
参考：<a href="https://github.com/docker/compose/releases">https://github.com/docker/compose/releases</a>

```yml
version: "3.7"
```

---

### Composeファイル(2/4)

* services  
Composeで起動するコンテナを定義します。
ここでは、PostgreSQLを起動する簡単な例を記載します。

```yml
services:
  psql:              # サービス名
    image: postgres  # コンテナを起動するイメージ
    volumes:         # ボリュームを指定
      - ./initdb:/docker-entrypoint-initdb.d  # 共有ディレクトリ
      - psql_data:/var/lib/postgresql/data    # NamedVolume
    port:            # 解放するポートを指定
      - 5432:5432
```

参考：<a href="http://docs.docker.jp/compose/compose-file.html">Composeファイル・リファレンス</a>

---

### Composeファイル(3/4)

* volumes  
servicesでNamedVolumeを使用する場合、ここで定義します。
dockerコマンドとは異なり、ボリュームは自動作成されません。

```yml
volumes:
  psql_data:
```

---

### Composeファイル(4/4)

ここまでで説明した内容をマージしたdocker-compose.ymlの全体像は以下のようになります。

```yml
version: "3.7"

services:
  psql:
    image: postgres
    volumes:
      - psql_data:/var/lib/postgresql/data
    port:
      - 5432:5432

volumes:
  psql_data:
```

---

### Composeの基本コマンド

docker-compose.ymlのあるディレクトリで、以下コマンドを実行するとComposeを操作できます。

・Composeの起動

```sh
docker-compose up -d
```

・Composeの停止

```sh
docker-compose stop
```

・作成済みComposeの起動

```sh
docker-compose start
```

・コンテナの確認

```sh
docker-compose ps
```

---

## ハンズオン
## Composeの作成

---

### ハンズオンの事前準備

以下コマンドでPostgreSQL、phpPgAdminのコンテナを停止・削除してください。

```sh
docker stop pgadmin jp-psql
docker rm pgadmin jp-psql
```

---

### 課題

<div style="font-size:0.9em;">

以下のComposeを作成してください。

* PostgreSQL
 * jp-psqlのDockerfileからイメージを作成して起動する
 * initdb/initilize.shを起動時に実行する
 * /var/lib/postgresql/dataにNamedVolumeをマウントする

* phpPgAdmin
 * dockage/phppgadminからコンテナを作成
 * PostgreSQLの起動を待って起動する
 * 前述のPostgreSQL:5432ポートに接続する
 * 80ポートでアクセス可能とする

作成できたらブラウザで<a href="http://localhost">http://localhost</a>にアクセスして、
dockerユーザーでPostgreSQLにログインできることを確認してください。

</div>

---

## それでは

## 作業を始めてください

---

## まとめ

---

### まとめ

* Dockerの構成要素
 * イメージからコンテナを構築してサーバーとして運用
 * コンテナにボリュームをマウントすることでデータを永続化
* コンテナでできること
 * サーバーとして運用可能
 * オプションの指定で単独処理が可能
* Dockerfileを用意することで独自のイメージを作成できる
* Composeにより、複数コンテナが連携するDockerアプリケーションを作成できる
 * 弊社プロダクト[sit-ds](https://github.com/sitoolkit/sit-ds)もComposeで実装
 * Jenkins, Redmine, Gitbucket, Sonarqube, Artifactoryのアセット
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
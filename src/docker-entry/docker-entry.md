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

* [Visual Studio Code](https://code.visualstudio.com/)

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
f4a5f3fe4fa1        redmine             "/docker-entrypoint.…"   20 seconds ago      Up 19 seconds       0.0.0.0:3000->3000/tcp   red
```

---

### イメージの作成(1/3)

前述の**docker run**コマンドは<a href="https://hub.docker.com/_/redmine">Docker Hub</a>
で公開されているRedmineの**Dockerイメージ**でコンテナを起動しています。

**Dockerイメージ**は公開されているイメージをカスタマイズして作ることもできます。

以下のコマンドを実行し、mywebserverディレクトリとDockerfile、docker-entry.htmlファイルを作成してください。

```sh
mkdir mywebserver
cd mywebserver
code Dockerfile
code docker-entry.html
```

---

### イメージの作成 (2/3)

VSCodeがDockerfile、docker-entry.htmlを開いたら以下の内容を貼り付けてください。

・Dockerfile
```docker
FROM nginx
ADD docker-entry.html /usr/share/nginx/html
```

・docker-entry.html
```html
<html><body><h1>WebServer working by docker</h1></body></html>
```

---

### イメージの作成 (3/3)

Dockerfile、docker-entry.htmlを保存し、以下のコマンドを実行してください。

```sh
docker build -t mywebserver .
```

これで、**docker-entry.htmlを追加したNginx**のイメージが作成できました。

---

### 作成したイメージでコンテナを起動

以下のコマンドでコンテナを起動してください。


```sh
docker run -d --name mywebserver -p 81:80 mywebserver
```

コンテナが起動したら、ブラウザで
<a href="http://localhost:81/docker-entry.html">http://localhost:81/docker-entry.html</a>
にアクセスしてください。
作成した**dokcer-entry.html**が表示できます。

---

### イメージとは

**イメージ**は、要するに**サーバーのインストールメディア**です。

イメージを作っておけば、OSやアプリケーションがセットアップされた状態のコンテナを即座に作成することができます。

```sh
docker images
# 出力
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
mywebserver         latest              1d0bbf6552e1        19 seconds ago      126MB
nginx               latest              540a289bab6c        9 days ago          126MB
redmine             latest              8023a425328b        10 days ago         511MB
```

---

### ボリュームのマウント(1/4)

コンテナに保存された情報はコンテナを削除すると失われます。
コンテナの情報を永続化するためには**ボリューム**を使用します。

以下のコマンドを実行し、share-htmlディレクトリとindex.htmlを作成してください。

```sh
cd ..
mkdir share-html
code share-html/index.html
```

---

### ボリュームのマウント(2/4)

VSCodeがindex.htmlを開いたら以下の内容を貼り付けてください。

```html
<html><body><h1>WebServer mount share directory</h1></body></html>
```

---

### ボリュームのマウント(3/4)

index.htmlを保存し、以下のコマンドでコンテナを起動してください。

```sh
# Windows
for /f "usebackq tokens=*" %i IN (`cd`) DO @set current_dir=%i
docker run -d --name mountwebserver -p 82:80 -v %current_dir%/share-html:/usr/share/nginx/html -v web_mnt:/mnt nginx

# Mac
docker run -d --name mountwebserver -p 82:80 -v ${PWD}/share-html:/usr/share/nginx/html -v web_mnt:/mnt nginx
```

<a href="http://localhost:82/">http://localhost:82/</a>にアクセスすると、作成したページが表示されます。

---

### ボリュームのマウント(4/4)

docker runに**-v**オプションをつけることでボリュームをマウントすることができます。  
オプションのフォーマットは**-v [ボリューム]:[コンテナのディレクトリ]**です。

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
local               web_mnt
```

---

### ボリュームとは

**ボリューム**は、要するに**HDDやSSDのような記憶装置**です。

コンテナにマウントすることで、コンテナのデータをボリュームに保存することができます。

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

### ファイルコピー(1/2)

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

---

### ファイルコピー(2/2)

コンテナ・ホストOSのパスを逆にすることで、ホストOSのファイルをコンテナにコピーすることもできます。

```sh
docker cp CONTRIBUTING.md red:/tmp/
docker exec red ls /tmp
```

---

### コンテナの停止

<div style="font-size:0.8em;">

以下のコマンドを実行してください。

```sh
docker stop red
```

コンテナが停止して、<a href="http://localhost:3000">http://localhost:3000</a>にアクセスできなくなります。
**-a**オプションでコンテナ一覧に停止コンテナを表示することができます。

```sh
docker ps
# 停止中のコンテナは表示されない
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
b8eaaaae6450        nginx               "nginx -g 'daemon of…"   5 minutes ago       Up 33 seconds       0.0.0.0:82->80/tcp   mountwebserver
3b4500688148        mywebserver         "nginx -g 'daemon of…"   5 minutes ago       Up 27 seconds       0.0.0.0:81->80/tcp   mywebserver

docker ps -a
# 停止中を含む全てのコンテナを表示
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS                NAMES
b8eaaaae6450        nginx               "nginx -g 'daemon of…"   5 minutes ago       Up 47 seconds               0.0.0.0:82->80/tcp   mountwebserver
3b4500688148        mywebserver         "nginx -g 'daemon of…"   6 minutes ago       Up 41 seconds               0.0.0.0:81->80/tcp   mywebserver
16830ca679ba        redmine             "/docker-entrypoint.…"   8 minutes ago       Exited (1) 16 seconds ago                        red
```

</div>

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
b8eaaaae6450        nginx               "nginx -g 'daemon of…"   5 minutes ago       Up About a minute   0.0.0.0:82->80/tcp       mountwebserver
3b4500688148        mywebserver         "nginx -g 'daemon of…"   6 minutes ago       Up About a minute   0.0.0.0:81->80/tcp       mywebserver
16830ca679ba        redmine             "/docker-entrypoint.…"   8 minutes ago       Up 1 second         0.0.0.0:3000->3000/tcp   red
```

---

### コンテナの削除

以下のコマンドを実行してください。

```sh
docker stop red mywebserver mountwebserver
docker rm red mywebserver mountwebserver
```

コンテナが削除され、コンテナの一覧でも表示されなくなります。

```sh
docker ps -a
# 出力
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

コンテナに保存されたデータも併せて削除されますが、ボリュームに保存されたデータは削除されません。

---

### ボリュームの削除

ボリュームは以下のコマンドで削除できます。

```sh
docker volume rm web_mnt
docker volume ls
```

**docker volume ls**で**web_mnt**が表示されなくなります。

---


### イメージの削除

イメージは以下のコマンドで削除できます。

```sh
docker rmi mywebserver
docker images
```

**docker images**で**mywebserver**が表示されなくなります。

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

Nginxをプロキシサーバーとして構築し、Nginx経由でRedmineにアクセスする環境の構築を通してイメージの作成方法を理解します。

1. プロキシ設定を追加したnginxイメージを作成
1. Nginx-Redmine連携環境を起動

---

### カスタムNginxイメージの作成(1/2)

1. proxyディレクトリとproxy.conf(Nginxのプロキシ設定ファイル)、Dockerfileを作成
```sh
mkdir proxy
code proxy/proxy.conf
code proxy/Dockerfile
```

2. VSCodeのproxy.confに以下の内容を貼り付けて保存
```
server {
    listen  80;
    server_name  localhost;

    location / {
        proxy_pass http://red:3000;
    }
}
```

---

### カスタムNginxイメージの作成(2/2)

3. VSCodeのDockerfileに以下の内容を貼り付けて保存
```docker
FROM nginx
RUN rm -f /etc/nginx/conf.d/*
ADD proxy.conf /etc/nginx/conf.d
```

4. 以下コマンドを実行してイメージを作成
```sh
docker build -t proxy proxy/
docker images
# 出力
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
proxy               latest              3bb82046179c        7 seconds ago       126MB
nginx               latest              540a289bab6c        3 weeks ago         126MB
redmine             latest              8023a425328b        3 weeks ago         511MB
```

---

### 連携環境の起動

以下のコマンドを実行すると、連携環境が起動できます。

```sh
# Redmineの起動
docker run -d --name red redmine
# カスタムNginxの起動
docker run -d --name proxy --link red:red -p 80:80 proxy
```

起動後、ブラウザで<a href="http://localhost">http://localhost</a>にアクセスしてください。
カスタムNginxコンテナで指定されている80番ポートへのアクセスで、Redmineへアクセスできます。

---

### Composeとは

前述の連携環境はNginx、Redmineのコンテナを個別に起動しました。

**Compose**を使用することで複数のコンテナを一括で起動することができます。
同様の連携環境をComposeで構築することで、Composeの構築方法を学びます。

---

### 事前準備

以下コマンドでカスタムNginx, Redmineのコンテナと、カスタムNginxイメージを削除してください。

```sh
docker stop proxy red
docker rm proxy red
docker rmi proxy
```

---

### Composeの作成(1/2)

Composeは**docker-compose.yml**ファイルで管理します。
以下のコマンドを実行して、docker-compose.ymlを作成してください。

```
code docker-compose.yml
```

---

### Composeの作成(2/2)

VSCodeのdocker-compose.ymlに以下内容を貼り付けて保存してください。

<dev style="font-size: 0.8em;">

```yml
version: "3.7"
services:
  red:
    image: redmine
    volumes:
      - redmine_data:/usr/src/redmine

  proxy:
    build:
      context: ./proxy
    ports: 
      - 80:80
    depends_on:
      - red

volumes:
  redmine_data:
```

</dev>

---

### Composeの起動

以下のコマンドでComposeの起動ができます。

```sh
docker-compose up -d
```

起動後、ブラウザで<a href="http://localhost">http://localhost</a>にアクセスしてください。
イメージ作成時と同様、80番ポートへのアクセスでRedmineへアクセスできます。

---

### Composeのコンテナ確認


以下のコマンドでdocker-compose.ymlで管理しているコンテナが確認できます。

```sh
docker-compose ps
# 出力
     Name                   Command               State         Ports
----------------------------------------------------------------------------
docker_proxy_1   nginx -g daemon off;             Up      0.0.0.0:80->80/tcp
docker_red_1     /docker-entrypoint.sh rail ...   Up      3000/tcp
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
----------------------------------------------------------------
docker_proxy_1   nginx -g daemon off;             Exit 0
docker_red_1     /docker-entrypoint.sh rail ...   Exit 1
```

---

### 作成済みComposeの起動

以下のコマンドで作成済みComposeの起動ができます。

```sh
docker-compose start
docker-compose ps
# 出力
     Name                   Command               State         Ports
----------------------------------------------------------------------------
docker_proxy_1   nginx -g daemon off;             Up      0.0.0.0:80->80/tcp
docker_red_1     /docker-entrypoint.sh rail ...   Up      3000/tcp
```

---

### Composeのログ確認

Composeのログ確認は、全コンテナのログの一括確認、コンテナ単位の確認ができます。

```
# 全コンテナのログを表示
docker-compose logs -f
# Nginxのログを表示
dokcer-compose logs -f proxy
```

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
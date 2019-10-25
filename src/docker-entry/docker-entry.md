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

### 環境の準備(1/2)

ハンズオンでは以下のソフトウェアが必要です。

* Docker
 * Windows  
   [Docker Desktop for Windows](https://docs.docker.com/docker-for-windows/install/)  
 * Mac  
   [Docker Desktop for Mac](https://docs.docker.com/docker-for-mac/install/)  
 * Linux  
   [Docker](https://docs.docker.com/install/linux/docker-ce/centos/)  
   [Docker Compose](https://docs.docker.com/compose/install/)

* 高機能エディタ
 * [Visual Studio Code](https://code.visualstudio.com/) 等

---

### 環境の準備(2/2)

ハンズオンではRedmine、Jenkinsを使用します。  
CLIで以下のコマンドを実行し、Redmine、JenkinsのイメージをPULLしてください。

```
$ docker pull redmine:alpine
$ docker pull jenkins/jenkins:alpine
```

---

## Dockerの構成要素

---

### 仮想環境を動かす

<div style="font-size:0.9em">

環境準備に記載したコマンドで、**イメージ**と呼ばれる仮想環境のベースをインストールしました。  
インストールしたイメージは以下のコマンドで確認できます。

```
$ docker images
```

以下のコマンドでインストールしたイメージを仮想環境として動かすことができます。

```
$ docker run -d -p 3000:3000 redmine:alpine
```

起動した仮想環境を**コンテナ**と呼びます。

Redmineには[http://localhost:3000/](http://localhost:3000/)でアクセスできます。  
ID:admin/PW:adminで管理者でログインできます。  
Redmineにログイン、初期パスワードを変更して、任意のプロジェクトを作成してください。

</div>

---

### イメージとコンテナ(1/3)

<div style="font-size:0.9em">

コンテナは以下のコマンドで確認できます。

```
$ docker ps
```

以下のコマンドでコンテナを停止できます。
[コンテナID]には**docker ps**で表示した**CONTAINER ID**を指定します。  
Redmineを一度停止して、再度[http://localhost:3000/](http://localhost:3000/)にアクセスしてください。

```
$ docker stop [コンテナID]
```

停止中のコンテナは**docker ps**で表示されません。  
停止中のコンテナも含め、すべてのコンテナを表示する場合は**-a**オプションをつけてください。

```
$ docker ps -a
```

</div>

---

### イメージとコンテナ(2/3)

Redmineを起動し、再度[http://localhost:3000/](http://localhost:3000/)にアクセスして、
作成したプロジェクトを確認してください。  
コンテナの停止ではアプリケーションの変更内容は保存されています。

Redmineを停止してコンテナを削除します。  
以下のコマンドを実行して下さい。

```
$ docker rm [コンテナID]
$ docker ps -a
```

コンテナが削除され、**docker ps -a**でもコンテナが表示されなくなっています。

---

### イメージとコンテナ(3/3)

もう一度コンテナを起動して、Redmineにアクセスしてください。
Redmineにはアクセスできますが、作成したプロジェクトは存在しません。

コンテナを削除すると保存された内容もあわせて削除されます。
イメージとはコンテナを起動するベースになるもので、
同一のイメージから再度コンテナを起動しても、先に存在したコンテナとは別物になります。

ここで、コンテナの停止・削除をしてください。

---

### ボリューム(1/2)

サーバーを運用する場合、データの永続化が必要となります。  
Dockerではデータ永続化のためのボリュームという機能があります。  
以下のコマンドでRedmineを起動してください。

```
$ docker run -d -p 3000:3000 -v redmine_data:/usr/src/redmine redmine:alpine
```

**-v**オプションでコンテナにボリュームと呼ばれる外部記憶領域をマウントすることができます。  
上記コマンドの場合、redmine_dataというボリュームを作成し、コンテナの/usr/src/redmineにマウントします。  
作成したボリュームは以下のコマンドで確認できます。

```
$ docker volume ls
```

---

### ボリューム(2/2)

コンテナを停止・削除してください。  

再度同じコマンドでコンテナを起動して、admin/変更後パスワードでログインしてください。    
パスワードの初回変更は問われずにログインでき、作成したプロジェクトが残っています。

---

## Dockerの基本操作

---

### Dockerの基本操作(1/5)

ここまでに紹介した以外に、よく使用するDockerの基本操作を紹介します。

・ログの確認  
以下のコマンドでログを確認できます。
コマンドを実行して、Redmineにアクセスしてください。

```
$ docker logs -f [コンテナID]
```

ログの停止は Ctrl+cをタイプしてください。

---

### Dockerの基本操作(2/5)

・コンテナへのコマンド実行  
以下のコマンドでコンテナでコマンドが実行できます。

```
$ docker exec [コンテナID] [実行コマンド]
```

例えば、以下のコマンドを実行することで、コンテナの作業ディレクトリのファイル一覧を表示できます。

```
$ docker exec [コンテナID] ls
```

コンテナのシェルにログインして作業する場合はオプションが必要になります。

```
$ docker exec -it [コンテナID] bash
-i：アタッチしていなくてもSTDINをオープンにし続ける
-t：疑似ターミナルの割り当て
```

---

### Dockerの基本操作(3/5)

・コンテナ・ホストOS間ファイルコピー  
以下のコマンドでコンテナ・ホストOS間でファイルコピーできます。

```
$ docker cp /local/file [コンテナID]:/container/path  # ホストOSのファイルをコンテナにコピー
$ docker cp [コンテナID]:/container/file /local/path  # コンテナのファイルをホストOSにコピー
```

任意のファイルを作成し、コンテナにファイルをコピーしてください。  
コピーができたら、コンテナにbashでログインして、ファイルがコピーされていることを確認してください。

---

### Dockerの基本操作(4/5)

・ボリュームの削除  
コンテナを停止してください。  
以下のコマンドでボリュームを削除できます。  

```
$ docker volume rm [ボリューム名]
```

コンテナを停止した状態だけではボリュームの削除はできず、
ボリュームを削除する場合はコンテナも削除する必要があります。

コンテナを削除して、再度ボリューム削除コマンドを実行してください。

---

### Dockerの基本操作(5/5)

・イメージの削除  
以下のコマンドで不要となったイメージを削除できます。

```
$ docker rmi [イメージID]
```

**※環境の準備で用意したRedmine、Jenkinsはこの後使用するため削除しないでください**

その他、実行可能なコマンドは
[Dockerコマンド](http://docs.docker.jp/engine/reference/commandline/index.html)
を参照してください。

---

## コンテナの用途

---

### コンテナの用途(1/2)

Dockerではコンテナをサーバーとして使用する以外に、単独処理を行うこともできます。

・サーバー用  
ここまでで触ったように、通常のサーバーのように仮想環境でアプリケーションサーバを運用することができます。  
Redmine以外にも様々なイメージが[DockerHub](https://hub.docker.com/)に用意されています。

---

### コンテナの用途(2/2)

・単独処理用  
docker runで**--rm**オプションを指定すると、コンテナ終了時にコンテナを自動的に削除することができます。
このオプションを利用することで、サーバーとしてではなく、コンテナを使用して単独処理を行うことができます。

例）ボリュームのバックアップ  
(弊社プロダクト[sit-ds](https://github.com/sitoolkit/sit-ds)で使用)

```
docker run --rm \                                     # --rmオプションを指定してコンテナ起動
  -v ${volume_name}:/target \                         # バックアップするボリュームをコンテナ：/targetにマウント
  -v ${LOCAL_BACKUP_DIR}:/backup \                    # ホストOSのバックアップ先ディレクトリをコンテナ：/backupにマウント
  ubuntu \                                            # ubuntuイメージを使用
  tar cfz /backup/${volume_name}.tar.gz -C /target .  # tarコマンドで/targetディレクトリを/backupにバックアップ
```

---

## イメージ・Composeの作成

---

### イメージの作成(1/3)

以下のコマンドでJenkinsを起動してください。

```
$ docker run -d -p 8080:8080 jenkins/jenkins:alpine
```

起動後、[http://localhost:8080](http://localhost:8080)にアクセスしてください。  
Jenkinsの初期設定が始まります。

様々なプロジェクトで開発用サーバーを運用する場合、構築の都度初期設定するのは作業コストがかかります。  
そこで、管理者ユーザ作成済み、初期設定をスキップするカスタマイズイメージを作成します。

初期設定画面を閉じて、Jenkinsのコンテナ停止・削除してください。

---

### イメージの作成(2/3)

<div style="font-size:0.8em;">

イメージ作成用ファイルをまとめて管理するため、**myjenkins**ディレクトリを作成してください。

Jenkinsの初期設定スクリプトを用意します。
下記内容をコピーして、**myjenkins**に**create-admin.groovy**という名前で保存してください。

```
import jenkins.model.*
import hudson.security.*

def adminUsername = "admin"
def adminPassword = "admin"

def instance = Jenkins.getInstance()

def hudsonRealm = new HudsonPrivateSecurityRealm(false)
hudsonRealm.createAccount(adminUsername, adminPassword)
instance.setSecurityRealm(hudsonRealm)

def strategy = new FullControlOnceLoggedInAuthorizationStrategy()
strategy.setAllowAnonymousRead(false)
instance.setAuthorizationStrategy(strategy)

instance.save()
```

</div>

---

### イメージの作成(3/3)

<div style="font-size:0.8em;">

イメージは**Dockerfile**というファイルで作成します。
下記内容をコピーして、**myjenkins**に**Dockerfile**という名前で保存してください。

```
FROM jenkins/jenkins:alpine
ENV JAVA_OPTS=-Djenkins.install.runSetupWizard=false
COPY create-admin.groovy /usr/share/jenkins/ref/init.groovy.d/
```

上記Dockerfileの内容の詳細は
[Dockerfileリファレンス](http://docs.docker.jp/engine/reference/builder.html)
を参照してください。  
Dockerfileを作成したら、以下のコマンドでイメージを作成することができます。

```
$ docker build -t myjenkins:latest myjenkins/
```

**docker images**で**myjenkins**イメージが作成されていることを確認してください。  
イメージからコンテナを起動する方法はこれまでと変わりません。
以下コマンドでmyjenkinsのコンテナを起動して、[http://localhost:8080](http://localhost:8080)にアクセス、
ID:admin/PW:adminでログインしてください。

```
$ docker run -d -p 8080:8080 myjenkins
```

</div>

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

### ご清聴ありがとうございました。

### ハンズオンお疲れさまでした。
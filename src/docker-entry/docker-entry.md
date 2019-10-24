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
 - イメージとコンテナ
 - ボリューム
- Dockerとは
- ハンズオン：Dockerの基本操作
- コンテナの用途
- ハンズオン：イメージ・composeの作成
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

### イメージとコンテナ(1/4)

環境準備に記載したコマンドで、DockerイメージをDockerにインストールしました。  
インストールしたイメージは以下のコマンドで確認できます。

```
$ docker images
```

以下のコマンドでインストールしたRedmineを起動できます。

```
$ docker run -d -p 8080:3000 redmine:alpine
```

このコマンドで起動したRedmineには[http://localhost:8080/](http://localhost:8080/)でアクセスできます。

デフォルトではID:admin/PW:adminでログインできます。  
Redmineにログインして、任意のプロジェクトを作成してください。

---

### イメージとコンテナ(2/4)

<div style="font-size:0.9em">

Dockerでは構築した仮想環境をコンテナと呼びます。  
コンテナは以下のコマンドで確認できます。

```
$ docker ps
```

以下のコマンドでコンテナを停止できます。
[コンテナID]には**docker ps**で表示した**CONTAINER ID**を指定します。  
Redmineを一度停止して、再度[http://localhost:8080/](http://localhost:8080/)にアクセスしてください。

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

### イメージとコンテナ(3/4)

Redmineを起動し、再度[http://localhost:8080/](http://localhost:8080/)にアクセスして、
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

### イメージとコンテナ(4/4)

もう一度コンテナを起動して、Redmineにアクセスしてください。
Redmineにはアクセスできますが、作成したプロジェクトは存在しません。

コンテナを削除すると保存された内容もあわせて削除されます。
イメージとはコンテナを起動するベースになるもので、
同一のイメージから再度コンテナを起動しても、先に存在したコンテナとは別物になります。

ここで、コンテナの停止・削除をしてください。

---

### ボリューム(1/2)

仮想環境を運用する場合、データの永続化が必要となります。  
Dockerではデータ永続化のためのボリュームという機能があります。  
以下のコマンドでRedmineを起動してください。

```
$ docker run -d -p 8080:3000 -v redmine_data:/usr/src/redmine redmine:alpine
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

---

## コンテナの用途

---

### コンテナの用途(1/2)

Dockerではコンテナをサーバーとして使用する以外に、単独処理を行うこともできます。

・サーバー用  
ここまでで触ったように、通常のサーバーと同様にログインしての作業以外にも、
コンテナの外からファイル転送やログの確認などができます。  
Redmine以外にも、DBやLDAP、SMTPなど、様々なイメージが[DockerHub](https://hub.docker.com/)に用意されています。

---

### コンテナの用途(2/2)

・単独処理用  
docker runで**--rm**オプションを指定すると、コンテナ終了時にコンテナを自動的に削除することができます。
このオプションを利用することで、サーバーとしてではなく、コンテナを使用して単独処理を行うことができます。

例）ボリュームのバックアップ(弊社プロダクト[sit-ds](https://github.com/sitoolkit/sit-ds)で使用)

```
docker run --rm \                                     # --rmオプションを指定してコンテナ起動
  -v ${volume_name}:/target \                         # バックアップするボリュームをコンテナ：/targetにマウント
  -v ${LOCAL_BACKUP_DIR}:/backup \                    # ホストOSのバックアップ先ディレクトリをコンテナ：/backupにマウント
  ubuntu \                                            # ubuntuイメージを使用
  tar cfz /backup/${volume_name}.tar.gz -C /target .  # tarコマンドで/targetディレクトリを/backupにバックアップ
```

---

## 以下作成中

---

## ハンズオン

## イメージ・Composeの作成

---

### イメージの作成

<div style="font-size:0.8em;">

Dockerイメージは、Dockerfileを用意することで自分で作成することができます。  
イメージの作成には**Dockerfile**を作成する必要があります。  
以下、nginxにbashをインストール、独自ページを追加するDokcerfileの例です。

```
FROM nginx:alpine                      # 継承元のイメージを指定
RUN apk add bash                       # コマンドを実行(本例ではbashを追加でインストール）
ADD mypage.html /usr/share/nginx/html  # ファイルを追加
CMD ["nginx", "-g", "daemon off;"]     # コンテナ起動時のコマンドを指定
```
※参考：[Dockerfileリファレンス](http://docs.docker.jp/engine/reference/builder.html)

Dockerfileを作成したら、以下のコマンドでイメージを作成することができます。


```
$ docker build -t [REPOSITORY:TAG] [Dockerfile配置ディレクトリ]
※-tオプションは任意のため指定しなくてもよいが、
ランダムなイメージIDで識別が必要になるので、オプションの使用を推奨
REPOSITORY：リポジトリ名、nginx:alpineの場合のnginxにあたる
TAG：タグ、nginx:alipineのalpineにあたる
```

</div>

---

### Composeの作成

Composeとは、複数のDockerコンテナを使用するDockerアプリケーションを実行するツールです。  
アプリケーションの定義には、docker-compose.ymlファイルを作成する必要があります。  
参考：[Composeファイルリファレンス](http://docs.docker.jp/compose/compose-file.html)

作成したdocker-compose.yml定義のアプリケーションは、以下のコマンドで起動します。

```
$ docker-compose up -d
※docker-compose.yml配置ディレクトリで実行
```

docker-composeが実行できるコマンドは概ねdockerコマンドと同様です。    
参考：[docker-composeコマンド概要](http://docs.docker.jp/compose/reference/overview.html)

---

### ハンズオン(1/2)

#### Dockerイメージの作成
  
以下のイメージを作成する。
* ベースイメージ - nginx:alpine
* 独自のページを事前に追加する
* Listenポートを8080とし、コンテナへの解放ポートも8080とする  
* docker run -d -p 8080:8080 [イメージ]でコンテナを起動、http://localhost:8080/[任意ページ] でアクセスを確認

---

### ハンズオン(2/2)

#### Composeの作成

<div style="font-size:0.8em;">

以下のComposeを作成する。
* ベースイメージ1 - httpd:alpine
 * 独自のページを事前に追加する
 * httpdのListenポートを8080とし、直接アクセスはできない設定とする

* ベースイメージ2 - nginx:alpine
 * nginxのListenポートを80として、/へのアクセスをhttpd:8080へproxyする設定ファイルを追加する
 * 直接アクセスできる設定とする

* 動作確認
 * http://localhost:8080/[任意ページ] → アクセスNG
 * http://localhost/[任意ページ] → アクセスOK

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
 * バックアップ・リストアシェルを用意

---

### ご清聴ありがとうございました。

### ハンズオンお疲れさまでした。
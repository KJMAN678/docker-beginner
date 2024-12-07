- [入門 Docker](https://y-ohgi.com/introduction-docker/) をやった
- 環境
  - Docker DeskTop 4.36.0
  - M1 Mac Sequoia 15.1.1

```sh
# -P でポートを公開して起動
$ docker run -P nginx

Docker DeskTop の下記赤枠のリンクをクリックすると、nginx のローカルサーバーを確認できる
```

<img src="./nginx.png" width=300px>

```sh
# 実際のローカルサーバーのURL
http://localhost:55000/

# UbuntuのDocker ImageをDockerHub上から取得
$ docker pull ubuntu

# Docker上でUbuntuを実行し、Ubuntuが動いていることを確認
$ docker run ubuntu cat /etc/issue
Ubuntu 24.04.1 LTS \n \l

# bash を使用してコンテナに入る
$ docker run -i -t ubuntu bash

# コンテナ内で書きを実行し、Ubuntuが動いていることを確認
$ cat /etc/issue

# コンテナから脱出
$ exit

-i はコンテナの標準入力を有効化、 -t はttyを有効化するためのオプションです。
docker run -it ... のように省略することも可能
```

### 好きな言語でDockerを起動する
```sh
# ruby のイメージを検索
$ docker search ruby

# ruby イメージを使用して標準出力を出力する
$ docker run ruby ruby -e 'puts "Hello, Docker!"'

# CentOS のコンテナを起動し、コンテナに入る
$ docker run -it centos bash

# Node.js のバージョン表示
$ docker run node node --version

$ docker run node node -p 'console.log("hoge")'

$ docker run python python --version

$ docker run python python -c 'print("hoge")'
```

### tag を使用する
```sh
$ docker run python:2.7 python --version
```
Dockerは公式で3パターンの命名規則を定義しています。

- Docker公式のイメージ
  - <イメージ名>:<タグ>
- ユーザーが作成したイメージ
  - <ユーザー名>/<イメージ名>:<タグ>
- 非公式レジストリのイメージ
  - <レジストリ名>/<ユーザー名>/<イメージ名>:<タグ>
  - :<タグ> の指定をしない場合は :latest タグが自動的に付与されます。
  - また、DockerHubを使用しない場合(ECRやGCRなどの非公式レジストリ)はtagの設定が必須になります(3番のパターンを使用します)。

### Dockerfile

```sh
cd docker/ubuntu

# docker build コマンドで Dockerfile からDocker Image を作成
# -t hello オプションは「Docker Imageを hello という名前にする」という意味です。
# . はdocker build 実行時のコンテキストの指定です。 . は COPY コマンドを実行する際にどのディレクトリを起点とするかを指定します。

$ docker build -t hello .

# Docker Image がビルドできたか確認
$ docker images

# Docker Image を実行
$ docker run --rm hello
```

### Docker Hub へ登録
```sh
# Docker Hub へログイン
$ docker login

# Docker Image の命名
$ docker tag hello <USER NAME>/hello

# Docker Image の確認
$ docker images

# Docker Image を Docker Hub にアップロードする
$ docker push <USER NAME>/hello

# Docker Hub から作成したイメージを取得
$ docker pull <USER NAME>/hello

# Docker イメージの実行
docker run <USER NAME>/hello
```

Dockerfileには17のコマンドが用意されています。

FROM , ENV , WORKDIR , RUN , ADD , ARGS , CMD , ONBUILD , LABEL , ENTRYPOINT ,
STOPSIGNAL , MAINTAINER , VOLUME , HEALTHCHECK , EXPOSE , USER , SHELL

MAINTAINER コマンドは非推奨です。

基本的なコマンド

- FROM
  - ベースとなるDocker Image（ベースイメージ）
  - ベースイメージはDocker HubやGoogle Cloudなどの信頼できるレジストリのイメージを使用するのが一般的
- ENV
  - Docker内で使用する環境変数を定義
- WORKDIR
  - Dockerfileでコマンドを実行する際に基準となるディレクトリを設定
  - この際存在しないディレクトリを指定すると自動的にディレクトリが作成される
- COPY
  - Docker内へホストのファイル/ディレクトリをコピーする
  - COPY `ホスト側のディレクトリ` `Docker側のディレクトリ`
  - ホスト側のディレクトリは docker build . で指定したディレクトリ。`.`の場合はカレントディレクトリ
  - Docker側はデフォルトのパス、もしくは WORKDIR で定義されたディレクトリを参照する
- RUN
  - Docker内でコマンドを実行する
- USER
  - イメージを起動時に使用するユーザーを指定する
  - デフォルトは root が設定されているため、セキュリティリスクを回避するために別のユーザーを指定するのが良い
- EXPOSE
  - コンテナ起動時に公開するポート
  - EXPOSE を記載することで他の人から「このDockerはどのポートを使用するのか」がわかりやすくなるため、記述すると丁寧
- CMD
  - Docker起動時にデフォルトで実行されるコマンド

### Container

```sh
# nginxのDockerコンテナをデーモンとして起動して、 pstree で確認
# pstreeは、プロセスリストをツリーとして表示する小さなプログラム
$ docker run -d nginx

https://formulae.brew.sh/formula/pstree
$ pstree
```

### ネットワーク

- nginxとphp-fpmのように複数プロセスを協調して動かす必要がある時はソケットではなく、ネットワークで通信を行うことが推奨されている。
- Dockerでのネットワークは特にKubernetes・ECS・docker-composeのような各種オーケストレーションツールを使用する際に意識する必要がある

### Driverの種類

- Dockerはネットワークの振る舞いを定義することが可能で、デフォルトでは2種類のNetwork Driver が存在する

1. bridge
Dockerを使用する際は基本的にこのNetwork Driverが使用されます。
Linuxカーネルのbridgeネットワークを使用するための機能

また、何も指定せずDocker Container を起動すると docker0 という名前のbridgeネットワークに所属します。

2. host
ホストマシンのeth0を直接使用する方法です。

(3. none )
どのDriverも使用せず、起動したコンテナをネットワークに所属させないための設定です。

### ネットワークを試す

```sh
# 現在Dockerが管理しているNetwork一覧を出力
$ docker network ls

# ホスト側のネットワークの確認
https://formulae.brew.sh/formula/iproute2mac
$ ip a

# 新しいネットワークの作成
$ docker network create myapp

# ネットワーク一覧をみると、myapp が増えているのがわかる
$ docker network ls

# 通信を受けるためのサーバーとしてnginxを構築する
$ docker run --name nginx --network=myapp -d nginx

# AmazonLinux2を起動し、Nginxコンテナへ接続する
# Nginxと疎通できるか myapp ネットワーク内にAmazonLinux2 イメージでコンテナを起動し、 curl を実行してみる。
$ docker run --network=myapp -it amazonlinux:2 curl nginx:80

# 新しくネットワークを作成し、疎通できないことを確認する
# myapp2 というネットワークを作成し、 nginx2 という命名でnginxを起動する。
$ docker network create myapp2
$ docker run --name nginx2 --network=myapp2 -d nginx

# myapp ネットワークに所属しているAmazonLinux2からcurlを実行し、疎通できないことを確認する
$ docker run --network=myapp -it --rm amazonlinux:2 curl nginx2:80
```

### Volume
ボリュームはデータを永続化するための機能
コンテナのライフサイクルとは独立してファイルの管理を行う

Volumeは2つの種類が存在します。

Data Volume
Docker Container のライフサイクルの外で管理されるファイル/ディレクトリの設定です。

```sh
# /tmp/text をボリュームとして実行後、volumeが作成されたか確認
$ docker run -v /tmp/text --rm ubuntu touch /tmp/text/hogefugapiyo

$ docker volume ls

# 詳細を取得して、volumeの実体を見る
# docker volume inspect コマンドから Mountpoint が取得できる
$ docker volume inspect a775e7f7be4471120a8d8b09d46b0be450422df4c9adf1912f78a7cb85d42ef4

# Mountpoint にアクセスする
$ docker run --rm -v a775e7f7be4471120a8d8b09d46b0be450422df4c9adf1912f78a7cb85d42ef4:/volume ubuntu ls -l /volume

# Data Volume Container
# 他のDocker Container で指定されているVolumeを参照するための機能

# 参照元となるコンテナを volume-test という名前で作成し、その中でファイルを3つ作ってみる
$ docker run --name volume-test -v /tmp/test ubuntu touch /tmp/test/{hoge,fuga,piyo}

# まずはオプションを設定せずに ls を実行し、存在しないことを確認する
$ docker run --rm ubuntu ls -l /tmp/test

# 次に --volumes-from オプションで先程作成した volume-test を指定して、 ls を実行する
$ docker run --volumes-from volume-test --rm  ubuntu ls -l /tmp/test
```

### セキュリティ

rootユーザを使わない
USER コマンド使用してユーザーを変更し、chmodなどで権限を最適化すべし

公式のベースイメージのデフォルトユーザーを確認
```sh
$ cd docker/secirity/root_user
$ docker build -t whoami .
$ docker run --rm whoami
```

USER コマンドを使用しユーザーを設定してみる
```sh
$ cd docker/secirity/node_user
$ docker build -t whoami-node .
$ docker run --rm whoami-node
```

野良のDockerイメージをベースイメージにしない

ビルド時に機微情報を与えない
ビルド時にパスワードや秘密鍵のような機微情報を与え、最終イメージに残らないようにしましょう。
基本的にビルド後に環境変数としてパスワードなどの機微情報を渡すことがベストプラクティスです。

.dockerignore ファイルでローカルの不要なパスを無視する
.git や node_modules のようなビルド時に不要なパスを記述することでビルドが高速かつイメージが軽量になるメリットがある

```sh
# BuildKitを無効化し、ビルドを実行
$ DOCKER_BUILDKIT=0 docker build -t pass .

Step 2/4 : RUN echo "mypassword" > /tmp/pass.txt
 ---> Running in fugafuga
 ---> Removed intermediate container fd58e37e1aa3
 ---> hogehoge

# hogehoge の部分のレイヤーを指定して /tmp/pass.txt の中身を見ると、パスワードが表示されてしまう
$ docker run --rm <hogehoge 部分> cat /tmp/pass.txt
```
### マルチステージビルド

イメージは軽量であることが重要です。
コンテナを起動するためにはレジストリからイメージをpullしてから立ち上げる必要がありますが、巨大なイメージはpullに時間がかってしまいコンテナを起動するためのリードタイムが長くなります。

リードタイムが長くなることにより様々なリスクが伴います。
* デプロイ・ロールバックが遅くなる。
* スケールアウトが遅くなりコンテナの起動が間に合わずにリクエストを捌けなくなる。
* 巨大なイメージを保存するためにレジストリの保存領域もしくは保存料が高くなる。

軽量なイメージをビルドするための機能がマルチステージビルドで、複数のイメージから1つのイメージをビルドするための技術

例えばgolangであればバイナリファイルのみで実行が可能なため、ビルドレイヤーでバイナリを生成し、最終イメージは軽量なベースイメージを使用することで軽量なイメージをビルドすることが可能

バイナリをビルドする Builderイメージ と、Builderイメージで生成したバイナリをコピーし実行する Runnerイメージ の2つからなるマルチステージビルドの例

これにより、最終イメージは Builderイメージ のバイナリを所有した軽量な Runnerイメージ が作成できる
```sh
$ cd docker/multi-stage-build

$ go mod init multi-stage-build
$ go mod download

$ docker build -t multistage-build .
$ docker run --rm multistage-build
```

--target オプションで中間イメージを指定することで、中間イメージのみをビルドすることも可能

```sh
$ docker buildx build -t multi-builder --target builder .
$ docker run --rm -p 8080:8080 multi-builder ./main
```

### 中間レイヤーでキャッシュを行う¶

Dockerではビルドの高速化を行うため、中間レイヤーに変更が無ければ古い中間レイヤーをキャッシュとして使用することが可能です。

開発中にライブラリは頻繁に追加しませんが、コードは頻繁に変更します。
以下のDockerfileの例だと毎回ライブラリをインストールを行なってしまい、ビルド時に生成される中間レイヤーを有効活用できません。

### docker compose

docker compose(以下compose)は複数のローカル開発環境で複数のコンテナを協調させるための機能

```sh
$ cd docker-compose

$ docker compose up -d

$ curl localhost:3000

$ docker compose down
```

compose内のコンテナ間通信

compose内では独自のBright networkを作成します。
そのnetwork内では services で命名したコンテナ名をホスト名と使用し通信可能です。
例えば今回の例であれば、 postgres に接続したい場合、 postgresql://postgres:postgres@postgres:5432/db というuriを使用することになります。

#### services
services: はそのコンテナ群を定義するブロック
このコンテナ名を元にコンテナ間通信を行う
そのため、同じコンテナ名を付けることはできない
```yaml
services:
  <CONTAINER_1>:
    :

  <CONTAINER_2>:
    :
```

#### image
使用するイメージを宣言するブロック
postgresやmysqlのようなイメージは自身でDockerfileを作成せず、Docker Hubの公式のイメージを使用することが一般的
```yaml
services:
  postgres:
    image: postgres
```

#### build
Dockerfileを使用しイメージのビルドを行い、そのイメージを使用することを宣言するブロック
Next.jsやLaravelなど、アプリケーションコードをDockerfileで管理している場合に使用
この際それぞれのディレクトリにDockerfileが含まれている必要がある
また、 target を使用することでマルチステージビルドのステージを指定することも可能
```yaml
services:
  front:
    build:
      context: ./front

  api:
    build:
      context: ./api
      target: builder

  postgres:
    image: postgres
```

#### volumes
コンテナにボリュームマウントするためのブロック
カレントディレクトリをコンテナの特定のパスと同期させたり、Docker Volumeを使用することが可能
```yaml
services:
  next:
    build: .
    volumes:
      - .:/app
      - node_modules:/app/node_modules

volumes:
  node_modules:
```

#### ports
コンテナのポートをホストに公開する宣言を行うブロック
```yaml
services:
  postgres:
    image: postgres
    ports:
      - 5432:5432
```

#### environment
コンテナ内に環境変数の定義を行うブロック
postgresやmysqlのようなイメージは特定の環境変数を宣言することでユーザー・パスワード・データベースの作成をサポートしている
```yaml
services:
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: next
```

#### env_file
特定のファイルを元にコンテナ内に環境変数の定義を行うブロック
dotenvのようなライブラリを使用している場合、 env_file で代替可能
```yaml
services:
  postgres:
    image: postgres:16-alpine
    env_file:
      - .env
```

#### command
任意のコマンド( CMD )を上書きするブロック
CMD で宣言されているコマンドを上書きし、任意のコマンドを実行可能
```yaml
services:
  next:
    build: .
    command: npm install & npm run dev
```

#### depends_on & healthcheck
depends_on は依存関係を示すもので、特定のコンテナの起動を前提として起動を行うためのブロック
また、 healthcheck ブロックを併用することでコンテナが動いていることを確認できてから順番に起動することが可能になる
```yaml
  next:
    # postgresの起動を待つ
    depends_on:
      postgres:
        condition: service_healthy

  postgres:
    image: postgres:16-alpine
    # ヘルスチェック
    healthcheck:
      test: ["CMD-SHELL", "pg_isready"]
      interval: 10s
      timeout: 5s
      retries: 5
```

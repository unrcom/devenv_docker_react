# devenv_docker_react

React + Redux Toolkit + TypeScript　を用いた Webフロントエンドの開発環境を　Docker container 上に構築する手順です

## プロジェクトフォルダを作成する

firebase_app というプロジェクトの環境を構築する場合の手順です (firebase_app　という記述は、プロジェクトごとに違う名称に置き換えてください)

- /Users/r2　は私のPCのホームディレクトリですが「デスクトップ」など任意のフォルダで作業を進めてください
- firebase_app　の記述は、プロジェクトごとに置き換えてください

```
$ cd
$ pwd
/Users/r2
$ mkdir firebase_app
$ cd firebase_app
$
```

## docker-compose.yml

- プロジェクトフォルダの直下に docker-compose.yml ファイルを配置します

```
$ pwd
/Users/r2/firebase_app
$ ls
docker-compose.yml
$ cat docker-compose.yml
version: '3'

services:
  node:
    image: node:current-slim
    container_name: 'firebase_app'
    hostname: 'firebase_app'
    ports:
      - '3001:3000'
    stdin_open: true
    tty: true
    working_dir: '/usr/src/app'
    volumes:
      - ./app:/usr/src/app
    networks:
      - dumynet
networks:
  dumynet:
    external: true
$
```

- container_name と hostname は プロジェクト名 + コンテナの役割 で命名します
- image で指定する node.js のバージョンは、本番環境構築時に固定します
- ports の 3001 は、PC側のポートを指定しているので、空いているポートを指定します
- volumes　の ./app は PC側のプロジェクトフォルダに app フォルダが作成され、コンテナの　/usr/src/app　フォルダをマウントします
- dumynet　はコンテナ間の通信に使用されるネットワークでその名称は任意ですが、あらかじめ docker network create コマンドで作成しておく必要があります

## ネットワークの確認

```
$ docker network ls
NETWORK ID     NAME                                   DRIVER    SCOPE
657eedb267a7   anaconda_default                       bridge    local
482cde19861a   bridge                                 bridge    local
02c5888e412d   mernnet                                bridge    local
26aa95ba6cca   mongo_default                          bridge    local
e5bee0a82a30   nginx_default                          bridge    local

(snip)

$
```

dumynet は存在しないので作成します

```
$ docker network create dumynet
9768464e81767fc54b5b7def8a513624ab4f40a390ecca86c40a5441796a2008
$
$ docker network ls | grep dumynet
9768464e8176   dumynet                                bridge    local
$
```

## コンテナ起動

```
$ pwd
/Users/r2/firebase_app
$ ls
docker-compose.yml
$ docker-compose up
Pulling node (node:current-slim)...
current-slim: Pulling from library/node
07aded7c29c6: Pull complete
92706508d124: Pull complete
29e8836fa52b: Pull complete
c69c406f761b: Pull complete
1ad0d9e4f0be: Pull complete
Digest: sha256:283d85e5a64183046abad478f5f98428720c1b30a72cc11d0cd1cedc1cb53493
Status: Downloaded newer image for node:current-slim
Recreating firebase_app ... done
Attaching to firebase_app
firebase_app | Welcome to Node.js v16.10.0.
firebase_app | Type ".help" for more information.
```

- docker-compose up コマンドは　-d オプションをつけるとバックグラウンドで実行できますが、開発中はフォアグラウンドで実行するとこにより、システムが出力するなんらかのメッセージを発見するチャンスを残しておきます

### コンテナ状態確認

```
$ docker container ls
CONTAINER ID   IMAGE               COMMAND                  CREATED          STATUS          PORTS                                       NAMES
014db5f60eda   node:current-slim   "docker-entrypoint.s…"   46 seconds ago   Up 44 seconds   0.0.0.0:3001->3000/tcp, :::3001->3000/tcp   firebase_app
$
```

- マウントボリュームが作成されているか確認する

```
$ pwd
/Users/r2/firebase_app
$ ls -l
total 8
drwxr-xr-x  2 r2  staff   64 10  9 13:13 app
-rw-r--r--  1 r2  staff  333 10  9 13:10 docker-compose.yml
$
```

## 実行中のコンテナに開発で必要になるモジュールをインストールする

### 実行中のコンテナの bash を起動してコマンド入力待ちにする

```
$ docker container ls
CONTAINER ID   IMAGE               COMMAND                  CREATED         STATUS         PORTS                                       NAMES
014db5f60eda   node:current-slim   "docker-entrypoint.s…"   6 minutes ago   Up 6 minutes   0.0.0.0:3001->3000/tcp, :::3001->3000/tcp   firebase_app
$
$ docker container exec -it firebase_app /bin/bash
root@firebase_app:/usr/src/app# export PS1='\h\$ '
firebase_app#
```

- bash 起動後にプロンプトをシンプルなものに変更しています

### React 開発環境のインストール

```
firebase_app# pwd
/usr/src/app
firebase_app# npx create-react-app . --template redux-typescript --use-npm
Need to install the following packages:
  create-react-app
Ok to proceed? (y) y

(snip)

Success! Created app at /usr/src/app
Inside that directory, you can run several commands:

  npm start
    Starts the development server.

  npm run build
    Bundles the app into static files for production.

  npm test
    Starts the test runner.

  npm run eject
    Removes this tool and copies build dependencies, configuration files
    and scripts into the app directory. If you do this, you can’t go back!

We suggest that you begin by typing:

  cd /usr/src/app
  npm start

Happy hacking!
npm notice
npm notice New major version of npm available! 7.24.0 -> 8.0.0
npm notice Changelog: https://github.com/npm/cli/releases/tag/v8.0.0
npm notice Run npm install -g npm@8.0.0 to update!
npm notice
firebase_app#
```

#### 開発サーバ起動

```
firebase_app# pwd
/usr/src/app
firebase_app# ls
README.md  node_modules  package-lock.json  package.json  public  src  tsconfig.json
firebase_app#
firebase_app# npm start

> app@0.1.0 start
> react-scripts start

ℹ ｢wds｣: Project is running at http://172.31.0.2/
ℹ ｢wds｣: webpack output is served from
ℹ ｢wds｣: Content not from webpack is served from /usr/src/app/public
ℹ ｢wds｣: 404s will fallback to /
Starting the development server...
Compiled successfully!

You can now view app in the browser.

  Local:            http://localhost:3000
  On Your Network:  http://172.31.0.2:3000

Note that the development build is not optimized.
To create a production build, use npm run build.
```

#### 動作確認

ブラウザから localhost:3001 にアクセスして React Redux のテンプレート画面が表示されれば OK (はじめてこの画面を見る方は、すべてのボタンを押して動作結果を確認しておきましょう)

以下はプロジェクトに応じて必要なモジュールが異なりますので、必要に応じてインストールすることになります

### Material UI

一番人気の UI コンポーネントライブラリです

先日 Version　5 がリリースされましたが当面は Version　4　でやっていこうと思っていますので、以下は v4 のインストール手順になります (v4 と v5 は同一の React コンポーネントで共存できそうですので、機会があればトライしてみます)

```
firebase_app# pwd
/usr/src/app
firebase_app# ls
README.md  node_modules  package-lock.json  package.json  public  src  tsconfig.json
firebase_app# npm i @material-ui/core @material-ui/icons @material-ui/lab

added 26 packages, and audited 2046 packages in 4m

158 packages are looking for funding
  run `npm fund` for details

66 vulnerabilities (24 moderate, 40 high, 2 critical)

To address issues that do not require attention, run:
  npm audit fix

To address all issues (including breaking changes), run:
  npm audit fix --force

Run `npm audit` for details.
firebase_app#
```

### axios

Rest API とメッセージをやりとりする場合は、なんらかの HTTP クライアントを使用しますが axios を使った非同期通信を採用しタイト思います

```
firebase_app# pwd
/usr/src/app
firebase_app# ls
README.md  node_modules  package-lock.json  package.json  public  src  tsconfig.json
firebase_app# npm i axios

added 1 package, and audited 2047 packages in 21s

158 packages are looking for funding
  run `npm fund` for details

66 vulnerabilities (24 moderate, 40 high, 2 critical)

To address issues that do not require attention, run:
  npm audit fix

To address all issues (including breaking changes), run:
  npm audit fix --force

Run `npm audit` for details.
firebase_app#
```

### react-router-dom

本プロジェクトでは使用しませんが、URLパスを使用して Reactコンポーネントを制御するには react-router-dom が必要にある場合があります

インストールは以下のようになります

```
firebase_app# pwd
/usr/src/app
firebase_app# ls
README.md  node_modules  package-lock.json  package.json  public  src  tsconfig.json
firebase_app#
firebase_app# npm i react-router-dom @types/react-router-dom

added 15 packages, and audited 2062 packages in 28s

158 packages are looking for funding
  run `npm fund` for details

66 vulnerabilities (24 moderate, 40 high, 2 critical)

To address issues that do not require attention, run:
  npm audit fix

To address all issues (including breaking changes), run:
  npm audit fix --force

Run `npm audit` for details.
firebase_app#
```

#### 動作確認

ここで再度ブラウザから localhost:3001 にアクセスして React Redux のテンプレート画面が表示されるか確認してみましょう

end of md, thank you for your attention

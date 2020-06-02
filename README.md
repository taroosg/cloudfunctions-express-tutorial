# Cloud Functions + ExpressでサーバレスAPIを実装

作成日：2020/02/16

更新日：2020/06/02

## 今日のゴール

- Cloud FunctionsでExpressを用いたNode.jsを実装し，動作を確認する．
- Cloud Functionsを使った開発の手順を把握する．
- Node.jsの開発手順の感覚を掴む．


## 今回実装するアプリケーション

- Cloud Functions上でGoogle books APIから情報を取得する．
- クライアントから送信されてきたキーワードを受け取り，APIに投げる．
- APIから返ってきたデータをクライアントに送信する．
- Cloud Functionsを利用することでサーバを用意することなくAPIを実装！


## Cloud Functionsの特徴と利点

### 概要 / 特徴

- イベントドリブンのサーバーレスコンピューティングプラットフォーム．
- Firebase上でNode.jsで記述した関数を実行することができる．

### 利点

- サーバーのプロビジョニング，管理，アップグレードが不要．
- 負荷に応じた自動スケーリング．
- 異なる言語にまたがる，複雑なアプリケーション開発を簡素化．

[参考（ドキュメント）](https://firebase.google.com/docs/functions?hl=ja)

## Expressの特徴と利点

### 概要 / 特徴

- 最小限で柔軟な Node.js Web アプリケーション・フレームワーク．
- 無数のHTTP ユーティリティー・メソッドとミドルウェアを自由に使用できるため，堅固な API を迅速かつ容易に作成できる．
- ほぼデファクトスタンダード．

### 利点

- Routingの設定が非常にわかりやすい．
- 環境構築が楽．
- 情報がとにかく多い．

[参考（ドキュメント）](https://expressjs.com/ja/)


## 環境構築

### 必要なツールのバージョン確認

- Node.jsとnpmが必要なので，以下のコマンドで状況を確認する．
- バージョンが表示されればOK．

```bash
$ node -v
v12.15.0
$ npm -v
6.14.5
```

### Firebaseのプロジェクト作成

- Firebaseのコンソールにログインし，新規プロジェクトを作成する．
- プロジェクト名は任意（今回は`20200601-functions`）．
- DBなどは特に設定しなくてOK（下記画面が表示された段階でOK）．

![firebaseプロジェクト画面](./images/project_view01.png)

### Fiirebaseを扱うツールのインストール

- firebase関連のコマンドを実行するため，下記のコマンドでインストールする．
- `-g`をつけてグローバルにインストールする．
- すでにインストールしている場合も，下記コマンドで最新版にアップデートできるため必ず行う．

```bash
$ npm install -g firebase-tools
```

実行結果

```bash
+ firebase-tools@8.4.1
updated 6 packages in 20.484s
```

【注意】バージョンが`8.4.0`の場合は後々エラーが発生して先へ進めなくなるので必ず`8.4.1`以上にしておくこと．

### 雛形の作成

- 適当な場所にディレクトリを作成し，ターミナルで移動して必要なファイルを準備する．
- 今回は例としてデスクトップに`20200601cloudfunctions`ディレクトリを作成する．
- 下記コマンドを順番に実行．

```bash
$ cd ~/Desktop
$ mkdir 20200601cloudfunctions
$ cd 20200601cloudfunctions
$ firebase init
```

- 選択肢が出るので，十字キーで`Functions`を選択してスペースキーでチェックを入れる（下図参照）．
- チェックを入れたらEnter．

```bash
? Which Firebase CLI features do you want to set up for this folder? Press Space
 to select features, then Enter to confirm your choices.
 ◯ Database: Deploy Firebase Realtime Database Rules
 ◯ Firestore: Deploy rules and create indexes for Firestore
❯◉ Functions: Configure and deploy Cloud Functions
 ◯ Hosting: Configure and deploy Firebase Hosting sites
 ◯ Storage: Deploy Cloud Storage security rules
 ◯ Emulators: Set up local emulators for Firebase features
```

- 続いて，以下の選択肢が表示される．
- `Use an existing project`を選択してEnter．

```bash
? Please select an option: (Use arrow keys)
❯ Use an existing project
  Create a new project
  Add Firebase to an existing Google Cloud Platform project
  Don't set up a default project
```

- プロジェクトの選択肢が出るので，上で作成したプロジェクトを選択してEnter．

```bash
? Select a default Firebase project for this directory:
  hoge-c83e4 (hoge)
  hoge-791f2 (hogehoge)
  fuga-813c6 (fuga)
❯ functions-69daf (20200601-functions)
  hoge-216007 (hogefuga)
  piyo (piyo)
  hogefuga (hoge-fuga)
```

- 選択肢が出るので，`JavaScript`を選択してEnter．

```bash
? What language would you like to use to write Cloud Functions? (Use arrow keys)

❯ JavaScript
  TypeScript
```

- 以降は下のような感じ．

```bash
? Do you want to use ESLint to catch probable bugs and enforce style? No
✔  Wrote functions/package.json
✔  Wrote functions/index.js
✔  Wrote functions/.gitignore
? Do you want to install dependencies with npm now? Yes
...
i  Writing configuration info to firebase.json...
i  Writing project information to .firebaserc...
i  Writing gitignore file to .gitignore...

✔  Firebase initialization complete!
```

これで準備完了！


## 動作確認&デプロイ

### ファイルの内容確認&解説

- 必要なファイルが準備されているので，エディタでプロジェクトのフォルダを開く．
- `functions/index.js`を開くと下記のような内容が記述されている．
- 1行目はモジュールの読み込み．
- `helloWorld`は関数名．この関数にリクエストが来ると，`Hello from Firebase!`という文字列を返すよう記述されている．

```js
// functions/index.js
const functions = require('firebase-functions');

// // Create and Deploy Your First Cloud Functions
// // https://firebase.google.com/docs/functions/write-firebase-functions
//
// exports.helloWorld = functions.https.onRequest((request, response) => {
//  response.send("Hello from Firebase!");
// });
```

### 編集&ローカルサーバーでの動作確認

- 下記のように編集する（コメントアウト外すだけ）．

```js
// functions/index.js
const functions = require('firebase-functions');

// // Create and Deploy Your First Cloud Functions
// // https://firebase.google.com/docs/functions/write-firebase-functions
//
exports.helloWorld = functions.https.onRequest((request, response) => {
 response.send("Hello from Firebase!");
});
```

- 専用コマンドが用意されているのでローカルサーバーを立ち上げる．

```bash
$ firebase serve
```

- 実行結果

```bash
=== Serving from '/Users/taroosg/Desktop/20200601cloudfunctions'...

⚠  Your requested "node" version "8" doesn't match your global version "12"
i  functions: Watching "/Users/taroosg/Desktop/20200601cloudfunctions/functions" for Cloud Functions...
✔  functions[helloWorld]: http function initialized (http://localhost:5000/cloudfunctions-3517c/us-central1/helloWorld).

```

- このとき，ローカルサーバのURLが発行されるためメモしておくことをオススメする（あとから確認もできるが煩雑なため）．

- ローカルサーバーが立ち上がったらターミナルからリクエストを送る．
- メッセージ（`Hello from Firebase!`）が返ってくればOK！

```bash
$ curl http://localhost:5000/cloudfunctions-3517c/us-central1/helloWorld
Hello from Firebase!
```

- 確認したら`ctrl + c`でローカルサーバを停止する．


### デプロイ&動作確認

- 動作確認したらデプロイする．
- ターミナルで下記を実行．

```bash
$ firebase deploy
```

- 実行結果
```bash
=== Deploying to 'fir-todo-8868b'...

i  deploying functions
i  functions: ensuring necessary APIs are enabled...
✔  functions: all necessary APIs are enabled
i  functions: preparing functions directory for uploading...
i  functions: packaged functions (26.53 KB) for uploading
✔  functions: functions folder uploaded successfully
i  functions: creating Node.js 8 function helloWorld(us-central1)...
✔  functions[helloWorld(us-central1)]: Successful create operation.
Function URL (helloWorld): https://hogehoge.cloudfunctions.net/helloWorld

✔  Deploy complete!

Project Console: https://console.firebase.google.com/project/fir-todo-8868b/overview
```

- このとき，デプロイ先のURL（`Function URL (helloWorld):...`）が発行されるためメモしておくことをオススメする（あとから確認もできるが煩雑なため）．

- デプロイが完了したらターミナルからリクエストを送る．
- メッセージが返ってくればOK！

```bash
$ curl https://hogehoge.cloudfunctions.net/helloWorld
Hello from Firebase!
```

## `Express`の導入

- ExpressはNode.jsのフレームワーク．
- APIのエンドポイントを手軽に実装できるので便利．
- 下記コマンドを実行してインストールする．
- 【重要】`functions`フォルダに移動しておく．

```bash
$ cd functions
$ npm install express
```

- インストールが終わったらindex.jsを編集する．
- `app.get()`でAPIエンドポイントを定義．
- `/hello`がエンドポイントのURL．リクエスト時に動作させたい関数のエンドポイントを指定する．

```js
// index.js
const functions = require('firebase-functions');
// Expressの読み込み
const express = require("express");

const app = express();

app.get('/hello', (req, res) => {
  // レスポンスの設定
  res.send('Hello Express!');
});

// 出力
const api = functions.https.onRequest(app);
module.exports = { api };
```

- 保存したら動作確認．

```bash
$ firebase serve
=== Serving from '/Users/taroosg/Desktop/20200601cloudfunctions'...

⚠  Your requested "node" version "8" doesn't match your global version "12"
i  functions: Watching "/Users/taroosg/Desktop/20200601cloudfunctions/functions" for Cloud Functions...
✔  functions[api]: http function initialized (http://localhost:5000/cloudfunctions-3517c/us-central1/api).

```

- ターミナルからリクエストを送る．

```bash
$ curl http://localhost:5000/cloudfunctions-3517c/us-central1/api/hello
Hello Express!
```

- 確認したらデプロイ．
- `helloworld`関数を削除していいかどうか訊かれたらyesでOK．

```bash
$ firebase deploy
? Would you like to proceed with deletion? Selecting no will continue the rest o
f the deployments. Yes
i  functions: deleting function helloWorld(us-central1)...
✔  functions[helloWorld(us-central1)]: Successful delete operation.
✔  functions[api(us-central1)]: Successful create operation.
Function URL (api): https://hogehoge.cloudfunctions.net/api

✔  Deploy complete!

Project Console: https://console.firebase.google.com/project/fir-todo-8868b/overview
```

- デプロイが完了しいたらリクエストしてみる．

```bash
$ curl https://hogehoge.cloudfunctions.net/api/hello
Hello Express!
```

これで動作OK！

## `Express`での値の受け取り

### URLのパラメータ取得

- `/user/:userId`のように記述すると，値を受け取ることができる．
- 例えば，`https://hogehoge.cloudfunctions.net/api/user/2`のようにリクエストを送信すると，Expressでは`2`の文字列を取得することができる．
- Express内では`req.params.userId`のように取得する．
- `index.js`を以下のように編集する．

```js
const functions = require('firebase-functions');
const express = require('express');

const app = express();

app.get('/hello', (req, res) => {
  res.send('Hello Express!');
});

// ↓↓↓ エンドポイントを追加 ↓↓↓
app.get('/user/:userId', (req, res) => {
  const users = [
    { id: 1, name: 'ジョナサン' },
    { id: 2, name: 'ジョセフ' },
    { id: 3, name: '承太郎' },
    { id: 4, name: '仗助' },
    { id: 5, name: 'ジョルノ' },
  ];
  // req.params.userIdでURLの後ろにつけた値をとれる．
  const targetUser = users.find(user => user.id === Number(req.params.userId));
  res.send(targetUser);
});

// 以降変更なし
const api = functions.https.onRequest(app);
module.exports = { api };
```

- まずはローカルサーバーで動作確認

```bash
$ firebase serve
```

- ユーザIDを指定してリクエスト送信

```bash
curl http://localhost:5000/cloudfunctions-3517c/us-central1/api/user/3
{"id":3,"name":"承太郎"}
```

- 動作を確認したらデプロイ

```bash
$ firebase deploy
```

- デプロイしたらリクエスト送信

```bash
$ curl https://hogehoge.cloudfunctions.net/api/user/2
{"id":2,"name":"ジョセフ"}
$ curl https://hogehoge.cloudfunctions.net/api/user/5
{"id":5,"name":"ジョルノ"}
```

レスポンスが返ってくれば動作OK．

## Google books APIへのhttpリクエスト

- リクエスト受信時に値を取得できたので，取得した値を用いてNode.jsから外部のAPIにリクエストを送る．
- 例によってGoogle books APIを利用する．
- Node.jsからAPIへリクエストを送信することで，クライアントアプリケーションの処理を単純にすることができる．
- webアプリでもネイティブアプリでも，Node.jsのエンドポイントにリクエストを送信するだけで良い．

### 必要なモジュールのインストール

- Node.jsの標準機能でもhttpリクエストを行えるが，記述が煩雑になるので`request`モジュールを利用する．
- ついでにPromiseを扱える`request-promise-native`もインストールする．
- 下記コマンドでインストール．

```bash
$ cd functions
$ npm install request
$ npm install request-promise-native
```

### リクエスト送信処理の追加

- Google books APIへのリクエスト関数を定義．
- エンドポイントを追加し，関数を実行．
- APIからのレスポンスをクライアントへ送信する．
- `index.js`を下記のように編集．

```js
// index.js
const functions = require('firebase-functions');
const express = require('express');
const requestPromise = require('request-promise-native'); // 追加

const app = express();

// APIにリクエストを送る関数を定義
const getDataFromApi = async keyword => {
  // cloud functionsから実行する場合には地域の設定が必要になるため，`country=JP`を追加している
  const requestUrl = 'https://www.googleapis.com/books/v1/volumes?country=JP&q=intitle:';
  const result = await requestPromise(`${requestUrl}${keyword}`);
  return result;
}

app.get('/hello', (req, res) => {
  res.send('Hello Express!');
});

app.get('/user/:userId', (req, res) => {
  // 省略
});

// エンドポイント追加
app.get('/gbooks/:keyword', (req, res) => {
  // APIリクエストの関数を実行
  const response = await getDataFromApi(req.params.keyword);
  res.send(response);
});

const api = functions.https.onRequest(app);
module.exports = { api };
```

### 動作確認&デプロイ

- ローカルサーバーで確認．

```bash
$ firebase serve
```

- リクエスト送信（例として`keyword`に`node.js`を指定）
```bash
$ curl http://localhost:5000/cloudfunctions-3517c/us-central1/api/gbooks/node.js
↓のようなJSONデータがたくさん返ってくればOK
{
  "kind": "books#volume",
  "id": "fOgtAgAAQBAJ",
  "etag": "McoTnjan+uE",
  "selfLink": "https://www.googleapis.com/books/v1/volumes/fOgtAgAAQBAJ",
  "volumeInfo": {
    "title": "Mastering Node.js",
    "authors": [
      "Sandro Pasquali"
    ],
    "publisher": "Packt Publishing Ltd",
    "publishedDate": "2013-11-25",
    "description": "This book contains an extensive set of practical examples and an easy-to-follow approach to creating 3D objects.This book is great for anyone who already knows JavaScript and who wants to start creating 3D graphics that run in any browser. You don’t need to know anything about advanced math or WebGL; all that is needed is a general knowledge of JavaScript and HTML. The required materials and examples can be freely downloaded and all tools used in this book are open source.",
    "industryIdentifiers": [
      {
        "type": "ISBN_13",
        "identifier": "9781782166337"
      },
      {
        "type": "ISBN_10",
        "identifier": "1782166335"
      }
    ],
    "readingModes": {
      "text": true,
      "image": true
    },
    "pageCount": 346,
    "printType": "BOOK",
    "categories": [
      "Computers"
    ],
    "maturityRating": "NOT_MATURE",
    "allowAnonLogging": true,
    "contentVersion": "2.2.2.0.preview.3",
    "panelizationSummary": {
      "containsEpubBubbles": false,
      "containsImageBubbles": false
    },
    "imageLinks": {
      "smallThumbnail": "http://books.google.com/books/content?id=fOgtAgAAQBAJ&printsec=frontcover&img=1&zoom=5&edge=curl&source=gbs_api",
      "thumbnail": "http://books.google.com/books/content?id=fOgtAgAAQBAJ&printsec=frontcover&img=1&zoom=1&edge=curl&source=gbs_api"
    },
    "language": "en",
    "previewLink": "http://books.google.co.jp/books?id=fOgtAgAAQBAJ&printsec=frontcover&dq=intitle:node.js&hl=&cd=10&source=gbs_api",
    "infoLink": "https://play.google.com/store/books/details?id=fOgtAgAAQBAJ&source=gbs_api",
    "canonicalVolumeLink": "https://play.google.com/store/books/details?id=fOgtAgAAQBAJ"
  },
  "saleInfo": {
    "country": "JP",
    "saleability": "FOR_SALE",
    "isEbook": true,
    "listPrice": {
      "amount": 3299,
      "currencyCode": "JPY"
    },
    "retailPrice": {
      "amount": 2969,
      "currencyCode": "JPY"
    },
    "buyLink": "https://play.google.com/store/books/details?id=fOgtAgAAQBAJ&rdid=book-fOgtAgAAQBAJ&rdot=1&source=gbs_api",
    "offers": [
      {
        "finskyOfferType": 1,
        "listPrice": {
          "amountInMicros": 3299000000,
          "currencyCode": "JPY"
        },
        "retailPrice": {
          "amountInMicros": 2969000000,
          "currencyCode": "JPY"
        }
      }
    ]
  },
  "accessInfo": {
    "country": "JP",
    "viewability": "PARTIAL",
    "embeddable": true,
    "publicDomain": false,
    "textToSpeechPermission": "ALLOWED",
    "epub": {
      "isAvailable": true
    },
    "pdf": {
      "isAvailable": true
    },
    "webReaderLink": "http://play.google.com/books/reader?id=fOgtAgAAQBAJ&hl=&printsec=frontcover&source=gbs_api",
    "accessViewStatus": "SAMPLE",
    "quoteSharingAllowed": false
  },
  "searchInfo": {
    "textSnippet": "This book contains an extensive set of practical examples and an easy-to-follow approach to creating 3D objects.This book is great for anyone who already knows JavaScript and who wants to start creating 3D graphics that run in any browser."
  }
}
```

- 動作確認したらデプロイ

```bash
$ firebase deploy
```

- ターミナルからリクエストを送る．
- ローカルサーバのときと同様にいろいろ返ってくればOK！

```bash
$ curl https://hogehoge.cloudfunctions.net/api/gbooks/react
うまくいっていればJSONデータが返ってくる
```


## CORS対策

- ターミナルから`curl`コマンドでリクエストを送信すると正常に動作するが，クライアントアプリから`axios`などでリクエストを送信するとCORSエラーが発生する．
- アプリケーションからも利用できるように，追加のモジュールをインストールする．

```bash
$ cd functions
$ npm install cors
```

### ファイル内全てのAPIについてCORSを許可したい場合

- 全部外部からのリクエストを許可する場合には下記のように追記すればOK．

```js
// index.js
const functions = require('firebase-functions');
const express = require('express');
const requestPromise = require('request-promise-native');
const cors = require('cors'); // 追加

const app = express();

app.use(cors());  // 追加

const getDataFromApi = async keyword => {
  // 省略
}

app.get('/hello', (req, res) => {
  // 省略
});

app.get('/user/:userId', (req, res) => {
  // 省略
});

app.get('/gbooks/:keyword', (req, res) => {
  // 省略
});

const api = functions.https.onRequest(app);
module.exports = { api };
```

### 個別のAPIについてCORSを許可したい場合

- 全部許可せずに，指定したエンドポイントのみアクセスを許可したい場合．
- 許可したいエンドポイントだけに追記を行う．

```js
// index.js
const functions = require('firebase-functions');
const express = require('express');
const requestPromise = require('request-promise-native');
const cors = require('cors'); // 追加

const app = express();

// app.use(cors());  // 一旦コメントアウト

const getDataFromApi = async keyword => {
  // 省略
}

app.get('/hello', (req, res) => {
  // 省略
});

app.get('/user/:userId', (req, res) => {
  // 省略
});

// ここに`cors()`を追加
app.get('/gbooks/:keyword', cors(), (req, res) => {
  // 省略
});

const api = functions.https.onRequest(app);
module.exports = { api };
```

- クライアントアプリケーションからリクエストを送信してデータが返ってくればOK．


## やってみよう！！

- cloud functions上に任意のAPIを絡めたNode.jsのアプリケーションをデプロイ．
- ターミナルやpostmanからリクエストを送信して動作している状態になっていればOK！
- できる人はReactの課題や他のクライアントアプリと連携させよう！

今回はここまで( `･ω･)b

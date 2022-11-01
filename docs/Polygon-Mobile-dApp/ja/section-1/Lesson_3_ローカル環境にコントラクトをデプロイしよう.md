### 🐣 ローカル環境でコントラクトをコンパイルしてデプロイする

前のセクションでスマートコントラクトが書けたので、次はそれをコンパイルします。

ターミナルを `todo-dApp-contract` ディレクトリで開き、以下のコマンドを実行します。

```bash
truffle compile
```

`truffle compile` でエラーが出た場合は下記をお試しください。

```bash
npx truffle compile
```

下のようにターミナルに表示されていれば成功です。

![](/public/images/Polygon-Mobile-dApp/section-1/1_3_1.png)

ブロックチェーンにコントラクトを移行します。`migrations` ディレクトリに移動し、`2_todo_contract_migration.js` というファイルを新規作成し、以下のコードを追加してください。

```js
// 2_todo_contract_migration.js
const TodoContract = artifacts.require("TodoContract");

module.exports = function (deployer) {
    deployer.deploy(TodoContract);
};
```

移行作業を開始する前に、`Ganache` がインストールされていることを確認してください。システムで `Ganache GUI` アプリを起動します。

`truffle-config.js` の既存の内容をすべて削除し、以下のコードに置き換えてください。

```js
//truffle-config.js
module.exports = {
  networks: {
    development: {
      host: "localhost",
      port: 7545,
      network_id: "*",
    },
  },
  contracts_directory: "./contracts",
  compilers: {
    solc: {
      optimizer: {
        enabled: true,
        runs: 200,
      },
    },
  },

  db: {
    enabled: false,
  },
};
```

少し説明します。

`truffle-config.js` では、Truffle の基本的な構成を定義しています。

```js
//truffle-config.js
  networks: {
    development: {
      host: "localhost",
      port: 7545,
      network_id: "*",
    },
  },
```

現在、Ganache ブロックチェーンが稼働している `localhost:7545` にスマートコントラクトをデプロイするよう指定しています。

それでは、Ganache 上にスマートコントラクトをデプロイするために、コマンドを実行していきます。

```bash
truffle migrate
```

`truffle migrate` でエラーが出た場合は下記をお試しください。

```bash
npx truffle migrate
```

こちらでもエラーが出た場合は、`truffle-config.js` ファイルを下記に変更して、もう一度上記の操作を行ってください。

```js
//truffle-config.js
module.exports = {
  networks: {
    development: {
      host: "localhost",
      port: 7545,
      network_id: "*",
    },
  },
  contracts_directory: "./contracts",

  compilers: {
    solc: {
      version: "0.8.11",
      optimizer: {
        enabled: true,
        runs: 200,
      },
    }
  },
  db: {
    enabled: false,
  },
};
```
下のようにターミナルに表示されていれば成功です。

![](/public/images/Polygon-Mobile-dApp/section-1/1_3_2.png)

ローカルネットワークへのコンパイルとデプロイの作業は以上で完了です。
次は、Flutter アプリケーションへ接続していきましょう。
### 🙋‍♂️ 質問する

ここまでの作業で何かわからないことがある場合は、Discord の `#polygon-mobile-dapp` で質問をしてください。

ヘルプをするときのフローが円滑になるので、エラーレポートには下記の 3 点を記載してください ✨

```
1. 質問が関連しているセクション番号とレッスン番号
2. 何をしようとしていたか
3. エラー文をコピー&ペースト
4. エラー画面のスクリーンショット
```

---
ターミナルの出力結果を Discord の `#polygon-mobile-dapp` に投稿して、コミュニティにシェアしてください!

次のセクションに進んで、Flutter アプリケーションへの接続を開始しましょう 🎉

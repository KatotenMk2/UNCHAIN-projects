### 🖥 本番環境の構築を行う

ローカル環境のイーサリアムネットワークを終了します。

- ターミナルを閉じれば完了です。

これから、実際のブロックチェーンにコントラクトをデプロイするための環境を構築していきます。

### 💳 トランザクション

**イーサリアムネットワーク上でブロックチェーンに新しく情報を書き込むこと**を、**トランザクション**と呼びます。

ここまでのレッスンに登場した**トランザクション**は以下です。

- **新規にスマートコントラクトをデプロイした**という情報をブロックチェーン上に書き込む。
- **Web サイト上で送信された「👋（wave）」の数**をブロックチェーンに書き込む。

トランザクションにはマイナーの承認が必要ですので、Alchemy を導入します。

Alchemy は、世界中のトランザクションを一元化し、マイナーの承認を促進するプラットフォームです。

[こちら](https://www.alchemy.com/) から Alchemy のアカウントを作成してください。

### 💎 Alchemy でネットワークを作成

Alchemy のアカウントを作成したら、`CREATE APP` ボタンを押してください。

![](/public/images/ETH-dApp/section-2/2_2_17.png)
Ecosystem 選択欄が出てきた場合は `Ethereum` を選択しましょう。

![](/public/images/ETH-dApp/section-2/2_2_1.png)
次に、下記の項目を埋めていきます。下図を参考にしてください。

![](/public/images/ETH-dApp/section-2/2_2_2.png)

- `NAME`: プロジェクトの名前（例: `WavePortal`）
- `DESCRIPTION`: プロジェクトの概要
- `CHAIN`: `Ethereum` を選択
- `NETWORK`: `Goerli` を選択

それから、作成した App の `VIEW DETAILS` をクリックします。
![](/public/images/ETH-dApp/section-2/2_2_3.png)

プロジェクトを開いたら、`VIEW KEY` ボタンをクリックします。
![](/public/images/ETH-dApp/section-2/2_2_4.png)
ポップアップが開くので、`HTTP` のリンクをコピーしてください。

これがあなたが本番環境のネットワークに接続する際に使用する `API Key` になります。

- **`API Key` は、今後必要になるので、PC 上のわかりやすいところに保存しておきましょう。**

### 🐣 テストネットから始める

今回のプロジェクトでは、コスト（＝ 本物の ETH）が発生するイーサリアムメインネットではなく、**テストネットにコントラクトをデプロイします。**

テストネットはイーサリアムメインネットを模しています。

- イーサリアムメインネットにコントラクトをデプロイした際に発生するイベントのテストを行うのに最適です。
- テストネットは偽の ETH を使用しているため、いくらでもトランザクションのテストを行えます。

今回は、以下のイベントをテストしていきます。

1. トランザクションの発生を世界中のマイナーたちに知らせる
2. あるマイナーがトランザクションを発見する
3. そのマイナーがトランザクションを承認する
4. そのマイナーがトランザクションを承認したことをほかのマイナーたちに知らせ、トランザクションのコピーを更新する

このセクションでは、コードを書きながら、これらのイベントについての理解を深めていきます。

### 🚰 Testnet ETH を取得する

今回は、`Goerli` というイーサリアム財団によって運営されているテストネットを使用します。

`Goerli` にコントラクトをデプロイし、コードのテストを行うために、偽の ETH を取得しましょう。ユーザーが偽の ETH を取得するために用意されたインフラは、「フォーセット（＝蛇口）」と呼ばれています。

フォーセットを使用する前に、あなたの MetaMask ウォレットを `Goerli Test Network` に設定してください。

> ✍️: MetaMask で `Goerli Test Network` を設定する方法
>
> 1 \. MetaMask ウォレットのネットワークトグルを開く。
>
> ![](/public/images/ETH-dApp/section-2/2_2_13.png)
>
> 2 \. `Show/hide test networks` をクリック。
>
> ![](/public/images/ETH-dApp/section-2/2_2_14.png)
>
> 3 \. `Show test networks` を `ON` にする。
>
> ![](/public/images/ETH-dApp/section-2/2_2_15.png)
>
> 4 \. `Goerli Test Network` を選択する。
>
> ![](/public/images/ETH-dApp/section-2/2_2_16.png)

MetaMask ウォレットに `Goerli Test Network` が設定されたら、下記のリンクの中から条件に合うものを選んで、少量の偽 ETH を取得しましょう。

- [Alchemy](https://goerlifaucet.com/) - 0.25 Goerli ETH（24 時間に 1 度もらうことができる）
  - ウォレットアドレスを入力して `Send Me ETH` ボタンを押下するとその場でもらえます。
- [Chainlink](https://faucets.chain.link/) - 0.1 Goerli ETH（その場でもらえる）
  - `Connect wallet` をクリックして MetaMask と接続する必要があります。
  - Twitter アカウントを連携する必要があります。


### 📈 `hardhat.config.js` ファイルを編集する

`hardhat.config.js` ファイルを変更する必要があります。

これは、スマートコントラクトプロジェクトのルートディレクトリにあります。

- 今回は、`my-wave-portal` ディレクトリの直下に `hardhat.config.js` が存在するはずです。

例）ターミナル上で `my-wave-portal` に移動し、`ls` を実行した結果

```
yukis4san@Yukis-MacBook-Pro my-wave-portal % ls
README.md			package-lock.json
artifacts			package.json
cache				scripts
contracts			test
hardhat.config.js node_modules
.gitignore
```

`hardhat.config.js` を VS Code で開いて、中身を編集していきます。

```javascript
// hardhat.config.js
require("@nomicfoundation/hardhat-toolbox");
module.exports = {
  solidity: "0.8.9",
  networks: {
    goerli: {
      url: "YOUR_ALCHEMY_API_URL",
      accounts: ["YOUR_PRIVATE_GOERLI_ACCOUNT_KEY"],
    },
  },
};
```

1\. `YOUR_ALCHEMY_API_URL`の取得

> `hardhat.config.js` の `YOUR_ALCHEMY_API_URL` の部分を先ほど取得した Alchemy の URL（ `HTTP` リンク） と入れ替えます。

2\. `YOUR_PRIVATE_GOERLI_ACCOUNT_KEY` の取得

> 1.  お使いのブラウザから、MetaMask プラグインをクリックして、ネットワークを `Goerli Test Network` に変更します。
>
> ![](/public/images/ETH-dApp/section-2/2_2_5.png)
>
> 2.  それから、`Account details` を選択してください。
>
> ![](/public/images/ETH-dApp/section-2/2_2_6.png)
>
> 3.  `Account details` から `Export Private Key` をクリックしてください。
>
> ![](/public/images/ETH-dApp/section-2/2_2_7.png)
>
> 4.  MetaMask のパスワードを求められるので、入力したら `Confirm` を押します。
>
> ![](/public/images/ETH-dApp/section-2/2_2_8.png)
>
> 5.  あなたの秘密鍵（＝ `Private Key` ）が表示されるので、クリックしてコピーします。
>
> ![](/public/images/ETH-dApp/section-2/2_2_9.png)

> - `hardhat.config.js` の `YOUR_PRIVATE_GOERLI_ACCOUNT_KEY` の部分をここで取得した秘密鍵とを入れ替えます。

### 🙊 秘密鍵は誰にも教えてはいけません

`hardhat.config.js` を更新したら、ここで一度立ち止まりましょう。

**🚨: `hardhat.config.js` ファイルをあなたの秘密鍵の情報を含んだ状態で GitHub にコミットしてはいけません**。

> この秘密鍵は、あなたのイーサリアムメインネットの秘密鍵と同じです。
>
> 秘密鍵が流出してしまうと、誰でもあなたのウォレットにアクセスすることができてしまうので、とても危険です。
>
> **絶対に秘密鍵を自分以外の人が見れる場所に置かないようにしましょう**。

下記を実行して、VS Code で `.gitignore` ファイルを編集しましょう。

```
code .gitignore
```

`.gitignore` に `hardhat.config.js` の行を追加します。

`.gitignore` の中身が下記のようになっていれば、問題ありません。

```bash
node_modules
.env
coverage
coverage.json
typechain
typechain-types

#Hardhat files
cache
artifacts
hardhat.config.js
```

`.gitignore` に記載されているファイルやディレクトリは、GitHub にディレクトリをプッシュされずに、ローカル環境にのみ保存されます。

> **✍️: スマートコントラクトをデプロイするのに秘密鍵が必要な理由** > **新しくスマートコントラクトをイーサリアムネットワーク上にデプロイすること**も、トランザクションの一つです。
>
> トランザクションを行うためには、ブロックチェーンに「ログイン」する必要があります。
>
> 「ログイン」には下記の情報が必要となります。
>
> - ユーザー名: 公開アドレス
>   ![](/public/images/ETH-dApp/section-2/2_2_10.png)
> - パスワード: 秘密鍵
>   ![](/public/images/ETH-dApp/section-2/2_2_11.png)
>   ユーザー名とパスワードを使用して、AWS にログインしてプロジェクトをデプロイするのと同じです。

⚠️: すでに GitHub に `hardhat.config.js` を push してしまった場合の対処法

すでに GitHub にコードを push している場合は、ターミナル上で下記を実行してください。

```
git rm --cached hardhat.config.js
```

これで、ローカルリポジトリの `hardhat.config.js` ファイルを残した状態で、リモートリポジトリのファイルを消すことができます。再度、`git push` まで行ったら、GitHub 上に `hardhat.config.js` が存在しないことを確認してください。

### 🚀 Goerli Test Network にコントラクトをデプロイする

`hardhat.config.js` の更新が完了したら、Goerli Test Network にコントラクトをデプロイしてみましょう。

ターミナル上で `my-wave-portal` ディレクトリに移動し、下記のコマンドを実行しましょう。

```bash
npx hardhat run scripts/deploy.js --network goerli
```

下記のような結果が出力されていれば成功です 🎉

```bash
Deploying contracts with account:  0x1A7f14FBF50acf10bCC08466743fB90384Cbd720
Account balance:  174646846389073382
Contract deployed to:  0x04da168454AFA19Eb43D6A28b63964D8DCE8351e
Contract deployed by:  0x1A7f14FBF50acf10bCC08466743fB90384Cbd720
```

あなたのターミナル上で、`Contract deployed to` の後に出力されたコントラクトアドレス（`0x..`）をコピーして、保存しておきましょう。後でフロントエンドを構築する際に必要となります。

### 👀 Etherscan でトランザクションを確認する

コピーした `Contract deployed to` に続くアドレスを、[Etherscan](https://goerli.etherscan.io/) に貼り付けて、あなたのスマートコントラクトのトランザクション履歴を見てみましょう。

Etherscan は、イーサリアムネットワーク上のトランザクションに関する情報を確認するのに便利なプラットフォームです。

_表示されるまでに約 1 分程度かかる場合があります。_

### 🙋‍♂️ 質問する

ここまでの作業で何かわからないことがある場合は、Discord の `#eth-dapp` で質問をしてください。

ヘルプをするときのフローが円滑になるので、エラーレポートには下記の 3 点を記載してください ✨

```
1. 質問が関連しているセクション番号とレッスン番号
2. 何をしようとしていたか
3. エラー文をコピー&ペースト
4. エラー画面のスクリーンショット
```

---

Goerli Test Network にスマートコントラクトをデプロイしたら、次のレッスンに進みましょう 🎉

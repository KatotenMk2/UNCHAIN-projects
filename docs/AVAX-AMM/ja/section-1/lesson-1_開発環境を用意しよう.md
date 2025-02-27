### ✅ 環境構築を行う

このプロジェクトの全体像は次のとおりです。

1 \. **スマートコントラクトを作成します。**

- ユーザーがトークンをスマートコントラクト上で交換するためのAMMロジックを実装します。
- 開発は`Ethereum`のローカルチェーンを利用して作成します。

2 \. **スマートコントラクトをブロックチェーン上にデプロイします。**

- スマートコントラクトを`Avalanche`の`Fuji C-Chain`へデプロイします。
- 世界中の誰もがあなたのスマートコントラクトにアクセスできます。
- **ブロックチェーンは,サーバーの役割を果たします。**

3 \. **Web アプリケーション（dApp）を構築します**。

- ユーザーはWebサイトを介して,ブロックチェーン上に展開されているあなたのスマートコントラクトと簡単にやりとりできます。
- スマートコントラクトの実装 + フロントエンドユーザー・インタフェースの作成 👉 dAppの完成を目指しましょう 🎉

### ✨ Hardhat をインストールする

スマートコントラクトをすばやくコンパイルし,ローカル環境でテストするために,**Hardhat** というツールを使用します。

- Hardhatにより,ローカル環境でイーサリアムネットワークを簡単に起動し,テストネットでethereumを利用できます。
- 「サーバー」がブロックチェーンであることを除けば,Hardhatはローカルサーバーと同じです。

まず,`node` / `npm`を取得する必要があります。
お持ちでない場合は,[こちら](https://hardhat.org/tutorial/setting-up-the-environment#installing-node.js) にアクセスし`Node.js`をインストールしてください。
`Node.js`をインストールすると, そのパッケージ管理ツールである`npm`も同時にインストールされます。

> 動作確認。
>
> ```
> $ node -v
> v18.6.0
> ```
>
> ```
> $ npm -v
> 8.13.2
> ```
>
> 併せて本プロジェクトの実行環境(上記のバージョン)もトラブル時の参考にしてください。

### 🛫 プロジェクトを作成しよう

アプリケーションのコードを格納するフォルダーを,モノレポ構成となるように準備していきましょう！

モノレポとは,コントラクトとクライアント（またはその他構成要素）の全コードをまとめて1つのリポジトリで管理する方法です。

作業したいディレクトリに移動したら,次のコマンドを実行します。

```bash
mkdir AVAX-AMM
cd AVAX-AMM
npm init -y
```

AVAX-AMMディレクトリ内に,package.jsonファイルが生成されます。

```bash
AVAX-AMM
 └── package.json
```

`package.json`ファイルを下記の内容で上書きします。

```json
// package.json

{
  "name": "AVAX-AMM",
  "version": "1.0.0",
  "description": "`Miniswap` is a amm dapp that allows tokens to be exchanged like `Uniswap`.",
  "license": "",
  "private": true,
  "repository": {
    "type": "git",
    "url": "git+https://github.com/unchain-tech/AVAX-AMM.git"
  },
  "author": "",
  "bugs": {
    "url": "https://github.com/unchain-tech/AVAX-AMM/issues"
  },
  "workspaces": {
    "packages": [
      "packages/*"
    ]
  },
  "scripts": {
    "contract": "npm run --workspace=contract",
    "client": "npm run --workspace=client",
    "test": "npm run contract test"
  }
}
```

`package.json`ファイルの内容を確認してみましょう。

モノレポを作成するにあたり,パッケージマネージャーの機能である[Workspaces](https://docs.npmjs.com/cli/v9/using-npm/workspaces?v=true)を利用しています。

この機能により,npm installを一度だけ実行すれば,すべてのパッケージ（今回はコントラクトのパッケージとクライアントのパッケージ）を一度にインストールできるようになります。

**workspaces**の定義をしている部分は以下になります。

```json
"workspaces": {
  "packages": [
    "packages/*"
  ]
},
```

また,ワークスペース内の各パッケージにアクセスするためのコマンドを以下の部分で定義しています。

```json
"scripts": {
  "contract": "npm run --workspace=contract",
  "client": "npm run --workspace=client",
  "test": "npm run contract test"
}
```

これにより,各パッケージのディレクトリへ階層を移動しなくてもプロジェクトのルート直下から以下のようにコマンドを実行することが可能となります（ただし,各パッケージ内に`package.json`ファイルが存在し,その中にコマンドが定義されていないと実行できません。そのため,現在は実行してもエラーとなります。ファイルは後ほど作成します）。

```bash
npm run <パッケージ名> <実行したいコマンド>
```

それでは,ワークスペースのパッケージを格納するディレクトリを作成しましょう。

以下のようなフォルダー構成となるように,`packages`ディレクトリとその中に`contract`ディレクトリを作成してください（`client`ディレクトリは,後ほどのレッスンで作成したいと思います）。

```diff
AVAX-AMM
 ├── package.json
+└── packages/
+    └── contract/
```

最後に,AVAX-AMMディレクトリ下に`.gitignore`ファイルを作成して以下の内容を書き込みます。

```bash
# dependencies
**/node_modules

# misc
**/.DS_Store
```

最終的に以下のようなフォルダー構成となっていることを確認してください。

```bash
AVAX-AMM
├── .gitignore
├── package.json
└── packages/
    └── contract/
```

これでモノレポの雛形が完成しました！

### ✨ パッケージ をインストールする

それでは,スマートコントラクトの開発に必要なパッケージをインストールしましょう。

先ほど作成した`packages/contract`ディレクトリ内にディレクトリに移動したら,次のコマンドを実行します。

```bash
cd ./packages/contract
npm init -y
npm install --save-dev hardhat @openzeppelin/test-helpers
npm install dotenv @openzeppelin/contracts
```

`npm init`によりnpmパッケージを管理するための環境をセットアップを行い,スマートコントラクトの開発に必要な以下のパッケージをインストールしています。

- `hardhat`: solidityを使った開発をサポートします。
- `dotenv`: 環境変数の設定で必要になります。コントラクトをデプロイする際に利用します。
- `@openzeppelin/test-helpers`: テストを支援するライブラリです。コントラクトのテストを書く際に利用します。
- `@openzeppelin/contracts`: `ERC20`が実装されています。コントラクトの実装時に`ERC20`を利用します。

では、生成された`contarct/package.json`ファイルの`"scripts"`部分を以下の内容に更新しましょう。また,`"private"`が`true`になっていることを確認（設定されていない場合は記述）してください。

```json
// package.json

{
  "name": "contract",
  "version": "1.0.0",
  "private": true,
  // 以下の内容に更新
  "scripts": {
    "deploy": "npx hardhat run scripts/deploy.ts --network fuji",
    "cp": "npm run cp:typechain && npm run cp:artifact",
    "cp:typechain": "cp -r typechain-types ../client/",
    "cp:artifact": "cp artifacts/contracts/ERC20Tokens.sol/USDCToken.json artifacts/contracts/ERC20Tokens.sol/JOEToken.json artifacts/contracts/AMM.sol/AMM.json ../client/utils/",
    "test": "npx hardhat test"
  },
}
```

### 👏 サンプルプロジェクトを開始する

次に,Hardhatを実行します。

`packages/contract`ディレクトリにいることを確認し,下記を実行します。
`npx hardhat`を実行すると対話形式で指示を求められるので下記のように回答します。
`Create a TypeScript project`を選択するところ以外は`enter`を押せば例通りになるはずです。

```
$ npx hardhat
...
✔ What do you want to do? · Create a TypeScript project
✔ Hardhat project root: · path/to/contract /contract
✔ Do you want to add a .gitignore? (Y/n) · y
✔ Do you want to install this sample project's dependencies with npm (@nomicfoundation/hardhat-toolbox)? (Y/n) · y
```

---

📓 `TypeScript`について

初めて`TypeScript`を触れる方向けに少し解説を入れさせて頂きます。

`TypeScript`のコードはコンパイルにより`JavaScript`のコードに変換されてから実行されます。

最終的には`JavaScript`のコードとなるので, 処理能力など`JavaScript`と変わることはありません。
ですが`TypeScript`には静的型付け機能を搭載しているという特徴があります。

静的型付けとは, ソースコード内の値やオブジェクトの型をコンパイル時に解析し, 安全性が保たれているかを検証するシステム・方法のことです。

`JavaScript`では明確に型を指定する必要がないため, コード内で型の違う値を誤って操作している場合は実行時にそのエラーが判明することがあります。

`TypeScript`はそれらのエラーはコンパイル時に判明するためバグの早期発見に繋がります。
バグの早期発見は開発コストを下げることにつながります。

本プロジェクトでは, コントラクトのテストとフロントエンドの構築に`TypeScript`を使用します。
フロントエンドの実装の方では自ら型の指定をする部分が多いのでより型について認識できるかもしれません。
（コントラクのテスト実装の方では, 自動的に型を判別する機能を使用しているので自ら型を指定する部分が少ないです）。

ひとまず, オブジェクトの型がわかっていないと実行できないような`JavaScript`コード, という認識でまずは進めてみてください。

---

### ⭐️ 実行する

すべてが機能していることを確認するには,`AVAX-AMM`ディレクトリ直下で次のコマンドを実行します。

```
$ npm run test
```

次のように表示されたら成功です! 🎉

![](/public/images/AVAX-AMM/section-1/1_1_1.png)

ここまできたら,フォルダーの中身を整理しましょう。

`packages/contract`内は以下のようなフォルダ構成になっているはずです。

```
packages/
└── contract
    ├── README.md
    ├── artifacts
    ├── cache
    ├── contracts
    ├── hardhat.config.ts
    ├── node_modules
    ├── package-lock.json
    ├── package.json
    ├── scripts
    ├── test
    ├── tsconfig.json
    └── typechain-types
```

`test`の下のファイル`Lock.ts`と
`contracts`の下のファイル`Lock.sol`を削除してください。

実際のフォルダは削除しないように注意しましょう。

### 🐊 `github`にソースコードをアップロードしよう

本プロジェクトの最後では, アプリをデプロイするために`github`へソースコードをアップロードする必要があります。

**AVAX-AMM**全体を対象としてアップロードしましょう。

今後の開発にも役に立つと思いますので, 今のうちに以下にアップロード方法をおさらいしておきます。

`GitHub`のアカウントをお持ちでない方は,[こちら](https://qiita.com/okumurakengo/items/848f7177765cf25fcde0) の手順に沿ってアカウントを作成してください。

`GitHub`へソースコードをアップロードをしたことがない方は以下を参考にしてください。

[新しいレポジトリを作成](https://docs.github.com/ja/get-started/quickstart/create-a-repo)（リポジトリ名などはご自由に）した後,  
手順に従いターミナルからアップロードを済ませます。  
以下ターミナルで実行するコマンドの参考です。(`AVAX-AMM`直下で実行することを想定しております)

```
$ git init
$ git add .
$ git commit -m "first commit"
$ git branch -M main
$ git remote add origin [作成したレポジトリの SSH URL]
$ git push -u origin main
```

> ✍️: SSH の設定を行う
>
> Github のレポジトリをクローン・プッシュする際に,SSHKey を作成し,GitHub に公開鍵を登録する必要があります。
>
> SSH（Secure SHell）はネットワークを経由してマシンを遠隔操作する仕組みのことで,通信が暗号化されているのが特徴的です。
>
> 主にクライアント（ローカル）からサーバー（リモート）に接続をするときに使われます。この SSH の暗号化について,仕組みを見ていく上で重要になるのが秘密鍵と公開鍵です。
>
> まずはクライアントのマシンで秘密鍵と公開鍵を作り,公開鍵をサーバーに渡します。そしてサーバー側で「この公開鍵はこのユーザー」というように,紐付けを行っていきます。
>
> 自分で管理して必ず見せてはいけない秘密鍵と,サーバーに渡して見せても良い公開鍵の 2 つが SSH の通信では重要になってきます。
> Github における SSH の設定は,[こちら](https://docs.github.com/ja/authentication/connecting-to-github-with-ssh) を参照してください!

### 🙋‍♂️ 質問する

ここまでの作業で何かわからないことがある場合は,Discordの`#avalanche`で質問をしてください。

ヘルプをするときのフローが円滑になるので,エラーレポートには下記の3点を記載してください ✨

```
1. 質問が関連しているセクション番号とレッスン番号
2. 何をしようとしていたか
3. エラー文をコピー&ペースト
4. エラー画面のスクリーンショット
```

---

環境設定が完了したら,次のレッスンに進んでください 🎉

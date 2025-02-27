### 💻 クライアントを設定する

このセクションでは、Webサイトの構築を通して、クライアントとスマートコントラクトの連携方法について学びます。

実装は下記をイメージしてください。

- クライアント＝フロントエンド

- スマートコントラクト＝バックエンド

それでは、始めましょう 🚀

### 🍽 Git リポジトリをあなたの GitHub にフォークする

まだGitHubのアカウントをお持ちでない方は、[こちら](https://qiita.com/okumurakengo/items/848f7177765cf25fcde0) の手順に沿ってアカウントを作成してください。

GitHubのアカウントをお持ちの方は、下記の手順に沿ってフロントエンドの基盤となるリポジトリをあなたのGitHubに[フォーク](https://denno-sekai.com/github-fork/)しましょう。

1. [こちら](https://github.com/unchain-tech/ETH-NFT-Collection)からunchain-tech/ETH-NFT-Collectionリポジトリにアクセスをして、ページ右上の`Fork`ボタンをクリックします。

![](/public/images/ETH-NFT-Collection/section-3/3_1_3.png)

2. Create a new forkページが開くので、以下2つの項目を設定します。
- Repository name: `ETH-NFT-Collection-starter`に変更します。
- Copy the `main` branch only: **チェックが入っていることを必ず確認します**。

![](/public/images/ETH-NFT-Collection/section-3/3_1_4.png)

設定が完了したら`Create fork`ボタンをクリックします。あなたのGitHubアカウントに`ETH-NFT-Collection`リポジトリのフォークが作成されたことを確認してください。

それでは、フォークしたリポジトリをローカル環境にクローンしましょう。

まず、下図のように、`Code`ボタンをクリックして`SSH`を選択し、Gitリンクをコピーしましょう。

![](/public/images/ETH-NFT-Collection/section-3/3_1_1.png)

ターミナル上で`ETH-NFT-Collection/packages`ディレクトリに移動し、先ほどコピーしたリンクを用いて下記を実行してください。

```bash
git clone コピーした_github_リンク client
```

この段階で、フォルダ構造は下記のようになっているはずです。

```diff
ETH-NFT-Collection
 ├── package.json
 └── packages/
+    ├── client/
     └── contract/
```

ターミナル上で`ETH-NFT-Collection`ディレクトリ下に移動して下記を実行しましょう。

```bash
yarn install
```

`yarn`コマンドを実行することで、JavaScriptライブラリのインストールが行われます。

次に、下記を実行してみましょう。

```bash
yarn client start
```

あなたのローカル環境で、Webサイトのフロントエンドが立ち上がりましたか？

例)ローカル環境で表示されているWebサイト

![](/public/images/ETH-NFT-Collection/section-3/3_1_2.png)

上記のような形でフロントエンドが確認できれば成功です。

これからフロントエンドの表示を確認したい時は、ターミナルに向かい、`ETH-NFT-Collection`ディレクトリ上で、`yarn client start`を実行します。これからも必要となる作業ですので、よく覚えておいてください。

ターミナルを閉じるときは、以下のコマンドが使えます ✍️

- Mac: `ctrl + c`
- Windows: `ctrl + shift + w`

### 🦊 MetaMask をダウンロードする

次に、イーサリアムウォレットをダウンロードしましょう。

このプロジェクトではMetaMaskを使用します。

- [こちら](https://MetaMask.io/download.html) からブラウザの拡張機能をダウンロードし、MetaMaskウォレットをあなたのブラウザに設定します。

すでに別のウォレットをお持ちの場合でも、今回はMetaMaskを使用してください。

> ✍️: MetaMask が必要な理由
> ユーザーが、スマートコントラクトを呼び出すとき、イーサリアムアドレスと秘密鍵を備えたウォレットが必要となります。
>
> - これは、認証作業のようなものです。

### 🙋‍♂️ 質問する

ここまでの作業で何かわからないことがある場合は、Discordの`#ethereum`で質問をしてください。

ヘルプをするときのフローが円滑になるので、エラーレポートには下記の3点を記載してください ✨

```
1. 質問が関連しているセクション番号とレッスン番号
2. 何をしようとしていたか
3. エラー文をコピー&ペースト
4. エラー画面のスクリーンショット
```

---

MetaMaskのダウンロードが完了したら次のレッスンに進みましょう 🎉

### 🤖 ウォレットプロバイダーを設定する

今回接続するウォレットは [Phantom Wallet](https://phantom.app/) ですが、Solana ウォレットならどれでも使用できるはずです（ただし、他のウォレットではテストしていません）。

この Web アプリケーションを作成するにあたり、一番始めにやるべきことはウォレットを接続することです。

そこで、`pages` ディレクトリ直下にある `_app.js` ファイルを以下のとおり更新します。

```jsx
// _app.js

import React, { useMemo } from "react";
import { WalletAdapterNetwork } from "@solana/wallet-adapter-base";
import { WalletModalProvider } from "@solana/wallet-adapter-react-ui";
import { ConnectionProvider, WalletProvider } from "@solana/wallet-adapter-react";
import {
  GlowWalletAdapter,
  PhantomWalletAdapter,
  SlopeWalletAdapter,
  SolflareWalletAdapter,
  TorusWalletAdapter,
} from "@solana/wallet-adapter-wallets";
import { clusterApiUrl } from "@solana/web3.js";

import "@solana/wallet-adapter-react-ui/styles.css";
import "../styles/globals.css";
import "../styles/App.css";

const App = ({ Component, pageProps }) => {
  // networkはdevnet、testnet、またはmainnet-betaに設定できます。
  const network = WalletAdapterNetwork.Devnet;

  // networkを設定してメモ化します。
  const endpoint = useMemo(() => clusterApiUrl(network), [network]);

  // ここで設定したウォレットがWebアプリケーションに組み込まれます。
  const wallets = useMemo(
    () => [
      new PhantomWalletAdapter(),
      new GlowWalletAdapter(),
      new SlopeWalletAdapter(),
      new SolflareWalletAdapter({ network }),
      new TorusWalletAdapter(),
    ],
    [network]
  );

  return (
    <ConnectionProvider endpoint={endpoint}>
      <WalletProvider wallets={wallets} autoConnect>
        <WalletModalProvider>
          <Component {...pageProps} />
        </WalletModalProvider>
      </WalletProvider>
    </ConnectionProvider>
  );
};

export default App;
```

まずは `import` しているライブラリ、モジュール等について触れていきます。

最初にインポートしているのは React の `useMemo()` [（参考）](https://reactjs.org/docs/hooks-reference.html#usememo) で、依存先が変更された場合にのみデータをロードする React hook です。

`_app.js` では、ユーザーが接続している **ネットワーク** が変更されなければ、`clusterApiUrl` の値は変更されません。

次は[@solana/wallet-adapter](https://solana-labs.github.io/wallet-adapter/) の `wallet-adapter-network` です。

これは、利用可能なネットワーク（mainnet-beta 、testnet、devnet）を略記した[オブジェクトを記述した配列](https://github.com/solana-labs/wallet-adapter/blob/469edb5dd45231d397751b0268c86dffd6ed730a/packages/core/base/src/types.ts)です（詳細はリンク先を参照）

次の `WalletModalProvider` はユーザーのウォレット選択を捗らせる素晴らしい React コンポーネントです。

その次の `ConnectionProvider` は RPC エンドポイントを受け取り、Solana ブロックチェーン上のノードと直接やり取りできるようにしてくれます。

つまり、Web アプリケーションが `ConnectionProvider` を使用して Solana ブロックチェーンにトランザクションを送信してくれます。

次の `WalletProvider` はあらゆるウォレットに接続する際の標準インタフェースを提供するものです。

`WalletProvider` により、それぞれのウォレットのドキュメントを読む必要がなくなります。

その次の `wallet-adapter-wallets` からは様々なウォレットアダプタが提供されるので、必要なアダプタをインポートして、`WalletProvider` にウォレットのリストを渡します。

※ウォレットアダプタが必要な理由については[こちら](https://solana.com/ja/news/solana-why-you-should-use-wallet-adapter)を参照ください。

最後の `clusterApiURL` は指定したネットワークに基づいて RPC エンドポイントを生成する関数です。

React App コンポーネント内の return ステートメントでは、子（アプリの残りの部分）をいくつかの[context](https://reactjs.org/docs/context.html#contextprovider)プロバイダーでラップしています。

これで、ウォレットを接続するための準備が整いました。


### 🧞‍♂️ プロバイダーを使用してウォレットに接続する

それでは、ウォレットを接続していきましょう。

`index.js` を以下のとおり更新します。

```jsx
// index.js

import React from 'react';
import { PublicKey } from '@solana/web3.js';
import { useWallet } from '@solana/wallet-adapter-react';
import { WalletMultiButton } from '@solana/wallet-adapter-react-ui';

// 定数を宣言します。
const TWITTER_HANDLE = "あなたのTwitterハンドル";
const TWITTER_LINK = `https://twitter.com/${TWITTER_HANDLE}`;

const App = () => {
  // サポートしているウォレットからユーザーのウォレットアドレスを取得します。
  const { publicKey } = useWallet();

  const renderNotConnectedContainer = () => (
    <div>
      <img src="https://media.giphy.com/media/FWAcpJsFT9mvrv0e7a/giphy.gif" alt="anya" />
      <div className="button-container">
        <WalletMultiButton className="cta-button connect-wallet-button" />
      </div>
    </div>
  );

  return (
    <div className="App">
      <div className="container">
        <header className="header-container">
          <p className="header"> 😳 UNCHAIN Image Store 😈</p>
          <p className="sub-text">The only Image store that accepts shitcoins</p>
        </header>

        <main>
          {/* ウォレットアドレスが存在しない場合はConnectボタンを表示します。 */}
          {publicKey ? 'Connected!' : renderNotConnectedContainer()}
        </main>

        <div className="footer-container">
          <img alt="Twitter Logo" className="twitter-logo" src="twitter-logo.svg" />
          <a
            className="footer-text"
            href={TWITTER_LINK}
            target="_blank"
            rel="noreferrer"
          >{`built on @${TWITTER_HANDLE}`}</a>
        </div>
      </div>
    </div>
  );
};

export default App;
```

`useWallet()` フックを使用することで、Web アプリケーションのどこからでも接続されたユーザーのアドレスを取得できます。

これで、ウォレットを接続することができるようになりました。

※ Twitter ハンドルを忘れずに更新してくださいね!

```jsx
const TWITTER_HANDLE = "あなたのTwitterハンドル";
```

さて、これで画像の下に `Select Wallet` ボタンが表示されるようになったはずです。


### 🙋‍♂️ 質問する

ここまでの作業で何かわからないことがある場合は、Discord の `#solana-online-store` で質問をしてください。

ヘルプをするときのフローが円滑になるので、エラーレポートには下記の 4 点を記載してください ✨

```
1. 質問が関連しているセクション番号とレッスン番号
2. 何をしようとしていたか
3. エラー文をコピー&ペースト
4. エラー画面のスクリーンショット
```

---

次のレッスンでは、商品を用意してダウンロード機能を実装していきます!

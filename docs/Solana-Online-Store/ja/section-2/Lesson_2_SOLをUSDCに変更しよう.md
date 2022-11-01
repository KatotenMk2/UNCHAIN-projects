### 🤑 SOL から USDC に変更する

USDC のトランザクションは SOL のトランザクションと非常によく似ています。

ここでの変更点は、トランザクション関数で別のタイプの命令を呼び出すことです。

まず、[ここ](https://spl-token-faucet.com/?token-name=USDC)で、Solana 上で展開されている devnet 用の USDC トークン受け取ってください。

ウォレットを接続して、「Network selection」から「DEVNET」を選択し、「Address for airdrop」に自分のアドレスを入れ、「USDC airdrop」の`GET USDC` ボタンを押すと USDC が Airdrop されます。

![USDC TOKEN FAUCET](/public/images/Solana-Online-Store/section-2/2_2_1.jpg)

USDC を手に入れたら、`pages/api` フォルダ内の `createTransaction.js` を以下のとおり更新します。

**※ コピー&ペーストした後、`sellerAddress` の値を自分のウォレットアドレスに変更するのを忘れないでください!**

```jsx
// createTransaction.js

import { WalletAdapterNetwork } from "@solana/wallet-adapter-base";
import { clusterApiUrl, Connection, PublicKey, Transaction } from "@solana/web3.js";
import { createTransferCheckedInstruction, getAssociatedTokenAddress, getMint } from "@solana/spl-token";
import BigNumber from "bignumber.js";
import products from "./products.json";

// devネット上のUSDCトークンのアドレスを設定します。
const usdcAddress = new PublicKey("Gh9ZwEmdLJ8DscKNTkTqPbNwLNNBjuSzaG9Vp2KGtKJr");
// このウォレットアドレスを書き換えましょう!
const sellerAddress = "あなたのウォレットアドレス";
const sellerPublicKey = new PublicKey(sellerAddress);

const createTransaction = async (req, res) => {
  try {
    const { buyer, orderID, itemID } = req.body;
    if (!buyer) {
      res.status(400).json({
        message: "Missing buyer address",
      });
    }

    if (!orderID) {
      res.status(400).json({
        message: "Missing order ID",
      });
    }

    const itemPrice = products.find((item) => item.id === itemID).price;

    if (!itemPrice) {
      res.status(404).json({
        message: "Item not found. please check item ID",
      });
    }

    const bigAmount = BigNumber(itemPrice);
    const buyerPublicKey = new PublicKey(buyer);

    const network = WalletAdapterNetwork.Devnet;
    const endpoint = clusterApiUrl(network);
    const connection = new Connection(endpoint);

    const buyerUsdcAddress = await getAssociatedTokenAddress(usdcAddress, buyerPublicKey);
    const shopUsdcAddress = await getAssociatedTokenAddress(usdcAddress, sellerPublicKey);
    const { blockhash } = await connection.getLatestBlockhash("finalized");

    // 転送するトークンのミントアドレスを取得しています。
    const usdcMint = await getMint(connection, usdcAddress);

    const tx = new Transaction({
      recentBlockhash: blockhash,
      feePayer: buyerPublicKey,
    });

    // SOLとは異なるタイプの命令を作成しています。
    const transferInstruction = createTransferCheckedInstruction(
      buyerUsdcAddress,
      usdcAddress,     // 転送するトークンのアドレスです。
      shopUsdcAddress,
      buyerPublicKey,
      bigAmount.toNumber() * 10 ** (await usdcMint).decimals,
      usdcMint.decimals // トークンには任意の数の小数を含めることができます。
    );

    // 以下の変更はなしです。
    transferInstruction.keys.push({
      pubkey: new PublicKey(orderID),
      isSigner: false,
      isWritable: false,
    });

    tx.add(transferInstruction);

    const serializedTransaction = tx.serialize({
      requireAllSignatures: false,
    });

    const base64 = serializedTransaction.toString("base64");

    res.status(200).json({
      transaction: base64,
    });
  } catch (err) {
    console.error(err);

    res.status(500).json({ error: "error creating transaction" });
    return;
  }
};

export default function handler(req, res) {
  if (req.method === "POST") {
    createTransaction(req, res);
  } else {
    res.status(405).end();
  }
}
```

最初に追加したのは faucet から取得（記載のある）した devnet 上に存在する USDC トークンのトークンアドレスです。

```jsx
const usdcAddress = new PublicKey("Gh9ZwEmdLJ8DscKNTkTqPbNwLNNBjuSzaG9Vp2KGtKJr");
```

このアドレスを使用して、USDC トークンアカウントのアドレスを探して扱えるようにします。

Solana 上の代替可能なトークンは、[トークンプログラム](https://spl.solana.com/token)により作成されます。

これは、各トークンが独自のアカウント（アドレス）を持つことを意味します。

Solana アカウントをホテル、各トークンのアカウントをホテルの部屋と考えると、ホテルの部屋の中を見るためには部屋番号が必要になります。

つまり、トークンアカウントからトークンを送信するためには、そのトークンのアドレスを知る必要があるということです。

※ユーザーアカウントに USDC が入っていない場合、USDC トークンアドレスが存在しないと認識されてしまうため、送信先・送信元の両方のユーザーアカウントに USDC が必要となります。

```jsx
    const transferInstruction = createTransferCheckedInstruction(
      buyerUsdcAddress,
      usdcAddress,
      shopUsdcAddress,
      buyerPublicKey,
      bigAmount.toNumber() * 10 ** (await usdcMint).decimals,
      usdcMint.decimals
    );
```

ちなみに、上記の部分を任意の[SPLトークン](https://spl.solana.com/)に置き換えても機能します。

`Buy now →` ボタンをクリックすると、トークン名（アドレスで表示される）「Gh9ZwEmdLJ8DscKNTkTqPbNwLNNBjuSzaG9Vp2KGtKJr」のリクエストが表示されるはずです。

これは**偽の USDC トークンのアドレス**ですが、メインネットではこれが USDC となるわけです。

説明だけでは分かりづらいところがあるので、`sellerAddress` に設定したアドレス A とは別のアドレス B で Web アプリケーションに接続し、実際に `Buy now →` ボタンを押して購入してみてください。

アドレス B から、所定の金額がアドレス A に送金されていることが分かるはずです（初期設定では金額が 0.09 USDC となっているので、分かりやすいように 10.00 USDC に変更して試してみてください）。

以上で、USDC での支払いの準備が整いました!


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

次のレッスンでは、注文情報の保存機能を実装します!

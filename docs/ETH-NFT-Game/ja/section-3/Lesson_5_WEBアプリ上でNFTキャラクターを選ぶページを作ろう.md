### 🐱 NFT キャラクターをフロントエンドに表示する

前回のレッスンでは、Web アプリケーションからスマートコントラクトを呼び出すコードを実装し、 `SelectCharacter` コンポーネントを作成しました。

これから、スマートコントラクトから NFT キャラクターを取得してフロントエンドに表示させていきましょう。

### 👀 `deploy.js` を整理する

Web アプリケーションの開発を進める前に、`epic-game/scripts` にある、`deploy.js` ファイルを整理しましょう。

`mintCharacterNFT` や `attackBoss` 関数を排除していきます。

もうすでに排除されている場合は、このプロセスをスキップして問題ありません。

`deploy.js` が下記のようになっていることを確認したら、次に進みましょう。

```javascript
// deploy.js
const main = async () => {
  const gameContractFactory = await hre.ethers.getContractFactory("MyEpicGame");

  const gameContract = await gameContractFactory.deploy(
    ["ZORO", "NAMI", "USOPP"], // キャラクターの名前
    [
      "https://i.imgur.com/TZEhCTX.png", // キャラクターの画像
      "https://i.imgur.com/WVAaMPA.png",
      "https://i.imgur.com/pCMZeiM.png",
    ],
    [100, 200, 300],
    [100, 50, 25],
    "CROCODILE", // Bossの名前
    "https://i.imgur.com/BehawOh.png", // Bossの画像
    10000, // Bossのhp
    50 // Bossの攻撃力
  );
  // ここでは、nftGame コントラクトが、
  // ローカルのブロックチェーンにデプロイされるまで待つ処理を行っています。
  const nftGame = await gameContract.deployed();

  console.log("Contract deployed to:", nftGame.address);
};
const runMain = async () => {
  try {
    await main();
    process.exit(0);
  } catch (error) {
    console.log(error);
    process.exit(1);
  }
};
runMain();
```

`deploy.js` を整理することにより、フロントエンドにおける状態エラーを防ぐことができます。

`deploy.js` を更新した後、もう一度スマートコントラクトをデプロイすると、これから Web アプリケーション上で NFT キャラクターを Mint するプロセスがスムーズになります。

復習も兼ねて、下記を実行していきましょう。

**1 \. 再度、コントラクトをデプロイする。**

- `npx hardhat run scripts/deploy.js --network goerli` を実行する必要があります。

**2 \. フロントエンド（ `constants.js`）の `CONTRACT_ADDRESS` を更新する。**

**3 \. の ABI ファイルを更新する。**

- `epic-game/artifacts/contracts/MyEpicGame.sol/MyEpicGame.json` の中身を新しく作成する `nft-game-starter-project/src/utils/MyEpicGame.json` の中に貼り付ける必要があります。

### ♻️ `index.js` を更新する

`nft-game-starter-project/src/Components/SelectCharacter` にある `index.js` は、プログラムの中で何度も登場する変数や関数をまとめているファイルです。

これから、`index.js` の中身を更新していきます。

まず、`index.js` の `import` の部分を下記のように更新してください。

```javascript
// index.js
import React, { useEffect, useState } from "react";
import "./SelectCharacter.css";
import { ethers } from "ethers";
import { CONTRACT_ADDRESS, transformCharacterData } from "../../constants";
import myEpicGame from "../../utils/MyEpicGame.json";
```

次に、`SelectCharacter` を下記のように更新しましょう。

```javascript
// index.js
// SelectCharacter コンポーネントを定義しています。
const SelectCharacter = ({ setCharacterNFT }) => {
  const [characters, setCharacters] = useState([]);
  const [gameContract, setGameContract] = useState(null);

  // ページがロードされた瞬間に下記を実行します。
  useEffect(() => {
    const { ethereum } = window;
    if (ethereum) {
      const provider = new ethers.providers.Web3Provider(ethereum);
      const signer = provider.getSigner();
      const gameContract = new ethers.Contract(
        CONTRACT_ADDRESS,
        myEpicGame.abi,
        signer
      );

      // gameContract の状態を更新します。
      setGameContract(gameContract);
    } else {
      console.log("Ethereum object not found");
    }
  }, []);

  return (
    <div className="select-character-container">
      <h2>⏬ 一緒に戦う NFT キャラクターを選択 ⏬</h2>
    </div>
  );
};
export default SelectCharacter;
```

追加したコードを詳しく見ていきましょう。

```javascript
// index.js
const [characters, setCharacters] = useState([]);
const [gameContract, setGameContract] = useState(null);
```

ここでは、いくつかの状態変数を設定していきます。

- `characters` : コントラクトから返される NFT キャラクターのメタデータを保持するプロパティ。

- `setCharacters` : `characters` の状態を更新するプロパティ。

- `gameContract` : コントラクトの状態を初期化して保存するプロパティ。

  プログラムの中でコントラクトは複数回呼び出されるので、いったん初期化した状態で保存し、コントラクト全体で使用できるようにします。

- `setGameContract` : `gameContract` の状態を更新するプロパティ。

次に、下記のコードを見ていきましょう。

```javascript
// index.js
// ページがロードされた瞬間に下記を実行します。
useEffect(() => {
  const { ethereum } = window;
  if (ethereum) {
    const provider = new ethers.providers.Web3Provider(ethereum);
    const signer = provider.getSigner();
    const gameContract = new ethers.Contract(
      CONTRACT_ADDRESS,
      myEpicGame.abi,
      signer
    );
    // gameContract の状態を更新します。
    setGameContract(gameContract);
  } else {
    console.log("Ethereum object not found");
  }
}, []);
```

ここでは `useEffect` を使って、`SelectCharacter` コンポーネントが呼び出されたら、すぐに `gameContract` を作成して、使用できるようにしています。

この処理により、フロントエンドで NFT キャラクターを表示する準備が整います。

### 😎 NFT キャラクターのデータを取得する

NFT キャラクターのデータをスマートコントラクトから取得するために、`getCharacters` 関数を作成します。

`gameContract` を使用する準備ができたら、すぐに `getCharacters` 関数を呼び出したいので、ここでも `useEffect` を使用していきます。

それでは、`SelectCharacter` の中に記載した `useEffect` 関数を確認しましょう。


```javascript
// index.js
useEffect(() => {
	const { ethereum } = window;

	if (ethereum) {
	  const provider = new ethers.providers.Web3Provider(ethereum);
	  const signer = provider.getSigner();
	  const gameContract = new ethers.Contract(
		CONTRACT_ADDRESS,
		myEpicGame.abi,
		signer
	  );
	  // gameContract の状態を更新します。
	  setGameContract(gameContract);
	} else {
	  console.log('Ethereum object not found');
	}
  	}, []);
```

この関数の直下に、下記を追加していきましょう。

```javascript
// index.js
useEffect(() => {
  // NFT キャラクターのデータをスマートコントラクトから取得します。
  const getCharacters = async () => {
    try {
      console.log("Getting contract characters to mint");
      // ミント可能な全 NFT キャラクター をコントラクトをから呼び出します。
      const charactersTxn = await gameContract.getAllDefaultCharacters();

      console.log("charactersTxn:", charactersTxn);

      // すべてのNFTキャラクターのデータを変換します。
      const characters = charactersTxn.map((characterData) =>
        transformCharacterData(characterData)
      );

      // ミント可能なすべてのNFTキャラクターの状態を設定します。
      setCharacters(characters);
    } catch (error) {
      console.error("Something went wrong fetching characters:", error);
    }
  };
  // gameContractの準備ができたら、NFT キャラクターを読み込みます。
  if (gameContract) {
    getCharacters();
  }
}, [gameContract]);
```

コードの中身を見ていきましょう。

```javascript
// index.js
// ミント可能な全 NFT キャラクター をコントラクトをから呼び出します。
const charactersTxn = await gameContract.getAllDefaultCharacters();
```

ここでは、`gameContract` を使用して、`MyEpicGame.sol` に記載した`getAllDefaultCharacters` 関数を呼び出しています。

> ✍️: `getAllDefaultCharacters` は、3 体の NFT キャラクターのデフォルト情報を取得する関数です。

次に、下記のコードを見ていきましょう。

```javascript
// index.js
// すべてのNFTキャラクターのデータを変換します。
const characters = charactersTxn.map((characterData) =>
  transformCharacterData(characterData)
);
```

ここでは、`transformCharacterData` を使用して、NFT キャラクターのデータを Web アプリケーションで扱えるオブジェクトに変換しています。

> ✍️: `map()` の使い方
> `map()` は配列データに使うメソッドです。
> `map()` メソッドを使って、配列に入っている NFT キャラクターそれぞれの属性情報（ HP など）に対して `transformCharacterData` を実行し、その結果を新しい配列（ `characters` ）として返しています。

次に下記のコードを見ていきましょう。

```javascript
// index.js
// ミント可能なすべてのNFTキャラクターの状態を設定します。
setCharacters(characters);
```

ここでは、コントラクトから取得した NFT キャラクターのデータを状態として保存しています。

この処理により、NFT キャラクターのデータをフロントエンドで使い始めることができます。

最後に、下記のコードを見ていきましょう。

```javascript
// index.js
// gameContractの準備ができたら、NFT キャラクターを読み込みます。
if (gameContract) {
  getCharacters();
}
```

ここでは、`gameContract` が更新されるたびに、中身が `null` でないことを確認し、`getCharacters` 関数を呼び出す処理を実装しています。

この処理により、NFT キャラクターのデータが更新されるたびに、キャラクターの状態を更新て、フロントエンドに反映させることができます。

### ⚡️ Web アプリケーション上でテストを行う

Web アプリケーション上で、NFT キャラクターの情報が取得できているか、確認してみましょう。

ローカル環境でホストされている Web アプリケーション上で、`Inspect` を実行し、Console に向かいましょう。

Web アプリケーションをリフレッシュして、ウォレット接続が完了したら、下記のような結果が Console に出力されているか確認してください。

```
charactersTxn:
(3) [Array(6), Array(6), Array(6)]
0: (6) [BigNumber, 'ZORO', 'https://i.imgur.com/TZEhCTX.png', BigNumber, BigNumber, BigNumber, characterIndex: BigNumber, name: 'ZORO', imageURI: 'https://i.imgur.com/TZEhCTX.png', hp: BigNumber, maxHp: BigNumber, …]
1: (6) [BigNumber, 'NAMI', 'https://i.imgur.com/WVAaMPA.png', BigNumber, BigNumber, BigNumber, characterIndex: BigNumber, name: 'NAMI', imageURI: 'https://i.imgur.com/WVAaMPA.png', hp: BigNumber, maxHp: BigNumber, …]
2: (6) [BigNumber, 'USOPP', 'https://i.imgur.com/pCMZeiM.png', BigNumber, BigNumber, BigNumber, characterIndex: BigNumber, name: 'USOPP', imageURI: 'https://i.imgur.com/pCMZeiM.png', hp: BigNumber, maxHp: BigNumber, …]
length: 3
[[Prototype]]: Array(0)
```

上記のような結果が Console に表示されていればテストは成功です。

### 👓 NFT キャラクター を Web アプリケーションにレンダリングする

それでは、NFT キャラクターの情報を Web アプリケーションに反映させていきましょう。

まず、`index.js` を開き、`SelectCharacter` コンポーネントの中に、`renderCharacters` メソッドを追加しましょう。

- 2 つ目に作成した、`useEffect` 関数の直下に、下記を貼り付けてください。

```javascript
// index.js
// NFT キャラクターをフロントエンドにレンダリングするメソッドです。
const renderCharacters = () =>
  characters.map((character, index) => (
    <div className="character-item" key={character.name}>
      <div className="name-container">
        <p>{character.name}</p>
      </div>
      <img src={character.imageURI} alt={character.name} />
      <button
        type="button"
        className="character-mint-button"
        //onClick={mintCharacterNFTAction(index)}
      >{`Mint ${character.name}`}</button>
    </div>
  ));
```

> ⚠️: 注意
>
> エラー処理を円滑にするため、`onClick={mintCharacterNFTAction(index)}` はコメントアウトしたままにしてください。
> 後で解除します。

次に、`index.js` の中の `return();` の中身を下記のように更新してください。

```javascript
// index.js
return (
  <div className="select-character-container">
    <h2>⏬ 一緒に戦う NFT キャラクターを選択 ⏬</h2>
    {/* キャラクターNFTがフロントエンド上で読み込めている際に、下記を表示します*/}
    {characters.length > 0 && (
      <div className="character-grid">{renderCharacters()}</div>
    )}
  </div>
);
```

それでは、Web アプリケーションをリフレッシュして、下記のように NFT キャラクターがフロントエンドに反映されていることを確認してください。

![](/public/images/ETH-NFT-Game/section-3/3_5_1.png)

### ✨ Web アプリケーションから NFT キャラクター を Mint する

これから、NFT キャラクターを Mint する `mintCharacterNFTAction` 関数を作成していきます。

`index.js` を開き、`const [gameContract, setGameContract] = useState(null);` の直下に下記を追加しましょう。

```javascript
// index.js
// NFT キャラクターを Mint します。
const mintCharacterNFTAction = (characterId) => async () => {
  try {
    if (gameContract) {
      console.log("Minting character in progress...");
      const mintTxn = await gameContract.mintCharacterNFT(characterId);
      await mintTxn.wait();
      console.log("mintTxn:", mintTxn);
    }
  } catch (error) {
    console.warn("MintCharacterAction Error:", error);
  }
};
```

> ⚠️: 注意
>
> `renderCharacters` 関数の中にある `onClick = {mintCharacterNFTAction(index)}` のコメントアウトを解除してください。

`mintCharacterNFTAction` 関数は、`MyEpicGame.sol` に記載されている `mintCharacterNFT` 関数を呼び出します。

- どの NFT キャラクターを Mint するかコントラクトに伝えるために、そのキャラクターのインデックス（`characterId`）を引数として取り巻す。

- `onClick = {mintCharacterNFTAction(index)}` の `index` が NFT キャラクターのインデックスです。

### 🏓 コントラクトで `emit` された `event` をフロントエンドで受け取る

NFT キャラクターが Mint されたことをフロントエンドに伝える `event` をコントラクト上に作成したことを覚えてますか？

これから、Web アプリケーション上で `event` の情報を「キャッチ」するコードを実装していきます。

`index.js` 内で `getCharacters` 関数を定義した `useEffect` のコードブロックを下記のように編集してください。

```javascript
// index.js
useEffect(() => {
  // NFT キャラクターのデータをスマートコントラクトから取得します。
  const getCharacters = async () => {
    try {
      console.log("Getting contract characters to mint");

      // ミント可能な全 NFT キャラクター をコントラクトをから呼び出します。
      const charactersTxn = await gameContract.getAllDefaultCharacters();

      console.log("charactersTxn:", charactersTxn);

      // すべてのNFTキャラクターのデータを変換します。
      const characters = charactersTxn.map((characterData) =>
        transformCharacterData(characterData)
      );

      // ミント可能なすべてのNFTキャラクターの状態を設定します。
      setCharacters(characters);
    } catch (error) {
      console.error("Something went wrong fetching characters:", error);
    }
  };

  // イベントを受信したときに起動するコールバックメソッド onCharacterMint を追加します。
  const onCharacterMint = async (sender, tokenId, characterIndex) => {
    console.log(
      `CharacterNFTMinted - sender: ${sender} tokenId: ${tokenId.toNumber()} characterIndex: ${characterIndex.toNumber()}`
    );
    // NFT キャラクターが Mint されたら、コントラクトからメタデータを受け取り、アリーナ（ボスとのバトルフィールド）に移動するための状態に設定します。
    if (gameContract) {
      const characterNFT = await gameContract.checkIfUserHasNFT();
      console.log("CharacterNFT: ", characterNFT);
      setCharacterNFT(transformCharacterData(characterNFT));
    }
  };

  if (gameContract) {
    getCharacters();
    // リスナーの設定：NFT キャラクターが Mint された通知を受け取ります。
    gameContract.on("CharacterNFTMinted", onCharacterMint);
  }

  return () => {
    // コンポーネントがマウントされたら、リスナーを停止する。

    if (gameContract) {
      gameContract.off("CharacterNFTMinted", onCharacterMint);
    }
  };
}, [gameContract]);
```

新しく追加したコードを詳しく見ていきましょう。

```javascript
//index.js
// イベントを受信したときに起動するコールバックメソッド onCharacterMint を追加します。
const onCharacterMint = async (sender, tokenId, characterIndex) => {
  console.log(
    `CharacterNFTMinted - sender: ${sender} tokenId: ${tokenId.toNumber()} characterIndex: ${characterIndex.toNumber()}`
  );
  // NFT キャラクターが Mint されたら、コントラクトからメタデータを受け取り、アリーナ（ボスとのバトルフィールド）に移動するための状態に設定します。
  if (gameContract) {
    const characterNFT = await gameContract.checkIfUserHasNFT();
    console.log("CharacterNFT: ", characterNFT);
    setCharacterNFT(transformCharacterData(characterNFT));
  }
};
```

下記は、`MyEpicGame.sol` に記載した NFT キャラクターが Mint された `event` をフロントエンドに知らせる（`emit`）コードです。

> ```solidity
> // MyEpicGame.sol
> // ユーザーが NFT を Mint したこと示すイベント
> event CharacterNFTMinted(address sender, uint256 tokenId, uint256 characterIndex);
> // ユーザーが NFT を Mint したことをフロントエンドに伝えます。
> emit CharacterNFTMinted(msg.sender, newItemId, _characterIndex);
> ```

`onCharacterMint` メソッドは、このイベントをキャッチするので、新しい NFT キャラクターが Mint されるたびに呼び出されます。

次に、下記のコードを詳しく見ていきましょう。

```javascript
// index.js, onCharacterMint()
// NFT キャラクターが Mint されたら、コントラクトからメタデータを受け取り、アリーナ（ボスとのバトルフィールド）に移動するための状態に設定します。
if (gameContract) {
  const characterNFT = await gameContract.checkIfUserHasNFT();
  console.log("CharacterNFT: ", characterNFT);
  setCharacterNFT(transformCharacterData(characterNFT));
}
```

まず、`if (gameContract)` で NFT キャラクターがすでに Mint されていることを確認しています。

ユーザーがすでに NFT キャラクターを Mint している場合は、`checkIfUserHasNFT` 関数を使って、コントラクト（`gameContract`）に保存されているその NFT キャラクターのメタデータ（HP など）を `characterNFT` に格納します。

`setCharacterNFT(transformCharacterData(characterNFT));` は、コントラクトに保存されているメタデータをフロントエンドで扱えるオブジェクト形式に変換する処理です。

**これらがすべて完了すると、NFT キャラクターがボスとバトルすることになる `Area` コンポーネントでメタデータが使用できます。**

次に、下記のコードを見ていきましょう。

```javascript
// index.js
if (gameContract) {
  getCharacters();
  // リスナーの設定：NFT キャラクターが Mint された通知を受け取ります。
  gameContract.on("CharacterNFTMinted", onCharacterMint);
}
```

`gameContract.on('CharacterNFTMinted', onCharacterMint)` により、フロントエンドは、`CharacterNFtMinted` イベントがコントラクトから発信されたときに、情報を受け取ります。これにより、情報がフロントエンドに反映されます。

- このことを、**コンポーネント（情報）がマウント（フロントエンドに反映）される**と言います。

- また、フロントエンドでイベントを受信する機能のことを「リスナ」と呼びます。

フロントエンドで `CharacterNFtMinted` イベントが受信されると、`onCharacterMint` メソッドが実行されます。

最後に、下記のコードを見ていきましょう。

```javascript
// index.js
return () => {
  // コンポーネントがマウントされたら、リスナーを停止する。
  if (gameContract) {
    gameContract.off("CharacterNFTMinted", onCharacterMint);
  }
};
```

ここでは、NFT キャラクターが一度 Mint された後、`CharacterNFTMinted` の受信を停止する処理を行っています。

コンポーネントがマウントされる状態をそのままにしておくと、メモリリーク（コンピュータを動作させている内に、使用可能なメモリの容量が減っていってしまう現象）が発生する可能性があります。

メモリリークを防ぐために、`gameContract.off('CharacterNFTMinted', onCharacterMint)` では、`onCharacterMint` 関数の稼働をやめています。これは、イベントリスナをやめることを意味しています。

### 🐝 OpenSea で Mint した NFT キャラクターを確認する

それでは、Web アプリケーションから、キャラクターをいったい Mint して、[テストネット用の OpenSea](https://testnets.opensea.io/) に反映されるか確認していきましょう。

**1️⃣ NFT キャラクターを Mint する**

Web アプリケーション上で、`Mint` ボタンを押したら、MetaMask 上で承認作業（`Confirm`）を行ってください。

Mint が成功した後に、Web アプリケーションをリフレッシュすると下記のような結果が、Console に表示されます。

```
Checking for Character NFT on address: 0x3a0a49fb3cf930e599f0fa7abe554dc18bd1f135

Getting contract characters to mint

User has character NFT
```

このような結果が表示されているということは、あなたのウォレットアドレスに NFT キャラクターが存在していることになります。

**2️⃣ OpenSea で NFT キャラクターを確認する**

[テストネット用の OpenSea](https://testnets.opensea.io/) で、NFT キャラクターを参照してみましょう。

あなたの `CONTACT_ADDRESS` と `TOKEN_ID` を取得して、下記のアドレスを更新したら、ブラウザに貼り付けてみてください。

```
https://testnets.opensea.io/assets/CONTRACT_ADDRES/TOKEN_ID
```

下記のように、オンライン上でもあなたの NFT キャラクターが表示されることを確認しましょう（画像は学習コンテンツ制作時に利用した Rarible rinkeby testnet のものになります）。

![](/public/images/ETH-NFT-Game/section-3/3_5_2.png)

### 🪄 おまけ

ユーザーに NFT キャラクターを確認する OpenSea リンクを発行しましょう。

`index.js` を開いて、`onCharacterMint` 関数の中身を変更します。

- `setCharacterNFT(transformCharacterData(characterNFT));` の直下に下記を追加しましょう。

```javascript
// index.js
alert(
  `NFT キャラクーが Mint されました -- リンクはこちらです: https://testnets.opensea.io/assets/${
    gameContract.address
  }:${tokenId.toNumber()}?tab=details`
);
```

### 🙋‍♂️ 質問する

ここまでの作業で何かわからないことがある場合は、Discord の `#eth-nft-game` で質問をしてください。

ヘルプをするときのフローが円滑になるので、エラーレポートには下記の 3 点を記載してください ✨

```
1. 質問が関連しているセクション番号とレッスン番号
2. 何をしようとしていたか
3. エラー文をコピー&ペースト
4. エラー画面のスクリーンショット
```

---

次のレッスンに進んで、ボスとのバトルフィールドを実装しましょう 🎉

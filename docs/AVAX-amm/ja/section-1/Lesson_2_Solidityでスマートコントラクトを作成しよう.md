section1 のこれから先の作業は, `Avalanche-AMM/contract`ディレクトリをルートディレクトリとして話を進めます。 🙌

### 👩‍💻 実装する内容の確認

AMM 機能を実装したスマートコントラクトを作成していきます。

ユーザは私たちのスマートコントラクトを利用して, トークンの交換をすることができます。

具体的には, 私たちのスマートコントラクトが USDC というトークンと JOE というトークンを交換することができる場合,
ユーザはスマートコントラクトへ接続し, USDC から JOE トークンへ(または JOE トークンから USDC トークンへ)交換することができます。

AMM を実装する上で重要なキーワードは以下の 3 つです。

🐦 プール

スマートコントラクトで交換可能なトークンの集まりのことを指します。

スマートコントラクトに USDC と JOE のプールが存在する場合, ユーザは USDC と JOE 間で取引をすることができます。

🦒 流動性の提供

プール内にあるトークン量が少なく(= 取引する際の価格変動が大きくなる), 取引が成立しずらい市場の状態を流動性が低いという表現を使います。
逆にトークン量が多く(= 価格変動が小さい), 取引がスムーズに成立しやすい状態を流動性が高いといいます。

多くの AMM では, 流動性の高い状態を作るために, トークン保有者にそのトークンをプールに預けてもらう(流動性の提供という)仕組みを用意しています。

例えば, USDC と JOE のプールがある場合は USDC と JOE を所有している人に両トークンをプールへ預けてもらいます。
(2 つのトークンを提供しなければいけないかどうかは DEX によります)。

流動性の提供者には様々な報酬が用意されています。
本プロジェクトでは, ユーザがトークンを交換する際に手数料を徴収し, その手数料を流動性の提供者へ与えるという形で報酬を実装します。

🦍 スワップ

トークンを交換することを指します。

---

本プロジェクトで実装する機能を整理します。

1. トークン保有者は流動性を提供することができる。
2. 流動性を提供したユーザは預けたトークンを引き出すことができる。
3. ユーザはトークンをスワップすることができる。
4. スワップには手数料がかかる。
5. スワップで発生した手数料は, 流動性提供者へ分配される。

### 🥮 コントラクトを作成する

今回作成するコントラクトは 3 つです。

本プロジェクトのスマートコントラクトとなる `AMMコントラクト` が 1 つと, AMM コントラクトの動きをシミュレーションするための `ERC20Tokenコントラクト` が 2 つです。

`AMMコントラクト` は `Fuji C-Chain` にデプロイするため, すでに `Fuji C-Chain` に存在するトークンを使用して `AMMコントラクト` を動かすことは可能ですが,
`ERC20` を実装したコントラクトを自前でデプロイし, トークン発行を自由に行えるとトークン取得のプロセスが柔軟で簡単になります。

`contracts` ディレクトリの下に `ERC20Tokens.sol`, `AMM.sol` という名前のファイルを作成します。

Hardhat を使用する場合ファイル構造は非常に重要ですので, 注意する必要があります。
ファイル構造が下記のようになっていれば大丈夫です 😊

```bash
contract
    |_ contracts
           ├── AMM.sol
           └── ERC20Tokens.sol
```

次に, コードエディタでプロジェクトのコードを開きます。

`ERC20Tokens.sol` の中に以下のコードを貼り付けてください。

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract USDCToken is ERC20 {
    constructor() ERC20("USDC Token", "USDC") {
        _mint(msg.sender, 10000 ether);
    }

    function faucet(address recipient, uint256 amount) external {
        _mint(recipient, amount);
    }
}

contract JOEToken is ERC20 {
    constructor() ERC20("JOE Token", "JOE") {
        _mint(msg.sender, 10000 ether);
    }

    function faucet(address recipient, uint256 amount) external {
        _mint(recipient, amount);
    }
}
```

`0.8.17` が `hardhat.config.ts` でも記載されていることを確認してください。

もし,`hardhat.config.ts` の中に記載されている Solidity のバージョンが`0.8.17` でなかった場合は,`Messenger.sol`の中身を`hardhat.config.ts` に記載されているバージョンに変更しましょう。

ここでは 2 つの `ERC20Tokenコントラクト` を作成しています。

`USDCToken` と `JOEToken` です。

`USDCToken` を例に中身を見ます。

```solidity
contract USDCToken is ERC20 {
    constructor() ERC20("USDC Token", "USDC") {
        _mint(msg.sender, 10000 ether);
    }

    function faucet(address recipient, uint256 amount) external {
        _mint(recipient, amount);
    }
}
```

コントラクト `USDCToken` は `ERC20` を継承しているため, `ERC20` の関数を実装しています。

[ERC20](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol) はトークンの規格です。

`USDCToken` のコンストラクタで `ERC20` のコンストラクタを引数とともに呼んでおり, ここでトークンのシンボルなどを指定しています。

コンストラクタの中では, コントラクトをデプロイしたアカウントに `10000 ether (= 10000 x 10^18)` 分 `USDC` を発行しています。
※ `USDC` と `JOE` は最小単位を ether と同じように扱うことにします。

`_mint` の実装は [ERC20](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol) の中にあります。
内部では, コントラクトの保持する 2 つの値(トークン総量を表す値と, 指定されたアカウントの所有トークン量を表す値)に指定されたトークンの量だけ加算しています。

`USDCToken` は `faucet` という関数も実装し, 中で `_mint` を呼び出しています。

こちらは指定したアカウントに簡単にトークンを発行することができるように用意しました。

次に `AMM.sol` のなかに以下のコードを貼り付けてください。

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "hardhat/console.sol";

contract AMM {
    IERC20 tokenX; // ERC20を実装したコントラクト
    IERC20 tokenY; // ERC20を実装したコントラクト
    uint256 public totalShare; // シェアの総量
    mapping(address => uint256) public share; // 各ユーザのシェア
    mapping(IERC20 => uint256) public totalAmount; // プールにロックされた各トークンの量

    uint256 public constant PRECISION = 1_000_000; // シェアの精度に使用する定数(= 6桁)

    // プールに使えるトークンを指定します。
    constructor(IERC20 _tokenX, IERC20 _tokenY) {
        tokenX = _tokenX;
        tokenY = _tokenY;
    }
}
```

ファイル上部では, `openzeppelin/contracts` から `IERC20.sol` をインポートし, `IERC20` を使用できるようにしています。

その下に続く `AMMコントラクト` の実装では `IERC20` を型としたオブジェクトを 2 つ保持しています。

```solidity
contract AMM {
    IERC20 tokenX; // ERC20を実装したコントラクト
    IERC20 tokenY; // ERC20を実装したコントラクト
    ...
}
```

この 2 つのオブジェクトはそれぞれ, 私たちが作る AMM が用意するプールのトークンペアのコントラクトです。
本プロジェクトでは `USDCToken` と `JOEToken` のアドレスをここに記録して AMM を動かします。

[IERC20](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/IERC20.sol) とは `ERC20` のインタフェースであり, `ERC20` の関数のみ定義された(= ERC20 がどのように振る舞うのかのみわかる) フレームのようなものです。

`AMMコントラクト` は tokenX と tokenY に特定のコントラクトを指定する必要はなく,
「`ERC20` の関数を実装しているコントラクト」を保持すれば良いので, インタフェースで型を用意しています。

コントラクトから別のコントラクトの関数を呼び出す場合はインタフェースを使うことで呼び出すことができます。

例えば `IERC20 tokenX` の関数を呼び出す際はこのように呼び出すことができます。

```solidity
tokenX.transfer()
```

[インターフェースの参考記事](https://medium.com/coinmonks/solidity-tutorial-all-about-interfaces-f547d2869499)

---

📓 シェアについて

次にシェアに関する状態変数について見ていきます。

本プロジェクトで使うシェアという言葉は, 流動性の提供者がプールにどのくらいのトークンを預けているのかを表す数値を指します。
DEX で用意されていることが多い LP トークンと同じ役目を果たします。

今回は簡単に実装するため, コントラクト内で数値として保持します。

> 📓 LP トークンとは
> Liquidity Provider( = 流動性提供者) トークンの略です。
> 流動性を提供したことを証明する債券のようなものであり,
> 預けたトークンを返却して欲しい時には, LP トークンを提示することにより, 元の資産および利子を受け取ることができます。

シェアに関する状態変数は以下です。

```solidity
uint256 public totalShare; // シェアの総量
mapping(address => uint256) public share; // 各ユーザのシェア

uint256 public constant PRECISION = 1_000_000; // シェアの精度に使用する定数(= 6桁)
```

`totalShare` は全てのユーザのシェアの合計値で, `share` は各ユーザのシェアの値を表します。

例えば, あるユーザ(アドレスを`addr`で表す)の提供しているトークン量の, 全てのユーザの提供量に対する割合を考える場合は $\frac{share[addr]}{totalShare}$ で求めることができます。
この考え方は, 流動性の提供者が預けているトークンを引き出す時に, そのトークン量を計算する際に使います。

`PRECISION` は share の小数点以下を表すもので 6 桁分用意しています。
share が 1.23 あるとすると, コントラクト内では 1_230_000 として保持します。
ethereum トークンにおける `ether` と `wei` の関係のようなものです。
見やすくなるため`1_000_000`で記載していますが, `1000000`と同じ意味です。

---

また, `totalAmount` という状態変数も用意していますが, 提供されたトークンの量をそれぞれのトークンに対し保持します。

最後に, 私たちが作る `AMMコントラクト` は, デプロイ時にコンストラクタによってプールのトークンペアを決定する仕様とします。
そのため引数に２つのコントラクトアドレスを受け取れるようにしています。

### 🧪 テストを実装する

コントラクトを実装したのでテストを書きます。

`test` ディレクトの下に `AMM.ts` を作成し, 以下のコードを記述してください。

```ts
import { ethers } from "hardhat";
import { BigNumber } from "ethers";
import { expect } from "chai";
import { loadFixture } from "@nomicfoundation/hardhat-network-helpers";

describe("AMM", function () {
  async function deployContract() {
    const [owner, otherAccount] = await ethers.getSigners();

    const amountForOther = ethers.utils.parseEther("5000");
    const USDCToken = await ethers.getContractFactory("USDCToken");
    const usdc = await USDCToken.deploy();
    await usdc.faucet(otherAccount.address, amountForOther);

    const JOEToken = await ethers.getContractFactory("JOEToken");
    const joe = await JOEToken.deploy();
    await joe.faucet(otherAccount.address, amountForOther);

    const AMM = await ethers.getContractFactory("AMM");
    const amm = await AMM.deploy(usdc.address, joe.address);

    return {
      amm,
      token0: usdc,
      token1: joe,
      owner,
      otherAccount,
    };
  }

  describe("init", function () {
    it("init", async function () {
      const { amm } = await loadFixture(deployContract);

      expect(await amm.totalShare()).to.eql(BigNumber.from(0));
    });
  });
});
```

ここで行っていることはデプロイの流れを確認するくらいのシンプルなものです。

`deployContract` 関数内では, 3 つのコントラクトを順番にデプロイしています。

トークンのコントラクトのデプロイ時には, `owner` だけでなく `otherAccount` に対しても `faucet` を呼び出すことでトークンを発行しています。

`AMMコントラクト` のデプロイ時には, `USDCToken`, `JoeToken` をコンストラクタに渡しています。

以下の部分では, デプロイした `AMMコントラクト` の状態変数の初期値を確かめています。
※ コントラクトからの数値の返り値は BigNumber です。

```ts
expect(await amm.totalShare()).to.eql(BigNumber.from(0));
```

本格的なテストは次のレッスンから実装していきます。

💁 hardhat で行うテストに関して詳しくは[こちら](https://hardhat.org/hardhat-runner/docs/guides/test-contracts)を参考にしてください。

### ⭐ テストを実行しましょう

`contract`ディレクトリ直下で以下のコマンドを実行してください。

```
$ npx hardhat test
```

以下のような表示がされます。
実行したテスト名とそのテストがパスしたことがわかります。

![](/public/images/AVAX-amm/section-1/1_2_1.png)

### 🙋‍♂️ 質問する

ここまでの作業で何かわからないことがある場合は,Discord の `#avax-amm` で質問をしてください。

ヘルプをするときのフローが円滑になるので,エラーレポートには下記の 3 点を記載してください ✨

```
1. 質問が関連しているセクション番号とレッスン番号
2. 何をしようとしていたか
3. エラー文をコピー&ペースト
4. エラー画面のスクリーンショット
```

---

コントラクトの作成とテストまでの流れを知ることができました 🎉
次のレッスンでは,スマートコントラクトに機能を追加していきます。

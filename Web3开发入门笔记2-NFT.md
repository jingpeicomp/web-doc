# Web3 开发入门笔记 2 — NFT

## 1. 背景

在区块链上，数字加密货币分为原生币和代币两大类。前者如 BTC、ETH、Filecoin 等，拥有自己的主链，使用链上的交易来维护账本数据;代币则是依附于现有的区块链，使用智能合约来进行账本的记录，如依附于以太坊上而发布的各种代币。代币之中又可分为同质化和非同质化两种。

我们常见的 Token(如 BTC，ETH 等)都是同质化的，即 FT(Fungible Token)，互相可以替代、可接近无限拆分的 token。

NFT 是非同质化通证(代币)，具有不可分割、不可替代、独一无二等特点。

本节我们的主要完成的任务有：

1. MetaMask、Alchemy、IPFS 在 Goerli 以太坊测试链上创建和部署一个 ERC-721 智能合约，
2. 用该智能合约铸造 NFT
3. 在 MetaMask 中查看 NFT，并上架到 opensea

## 2. 开发环境安装

见 Web3 开发入门笔记 1

## 3. 创建和部署合约

### 3.1 Alchemy 平台

教程 1 我们使用的本地开发链，这次我们使用以太测试链。要和以太坊区块链进行交互，必须要连接到以太坊节点。Alchemy 是一个区块链开发平台，能够提供应用程序接口，让我们无需运行自己的节点，即可与以太坊区块链进行通信。

创建了 Alchemy 帐户后，可以通过创建应用程序来生成应用程序接口密钥。

![https://s1.ax1x.com/2022/12/06/z6gqeg.png](https://s1.ax1x.com/2022/12/06/z6gqeg.png)

![https://s1.ax1x.com/2022/12/06/z62SS0.png](https://s1.ax1x.com/2022/12/06/z62SS0.png)

### 3.2 MetaMask 钱包

#### 3.2.1 创建以太坊账户

下载 [MetaMask](https://metamask.io/download/) 浏览器插件，注册账号，并切换到 Goerli 测试网络。

![https://s1.ax1x.com/2022/12/06/z6gFPI.png](https://s1.ax1x.com/2022/12/06/z6gFPI.png)

#### 3.2.2 获取测试以太币

为了将智能合约部署到测试网络，需要一些虚拟以太币。 要获取以太币，您可以前往 [FaucETH](https://fauceth.komputing.org/) 并输入 MetaMask 中测试网络帐户地址，单击“Request funds”，然后在下拉菜单中选择“Gorli”，最后再次单击“Request funds”按钮，如果成功的话会在 MetaMask 帐户中看到以太币！

### 3.3 初始化项目

创建 nft-demo 文件夹，在文件夹下执行

```
cd nft-demo
truffle init
```

在根目录下创建`package.json`文件，填入如下内容：

```json
{
  "dependencies": {
    "@alch/alchemy-web3": "^1.4.7",
    "@openzeppelin/contracts": "^4.8.0",
    "@truffle/hdwallet-provider": "^2.1.2",
    "alchemy-sdk": "^2.2.3",
    "dotenv": "^16.0.3"
  },
  "name": "nft-demo",
  "version": "1.0.0",
  "directories": {
    "test": "test"
  },
  "author": "",
  "license": "ISC",
  "description": ""
}
```

在命令行中进入项目目录，执行`npm install`命令，安装依赖包。

### 3.4 编写合约

具体的合约开发见教程 1。

合约文件：FatherGift.sol

```sol
// SPDX-License-Identifier: MIT
// Tells the Solidity compiler to compile only from v0.8.13 to v0.9.0
pragma solidity ^0.8.13;

import "../node_modules/@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "../node_modules/@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
import "../node_modules/@openzeppelin/contracts/access/Ownable.sol";
import "../node_modules/@openzeppelin/contracts/utils/Counters.sol";

contract FatherGift is ERC721URIStorage, Ownable {
    using Counters for Counters.Counter;
    Counters.Counter private _tokenIds;

    constructor() ERC721("FatherDayGift", "NFT") {}

    function mintNFT(address recipient, string memory tokenURI)
        public
        onlyOwner
        returns (uint256)
    {
        _tokenIds.increment();

        uint256 newItemId = _tokenIds.current();
        _mint(recipient, newItemId);
        _setTokenURI(newItemId, tokenURI);

        return newItemId;
    }
}
```

部署文件：1_deploy_contracts.js

```js
const FatherGift = artifacts.require("FatherGift");

module.exports = function (deployer) {
  deployer.deploy(FatherGift);
};
```

### 3.5 连接 MetaMask 和 Alchemy

从虚拟钱包发送的每笔交易都需要使用私钥签名。 为了给程序提供此项许可，我们可以安全地将私钥（和 Alchemy 应用程序接口密钥）存储在一个环境文件中。在项目的根目录中创建 .env 文件，并将 Metamask 公钥、私钥和 Alchemy 应用程序接口网址添加到其中。

- MetaMask 账户地址
  ![https://s1.ax1x.com/2022/12/06/z64PIg.png](https://s1.ax1x.com/2022/12/06/z64PIg.png)
- 遵循[这些说明](https://metamask.zendesk.com/hc/en-us/articles/360015289632-How-to-Export-an-Account-Private-Key)，从 MetaMask 导出私钥
- 请从下方获取超文本传输协议 Alchemy 应用程序接口网址并将其复制到剪贴板
  ![https://ethereum.org/2fd7b78a5e6dadea7018633fa76b5c2a/copy-alchemy-api-url.gif](https://ethereum.org/2fd7b78a5e6dadea7018633fa76b5c2a/copy-alchemy-api-url.gif)

操作完后，.env 文件的内容应该如下：

```env
API_URL="https://eth-goerli.g.alchemy.com/v2/your-api-key"
PUBLIC_KEY="MetaMask账户地址"
PRIVATE_KEY="your-metamask-private-key"
```

不要提交 .env！ 请确保永远不要与任何人共享或公开 .env 文件，因为这样做会泄露您的秘密。 如果您使用 git，请将您的 .env 添加到 gitignore 文件中。

### 3.6 修改 truffle 网络配置

修改 truffle-config.js 文件，添加 goerli 网络相关配置。

```js
require("dotenv").config();
const { API_URL, PRIVATE_KEY } = process.env;

const HDWalletProvider = require("@truffle/hdwallet-provider");

module.exports = {
  networks: {
    goerli: {
      network_id: "*",
      provider: () =>
        new HDWalletProvider({
          privateKeys: [PRIVATE_KEY],
          providerOrUrl: API_URL,
        }),
    },
  },
};
```

### 3.7 部署合约

在命令行执行如下命令部署合约到以太坊 goerli 测试链上.

```sh
cd nft-demo
truffle migrate --network goerli
```

如果部署成功，会打印出合约相关信息。

在[https://goerli.etherscan.io/](https://goerli.etherscan.io/)网站搜索合约地址，会出现合约的详细信息。
![https://s1.ax1x.com/2022/12/06/z650BV.png](https://s1.ax1x.com/2022/12/06/z650BV.png)

## 4. 铸造 NFT

### 4.1 NFT 元数据存储方式

NFT 通过元数据为特定的标记 ID 提供描述性的附加信息。为了方便 NFT 的应用程序能够访问这些数据，如何以及在哪里存储这些数据呢？是链上存储还是链下存储？也就是说，是将元数据直接放入代表令牌的智能协议中，还是单独托管?

链上存储元数据的好处是:

它永久地驻留在通证中，超出了任何给定应用程序的生命周期;
它可以根据链上逻辑进行更改。
如果数字资产的长期价值远远超过其最初创造的价值，例如，一件数字艺术作品被认为会流传千古，那么就不管用来创作这件艺术作品的原始站点是否仍然存在。因此，NFT 的元数据必须与标记标识符的生命周期保持在一起。

尽管有这些好处，但由于以太坊区块链的存储限制，目前大多数项目的 NFT 存储仍然是链下存储。因此，ERC721 标准包含一个名为 tokenURI 的方法，人们可以实现这个方法来告诉应用程序在哪里可以找到给定项的元数据。

tokenURI 方法返回一个公有的 URL，通过 URL 返回一个 JSON 数据字典，类似于上面 CryptoKitty 的示例字典。这些元数据应该符合官方的 ERC721 元数据标准，以便 OpenSea 之类的应用程序能够获取。

链下存储最简单的方法是在某个集中式服务器上，或者在像 AWS 这样的云存储解决方案上。当然，这也有缺点:

1. 开发人员可以随意更改元数据;
2. 如果服务挂掉，NFT 的元数据可能会从原始源中消失。

为了解决这个问题，越来越多的开发人员，尤其是数字艺术领域的开发人员，正在使用 IPFS 来实现 NFT 的链下存储。IPFS 是一个 p2p 的文件存储系统，允许内容跨计算机托管，这样文件就可以在许多不同的位置复制。确切地说，是用另一个链来存储 NFT 的元数据。这样可以确保：

1. 元数据是不可变的，因为它是由文件的散列唯一解决的;
2. 只要有节点愿意承载数据，数据就会随着时间的推移持久化。

### 4.2 使用 IPFS 存储 NFT 元数据

本节将使用 Pinata，一个方便的 IPFS 应用程序接口和工具包，用于存储我们的非同质化代币资产和元数据，以确保我们的非同质化代币真正去中心化。 如果没有 Pinata 帐户，请在[此处](https://app.pinata.cloud/) 注册一个免费帐户，并完成电子邮件验证步骤。

创建帐户后：

导航到“文件”页面，点击页面左上方蓝色的“上传”按钮。

将图像上传到 Pinata — 这将是你的非同质化代币的图像资产。 您可以随意为该资产命名。

上传之后，你将在“File”页面的表格中看到文件信息。 您还会看到 CID 列。 您可以通过单击旁边的复制按钮来复制 CID。 您可以通过 https://gateway.pinata.cloud/ipfs/<CID> 查看您上传的信息。 例如，您可以在这里找到我们在星际文件系统上使用的图片。

创建帐户后：

- 导航到“文件”页面，点击页面左上方蓝色的“上传”按钮。

- 将图像上传到 Pinata — 这将是你的非同质化代币的图像资产。 您可以随意为该资产命名。

- 上传之后，你将在“File”页面的表格中看到文件信息。 您还会看到 CID 列。 您可以通过单击旁边的复制按钮来复制 CID。 您可以通过 https://gateway.pinata.cloud/ipfs/<CID> 查看您上传的信息。 例如，您可以在[这里](https://gateway.pinata.cloud/ipfs/QmZAxkZQfr9SZJFoAgPitFqs7ME9L4WjDyNyd34igK8i5d)找到我们在星际文件系统上使用的图片。

上述步骤总结如下：
![](https://gateway.pinata.cloud/ipfs/Qmcdt5VezYzAJDBc4qN5JbANy5paFg9iKDjq8YksRvZhtL)

在项目根目录中，创建一个名为`nft-metadata.json`的新文件，并添加以下 json 代码：

```json
{
  "attributes": [
    {
      "key": "fileType",
      "value": "png"
    },
    {
      "key": "name",
      "value": "gift"
    },
    {
      "key": "content",
      "value": "Chinese calligraphy and painting"
    }
  ],
  "description": "The world's most precious gift.",
  "image": "ipfs://QmZAxkZQfr9SZJFoAgPitFqs7ME9L4WjDyNyd34igK8i5d",
  "name": "Lily's gift for Father's Day"
}
```

同样上传`nft-metadata.json`到 Pinata，步骤和图片相同。

### 4.3 铸造 NFT 脚本

在 scripts 目录下创建`mint-nft.js`文件，输入如下内容：

```js
require("dotenv").config();
const { API_URL, PRIVATE_KEY, PUBLIC_KEY } = process.env;

const { createAlchemyWeb3 } = require("@alch/alchemy-web3");
const web3 = createAlchemyWeb3(API_URL);

const contract = require("../build/contracts/FatherGift.json");
const contractAddress = "0x5A03F8EF5CE08936cd23cF0c34E4CaA4c4f2c259";
const nftContract = new web3.eth.Contract(contract.abi, contractAddress);

async function mintNFT(tokenURL) {
  const nonce = await web3.eth.getTransactionCount(PUBLIC_KEY, "latest");

  const tx = {
    from: PUBLIC_KEY,
    to: contractAddress,
    nonce: nonce,
    gas: 500000,
    data: nftContract.methods.mintNFT(PUBLIC_KEY, tokenURL).encodeABI(),
  };

  const signPromise = web3.eth.accounts.signTransaction(tx, PRIVATE_KEY);
  signPromise
    .then((signedTx) => {
      web3.eth.sendSignedTransaction(signedTx.rawTransaction, function (err, hash) {
        if (!err) {
          console.log("The hash of you transaction is ", hash);
        } else {
          console.log("Something went wrong when submitting your transaction: ", err);
        }
      });
    })
    .catch((err) => {
      console.log("Promise failed:", err);
    });
}

const nftMetaURL = "https://gateway.pinata.cloud/ipfs/QmZWE2nvf1aTycsDUR3hVqJJzyYZMiCLbgSgGtJYochdRT";
mintNFT(nftMetaURL);
```

- 将其中的 contractAddress 合约地址换成 3.7 实际部署的合约地址
- 将 nftMetaURL 元数据 url 换成 Pinaca 上`nft-metadata.json`文件的地址

执行`node scripts/mint-nft.js`命令铸造 NFT。几秒后，会在终端看到如下的响应：

```text
The hash of you transaction is  0x049d1e32220ad304d17cb01fcd5871bb36ec2eea578f723dbfabdffd82223c65
```

![](https://s1.ax1x.com/2022/12/06/z6ToNT.png)

将交易 Hash 到[etherscan 网站](https://goerli.etherscan.io/tx/0x049d1e32220ad304d17cb01fcd5871bb36ec2eea578f723dbfabdffd82223c65)搜索，可以看到交易及交易铸造的 NFT。

![https://s1.ax1x.com/2022/12/06/z67Vbt.png](https://s1.ax1x.com/2022/12/06/z67Vbt.png)

![https://s1.ax1x.com/2022/12/06/z67KPS.png](https://s1.ax1x.com/2022/12/06/z67KPS.png)

## 5. NFT 上架到 OpenSea

### 5.1 钱包查看 NFT 资产

目前 Goerli 测试网络 NFT 需要手工导入。

- 点击浏览器 MetaMask 控件下方的添加资产按钮。
  ![https://s1.ax1x.com/2022/12/06/z67oqI.png](https://s1.ax1x.com/2022/12/06/z67oqI.png)
- 填入合约地址，代币小数填 0 即可。
  ![https://s1.ax1x.com/2022/12/06/z67OJS.png](https://s1.ax1x.com/2022/12/06/z67OJS.png)

添加完后 MetaMask 会自动同步 NFT 信息，获取网络中铸造的 NFT 数目。

![https://s1.ax1x.com/2022/12/06/z6HkJU.png](https://s1.ax1x.com/2022/12/06/z6HkJU.png)

### 5.2 OpenSea 售卖 NTF

OpenSea 支持以太坊测试网络，地址是：[https://testnets.opensea.io/](https://testnets.opensea.io/)

- 如果没有 OpenSea 账号，需要注册一个，并绑定 MetaMask 账号。
- 绑定完 MetaMask 账户后，点击设置按钮，
  ![https://s1.ax1x.com/2022/12/06/z6bQns.png](https://s1.ax1x.com/2022/12/06/z6bQns.png)
- 弹出 MetaMask 交易签名页面
  ![https://s1.ax1x.com/2022/12/06/z6bUc4.png](https://s1.ax1x.com/2022/12/06/z6bUc4.png)
- 点击签名按钮

此时可以在 OpenSea 中看到你的 NFT 了，你可以为 NFT 设置价格上架。

[这里 https://testnets.opensea.io/zh-CN/collection/fastherdaygift](https://testnets.opensea.io/zh-CN/collection/fastherdaygift)是 Demo 创建的 NFT。

![https://s1.ax1x.com/2022/12/06/z6qmUx.png](https://s1.ax1x.com/2022/12/06/z6qmUx.png)

# Web3 开发入门笔记 1 — 智能合约

## 1. 背景

智能合约是运行在以太坊链上的程序。它是位于以太坊区块链上一个特定地址的一系列代码（函数）和数据（状态）。智能合约也是一个以太坊帐户，一般称之为合约帐户。智能合约无法被人操控，个人用户可以通过提交交易执行智能合约的某一个函数来与智能合约进行交互。 智能合约能像常规合约一样定义规则，并通过代码自动强制执行。 默认情况下，无法删除智能合约，与它们的交互是不可逆的。

智能合约的开发语言有多种，最受欢迎的是 Solidity。本文基于 Solidity 介绍智能合约的开发。Solidity 文档见[https://docs.soliditylang.org/en/v0.8.17/](https://docs.soliditylang.org/en/v0.8.17/)。

## 2.环境安装

### 2.1 Node.js

Node 的版本不低于 V14，Node 的安装包可以从[https://nodejs.org/zh-cn/download](https://nodejs.org/zh-cn/download) 地址下载。

安装完执行 `node -v` 命令查询版本号:

![https://s1.ax1x.com/2022/12/03/zDxHsJ.png](https://s1.ax1x.com/2022/12/03/zDxHsJ.png)

### 2.2 智能合约开发框架 Truffle

Truffle 包含开发环境、测试框架和部署通道等，文档地址：[https://trufflesuite.com/docs/truffle/](https://trufflesuite.com/docs/truffle/)

Truffle 命令：`npm install -g truffle`

安装完成后，通过`truffle version`命令查看信息。

![https://s1.ax1x.com/2022/12/03/zDz9Qe.png](https://s1.ax1x.com/2022/12/03/zDz9Qe.png)

### 2.3 IDE VS Code

VS Code 下载地址：[https://code.visualstudio.com/](https://code.visualstudio.com/)

安装完 VS Code 后，再安装 [truffle 插件](https://marketplace.visualstudio.com/items?itemName=trufflesuite-csi.truffle-vscode)

## 3. 代码开发

任何编程语言的入门示例都是 HelloWorld，这里也不例外。

### 3.1 工程创建

在电脑上创建工程目录 helloworld，然后执行 `truffle init` 初始化项目工程目录。

```sh
cd helloworld
truffle init
```

初始化完成后项目的目录如下：

![https://s1.ax1x.com/2022/12/03/zDzR6e.png](https://s1.ax1x.com/2022/12/03/zDzR6e.png)

- contracts 是合约源代码目录
- migrations 是合约的部署文件目录
- test 是合约的测试代码目录

### 3.2 创建合约

在 contracts 目录创建 HelloWorld.sol 文件，代码如下：

```solidity
// SPDX-License-Identifier: MIT
// Tells the Solidity compiler to compile only from v0.8.13 to v0.9.0
pragma solidity ^0.8.13;

import "@openzeppelin/contracts/utils/Strings.sol";

contract HelloWorld {
    string private message;

    constructor() {
        message = "Welcome to Web3 world ";
    }

    function sayHello() public view returns (string memory) {
        return
            string(
                abi.encodePacked(
                    message,
                    Strings.toHexString(uint160(msg.sender), 20)
                )
            );
    }
}
```

这个合约实现的功能很简单，就是对调用其 sayHello 方法的账号说「Welcome to Web3 world 账户地址」

### 3.3 引入第三方包

在代码中我们引入了 openzeppelin 包下面的文件，看名字似乎是一个工具类。

OpenZeppelin 代码库包含了经过社区审查的 ERC 代币标准、安全协议以及很多的辅助工具库，这些代码可以帮助开发者专注业务逻辑的，而无需重新发明轮子。基于 OpenZeppelin 开发合约，既可以提高代码的安全性，又可以提高开发效率。

安装 OpenZeppelin 包：

```shell
npm install @openzeppelin/contracts
```

### 3.4 创建合约部署文件

在 migrations 目录新建 1_deploy_contracts.js 文件，文件中输入如下内容：

```js
const HelloWorld = artifacts.require("HelloWorld");

module.exports = function (deployer) {
  deployer.deploy(HelloWorld);
};
```

### 3.5 创建合约单元测试代码

在 test 目录新建 helloWorld.js 文件，文件中输入如下内容：

```js
const HelloWorld = artifacts.require("HelloWorld");

contract("HelloWorld", (accounts) => {
  it("say hello", async () => {
    const helloWorldInstance = await HelloWorld.deployed();
    const message = await helloWorldInstance.sayHello.call();

    assert.equal(message, "Welcome to Web3 world " + accounts[0].toLowerCase(), "Message is invalid");
  });
});
```

### 3.6 执行单元测试

在项目目录下执行 `truffle test`命令即执行单元测试，

![https://s1.ax1x.com/2022/12/03/zrPGxf.png](https://s1.ax1x.com/2022/12/03/zrPGxf.png)

至此 HelloWorld 合约的开发工作就完成了。

## 4.部署

### 4.1 启动开发网络

如果要测试的话，我们可以把合约部署到开发网络或者公共以太坊测试链（需要测试以太币）。一般我们会选用开发网络，开发网络有如下优点：

- 为本地区块链提供数据，例如任意数量任意以太币余额的帐户
- 在接受到每个交易时就立即按顺序和没有延迟地挖掘区块
- 调试和日志功能

一般开发框架都包含内置的开发网络，Truffle 框架中包含的是 Ganache，当然你也可以使用 Hardhat 网络。

通过`ganache`命令在本地启动一个以太坊开发网络。
![https://s1.ax1x.com/2022/12/03/zrEIK0.png](https://s1.ax1x.com/2022/12/03/zrEIK0.png)

可以看到开发网络默认创建了 10 个账户，每个账户有 1000 以太币，网络的监听端口为 8545。

### 4.2 部署合约

修改 truffle-config.js 文件 networks 网络配置项，放开如下配置：

```js
networks: {
    development: {
      host: "127.0.0.1", // Localhost (default: none)
      port: 8545, // Standard Ethereum port (default: none)
      network_id: "*", // Any network (default: none)
    },
}
```

执行`truffle migrate`命令部署合约
![https://s1.ax1x.com/2022/12/03/zrVLef.png](https://s1.ax1x.com/2022/12/03/zrVLef.png)

可以看到 HelloWorld 合约已经成功部署到链上，合约账户地址为 0xb536fB46D267175B96fF91696FcC439a369f38b2，部署合约花费了 457403 gas，一共值 0.001543735125 个以太币。

### 4.3 调用合约

为了让软件应用程序与以太坊区块链交互（通过读取区块链数据或向网络发送交易），以太坊定义了一套 JSON-RPC 规范，每个以太坊的客户端都需要实现。正如我们网络编程不会直接使用 Socket，而会使用 Netty、Jetty 这样的框架。对 DApp 开发者来说，也不需要直接使用 JSON-RPC 接口，可以选用封装 API。一般来说 JS 语言可以选用 web3.js 库，Java 可以使用 web3j 库。

我们使用 web3.js 库和智能合约进行交互。

```shell
truffle console
truffle(development)> let helloWorldInstance = await HelloWorld.deployed();
undefined
truffle(development)> let message = await helloWorldInstance.sayHello.call({from: '0xC81BF147E3f8d3b97946378aAeEffd25903fDb97'});
undefined
truffle(development)> message
'Welcome to Web3 world 0xc81bf147e3f8d3b97946378aaeeffd25903fdb97'
truffle(development)>
```

1. 通过`truffle console`命令启动一个 truffle web3.js 客户端。
2. 从链上加载 HelloWorld 合约，let helloWorldInstance = await HelloWorld.deployed()
3. 通过 0xC81BF147E3f8d3b97946378aAeEffd25903fDb97 账户调用合约的 sayHello 方法
4. 查看合约 sayHello 方法的输出结果如预期

![https://s1.ax1x.com/2022/12/03/zrK4Ff.png](https://s1.ax1x.com/2022/12/03/zrK4Ff.png)

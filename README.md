###以太坊开发教程
- 1.以太坊概述
- 2.环境配置以及私有链搭建
- 3.钱包账户的基本操作
- 4.智能合约的部署
- 5.Dapp应用实例

### part1 以太坊概述
基本概念：以太坊（Ethereum）是一个建立在区块链技术之上， 去中心化应用平台。它允许任何人在平台中建立和使用通过区块链技术运行去中心化应用。
##### 以太坊 = 比特币 + 无限可能
### part2 环境配置以及私有链搭建
以太坊开发工具
- 1.Ubuntu 16.04
- 2.以太坊客户端
：以太坊客户端用于接入以太坊网络，进行账户管理、交易、挖矿、智能合约相关的操作。
- 3.智能合约编译器
- 4.以太坊钱包（可用可不用，本篇未使用）
：以太坊提供了图形界面的钱包 Ethereum Wallet 和 Mist Dapp 浏览器。

环境搭建
1. 安装以太坊客户端
  - 1.1 安装必要的工具包：
```
sudo apt install software-properties-common
```
  - 1.2添加以太坊源：
```
  sudo add-apt-repository -y ppa:ethereum/ethereum
  sudo apt update
```
  - 1.3安装 go-ethereum：
 ```
  sudo apt install ethereum
 ```
  - 1.4安装solidity编译器
 ```
    sudo add-apt-repository ppa:ethereum/ethereum
    sudo apt update
    sudo apt install solc
 ```
  - 1.5开发环境选择：私有链/测试链<br/>
  在以太坊的公有链上部署智能合约、发起交易需要花费以太币。而通过修改配置，可以在本机搭建一套以太坊私有链，因为与公有链没关系，既不用同步公有链庞大的数据，也不用花钱购买以太币，很好地满足了智能合约开发和测试的要求，开发好的智能合约也可以很容易地切换接口部署到以太坊公有链上。<br/>
2. 私有链的搭建
  - 2.1 配置初始状态<br/>
要运行以太坊私有链，需要定义自己的创世区块，创世区块信息写在一个 JSON 格式的配置文件中。配置文件如下：<br/>
 ```
{ "config":{
 "chainID": 15,
 "homesteadBlock": 0,
 "eip155Block": 0,
 "eip158Block": 0
 },
 "alloc": {},
 "coinbase": "0x0000000000000000000000000000000000000000",
  "difficulty": "0x400",
 "extraData": "",
 "gasLimit": "0x2fefd8",
  "nonce": "0xdeadbeefdeadbeef",
  "mixhash":"0x0000000000000000000000000000000000000000000000000000000000000000",
 "parentHash":"0x0000000000000000000000000000000000000000000000000000000000000000",
  "timestamp": "0x00"
}
 ```
  - 2.2初始化：写入创世区块<br/>
首先要新建一个目录用来存放区块链数据，
假设新建的数据目录为 ~/privatechain/data0，genesis.json 保存在privatechain中
此时目录结构应该是这样的：<br/>
privatechain<br/>
├── data0<br/>
└── genesis.json<br/>

```
 cd privatechain<br/>
 sudo geth --datadir data0 init genesis.json<br/>
```
其中 --datadir 指定数据存储在 data0中，init指定创世块文件的位置，并创建初始块。
 - 2.3启动私有链节点
 ```
 sudo geth --networkid 15 --rpc --rpcport "8545" --datadir data0 --port "30303" --nodiscover console
 ```
这是一个交互式的Javascript执行环境，在这里面可以执行Javascript代码，其中>是命令提示符。在这个环境里也内置了一些用来操作以太坊的Javascript对象，可以直接使用这些对象。
### part3 钱包账户的基本操作
在私有网络下进行账户钱包操作
  - 1. 创建账户
```
> personal.newAccount()  账户0
> personal.newAccount()  账户1
```
  - 2. 查看账户余额

```
eth.getBalance(eth.accounts[0])账户0的余额（初始为0)
eth.getBalance(eth.accounts[1])账户1的余额
```
 - 3. 启动&停止挖矿
 启动挖矿：
```
miner.start(1)
```
其中 start 的参数表示挖矿使用的线程数。第一次启动挖矿会先生成挖矿所需的 DAG 文件，这个过程有点慢，等进度达到 100% 后，就会开始挖矿，此时屏幕会被挖矿信息刷屏。<br/>
停止挖矿，在 console 中输入
 ```
 miner.stop()
 ```
 - 挖到一个区块会奖励5个以太币，挖矿所得的奖励会进入矿工的账户。
- 4. 发送交易
(1)查看账户余额:

```
eth.getBalance(eth.accounts[0])
```
(2)我们要从账户 0 向账户 1 转账，所以要先解锁账户 0，才能发起交易

```
personal.unlockAccount(eth.accounts[0])
```
3)发送交易，账户 0 -> 账户 1:

> amount = web3.toWei(10,'ether')<br/>
"1000000000000000000"
```
eth.sendTransaction({from:eth.accounts[0],to:eth.accounts[1],value:amount})

```
4)使用 miner.start() 命令开始挖矿:<br/>

```
 miner.start(2);admin.sleepBlocks(1);miner.stop();
```
5)新区块挖出后，挖矿结束，查看账户 1 的余额，已经收到了账户 0 的以太币：

```
web3.fromWei(eth.getBalance(eth.accounts[1]),'ether’)
```
10(结果)<br/>
5. 查看交易和区块
>eth.blockNumber
通过交易 Hash 查看交易（Hash 值包含在上面交易返回值中）：
 ```
eth.getTransaction("交易Hash 值")
 ```
通过区块号查看区块：
>eth.getBlock(0)
6. 连接到其他节点
为了在本地网络运行多个以太坊节点的实例,必须确保以下几点:
- 每个实例都有独立的数据目录(--datadir)
- 每个实例运行都有独立的端口.(eth和rpc两者都是)(--port 和 --rpcprot)
- 在集群的情况下, 实例之间都必须要知道彼此.
- 唯一的ipc通信端点,或者禁用ipc.<br/>
下面是输入指令：<br/>
节点2
```
>geth --datadir data5 init genesis.json
>geth --datadir data5 --networkid 15 --ipcdisable --port 6191 --rpcaddr ip_address --rpcport 8105 console
```
- 首先要知道节点1的 enode 信息，在节点1的 JavaScript console 中执行下面的命令查看 enode 信息：<br/>
>admin.nodeInfo.enode<br/>
- 然后在节点一的 JavaScript console 中执行 admin.addPeer()，就可以连接到节点二：<br/>
>admin.addPeer("节点二的enode")<br/>
- addPeer() 的参数就是节点二的 enode 信息，注意要把 enode 中的 [::] 替换成节点二的 IP 地址。
- 通过 admin.peers 可以查看连接到的其他节点信息，通过 net.peerCount 可以查看已连接到的节点数量。

### part4 智能合约部署流程
一. 创建和编译智能合约<br/>
新建一个 Solidity 智能合约文件，命名为 HelloWorld.sol，该合约包含一个方法 sayHello ( )，将输出Hello World.

```solidity
pragma solidity ^0.4.4;
contract HelloWorld
{
    function sayHello() returns (string)
    {
        return ("Hello World");
    }
}
```
- 编译智能合约，获得编译后的 EVM 二进制码：
```
$ solc --bin HelloWorld.sol
```
- 再用 solc 获取智能合约的 JSON ABI（Application Binary Interface），其中指定了合约接口，包括可调用的合约方法、变量、事件等：
```
$ solc --abi HelloWorld.sol
```
回到 Geth 的控制台，用变量 code 和 abi 记录上面两个值，注意在 code 前加上 0x 前缀.
二、部署智能合约

这里使用账户 0 来部署合约，首先解锁账户：
```
personal.unlockAccount(eth.accounts[0])
Unlock account 0x3443ffb2a5ce3f4b80080791e0fde16a3fac2802
Passphrase:
true
```
发送部署合约的交易：
创建合约：通过eth.contract()创建一个合约对象，这个对象包含合约接口的abi
```
myContract = eth.contract(abi)
```
- 发送交易部署合约：data为上述编译得到的code
```
contract = myContract.new({from:eth.accounts[0],data:code,gas:1000000})
```
- 此时如果没有挖矿，用 txpool.status 命令可以看到本地交易池中有一个待确认的交易。
//我们知道部署合约的过程实际也是由创建合约的账户发送的一笔交易
需要挖矿进行确认。
```
miner.start(2);admin.sleepBlocks(1);miner.stop();
```
三、调用智能合约
使用以下命令发送交易，sendTransaction 方法的前几个参数应该与合约中sayHello方法的输入参数对应。这种情况下，交易会通过挖矿记录到区块链中：
```
contract.sayHello.sendTransaction({from:eth.accounts[0]})
 ```
如果只是本地运行该方法查看返回结果，可以采用如下方式：
```
contract.sayHello.call()
```
### part5 Dapp应用开发
Ethereum Web开发 ——Dapp（投票）<br/>
框架选择：truffle + react<br/>
环境选择：测试链<br/>
使用solidity的truffle框架开发智能合约，前端使用react框架，最终完成智能合约从前端到后端，从开发到部署的完整流程。
```solidity
pragma solidity ^0.4.4;
contract Voting{
     bytes32[] candidates; //定义一个候选人数组
     mapping(bytes32 => uint) candidatesVotingCount;//记录候选人得票数的字典
     //初始化操作
     function Voting(bytes32[] _candidates)public{
             candidates = _candidates;
     }
     //投票
     function votingToPerson(bytes32 person)public {
             require(isValidToPerson(person));
             candidatesVotingCount[person]+=1;
    }
    //统计对应候选人的票数
    function votingTotalToPerson(bytes32 person) constant public returns(uint){
            assert(isValidToPerson(person));
            return candidatesVotingCount[person];
     }
    //判段输入的是否为候选人
    function isValidToPerson(bytes32 person) constant public returns(bool){
            for(uint i=0;i<candidates.length;i++){
	  if(candidates[i] == person){
	          	return true;
	}
             }
            return false;
     }
}
```
- 开发流程<br/>
需要的工具
:<br>
(1)Truffle (2). metamask浏览器插件 (3).文本编辑器（Atom）<br/>
1.Truffle 安装
```
npm install –g truffle
```
Truffle是现在比较流行的Solidity智能合约开发框架，功能十分强大，可以帮助开发者迅速搭建起一个Dapp。<br/>
2.创建项目
```
truffle unbox React
```
3. 合约编写
4. 合约编译
打开所在项目的文件目录下，打开终端，运行truffle控制台
```
truffle develop
```
2)编译
```
compile
```
编译过程非常简单，在项目根目录下执行truffle compile即可，会在根目录生成build编译目录。这里的编译与部署合约无关，只是为了后续前端代码的调用。<br/>
5. 部署合约<br/>
1)打开http://remix.ethereum.org/<br/>
2)将上述代码直接粘贴到左侧编辑器中，会自动编译查错，正常编译通过后，可以在右侧面板点击run选项。<br/>
3)初始化合约，点击deploy(create)，通过remix + metamask部署合约到测试网络，并得到合约地址。<br/>
6.修改app.js前端代码和合约进行互动<br/>
7.终端运行 npm start，查看网页效果<br/>

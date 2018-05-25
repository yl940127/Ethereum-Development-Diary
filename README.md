# Ethereum-Development-Diary
基于以太坊的区块链技术开发
###以太坊开发教程
- 1.以太坊概述
- 2.环境配置以及私有链搭建
- 3.钱包账户的基本操作
- 4.智能合约的部署
- 5.Dapp应用实例

# part1 以太坊概述
基本概念：以太坊（Ethereum）是一个建立在区块链技术之上， 去中心化应用平台。它允许任何人在平台中建立和使用通过区块链技术运行去中心化应用。
## 以太坊 = 比特币 + 无限可能
**以太坊架构**
![avatar](images/frame.png)
###part2 环境配置以及私有链搭建
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
    >sudo apt install software-properties-common
  - 1.2添加以太坊源：
    >sudo add-apt-repository -y     ppa:ethereum/ethereum
    >sudo apt update
  - 1.3安装 go-ethereum：
    >sudo apt install ethereum
  - 1.4安装solidity编译器
    >sudo add-apt-repository ppa:ethereum/ethereum
    >sudo apt update
    >sudo apt install solc
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
privatechain
├── data0
└── genesis.json
  >cd privatechain
  >sudo geth --datadir data0 init genesis.json
其中 --datadir 指定数据存储在 data0中，init指定创世块文件的位置，并创建初始块。
 - 2.3启动私有链节点
 >sudo geth --networkid 15 --rpc --rpcport "8545" --datadir data0 --port "30303" --nodiscover console
 各选项含义如下:
![avatar](images/2.png)
运行上面的命令的得到的效果图：
![avatar](images/3.png)
这是一个交互式的Javascript执行环境，在这里面可以执行Javascript代码，其中>是命令提示符。在这个环境里也内置了一些用来操作以太坊的Javascript对象，可以直接使用这些对象。
###part3 钱包账户的基本操作
在私有网络下进行账户钱包操作
  1. 创建账户
> personal.newAccount()  账户0
> personal.newAccount()  账户1
  2. 查看账户余额
> eth.getBalance(eth.accounts[0])账户0的余额（初始为0)
> eth.getBalance(eth.accounts[1])账户1的余额
 3. 启动&停止挖矿
  - 启动挖矿：
> miner.start(1)<br/>
- 其中 start 的参数表示挖矿使用的线程数。第一次启动挖矿会先生成挖矿所需的 DAG 文件，这个过程有点慢，等进度达到 100% 后，就会开始挖矿，此时屏幕会被挖矿信息刷屏。<br/>
停止挖矿，在 console 中输入<br/>
> miner.stop()<br/>
 - 挖到一个区块会奖励5个以太币，挖矿所得的奖励会进入矿工的账户。
4. 发送交易<br/>
1)查看账户余额:<bt/>
>eth.getBalance(eth.accounts[0])<br/>
2)我们要从账户 0 向账户 1 转账，所以要先解锁账户 0，才能发起交易：<br/>
> personal.unlockAccount(eth.accounts[0])<br/>
3)发送交易，账户 0 -> 账户 1:<br/>
> amount = web3.toWei(10,'ether')<br/>
"1000000000000000000"<br/>
>eth.sendTransaction({from:eth.accounts[0],to:eth.accounts[1],value:amount})<br/>
4)使用 miner.start() 命令开始挖矿:<br/>
> miner.start(2);admin.sleepBlocks(1);miner.stop();<br/>
5)新区块挖出后，挖矿结束，查看账户 1 的余额，已经收到了账户 0 的以太币：<br/>
>web3.fromWei(eth.getBalance(eth.accounts[1]),'ether’)<br/>
>10(结果)<br/>
5. 查看交易和区块
> eth.blockNumber<br/>
通过交易 Hash 查看交易（Hash 值包含在上面交易返回值中）：<br/>
>eth.getTransaction("交易Hash 值")<br/>
通过区块号查看区块：<br/>
> eth.getBlock(0)<br/>
![avatar](images/4.png)
6. 连接到其他节点
为了在本地网络运行多个以太坊节点的实例,必须确保以下几点:
- 每个实例都有独立的数据目录(--datadir)
- 每个实例运行都有独立的端口.(eth和rpc两者都是)(--port 和 --rpcprot)
- 在集群的情况下, 实例之间都必须要知道彼此.
- 唯一的ipc通信端点,或者禁用ipc.<br/>
下面是输入指令：<br/>
节点2
>geth --datadir data5 init genesis.json<br/>
>geth --datadir data5 --networkid 15 --ipcdisable --port 6191 --rpcaddr ip_address --rpcport 8105 console<br/>
- 首先要知道节点1的 enode 信息，在节点1的 JavaScript console 中执行下面的命令查看 enode 信息：<br/>
> admin.nodeInfo.enode<br/>
![avatar](images/5.png)
- 然后在节点一的 JavaScript console 中执行 admin.addPeer()，就可以连接到节点二：<br/>
>admin.addPeer("节点二的enode")<br/>
- addPeer() 的参数就是节点二的 enode 信息，注意要把 enode 中的 [::] 替换成节点二的 IP 地址。
- 通过 admin.peers 可以查看连接到的其他节点信息，通过 net.peerCount 可以查看已连接到的节点数量。
![avatar](images/6.png)

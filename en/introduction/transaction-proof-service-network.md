# Transaction Proof Service Network

在通常的BTC应用场景里，比如BTC Wallet，要验证一笔交易是否成功，为了安全起见，用户不会轻易使用第三方的RPC服务返回的信息，而是通过自己搭建的，或者由可信第三方搭建的BTC全节点返回信息来确认交易。当然也可以使用SPV节点在本地验证交易。但无论是全节点还是SPV节点，都需要耗费漫长的同步过程，带宽和本地存储，这对应用场景里的设备和网络环境都提出了不低的要求。

所谓全节点，就是指可以在本地，通过同步所有数据，逐条验证以后，得到全网共识后的最新状态集，再基于这个状态集，我们就可以验证一笔新交易的真实性和有效性。

零知识证明技术的特点是，它的电路程序可以像智能合约一样执行规定的计算过程，产生一个结果，并且它可以证明这是完全按照电路程序的指令产生的结果。这个结果可以使用非常简单的数学计算进行验证。如果说Sha256是内容的“指纹”，零知识证明就是计算过程的“指纹”。

结合零知识证明的这个特点，BeL2采用Cairo语言，按照Bitcoin的共识代码编写验证电路，如果我们可以在一台服务器上对BTC交易完成验证，并生成证明，第三方可以审计BeL2的电路代码是否与Bitcoin的共识一致，如果它们是一致的，逻辑等价的，那么就可以信任BeL2电路产生的证明。第三方拿到这个证明，可以在本地对其进行验证，从而确定这个交易的有效性。

这就好像是BeL2用零知识证明技术为Bitcoin共识生成了一个快照，这个快照可以被验证，任何人无需额外信任即可相信这个快照所代表的BTC共识结果。

BeL2基于零知识证明技术提供的交易证明服务包括以下几个部分

1.  ZKP交易证明服务合约

    面向第三方提供的验证服务，用户向合约提交请求，服务商看到请求以后可以竞争执行生成证明。服务商提交的证明可以被合约验证，第一个提交有效证明的服务商会得到奖励。
2.  若干Cairo电路

    为了面向多种场景提供服务，BeL2将BTC交易的验证拆解为多个子电路模块，分别用于验证交易有效性、输入输出UTXO、签名、脚本等等多的方面。可以根据场景的不同，利用递归证明的方式组合这些模块，生成一个压缩的证明用于在合约中验证。
3.  ZKP服务程序

    证明服务是一个服务商网络，服务商可以运行BeL2的ZKP服务程序，监听ZKP服务合约中的请求，为用户计算和生成证明，从而获得奖励。这是一个无需准入许可的网络，任何人都可以运行这个服务程序加入进来。同时，只要遵守协议规范，服务商也可以改写和优化服务程序，通过提供更高效的证明生成服务获得奖励。

第三方开发者或者用户可以通过向BeL2验证服务合约提交BTC交易的方式获得其验证结果，第三方dApp可以基于BeL2的ZKP验证服务合约实现主网BTC的交易协议。

让我们以一个主网BTC的SWAP为例来介绍BeL2的ZKP验证服务。

假设Alice想用0.1 BTC兑换Bob的6500 USDC。

1. Bob先创建一个订单合约，并设置了预期的BTC数量，再存入自己的6500 USDC
2. Alice看到订单以后，决定接受订单，于是先Take订单，避免其他人也在Take相同的订单
3. Alice向Bob的地址发送BTC
4. Alice向BeL2的交易证明服务提交她的BTC交易，并生成证明
5. Alice向订单合约提交证明，被验证通过以后Alice得到6500 USDC

通过这个例子我们可以看到，基于BeL2的交易证明，Alice可以自助换取EVM链上的USDC。开发者可以在更多的场景里使用它。

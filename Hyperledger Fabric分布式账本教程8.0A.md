# *Hyperledger Fabric* 分布式账本教程8.0A



> # *Ubuntu*一键更换国内源
>
> 直接复制下面的命令到窗口中执行：
>
> ```bash
>sudo sed -i 's/archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
> ```
>
> 如果是cn开头的
>
> ```bash
>sudo sed -i 's/cn.archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
> ```
>
> 如果想替换security源：
> 
> ```text
>sudo sed -i 's/security.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
> ```
>
> 其它更多国内源参考，将上面的[http://mirrors.ustc.edu.cn](https://link.zhihu.com/?target=http%3A//mirrors.ustc.edu.cn)换成下面任一种即可，不同地区下载速度有一定差别。
> 



## 1.1 安装 *Golang* 环境

<p style = "margin:20px">
    <font face="微软雅黑"  size=3 color=#0000FF > Go 语言支持以下系统：</font>
</p>

- Linux
- FreeBSD
- Mac OS X（也称为 Darwin）
- Windows

安装包下载地址为：https://golang.org/dl/。

如果打不开可以使用这个地址：https://golang.google.cn/dl/。

![image-20221213215920692](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221213215920692.png)

```shell
$ cd ~
$ mkdir -p ~/go/src ~/go/pkg ~/go/bin
$ wget https://golang.google.cn/dl/go1.20.3.linux-amd64.tar.gz
$ sudo tar -zxvf go1.20.3.linux-amd64.tar.gz -C /usr/local/
$ ls /usr/local
```

![image-20221213215658724](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221213215658724.png)



### 1.1.1 *Go*全局环境配置

```shell
$ sudo vim /etc/profile
```

```shell
# GOROOT
export GOROOT=/usr/local/go
# GOROOT bin
export PATH=$PATH:$GOROOT/bin
# GOPATH
export GOPATH=$HOME/go
# GOPATH bin
export PATH=$PATH:$GOPATH/bin
```

```shell
//或如下：
export PATH=$PATH:/usr/local/go/bin
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin
```

![image-20221213220447550](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221213220447550.png)

更新环境变量

```shell
$ source /etc/profile
```

- 查看Go环境

```shell
$ go env
```

![image-20221213220817554](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221213220817554.png)



###  1.1.2 *GO111MODULE*设置

*Go version* >= 1.13，直接用*go env -w* 设置（注意大小写）

```bash
go env -w GOPROXY=https://goproxy.cn,direct
#go env -w GOPROXY=https://mirrors.aliyun.com/goproxy,direct
go env -w GO111MODULE=on
```

![image-20221213221244088](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221213221244088.png)

> 注：可以用*go env -u* 恢复初始设置；GOPROXY*值还可以是*https://goproxy.cn
>
> *go env -w GOPROXY=https://goproxy/cn,direct*
>
> *go env -w GOPROXY=https://gocenter.io,direct*



### 1.1.3 *go mod* 命令

*go module* 使用前置条件: *GO111MODULE* 设置为 *on*

- 在当前文件夹下初始化一个新的 *module*，创建 *go.mod* 文件： `go mod init name`
- 拉取缺少的模块，移除不用的模块 ： `go mod tidy`
- 将依赖复制到 vendor 下 : `go mod vendor`
- 下载依赖 : `go mod download`
- 检验依赖: `go mod verify`
- 显示模块依赖图: `go mod graph`
- 解释为什么需要依赖: `go mod why`
- 编辑 go.mod 文件: `go eidt`
- 查看命令列表: `go mod`
- 查看命令帮助文档: `go help mod <command>`



## 1.2 安装 Docker

**Fabric 目前采用 Docker 容器作为链码执行环境，因此即使在本地运行，Peer 节点上也需要安装 Docker 环境，**推荐使用 1.18 或者最新的版本。目前，部署Docker 的方式主要有2种方式，一种是利用apt存储库安装，这需要更新存储库，另外一种最简单可靠的方式是一键部署。两种安装方式没有优劣之分，初学者建议使用apt存储库去分步安装，从而更好地理解docker安装的步骤。熟练者可以使用一键部署方式。

- 服务器上如果已经安装 docker，部署无需安装
- 具体官方安装参考 [官方文档](https://docs.docker.com/install/linux/docker-ce/centos/)
- https://docs.docker.com/engine/install/ubuntu/



### 1.2.1 使用 apt 存储库安装

新主机首次安装 Docker 引擎之前，您需要设置 Docker 存储库。之后，您可以安装和更新存储库中的 Docker。

1.如果Ubuntu系统中有旧版本的Docker，需要卸载后重新安装。可以使用以下命令进行卸载：

```bash
$ sudo apt remove docker docker-engine docker.io containerd runc
```

2.更新软件包索引并安装软件包以允许使用基于 HTTPS 的存储库：

```bash
$ sudo apt update
$ sudo apt install apt-transport-https ca-certificates curl software-properties-common
```

3.添加 Docker 的 阿里 GPG 密钥：

```bash
$ sudo mkdir -p /etc/apt/keyrings
$ sudo install -m 0755 -d /etc/apt/keyrings
$ curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
$ sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

4.使用以下命令设置阿里存储库：

```bash
$ echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://mirrors.aliyun.com/docker-ce/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  
$ sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

5.安装 Docker CE引擎（社区版）

```bash
#确保从Docker repo 安装docker
$ sudo apt-cache policy docker-ce

#安装Docker(社区版)
$ sudo apt update
$ sudo apt install -y docker-ce

#检测是否安装好
$ sudo docker version
```



> 将Docker存储库添加到APT源
> ①. 官方源
>
> ```bash
> $ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
> ```
>
> ②. 阿里源
>
> ```bash
> $ sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
> ```
>
> 将Docker存储库的GPG密钥添加到系统：
> ①. 官方的GPG证书
>
> ```bash
> $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
> ```
>
> ②. 阿里的GPG证书
>
> ```bash
> $ curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
> ```
>
> 确保从Docker repo 安装docker
>
> ```bash
> $ sudo apt-cache policy docker-ce
> ```
>
> 安装Docker(社区版)
>
> ```bash
> $ sudo apt update
> $ sudo apt install -y docker-ce
> ```
>
> 
>
> ————————————————
> 版权声明：本文为CSDN博主「Beyonderwei」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
> 原文链接：https://blog.csdn.net/csdn_x_w/article/details/103847762



6.当前用户添加到Docker用户组

```shell
#创建docker用户组
$ sudo groupadd docker

#将当前用户添加到docker用户组
$ sudo usermod -aG docker $USER

$ sudo usermod -aG root $USER
$ sudo usermod -aG sudo $USER

#退出当前终端
$ exit
```

```shell
sudo systemctl daemon-reload
sudo systemctl enable docker.service
sudo systemctl restart docker.service
sudo systemctl status docker.service

#执行以下命令如果输出docker版本信息如：Docker version 19.03.08则说明安装成功
& docker version
```

![image-20221213223023057](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221213223023057.png)



### 1.2.2 使用一键安装Docker（阿里）

```bash
$ curl -fsSL https://get.docker.com | sudo bash -s docker --mirror Aliyun
```

![image-20230511193700521](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20230511193700521.png)



### 1.2.3 安装Docker-Compose

> [Install Compose standalone | Docker Documentation](https://docs.docker.com/compose/install/standalone/)

下载docker-compose的二进制文件：

```bash
#如果用户不是root,建议切换到root
$ su -

#由于网络原因，可能造成下载不稳定，多次下载才可能完成
$ curl -SL https://github.com/docker/compose/releases/download/v2.20.4/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose

#如果Github下载缓慢，建议通过https://ghproxy.com/进行了一次代理。
$ curl -SL https://ghproxy.com/https://github.com/docker/compose/releases/download/v2.20.3/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose

$ chmod u+x /usr/local/bin/docker-compose

#检测docker-compose是否安装成功：
$ exit
$ docker-compose version
```

如果以上步骤可以顺利完成的话，接下来就可以进入正题了.

![image-20230512195709039](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20230512195709039.png)



### 1.2.4 *Docker* 镜像加速

1.进入阿里云镜像加速页面；

https://cr.console.aliyun.com/#/accelerator

2.修改/etc/docker/daemon.json文件配置，没有则新建；

您可以通过修改daemon配置文件/etc/docker/daemon.json来使用加速器

```shell
$ sudo mkdir -p /etc/docker

$ sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": [
  "https://s5qx8bjv.mirror.aliyuncs.com",
  "https://docker.mirrors.ustc.edu.cn"
   ]
}
EOF

sudo systemctl daemon-reload
sudo systemctl enable docker.service
sudo systemctl restart docker.service
sudo systemctl status docker.service
```

###  1.2.5 *Docker*可视化—Portainer

```bash
#通过命令安装 Portainer
$ docker volume create portainer_data
$ docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer
```



## 1.3 安装*NVM*

由于无法下载原因`，导致GitHub的`raw.githubusercontent.com域名解析被污染了。

官方安装模式可能无法完成，这样通过如下方式安装：

```bash
$ git clone https://github.com/nvm-sh/nvm.git
$ cd nvm
$ ./install.sh
```

主要环境变量.bashrc的修改和启动

```bash
$ source .bashrc
```

检验安装

```bash
$ nvm -v
```

NVM其它常用命令

```bash
nvm ls
nvm ls-remote
nvm install --lts 
nvm uninstall --lts 
nvm use --lts 
nvm exec --lts 
nvm run --lts 
nvm version-remote --lts 
```



## 1.4 安装Nodejs

*nvm* 下载太慢问题解决 以及下载稳定版本方法

更换下载源默认是从 [http://nodejs.org/dist/](https://link.jianshu.com/?t=http://nodejs.org/dist/) 下载的, 国外服务器, 必然很慢，建议如下安装方式：

```bash
$ NVM_NODEJS_ORG_MIRROR=https://npm.taobao.org/mirrors/node
```

```、bash
$ nvm install --lts #安装最新稳定版 node
```

或者采用一键安装：

```bash
NVM_NODEJS_ORG_MIRROR=https://npm.taobao.org/mirrors/node nvm install --lts
```

应用lts版本

```shell
nvm ls
nvm ls-remote
nvm use --lts 
```



## 1.5 Nodejs 加速

外网在国内访问太慢了，所以一般大家都把镜像改成taobao镜像会加快安装速度。。。

1.查看原始配置 *npm config ls*

会发现里面的registry是npm原始的镜像：https://registry.npmjs.org/

2.*npm*临时使用淘宝镜像安装依赖包

```bash
npm i -g express --registry https://registry.npm.taobao.org
```

3.*npm*持久使用淘宝镜像安装依赖包

```shell
npm config set registry https://registry.npm.taobao.org
npm config get registry
```

注意，不推荐这样子，因为把*npm*的镜像完全设为了淘宝的镜像，万一我们有些依赖包只有*npm*原始镜像里面才有，而淘宝里面没有，那就悲剧了。所以分开*npm*和*cnpm*是最好的。

```shell
npm config ls
```



## 1.6 创建用户并设置密码和*sudo*权限

```shell
# 添加用户(可选)
sudo adduser xxxx
# 为新用户设置密码
sudo passwd xxxx
# 为新用户添加sudo权限
sudo echo 'xxxx ALL=(ALL) ALL' >> /etc/sudoers
```





# *Hyperledger Fabric* 分布式账本技术简介

## 2.1 简介

一般来说，区块链是不可篡改的事务账本，通过对等节点构成的分布式网络维护。这些节点通过把由共识协议验证后的事务应用到本地来维护一个分类账副本，这些事务被打包到一个个块中，每个块中都绑定了其前一个块的哈希值。

区块链的第一个也是最广泛认可的应用是[比特币](https://en.wikipedia.org/wiki/Bitcoin)加密货币，尽管其他 纷纷追随它的脚步。以太坊，一种替代加密货币，采取了一个 不同的方法，整合了许多与比特币相同的特征，但 添加*智能合约*，为分布式应用创建平台。 比特币和以太坊属于一类区块链，我们将它归类为*公共无许可*区块链技术。基本上，这些是公开的 网络，向任何人开放，参与者匿名互动。

随着比特币、以太坊和其他一些衍生技术的普及 增长，对应用区块链底层技术的兴趣， 分布式账本和分布式应用平台到更具创新性*的企业*用例也在增长。但是，许多企业用例需要 无许可区块链技术的性能特征 无法（目前）交付。此外，在许多用例中，标识 参与者是一个硬性要求，例如在财务的情况下 了解您的客户 （KYC） 和反洗钱 （AML） 的交易 必须遵守规定。

对于企业使用，我们需要考虑以下要求：

- 参与者必须被识别/可识别
- 网络需要*获得许可*
- 高事务吞吐量性能
- 交易确认延迟低
- 与业务有关的交易和数据的隐私和机密性交易

虽然许多早期的区块链平台目前正在*适应* 企业使用，Hyperledger *Fabric专为企业*使用而设计，来自 开始。以下部分介绍如何使用Hyperledger Fabric（Fabric） 将自己与其他区块链平台区分开来，并描述了一些 其架构决策的动机。

Fabric 是超级账本社区首个项目也是最流行的分布式账本实现，由 IBM、DAH 等会员企业于 2015 年底贡献到社区。

作为面向企业场景的联盟链，Fabric 中有许多经典的设计和先进的理念。包括多通道、身份证书机制、隐私保护、运维管理接口等。另外，其可扩展的架构可以满足不同场景下的性能需求，如虚拟机部署场景下可以达到 3500 tps 的吞吐量和小于 1 秒的延迟（参考《Hyperledger Fabric: A Distributed Operating System for Permissioned Blockchains》），更多物理资源情况下可以达到更大的（10 K+）的吞吐量。



## 2.2 *Hyperledger Fabric*

*Hyperledger Fabric*是一个开源的企业级许可分布式 账本技术 （DLT） 平台，专为企业环境而设计， 与其他流行的产品相比，它提供了一些关键的差异化功能 分布式账本或区块链平台。

一个关键的区别点是Hyperledger是在 Linux基金会，它本身有着悠久而非常成功的历史 在**开放治理**下培育开源项目，发展壮大 维持社区和繁荣的生态系统。超级账本由 多元化的技术指导委员会，以及Hyperledger Fabric项目。 来自多个组织的各种维护者。它有一个发展 社区已发展到超过 35 个组织和近 200 名开发人员 自最早提交以来。

Fabric 具有高度**模块化**和**可配置**的架构，可实现 针对各种行业用例的创新、多功能性和优化 包括银行、金融、保险、医疗保健、人力资源、供应 连锁甚至数字音乐交付。

Fabric是第一个支持智能合约的分布式账本平台 **用通用编程语言编写**，如Java，Go和 节点.js，而不是受约束的领域特定语言 （DSL）。这意味着 大多数企业已经具备开发智能所需的技能 合同，并且不需要额外的培训来学习新语言或DSL。

Fabric平台也是**经过许可的**，这意味着与公共平台不同。 无需许可的网络，参与者彼此认识，而不是 匿名，因此完全不受信任。这意味着虽然参与者 可能不*完全*信任彼此（例如，他们可能是竞争对手 同行业），网络可以在构建的治理模式下运行 参与者之间*是否存在*信任，例如法律协议 或处理争议的框架。

该平台最重要的区别之一是它支持**可插拔的共识协议**，使平台更加强大。 有效定制以适应特定用例和信任模型。为 实例，当部署在单个企业中时，或由受信任的企业运营时 权威，完全拜占庭容错共识可以考虑 不必要且过度拖累性能和吞吐量。在各种情况下 例如，[崩溃容错](https://en.wikipedia.org/wiki/Fault_tolerance) （CFT） 共识协议可能绰绰有余，而在多方中， 去中心化用例，可能需要更传统的[拜占庭容错](https://en.wikipedia.org/wiki/Byzantine_fault_tolerance)（BFT）共识协议。

Fabric 可以利用不需要本机的共识协议 **加密货币**来激励昂贵的采矿或推动智能合约的执行。 避免使用加密货币可以减少一些重大的风险/攻击媒介， 并且没有加密挖掘操作意味着该平台可以 部署的运营成本与任何其他分布式系统大致相同。

这些差异化设计特征的结合使Fabric成为其中之一 当今在交易方面表现**更好的平台** 处理和交易确认延迟，它使交易和智能合约的**隐私和机密性**（Fabric称之为 “链码”）来实现它们。

让我们更详细地探讨这些差异化功能。



## 2.3 模块性

Hyperledger Fabric经过专门设计，具有模块化 建筑。无论是可插拔共识，还是可插拔身份管理 协议，如 LDAP 或 OpenID 连接、密钥管理协议或 加密库，该平台的核心设计为 配置为满足企业用例要求的多样性。

在高级别上，Fabric 由以下模块化组件组成：

- 可插拔*排序服务*建立关于 事务，然后将块广播到对等方。
- 可插入*成员资格服务提供商*负责关联 网络中具有加密标识的实体。
- 可选的点*对点八卦服务*通过以下方式传播输出的块 向其他对等方订购服务。
- 智能合约（“链码”）在容器环境中运行（例如Docker） 用于隔离。它们可以用标准编程语言编写，但不能 可以直接访问账本状态。
- 账本可以配置为支持各种 DBMS。
- 可插拔的认可和验证策略实施，可以是 每个应用程序独立配置。

业内有公平的共识，即没有“一个区块链 统治他们所有人”。Hyperledger Fabric可以通过多种方式进行配置，以 满足多个行业用例的多样化解决方案需求。



## 2.4 许可与无许可区块链

在无需许可的区块链中，几乎任何人都可以参与，并且每个 参与者是匿名的。在这种情况下，除了 在达到一定深度之前，区块链的状态是不可变的。在 为了缓解这种信任的缺失，无许可区块链通常 使用“开采”的原生加密货币或交易费用来提供经济 以某种形式抵消参与的特殊成本的激励措施 基于“工作量证明”（PoW）的拜占庭容错共识。

另一方面，**许可区块链**在 一组已知、已识别且经常经过审查的参与者，在 产生一定程度信任的治理模型。一个许可的 区块链提供了一种保护一组实体之间交互的方法 有共同的目标，但可能不完全信任对方。依靠 参与者的身份，一个许可的区块链可以使用更多 传统崩溃容错 （CFT） 或拜占庭容错 （BFT） 不需要昂贵挖矿的共识协议。

此外，在这种许可的上下文中，参与者的风险 通过智能合约故意引入恶意代码的情况减少了。 首先，参与者彼此了解，所有行动，无论 提交应用程序事务，修改网络配置 或部署智能合约记录在区块链上，然后 为网络和相关交易制定的背书政策 类型。而不是完全匿名，有罪的一方可以很容易地 根据以下条款确定并处理事件 治理模型。



## 2.5 智能合约

智能合约，或Fabric所谓的“链码”，充当受信任的 从区块链中获得安全性/信任的分布式应用程序 同行之间的基本共识。它是 区块链应用。

有三个关键点适用于智能合约，特别是当 应用于平台：

- 许多智能合约在网络中同时运行，
- 它们可以动态部署（在许多情况下由任何人部署），并且
- 应用程序代码应被视为不受信任，甚至可能 恶意。

大多数现有的智能合约区块链平台都遵循**订单执行**架构，其中共识协议：

- 验证和排序事务，然后将其传播到所有对等节点，
- 然后，每个对等方按顺序执行事务。

订单执行架构几乎可以在所有现有的区块链中找到 系统，从公共/无许可平台，如[以太坊](https://ethereum.org/)（具有基于PoW的共识）到许可的 平台，如[Tendermint](http://tendermint.com/)，[Chain](http://chain.com/)和[Quorum](http://www.jpmorgan.com/global/Quorum)。

在区块链中执行的智能合约与订单执行一起运行 架构必须是确定性的;否则，可能永远无法达成共识。 为了解决非确定性问题，许多平台要求智能 合同应以非标准或特定领域的语言编写 （如[坚固性](https://solidity.readthedocs.io/en/v0.4.23/)）以便 可以消除非确定性操作。这阻碍了广泛传播 采用，因为它需要开发人员编写智能合约来学习新的 语言，并可能导致编程错误。

此外，由于所有事务都由所有节点按顺序执行， 性能和规模有限。智能合约代码执行的事实 在系统中的每个节点上，都需要采取复杂的措施来保护 整个系统从潜在的恶意合同中得到保证 整个系统的复原能力。



## 2.6 新方法

Fabric 为我们称之为 **execute-order-validate** 的交易引入了一种新的架构。它解决了弹性、灵活性、 可扩展性、性能和机密性挑战 通过将事务流分为三个步骤来建立订单-执行模型：

- *执行*交易并检查其正确性，从而认可它，
- 通过（可插拔）共识协议*订购交易，*以及
- 根据特定于应用程序的背书策略*验证*事务 在将它们提交到账本之前

这种设计与该结构中的顺序执行范式完全不同 在就订单达成最终协议之前执行交易。

在 Fabric 中，特定于应用程序的认可策略指定哪个对等方 节点，或者其中有多少节点，需要保证正确执行给定的节点 智能合约。因此，每笔交易只需要由 满足事务背书所需的对等节点子集 政策。这允许并行执行，从而提高整体性能和 系统的规模。第一阶段也**消除了任何非确定性**， 因为在订购前可以过滤掉不一致的结果。

因为我们已经消除了非确定性，所以Fabric是第一个区块链。 **支持使用标准编程语言**的技术。



## 2.7 隐私和保密

正如我们所讨论的，在一个公共的、无需许可的区块链网络中， 利用PoW作为其共识模型，交易在每个节点上执行。 这意味着合同也不能保密 他们自己，也不是他们处理的交易数据。每笔交易， 实现它的代码对网络中的每个节点都是可见的。在 在这种情况下，我们用合同和数据的保密性换取了拜占庭式的 PoW 提供的容错共识。

这种缺乏机密性对于许多企业/企业使用来说可能是个问题 例。例如，在供应链合作伙伴网络中，一些消费者可能会 获得优惠利率作为巩固关系的手段，或 促进额外销售。如果每个参与者都可以看到每份合同，并且 交易，不可能在 完全透明的网络 - 每个人都会想要优惠的价格！

作为第二个例子，考虑证券行业，其中交易者建立 一个职位（或处置一个职位）不会希望她的竞争对手知道这一点， 否则，他们将寻求参与游戏，削弱交易者的赌注。

为了解决缺乏隐私和保密性的问题 满足企业用例要求，区块链平台具有 采取了多种方法。所有人都有自己的权衡。

加密数据是提供机密性的一种方法;然而，在一个 利用PoW达成共识的无许可网络，加密数据是 坐在每个节点上。如果有足够的时间和计算资源， 加密可能会被破坏。对于许多企业用例，其风险 信息可能被泄露是不可接受的。

零知识证明（ZKP）是另一个正在探索的研究领域 解决这个问题，这里的权衡是，目前，计算一个ZKP 需要大量的时间和计算资源。因此，权衡 本案为保密性履行。

在可以利用替代形式的共识的许可上下文中，一个 可能会探索限制机密分发的方法 仅向授权节点提供信息。

Hyperledger Fabric是一个许可平台，可以实现机密性 通过其通道架构和[私有数据](https://hyperledger-fabric.readthedocs.io/en/release-2.5/private-data/private-data.html)功能。在频道中，结构网络上的参与者建立子网 其中每个成员都可以查看一组特定的事务。因此，只有 那些参与通道的节点可以访问智能合约 （链码）和交易的数据，保护隐私和机密性 双。私有数据允许频道上成员之间的集合，允许 与通道相同的保护，没有 创建和维护单独的频道。



## 2.8 可插拔共识

交易的顺序被委托给模块化组件以达成共识 在逻辑上与执行事务的对等方分离，并且 维护分类账。具体来说，订购服务。由于共识是 模块化，其实现可以根据信任假设进行定制 特定部署或解决方案。这种模块化架构允许平台 依靠完善的 CFT（崩溃容错）或 BFT 工具包 （拜占庭容错）排序。

Fabric 目前提供 CFT 排序服务实现 基于 [Raft 协议](https://raft.github.io/raft.pdf)[``的库](https://coreos.com/etcd/)。 有关当前可用的订购服务的信息，请查看 请参阅我们[关于订购的概念文档](https://hyperledger-fabric.readthedocs.io/en/release-2.5/orderer/ordering_service.html)。

另请注意，它们并不相互排斥。结构网络可以具有 支持不同应用或应用的多种订购服务 要求。



## 2.9 性能和可扩展性

区块链平台的性能可能会受到许多变量的影响，例如 事务大小、块大小、网络大小以及可用的硬件资源，如 CPU、内存、磁盘空间、磁盘和网络 I/O。



## 2.10 结论

任何对区块链平台的认真评估都应该包括Hyperledger Fabric。 在其短名单中。

综合起来，Fabric 的差异化功能使其成为高度可扩展的 支持灵活信任假设的许可区块链系统 使该平台能够支持广泛的行业用例，包括 政府、金融、供应链物流、医疗保健等等 更多。

Hyperledger Fabric是Hyperledger项目中最活跃的。社区 围绕平台的构建正在稳步增长，并且交付了创新 随着每个连续的版本远远超过任何其他企业区块链。 平台。



## 2.11 确认

以上内容来自同行评审的[“Hyperledger Fabric： A Distributed Operating System for Permissioned Blockchains”](https://dl.acm.org/doi/10.1145/3190508.3190538) - Elli Androulaki，Artem 巴格、维塔·博尔特尼科夫、克里斯蒂安·卡钦、康斯坦丁诺斯·克里斯蒂迪斯、安杰洛·德 卡罗、大卫·恩伊尔特、克里斯托弗·费里斯、根纳季·拉文特曼、雅科夫·马内维奇、 斯里尼瓦桑·穆拉利达兰、切特·穆尔蒂、平阮、马尼什·塞西、加里·辛格、 基思·史密斯、亚历山德罗·索尼奥蒂、克里斯苏拉·斯塔塔科普卢、马尔科·武科利奇、 莎朗·威德·科科，杰森·耶利克



## 2.12 **主要版本历史**

Fabric 首个主版本 1.0.0 于 2017 年 7 月发布，该版本根据之前测试版的应用反馈，在架构上进行了重新设计，在可扩展性和可插拔性方面都有了不少改进。后续版本基本上按照每季度一个小版本的节奏发布。重点对性能和安全性、隐私性进行了完善和提升。目前最新大版本为 2.0.0 版本。Fabric 的主要版本历史总结如下表所示，也可在 https://github.com/hyperledger/fabric/releases 找到。

| 版本  | 发布时间    | 新特性                                                       | 增强和改善                                                   |
| ----- | ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1.0.0 | 2017-07-11  | 重新设计架构，支持多通道、Kafka 共识机制、系统链码、分角色节点 | 大幅度提升性能和可扩展性                                     |
| 1.1.0 | 2018-03-15  | Node.Js 链码，链码加密库，链码中基于证书属性的访问控制，节点之间的双向 TLS，Fabric CA 中对 CRL 的支持。部分实验特性，如 sideDB、idemix、细粒度的权限控制。 | 大幅优化了性能，某些场景下可提升一个数量级；提供基于通道的事件通知模型 |
| 1.2.0 | 2018-07-04  | 正式支持私有数据库（Private Data），提供可插拔的 ESCC 和 VSCC，细粒度访问控制，服务自动发现 | 提高了稳定性和易用性                                         |
| 1.3.0 | 2018-10-11  | 正式支持 Java 链码；提供细粒度的基于状态的背书策略；支持 idemix 增强隐私保护 | 部分重构链码生命周期管理；通过分页机制优化链码中对大量数据的查询 |
| 1.4.0 | 2019-01-09  | 提供运维相关的 RESTful API（统计、健康检查、日志级别）；使用新的日志控制环境变量 | 增强私有数据：新 Peer 自动获取缺失私有数据，客户端层面对私有数据的权限控制；开始支持新的 RAFT 排序机制 |
| 2.0.0 | 2020-03-TBD | 新的面向通道的链码生命周期管理；独立 shim 库等               | 增强私有数据支持；增强排序服务；改进性能                     |



## 2.13 **网络基本结构**

Fabric 网络中存在四种不同种类的服务节点，彼此协作完成整个区块链系统的记账功能：

- 背书节点（Endorser Peer）：一类特殊的 Peer，对交易提案（Transaction Proposal）进行检查，通过执行智能合约计算交易执行结果（读写集合）并对其进行背书；
- 记账节点（Committer Peer）：负责维护账本，检查排序后交易结果合法性，接受合法修改，并写入到本地账本结构，目前所有 Peer 默认都是记账节点；
- 排序节点（Orderer）：正式交易会发给排序节点，排序节点负责对网络中所有交易进行排序处理，并整理为区块结构，之后被记账节点拉取提交到本地账本；
- 证书节点（CA）：提供标准的 PKI 服务，负责对网络中所有的证书进行管理，包括签发和撤销。

节点角色是 Fabric 设计中的一大创新。根据性能和安全需求，不同的节点可以由组织分别管理，共同构建联盟链。

此外，网络账本的基本单位是通道（Channel），每个通道内的成员可以共享账本，不同通道内账本则彼此隔离。客户端可以向网络内发送交易，交易经过共识后被通道内的 Peer 节点接收并更新本地对应的账本。

本章后续将详细介绍安装、部署 Fabric 网络并进行启动的操作过程，建议读者跟随步骤进行实践学习。



# *Hyperledger Fabric*本地编译组件实战教程

本地编译生成 *Fabric* 网络的各个组件，可以形成更直观的认识。*Fabric* 采用 *Go* 语言实现，推荐使用 Golang 1.20+ 版本进行编译。

下面将讲解如何编译生成 fabric-peer、fabric-orderer 和 fabric-ca 等组件的二进制文件，以及如何安装一些配置和开发辅助工具。如果用户在多服务器环境下进行部署，需要注意将文件复制到对应的服务器上。

## **1.获取Fabric源码**

目前，Fabric 官方仓库托管在 Github 仓库（github.com/hyperledger/fabric）中供下载使用。

如果使用 1.13 之前版本的 Go 环境，需要将 Fabric 项目放到 $GOPATH 路径下。如下命令所示，创建 `$GOPATH/src/github.com/hyperledger` 目录结构并切换到该路径：

```shell
$ cd ~
$ sudo apt install jq gcc make
$ mkdir -p ~/go/src ~/go/pkg ~/go/bin
$ mkdir -p $GOPATH/src/github.com/hyperledger
$ cd $GOPATH/src/github.com/hyperledger
```

获取 Peer 和 Orderer 组件编译所需要的代码，两者目前在同一个 fabric 仓库中：

```shell
$ git clone https://github.com/hyperledger/fabric.git
$ cd fabric
$ git branch -a
$ git checkout -b v2.5.3
$ git branch
```

![image-20221002173427662](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221002173427662.png)

为节约下载时间，读者可以指定 `--single-branch -b master --depth 1` 命令选项来指定只获取 master 分支最新代码：

```shell
$ git clone --single-branch -b master --depth 1 https://github.com/hyperledger/fabric.git
```

Fabric CA 组件在独立的 fabric-ca 仓库中，可以通过如下命令获取：

```shell
$ cd $GOPATH/src/github.com/hyperledger
$ git clone https://github.com/hyperledger/fabric-ca.git
$ git checkout -b v1.5.7
$ git branch
```



> 读者也可以直接访问 https://github.com/hyperledger/fabric/releases 和 https://github.com/hyperledger/fabric-ca/releases 来下载特定的 fabric 和 fabric-ca 发行版。



最后，检查确认 fabric 和 fabric-ca 两个仓库下载成功：

```shell
$ ls $GOPATH/src/github.com/hyperledger
fabric fabric-ca
```

![image-20221002192005270](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221002192005270.png)



## 2.编译脚本文件（Makefile）

```makefile
# Copyright IBM Corp All Rights Reserved.
# Copyright London Stock Exchange Group All Rights Reserved.
#
# SPDX-License-Identifier: Apache-2.0
#
# -------------------------------------------------------------
# This makefile defines the following targets
#
#   - all (default) - builds all targets and runs all non-integration tests/checks
#   - basic-checks - performs basic checks like license, spelling, trailing spaces and linter
#   - check-deps - check for vendored dependencies that are no longer used
#   - checks - runs all non-integration tests/checks
#   - clean-all - superset of 'clean' that also removes persistent state
#   - clean - cleans the build area
#   - configtxgen - builds a native configtxgen binary
#   - configtxlator - builds a native configtxlator binary
#   - cryptogen - builds a native cryptogen binary
#   - desk-check - runs linters and verify to test changed packages
#   - dist-clean - clean release packages for all target platforms
#   - docker[-clean] - ensures all docker images are available[/cleaned]
#   - docker-list - generates a list of docker images that 'make docker' produces
#   - docker-tag-latest - re-tags the images made by 'make docker' with the :latest tag
#   - docker-tag-stable - re-tags the images made by 'make docker' with the :stable tag
#   - docker-thirdparty - pulls thirdparty images (couchdb, etc)
#   - docs - builds the documentation in html format
#   - gotools - installs go tools like golint
#   - help-docs - generate the command reference docs
#   - integration-test-prereqs - setup prerequisites for integration tests
#   - integration-test - runs the integration tests
#   - ledgerutil - builds a native ledgerutil binary
#   - license - checks go source files for Apache license header
#   - linter - runs all code checks
#   - native - ensures all native binaries are available
#   - orderer - builds a native fabric orderer binary
#   - orderer-docker[-clean] - ensures the orderer container is available[/cleaned]
#   - osnadmin - builds a native fabric osnadmin binary
#   - peer - builds a native fabric peer binary
#   - peer-docker[-clean] - ensures the peer container is available[/cleaned]
#   - profile - runs unit tests for all packages in coverprofile mode (slow)
#   - protos - generate all protobuf artifacts based on .proto files
#   - publish-images - publishes release docker images to nexus3 or docker hub.
#   - release-all - builds release packages for all target platforms
#   - release - builds release packages for the host platform
#   - tools-docker[-clean] - ensures the tools container is available[/cleaned]
#   - unit-test-clean - cleans unit test state (particularly from docker)
#   - unit-test - runs the go-test based unit tests
#   - verify - runs unit tests for only the changed package tree

UBUNTU_VER ?= 22.04
FABRIC_VER ?= 2.5.3

# 3rd party image version
# These versions are also set in the runners in ./integration/runners/
COUCHDB_VER ?= 3.3.2

# Disable implicit rules
.SUFFIXES:
MAKEFLAGS += --no-builtin-rules

BUILD_DIR ?= build

EXTRA_VERSION ?= $(shell git rev-parse --short HEAD)
PROJECT_VERSION=$(FABRIC_VER)-snapshot-$(EXTRA_VERSION)

# TWO_DIGIT_VERSION is derived, e.g. "2.0", especially useful as a local tag
# for two digit references to most recent baseos and ccenv patch releases
# TWO_DIGIT_VERSION removes the (optional) semrev 'v' character from the git
# tag triggering a Fabric release.
TWO_DIGIT_VERSION = $(shell echo $(FABRIC_VER) | sed -e  's/^v\(.*\)/\1/' | cut -d '.' -f 1,2)

PKGNAME = github.com/hyperledger/fabric
ARCH=$(shell go env GOARCH)
MARCH=$(shell go env GOOS)-$(shell go env GOARCH)

# defined in common/metadata/metadata.go
METADATA_VAR = Version=$(FABRIC_VER)
METADATA_VAR += CommitSHA=$(EXTRA_VERSION)
METADATA_VAR += BaseDockerLabel=$(BASE_DOCKER_LABEL)
METADATA_VAR += DockerNamespace=$(DOCKER_NS)

GO_VER = 1.20.8
GO_TAGS ?=

RELEASE_EXES = orderer $(TOOLS_EXES)
RELEASE_IMAGES = baseos ccenv orderer peer tools
RELEASE_PLATFORMS = darwin-amd64 darwin-arm64 linux-amd64 linux-arm64 windows-amd64
TOOLS_EXES = configtxgen configtxlator cryptogen discover ledgerutil osnadmin peer

pkgmap.configtxgen    := $(PKGNAME)/cmd/configtxgen
pkgmap.configtxlator  := $(PKGNAME)/cmd/configtxlator
pkgmap.cryptogen      := $(PKGNAME)/cmd/cryptogen
pkgmap.discover       := $(PKGNAME)/cmd/discover
pkgmap.ledgerutil     := $(PKGNAME)/cmd/ledgerutil
pkgmap.orderer        := $(PKGNAME)/cmd/orderer
pkgmap.osnadmin       := $(PKGNAME)/cmd/osnadmin
pkgmap.peer           := $(PKGNAME)/cmd/peer

.DEFAULT_GOAL := all

include docker-env.mk
include gotools.mk

.PHONY: all
all: check-go-version native docker checks

.PHONY: checks
checks: basic-checks unit-test integration-test

.PHONY: basic-checks
basic-checks: check-go-version license spelling references trailing-spaces linter check-help-docs check-metrics-doc filename-spaces check-swagger

.PHONY: desk-checks
desk-check: checks verify

.PHONY: help-docs
help-docs: native
	@scripts/help_docs.sh

.PHONY: check-help-docs
check-help-docs: native
	@scripts/help_docs.sh check

.PHONY: spelling
spelling: gotool.misspell
	@scripts/check_spelling.sh

.PHONY: references
references:
	@scripts/check_references.sh

.PHONY: license
license:
	@scripts/check_license.sh

.PHONY: trailing-spaces
trailing-spaces:
	@scripts/check_trailingspaces.sh

.PHONY: gotools
gotools: gotools-install

.PHONY: check-go-version
check-go-version:
	@scripts/check_go_version.sh $(GO_VER)

.PHONY: integration-test
integration-test: integration-test-prereqs
	./scripts/run-integration-tests.sh $(INTEGRATION_TEST_SUITE)

.PHONY: integration-test-prereqs
integration-test-prereqs: gotool.ginkgo baseos-docker ccenv-docker docker-thirdparty ccaasbuilder

.PHONY: unit-test
unit-test: unit-test-clean docker-thirdparty-couchdb
	./scripts/run-unit-tests.sh

.PHONY: unit-tests
unit-tests: unit-test

# Pull thirdparty docker images based on the latest baseimage release version
# Also pull ccenv-1.4 for compatibility test to ensure pre-2.0 installed chaincodes
# can be built by a peer configured to use the ccenv-1.4 as the builder image.
.PHONY: docker-thirdparty
docker-thirdparty: docker-thirdparty-couchdb
	docker pull hyperledger/fabric-ccenv:1.4

.PHONY: docker-thirdparty-couchdb
docker-thirdparty-couchdb:
	docker pull couchdb:${COUCHDB_VER}

.PHONY: verify
verify: export JOB_TYPE=VERIFY
verify: unit-test

.PHONY: profile
profile: export JOB_TYPE=PROFILE
profile: unit-test

.PHONY: linter
linter: check-deps gotool.goimports gotool.gofumpt gotool.staticcheck
	@echo "LINT: Running code checks.."
	./scripts/golinter.sh

.PHONY: check-deps
check-deps:
	@echo "DEP: Checking for dependency issues.."
	./scripts/check_deps.sh

.PHONY: check-metrics-docs
check-metrics-doc:
	@echo "METRICS: Checking for outdated reference documentation.."
	./scripts/metrics_doc.sh check

.PHONY: generate-metrics-docs
generate-metrics-doc:
	@echo "Generating metrics reference documentation..."
	./scripts/metrics_doc.sh generate

.PHONY: check-swagger
check-swagger: gotool.swagger
	@echo "SWAGGER: Checking for outdated swagger..."
	./scripts/swagger.sh check

.PHONY: generate-swagger
generate-swagger: gotool.swagger
	@echo "Generating swagger..."
	./scripts/swagger.sh generate

.PHONY: protos
protos: gotool.protoc-gen-go
	@echo "Compiling non-API protos..."
	./scripts/compile_protos.sh

.PHONY: native
native: $(RELEASE_EXES)

.PHONY: tools
tools: $(TOOLS_EXES)

.PHONY: $(RELEASE_EXES)
$(RELEASE_EXES): %: $(BUILD_DIR)/bin/%

$(BUILD_DIR)/bin/%: GO_LDFLAGS = $(METADATA_VAR:%=-X $(PKGNAME)/common/metadata.%)
$(BUILD_DIR)/bin/%:
	@echo "Building $@"
	@mkdir -p $(@D)
	GOBIN=$(abspath $(@D)) go install -tags "$(GO_TAGS)" -ldflags "$(GO_LDFLAGS)" -buildvcs=false $(pkgmap.$(@F))
	@touch $@

.PHONY: docker
docker: $(RELEASE_IMAGES:%=%-docker) ccaasbuilder

.PHONY: $(RELEASE_IMAGES:%=%-docker)
$(RELEASE_IMAGES:%=%-docker): %-docker: $(BUILD_DIR)/images/%/$(DUMMY)

$(BUILD_DIR)/images/baseos/$(DUMMY):  BUILD_CONTEXT=images/baseos
$(BUILD_DIR)/images/ccenv/$(DUMMY):   BUILD_CONTEXT=images/ccenv
$(BUILD_DIR)/images/peer/$(DUMMY):    BUILD_ARGS=--build-arg GO_TAGS=${GO_TAGS}
$(BUILD_DIR)/images/orderer/$(DUMMY): BUILD_ARGS=--build-arg GO_TAGS=${GO_TAGS}
$(BUILD_DIR)/images/tools/$(DUMMY):   BUILD_ARGS=--build-arg GO_TAGS=${GO_TAGS}

$(BUILD_DIR)/images/%/$(DUMMY):
	@echo "Building Docker image $(DOCKER_NS)/fabric-$*"
	@mkdir -p $(@D)
	$(DBUILD) -f images/$*/Dockerfile \
		--build-arg GO_VER=$(GO_VER) \
		--build-arg UBUNTU_VER=$(UBUNTU_VER) \
		--build-arg FABRIC_VER=$(FABRIC_VER) \
		--build-arg TARGETARCH=$(ARCH) \
		--build-arg TARGETOS=linux \
		$(BUILD_ARGS) \
		-t $(DOCKER_NS)/fabric-$* ./$(BUILD_CONTEXT)

# builds release packages for the host platform
.PHONY: release
release: check-go-version $(MARCH:%=release/%)

# builds release packages for all target platforms
.PHONY: release-all
release-all: check-go-version $(RELEASE_PLATFORMS:%=release/%)

.PHONY: $(RELEASE_PLATFORMS:%=release/%)
$(RELEASE_PLATFORMS:%=release/%): GO_LDFLAGS = $(METADATA_VAR:%=-X $(PKGNAME)/common/metadata.%)
$(RELEASE_PLATFORMS:%=release/%): release/%: $(foreach exe,$(RELEASE_EXES),release/%/bin/$(exe))
$(RELEASE_PLATFORMS:%=release/%): release/%: ccaasbuilder/%

# explicit targets for all platform executables
$(foreach platform, $(RELEASE_PLATFORMS), $(RELEASE_EXES:%=release/$(platform)/bin/%)):
	$(eval platform = $(patsubst release/%/bin,%,$(@D)))
	$(eval GOOS = $(word 1,$(subst -, ,$(platform))))
	$(eval GOARCH = $(word 2,$(subst -, ,$(platform))))
	@echo "Building $@ for $(GOOS)-$(GOARCH)"
	mkdir -p $(@D)
	GOOS=$(GOOS) GOARCH=$(GOARCH) go build -o $@ -tags "$(GO_TAGS)" -ldflags "$(GO_LDFLAGS)" -buildvcs=false $(pkgmap.$(@F))

.PHONY: dist
dist: dist-clean dist/$(MARCH)

.PHONY: dist-all
dist-all: dist-clean $(RELEASE_PLATFORMS:%=dist/%)
dist/%: release/% ccaasbuilder
	mkdir -p release/$(@F)/config
	cp -r sampleconfig/*.yaml release/$(@F)/config
	cd release/$(@F) && tar -czvf hyperledger-fabric-$(@F).$(PROJECT_VERSION).tar.gz *

.PHONY: docker-list
docker-list: $(RELEASE_IMAGES:%=%-docker-list)
%-docker-list:
	@echo $(DOCKER_NS)/fabric-$*:$(DOCKER_TAG)

.PHONY: docker-clean
docker-clean: $(RELEASE_IMAGES:%=%-docker-clean)
%-docker-clean:
	-@for image in "$$(docker images --quiet --filter=reference='$(DOCKER_NS)/fabric-$*')"; do \
		[ -z "$$image" ] || docker rmi -f $$image; \
	done
	-@rm -rf $(BUILD_DIR)/images/$* || true

.PHONY: docker-tag-latest
docker-tag-latest: $(RELEASE_IMAGES:%=%-docker-tag-latest)
%-docker-tag-latest:
	docker tag $(DOCKER_NS)/fabric-$*:$(DOCKER_TAG) $(DOCKER_NS)/fabric-$*:latest

.PHONY: docker-tag-stable
docker-tag-stable: $(RELEASE_IMAGES:%=%-docker-tag-stable)
%-docker-tag-stable:
	docker tag $(DOCKER_NS)/fabric-$*:$(DOCKER_TAG) $(DOCKER_NS)/fabric-$*:stable

.PHONY: publish-images
publish-images: $(RELEASE_IMAGES:%=%-publish-images)
%-publish-images:
	@docker login $(DOCKER_HUB_USERNAME) $(DOCKER_HUB_PASSWORD)
	@docker push $(DOCKER_NS)/fabric-$*:$(PROJECT_VERSION)

.PHONY: clean
clean: docker-clean unit-test-clean release-clean
	-@rm -rf $(BUILD_DIR)

.PHONY: clean-all
clean-all: clean gotools-clean dist-clean
	-@rm -rf /var/hyperledger/*
	-@rm -rf docs/build/

.PHONY: dist-clean
dist-clean:
	-@for platform in $(RELEASE_PLATFORMS) ""; do \
		[ -z "$$platform" ] || rm -rf release/$${platform}/hyperledger-fabric-$${platform}.$(PROJECT_VERSION).tar.gz; \
	done

.PHONY: release-clean
release-clean: $(RELEASE_PLATFORMS:%=%-release-clean)
%-release-clean:
	-@rm -rf release/$*

.PHONY: unit-test-clean
unit-test-clean:

.PHONY: filename-spaces
spaces:
	@scripts/check_file_name_spaces.sh

.PHONY: docs
docs:
	@docker run --rm -v $$(pwd):/docs n42org/tox:3.4.0 sh -c 'cd /docs && tox -e docs'

.PHONY: ccaasbuilder-clean
ccaasbuilder-clean/%:
	$(eval platform = $(patsubst ccaasbuilder/%,%,$@) )
	cd ccaas_builder &&	rm -rf $(strip $(platform))

.PHONY: ccaasbuilder
ccaasbuilder/%: ccaasbuilder-clean
	$(eval platform = $(patsubst ccaasbuilder/%,%,$@) )
	$(eval GOOS = $(word 1,$(subst -, ,$(platform))))
	$(eval GOARCH = $(word 2,$(subst -, ,$(platform))))
	@mkdir -p release/$(strip $(platform))/builders/ccaas/bin
	cd ccaas_builder && go test -v ./cmd/detect && GOOS=$(GOOS) GOARCH=$(GOARCH) go build -buildvcs=false -o ../release/$(strip $(platform))/builders/ccaas/bin/ ./cmd/detect/
	cd ccaas_builder && go test -v ./cmd/build && GOOS=$(GOOS) GOARCH=$(GOARCH) go build -buildvcs=false -o ../release/$(strip $(platform))/builders/ccaas/bin/ ./cmd/build/
	cd ccaas_builder && go test -v ./cmd/release && GOOS=$(GOOS) GOARCH=$(GOARCH) go build -buildvcs=false -o ../release/$(strip $(platform))/builders/ccaas/bin/ ./cmd/release/

ccaasbuilder: ccaasbuilder/$(MARCH)

.PHONY: scan
scan: scan-govulncheck

.PHONY: scan-govulncheck
scan-govulncheck: gotool.govulncheck
	govulncheck ./...

```



## 3.**编译安装 Peer 组件**

配置版本号和编译参数：

```shell

$ PROJECT_VERSION=2.5.3
$ LD_FLAGS="-X github.com/hyperledger/fabric/common/metadata.Version=${PROJECT_VERSION} \
            -X github.com/hyperledger/fabric/common/metadata.BaseDockerLabel=org.hyperledger.fabric \
            -X github.com/hyperledger/fabric/common/metadata.DockerNamespace=hyperledger \
            -X github.com/hyperledger/fabric/common/metadata.BaseDockerNamespace=hyperledger"
```

**方式1：**通过如下命令编译并安装 fabric 的 peer 组件到 $GOPATH/bin 下：

```shell
$ cd ~/go/src/github.com/hyperledger/fabric
$ CGO_CFLAGS=" " go install -tags "" -ldflags "$LD_FLAGS" \
github.com/hyperledger/fabric/cmd/peer
```



**方式2：**当然，用户也可直接使用源码中的 Makefile 来进行编译，相关命令如下：

```shell
$ make peer
```

这种情况下编译生成的 peer 组件会默认放在 *build/bin* 路径下。

## 4.**编译安装 Orderer 组件**

**方式1：**通过如下命令编译并安装 fabric orderer 组件到 $GOPATH/bin 下：

```shell
$ CGO_CFLAGS=" " go install -tags "" -ldflags "$LD_FLAGS" \
github.com/hyperledger/fabric/cmd/orderer
```



**方式2：**同样的，也可使用 Makefile 来编译安装 orderer 组件到 build/bin 路径下：

```shell
$ make orderer
```

## 5.**编译安装 Fabric CA 组件**

采用如下命令编译并安装 fabric-ca 相关组件到 $GOPATH/bin 下：

```bash
$ cd ~/go/src/github.com/hyperledger/fabric-ca
$ go install -ldflags "-X github.com/hyperledger/fabric-ca/lib/metadata.Version=$PROJECT_VERSION" \
github.com/hyperledger/fabric-ca/cmd/...

或
$ go install github.com/hyperledger/fabric-ca/cmd/...

或链接静态库
$go install -ldflags "-X github.com/hyperledger/fabric-ca/lib/metadata.Version=$PROJECT_VERSION\
-linkmode external -extldflags '-static -lpthread'" \
github.com/hyperledger/fabric-ca/cmd/...
```



## 6.**编译安装配置辅助工具**

Fabric 中还提供了一系列配置辅助工具，包括：

- cryptogen（本地生成组织结构和身份文件）
- configtxgen（生成配置区块和配置交易）
- configtxlator（解析转换配置信息）
- discover（拓扑探测）
- idemixgen（Idemix 证书生成）等

可以通过如下命令来快速编译和安装：

```shell
# 编译安装 cryptogen，等价于执行 make cryptogen
$ CGO_CFLAGS=" " \
go install -tags "" -ldflags "$LD_FLAGS" \
github.com/hyperledger/fabric/cmd/cryptogen

# 编译安装 configtxgen，等价于执行 make configtxgen
$ CGO_CFLAGS=" " \
go install -tags "" -ldflags "$LD_FLAGS" \
github.com/hyperledger/fabric/cmd/configtxgen

# 编译安装 configtxlator，等价于执行 make configtxlator
$ CGO_CFLAGS=" " \
go install -tags "" -ldflags "$LD_FLAGS" \
github.com/hyperledger/fabric/cmd/configtxlator

# 编译安装 discover，等价于执行 make discover
$ CGO_CFLAGS=" " \
go install -tags "" -ldflags "$LD_FLAGS" \
github.com/hyperledger/fabric/cmd/discover

# 编译安装 idemixgen，等价于执行 make idemixgen
# fabric 2.5.3没有idemixen,另外新增了ledgerutil,不在编译该命令
$ CGO_CFLAGS=" " \
go install -tags "" -ldflags "$LD_FLAGS" \
github.com/hyperledger/fabric/cmd/idemixgen

# 编译安装 osnadmin，等价于执行 make osnadmin,v2.2.3版本所需
$ CGO_CFLAGS=" " \
  go install -tags "" -ldflags "$LD_FLAGS" \
  github.com/hyperledger/fabric/cmd/osnadmin
  
# 编译安装ledgerutil，等价于执行 make ledgerutil,v2.5.3版本所需
$ CGO_CFLAGS=" " \
  go install -tags "" -ldflags "$LD_FLAGS" \
  github.com/hyperledger/fabric/cmd/ledgerutil
```



另外，fabric 项目还提供了不少常见的编译命令，可以参考 Makefile 文件，例如编译所有的二进制文件可以使用如下命令：

```sh
$ make native
```

## 5.**安装 Protobuf 支持和 Go 语言相关工具**

Fabric 代码由 Go 语言构建，开发者可以选择安装如下的 Go 语言相关工具，方便开发和调试：

```shell
# 进入GOPATH目录,如果成功，会在GOPATH/bin下生成protoc-gen-go.exe.....文件
$ go get github.com/golang/protobuf/protoc-gen-go
$ go get github.com/maxbrunsfeld/counterfeiter/v6
$ go get github.com/axw/gocov/...
$ go get github.com/AlekSi/gocov-xml
$ go get golang.org/x/tools/cmd/goimports
$ go get golang.org/x/lint/golint
$ go get github.com/estesp/manifest-tool
$ go get github.com/client9/misspell/cmd/misspell
$ go get github.com/onsi/ginkgo/ginkgo
```

### go get问题

对我来说，`go get`不知道为什么，就是各种`time out`，尝试了很多方法还是一样。这里说一下，`go get`实际上做了什么事情，首先他通过`git`将源码下载到`$GOPATH/src`中，然后执行`go install`来编译源码，输出可执行文件到`$GOPATH/bin`中，所以`go get`失败的可以尝试去把源码下载下来，然后手动`go install`。
举个例子，安装`maxbrunsfeld/counterfeiter`，这个在搭建`fabric`的时候会用到：看`README.md`的安装说明：
`go get -u github.com/maxbrunsfeld/counterfeiter`，然后执行以下命令，

```shell
$ mkdir -p $GOPATH/src/github.com/maxbrunsfeld
$ cd $GOPATH/src/github.com/maxbrunsfeld
$ git clone https://github.com/maxbrunsfeld/counterfeiter.git
$ go install ./counterfeiter
```

### golang/tools

这个也是搭建`fabric`会用到

```shell
$ mkdir -p $GOPATH/src/golang.org/x/
$ cd $GOPATH/src/golang.org/x/
$ git clone https://github.com/golang/tools.git
```

这个不需要`go install`



## 6.Sampleconfig 示例配置

sampleconfig 目录下包括了一些示例配置文件，可以作为参考，包括：

- configtx.yaml：示例配置区块生成文件夹；
- orderer.yaml：示例 Orderer 节点配置文件；
- core.yaml：示例 Peer 节点配置文件；
- msp/config.yaml：示例组织身份配置文件；
- msp：示例证书和秘钥文件。

可以将它们复制到fabric-samples目录下进行使用演示。



## 7.*Docker*容器镜像

除了手动进行本地编译外，还可以采用容器（Docker）镜像的方式快速获取和运行 Fabric 网络，节约本地编译等待的时间。

### **Docker 镜像依赖关系**

Docker 镜像可以自行从源码编译，或通过从 DockerHub 仓库下载等方式。目前，Fabric 项目相关的 Docker 镜像有十几个，这些镜像都在 `hyperledger` 仓库中，它们之间的相互依赖关系如下图所示。



​                                                                    镜像之间的依赖关系

从用途上，可以可以大致分为三类：核心镜像、辅助镜像和第三方镜像。

### **核心镜像**

提供 Fabric 网络运行的核心功能，目前包括 fabric-peer、fabric-orderer、fabric-ca、fabric-baseos、fabric-ccenv、fabric-javaenv、fabric-nodeenv 共 7 种镜像。如下表所示。

| 镜像名称       | 父镜像                                      | 功能描述                                                     |
| -------------- | :------------------------------------------ | ------------------------------------------------------------ |
| fabric-peer    | alpine:3.10                                 | peer 节点镜像，安装了 peer 相关文件。生成过程使用 golang:1.13.4-alpine3.10 镜像。 |
| fabric-orderer | alpine:3.10                                 | orderer 节点镜像，安装了 orderer 相关文件。生成过程使用 golang:1.13.4-alpine3.10 镜像。 |
| fabric-ca      | alpine:3.10                                 | fabric-ca 镜像，安装了 fabric-ca 相关文件。生成过程使用 golang:1.13.4-alpine 镜像。 |
| fabric-baseos  | alpine:3.10                                 | 基础镜像，用来生成其它镜像（包括 Peer、Orderer、fabric-ca），并作为 Go 链码的默认运行时镜像。 |
| fabric-ccenv   | golang:1.13.4-alpine3.10                    | 支持 Go 语言的链码基础镜像，其中安装了 g++、gcc、git、musl-dev 等，并创建 chaincode 存放目录。在链码实例化过程中作为默认编译环境将链码编译为二进制文件。 |
| fabric-javaenv | adoptopenjdk/openjdk11:jdk-11.0.4_11-alpine | 支持 Java 语言的链码基础镜像，其中安装了 Gradle、Maven、Java 链码 shim 层等，作为 Java 链码的默认运行时镜像。 |
| fabric-nodeenv | node:12.13.0-alpine                         | 支持 Node.Js 语言的链码基础镜像，其中安装了 make、python、g++。在链码实例化过程中作为默认编译环境生成 Node.Js 链码镜像，同时作为 Node.Js 链码运行环境。 |

### **辅助镜像**

提供支持功能，目前包括 fabric-baseimage、fabric-tools 镜像。如下表所示。

| 镜像名称         | 父镜像                                   | 功能描述                                                     |
| ---------------- | ---------------------------------------- | ------------------------------------------------------------ |
| fabric-baseimage | adoptopenjdk:8u222-b10-jdk-openj9-0.15.1 | 基础镜像，安装了 wget、Golang、Node.JS、Python、Protocol buffer 支持等，用来生成其它镜像。作为运行时可以用来生成 Node.Js 链码镜像。 |
| fabric-tools     | golang:1.13.4-alpine                     | 安装了 bash、jq、peer、cryptogen、configtxgen 等常见命令，可以作为测试客户端使用。 |

### **第三方镜像**

主要是一些第三方开源软件，提供支持功能，目前包括 coudhdb、kafka、zookeeper 共 3 种镜像。如下表所示。

| 镜像名称         | 父镜像                                   | 功能描述                                                     |
| ---------------- | ---------------------------------------- | ------------------------------------------------------------ |
| fabric-couchdb   | debian:stretch-20190910-slim             | couchdb 镜像，可以启动 couchdb 服务，供 peer 使用。          |
| fabric-kafka     | adoptopenjdk:8u222-b10-jre-openj9-0.15.1 | kafka 镜像，可以启动 kafka 服务，供 orderer 在 kafka 模式下使用。已经不再支持。 |
| fabric-zookeeper | adoptopenjdk:8u222-b10-jre-openj9-0.15.1 | zookeeper 镜像，可以启动 zookeeper 服务，供 orderer 在 kafka 模式下使用。已经不再支持。 |

## 8.**源码生成Docker镜像**

可以通过如下命令在本地快速生成包括 fabric-baseos、fabric-peer、fabric-orderer、fabric-ccenv、fabric-tools 等多个 Docker 镜像：

```shell
$ make docker
```

注意，从源码直接生成的镜像，除了版本标签外，还会带有所编译版本快照信息的标签，例如 `amd64-2.0.0-snapshot123456`。



### DockerFile常用指令

```dockerfile
FROM			 指明构建的新镜像是来自于哪个基础镜像
MAINTAINER  	 指明镜像维护者及其联系方式（姓名+邮箱）
RUN				 制作镜像需要运行的命令，RUN后面接linux里的命令，这个命令是在制作镜像的时候使用的
WORKDIR			 镜像的工作目录，docker exec 命令执行的时候进入的目录
VOLUME			 向镜像创建的容器中添加数据卷，数据卷可以在容器之间共享和重用
EXPOSE			 指明容器运行的时候暴露的端口
ONBUILD			 当构建一个被继承DockerFile的时候，就会运行ONBUILD指令，触发指令
ENV				 设置镜像环境变量
ADD       		 可以从一个URL地址下载内容复制到容器文件系统中，还可以将压缩包文件解压并复制到指定位置
COPY			 COPY指令用来将本地的文件或者文件夹复制到镜像的指定路径下

# 文件复制均使用 COPY 指令,在需要自动解压缩的场合使用 ADD 指令
CMD				 指定这个容器启动的时候要运行的命令
				 Dockerfile只允许使用一次CMD指令，一般都是脚本中最后一条指令
ENTRYPOINT		 指定这个容器启动的时候要运行的命令，可以追加命令

# 如果docker run后面出现与CMD指定的相同的命令，那么CMD就会被覆盖。而ENTRYPOINT会把容器名后面的所有内容都当成参数传递给其指定的命令
CMD和ENTRYPOINT的区别
```



读者也可自行通过编写 Dockefile 的方式来生成相关镜像。其镜像文件在子目录：

```bash
cd ~/go/src/github.com/hyperledger/fabric/images
ls
```

截图：

![image-20230523001325671](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20230523001325671.png)

Dockerfile 中指令跟本地编译过程十分类似，这里给出 fabric-base 镜像、fabric-orderer 镜像和 fabric-peer 镜像等关键镜像的 Dockerfile，供读者参考使用。

### **fabric-base 镜像**

笔者提供的 fabric-base 镜像基于 golang:1.20.8 镜像生成，可以作为 Go 链码容器的基础镜像。该镜像中包含了 Fabric 相关的代码，并安装了一些有用的工具，包括 gotools、 configtxgen、configtxlator、cryptogen、discover、token、idemixgen 等。

该 Dockerfile 内容如下：

```dockerfile
#
# Copyright contributors to the Hyperledger Fabric project
#
# SPDX-License-Identifier: Apache-2.0
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at:
#
# 	  http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

ARG GO_VER
ARG UBUNTU_VER

FROM ubuntu:${UBUNTU_VER} as base

RUN sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list

RUN apt update && apt install -y tzdata

RUN addgroup --gid 500 chaincode
RUN adduser --disabled-password --gecos "" --uid 500 --gid 500 --home /home/chaincode chaincode

USER chaincode
```

该镜像也可以用作替代 `hyperledger/fabric-baseimage:latest` 镜像。



### **fabric-orderer 镜像**

fabric-orderer 镜像基于 fabric-base 生成，编译安装了 orderer 组件。

参考 Dockerfile 内容如下:

```dockerfile
#
# Copyright contributors to the Hyperledger Fabric project
#
# SPDX-License-Identifier: Apache-2.0
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at:
#
# 	  http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

###############################################################################
# Build image
###############################################################################

ARG UBUNTU_VER
FROM ubuntu:${UBUNTU_VER} as builder

RUN sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list

ARG TARGETARCH
ARG TARGETOS
ARG FABRIC_VER
ARG GO_VER
ARG GO_TAGS

RUN apt update && apt install -y \
    git \
    gcc \
    curl \
    make

RUN curl -sL https://golang.google.cn/dl/go${GO_VER}.${TARGETOS}-${TARGETARCH}.tar.gz | tar zxf - -C /usr/local
ENV PATH="/usr/local/go/bin:$PATH"

ADD . .

RUN make orderer GO_TAGS=${GO_TAGS} FABRIC_VER=${FABRIC_VER}


###############################################################################
# Runtime image
###############################################################################

ARG UBUNTU_VER
FROM ubuntu:${UBUNTU_VER}

RUN sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list

ARG FABRIC_VER

# set up nsswitch.conf for Go's "netgo" implementation
# - https://github.com/golang/go/blob/go1.9.1/src/net/conf.go#L194-L275
# - docker run --rm debian:stretch grep '^hosts:' /etc/nsswitch.conf
RUN echo 'hosts: files dns' > /etc/nsswitch.conf

ENV FABRIC_CFG_PATH /var/hyperledger/fabric/config
ENV FABRIC_VER      ${FABRIC_VER}

COPY --from=builder  build/bin/orderer           /usr/local/bin
COPY --from=builder  sampleconfig/msp            ${FABRIC_CFG_PATH}/msp
COPY --from=builder  sampleconfig/orderer.yaml   ${FABRIC_CFG_PATH}
COPY --from=builder  sampleconfig/configtx.yaml  ${FABRIC_CFG_PATH}

VOLUME /etc/hyperledger/fabric
VOLUME /var/hyperledger

EXPOSE 7050

CMD ["orderer", "start" ]
```

### **fabric-peer 镜像**

fabric-peer 镜像基于 fabric-base 生成，编译安装了 peer 命令。

Dockerfile 内容如下:

```dockerfile
#
# Copyright contributors to the Hyperledger Fabric project
#
# SPDX-License-Identifier: Apache-2.0
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at:
#
# 	  http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

###############################################################################
# Build image
###############################################################################

ARG UBUNTU_VER
FROM ubuntu:${UBUNTU_VER} as builder

RUN sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list

ARG TARGETARCH
ARG TARGETOS
ARG FABRIC_VER
ARG GO_VER
ARG GO_TAGS

RUN apt update && apt install -y \
    git \
    gcc \
    curl \
    make

RUN curl -sL https://golang.google.cn/dl/go${GO_VER}.${TARGETOS}-${TARGETARCH}.tar.gz | tar zxf - -C /usr/local
ENV PATH="/usr/local/go/bin:$PATH"

ADD . .

RUN make peer GO_TAGS=${GO_TAGS} FABRIC_VER=${FABRIC_VER}
RUN make ccaasbuilder


###############################################################################
# Runtime image
###############################################################################

ARG UBUNTU_VER
FROM ubuntu:${UBUNTU_VER}

RUN sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list

ARG TARGETOS
ARG TARGETARCH
ARG FABRIC_VER

# set up nsswitch.conf for Go's "netgo" implementation
# - https://github.com/golang/go/blob/go1.9.1/src/net/conf.go#L194-L275
# - docker run --rm debian:stretch grep '^hosts:' /etc/nsswitch.conf
RUN echo 'hosts: files dns' > /etc/nsswitch.conf

ENV FABRIC_CFG_PATH /var/hyperledger/fabric/config
ENV FABRIC_VER ${FABRIC_VER}

COPY --from=builder build/bin/peer /usr/local/bin
COPY --from=builder sampleconfig/msp ${FABRIC_CFG_PATH}/msp
COPY --from=builder sampleconfig/core.yaml ${FABRIC_CFG_PATH}/core.yaml

COPY --from=builder release/${TARGETOS}-${TARGETARCH}/builders/ccaas/bin /opt/hyperledger/ccaas_builder/bin

VOLUME /etc/hyperledger/fabric
VOLUME /var/hyperledger

EXPOSE  7051
CMD [ "peer","node","start" ]
```

编译信息：

```
Building Docker image hyperledger/fabric-peer
docker build --force-rm  -f images/peer/Dockerfile \
        --build-arg GO_VER=1.20.8 \
        --build-arg UBUNTU_VER=22.04 \
        --build-arg FABRIC_VER=2.5.3 \
        --build-arg TARGETARCH=amd64 \
        --build-arg TARGETOS=linux \
        --build-arg GO_TAGS= \
        -t hyperledger/fabric-peer ./
[+] Building 58.6s (12/16)                                                                                                                                                                docker:default
 => [internal] load build definition from Dockerfile                                                                                                                                                0.0s
 => => transferring dockerfile: 2.49kB                                                                                                                                                              0.0s
 => [internal] load .dockerignore                                                                                                                                                                   0.0s
 => => transferring context: 277B                                                                                                                                                                   0.0s
 => [internal] load metadata for docker.io/library/ubuntu:22.04                                                                                                                                    15.2s
 => [builder 1/7] FROM docker.io/library/ubuntu:22.04@sha256:f154feaf13b51d16e2b4b5575d69abc808da40c4b80e3a5055aaa4bcc5099d5b                                                                       0.0s
 => [internal] load build context                                                                                                                                                                   0.5s
 => => transferring context: 422.57kB                                                                                                                                                               0.5s
 => CACHED [builder 2/7] RUN sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list                                                                                               0.0s
 => CACHED [stage-1 3/7] RUN echo 'hosts: files dns' > /etc/nsswitch.conf                                                                                                                           0.0s
 => CACHED [builder 3/7] RUN apt update && apt install -y     git     gcc     curl     make                                                                                                         0.0s
 => CACHED [builder 4/7] RUN curl -sL https://golang.google.cn/dl/go1.20.8.linux-amd64.tar.gz | tar zxf - -C /usr/local                                                                             0.0s
 => CACHED [builder 5/7] ADD . .                                                                                                                                                                    0.0s
 => CACHED [builder 6/7] RUN make peer GO_TAGS= FABRIC_VER=2.5.3                                                                                                                                    0.0s
 => ERROR [builder 7/7] RUN make ccaasbuilder          
```



### **fabric-ccenv 镜像**

```dockerfile
#
# Copyright contributors to the Hyperledger Fabric project
#
# SPDX-License-Identifier: Apache-2.0
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at:
#
# 	  http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

ARG UBUNTU_VER
FROM ubuntu:${UBUNTU_VER}

RUN sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list

ARG TARGETARCH
ARG TARGETOS
ARG GO_VER

RUN apt update && apt install -y \
    binutils-gold \
    curl \
    g++ \
    gcc \
    git

RUN curl -sL https://golang.google.cn/dl/go${GO_VER}.${TARGETOS}-${TARGETARCH}.tar.gz | tar zxvf - -C /usr/local
ENV PATH="/usr/local/go/bin:$PATH"

RUN addgroup --gid 500 chaincode
RUN adduser  --disabled-password --gecos "" --uid 500 --gid 500 --home /home/chaincode chaincode

RUN mkdir    -p /chaincode/output /chaincode/input
RUN chown    -R chaincode:chaincode /chaincode

USER chaincode
```



### **fabric-tools镜像**

```dockerfile
#
# Copyright contributors to the Hyperledger Fabric project
#
# SPDX-License-Identifier: Apache-2.0
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at:
#
# 	  http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

###############################################################################
# Build image
###############################################################################

ARG UBUNTU_VER
FROM ubuntu:${UBUNTU_VER} as builder

RUN sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list

ARG TARGETARCH
ARG TARGETOS
ARG FABRIC_VER
ARG GO_VER
ARG GO_TAGS

RUN apt update && apt install -y \
    git \
    gcc \
    curl \
    make

RUN curl -sL https://golang.google.cn/dl/go${GO_VER}.${TARGETOS}-${TARGETARCH}.tar.gz | tar zxf - -C /usr/local
ENV PATH="/usr/local/go/bin:$PATH"

ADD . .

RUN make tools GO_TAGS=${GO_TAGS} FABRIC_VER=${FABRIC_VER}


###############################################################################
# Runtime image
###############################################################################

ARG UBUNTU_VER
FROM ubuntu:${UBUNTU_VER}

RUN sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list

ARG TARGETARCH
ARG TARGETOS
ARG FABRIC_VER
ARG GO_VER

RUN apt update && apt install -y \
    bash \
    curl \
    jq \
    tzdata

RUN curl -sL https://golang.google.cn/dl/go${GO_VER}.${TARGETOS}-${TARGETARCH}.tar.gz | tar zxvf - -C /usr/local
ENV PATH="/usr/local/go/bin:$PATH"

# set up nsswitch.conf for Go's "netgo" implementation
# - https://github.com/golang/go/blob/go1.9.1/src/net/conf.go#L194-L275
# - docker run --rm debian:stretch grep '^hosts:' /etc/nsswitch.conf
RUN echo 'hosts: files dns' > /etc/nsswitch.conf

ENV FABRIC_CFG_PATH /var/hyperledger/fabric/config
ENV FABRIC_VER      ${FABRIC_VER}

COPY --from=builder  sampleconfig    ${FABRIC_CFG_PATH}
COPY --from=builder  build/bin       /usr/local/bin

VOLUME  /etc/hyperledger/fabric
VOLUME  /var/hyperledger
```



### **fabric-ca 镜像**

fabric-ca 镜像基于 golang:1.20镜像生成，提供对证书的签发功能。

Dockerfile 内容如下:

```dockerfile
#
# Copyright IBM Corp. All Rights Reserved.
#
# SPDX-License-Identifier: Apache-2.0
#

###############################################################################
# Build image
###############################################################################

ARG UBUNTU_VER
FROM ubuntu:${UBUNTU_VER} as builder

RUN sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list

ARG TARGETARCH
ARG TARGETOS
ARG GO_VER
ARG GO_LDFLAGS
ARG GO_TAGS

RUN apt update && apt install -y \
    gcc \
    binutils-gold \
    git \
    curl \
    make

RUN curl -sL https://golang.google.cn/dl/go${GO_VER}.${TARGETOS}-${TARGETARCH}.tar.gz | tar zxf - -C /usr/local
ENV GOBIN="/usr/local/go/bin"
ENV PATH="$GOBIN:$PATH"

ADD . /build/fabric-ca
WORKDIR /build/fabric-ca

RUN go install \
    -tags "${GO_TAGS}" \
    -ldflags "${GO_LDFLAGS}" \
    github.com/hyperledger/fabric-ca/cmd/fabric-ca-server

RUN go install \
    -tags "${GO_TAGS}" \
    -ldflags "${GO_LDFLAGS}" \
    github.com/hyperledger/fabric-ca/cmd/fabric-ca-client


###############################################################################
# Runtime image
###############################################################################

ARG UBUNTU_VER
FROM ubuntu:${UBUNTU_VER}

RUN sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/g' /etc/apt/sources.list

RUN apt update
RUN DEBIAN_FRONTEND=noninteractive apt install -y tzdata

ENV FABRIC_CA_HOME /etc/hyperledger/fabric-ca-server
COPY --from=builder /usr/local/go/bin /usr/local/bin

EXPOSE 7054

CMD [ "fabric-ca-server", "start", "-b", "admin:adminpw" ]
```



## 9.***Dockerhub* 获取镜像**

除了从源码编译外，还可以直接从 Dockerhub 来拉取相关的镜像，命令格式为 `docker pull <IMAGE_NAME:TAG>`。

从社区仓库直接获取 fabric-peer、fabric-orderer、fabric-ca、fabric-tools 等镜像的版本可以使用如下命令：

```shell
$ PROJECT_VERSION=2.5

# 拉取镜像
$ docker pull hyperledger/fabric-peer:$PROJECT_VERSION
$ docker pull hyperledger/fabric-orderer:$PROJECT_VERSION

$ docker pull hyperledger/fabric-tools:$PROJECT_VERSION
$ docker pull hyperledger/fabric-ccenv:$PROJECT_VERSION
$ docker pull hyperledger/fabric-baseos:$PROJECT_VERSION

$ CA_VERSION=1.5.7
$ docker pull hyperledger/fabric-ca:$CA_VERSION
```









# *Hyperledger Fabric*测试网络（*test-network*）官方教程

下载 *Hyperledger Fabric Docker*镜像和*fabric-sample*后，您可以使用存储库中提供的脚本部署测试网络。提供测试网络以学习Fabric ，通过在本地计算机上运行节点。开发人员可以使用网络来测试他们的智能合约和应用程序。网络旨在仅用作教育和测试的工具，而不是如何设置的模型一个网络。通常，不鼓励修改脚本，否则可能会破坏网络。它基于不应用作部署生产网络的模板的有限配置：`fabric-samples`

- 它包括两个对等组织和一个排序组织。
- 为简单起见，配置了单节点 Raft 排序服务。
- 为了降低复杂性，未部署 TLS 证书颁发机构 （CA）。所有证书均由根 CA 颁发。
- 示例网络使用 Docker Compose 部署Fabric网络。因为 节点在 Docker 撰写网络中隔离，测试网络未配置为连接到其他正在运行的结构节点。

要了解如何在生产中使用 Fabric，请参阅[部署生产网络](https://hyperledger-fabric.readthedocs.io/en/release-2.5/deployment_guide_overview.html)。

**注意：**这些说明已经过验证，可针对 最新的稳定结构 Docker 映像和 提供的焦油文件。如果使用来自 当前主分支，您可能会遇到错误。



## 开始之前

在运行测试网络之前，您需要在 环境。按照[getting_started](https://hyperledger-fabric.readthedocs.io/en/release-2.5/getting_started.html)上的说明安装所需的软件。

**注意：**测试网络已成功通过 Docker 桌面版本 2.5.0.1 验证，是目前推荐的版本。更高版本可能无法正常工作。



## 启动测试网络

您可以在目录中找到用于启动网络的脚本 存储库。通过以下方式导航到测试网络目录 使用以下命令：`test-network``fabric-samples`

```bash
cd fabric-samples/test-network
```

在此目录中，您可以找到一个带注释的脚本 ，该脚本位于 使用本地计算机上的 Docker 映像连接到结构网络。您可以运行以打印脚本帮助文本：`network.sh``./network.sh -h`

```bash
Usage:
  network.sh <Mode> [Flags]
    Modes:
      up - Bring up Fabric orderer and peer nodes. No channel is created
      up createChannel - Bring up fabric network with one channel
      createChannel - Create and join a channel after the network is created
      deployCC - Deploy a chaincode to a channel (defaults to asset-transfer-basic)
      down - Bring down the network

    Flags:
    Used with network.sh up, network.sh createChannel:
    -ca <use CAs> -  Use Certificate Authorities to generate network crypto material
    -c <channel name> - Name of channel to create (defaults to "mychannel")
    -s <dbtype> - Peer state database to deploy: goleveldb (default) or couchdb
    -r <max retry> - CLI times out after certain number of attempts (defaults to 5)
    -d <delay> - CLI delays for a certain number of seconds (defaults to 3)
    -i <imagetag> - Docker image tag of Fabric to deploy (defaults to "latest")
    -cai <ca_imagetag> - Docker image tag of Fabric CA to deploy (defaults to "latest")
    -verbose - Verbose mode

    Used with network.sh deployCC
    -c <channel name> - Name of channel to deploy chaincode to
    -ccn <name> - Chaincode name.
    -ccl <language> - Programming language of the chaincode to deploy: go (default), java, javascript, typescript
    -ccv <version>  - Chaincode version. 1.0 (default), v2, version3.x, etc
    -ccs <sequence>  - Chaincode definition sequence. Must be an integer, 1 (default), 2, 3, etc
    -ccp <path>  - File path to the chaincode.
    -ccep <policy>  - (Optional) Chaincode endorsement policy using signature policy syntax. The default policy requires an endorsement from Org1 and Org2
    -cccg <collection-config>  - (Optional) File path to private data collections configuration file
    -cci <fcn name>  - (Optional) Name of chaincode initialization function. When a function is provided, the execution of init will be requested and the function will be invoked.

    -h - Print this message

 Possible Mode and flag combinations
   up -ca -r -d -s -i -cai -verbose
   up createChannel -ca -c -r -d -s -i -cai -verbose
   createChannel -c -r -d -verbose
   deployCC -ccn -ccl -ccv -ccs -ccp -cci -r -d -verbose

 Examples:
   network.sh up createChannel -ca -c mychannel -s couchdb -i 2.5.1
   network.sh createChannel -c channelName
   network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-javascript/ -ccl javascript
   network.sh deployCC -ccn mychaincode -ccp ./user/mychaincode -ccv 1 -ccl javascript
```

在目录中，运行以下命令以删除 以前运行的任何容器或项目：`test-network`

```bash
./network.sh down
```

然后，您可以通过发出以下命令来启动网络。你会 尝试从另一个目录运行脚本时遇到问题：

```bash
./network.sh up
```

此命令创建一个由两个对等节点组成的结构网络，一个 排序节点。运行时不会创建通道，尽管我们 将在[以后的步骤](https://hyperledger-fabric.readthedocs.io/en/release-2.5/test_network.html#creating-a-channel)中到达那里。如果命令完成 成功，您将看到正在创建的节点的日志：`./network.sh up`

```bash
Creating network "fabric_test" with the default driver
Creating volume "net_orderer.example.com" with default driver
Creating volume "net_peer0.org1.example.com" with default driver
Creating volume "net_peer0.org2.example.com" with default driver
Creating peer0.org2.example.com ... done
Creating orderer.example.com    ... done
Creating peer0.org1.example.com ... done
Creating cli                    ... done
CONTAINER ID   IMAGE                               COMMAND             CREATED         STATUS                  PORTS                                            NAMES
1667543b5634   hyperledger/fabric-tools:latest     "/bin/bash"         1 second ago    Up Less than a second                                                    cli
b6b117c81c7f   hyperledger/fabric-peer:latest      "peer node start"   2 seconds ago   Up 1 second             0.0.0.0:7051->7051/tcp                           peer0.org1.example.com
703ead770e05   hyperledger/fabric-orderer:latest   "orderer"           2 seconds ago   Up Less than a second   0.0.0.0:7050->7050/tcp, 0.0.0.0:7053->7053/tcp   orderer.example.com
718d43f5f312   hyperledger/fabric-peer:latest      "peer node start"   2 seconds ago   Up 1 second             7051/tcp, 0.0.0.0:9051->9051/tcp                 peer0.org2.example.com
```

如果未得到此结果，请跳转到[疑难解答](https://hyperledger-fabric.readthedocs.io/en/release-2.5/test_network.html#troubleshooting)，以获取有关可能出错的帮助。默认情况下，网络使用 加密工具，以启动网络。但是，您也可以[使用证书颁发机构启动网络](https://hyperledger-fabric.readthedocs.io/en/release-2.5/test_network.html#bring-up-the-network-with-certificate-authorities)。

运行详细信息：

```bash
magpie@DLTServer:~/go/src/github.com/hyperledger/fabric-samples/test-network$ ./network.sh up
Using docker and docker-compose
Starting nodes with CLI timeout of '5' tries and CLI delay of '3' seconds and using database 'leveldb' with crypto from 'cryptogen'
LOCAL_VERSION=2.5.3
DOCKER_IMAGE_VERSION=2.5.3
/home/magpie/go/bin/cryptogen
Generating certificates using cryptogen tool
Creating Org1 Identities
+ cryptogen generate --config=./organizations/cryptogen/crypto-config-org1.yaml --output=organizations
org1.example.com
+ res=0
Creating Org2 Identities
+ cryptogen generate --config=./organizations/cryptogen/crypto-config-org2.yaml --output=organizations
org2.example.com
+ res=0
Creating Orderer Org Identities
+ cryptogen generate --config=./organizations/cryptogen/crypto-config-orderer.yaml --output=organizations
+ res=0
Generating CCP files for Org1 and Org2
[+] Building 0.0s (0/0)                                                                                                                                                        
[+] Running 8/8
 ✔ Network fabric_test                      Created                                                                                                                       0.1s 
 ✔ Volume "compose_orderer.example.com"     Created                                                                                                                       0.0s 
 ✔ Volume "compose_peer0.org1.example.com"  Created                                                                                                                       0.0s 
 ✔ Volume "compose_peer0.org2.example.com"  Created                                                                                                                       0.0s 
 ✔ Container peer0.org2.example.com         Started                                                                                                                       1.5s 
 ✔ Container orderer.example.com            Started                                                                                                                       2.4s 
 ✔ Container peer0.org1.example.com         Started                                                                                                                       1.8s 
 ✔ Container cli                            Started                                                                                                                       2.7s 
CONTAINER ID   IMAGE                               COMMAND             CREATED         STATUS                  PORTS                                                                                                                             NAMES
aa83a606079f   hyperledger/fabric-tools:latest     "/bin/bash"         3 seconds ago   Up Less than a second                                                                                                                                     cli
77af6dafa964   hyperledger/fabric-peer:latest      "peer node start"   3 seconds ago   Up 1 second             0.0.0.0:9051->9051/tcp, :::9051->9051/tcp, 7051/tcp, 0.0.0.0:9445->9445/tcp, :::9445->9445/tcp                                    peer0.org2.example.com
5fb370f11f38   hyperledger/fabric-peer:latest      "peer node start"   3 seconds ago   Up 1 second             0.0.0.0:7051->7051/tcp, :::7051->7051/tcp, 0.0.0.0:9444->9444/tcp, :::9444->9444/tcp                                              peer0.org1.example.com
5423ceee762e   hyperledger/fabric-orderer:latest   "orderer"           3 seconds ago   Up Less than a second   0.0.0.0:7050->7050/tcp, :::7050->7050/tcp, 0.0.0.0:7053->7053/tcp, :::7053->7053/tcp, 0.0.0.0:9443->9443/tcp, :::9443->9443/tcp   orderer.example.com
10be4da9b6a0   portainer/portainer                 "/portainer"        7 hours ago     Up 7 hours              0.0.0.0:9000->9000/tcp, :::9000->9000/tcp                                                                                         jovial_chebyshev
magpie@DLTServer:~/go/src/github.com/hyperledger/fabric-samples/test-network$ 
```

![image-20230917182049449](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20230917182049449.png)





## 测试网络的组成部分

部署测试网络后，可以花一些时间来检查其 组件。运行以下命令以列出所有 Docker 容器 正在您的计算机上运行。您应该看到由 脚本：`network.sh`

```bash
docker ps -a
```

信息：

![image-20230917182244907](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20230917182244907.png)



与结构网络交互的每个节点和用户都需要属于 组织，以便参与网络。测试内容 网络包括两个对等组织，即组织 1 和组织 2。它还包括一个 维护网络排序服务的排序者组织。

[对等体](https://hyperledger-fabric.readthedocs.io/en/release-2.5/peers/peers.html)是任何结构网络的基本组件。 对等节点存储区块链分类账并在交易之前验证交易 提交到账本。对等方运行包含业务的智能合约 用于管理区块链账本上的资产的逻辑。

网络中的每个对等方都需要属于一个组织。在 测试网络，每个组织各自运行一个对等点，以及 .`peer0.org1.example.com``peer0.org2.example.com`

每个结构网络还包括一个[订购服务](https://hyperledger-fabric.readthedocs.io/en/release-2.5/orderer/ordering_service.html)。 当对等方验证事务并将事务块添加到 区块链分类账，他们不决定交易顺序或包括 他们进入新的块。在分布式网络上，对等方可能跑得很远 彼此之间，并且对事务的创建时间没有共同的视图。 就交易顺序达成共识是一个代价高昂的过程，它将 为对等方创建开销。

排序服务允许对等方专注于验证交易和 将它们提交到账本。排序节点收到背书交易后 从客户那里，他们就交易顺序达成共识，然后添加 他们到块。然后将块分发到对等节点，这些节点添加 区块链分类账的块。

示例网络使用由 排序者组织。您可以看到计算机上运行的排序节点 如。而测试网络仅使用单节点排序 服务，一个生产网络将有多个排序节点，由一个或 多个订购者组织。不同的排序节点将使用 Raft 共识算法就交易顺序达成一致 网络。`orderer.example.com`



## 创建通道

现在我们的机器上运行了对等节点和排序节点，我们可以使用 脚本，用于为 Org1 和 Org2 之间的事务创建结构通道。 通道是特定网络成员之间的专用通信层。 频道只能由受邀加入频道的组织使用，并且 对网络的其他成员不可见。每个通道都有一个单独的 区块链分类账。受邀“加入”其同行的组织 用于存储通道账本并验证交易的通道 渠道。

您可以使用该脚本在 Org1 和 Org2 之间创建通道 并将他们的同行加入频道。运行以下命令以创建一个 默认名称为 mychannel的通道：

```bash
./network.sh createChannel
```

截图：

![image-20230917190150327](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20230917190150327.png)

详细信息：

```bash
magpie@DLTServer:~/go/src/github.com/hyperledger/fabric-samples/test-network$ ./network.sh createChannel
Using docker and docker-compose
Creating channel 'mychannel'.
If network is not up, starting nodes with CLI timeout of '5' tries and CLI delay of '3' seconds and using database 'leveldb 
Network Running Already
Using docker and docker-compose
Generating channel genesis block 'mychannel.block'
Using organization 1
/home/magpie/go/bin/configtxgen
+ '[' 0 -eq 1 ']'
+ configtxgen -profile ChannelUsingRaft -outputBlock ./channel-artifacts/mychannel.block -channelID mychannel
2023-09-17 10:38:48.694 UTC 0001 INFO [common.tools.configtxgen] main -> Loading configuration
2023-09-17 10:38:48.722 UTC 0002 INFO [common.tools.configtxgen.localconfig] completeInitialization -> orderer type: etcdraft
2023-09-17 10:38:48.722 UTC 0003 INFO [common.tools.configtxgen.localconfig] completeInitialization -> Orderer.EtcdRaft.Options unset, setting to tick_interval:"500ms" election_tick:10 heartbeat_tick:1 max_inflight_blocks:5 snapshot_interval_size:16777216 
2023-09-17 10:38:48.722 UTC 0004 INFO [common.tools.configtxgen.localconfig] Load -> Loaded configuration: /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/configtx/configtx.yaml
2023-09-17 10:38:48.742 UTC 0005 INFO [common.tools.configtxgen] doOutputBlock -> Generating genesis block
2023-09-17 10:38:48.742 UTC 0006 INFO [common.tools.configtxgen] doOutputBlock -> Creating application channel genesis block
2023-09-17 10:38:48.749 UTC 0007 INFO [common.tools.configtxgen] doOutputBlock -> Writing genesis block
+ res=0
Creating channel mychannel
Adding orderers
+ . scripts/orderer.sh mychannel
+ '[' 0 -eq 1 ']'
+ res=0
Status: 201
{
        "name": "mychannel",
        "url": "/participation/v1/channels/mychannel",
        "consensusRelation": "consenter",
        "status": "active",
        "height": 1
}

Channel 'mychannel' created
Joining org1 peer to the channel...
Using organization 1
+ peer channel join -b ./channel-artifacts/mychannel.block
+ res=0
2023-09-17 10:38:55.043 UTC 0001 INFO [channelCmd] InitCmdFactory -> Endorser and orderer connections initialized
2023-09-17 10:38:55.123 UTC 0002 INFO [channelCmd] executeJoin -> Successfully submitted proposal to join channel
Joining org2 peer to the channel...
Using organization 2
+ peer channel join -b ./channel-artifacts/mychannel.block
+ res=0
2023-09-17 10:38:58.233 UTC 0001 INFO [channelCmd] InitCmdFactory -> Endorser and orderer connections initialized
2023-09-17 10:38:58.291 UTC 0002 INFO [channelCmd] executeJoin -> Successfully submitted proposal to join channel
Setting anchor peer for org1...
Using organization 1
Fetching channel config for channel mychannel
Using organization 1
Fetching the most recent configuration block for the channel
+ peer channel fetch config config_block.pb -o orderer.example.com:7050 --ordererTLSHostnameOverride orderer.example.com -c mychannel --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/organizations/ordererOrganizations/example.com/tlsca/tlsca.example.com-cert.pem
2023-09-17 10:38:58.619 UTC 0001 INFO [channelCmd] InitCmdFactory -> Endorser and orderer connections initialized
2023-09-17 10:38:58.626 UTC 0002 INFO [cli.common] readBlock -> Received block: 0
2023-09-17 10:38:58.626 UTC 0003 INFO [channelCmd] fetch -> Retrieving last config block: 0
2023-09-17 10:38:58.629 UTC 0004 INFO [cli.common] readBlock -> Received block: 0
+ configtxlator proto_decode --input config_block.pb --type common.Block --output config_block.json
Decoding config block to JSON and isolating config to Org1MSPconfig.json
+ jq '.data.data[0].payload.data.config' config_block.json
Generating anchor peer update transaction for Org1 on channel mychannel
+ jq '.channel_group.groups.Application.groups.Org1MSP.values += {"AnchorPeers":{"mod_policy": "Admins","value":{"anchor_peers": [{"host": "peer0.org1.example.com","port": 7051}]},"version": "0"}}' Org1MSPconfig.json
+ configtxlator proto_encode --input Org1MSPconfig.json --type common.Config --output original_config.pb
+ configtxlator proto_encode --input Org1MSPmodified_config.json --type common.Config --output modified_config.pb
+ configtxlator compute_update --channel_id mychannel --original original_config.pb --updated modified_config.pb --output config_update.pb
+ configtxlator proto_decode --input config_update.pb --type common.ConfigUpdate --output config_update.json
+ jq .
++ cat config_update.json
+ echo '{"payload":{"header":{"channel_header":{"channel_id":"mychannel", "type":2}},"data":{"config_update":{' '"channel_id":' '"mychannel",' '"isolated_data":' '{},' '"read_set":' '{' '"groups":' '{' '"Application":' '{' '"groups":' '{' '"Org1MSP":' '{' '"groups":' '{},' '"mod_policy":' '"",' '"policies":' '{' '"Admins":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Endorsement":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Readers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Writers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '}' '},' '"values":' '{' '"MSP":' '{' '"mod_policy":' '"",' '"value":' null, '"version":' '"0"' '}' '},' '"version":' '"0"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '},' '"write_set":' '{' '"groups":' '{' '"Application":' '{' '"groups":' '{' '"Org1MSP":' '{' '"groups":' '{},' '"mod_policy":' '"Admins",' '"policies":' '{' '"Admins":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Endorsement":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Readers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Writers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '}' '},' '"values":' '{' '"AnchorPeers":' '{' '"mod_policy":' '"Admins",' '"value":' '{' '"anchor_peers":' '[' '{' '"host":' '"peer0.org1.example.com",' '"port":' 7051 '}' ']' '},' '"version":' '"0"' '},' '"MSP":' '{' '"mod_policy":' '"",' '"value":' null, '"version":' '"0"' '}' '},' '"version":' '"1"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '}' '}}}}'
+ configtxlator proto_encode --input config_update_in_envelope.json --type common.Envelope --output Org1MSPanchors.tx
2023-09-17 10:38:59.353 UTC 0001 INFO [channelCmd] InitCmdFactory -> Endorser and orderer connections initialized
2023-09-17 10:38:59.386 UTC 0002 INFO [channelCmd] update -> Successfully submitted channel update
Anchor peer set for org 'Org1MSP' on channel 'mychannel'
Setting anchor peer for org2...
Using organization 2
Fetching channel config for channel mychannel
Using organization 2
Fetching the most recent configuration block for the channel
+ peer channel fetch config config_block.pb -o orderer.example.com:7050 --ordererTLSHostnameOverride orderer.example.com -c mychannel --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/organizations/ordererOrganizations/example.com/tlsca/tlsca.example.com-cert.pem
2023-09-17 10:38:59.708 UTC 0001 INFO [channelCmd] InitCmdFactory -> Endorser and orderer connections initialized
2023-09-17 10:38:59.716 UTC 0002 INFO [cli.common] readBlock -> Received block: 1
2023-09-17 10:38:59.717 UTC 0003 INFO [channelCmd] fetch -> Retrieving last config block: 1
2023-09-17 10:38:59.720 UTC 0004 INFO [cli.common] readBlock -> Received block: 1
Decoding config block to JSON and isolating config to Org2MSPconfig.json
+ configtxlator proto_decode --input config_block.pb --type common.Block --output config_block.json
+ jq '.data.data[0].payload.data.config' config_block.json
+ jq '.channel_group.groups.Application.groups.Org2MSP.values += {"AnchorPeers":{"mod_policy": "Admins","value":{"anchor_peers": [{"host": "peer0.org2.example.com","port": 9051}]},"version": "0"}}' Org2MSPconfig.json
Generating anchor peer update transaction for Org2 on channel mychannel
+ configtxlator proto_encode --input Org2MSPconfig.json --type common.Config --output original_config.pb
+ configtxlator proto_encode --input Org2MSPmodified_config.json --type common.Config --output modified_config.pb
+ configtxlator compute_update --channel_id mychannel --original original_config.pb --updated modified_config.pb --output config_update.pb
+ configtxlator proto_decode --input config_update.pb --type common.ConfigUpdate --output config_update.json
+ jq .
++ cat config_update.json
+ echo '{"payload":{"header":{"channel_header":{"channel_id":"mychannel", "type":2}},"data":{"config_update":{' '"channel_id":' '"mychannel",' '"isolated_data":' '{},' '"read_set":' '{' '"groups":' '{' '"Application":' '{' '"groups":' '{' '"Org2MSP":' '{' '"groups":' '{},' '"mod_policy":' '"",' '"policies":' '{' '"Admins":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Endorsement":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Readers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Writers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '}' '},' '"values":' '{' '"MSP":' '{' '"mod_policy":' '"",' '"value":' null, '"version":' '"0"' '}' '},' '"version":' '"0"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '},' '"write_set":' '{' '"groups":' '{' '"Application":' '{' '"groups":' '{' '"Org2MSP":' '{' '"groups":' '{},' '"mod_policy":' '"Admins",' '"policies":' '{' '"Admins":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Endorsement":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Readers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Writers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '}' '},' '"values":' '{' '"AnchorPeers":' '{' '"mod_policy":' '"Admins",' '"value":' '{' '"anchor_peers":' '[' '{' '"host":' '"peer0.org2.example.com",' '"port":' 9051 '}' ']' '},' '"version":' '"0"' '},' '"MSP":' '{' '"mod_policy":' '"",' '"value":' null, '"version":' '"0"' '}' '},' '"version":' '"1"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '}' '}}}}'
+ configtxlator proto_encode --input config_update_in_envelope.json --type common.Envelope --output Org2MSPanchors.tx
2023-09-17 10:39:00.504 UTC 0001 INFO [channelCmd] InitCmdFactory -> Endorser and orderer connections initialized
2023-09-17 10:39:00.551 UTC 0002 INFO [channelCmd] update -> Successfully submitted channel update
Anchor peer set for org 'Org2MSP' on channel 'mychannel'
Channel 'mychannel' joined
magpie@DLTServer:~/go/src/github.com/hyperledger/fabric-samples/test-network$ 
```

如果命令成功，您可以在 末尾显示：

```bash
Channel 'mychannel' joined
```

![image-20230917190333613](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20230917190333613.png)



您还可以使用通道标志创建具有自定义名称的通道。作为一个 例如，以下命令将创建一个名为 的通道：`channel1`

```bash
./network.sh createChannel -c channel1
```

通道标志还允许您通过指定来创建多个通道 不同的频道名称。创建 或 后，可以使用 以下命令创建名为 的第二个通道：`mychannel``channel1``channel2`

```bash
./network.sh createChannel -c channel2
```

**注意：**确保通道的名称应用以下限制：

- 仅包含小写 ASCII 字母数字、点 '.） 和短划线 '-'
- 少于 250 个字符
- 以字母开头

如果您想一步启动网络并创建通道，您可以 可以同时使用 和 模式：`up``createChannel`

```bash
./network.sh up createChannel
```



## 在通道上启动链码

创建通道后，您可以开始使用[智能合约](https://hyperledger-fabric.readthedocs.io/en/release-2.5/smartcontract/smartcontract.html)来与通道账本交互。智能合约包含业务逻辑管理区块链分类账上的资产。由 成员运行的应用程序 网络可以调用智能合约在分类账上创建资产，以及 更改和转移这些资产。应用程序还查询智能合约以 读取账本上的数据。

为了确保交易有效，使用智能合约创建的交易 通常需要由多个组织签名才能提交到 通道分类账。多重签名是 Fabric 信任模型不可或缺的一部分。 要求对一个事务进行多次背书会阻止一个组织 篡改其对等节点上的账本或使用业务逻辑的通道 这没有得到同意。要签署交易，每个组织都需要调用 并在他们的对等节点上执行智能合约，然后签署 交易。如果输出一致并且已由足够多的人签名 组织，交易可以提交到账本。该政策 指定通道上需要执行智能的设置组织 合同称为背书策略，为每个 链码作为链码定义的一部分。

在 Fabric 中，智能合约以称为 作为链码。链码安装在组织的对等节点上，然后 部署到通道，然后可用于背书交易和 与区块链分类账交互。在链码可以部署到 通道，通道的成员需要就链码定义达成一致 建立链码治理。当所需数量的组织 同意，链码定义可以提交到通道，并且 链码已准备就绪，可供使用。

使用 创建频道后，您可以启动 使用以下命令在通道上链码：`network.sh`

```bash
$ bash network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go -ccl go
```

运行详细信息：

```bash
magpie@DLTServer:~/go/src/github.com/hyperledger/fabric-samples/test-network$  bash network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go -ccl go
Using docker and docker-compose
deploying chaincode on channel 'mychannel'
executing with the following
- CHANNEL_NAME: mychannel
- CC_NAME: basic
- CC_SRC_PATH: ../asset-transfer-basic/chaincode-go
- CC_SRC_LANGUAGE: go
- CC_VERSION: 1.0
- CC_SEQUENCE: 1
- CC_END_POLICY: NA
- CC_COLL_CONFIG: NA
- CC_INIT_FCN: NA
- DELAY: 3
- MAX_RETRY: 5
- VERBOSE: false
Vendoring Go dependencies at ../asset-transfer-basic/chaincode-go
~/go/src/github.com/hyperledger/fabric-samples/asset-transfer-basic/chaincode-go ~/go/src/github.com/hyperledger/fabric-samples/test-network
go: downloading github.com/hyperledger/fabric-chaincode-go v0.0.0-20230228194215-b84622ba6a7a
go: downloading github.com/hyperledger/fabric-contract-api-go v1.2.1
go: downloading github.com/stretchr/testify v1.8.2
go: downloading github.com/xeipuuv/gojsonschema v1.2.0
go: downloading github.com/go-openapi/spec v0.20.8
go: downloading github.com/gobuffalo/packr v1.30.1
go: downloading github.com/gobuffalo/envy v1.10.1
go: downloading github.com/gobuffalo/packd v1.0.1
go: downloading github.com/xeipuuv/gojsonreference v0.0.0-20180127040603-bd5ef7bd5415
go: downloading github.com/joho/godotenv v1.4.0
go: downloading github.com/xeipuuv/gojsonpointer v0.0.0-20190905194746-02993c407bfb
go: downloading github.com/rogpeppe/go-internal v1.8.1
go: downloading github.com/go-openapi/jsonpointer v0.19.5
go: downloading github.com/go-openapi/jsonreference v0.20.0
go: downloading github.com/go-openapi/swag v0.21.1
go: downloading github.com/mailru/easyjson v0.7.7
go: downloading github.com/josharian/intern v1.0.0
~/go/src/github.com/hyperledger/fabric-samples/test-network
Finished vendoring Go dependencies
+ peer lifecycle chaincode package basic.tar.gz --path ../asset-transfer-basic/chaincode-go --lang golang --label basic_1.0
+ res=0
++ peer lifecycle chaincode calculatepackageid basic.tar.gz
+ PACKAGE_ID=basic_1.0:e4de097efb5be42d96aebc4bde18eea848aad0f5453453ba2aad97f2e41e0d57
Chaincode is packaged
Installing chaincode on peer0.org1...
Using organization 1
+ peer lifecycle chaincode queryinstalled --output json
+ jq -r 'try (.installed_chaincodes[].package_id)'
+ grep '^basic_1.0:e4de097efb5be42d96aebc4bde18eea848aad0f5453453ba2aad97f2e41e0d57$'
+ test 1 -ne 0
+ peer lifecycle chaincode install basic.tar.gz
+ res=0
2023-09-17 11:14:07.963 UTC 0001 INFO [cli.lifecycle.chaincode] submitInstallProposal -> Installed remotely: response:<status:200 payload:"\nJbasic_1.0:e4de097efb5be42d96aebc4bde18eea848aad0f5453453ba2aad97f2e41e0d57\022\tbasic_1.0" > 
2023-09-17 11:14:07.996 UTC 0002 INFO [cli.lifecycle.chaincode] submitInstallProposal -> Chaincode code package identifier: basic_1.0:e4de097efb5be42d96aebc4bde18eea848aad0f5453453ba2aad97f2e41e0d57
Chaincode is installed on peer0.org1
Install chaincode on peer0.org2...
Using organization 2
+ peer lifecycle chaincode queryinstalled --output json
+ jq -r 'try (.installed_chaincodes[].package_id)'
+ grep '^basic_1.0:e4de097efb5be42d96aebc4bde18eea848aad0f5453453ba2aad97f2e41e0d57$'
+ test 1 -ne 0
+ peer lifecycle chaincode install basic.tar.gz
+ res=0
2023-09-17 11:15:08.190 UTC 0001 INFO [cli.lifecycle.chaincode] submitInstallProposal -> Installed remotely: response:<status:200 payload:"\nJbasic_1.0:e4de097efb5be42d96aebc4bde18eea848aad0f5453453ba2aad97f2e41e0d57\022\tbasic_1.0" > 
2023-09-17 11:15:08.191 UTC 0002 INFO [cli.lifecycle.chaincode] submitInstallProposal -> Chaincode code package identifier: basic_1.0:e4de097efb5be42d96aebc4bde18eea848aad0f5453453ba2aad97f2e41e0d57
Chaincode is installed on peer0.org2
Using organization 1
+ peer lifecycle chaincode queryinstalled --output json
+ jq -r 'try (.installed_chaincodes[].package_id)'
+ grep '^basic_1.0:e4de097efb5be42d96aebc4bde18eea848aad0f5453453ba2aad97f2e41e0d57$'
+ res=0
basic_1.0:e4de097efb5be42d96aebc4bde18eea848aad0f5453453ba2aad97f2e41e0d57
Query installed successful on peer0.org1 on channel
Using organization 1
+ peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/tlsca/tlsca.example.com-cert.pem --channelID mychannel --name basic --version 1.0 --package-id basic_1.0:e4de097efb5be42d96aebc4bde18eea848aad0f5453453ba2aad97f2e41e0d57 --sequence 1
+ res=0
2023-09-17 11:15:11.130 UTC 0001 INFO [chaincodeCmd] ClientWait -> txid [0a675535b907cd37137e66ca119352527cc840dc8b183c190f80b05b78aacee9] committed with status (VALID) at localhost:7051
Chaincode definition approved on peer0.org1 on channel 'mychannel'
Using organization 1
Checking the commit readiness of the chaincode definition on peer0.org1 on channel 'mychannel'...
Attempting to check the commit readiness of the chaincode definition on peer0.org1, Retry after 3 seconds.
+ peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name basic --version 1.0 --sequence 1 --output json
+ res=0
{
        "approvals": {
                "Org1MSP": true,
                "Org2MSP": false
        }
}
Checking the commit readiness of the chaincode definition successful on peer0.org1 on channel 'mychannel'
Using organization 2
Checking the commit readiness of the chaincode definition on peer0.org2 on channel 'mychannel'...
Attempting to check the commit readiness of the chaincode definition on peer0.org2, Retry after 3 seconds.
+ peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name basic --version 1.0 --sequence 1 --output json
+ res=0
{
        "approvals": {
                "Org1MSP": true,
                "Org2MSP": false
        }
}
Checking the commit readiness of the chaincode definition successful on peer0.org2 on channel 'mychannel'
Using organization 2
+ peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/tlsca/tlsca.example.com-cert.pem --channelID mychannel --name basic --version 1.0 --package-id basic_1.0:e4de097efb5be42d96aebc4bde18eea848aad0f5453453ba2aad97f2e41e0d57 --sequence 1
+ res=0
2023-09-17 11:15:19.531 UTC 0001 INFO [chaincodeCmd] ClientWait -> txid [eac0a88cf32a339d0768559a04fd356e0d3c2a92fefc81706bcaaf912f967f5f] committed with status (VALID) at localhost:9051
Chaincode definition approved on peer0.org2 on channel 'mychannel'
Using organization 1
Checking the commit readiness of the chaincode definition on peer0.org1 on channel 'mychannel'...
Attempting to check the commit readiness of the chaincode definition on peer0.org1, Retry after 3 seconds.
+ peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name basic --version 1.0 --sequence 1 --output json
+ res=0
{
        "approvals": {
                "Org1MSP": true,
                "Org2MSP": true
        }
}
Checking the commit readiness of the chaincode definition successful on peer0.org1 on channel 'mychannel'
Using organization 2
Checking the commit readiness of the chaincode definition on peer0.org2 on channel 'mychannel'...
Attempting to check the commit readiness of the chaincode definition on peer0.org2, Retry after 3 seconds.
+ peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name basic --version 1.0 --sequence 1 --output json
+ res=0
{
        "approvals": {
                "Org1MSP": true,
                "Org2MSP": true
        }
}
Checking the commit readiness of the chaincode definition successful on peer0.org2 on channel 'mychannel'
Using organization 1
Using organization 2
+ peer lifecycle chaincode commit -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/tlsca/tlsca.example.com-cert.pem --channelID mychannel --name basic --peerAddresses localhost:7051 --tlsRootCertFiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/tlsca/tlsca.org1.example.com-cert.pem --peerAddresses localhost:9051 --tlsRootCertFiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/tlsca/tlsca.org2.example.com-cert.pem --version 1.0 --sequence 1
+ res=0
2023-09-17 11:15:28.341 UTC 0001 INFO [chaincodeCmd] ClientWait -> txid [27a608d3244e8de56319be5691e994058e353c71836fe7925a81632c4df293e4] committed with status (VALID) at localhost:7051
2023-09-17 11:15:28.389 UTC 0002 INFO [chaincodeCmd] ClientWait -> txid [27a608d3244e8de56319be5691e994058e353c71836fe7925a81632c4df293e4] committed with status (VALID) at localhost:9051
Chaincode definition committed on channel 'mychannel'
Using organization 1
Querying chaincode definition on peer0.org1 on channel 'mychannel'...
Attempting to Query committed status on peer0.org1, Retry after 3 seconds.
+ peer lifecycle chaincode querycommitted --channelID mychannel --name basic
+ res=0
Committed chaincode definition for chaincode 'basic' on channel 'mychannel':
Version: 1.0, Sequence: 1, Endorsement Plugin: escc, Validation Plugin: vscc, Approvals: [Org1MSP: true, Org2MSP: true]
Query chaincode definition successful on peer0.org1 on channel 'mychannel'
Using organization 2
Querying chaincode definition on peer0.org2 on channel 'mychannel'...
Attempting to Query committed status on peer0.org2, Retry after 3 seconds.
+ peer lifecycle chaincode querycommitted --channelID mychannel --name basic
+ res=0
Committed chaincode definition for chaincode 'basic' on channel 'mychannel':
Version: 1.0, Sequence: 1, Endorsement Plugin: escc, Validation Plugin: vscc, Approvals: [Org1MSP: true, Org2MSP: true]
Query chaincode definition successful on peer0.org2 on channel 'mychannel'
Chaincode initialization is not required
magpie@DLTServer:~/go/src/github.com/hyperledger/fabric-samples/test-network$ 
```

截图：

![image-20230917191251947](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20230917191251947.png)

子命令将安装**转移（基本）**链码，然后部署 使用通道标志指定的通道上的链码（如果未指定通道）。如果您要为第一个部署链码 时间，脚本将安装链码依赖项。您可以使用 语言标志，，以安装链码的 Go、打字稿或 JavaScript 版本。 您可以在目录的文件夹中找到资产转移（基本）链码。此文件夹包含作为示例提供的示例链码和 教程用于突出显示结构功能。`deployCC``peer0.org1.example.com``peer0.org2.example.com``mychannel``-ccl``asset-transfer-basic``fabric-samples`



## 与网络交互

启动测试网络后，可以使用 CLI 进行交互 与您的网络。CLI 允许您调用已部署的智能合约， 更新通道，或从 CLI 安装和部署新的智能合约。`peer``peer`

确保您正在从目录中进行操作。如果你 按照说明安装[示例、二进制文件和 Docker 映像](https://hyperledger-fabric.readthedocs.io/en/release-2.5/install.html)， 您可以在存储库的文件夹中找到二进制文件。使用以下命令将这些二进制文件添加到 CLI 路径：`test-network``peer``bin``fabric-samples`

```bash
export PATH=${PWD}/../bin:$PATH
```

您还需要将 设置为 指向 中的文件 存储库：`FABRIC_CFG_PATH``core.yaml``fabric-samples`

```bash
export FABRIC_CFG_PATH=$PWD/../config/
```

现在，您可以设置允许您将 CLI 作为 Org1 运行的环境变量：`peer`

```bash
# Environment variables for Org1

export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
```

和环境 变量指向文件夹中的 Org1 加密材料。`CORE_PEER_TLS_ROOTCERT_FILE``CORE_PEER_MSPCONFIGPATH``organizations`

![image-20230917204341081](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20230917204341081.png)

如果您曾经安装并启动资产转移（基本）链码，则可以调用 （Go） 链码的函数将初始资产列表放在账本上（例如，如果使用 TypeScript 或 JavaScript，您将调用相应链码的功能）。`./network.sh deployCC -ccl go``InitLedger``./network.sh deployCC -ccl javascript``InitLedger`

运行以下命令以使用资产初始化账本。（请注意，CLI 不访问结构网关对等方，因此必须指定每个认可对等方。

```bash
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" --peerAddresses localhost:9051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt" -c '{"function":"InitLedger","Args":[]}'
```

如果成功，应会看到类似于以下示例的输出：

```
-> INFO 001 Chaincode invoke successful. result: status:200
```

![image-20230917204412358](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20230917204412358.png)

现在，您可以从 CLI 查询账本。运行以下命令以获取已添加到通道账本的资产列表：

```bash
peer chaincode query -C mychannel -n basic -c '{"Args":["GetAllAssets"]}'
```

如果成功，应会看到以下输出：

```bash
[
  {"ID": "asset1", "color": "blue", "size": 5, "owner": "Tomoko", "appraisedValue": 300},
  {"ID": "asset2", "color": "red", "size": 5, "owner": "Brad", "appraisedValue": 400},
  {"ID": "asset3", "color": "green", "size": 10, "owner": "Jin Soo", "appraisedValue": 500},
  {"ID": "asset4", "color": "yellow", "size": 10, "owner": "Max", "appraisedValue": 600},
  {"ID": "asset5", "color": "black", "size": 15, "owner": "Adriana", "appraisedValue": 700},
  {"ID": "asset6", "color": "white", "size": 15, "owner": "Michel", "appraisedValue": 800}
]
```

![image-20230917204459666](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20230917204459666.png)

当网络成员想要转移或更改账本上的资产时，会调用链码。使用以下命令通过调用资产转移（基本）链码来更改账本上资产的所有者：

```bash
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" --peerAddresses localhost:9051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt" -c '{"function":"TransferAsset","Args":["asset6","Christopher"]}'
```

如果命令成功，应会看到以下响应：

![image-20230917204550949](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20230917204550949.png)

```
2019-12-04 17:38:21.048 EST [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 001 Chaincode invoke successful. result: status:200
```

因为资产转移（基本）链码的背书策略需要交易 要由 Org1 和 Org2 签名，链码调用命令需要针对两者并使用标志。由于为网络启用了 TLS，因此该命令还需要引用 使用该标志的每个对等方的 TLS 证书。`peer0.org1.example.com``peer0.org2.example.com``--peerAddresses``--tlsRootCertFiles`

在我们调用链码之后，我们可以使用另一个查询来查看调用如何 更改了区块链分类账上的资产。由于我们已经查询了 Org1 peer，我们可以借此机会查询 Org2 上运行的链码 同辈。将以下环境变量设置为作为 Org2 运行：

```bash
# Environment variables for Org2

export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051
```

![image-20230917204624496](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20230917204624496.png)

您现在可以查询在以下设备上运行的资产转移（基本）链码：`peer0.org2.example.com`

```bash
peer chaincode query -C mychannel -n basic -c '{"Args":["ReadAsset","asset6"]}'
```

结果将显示已转移到克里斯托弗：`"asset6"`

```bash
{"ID":"asset6","color":"white","size":15,"owner":"Christopher","appraisedValue":800}
```

![image-20230917204647033](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20230917204647033.png)

以上所有详细信息：

```bash
magpie@DLTServer:~/go/src/github.com/hyperledger/fabric-samples/test-network$ export FABRIC_CFG_PATH=$PWD/../config/
magpie@DLTServer:~/go/src/github.com/hyperledger/fabric-samples/test-network$ export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
magpie@DLTServer:~/go/src/github.com/hyperledger/fabric-samples/test-network$ peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" --peerAddresses localhost:9051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt" -c '{"function":"InitLedger","Args":[]}'
2023-09-17 12:32:14.342 UTC 0001 INFO [chaincodeCmd] chaincodeInvokeOrQuery -> Chaincode invoke successful. result: status:200 
magpie@DLTServer:~/go/src/github.com/hyperledger/fabric-samples/test-network$ peer chaincode query -C mychannel -n basic -c '{"Args":["GetAllAssets"]}'
[{"AppraisedValue":300,"Color":"blue","ID":"asset1","Owner":"Tomoko","Size":5},{"AppraisedValue":400,"Color":"red","ID":"asset2","Owner":"Brad","Size":5},{"AppraisedValue":500,"Color":"green","ID":"asset3","Owner":"Jin Soo","Size":10},{"AppraisedValue":600,"Color":"yellow","ID":"asset4","Owner":"Max","Size":10},{"AppraisedValue":700,"Color":"black","ID":"asset5","Owner":"Adriana","Size":15},{"AppraisedValue":800,"Color":"white","ID":"asset6","Owner":"Michel","Size":15}]
magpie@DLTServer:~/go/src/github.com/hyperledger/fabric-samples/test-network$ peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" --peerAddresses localhost:9051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt" -c '{"function":"TransferAsset","Args":["asset6","Christopher"]}'
2023-09-17 12:33:04.567 UTC 0001 INFO [chaincodeCmd] chaincodeInvokeOrQuery -> Chaincode invoke successful. result: status:200 payload:"Michel" 
magpie@DLTServer:~/go/src/github.com/hyperledger/fabric-samples/test-network$ export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051
magpie@DLTServer:~/go/src/github.com/hyperledger/fabric-samples/test-network$ peer chaincode query -C mychannel -n basic -c '{"Args":["ReadAsset","asset6"]}'
{"AppraisedValue":800,"Color":"white","ID":"asset6","Owner":"Christopher","Size":15}
magpie@DLTServer:~/go/src/github.com/hyperledger/fabric-samples/test-network$ 
```

![image-20230917204216661](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20230917204216661.png)

## 关闭网络

使用完测试网络后，可以关闭网络 使用以下命令：

```bash
./network.sh down
```

该命令将停止并删除节点和链码容器，删除 组织加密材料，并从您的 Docker 中删除链码映像 注册表。该命令还会从中删除通道项目和 docker 卷 以前的运行，允许您在遇到时再次运行 任何问题。`./network.sh up`

![image-20230917210219897](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20230917210219897.png)

## 后续步骤

现在，您已经使用测试网络在 本地计算机，您可以使用教程开始开发自己的解决方案：

- 了解如何使用将智能合约[部署到通道教程将您自己的智能合约部署到测试](https://hyperledger-fabric.readthedocs.io/en/release-2.5/deploy_chaincode.html)网络。
- 访问[运行结构应用程序](https://hyperledger-fabric.readthedocs.io/en/release-2.5/write_first_app.html)教程 了解如何使用结构 SDK 提供的 API 调用智能 来自客户端应用程序的合同。

您可以在[教程](https://hyperledger-fabric.readthedocs.io/en/release-2.5/tutorials.html)页面上找到 Fabric 教程的完整列表。



## 使用证书颁发机构启动网络

Hyperledger Fabric使用公钥基础设施（PKI）来验证 所有网络参与者。每个节点、网络管理员和用户提交 交易需要有公共证书和私钥来验证其 身份。这些标识需要具有有效的信任根，从而建立 证书是由属于 网络。该脚本创建所有加密材料 在创建对等体之前部署和操作网络所必需的 和排序节点。`network.sh`

默认情况下，脚本使用加密工具创建证书 和钥匙。该工具提供用于开发和测试，可以快速 为具有有效根的结构组织创建所需的加密材料 的信任。当您运行时，您可以看到加密工具正在创建 组织 1、组织 2 和排序者组织的证书和密钥。`./network.sh up`

```bash
creating Org1, Org2, and ordering service organization with crypto from 'cryptogen'

/Usr/fabric-samples/test-network/../bin/cryptogen

##########################################################
##### Generate certificates using cryptogen tool #########
##########################################################

##########################################################
############ Create Org1 Identities ######################
##########################################################
+ cryptogen generate --config=./organizations/cryptogen/crypto-config-org1.yaml --output=organizations
org1.example.com
+ res=0
+ set +x
##########################################################
############ Create Org2 Identities ######################
##########################################################
+ cryptogen generate --config=./organizations/cryptogen/crypto-config-org2.yaml --output=organizations
org2.example.com
+ res=0
+ set +x
##########################################################
############ Create Orderer Org Identities ###############
##########################################################
+ cryptogen generate --config=./organizations/cryptogen/crypto-config-orderer.yaml --output=organizations
+ res=0
+ set +x
```

但是，测试网络脚本还提供了使用 证书颁发机构 （CA）。在生产网络中，每个组织 操作一个 CA（或多个中间 CA），用于创建标识属于他们的组织。由组织共享相同的信任根。虽然花费的时间比使用加密，使用 CA 启动测试网络提供了简介 网络在生产中的部署方式。部署 CA 还允许您注册 使用结构 SDK 的客户端身份，并创建证书和私钥 为您的应用。

如果要使用结构 CA 启动网络，请首先运行以下命令 命令以关闭任何正在运行的网络：

```bash
./network.sh down
```

然后，您可以使用 CA 标志启动网络：

```bash
./network.sh up -ca
```

发出命令后，您可以看到脚本启动三个 CA，一个 对于网络中的每个组织。

```bash
##########################################################
##### Generate certificates using Fabric CA's ############
##########################################################
Creating network "net_default" with the default driver
Creating ca_org2    ... done
Creating ca_org1    ... done
Creating ca_orderer ... done
```

在部署 CA 后，值得花时间检查脚本生成的日志。测试网络使用结构 CA 客户端，用于向每个组织的 CA 注册节点和用户身份。这 然后，脚本使用注册命令为每个身份生成一个 MSP 文件夹。 MSP 文件夹包含每个身份的证书和私钥，并且 在运营的组织中建立标识的角色和成员身份 中科院。您可以使用以下命令检查 Org1 的 MSP 文件夹 管理员用户：`./network.sh`

```bash
tree organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/
```

该命令将显示 MSP 文件夹结构和配置文件：

```bash
organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/
└── msp
    ├── IssuerPublicKey
    ├── IssuerRevocationPublicKey
    ├── cacerts
    │   └── localhost-7054-ca-org1.pem
    ├── config.yaml
    ├── keystore
    │   └── 58e81e6f1ee8930df46841bf88c22a08ae53c1332319854608539ee78ed2fd65_sk
    ├── signcerts
    │   └── cert.pem
    └── user
```

您可以在文件夹中找到管理员用户的证书，然后 文件夹中的私钥。若要了解有关 MSP 的详细信息，请参阅[成员资格服务提供商](https://hyperledger-fabric.readthedocs.io/en/release-2.5/membership/membership.html)概念主题。`signcerts``keystore`

cryptogen 和 Fabric CA 都为每个组织生成加密材料 在文件夹中。您可以找到用于设置 目录中脚本中的网络。 要了解有关如何使用结构 CA 部署结构网络的更多信息， 访问[结构 CA 操作指南](https://hyperledger-fabric-ca.readthedocs.io/en/latest/operations_guide.html)。 可以通过访问[标识](https://hyperledger-fabric.readthedocs.io/en/release-2.5/identity/identity.html)和[成员资格](https://hyperledger-fabric.readthedocs.io/en/release-2.5/membership/membership.html)概念主题了解有关 Fabric 如何使用 PKI 的更多信息。`organizations``registerEnroll.sh``organizations/fabric-ca`



## 幕后发生了什么？

如果您有兴趣了解有关示例网络的更多信息，可以 调查目录中的文件和脚本。步骤 下面提供了发出 命令时发生的情况的导览。`test-network``./network.sh up`

- `network.sh`为两个对等组织创建证书和密钥 和订购者组织。默认情况下，脚本使用加密工具 使用文件夹中的配置文件。 如果使用标志创建证书颁发机构，则脚本使用 结构 CA 服务器配置文件和脚本位于 文件夹。加密基因和结构 CA 都创建 文件夹中所有三个组织的加密材料和 MSP 文件夹。`organizations/cryptogen``-ca``registerEnroll.sh``organizations/fabric-ca``organizations`
- 一旦组织加密材料生成，就可以启动网络的节点。这 脚本使用文件夹中的文件 以创建对等节点和排序节点。该文件夹还包含启动网络节点的文件 以及三个结构 CA。此文件旨在用于端到端运行 由结构 SDK 进行测试。有关运行这些测试的详细信息，请参阅 [Node SDK](https://github.com/hyperledger/fabric-sdk-node) 存储库。`network.sh``docker-compose-test-net.yaml``docker``docker``docker-compose-e2e.yaml`
- 如果使用子命令，则运行文件夹中的脚本以创建通道 使用提供的通道名称。该脚本使用该工具创建通道创世块 基于文件中的通道配置文件。创建通道后，脚本使用对等方 cli 加入通道，并使两个对等方都锚定对等方。`createChannel``./network.sh``createChannel.sh``scripts``configtxgen``TwoOrgsApplicationGenesis``configtx/configtx.yaml``peer0.org1.example.com``peer0.org2.example.com`
- 如果您发出命令，则运行脚本以在两个对等节点上安装**资产转移（基本）**链码，然后定义 通道上的链码。一旦链码定义提交到 通道，对等 CLI 使用 和 调用初始化链码 将初始数据放在账本上的链码。`deployCC``network.sh``deployCC.sh``Init`



## 故障排除

如果您对本教程有任何问题，请查看以下内容：

- 您应该始终重新启动网络。您可以使用以下命令删除工件、加密材料、容器、卷、链码和以前运行的镜像：

  ```bash
  $ bash network.sh down
  ```

  如果不删除旧容器、映像和卷。

- 如果看到 Docker 错误，请先检查 Docker 版本（[先决条件](https://hyperledger-fabric.readthedocs.io/en/release-2.5/prereqs.html)）， ，然后尝试重新启动 Docker 进程。Docker的问题在于 通常无法立即识别。例如，您可能会看到错误 这是您的节点无法访问加密材料的结果 安装在容器内。

  如果问题仍然存在，您可以删除图像并从头开始：

  ```bash
  $ docker rm -f $(docker ps -aq)
  $ docker rmi -f $(docker images -q)
  ```

- 如果您在 macOS 上运行 Docker 桌面，并在链码安装过程中遇到以下错误：

  ```bash
  Error: chaincode install failed with status: 500 - failed to invoke backing implementation of 'InstallChaincode': could not build chaincode: docker build failed: docker image inspection failed: Get "http://unix.sock/images/dev-peer0.org1.example.com-basic_1.0-4ec191e793b27e953ff2ede5a8bcc63152cecb1e4c3f301a26e22692c61967ad-42f57faac8360472e47cbbbf3940e81bba83439702d085878d148089a1b213ca/json": dial unix /host/var/run/docker.sock: connect: no such file or directory
  Chaincode installation on peer0.org1 has failed
  Deploying chaincode failed
  ```

  此问题是由适用于 macOS 的较新版本的 Docker Desktop 引起的。要解决此问题，请在 Docker 桌面首选项中取消选中该框以改用旧版 osxfs 文件共享，然后单击“**应用并重新启动**”。`Use gRPC FUSE for file sharing`

- 如果您在创建、批准、提交、调用或查询命令时看到错误， 确保您已正确更新通道名称和链码名称。 提供的示例命令中有占位符值。

- 如果您看到以下错误：

  ```bash
  Error: Error endorsing chaincode: rpc error: code = 2 desc = Error installing chaincode code mycc:1.0(chaincode /var/hyperledger/production/chaincodes/mycc.1.0 exits)
  ```

  您可能有链码图像（例如 或 ） 从以前的运行。删除它们并尝试 再。`dev-peer1.org2.example.com-asset-transfer-1.0``dev-peer0.org1.example.com-asset-transfer-1.0`

  ```bash
  docker rmi -f $(docker images | grep dev-peer[0-9] | awk '{print $3}')
  ```

- 如果您看到以下错误：

  ```bash
  [configtx/tool/localconfig] Load -> CRIT 002 Error reading configuration: Unsupported Config Type ""
  panic: Error reading configuration: Unsupported Config Type ""
  ```

  然后，您没有正确设置环境变量。这 configtxgen 工具需要此变量才能找到 configtx.yaml。去 返回并执行一个 ， 然后重新创建频道项目。`FABRIC_CFG_PATH``export FABRIC_CFG_PATH=$PWD/configtx/configtx.yaml`

- 如果您看到一个错误，指出您仍有“活动端点”，请修剪 您的 Docker 网络。这将擦除您以前的网络，并以 清新环境：

  ```bash
  docker network prune
  ```

  您将看到以下消息：

  ```bash
  WARNING! This will remove all networks not used by at least one container.
  Are you sure you want to continue? [y/N]
  ```

  选择。`y`

- 如果尝试使用命令创建通道， 并且失败并显示以下错误：`./network.sh createChannel`

  ```bash
  [comm.tls] ClientHandshake -> ERRO 003 Client TLS handshake failed after 1.908956ms with error: EOF remoteaddress=127.0.0.1:7051
  Error: error getting endorser client for channel: endorser client failed to connect to localhost:7051: failed to create new connection: context deadline exceeded
  After 5 attempts, peer0.org1 has failed to join channel 'mychannel'
  ```

  您需要卸载 Docker 桌面并重新安装推荐的版本 2.5.0.1。然后，在重新尝试命令之前重新克隆存储库。`fabric-samples`

- 如果您看到类似于以下内容的错误：

  ```bash
  /bin/bash: ./scripts/createChannel.sh: /bin/bash^M: bad interpreter: No such file or directory
  ```

  确保有问题的文件（在本例中 **createChannel.sh**）是 以 Unix 格式编码。这很可能是由于未在 Git 配置中设置为 （请参阅 [Windows](https://hyperledger-fabric.readthedocs.io/en/release-2.5/prereqs.html#windows)） 引起的。有几种方法可以解决此问题。如果你有 访问 vim 编辑器 例如，打开文件：`core.autocrlf``false`

  ```
  vim fabric-samples/test-network/scripts/createChannel.sh
  ```

  然后通过执行以下 vim 命令更改其格式：

  ```
  :set ff=unix
  ```

如果您仍然看到错误，请在其中一个结构[不和谐频道](https://discord.com/invite/hyperledger)或 [StackOverflow](https://stackoverflow.com/questions/tagged/hyperledger-fabric) 上共享您的日志。

## 清理环境命令

```bash
docker rm  $(docker ps -a | grep "hyperledger/*" | awk "{print \$1}") && \
docker-compose down --volumes --remove-orphans && \
docker volume prune && \
docker network prune 
```



# *Hyperledger* *fabric-sample* 的*test-network*实战教程

## 1.下载*fabric-sample*

```shell
$ cd ~/go/src/github.com/hyperledger
$ git clone https://github.com/hyperledger/fabric-samples.git
$ cd fabric-samples
$ git checkout -b v2.5.3
```

## 2.移动*config*文件夹（移动配置文件）

（1）进入*fabric*文件夹，会发现*sampleconfig*，其实就是*fabric-samples*的*config*文件夹

（2）将*sampleconfig*文件夹移动到*fabric-samples*中，并且重命名为*config*

```shell
$ cd ~/go/src/github.com/hyperledger/fabric
$ cp -r sampleconfig ~/go/src/github.com/hyperledger/fabric-samples
$ cd ~/go/src/github.com/hyperledger/fabric-samples
$ mv sampleconfig config
```

![image-20230528220406938](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20230528220406938.png)

## 3.创建并初始化*Fabric*网络

```bash
$ cd test-network
$ ./network.sh up -ca
$ ./network.sh createChannel
$ ./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go -ccl go
```


./network.sh up -ca执行完毕后如下图所示

```bash
magpie@DLTFabricer:~/go/src/github.com/hyperledger/fabric-samples/test-network$ ./network.sh up -ca
Using docker and docker-compose
Starting nodes with CLI timeout of '5' tries and CLI delay of '3' seconds and using database 'leveldb' with crypto from 'Certificate Authorities'
LOCAL_VERSION=2.5.1
DOCKER_IMAGE_VERSION=2.5.1
CA_LOCAL_VERSION=v1.5.6
CA_DOCKER_IMAGE_VERSION=v1.5.6
Generating certificates using Fabric CA
[+] Running 4/4
 ✔ Network fabric_test   Created                                                                                                                                                  0.1s 
 ✔ Container ca_org1     Started                                                                                                                                                  2.1s 
 ✔ Container ca_org2     Started                                                                                                                                                  2.4s 
 ✔ Container ca_orderer  Started                                                                                                                                                  2.1s 
Creating Org1 Identities
Enrolling the CA admin
+ fabric-ca-client enroll -u https://admin:adminpw@localhost:7054 --caname ca-org1 --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/org1/ca-cert.pem
2023/05/23 16:04:25 [INFO] Created a default configuration file at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/fabric-ca-client-config.yaml
2023/05/23 16:04:25 [INFO] TLS Enabled
2023/05/23 16:04:25 [INFO] generating key: &{A:ecdsa S:256}
2023/05/23 16:04:25 [INFO] encoded CSR
2023/05/23 16:04:25 [INFO] Stored client certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/msp/signcerts/cert.pem
2023/05/23 16:04:25 [INFO] Stored root CA certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/msp/cacerts/localhost-7054-ca-org1.pem
2023/05/23 16:04:25 [INFO] Stored Issuer public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/msp/IssuerPublicKey
2023/05/23 16:04:25 [INFO] Stored Issuer revocation public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/msp/IssuerRevocationPublicKey
Registering peer0
+ fabric-ca-client register --caname ca-org1 --id.name peer0 --id.secret peer0pw --id.type peer --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/org1/ca-cert.pem
2023/05/23 16:04:25 [INFO] Configuration file location: /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/fabric-ca-client-config.yaml
2023/05/23 16:04:25 [INFO] TLS Enabled
2023/05/23 16:04:25 [INFO] TLS Enabled
Password: peer0pw
Registering user
+ fabric-ca-client register --caname ca-org1 --id.name user1 --id.secret user1pw --id.type client --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/org1/ca-cert.pem
2023/05/23 16:04:26 [INFO] Configuration file location: /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/fabric-ca-client-config.yaml
2023/05/23 16:04:26 [INFO] TLS Enabled
2023/05/23 16:04:26 [INFO] TLS Enabled
Password: user1pw
Registering the org admin
+ fabric-ca-client register --caname ca-org1 --id.name org1admin --id.secret org1adminpw --id.type admin --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/org1/ca-cert.pem
2023/05/23 16:04:26 [INFO] Configuration file location: /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/fabric-ca-client-config.yaml
2023/05/23 16:04:26 [INFO] TLS Enabled
2023/05/23 16:04:26 [INFO] TLS Enabled
Password: org1adminpw
Generating the peer0 msp
+ fabric-ca-client enroll -u https://peer0:peer0pw@localhost:7054 --caname ca-org1 -M /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/org1/ca-cert.pem
2023/05/23 16:04:26 [INFO] TLS Enabled
2023/05/23 16:04:26 [INFO] generating key: &{A:ecdsa S:256}
2023/05/23 16:04:26 [INFO] encoded CSR
2023/05/23 16:04:26 [INFO] Stored client certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp/signcerts/cert.pem
2023/05/23 16:04:26 [INFO] Stored root CA certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp/cacerts/localhost-7054-ca-org1.pem
2023/05/23 16:04:26 [INFO] Stored Issuer public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp/IssuerPublicKey
2023/05/23 16:04:26 [INFO] Stored Issuer revocation public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp/IssuerRevocationPublicKey
Generating the peer0-tls certificates, use --csr.hosts to specify Subject Alternative Names
+ fabric-ca-client enroll -u https://peer0:peer0pw@localhost:7054 --caname ca-org1 -M /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls --enrollment.profile tls --csr.hosts peer0.org1.example.com --csr.hosts localhost --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/org1/ca-cert.pem
2023/05/23 16:04:26 [INFO] TLS Enabled
2023/05/23 16:04:26 [INFO] generating key: &{A:ecdsa S:256}
2023/05/23 16:04:26 [INFO] encoded CSR
2023/05/23 16:04:26 [INFO] Stored client certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/signcerts/cert.pem
2023/05/23 16:04:26 [INFO] Stored TLS root CA certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/tlscacerts/tls-localhost-7054-ca-org1.pem
2023/05/23 16:04:26 [INFO] Stored Issuer public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/IssuerPublicKey
2023/05/23 16:04:26 [INFO] Stored Issuer revocation public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/IssuerRevocationPublicKey
Generating the user msp
+ fabric-ca-client enroll -u https://user1:user1pw@localhost:7054 --caname ca-org1 -M /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/org1/ca-cert.pem
2023/05/23 16:04:26 [INFO] TLS Enabled
2023/05/23 16:04:26 [INFO] generating key: &{A:ecdsa S:256}
2023/05/23 16:04:26 [INFO] encoded CSR
2023/05/23 16:04:26 [INFO] Stored client certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp/signcerts/cert.pem
2023/05/23 16:04:26 [INFO] Stored root CA certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp/cacerts/localhost-7054-ca-org1.pem
2023/05/23 16:04:26 [INFO] Stored Issuer public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp/IssuerPublicKey
2023/05/23 16:04:26 [INFO] Stored Issuer revocation public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp/IssuerRevocationPublicKey
Generating the org admin msp
+ fabric-ca-client enroll -u https://org1admin:org1adminpw@localhost:7054 --caname ca-org1 -M /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/org1/ca-cert.pem
2023/05/23 16:04:27 [INFO] TLS Enabled
2023/05/23 16:04:27 [INFO] generating key: &{A:ecdsa S:256}
2023/05/23 16:04:27 [INFO] encoded CSR
2023/05/23 16:04:27 [INFO] Stored client certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/signcerts/cert.pem
2023/05/23 16:04:27 [INFO] Stored root CA certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/cacerts/localhost-7054-ca-org1.pem
2023/05/23 16:04:27 [INFO] Stored Issuer public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/IssuerPublicKey
2023/05/23 16:04:27 [INFO] Stored Issuer revocation public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/IssuerRevocationPublicKey
Creating Org2 Identities
Enrolling the CA admin
+ fabric-ca-client enroll -u https://admin:adminpw@localhost:8054 --caname ca-org2 --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/org2/ca-cert.pem
2023/05/23 16:04:27 [INFO] Created a default configuration file at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/fabric-ca-client-config.yaml
2023/05/23 16:04:27 [INFO] TLS Enabled
2023/05/23 16:04:27 [INFO] generating key: &{A:ecdsa S:256}
2023/05/23 16:04:27 [INFO] encoded CSR
2023/05/23 16:04:27 [INFO] Stored client certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/msp/signcerts/cert.pem
2023/05/23 16:04:27 [INFO] Stored root CA certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/msp/cacerts/localhost-8054-ca-org2.pem
2023/05/23 16:04:27 [INFO] Stored Issuer public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/msp/IssuerPublicKey
2023/05/23 16:04:27 [INFO] Stored Issuer revocation public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/msp/IssuerRevocationPublicKey
Registering peer0
+ fabric-ca-client register --caname ca-org2 --id.name peer0 --id.secret peer0pw --id.type peer --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/org2/ca-cert.pem
2023/05/23 16:04:27 [INFO] Configuration file location: /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/fabric-ca-client-config.yaml
2023/05/23 16:04:27 [INFO] TLS Enabled
2023/05/23 16:04:27 [INFO] TLS Enabled
Password: peer0pw
Registering user
+ fabric-ca-client register --caname ca-org2 --id.name user1 --id.secret user1pw --id.type client --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/org2/ca-cert.pem
2023/05/23 16:04:27 [INFO] Configuration file location: /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/fabric-ca-client-config.yaml
2023/05/23 16:04:27 [INFO] TLS Enabled
2023/05/23 16:04:27 [INFO] TLS Enabled
Password: user1pw
Registering the org admin
+ fabric-ca-client register --caname ca-org2 --id.name org2admin --id.secret org2adminpw --id.type admin --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/org2/ca-cert.pem
2023/05/23 16:04:27 [INFO] Configuration file location: /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/fabric-ca-client-config.yaml
2023/05/23 16:04:27 [INFO] TLS Enabled
2023/05/23 16:04:27 [INFO] TLS Enabled
Password: org2adminpw
Generating the peer0 msp
+ fabric-ca-client enroll -u https://peer0:peer0pw@localhost:8054 --caname ca-org2 -M /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/msp --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/org2/ca-cert.pem
2023/05/23 16:04:27 [INFO] TLS Enabled
2023/05/23 16:04:27 [INFO] generating key: &{A:ecdsa S:256}
2023/05/23 16:04:27 [INFO] encoded CSR
2023/05/23 16:04:28 [INFO] Stored client certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/msp/signcerts/cert.pem
2023/05/23 16:04:28 [INFO] Stored root CA certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/msp/cacerts/localhost-8054-ca-org2.pem
2023/05/23 16:04:28 [INFO] Stored Issuer public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/msp/IssuerPublicKey
2023/05/23 16:04:28 [INFO] Stored Issuer revocation public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/msp/IssuerRevocationPublicKey
Generating the peer0-tls certificates, use --csr.hosts to specify Subject Alternative Names
+ fabric-ca-client enroll -u https://peer0:peer0pw@localhost:8054 --caname ca-org2 -M /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls --enrollment.profile tls --csr.hosts peer0.org2.example.com --csr.hosts localhost --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/org2/ca-cert.pem
2023/05/23 16:04:28 [INFO] TLS Enabled
2023/05/23 16:04:28 [INFO] generating key: &{A:ecdsa S:256}
2023/05/23 16:04:28 [INFO] encoded CSR
2023/05/23 16:04:28 [INFO] Stored client certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/signcerts/cert.pem
2023/05/23 16:04:28 [INFO] Stored TLS root CA certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/tlscacerts/tls-localhost-8054-ca-org2.pem
2023/05/23 16:04:28 [INFO] Stored Issuer public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/IssuerPublicKey
2023/05/23 16:04:28 [INFO] Stored Issuer revocation public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/IssuerRevocationPublicKey
Generating the user msp
+ fabric-ca-client enroll -u https://user1:user1pw@localhost:8054 --caname ca-org2 -M /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/users/User1@org2.example.com/msp --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/org2/ca-cert.pem
2023/05/23 16:04:28 [INFO] TLS Enabled
2023/05/23 16:04:28 [INFO] generating key: &{A:ecdsa S:256}
2023/05/23 16:04:28 [INFO] encoded CSR
2023/05/23 16:04:28 [INFO] Stored client certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/users/User1@org2.example.com/msp/signcerts/cert.pem
2023/05/23 16:04:28 [INFO] Stored root CA certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/users/User1@org2.example.com/msp/cacerts/localhost-8054-ca-org2.pem
2023/05/23 16:04:28 [INFO] Stored Issuer public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/users/User1@org2.example.com/msp/IssuerPublicKey
2023/05/23 16:04:28 [INFO] Stored Issuer revocation public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/users/User1@org2.example.com/msp/IssuerRevocationPublicKey
Generating the org admin msp
+ fabric-ca-client enroll -u https://org2admin:org2adminpw@localhost:8054 --caname ca-org2 -M /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/org2/ca-cert.pem
2023/05/23 16:04:28 [INFO] TLS Enabled
2023/05/23 16:04:28 [INFO] generating key: &{A:ecdsa S:256}
2023/05/23 16:04:28 [INFO] encoded CSR
2023/05/23 16:04:28 [INFO] Stored client certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp/signcerts/cert.pem
2023/05/23 16:04:28 [INFO] Stored root CA certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp/cacerts/localhost-8054-ca-org2.pem
2023/05/23 16:04:28 [INFO] Stored Issuer public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp/IssuerPublicKey
2023/05/23 16:04:28 [INFO] Stored Issuer revocation public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp/IssuerRevocationPublicKey
Creating Orderer Org Identities
Enrolling the CA admin
+ fabric-ca-client enroll -u https://admin:adminpw@localhost:9054 --caname ca-orderer --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/ordererOrg/ca-cert.pem
2023/05/23 16:04:28 [INFO] Created a default configuration file at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/fabric-ca-client-config.yaml
2023/05/23 16:04:28 [INFO] TLS Enabled
2023/05/23 16:04:28 [INFO] generating key: &{A:ecdsa S:256}
2023/05/23 16:04:28 [INFO] encoded CSR
2023/05/23 16:04:28 [INFO] Stored client certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/msp/signcerts/cert.pem
2023/05/23 16:04:28 [INFO] Stored root CA certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/msp/cacerts/localhost-9054-ca-orderer.pem
2023/05/23 16:04:28 [INFO] Stored Issuer public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/msp/IssuerPublicKey
2023/05/23 16:04:28 [INFO] Stored Issuer revocation public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/msp/IssuerRevocationPublicKey
Registering orderer
+ fabric-ca-client register --caname ca-orderer --id.name orderer --id.secret ordererpw --id.type orderer --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/ordererOrg/ca-cert.pem
2023/05/23 16:04:28 [INFO] Configuration file location: /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/fabric-ca-client-config.yaml
2023/05/23 16:04:28 [INFO] TLS Enabled
2023/05/23 16:04:28 [INFO] TLS Enabled
Password: ordererpw
Registering the orderer admin
+ fabric-ca-client register --caname ca-orderer --id.name ordererAdmin --id.secret ordererAdminpw --id.type admin --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/ordererOrg/ca-cert.pem
2023/05/23 16:04:29 [INFO] Configuration file location: /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/fabric-ca-client-config.yaml
2023/05/23 16:04:29 [INFO] TLS Enabled
2023/05/23 16:04:29 [INFO] TLS Enabled
Password: ordererAdminpw
Generating the orderer msp
+ fabric-ca-client enroll -u https://orderer:ordererpw@localhost:9054 --caname ca-orderer -M /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/ordererOrg/ca-cert.pem
2023/05/23 16:04:29 [INFO] TLS Enabled
2023/05/23 16:04:29 [INFO] generating key: &{A:ecdsa S:256}
2023/05/23 16:04:29 [INFO] encoded CSR
2023/05/23 16:04:29 [INFO] Stored client certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/signcerts/cert.pem
2023/05/23 16:04:29 [INFO] Stored root CA certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/cacerts/localhost-9054-ca-orderer.pem
2023/05/23 16:04:29 [INFO] Stored Issuer public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/IssuerPublicKey
2023/05/23 16:04:29 [INFO] Stored Issuer revocation public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/IssuerRevocationPublicKey
Generating the orderer-tls certificates, use --csr.hosts to specify Subject Alternative Names
+ fabric-ca-client enroll -u https://orderer:ordererpw@localhost:9054 --caname ca-orderer -M /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls --enrollment.profile tls --csr.hosts orderer.example.com --csr.hosts localhost --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/ordererOrg/ca-cert.pem
2023/05/23 16:04:29 [INFO] TLS Enabled
2023/05/23 16:04:29 [INFO] generating key: &{A:ecdsa S:256}
2023/05/23 16:04:29 [INFO] encoded CSR
2023/05/23 16:04:29 [INFO] Stored client certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/signcerts/cert.pem
2023/05/23 16:04:29 [INFO] Stored TLS root CA certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/tlscacerts/tls-localhost-9054-ca-orderer.pem
2023/05/23 16:04:29 [INFO] Stored Issuer public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/IssuerPublicKey
2023/05/23 16:04:29 [INFO] Stored Issuer revocation public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/IssuerRevocationPublicKey
Generating the admin msp
+ fabric-ca-client enroll -u https://ordererAdmin:ordererAdminpw@localhost:9054 --caname ca-orderer -M /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/users/Admin@example.com/msp --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/ordererOrg/ca-cert.pem
2023/05/23 16:04:29 [INFO] TLS Enabled
2023/05/23 16:04:29 [INFO] generating key: &{A:ecdsa S:256}
2023/05/23 16:04:29 [INFO] encoded CSR
2023/05/23 16:04:29 [INFO] Stored client certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/users/Admin@example.com/msp/signcerts/cert.pem
2023/05/23 16:04:29 [INFO] Stored root CA certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/users/Admin@example.com/msp/cacerts/localhost-9054-ca-orderer.pem
2023/05/23 16:04:29 [INFO] Stored Issuer public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/users/Admin@example.com/msp/IssuerPublicKey
2023/05/23 16:04:29 [INFO] Stored Issuer revocation public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/users/Admin@example.com/msp/IssuerRevocationPublicKey
Generating CCP files for Org1 and Org2
WARN[0000] Found orphan containers ([ca_org2 ca_orderer ca_org1]) for this project. If you removed or renamed this service in your compose file, you can run this command with the --remove-orphans flag to clean it up. 
[+] Running 7/7
 ✔ Volume "compose_orderer.example.com"     Created                                                                                                                               0.0s 
 ✔ Volume "compose_peer0.org1.example.com"  Created                                                                                                                               0.0s 
 ✔ Volume "compose_peer0.org2.example.com"  Created                                                                                                                               0.0s 
 ✔ Container peer0.org2.example.com         Started                                                                                                                               1.0s 
 ✔ Container peer0.org1.example.com         Started                                                                                                                               1.4s 
 ✔ Container orderer.example.com            Started                                                                                                                               1.7s 
 ✔ Container cli                            Started                                                                                                                               2.0s 
CONTAINER ID   IMAGE                               COMMAND                  CREATED          STATUS                  PORTS                                                                                                                             NAMES
e8cd11c190a2   hyperledger/fabric-tools:latest     "/bin/bash"              2 seconds ago    Up Less than a second                                                                                                                                     cli
8dfc4f265a72   hyperledger/fabric-peer:latest      "peer node start"        2 seconds ago    Up 1 second             0.0.0.0:9051->9051/tcp, :::9051->9051/tcp, 7051/tcp, 0.0.0.0:9445->9445/tcp, :::9445->9445/tcp                                    peer0.org2.example.com
d31f1c894d8c   hyperledger/fabric-peer:latest      "peer node start"        2 seconds ago    Up Less than a second   0.0.0.0:7051->7051/tcp, :::7051->7051/tcp, 0.0.0.0:9444->9444/tcp, :::9444->9444/tcp                                              peer0.org1.example.com
b7f03702de7d   hyperledger/fabric-orderer:latest   "orderer"                2 seconds ago    Up Less than a second   0.0.0.0:7050->7050/tcp, :::7050->7050/tcp, 0.0.0.0:7053->7053/tcp, :::7053->7053/tcp, 0.0.0.0:9443->9443/tcp, :::9443->9443/tcp   orderer.example.com
0298dc62f791   hyperledger/fabric-ca:latest        "sh -c 'fabric-ca-se…"   10 seconds ago   Up 7 seconds            0.0.0.0:8054->8054/tcp, :::8054->8054/tcp, 7054/tcp, 0.0.0.0:18054->18054/tcp, :::18054->18054/tcp                                ca_org2
b106b3d8b601   hyperledger/fabric-ca:latest        "sh -c 'fabric-ca-se…"   10 seconds ago   Up 8 seconds            0.0.0.0:9054->9054/tcp, :::9054->9054/tcp, 7054/tcp, 0.0.0.0:19054->19054/tcp, :::19054->19054/tcp                                ca_orderer
9e040ce002f1   hyperledger/fabric-ca:latest        "sh -c 'fabric-ca-se…"   10 seconds ago   Up 8 seconds            0.0.0.0:7054->7054/tcp, :::7054->7054/tcp, 0.0.0.0:17054->17054/tcp, :::17054->17054/tcp                                          ca_org1
f1e8a0f68934   portainer/portainer                 "/portainer"             2 days ago       Up 5 hours              0.0.0.0:9000->9000/tcp, :::9000->9000/tcp                                                                                         epic_solomon
```

./network.sh createChannel执行完毕后如下图所示

```bash
magpie@DLTFabricer:~/go/src/github.com/hyperledger/fabric-samples/test-network$ ./network.sh createChannel
Using docker and docker-compose
Creating channel 'mychannel'.
If network is not up, starting nodes with CLI timeout of '5' tries and CLI delay of '3' seconds and using database 'leveldb 
Network Running Already
Using docker and docker-compose
Generating channel genesis block 'mychannel.block'
/home/magpie/go/bin/configtxgen
+ configtxgen -profile TwoOrgsApplicationGenesis -outputBlock ./channel-artifacts/mychannel.block -channelID mychannel
2023-05-23 16:06:43.977 UTC 0001 INFO [common.tools.configtxgen] main -> Loading configuration
2023-05-23 16:06:43.989 UTC 0002 INFO [common.tools.configtxgen.localconfig] completeInitialization -> orderer type: etcdraft
2023-05-23 16:06:43.989 UTC 0003 INFO [common.tools.configtxgen.localconfig] completeInitialization -> Orderer.EtcdRaft.Options unset, setting to tick_interval:"500ms" election_tick:10 heartbeat_tick:1 max_inflight_blocks:5 snapshot_interval_size:16777216 
2023-05-23 16:06:43.989 UTC 0004 INFO [common.tools.configtxgen.localconfig] Load -> Loaded configuration: /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/configtx/configtx.yaml
2023-05-23 16:06:43.992 UTC 0005 INFO [common.tools.configtxgen] doOutputBlock -> Generating genesis block
2023-05-23 16:06:43.992 UTC 0006 INFO [common.tools.configtxgen] doOutputBlock -> Creating application channel genesis block
2023-05-23 16:06:43.993 UTC 0007 INFO [common.tools.configtxgen] doOutputBlock -> Writing genesis block
+ res=0
Creating channel mychannel
Using organization 1
+ osnadmin channel join --channelID mychannel --config-block ./channel-artifacts/mychannel.block -o localhost:7053 --ca-file /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/tlsca/tlsca.example.com-cert.pem --client-cert /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt --client-key /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.key
+ res=0
Status: 201
{
        "name": "mychannel",
        "url": "/participation/v1/channels/mychannel",
        "consensusRelation": "consenter",
        "status": "active",
        "height": 1
}

Channel 'mychannel' created
Joining org1 peer to the channel...
Using organization 1
+ peer channel join -b ./channel-artifacts/mychannel.block
+ res=0
2023-05-23 16:06:50.496 UTC 0001 INFO [channelCmd] InitCmdFactory -> Endorser and orderer connections initialized
2023-05-23 16:06:50.611 UTC 0002 INFO [channelCmd] executeJoin -> Successfully submitted proposal to join channel
Joining org2 peer to the channel...
Using organization 2
+ peer channel join -b ./channel-artifacts/mychannel.block
+ res=0
2023-05-23 16:06:53.729 UTC 0001 INFO [channelCmd] InitCmdFactory -> Endorser and orderer connections initialized
2023-05-23 16:06:53.762 UTC 0002 INFO [channelCmd] executeJoin -> Successfully submitted proposal to join channel
Setting anchor peer for org1...
Using organization 1
Fetching channel config for channel mychannel
Using organization 1
Fetching the most recent configuration block for the channel
+ peer channel fetch config config_block.pb -o orderer.example.com:7050 --ordererTLSHostnameOverride orderer.example.com -c mychannel --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/organizations/ordererOrganizations/example.com/tlsca/tlsca.example.com-cert.pem
2023-05-23 16:06:53.963 UTC 0001 INFO [channelCmd] InitCmdFactory -> Endorser and orderer connections initialized
2023-05-23 16:06:53.968 UTC 0002 INFO [cli.common] readBlock -> Received block: 0
2023-05-23 16:06:53.969 UTC 0003 INFO [channelCmd] fetch -> Retrieving last config block: 0
2023-05-23 16:06:53.971 UTC 0004 INFO [cli.common] readBlock -> Received block: 0
Decoding config block to JSON and isolating config to Org1MSPconfig.json
+ configtxlator proto_decode --input config_block.pb --type common.Block --output config_block.json
+ jq '.data.data[0].payload.data.config' config_block.json
Generating anchor peer update transaction for Org1 on channel mychannel
+ jq '.channel_group.groups.Application.groups.Org1MSP.values += {"AnchorPeers":{"mod_policy": "Admins","value":{"anchor_peers": [{"host": "peer0.org1.example.com","port": 7051}]},"version": "0"}}' Org1MSPconfig.json
+ configtxlator proto_encode --input Org1MSPconfig.json --type common.Config --output original_config.pb
+ configtxlator proto_encode --input Org1MSPmodified_config.json --type common.Config --output modified_config.pb
+ configtxlator compute_update --channel_id mychannel --original original_config.pb --updated modified_config.pb --output config_update.pb
+ configtxlator proto_decode --input config_update.pb --type common.ConfigUpdate --output config_update.json
+ jq .
++ cat config_update.json
+ echo '{"payload":{"header":{"channel_header":{"channel_id":"mychannel", "type":2}},"data":{"config_update":{' '"channel_id":' '"mychannel",' '"isolated_data":' '{},' '"read_set":' '{' '"groups":' '{' '"Application":' '{' '"groups":' '{' '"Org1MSP":' '{' '"groups":' '{},' '"mod_policy":' '"",' '"policies":' '{' '"Admins":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Endorsement":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Readers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Writers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '}' '},' '"values":' '{' '"MSP":' '{' '"mod_policy":' '"",' '"value":' null, '"version":' '"0"' '}' '},' '"version":' '"0"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '},' '"write_set":' '{' '"groups":' '{' '"Application":' '{' '"groups":' '{' '"Org1MSP":' '{' '"groups":' '{},' '"mod_policy":' '"Admins",' '"policies":' '{' '"Admins":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Endorsement":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Readers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Writers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '}' '},' '"values":' '{' '"AnchorPeers":' '{' '"mod_policy":' '"Admins",' '"value":' '{' '"anchor_peers":' '[' '{' '"host":' '"peer0.org1.example.com",' '"port":' 7051 '}' ']' '},' '"version":' '"0"' '},' '"MSP":' '{' '"mod_policy":' '"",' '"value":' null, '"version":' '"0"' '}' '},' '"version":' '"1"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '}' '}}}}'
+ configtxlator proto_encode --input config_update_in_envelope.json --type common.Envelope --output Org1MSPanchors.tx
2023-05-23 16:06:54.985 UTC 0001 INFO [channelCmd] InitCmdFactory -> Endorser and orderer connections initialized
2023-05-23 16:06:55.001 UTC 0002 INFO [channelCmd] update -> Successfully submitted channel update
Anchor peer set for org 'Org1MSP' on channel 'mychannel'
Setting anchor peer for org2...
Using organization 2
+ peer channel fetch config config_block.pb -o orderer.example.com:7050 --ordererTLSHostnameOverride orderer.example.com -c mychannel --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/organizations/ordererOrganizations/example.com/tlsca/tlsca.example.com-cert.pem
Fetching channel config for channel mychannel
Using organization 2
Fetching the most recent configuration block for the channel
2023-05-23 16:06:55.200 UTC 0001 INFO [channelCmd] InitCmdFactory -> Endorser and orderer connections initialized
2023-05-23 16:06:55.204 UTC 0002 INFO [cli.common] readBlock -> Received block: 1
2023-05-23 16:06:55.204 UTC 0003 INFO [channelCmd] fetch -> Retrieving last config block: 1
2023-05-23 16:06:55.208 UTC 0004 INFO [cli.common] readBlock -> Received block: 1
Decoding config block to JSON and isolating config to Org2MSPconfig.json
+ configtxlator proto_decode --input config_block.pb --type common.Block --output config_block.json
+ jq '.data.data[0].payload.data.config' config_block.json
+ jq '.channel_group.groups.Application.groups.Org2MSP.values += {"AnchorPeers":{"mod_policy": "Admins","value":{"anchor_peers": [{"host": "peer0.org2.example.com","port": 9051}]},"version": "0"}}' Org2MSPconfig.json
Generating anchor peer update transaction for Org2 on channel mychannel
+ configtxlator proto_encode --input Org2MSPconfig.json --type common.Config --output original_config.pb
+ configtxlator proto_encode --input Org2MSPmodified_config.json --type common.Config --output modified_config.pb
+ configtxlator compute_update --channel_id mychannel --original original_config.pb --updated modified_config.pb --output config_update.pb
+ configtxlator proto_decode --input config_update.pb --type common.ConfigUpdate --output config_update.json
+ jq .
++ cat config_update.json
+ echo '{"payload":{"header":{"channel_header":{"channel_id":"mychannel", "type":2}},"data":{"config_update":{' '"channel_id":' '"mychannel",' '"isolated_data":' '{},' '"read_set":' '{' '"groups":' '{' '"Application":' '{' '"groups":' '{' '"Org2MSP":' '{' '"groups":' '{},' '"mod_policy":' '"",' '"policies":' '{' '"Admins":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Endorsement":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Readers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Writers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '}' '},' '"values":' '{' '"MSP":' '{' '"mod_policy":' '"",' '"value":' null, '"version":' '"0"' '}' '},' '"version":' '"0"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '},' '"write_set":' '{' '"groups":' '{' '"Application":' '{' '"groups":' '{' '"Org2MSP":' '{' '"groups":' '{},' '"mod_policy":' '"Admins",' '"policies":' '{' '"Admins":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Endorsement":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Readers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Writers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '}' '},' '"values":' '{' '"AnchorPeers":' '{' '"mod_policy":' '"Admins",' '"value":' '{' '"anchor_peers":' '[' '{' '"host":' '"peer0.org2.example.com",' '"port":' 9051 '}' ']' '},' '"version":' '"0"' '},' '"MSP":' '{' '"mod_policy":' '"",' '"value":' null, '"version":' '"0"' '}' '},' '"version":' '"1"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '}' '}}}}'
+ configtxlator proto_encode --input config_update_in_envelope.json --type common.Envelope --output Org2MSPanchors.tx
2023-05-23 16:06:55.602 UTC 0001 INFO [channelCmd] InitCmdFactory -> Endorser and orderer connections initialized
2023-05-23 16:06:55.618 UTC 0002 INFO [channelCmd] update -> Successfully submitted channel update
Anchor peer set for org 'Org2MSP' on channel 'mychannel'
Channel 'mychannel' joined
magpie@DLTFabricer:~/go/src/github.com/hyperledger/fabric-samples/test-network$ 
```

./network.sh deployCC执行完毕后如下图所示

```bash
./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go -ccl go
```

具体

```bash
magpie@DLTFabricer:~/go/src/github.com/hyperledger/fabric-samples/test-network$ ./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go -ccl go
Using docker and docker-compose
deploying chaincode on channel 'mychannel'
executing with the following
- CHANNEL_NAME: mychannel
- CC_NAME: basic
- CC_SRC_PATH: ../asset-transfer-basic/chaincode-go
- CC_SRC_LANGUAGE: go
- CC_VERSION: 1.0
- CC_SEQUENCE: 1
- CC_END_POLICY: NA
- CC_COLL_CONFIG: NA
- CC_INIT_FCN: NA
- DELAY: 3
- MAX_RETRY: 5
- VERBOSE: false
Vendoring Go dependencies at ../asset-transfer-basic/chaincode-go
~/go/src/github.com/hyperledger/fabric-samples/asset-transfer-basic/chaincode-go ~/go/src/github.com/hyperledger/fabric-samples/test-network
~/go/src/github.com/hyperledger/fabric-samples/test-network
Finished vendoring Go dependencies
+ peer lifecycle chaincode package basic.tar.gz --path ../asset-transfer-basic/chaincode-go --lang golang --label basic_1.0
+ res=0
++ peer lifecycle chaincode calculatepackageid basic.tar.gz
+ PACKAGE_ID=basic_1.0:e920c6326d85a919c6f533f6e770f184da9fef953abefbac8e1a7f22a6b16c4c
Chaincode is packaged
Installing chaincode on peer0.org1...
Using organization 1
+ peer lifecycle chaincode queryinstalled --output json
+ jq -r 'try (.installed_chaincodes[].package_id)'
+ grep '^basic_1.0:e920c6326d85a919c6f533f6e770f184da9fef953abefbac8e1a7f22a6b16c4c$'
+ test 1 -ne 0
+ peer lifecycle chaincode install basic.tar.gz
+ res=0
2023-05-24 12:28:17.370 UTC 0001 INFO [cli.lifecycle.chaincode] submitInstallProposal -> Installed remotely: response:<status:200 payload:"\nJbasic_1.0:e920c6326d85a919c6f533f6e770f184da9fef953abefbac8e1a7f22a6b16c4c\022\tbasic_1.0" > 
2023-05-24 12:28:17.692 UTC 0002 INFO [cli.lifecycle.chaincode] submitInstallProposal -> Chaincode code package identifier: basic_1.0:e920c6326d85a919c6f533f6e770f184da9fef953abefbac8e1a7f22a6b16c4c
Chaincode is installed on peer0.org1
Install chaincode on peer0.org2...
Using organization 2
+ jq -r 'try (.installed_chaincodes[].package_id)'
+ peer lifecycle chaincode queryinstalled --output json
+ grep '^basic_1.0:e920c6326d85a919c6f533f6e770f184da9fef953abefbac8e1a7f22a6b16c4c$'
+ test 1 -ne 0
+ peer lifecycle chaincode install basic.tar.gz
+ res=0
2023-05-24 12:29:02.172 UTC 0001 INFO [cli.lifecycle.chaincode] submitInstallProposal -> Installed remotely: response:<status:200 payload:"\nJbasic_1.0:e920c6326d85a919c6f533f6e770f184da9fef953abefbac8e1a7f22a6b16c4c\022\tbasic_1.0" > 
2023-05-24 12:29:02.172 UTC 0002 INFO [cli.lifecycle.chaincode] submitInstallProposal -> Chaincode code package identifier: basic_1.0:e920c6326d85a919c6f533f6e770f184da9fef953abefbac8e1a7f22a6b16c4c
Chaincode is installed on peer0.org2
Using organization 1
+ peer lifecycle chaincode queryinstalled --output json
+ grep '^basic_1.0:e920c6326d85a919c6f533f6e770f184da9fef953abefbac8e1a7f22a6b16c4c$'
+ jq -r 'try (.installed_chaincodes[].package_id)'
+ res=0
basic_1.0:e920c6326d85a919c6f533f6e770f184da9fef953abefbac8e1a7f22a6b16c4c
Query installed successful on peer0.org1 on channel
Using organization 1
+ peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/tlsca/tlsca.example.com-cert.pem --channelID mychannel --name basic --version 1.0 --package-id basic_1.0:e920c6326d85a919c6f533f6e770f184da9fef953abefbac8e1a7f22a6b16c4c --sequence 1
+ res=0
2023-05-24 12:29:04.399 UTC 0001 INFO [chaincodeCmd] ClientWait -> txid [469182f6aee7fc3998d39f1b490f0ea3af01ff871e70e854cfa23050edfb9467] committed with status (VALID) at localhost:7051
Chaincode definition approved on peer0.org1 on channel 'mychannel'
Using organization 1
Checking the commit readiness of the chaincode definition on peer0.org1 on channel 'mychannel'...
Attempting to check the commit readiness of the chaincode definition on peer0.org1, Retry after 3 seconds.
+ peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name basic --version 1.0 --sequence 1 --output json
+ res=0
{
        "approvals": {
                "Org1MSP": true,
                "Org2MSP": false
        }
}
Checking the commit readiness of the chaincode definition successful on peer0.org1 on channel 'mychannel'
Using organization 2
Checking the commit readiness of the chaincode definition on peer0.org2 on channel 'mychannel'...
Attempting to check the commit readiness of the chaincode definition on peer0.org2, Retry after 3 seconds.
+ peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name basic --version 1.0 --sequence 1 --output json
+ res=0
{
        "approvals": {
                "Org1MSP": true,
                "Org2MSP": false
        }
}
Checking the commit readiness of the chaincode definition successful on peer0.org2 on channel 'mychannel'
Using organization 2
+ peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/tlsca/tlsca.example.com-cert.pem --channelID mychannel --name basic --version 1.0 --package-id basic_1.0:e920c6326d85a919c6f533f6e770f184da9fef953abefbac8e1a7f22a6b16c4c --sequence 1
+ res=0
2023-05-24 12:29:12.728 UTC 0001 INFO [chaincodeCmd] ClientWait -> txid [102bb66e63a8c5c288df445160508097dad890111018242dfe4d3c950229fc8a] committed with status (VALID) at localhost:9051
Chaincode definition approved on peer0.org2 on channel 'mychannel'
Using organization 1
Checking the commit readiness of the chaincode definition on peer0.org1 on channel 'mychannel'...
Attempting to check the commit readiness of the chaincode definition on peer0.org1, Retry after 3 seconds.
+ peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name basic --version 1.0 --sequence 1 --output json
+ res=0
{
        "approvals": {
                "Org1MSP": true,
                "Org2MSP": true
        }
}
Checking the commit readiness of the chaincode definition successful on peer0.org1 on channel 'mychannel'
Using organization 2
Checking the commit readiness of the chaincode definition on peer0.org2 on channel 'mychannel'...
Attempting to check the commit readiness of the chaincode definition on peer0.org2, Retry after 3 seconds.
+ peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name basic --version 1.0 --sequence 1 --output json
+ res=0
{
        "approvals": {
                "Org1MSP": true,
                "Org2MSP": true
        }
}
Checking the commit readiness of the chaincode definition successful on peer0.org2 on channel 'mychannel'
Using organization 1
Using organization 2
+ peer lifecycle chaincode commit -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/tlsca/tlsca.example.com-cert.pem --channelID mychannel --name basic --peerAddresses localhost:7051 --tlsRootCertFiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/tlsca/tlsca.org1.example.com-cert.pem --peerAddresses localhost:9051 --tlsRootCertFiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/tlsca/tlsca.org2.example.com-cert.pem --version 1.0 --sequence 1
+ res=0
2023-05-24 12:29:21.230 UTC 0001 INFO [chaincodeCmd] ClientWait -> txid [ebd65edb6b94d94a555e5bfdbcdded0ca8e551a1c8a80aaed0731b274280784f] committed with status (VALID) at localhost:9051
2023-05-24 12:29:21.266 UTC 0002 INFO [chaincodeCmd] ClientWait -> txid [ebd65edb6b94d94a555e5bfdbcdded0ca8e551a1c8a80aaed0731b274280784f] committed with status (VALID) at localhost:7051
Chaincode definition committed on channel 'mychannel'
Using organization 1
Querying chaincode definition on peer0.org1 on channel 'mychannel'...
Attempting to Query committed status on peer0.org1, Retry after 3 seconds.
+ peer lifecycle chaincode querycommitted --channelID mychannel --name basic
+ res=0
Committed chaincode definition for chaincode 'basic' on channel 'mychannel':
Version: 1.0, Sequence: 1, Endorsement Plugin: escc, Validation Plugin: vscc, Approvals: [Org1MSP: true, Org2MSP: true]
Query chaincode definition successful on peer0.org1 on channel 'mychannel'
Using organization 2
Querying chaincode definition on peer0.org2 on channel 'mychannel'...
Attempting to Query committed status on peer0.org2, Retry after 3 seconds.
+ peer lifecycle chaincode querycommitted --channelID mychannel --name basic
+ res=0
Committed chaincode definition for chaincode 'basic' on channel 'mychannel':
Version: 1.0, Sequence: 1, Endorsement Plugin: escc, Validation Plugin: vscc, Approvals: [Org1MSP: true, Org2MSP: true]
Query chaincode definition successful on peer0.org2 on channel 'mychannel'
Chaincode initialization is not required
magpie@DLTFabricer:~/go/src/github.com/hyperledger/fabric-samples/test-network$ 
```

## 4.账本交互

### 4.1 交互环境配置

以Org1的Peer0的身份进行账本交互，在命令行中输入以下指令

```bash
export PATH=${PWD}/../bin:${PWD}:$PATH
export FABRIC_CFG_PATH=$PWD/../config/

export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
```

### 4.2 创建资产

```bash
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"CreateAsset","Args":["asset6","black","20","Jack","630"]}'
```

运行结果：

```bash
magpie@DLTFabricer:~/go/src/github.com/hyperledger/fabric-samples/test-network$ peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"CreateAsset","Args":["asset6","black","20","Jack","630"]}'
2023-05-24 12:54:51.080 UTC 0001 INFO [chaincodeCmd] chaincodeInvokeOrQuery -> Chaincode invoke successful. result: status:200 
```

![image-20230524205610202](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20230524205610202.png)

### 4.3 查询创建的资产

```bash
peer chaincode query -C mychannel -n basic -c '{"Args":["ReadAsset","asset6"]}'
```

![image-20230524205650947](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20230524205650947.png)



### 4.4 更新资产

将颜色改为blue蓝色，价值改为680

```bash
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"UpdateAsset","Args":["asset6","blue","20","Jack","680"]}'
```


查询资产是否更新成功

```bash
peer chaincode query -C mychannel -n basic -c '{"Args":["ReadAsset","asset6"]}'
```

![image-20230524205746195](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20230524205746195.png)

### 4.5 查询资产是否存在

```bash
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"AssetExists","Args":["asset6"]}'
```

![image-20230524205823563](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20230524205823563.png)

### 4.6 资产转移

将asset6资产由Jack转给Michel

```bash
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"TransferAsset","Args":["asset6","Michel"]}'
```

![image-20230524205914658](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20230524205914658.png)

### 4.7 删除资产

将asset6资产删除

```bash
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"DeleteAsset","Args":["asset6"]}'
```

![image-20230524210013163](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20230524210013163.png)



————————————————
版权声明：本文为CSDN博主「余府」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/bean_business/article/details/109079641





# *Hyperledger Fabric Explorer* 浏览器部署教程

> ***Hyperledger Explorer*视频教程**
>
> https://www.bilibili.com/video/BV1yQ4y1Z7sJ/?spm_id_from=333.999.0.0&vd_source=1eecda9234223595880a3a83e762d22e
>

> Hyperledger Fabric项目搭建区块链浏览器
>
> [Hyperledger Fabric项目搭建区块链浏览器Hyperledger-blockchain-explorer_Pistachiout的博客-CSDN博客](https://blog.csdn.net/qq_45808700/article/details/130183648)

> *Hyperledger Fabric项目搭建区块链浏览器Explorer* 代码库
>
> 是一个简单、强大、易于使用、维护良好的开源工具，可以浏览底层区块链网络上的活动。 项目地址：[https://github.com/hyperledger/blockchain-explorer](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fgithub.com%2Fhyperledger%2Fblockchain-explorer)

![img](https://gitcode.net/mirrors/hyperledger/blockchain-explorer/-/raw/master/docs/source/images/Hyperledger_Explorer_Logo_Color.png)

Hyperledger Explorer是一个简单，强大，易于使用，维护良好的开源实用程序，用于浏览底层区块链网络上的活动。用户可以在MacOS和Ubuntu上配置和构建Hyperledger Explorer。

## 当前版本

| 资源管理器版本                                               | 支持的架构版本                                               | 支持的节点JS版本                                             |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **[v2.0.0](https://github.com/hyperledger-labs/blockchain-explorer/blob/main/release_notes/v2.0.0.md)** （05年2023月<>日） | [2.2、2.4 版、2.5 版](https://hyperledger-fabric.readthedocs.io/en/release-2.5) | [^12.13.1， ^14.13.1， ^16.14.1](https://nodejs.org/en/download/releases) |

------



## 下载配置文件

区块链浏览器官网：https://github.com/hyperledger/blockchain-explorer

```bash
# 根据官网来部署
# 在项目目录创建文件夹
# org1部署区块浏览器
cd ~/go/src/github.com/hyperledger
mkdir explorer
cd explorer

# 下载配置文件
wget https://raw.githubusercontent.com/hyperledger/blockchain-explorer/main/examples/net1/config.json
wget https://raw.githubusercontent.com/hyperledger/blockchain-explorer/main/examples/net1/connection-profile/test-network.json -P connection-profile
wget https://raw.githubusercontent.com/hyperledger/blockchain-explorer/main/docker-compose.yaml

# 如果虚拟机没有联网，导致下载不下来，可以在官网点击三个配置文件，自己创建相对应名称，复制并保存，
# config.json与docker-compose.yaml直接放在explorer文件夹下，
# 但注意test-network.json，需要先新建connection-profile文件夹，然后将test-network.json放入connection-profile文件夹内
```

![img](https://img-blog.csdnimg.cn/dcc4496cd5974f3794098acc2cce6db7.png)

如果虚拟机没有联网，导致下载不下来，也可以前往[Fabric-explorer](https://download.csdn.net/download/qq_45808700/87971964)进行下载，之后在本地根据需要修改，修改后上传到虚拟机

## 快速入门（使用 Docker）

### 先决条件

- docker
- docker-compose 
  - **注意（适用于v2.0.0及以上版本）：**启动 docker-compose 时，会自动从 **GHCR 而不是 Docker Hub 中提取以下 docker 映像**。
    - [Hyperledger Explorer ghcr repository](https://github.com/hyperledger-labs/blockchain-explorer/pkgs/container/explorer)
    - [Hyperledger Explorer PostgreSQL ghcr repository](https://github.com/hyperledger-labs/blockchain-explorer/pkgs/container/explorer-db)
  - **注意（适用于v1.1.8及以下版本）：**启动 docker 编写时，会自动从 Docker Hub 中提取以下 docker 映像。
    - [Hyperledger Explorer docker repository](https://hub.docker.com/r/hyperledger/explorer/)
    - [Hyperledger Explorer PostgreSQL docker repository](https://hub.docker.com/r/hyperledger/explorer-db)

### 启动Hyperledger Fabric test-network网络

本指南假设您已经按照[Hyperledger Fabric官方教程](https://hyperledger-fabric.readthedocs.io/en/latest/test_network.html)启动了测试网络。

在本指南中，我们假设您已经按照[Hyperledger Fabric官方教程](https://hyperledger-fabric.readthedocs.io/en/latest/test_network.html)启动了测试网络。

```bash
$ cd ~/go/src/github.com/hyperledger/fabric-samples/test-network
$ ./network.sh up
$ ./network.sh createChannel
$ ./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go -ccl go
```

### 配置说明

创建一个新目录（例如`explorer`)

```bash
cd ~/go/src/github.com/hyperledger
mkdir explorer
cd explorer
```



从存储库复制以下文件

- [.env](https://github.com/hyperledger/blockchain-explorer/blob/main/.env)
- [docker-compose.yaml](https://github.com/hyperledger/blockchain-explorer/blob/main/docker-compose.yaml)
- [examples/net1/connection-profile/test-network.json](https://github.com/hyperledger/blockchain-explorer/blob/main/examples/net1/connection-profile/test-network.json)
- [examples/net1/config.json](https://github.com/hyperledger/blockchain-explorer/blob/main/examples/net1/config.json)

```bash
wget https://raw.githubusercontent.com/hyperledger/blockchain-explorer/main/examples/net1/config.json
wget https://raw.githubusercontent.com/hyperledger/blockchain-explorer/main/examples/net1/connection-profile/test-network.json -P connection-profile
wget https://raw.githubusercontent.com/hyperledger/blockchain-explorer/main/docker-compose.yaml
```

复制fabric-samples/test-networ测试网络（例如 ：fabric-samples/test-network）目录下organizations目录（组织/）到当前(explorer)目录：

```bash
cp -r ../fabric-samples/test-network/organizations/ .
```

当然，如果利用.env的方式导入环境变量，不需要这一步拷贝，可以直接设置环境变量到fabric-samples/test-network/organizations/，现在，您应该具有以下文件和目录结构。

```bash
docker-compose.yaml
config.json
connection-profile/test-network.json
organizations/ordererOrganizations/
organizations/peerOrganizations/
```

### 修改test-network.json 配置

```bash
cd ~/go/src/github.com/hyperledger/explorer/connection-profile
# 先修改test-network.json文件为org1-network.json
mv test-network.json org1-network.json
# 进入修改org1-network.json中对应参数
vim org1-network.json
# 修改证书连接文件
# 将用户的证书替换为连接配置文件 （test-network.json） 中的管理员证书和机密（私钥）

#修改前
"adminPrivateKey": {
    "path": "/tmp/crypto/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp/keystore/priv_sk"
}

#修改后，将User1@org1.example.com改为Admin@org1.example.com
"adminPrivateKey": {
    "path": "/tmp/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/keystore/priv_sk"
}

# 修改完毕保存退出

# 拷贝一份并命名为org2-network.json
cp org1-network.json org2-network.json
# 修改org2-network.json中对应参数
vim org2-network.json
#将所有的Org1修改文Org2，如下所示
Org1MSP -----> Org2MSP，org1 -----> org2
# 修改完毕保存退出

#或者使用sed
sed -i "s/org1/org2/g" org2-network.json
sed -i "s/Org1/Org2/g" org2-network.json
```



### 修改config.json—多网络节点配置文件

```json
# config.json文件内只配置了一个组织的网络，所以需要添加第二个组织网络

vim config.json
# 修改为以下配置
{
  "network-configs": {
        "org1-network": {
             "name": "org1-network",
             "profile": "./connection-profile/org1-network.json"		// 对应配置文件
        },
       "org2-network": {
             "name": "org2-network",
             "profile": "./connection-profile/org2-network.json"		// 对应配置文件
                }
  },
        "license": "Apache-2.0"
}
# 修改完毕后退出

```



编辑环境变量以与您的环境保持一致`docker-compose.yaml`

```yaml
# SPDX-License-Identifier: Apache-2.0
version: '2.1'

volumes:
  pgdata:
  walletstore:


networks:
  mynetwork.com:
    name: fabric_test

services:

  explorerdb.mynetwork.com:
    image: ghcr.io/hyperledger-labs/explorer-db:latest
    container_name: explorerdb.mynetwork.com
    hostname: explorerdb.mynetwork.com
    environment:
      - DATABASE_DATABASE=fabricexplorer
      - DATABASE_USERNAME=hppoc
      - DATABASE_PASSWORD=password
    healthcheck:
      test: "pg_isready -h localhost -p 5432 -q -U postgres"
      interval: 30s
      timeout: 10s
      retries: 5
    volumes:
      - pgdata:/var/lib/postgresql/data
    networks:
      - mynetwork.com

  explorer.mynetwork.com:
    image: ghcr.io/hyperledger-labs/explorer:latest
    container_name: explorer.mynetwork.com
    hostname: explorer.mynetwork.com
    environment:
      - DATABASE_HOST=explorerdb.mynetwork.com
      - DATABASE_DATABASE=fabricexplorer
      - DATABASE_USERNAME=hppoc
      - DATABASE_PASSWD=password
      - LOG_LEVEL_APP=info
      - LOG_LEVEL_DB=info
      - LOG_LEVEL_CONSOLE=debug
      - LOG_CONSOLE_STDOUT=true
      - DISCOVERY_AS_LOCALHOST=false

    volumes:
      - ${EXPLORER_CONFIG_FILE_PATH}:/opt/explorer/app/platform/fabric/config.json
      - ${EXPLORER_PROFILE_DIR_PATH}:/opt/explorer/app/platform/fabric/connection-profile
      - ${FABRIC_CRYPTO_PATH}:/tmp/crypto
      - walletstore:/opt/explorer/wallet

    ports:
      - 8080:8080

    depends_on:
      explorerdb.mynetwork.com:
        condition: service_healthy

    networks:
      - mynetwork.com
```



另一种选择是在 shell 中导出环境变量。

```bash
export EXPLORER_CONFIG_FILE_PATH=./config.json
export EXPLORER_PROFILE_DIR_PATH=./connection-profile
export FABRIC_CRYPTO_PATH=./organizations
```



当您通过桥接网络将资源管理器连接到结构网络时，需要设置为 以禁用主机名映射到本地主机。`DISCOVERY_AS_LOCALHOST`false`

```bash
services:

  ...

  explorer.mynetwork.com:

    ...

    environment:
      - DISCOVERY_AS_LOCALHOST=false
```



将用户的证书替换为连接配置文件 （test-network.json） 中的管理员证书和机密（私钥）。需要在资源管理器容器上指定绝对路径。

以前：

```bash
	"organizations": {
		"Org1MSP": {
			"mspid": "Org1MSP",
			"adminPrivateKey": {
				"path": "/tmp/crypto/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp/keystore/priv_sk"
			},
			"peers": [
				"peer0.org1.example.com"
			],
			"signedCert": {
				"path": "/tmp/crypto/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp/signcerts/User1@org1.example.com-cert.pem"
			}
		}
	},
```



后：

```bash
	"organizations": {
		"Org1MSP": {
			"mspid": "Org1MSP",
			"adminPrivateKey": {
				"path": "/tmp/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/keystore/priv_sk"
			},
			"peers": [
				"peer0.org1.example.com"
			],
			"signedCert": {
				"path": "/tmp/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/signcerts/Admin@org1.example.com-cert.pem"
			}
		}
	},
```



### 最终目录结构

![image-20230922184159569](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20230922184159569.png)



### 所有配置文件内容

#### 1.docker-compose.yaml

```yaml
# SPDX-License-Identifier: Apache-2.0
version: '2.1'

volumes:
  pgdata:
  walletstore:


networks:
  mynetwork.com:
    name: fabric_test

services:

  explorerdb.mynetwork.com:
    image: ghcr.io/hyperledger-labs/explorer-db:latest
    container_name: explorerdb.mynetwork.com
    hostname: explorerdb.mynetwork.com
    environment:
      - DATABASE_DATABASE=fabricexplorer
      - DATABASE_USERNAME=hppoc
      - DATABASE_PASSWORD=password
    healthcheck:
      test: "pg_isready -h localhost -p 5432 -q -U postgres"
      interval: 30s
      timeout: 10s
      retries: 5
    volumes:
      - pgdata:/var/lib/postgresql/data
    networks:
      - mynetwork.com

  explorer.mynetwork.com:
    image: ghcr.io/hyperledger-labs/explorer:latest
    container_name: explorer.mynetwork.com
    hostname: explorer.mynetwork.com
    environment:
      - DATABASE_HOST=explorerdb.mynetwork.com
      - DATABASE_DATABASE=fabricexplorer
      - DATABASE_USERNAME=hppoc
      - DATABASE_PASSWD=password
      - LOG_LEVEL_APP=info
      - LOG_LEVEL_DB=info
      - LOG_LEVEL_CONSOLE=debug
      - LOG_CONSOLE_STDOUT=true
      - DISCOVERY_AS_LOCALHOST=false

    volumes:
      - ${EXPLORER_CONFIG_FILE_PATH}:/opt/explorer/app/platform/fabric/config.json
      - ${EXPLORER_PROFILE_DIR_PATH}:/opt/explorer/app/platform/fabric/connection-profile
      - ${FABRIC_CRYPTO_PATH}:/tmp/crypto
      - walletstore:/opt/explorer/wallet

    ports:
      - 8080:8080

    depends_on:
      explorerdb.mynetwork.com:
        condition: service_healthy

    networks:
      - mynetwork.com
```



#### 2.config.json

```json
{
	"network-configs": {
		"org1-network": {
			"name": "Org1-Network",
			"profile": "./connection-profile/org1-network.json"
		},
		"org2-network": {
			"name": "Org2-Network",
			"profile": "./connection-profile/org2-network.json"
		}
	},
	"license": "Apache-2.0"
}
```



#### 3.env

```bash
PORT=8080
EXPLORER_CONFIG_FILE_PATH=./config.json
EXPLORER_PROFILE_DIR_PATH=./connection-profile
FABRIC_CRYPTO_PATH=/home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations
```



#### 4.org1-network.json

```json
{
	"name": "org1-network",
	"version": "1.0.0",
	"client": {
		"tlsEnable": true,
		"adminCredential": {
			"id": "exploreradmin",
			"password": "exploreradminpw"
		},
		"enableAuthentication": true,
		"organization": "Org1MSP",
		"connection": {
			"timeout": {
				"peer": {
					"endorser": "300"
				},
				"orderer": "300"
			}
		}
	},
	"channels": {
		"mychannel": {
			"peers": {
				"peer0.org1.example.com": {}
			}
		}
	},
	"organizations": {
		"Org1MSP": {
			"mspid": "Org1MSP",
			"adminPrivateKey": {
				"path": "/tmp/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/keystore/priv_sk"
			},
			"peers": [
				"peer0.org1.example.com"
			],
			"signedCert": {
				"path": "/tmp/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/signcerts/Admin@org1.example.com-cert.pem"
			}
		}
	},
	"peers": {
		"peer0.org1.example.com": {
			"tlsCACerts": {
				"path": "/tmp/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt"
			},
			"url": "grpcs://peer0.org1.example.com:7051"
		}
	}
}
```



#### 5.org2-network.json

```json
{
	"name": "org2-network",
	"version": "1.0.0",
	"client": {
		"tlsEnable": true,
		"adminCredential": {
			"id": "exploreradmin",
			"password": "exploreradminpw"
		},
		"enableAuthentication": true,
		"organization": "Org2MSP",
		"connection": {
			"timeout": {
				"peer": {
					"endorser": "300"
				},
				"orderer": "300"
			}
		}
	},
	"channels": {
		"mychannel": {
			"peers": {
				"peer0.org2.example.com": {}
			}
		}
	},
	"organizations": {
		"Org2MSP": {
			"mspid": "Org2MSP",
			"adminPrivateKey": {
				"path": "/tmp/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp/keystore/priv_sk"
			},
			"peers": [
				"peer0.org2.example.com"
			],
			"signedCert": {
				"path": "/tmp/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp/signcerts/Admin@org2.example.com-cert.pem"
			}
		}
	},
	"peers": {
		"peer0.org2.example.com": {
			"tlsCACerts": {
				"path": "/tmp/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt"
			},
			"url": "grpcs://peer0.org2.example.com:9051"
		}
	}
}
```





### 启动容器服务

- 运行以下命令以在启动结构网络后启动探索和资源管理器数据库服务：

  ```bash
  $ docker-compose up -d
  ```

  

### 收拾

- 若要在不删除持久性数据的情况下停止服务，请运行以下命令：

  ```bash
  $ docker-compose down
  ```

  

- 在docker-compose.yaml中，为持久数据分配了两个命名卷（用于Postgres数据和用户钱包）。如果要清除这些命名卷，请运行以下命令：

  ```bash
  $ docker-compose down -v
  ```

  

# *Hyperledger Fabric 2.4* 单机容器部署多*3-Orderer-4Org-2Peer-4User*网络

## 前言

本文主要搭建一个3 *ordere*r节点、4个*org*组织，其中每个组织各2个*peer*节点的*fabric* 2.4.6网络，共识集群采用的3-*etcd raft*协议，4个用户（Users）节点

## 1.相关环境的安装与准备

这里的环境主要包括以下内容：

> - *go*
> - *git*
> - *docker*
> - *docker-compose*
> - *jq*
> - *fabric、fabric-ca*以及fabric-sample相关docker镜像和相关二进制文件
>   详情可以参考另一篇文章：fabric基础网络环境准备与搭建。

注：下面的操作都是基于上面文章里面的软件版本来进行操作，请尽量确保版本一致，否则可能会出现不一样的错误。



| 节点       | 宿主机 IP      | hosts                  | 端口                       |
| ---------- | -------------- | ---------------------- | -------------------------- |
| orderer0   | 192.168.31.188 | orderer0.example.com   | 7050 , 8443 , 9443         |
| orderer1   | 192.168.31.188 | orderer1.example.com   | 8050 , 8444 ,9444          |
| orderer2   | 192.168.31.188 | orderer2.example.com   | 9050 , 8445 ，9445         |
| peer0.org1 | 192.168.31.188 | peer0.org1.example.com | 7051 , 7052 , 9446 , 8125  |
| peer1.org1 | 192.168.31.188 | peer1.org1.example.com | 8051 , 7053 , 9447 , 8126  |
| peer0.org2 | 192.168.31.188 | peer0.org2.example.com | 9051 , 7054 , 9448 , 8127  |
| peer1.org2 | 192.168.31.188 | peer1.org2.example.com | 10051 , 7055 , 9449 , 8128 |
| peer0.org3 | 192.168.31.188 | peer0.org3.example.com |                            |
| peer1.org3 | 192.168.31.188 | peer0.org3.example.com |                            |
| peer0.org4 | 192.168.31.188 | peer0.org4.example.com |                            |
| peer1.org4 | 192.168.31.188 | peer1.org4.example.com |                            |
| cli1.org1  | 192.168.31.188 | User1@org1.example.com |                            |
| cli2.org2  | 192.168.31.188 | User1@org2.example.com |                            |
| cli3.org3  | 192.168.31.188 | User1@org3.example.com |                            |
| cli4.org4  | 192.168.31.188 | User1@org4.example.com |                            |
| Admin.org1 | 192.168.31.188 | Admin@org1.example.com |                            |
| Admin.org2 | 192.168.31.188 | Admin@org2.example.com |                            |
| Admin.org3 | 192.168.31.188 | Admin@org3.example.com |                            |
| Admin.org4 | 192.168.31.188 | Admin@org4.example.com |                            |



## 2.生成相关的证书材料

### 2.1 建立项目目录

```sh
$ mkdir raft-test  
$ cd raft-test
```

### 2.2 编写配置文件

生成组织证书配置模板

```bash
$ cryptogen showtemplate > crypto-config.yaml
```

在*raft-test*目录下，新建*crypto-config.yaml*文件。文件内容如下面所示。

*crypto-config.yaml*文件内容

```yaml
# ---------------------------------------------------------------------------
# "OrdererOrgs" - Definition of organizations managing orderer nodes
# ---------------------------------------------------------------------------
OrdererOrgs:
  # ---------------------------------------------------------------------------
  # Orderer
  # ---------------------------------------------------------------------------
  - Name: Orderer
    Domain: example.com
    EnableNodeOUs: true

    # ---------------------------------------------------------------------------
    # "Specs" - See PeerOrgs below for complete description
    # ---------------------------------------------------------------------------
    Specs:
      - Hostname: orderer0
      - Hostname: orderer1
      - Hostname: orderer2

# ---------------------------------------------------------------------------
# "PeerOrgs" - Definition of organizations managing peer nodes
# ---------------------------------------------------------------------------
PeerOrgs:
  # ---------------------------------------------------------------------------
  # Org1
  # ---------------------------------------------------------------------------
  - Name: Org1
    Domain: org1.supervisor.com
    EnableNodeOUs: true

    # ---------------------------------------------------------------------------
    # "CA"
    # ---------------------------------------------------------------------------
    # Uncomment this section to enable the explicit definition of the CA for this
    # organization.  This entry is a Spec.  See "Specs" section below for details.
    # ---------------------------------------------------------------------------
    # CA:
    #    Hostname: ca # implicitly ca.org1.example.com
    #    Country: US
    #    Province: California
    #    Locality: San Francisco
    #    OrganizationalUnit: Hyperledger Fabric
    #    StreetAddress: address for org # default nil
    #    PostalCode: postalCode for org # default nil

    # ---------------------------------------------------------------------------
    # "Specs"
    # ---------------------------------------------------------------------------
    # Uncomment this section to enable the explicit definition of hosts in your
    # configuration.  Most users will want to use Template, below
    #
    # Specs is an array of Spec entries.  Each Spec entry consists of two fields:
    #   - Hostname:   (Required) The desired hostname, sans the domain.
    #   - CommonName: (Optional) Specifies the template or explicit override for
    #                 the CN.  By default, this is the template:
    #
    #                              "{{.Hostname}}.{{.Domain}}"
    #
    #                 which obtains its values from the Spec.Hostname and
    #                 Org.Domain, respectively.
    #   - SANS:       (Optional) Specifies one or more Subject Alternative Names
    #                 to be set in the resulting x509. Accepts template
    #                 variables {{.Hostname}}, {{.Domain}}, {{.CommonName}}. IP
    #                 addresses provided here will be properly recognized. Other
    #                 values will be taken as DNS names.
    #                 NOTE: Two implicit entries are created for you:
    #                     - {{ .CommonName }}
    #                     - {{ .Hostname }}
    # ---------------------------------------------------------------------------
    # Specs:
    #   - Hostname: foo # implicitly "foo.org1.example.com"
    #     CommonName: foo27.org5.example.com # overrides Hostname-based FQDN set above
    #     SANS:
    #       - "bar.{{.Domain}}"
    #       - "altfoo.{{.Domain}}"
    #       - "{{.Hostname}}.org6.net"
    #       - 172.16.10.31
    #   - Hostname: bar
    #   - Hostname: baz

    # ---------------------------------------------------------------------------
    # "Template"
    # ---------------------------------------------------------------------------
    # Allows for the definition of 1 or more hosts that are created sequentially
    # from a template. By default, this looks like "peer%d" from 0 to Count-1.
    # You may override the number of nodes (Count), the starting index (Start)
    # or the template used to construct the name (Hostname).
    #
    # Note: Template and Specs are not mutually exclusive.  You may define both
    # sections and the aggregate nodes will be created for you.  Take care with
    # name collisions
    # ---------------------------------------------------------------------------
    Template:
      Count: 2
      # Start: 5
      # Hostname: {{.Prefix}}{{.Index}} # default
      # SANS:
      #   - "{{.Hostname}}.alt.{{.Domain}}"

    # ---------------------------------------------------------------------------
    # "Users"
    # ---------------------------------------------------------------------------
    # Count: The number of user accounts _in addition_ to Admin
    # ---------------------------------------------------------------------------
    Users:
      Count: 1

  # ---------------------------------------------------------------------------
  # Org2: See "Org1" for full specification
  # ---------------------------------------------------------------------------
  - Name: Org2
    Domain: org2.build.com
    EnableNodeOUs: true
    Template:
      Count: 2
    Users:
      Count: 1

  - Name: Org3
    Domain: org3.supplier.com
    EnableNodeOUs: true
    Template:
      Count: 2
    Users:
      Count: 1

  - Name: Org4
    Domain: org4.logistics.com
    EnableNodeOUs: true
    Template:
      Count: 2
    Users:
      Count: 1
```

*Name*和*Domain*就是关于这个组织的名字和域名，这主要是用于生成证书的时候，证书内会包含该信息。而Template Count=2是说要生成2套公私钥和证书，一套是*peer0.org2*的，还有一套是*peer1.org2*的。最后*Users.* Count=1是说每个Template下面会有几个普通User（注意，*Admin*是*Admin*，不包含在这个计数中），这里配置了1，也就是说我们只需要一个普通用户[User1@org2.example.com](mailto:User1@org2.example.com) 我们可以根据实际需要调整这个配置文件，增删*Org Users*等。

### 2.3 生成组织证书

根据配置文件crypto-config.yaml来生成证书，请确保已经将生成的fabric二进制命令所在的bin目录加到环境变量，否则无法执行下面的命令。执行完后生成的证书放在了./crypto-config/

```bash
$ cryptogen generate --config=crypto-config.yaml

#或
$ cryptogen generate --config=crypto-config.yaml --output=organizations
    
org1.supervisor.com
org2.build.com
org3.supplier.com
org4.logistics.com

$ ll

drwxr-xr-x. 4 root root 4096 5月   5 19:24 crypto-config
-rw-r--r--. 1 root root 5760 5月   5 19:23 crypto-config.yaml
```

![image-20221118231052100](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221118231052100.png)



## 3.生成相关的通道配置文件

### 3.1编辑配置文件

在*raft-test*目录下，新建*configtx.yaml*文件。文件内容如下面所示。

> *configtx.yaml文件内容*

```yaml
# Copyright IBM Corp. All Rights Reserved.
#
# SPDX-License-Identifier: Apache-2.0
#

---
################################################################################
#
#   Section: Organizations
#
#   - This section defines the different organizational identities which will
#   be referenced later in the configuration.
#
################################################################################
Organizations:

    # SampleOrg defines an MSP using the sampleconfig.  It should never be used
    # in production but may be used as a template for other definitions
    - &OrdererOrg
        # DefaultOrg defines the organization which is used in the sampleconfig
        # of the fabric.git development environment
        Name: OrdererOrg

        # ID to load the MSP definition as
        ID: OrdererMSP

        # MSPDir is the filesystem path which contains the MSP configuration
        MSPDir: crypto-config/ordererOrganizations/example.com/msp

        # Policies defines the set of policies at this level of the config tree
        # For organization policies, their canonical path is usually
        #   /Channel/<Application|Orderer>/<OrgName>/<PolicyName>
        Policies:
            Readers:
                Type: Signature
                Rule: "OR('OrdererMSP.member')"
            Writers:
                Type: Signature
                Rule: "OR('OrdererMSP.member')"
            Admins:
                Type: Signature
                Rule: "OR('OrdererMSP.admin')"

    - &Org1
        # DefaultOrg defines the organization which is used in the sampleconfig
        # of the fabric.git development environment
        Name: Org1

        # ID to load the MSP definition as
        ID: Org1MSP

        MSPDir: crypto-config/peerOrganizations/org1.supervisor.com/msp

        # Policies defines the set of policies at this level of the config tree
        # For organization policies, their canonical path is usually
        #   /Channel/<Application|Orderer>/<OrgName>/<PolicyName>
        Policies:
            Readers:
                Type: Signature
                Rule: "OR('Org1MSP.admin', 'Org1MSP.peer', 'Org1MSP.client')"
            Writers:
                Type: Signature
                Rule: "OR('Org1MSP.admin', 'Org1MSP.client')"
            Admins:
                Type: Signature
                Rule: "OR('Org1MSP.admin')"
            Endorsement:
                Type: Signature
                Rule: "OR('Org1MSP.peer')"

        # leave this flag set to true.
        AnchorPeers:
            # AnchorPeers defines the location of peers which can be used
            # for cross org gossip communication.  Note, this value is only
            # encoded in the genesis block in the Application section context
            - Host: peer0.org1.supervisor.com
              Port: 7051

    - &Org2
        # DefaultOrg defines the organization which is used in the sampleconfig
        # of the fabric.git development environment
        Name: Org2

        # ID to load the MSP definition as
        ID: Org2MSP

        MSPDir: crypto-config/peerOrganizations/org2.build.com/msp

        # Policies defines the set of policies at this level of the config tree
        # For organization policies, their canonical path is usually
        #   /Channel/<Application|Orderer>/<OrgName>/<PolicyName>
        Policies:
            Readers:
                Type: Signature
                Rule: "OR('Org2MSP.admin', 'Org2MSP.peer', 'Org2MSP.client')"
            Writers:
                Type: Signature
                Rule: "OR('Org2MSP.admin', 'Org2MSP.client')"
            Admins:
                Type: Signature
                Rule: "OR('Org2MSP.admin')"
            Endorsement:
                Type: Signature
                Rule: "OR('Org2MSP.peer')"

        AnchorPeers:
            # AnchorPeers defines the location of peers which can be used
            # for cross org gossip communication.  Note, this value is only
            # encoded in the genesis block in the Application section context
            - Host: peer0.org2.build.com
              Port: 9051

    - &Org3
        # DefaultOrg defines the organization which is used in the sampleconfig
        # of the fabric.git development environment
        Name: Org3

        # ID to load the MSP definition as
        ID: Org3MSP

        MSPDir: crypto-config/peerOrganizations/org3.supplier.com/msp

        # Policies defines the set of policies at this level of the config tree
        # For organization policies, their canonical path is usually
        #   /Channel/<Application|Orderer>/<OrgName>/<PolicyName>
        Policies:
            Readers:
                Type: Signature
                Rule: "OR('Org3MSP.admin', 'Org3MSP.peer', 'Org3MSP.client')"
            Writers:
                Type: Signature
                Rule: "OR('Org3MSP.admin', 'Org3MSP.client')"
            Admins:
                Type: Signature
                Rule: "OR('Org3MSP.admin')"
            Endorsement:
                Type: Signature
                Rule: "OR('Org3MSP.peer')"

        AnchorPeers:
            # AnchorPeers defines the location of peers which can be used
            # for cross org gossip communication.  Note, this value is only
            # encoded in the genesis block in the Application section context
            - Host: peer0.org3.supplier.com
              Port: 11051

    - &Org4
        # DefaultOrg defines the organization which is used in the sampleconfig
        # of the fabric.git development environment
        Name: Org4

        # ID to load the MSP definition as
        ID: Org4MSP

        MSPDir: crypto-config/peerOrganizations/org4.logistics.com/msp

        # Policies defines the set of policies at this level of the config tree
        # For organization policies, their canonical path is usually
        #   /Channel/<Application|Orderer>/<OrgName>/<PolicyName>
        Policies:
            Readers:
                Type: Signature
                Rule: "OR('Org4MSP.admin', 'Org4MSP.peer', 'Org4MSP.client')"
            Writers:
                Type: Signature
                Rule: "OR('Org4MSP.admin', 'Org4MSP.client')"
            Admins:
                Type: Signature
                Rule: "OR('Org4MSP.admin')"
            Endorsement:
                Type: Signature
                Rule: "OR('Org4MSP.peer')"

        AnchorPeers:
            # AnchorPeers defines the location of peers which can be used
            # for cross org gossip communication.  Note, this value is only
            # encoded in the genesis block in the Application section context
            - Host: peer0.org4.logistics.com
              Port: 13051

################################################################################
#
#   SECTION: Capabilities
#
#   - This section defines the capabilities of fabric network. This is a new
#   concept as of v1.1.0 and should not be utilized in mixed networks with
#   v1.0.x peers and orderers.  Capabilities define features which must be
#   present in a fabric binary for that binary to safely participate in the
#   fabric network.  For instance, if a new MSP type is added, newer binaries
#   might recognize and validate the signatures from this type, while older
#   binaries without this support would be unable to validate those
#   transactions.  This could lead to different versions of the fabric binaries
#   having different world states.  Instead, defining a capability for a channel
#   informs those binaries without this capability that they must cease
#   processing transactions until they have been upgraded.  For v1.0.x if any
#   capabilities are defined (including a map with all capabilities turned off)
#   then the v1.0.x peer will deliberately crash.
#
################################################################################
Capabilities:
    # Channel capabilities apply to both the orderers and the peers and must be
    # supported by both.
    # Set the value of the capability to true to require it.
    Channel: &ChannelCapabilities
        # V2_0 capability ensures that orderers and peers behave according
        # to v2.0 channel capabilities. Orderers and peers from
        # prior releases would behave in an incompatible way, and are therefore
        # not able to participate in channels at v2.0 capability.
        # Prior to enabling V2.0 channel capabilities, ensure that all
        # orderers and peers on a channel are at v2.0.0 or later.
        V2_0: true

    # Orderer capabilities apply only to the orderers, and may be safely
    # used with prior release peers.
    # Set the value of the capability to true to require it.
    Orderer: &OrdererCapabilities
        # V2_0 orderer capability ensures that orderers behave according
        # to v2.0 orderer capabilities. Orderers from
        # prior releases would behave in an incompatible way, and are therefore
        # not able to participate in channels at v2.0 orderer capability.
        # Prior to enabling V2.0 orderer capabilities, ensure that all
        # orderers on channel are at v2.0.0 or later.
        V2_0: true

    # Application capabilities apply only to the peer network, and may be safely
    # used with prior release orderers.
    # Set the value of the capability to true to require it.
    Application: &ApplicationCapabilities
        # V2_0 application capability ensures that peers behave according
        # to v2.0 application capabilities. Peers from
        # prior releases would behave in an incompatible way, and are therefore
        # not able to participate in channels at v2.0 application capability.
        # Prior to enabling V2.0 application capabilities, ensure that all
        # peers on channel are at v2.0.0 or later.
        V2_0: true

################################################################################
#
#   SECTION: Application
#
#   - This section defines the values to encode into a config transaction or
#   genesis block for application related parameters
#
################################################################################
Application: &ApplicationDefaults

    # Organizations is the list of orgs which are defined as participants on
    # the application side of the network
    Organizations:

    # Policies defines the set of policies at this level of the config tree
    # For Application policies, their canonical path is
    #   /Channel/Application/<PolicyName>
    Policies:
        Readers:
            Type: ImplicitMeta
            Rule: "ANY Readers"
        Writers:
            Type: ImplicitMeta
            Rule: "ANY Writers"
        Admins:
            Type: ImplicitMeta
            Rule: "MAJORITY Admins"
        LifecycleEndorsement:
            Type: ImplicitMeta
            Rule: "MAJORITY Endorsement"
        Endorsement:
            Type: ImplicitMeta
            Rule: "MAJORITY Endorsement"

    Capabilities:
        <<: *ApplicationCapabilities
################################################################################
#
#   SECTION: Orderer
#
#   - This section defines the values to encode into a config transaction or
#   genesis block for orderer related parameters
#
################################################################################
Orderer: &OrdererDefaults

    # Orderer Type: The orderer implementation to start
    OrdererType: etcdraft
    
    # Addresses used to be the list of orderer addresses that clients and peers
    # could connect to.  However, this does not allow clients to associate orderer
    # addresses and orderer organizations which can be useful for things such
    # as TLS validation.  The preferred way to specify orderer addresses is now
    # to include the OrdererEndpoints item in your org definition
    Addresses:
        - orderer0.example.com:7050
        - orderer1.example.com:8050
        - orderer2.example.com:9050

    EtcdRaft:
        Consenters:
        - Host: orderer0.example.com
          Port: 7050
          ClientTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt
          ServerTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt

    # Batch Timeout: The amount of time to wait before creating a batch
    BatchTimeout: 2s

    # Batch Size: Controls the number of messages batched into a block
    BatchSize:

        # Max Message Count: The maximum number of messages to permit in a batch
        MaxMessageCount: 10

        # Absolute Max Bytes: The absolute maximum number of bytes allowed for
        # the serialized messages in a batch.
        AbsoluteMaxBytes: 99 MB

        # Preferred Max Bytes: The preferred maximum number of bytes allowed for
        # the serialized messages in a batch. A message larger than the preferred
        # max bytes will result in a batch larger than preferred max bytes.
        PreferredMaxBytes: 512 KB

    # Organizations is the list of orgs which are defined as participants on
    # the orderer side of the network
    Organizations:

    # Policies defines the set of policies at this level of the config tree
    # For Orderer policies, their canonical path is
    #   /Channel/Orderer/<PolicyName>
    Policies:
        Readers:
            Type: ImplicitMeta
            Rule: "ANY Readers"
        Writers:
            Type: ImplicitMeta
            Rule: "ANY Writers"
        Admins:
            Type: ImplicitMeta
            Rule: "MAJORITY Admins"
        # BlockValidation specifies what signatures must be included in the block
        # from the orderer for the peer to validate it.
        BlockValidation:
            Type: ImplicitMeta
            Rule: "ANY Writers"

################################################################################
#
#   CHANNEL
#
#   This section defines the values to encode into a config transaction or
#   genesis block for channel related parameters.
#
################################################################################
Channel: &ChannelDefaults
    # Policies defines the set of policies at this level of the config tree
    # For Channel policies, their canonical path is
    #   /Channel/<PolicyName>
    Policies:
        # Who may invoke the 'Deliver' API
        Readers:
            Type: ImplicitMeta
            Rule: "ANY Readers"
        # Who may invoke the 'Broadcast' API
        Writers:
            Type: ImplicitMeta
            Rule: "ANY Writers"
        # By default, who may modify elements at this config level
        Admins:
            Type: ImplicitMeta
            Rule: "MAJORITY Admins"

    # Capabilities describes the channel level capabilities, see the
    # dedicated Capabilities section elsewhere in this file for a full
    # description
    Capabilities:
        <<: *ChannelCapabilities

################################################################################
#
#   Profile
#
#   - Different configuration profiles may be encoded here to be specified
#   as parameters to the configtxgen tool
#
################################################################################
Profiles:

    FourOrgsOrdererGenesis:
        <<: *ChannelDefaults
        Orderer:
            <<: *OrdererDefaults
            OrdererType: etcdraft
            EtcdRaft:
                Consenters:
                - Host: orderer0.example.com
                  Port: 7050
                  ClientTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer0.example.com/tls/server.crt
                  ServerTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer0.example.com/tls/server.crt
                - Host: orderer1.example.com
                  Port: 8050
                  ClientTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer1.example.com/tls/server.crt
                  ServerTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer1.example.com/tls/server.crt
                - Host: orderer2.example.com
                  Port: 9050
                  ClientTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer2.example.com/tls/server.crt
                  ServerTLSCert: crypto-config/ordererOrganizations/example.com/orderers/orderer2.example.com/tls/server.crt
            Addresses:
                - orderer0.example.com:7050
                - orderer1.example.com:8050
                - orderer2.example.com:9050
            Organizations:
                - *OrdererOrg
            Capabilities:
                <<: *OrdererCapabilities
        Consortiums:
            SampleConsortium:
                Organizations:
                    - *Org1
                    - *Org2
                    - *Org3
                    - *Org4

    FourOrgsChannel:
        Consortium: SampleConsortium
        <<: *ChannelDefaults
        Application:
            <<: *ApplicationDefaults
            Organizations:
                - *Org1
                - *Org2
                - *Org3
                - *Org4
            Capabilities:
                <<: *ApplicationCapabilities
```



### 3.2 生成创世区块文件

> 特别提醒：创世区块通道名称*channelID*采用*syschannel*

```sh
$ configtxgen -profile FourOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block -channelID syschannel
```

![image-20221118232255754](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221118232255754.png)

> 备注：创世通道*channelID* 采用*fabric-channel*名称，与下面通道名称不能相同，下面采用appchannel

### 3.3 生成通道文件

```sh
$ configtxgen -profile FourOrgsChannel -outputCreateChannelTx ./channel-artifacts/appchannel.tx -channelID appchannel
```

![image-20221118232431791](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221118232431791.png)



### 3.4 创建更新锚节点的配置更新

因为联盟有4个组织，所以本例中需要对每个组织分别调用一次

```sh
$ configtxgen -profile FourOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID appchannel -asOrg Org1

$ configtxgen -profile FourOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID appchannel -asOrg Org2

$ configtxgen -profile FourOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org3MSPanchors.tx -channelID appchannel -asOrg Org3

$ configtxgen -profile FourOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org4MSPanchors.tx -channelID appchannel -asOrg Org4
```



![image-20221118233540204](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221118233540204.png)



### 3.5 查看是否成功

```sh
$ ll channel-artifacts
```

![image-20221118233647428](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221118233647428.png)



> ## 如何查看创世区块 Genesis Block和channel.tx文件
>
> ### 一、查看Block文件
>
> 1.生成创世区块（如果前面实验没有创建genesis.block，可以按照此命令产生）
>
> ```bash
> $ configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block
> ```
>
>
> 生成的文件位于目录 channel-artifacts 下。
>
> 2.将 Block 详细内容导入到 json 文件查看
>
> ```bash
> $ configtxgen -inspectBlock channel-artifacts/genesis.block > genesis.block.json
> ```
>
> ![img](https://img-blog.csdnimg.cn/20190228101311587.png)
>
> ![image-20221120171803254](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221120171803254.png)
>
> 账本创世区块结构：
>
> ![image-20221123141307373](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221123141307373.png)
>
> ![image-20221122204509087](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221122204509087.png)
>
> ```json
> {
> 	"data": {
> 		"data": [
> 			{
> 				"payload": {
> 					"data": {
> 						"config": {
> 							"channel_group": {
> 								"groups": {
> 									"Consortiums": {
> 										"groups": {
> 											"SampleConsortium": {
> 												"groups": {
> 													"Org1MSP": {
> 														"groups": {},
> 														"mod_policy": "Admins",
> 														"policies": {
> 															"Admins": {
> 																"mod_policy": "Admins",
> 																"policy": {
> 																	"type": 1,
> 																	"value": {
> 																		"identities": [
> 																			{
> 																				"principal": {
> 																					"msp_identifier": "Org1MSP",
> 																					"role": "ADMIN"
> 																				},
> 																				"principal_classification": "ROLE"
> 																			}
> 																		],
> 																		"rule": {
> 																			"n_out_of": {
> 																				"n": 1,
> 																				"rules": [
> 																					{
> 																						"signed_by": 0
> 																					}
> 																				]
> 																			}
> 																		},
> 																		"version": 0
> 																	}
> 																},
> 																"version": "0"
> 															},
> 															"Endorsement": {
> 																"mod_policy": "Admins",
> 																"policy": {
> 																	"type": 1,
> 																	"value": {
> 																		"identities": [
> 																			{
> 																				"principal": {
> 																					"msp_identifier": "Org1MSP",
> 																					"role": "PEER"
> 																				},
> 																				"principal_classification": "ROLE"
> 																			}
> 																		],
> 																		"rule": {
> 																			"n_out_of": {
> 																				"n": 1,
> 																				"rules": [
> 																					{
> 																						"signed_by": 0
> 																					}
> 																				]
> 																			}
> 																		},
> 																		"version": 0
> 																	}
> 																},
> 																"version": "0"
> 															},
> 															"Readers": {
> 																"mod_policy": "Admins",
> 																"policy": {
> 																	"type": 1,
> 																	"value": {
> 																		"identities": [
> 																			{
> 																				"principal": {
> 																					"msp_identifier": "Org1MSP",
> 																					"role": "ADMIN"
> 																				},
> 																				"principal_classification": "ROLE"
> 																			},
> 																			{
> 																				"principal": {
> 																					"msp_identifier": "Org1MSP",
> 																					"role": "PEER"
> 																				},
> 																				"principal_classification": "ROLE"
> 																			},
> 																			{
> 																				"principal": {
> 																					"msp_identifier": "Org1MSP",
> 																					"role": "CLIENT"
> 																				},
> 																				"principal_classification": "ROLE"
> 																			}
> 																		],
> 																		"rule": {
> 																			"n_out_of": {
> 																				"n": 1,
> 																				"rules": [
> 																					{
> 																						"signed_by": 0
> 																					},
> 																					{
> 																						"signed_by": 1
> 																					},
> 																					{
> 																						"signed_by": 2
> 																					}
> 																				]
> 																			}
> 																		},
> 																		"version": 0
> 																	}
> 																},
> 																"version": "0"
> 															},
> 															"Writers": {
> 																"mod_policy": "Admins",
> 																"policy": {
> 																	"type": 1,
> 																	"value": {
> 																		"identities": [
> 																			{
> 																				"principal": {
> 																					"msp_identifier": "Org1MSP",
> 																					"role": "ADMIN"
> 																				},
> 																				"principal_classification": "ROLE"
> 																			},
> 																			{
> 																				"principal": {
> 																					"msp_identifier": "Org1MSP",
> 																					"role": "CLIENT"
> 																				},
> 																				"principal_classification": "ROLE"
> 																			}
> 																		],
> 																		"rule": {
> 																			"n_out_of": {
> 																				"n": 1,
> 																				"rules": [
> 																					{
> 																						"signed_by": 0
> 																					},
> 																					{
> 																						"signed_by": 1
> 																					}
> 																				]
> 																			}
> 																		},
> 																		"version": 0
> 																	}
> 																},
> 																"version": "0"
> 															}
> 														},
> 														"values": {
> 															"MSP": {
> 																"mod_policy": "Admins",
> 																"value": {
> 																	"config": {
> 																		"admins": [],
> 																		"crypto_config": {
> 																			"identity_identifier_hash_function": "SHA256",
> 																			"signature_hash_family": "SHA2"
> 																		},
> 																		"fabric_node_ous": {
> 																			"admin_ou_identifier": {
> 																				"certificate": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNYRENDQWdPZ0F3SUJBZ0lRYitha2dLcUIyWURrRkRxaWVSUFhCVEFLQmdncWhrak9QUVFEQWpCNU1Rc3cKQ1FZRFZRUUdFd0pWVXpFVE1CRUdBMVVFQ0JNS1EyRnNhV1p2Y201cFlURVdNQlFHQTFVRUJ4TU5VMkZ1SUVaeQpZVzVqYVhOamJ6RWNNQm9HQTFVRUNoTVRiM0puTVM1emRYQmxjblpwYzI5eUxtTnZiVEVmTUIwR0ExVUVBeE1XClkyRXViM0puTVM1emRYQmxjblpwYzI5eUxtTnZiVEFlRncweU1qRXhNVGd4TlRBMU1EQmFGdzB6TWpFeE1UVXgKTlRBMU1EQmFNSGt4Q3pBSkJnTlZCQVlUQWxWVE1STXdFUVlEVlFRSUV3cERZV3hwWm05eWJtbGhNUll3RkFZRApWUVFIRXcxVFlXNGdSbkpoYm1OcGMyTnZNUnd3R2dZRFZRUUtFeE52Y21jeExuTjFjR1Z5ZG1semIzSXVZMjl0Ck1SOHdIUVlEVlFRREV4WmpZUzV2Y21jeExuTjFjR1Z5ZG1semIzSXVZMjl0TUZrd0V3WUhLb1pJemowQ0FRWUkKS29aSXpqMERBUWNEUWdBRTdGdENrQ2pXWVNHU0hJQnZsUWdYaXB5UEdGNjhNaHFjY3NmMm56dEdqMEtseGxzUwpteTcvVXFIanhybzM4UlFQaWJMZTdxL1BxcTlBanVjTFBjSXZzNk50TUdzd0RnWURWUjBQQVFIL0JBUURBZ0dtCk1CMEdBMVVkSlFRV01CUUdDQ3NHQVFVRkJ3TUNCZ2dyQmdFRkJRY0RBVEFQQmdOVkhSTUJBZjhFQlRBREFRSC8KTUNrR0ExVWREZ1FpQkNBQzdNRFdhQWttZ3Z4OGRrcEJ2UkthS0FDY3pMYnVlWmFKNU9YVFRidU1RREFLQmdncQpoa2pPUFFRREFnTkhBREJFQWlCaVEzRDhOMklHdWxFVVRMS3FjNTgzU29EMWVkbVBMZTBVeDRBQ0dROXpSd0lnClhnVUczSzhCbklETk5Gcjd3enl1MlhXOW94Z3VvaHI4TmpzS3hnSGxqbmM9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K",
> 																				"organizational_unit_identifier": "admin"
> 																			},
> 																			"client_ou_identifier": {
> 																				"certificate": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNYRENDQWdPZ0F3SUJBZ0lRYitha2dLcUIyWURrRkRxaWVSUFhCVEFLQmdncWhrak9QUVFEQWpCNU1Rc3cKQ1FZRFZRUUdFd0pWVXpFVE1CRUdBMVVFQ0JNS1EyRnNhV1p2Y201cFlURVdNQlFHQTFVRUJ4TU5VMkZ1SUVaeQpZVzVqYVhOamJ6RWNNQm9HQTFVRUNoTVRiM0puTVM1emRYQmxjblpwYzI5eUxtTnZiVEVmTUIwR0ExVUVBeE1XClkyRXViM0puTVM1emRYQmxjblpwYzI5eUxtTnZiVEFlRncweU1qRXhNVGd4TlRBMU1EQmFGdzB6TWpFeE1UVXgKTlRBMU1EQmFNSGt4Q3pBSkJnTlZCQVlUQWxWVE1STXdFUVlEVlFRSUV3cERZV3hwWm05eWJtbGhNUll3RkFZRApWUVFIRXcxVFlXNGdSbkpoYm1OcGMyTnZNUnd3R2dZRFZRUUtFeE52Y21jeExuTjFjR1Z5ZG1semIzSXVZMjl0Ck1SOHdIUVlEVlFRREV4WmpZUzV2Y21jeExuTjFjR1Z5ZG1semIzSXVZMjl0TUZrd0V3WUhLb1pJemowQ0FRWUkKS29aSXpqMERBUWNEUWdBRTdGdENrQ2pXWVNHU0hJQnZsUWdYaXB5UEdGNjhNaHFjY3NmMm56dEdqMEtseGxzUwpteTcvVXFIanhybzM4UlFQaWJMZTdxL1BxcTlBanVjTFBjSXZzNk50TUdzd0RnWURWUjBQQVFIL0JBUURBZ0dtCk1CMEdBMVVkSlFRV01CUUdDQ3NHQVFVRkJ3TUNCZ2dyQmdFRkJRY0RBVEFQQmdOVkhSTUJBZjhFQlRBREFRSC8KTUNrR0ExVWREZ1FpQkNBQzdNRFdhQWttZ3Z4OGRrcEJ2UkthS0FDY3pMYnVlWmFKNU9YVFRidU1RREFLQmdncQpoa2pPUFFRREFnTkhBREJFQWlCaVEzRDhOMklHdWxFVVRMS3FjNTgzU29EMWVkbVBMZTBVeDRBQ0dROXpSd0lnClhnVUczSzhCbklETk5Gcjd3enl1MlhXOW94Z3VvaHI4TmpzS3hnSGxqbmM9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K",
> 																				"organizational_unit_identifier": "client"
> 																			},
> 																			"enable": true,
> 																			"orderer_ou_identifier": {
> 																				"certificate": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNYRENDQWdPZ0F3SUJBZ0lRYitha2dLcUIyWURrRkRxaWVSUFhCVEFLQmdncWhrak9QUVFEQWpCNU1Rc3cKQ1FZRFZRUUdFd0pWVXpFVE1CRUdBMVVFQ0JNS1EyRnNhV1p2Y201cFlURVdNQlFHQTFVRUJ4TU5VMkZ1SUVaeQpZVzVqYVhOamJ6RWNNQm9HQTFVRUNoTVRiM0puTVM1emRYQmxjblpwYzI5eUxtTnZiVEVmTUIwR0ExVUVBeE1XClkyRXViM0puTVM1emRYQmxjblpwYzI5eUxtTnZiVEFlRncweU1qRXhNVGd4TlRBMU1EQmFGdzB6TWpFeE1UVXgKTlRBMU1EQmFNSGt4Q3pBSkJnTlZCQVlUQWxWVE1STXdFUVlEVlFRSUV3cERZV3hwWm05eWJtbGhNUll3RkFZRApWUVFIRXcxVFlXNGdSbkpoYm1OcGMyTnZNUnd3R2dZRFZRUUtFeE52Y21jeExuTjFjR1Z5ZG1semIzSXVZMjl0Ck1SOHdIUVlEVlFRREV4WmpZUzV2Y21jeExuTjFjR1Z5ZG1semIzSXVZMjl0TUZrd0V3WUhLb1pJemowQ0FRWUkKS29aSXpqMERBUWNEUWdBRTdGdENrQ2pXWVNHU0hJQnZsUWdYaXB5UEdGNjhNaHFjY3NmMm56dEdqMEtseGxzUwpteTcvVXFIanhybzM4UlFQaWJMZTdxL1BxcTlBanVjTFBjSXZzNk50TUdzd0RnWURWUjBQQVFIL0JBUURBZ0dtCk1CMEdBMVVkSlFRV01CUUdDQ3NHQVFVRkJ3TUNCZ2dyQmdFRkJRY0RBVEFQQmdOVkhSTUJBZjhFQlRBREFRSC8KTUNrR0ExVWREZ1FpQkNBQzdNRFdhQWttZ3Z4OGRrcEJ2UkthS0FDY3pMYnVlWmFKNU9YVFRidU1RREFLQmdncQpoa2pPUFFRREFnTkhBREJFQWlCaVEzRDhOMklHdWxFVVRMS3FjNTgzU29EMWVkbVBMZTBVeDRBQ0dROXpSd0lnClhnVUczSzhCbklETk5Gcjd3enl1MlhXOW94Z3VvaHI4TmpzS3hnSGxqbmM9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K",
> 																				"organizational_unit_identifier": "orderer"
> 																			},
> 																			"peer_ou_identifier": {
> 																				"certificate": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNYRENDQWdPZ0F3SUJBZ0lRYitha2dLcUIyWURrRkRxaWVSUFhCVEFLQmdncWhrak9QUVFEQWpCNU1Rc3cKQ1FZRFZRUUdFd0pWVXpFVE1CRUdBMVVFQ0JNS1EyRnNhV1p2Y201cFlURVdNQlFHQTFVRUJ4TU5VMkZ1SUVaeQpZVzVqYVhOamJ6RWNNQm9HQTFVRUNoTVRiM0puTVM1emRYQmxjblpwYzI5eUxtTnZiVEVmTUIwR0ExVUVBeE1XClkyRXViM0puTVM1emRYQmxjblpwYzI5eUxtTnZiVEFlRncweU1qRXhNVGd4TlRBMU1EQmFGdzB6TWpFeE1UVXgKTlRBMU1EQmFNSGt4Q3pBSkJnTlZCQVlUQWxWVE1STXdFUVlEVlFRSUV3cERZV3hwWm05eWJtbGhNUll3RkFZRApWUVFIRXcxVFlXNGdSbkpoYm1OcGMyTnZNUnd3R2dZRFZRUUtFeE52Y21jeExuTjFjR1Z5ZG1semIzSXVZMjl0Ck1SOHdIUVlEVlFRREV4WmpZUzV2Y21jeExuTjFjR1Z5ZG1semIzSXVZMjl0TUZrd0V3WUhLb1pJemowQ0FRWUkKS29aSXpqMERBUWNEUWdBRTdGdENrQ2pXWVNHU0hJQnZsUWdYaXB5UEdGNjhNaHFjY3NmMm56dEdqMEtseGxzUwpteTcvVXFIanhybzM4UlFQaWJMZTdxL1BxcTlBanVjTFBjSXZzNk50TUdzd0RnWURWUjBQQVFIL0JBUURBZ0dtCk1CMEdBMVVkSlFRV01CUUdDQ3NHQVFVRkJ3TUNCZ2dyQmdFRkJRY0RBVEFQQmdOVkhSTUJBZjhFQlRBREFRSC8KTUNrR0ExVWREZ1FpQkNBQzdNRFdhQWttZ3Z4OGRrcEJ2UkthS0FDY3pMYnVlWmFKNU9YVFRidU1RREFLQmdncQpoa2pPUFFRREFnTkhBREJFQWlCaVEzRDhOMklHdWxFVVRMS3FjNTgzU29EMWVkbVBMZTBVeDRBQ0dROXpSd0lnClhnVUczSzhCbklETk5Gcjd3enl1MlhXOW94Z3VvaHI4TmpzS3hnSGxqbmM9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K",
> 																				"organizational_unit_identifier": "peer"
> 																			}
> 																		},
> 																		"intermediate_certs": [],
> 																		"name": "Org1MSP",
> 																		"organizational_unit_identifiers": [],
> 																		"revocation_list": [],
> 																		"root_certs": [
> 																			"LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNYRENDQWdPZ0F3SUJBZ0lRYitha2dLcUIyWURrRkRxaWVSUFhCVEFLQmdncWhrak9QUVFEQWpCNU1Rc3cKQ1FZRFZRUUdFd0pWVXpFVE1CRUdBMVVFQ0JNS1EyRnNhV1p2Y201cFlURVdNQlFHQTFVRUJ4TU5VMkZ1SUVaeQpZVzVqYVhOamJ6RWNNQm9HQTFVRUNoTVRiM0puTVM1emRYQmxjblpwYzI5eUxtTnZiVEVmTUIwR0ExVUVBeE1XClkyRXViM0puTVM1emRYQmxjblpwYzI5eUxtTnZiVEFlRncweU1qRXhNVGd4TlRBMU1EQmFGdzB6TWpFeE1UVXgKTlRBMU1EQmFNSGt4Q3pBSkJnTlZCQVlUQWxWVE1STXdFUVlEVlFRSUV3cERZV3hwWm05eWJtbGhNUll3RkFZRApWUVFIRXcxVFlXNGdSbkpoYm1OcGMyTnZNUnd3R2dZRFZRUUtFeE52Y21jeExuTjFjR1Z5ZG1semIzSXVZMjl0Ck1SOHdIUVlEVlFRREV4WmpZUzV2Y21jeExuTjFjR1Z5ZG1semIzSXVZMjl0TUZrd0V3WUhLb1pJemowQ0FRWUkKS29aSXpqMERBUWNEUWdBRTdGdENrQ2pXWVNHU0hJQnZsUWdYaXB5UEdGNjhNaHFjY3NmMm56dEdqMEtseGxzUwpteTcvVXFIanhybzM4UlFQaWJMZTdxL1BxcTlBanVjTFBjSXZzNk50TUdzd0RnWURWUjBQQVFIL0JBUURBZ0dtCk1CMEdBMVVkSlFRV01CUUdDQ3NHQVFVRkJ3TUNCZ2dyQmdFRkJRY0RBVEFQQmdOVkhSTUJBZjhFQlRBREFRSC8KTUNrR0ExVWREZ1FpQkNBQzdNRFdhQWttZ3Z4OGRrcEJ2UkthS0FDY3pMYnVlWmFKNU9YVFRidU1RREFLQmdncQpoa2pPUFFRREFnTkhBREJFQWlCaVEzRDhOMklHdWxFVVRMS3FjNTgzU29EMWVkbVBMZTBVeDRBQ0dROXpSd0lnClhnVUczSzhCbklETk5Gcjd3enl1MlhXOW94Z3VvaHI4TmpzS3hnSGxqbmM9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K"
> 																		],
> 																		"signing_identity": null,
> 																		"tls_intermediate_certs": [],
> 																		"tls_root_certs": [
> 																			"LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNZekNDQWdxZ0F3SUJBZ0lSQUxWTHhDcWE5aXVEWFJ3OHgrT0dkbW93Q2dZSUtvWkl6ajBFQXdJd2ZERUwKTUFrR0ExVUVCaE1DVlZNeEV6QVJCZ05WQkFnVENrTmhiR2xtYjNKdWFXRXhGakFVQmdOVkJBY1REVk5oYmlCRwpjbUZ1WTJselkyOHhIREFhQmdOVkJBb1RFMjl5WnpFdWMzVndaWEoyYVhOdmNpNWpiMjB4SWpBZ0JnTlZCQU1UCkdYUnNjMk5oTG05eVp6RXVjM1Z3WlhKMmFYTnZjaTVqYjIwd0hoY05Nakl4TVRFNE1UVXdOVEF3V2hjTk16SXgKTVRFMU1UVXdOVEF3V2pCOE1Rc3dDUVlEVlFRR0V3SlZVekVUTUJFR0ExVUVDQk1LUTJGc2FXWnZjbTVwWVRFVwpNQlFHQTFVRUJ4TU5VMkZ1SUVaeVlXNWphWE5qYnpFY01Cb0dBMVVFQ2hNVGIzSm5NUzV6ZFhCbGNuWnBjMjl5CkxtTnZiVEVpTUNBR0ExVUVBeE1aZEd4elkyRXViM0puTVM1emRYQmxjblpwYzI5eUxtTnZiVEJaTUJNR0J5cUcKU000OUFnRUdDQ3FHU000OUF3RUhBMElBQkhZUkpBRzlVNE9qbGlGUW9OWmJyNWh2TU1CY0dLYWVaT1NWNldzcQpBRWFqSjRmM3dZdWdzR1gwYkluczh5aDMrNThLWmJsZlZqN2JJZGtENGh5UUlUNmpiVEJyTUE0R0ExVWREd0VCCi93UUVBd0lCcGpBZEJnTlZIU1VFRmpBVUJnZ3JCZ0VGQlFjREFnWUlLd1lCQlFVSEF3RXdEd1lEVlIwVEFRSC8KQkFVd0F3RUIvekFwQmdOVkhRNEVJZ1FnMkRwcWZrOXBGZVZnZ0F6eWUzaU11QlpSV24vVzVpNmFTRWFpK04vKwpRVmd3Q2dZSUtvWkl6ajBFQXdJRFJ3QXdSQUlnVVNDUGM5ZnlUTFpHUWRuWStudUFFV1Rrak5uV1BkVXdHd1I0CnoxQjR6eVlDSUYvei9KNDhVd0pCL0htMWZGWnYyL1ZJZjYxWXNuQVR6ZjF6eFlVdUJwNlUKLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo="
> 																		]
> 																	},
> 																	"type": 0
> 																},
> 																"version": "0"
> 															}
> 														},
> 														"version": "0"
> 													},
> 													"Org2MSP": {
> 														"groups": {},
> 														"mod_policy": "Admins",
> 														"policies": {
> 															"Admins": {
> 																"mod_policy": "Admins",
> 																"policy": {
> 																	"type": 1,
> 																	"value": {
> 																		"identities": [
> 																			{
> 																				"principal": {
> 																					"msp_identifier": "Org2MSP",
> 																					"role": "ADMIN"
> 																				},
> 																				"principal_classification": "ROLE"
> 																			}
> 																		],
> 																		"rule": {
> 																			"n_out_of": {
> 																				"n": 1,
> 																				"rules": [
> 																					{
> 																						"signed_by": 0
> 																					}
> 																				]
> 																			}
> 																		},
> 																		"version": 0
> 																	}
> 																},
> 																"version": "0"
> 															},
> 															"Endorsement": {
> 																"mod_policy": "Admins",
> 																"policy": {
> 																	"type": 1,
> 																	"value": {
> 																		"identities": [
> 																			{
> 																				"principal": {
> 																					"msp_identifier": "Org2MSP",
> 																					"role": "PEER"
> 																				},
> 																				"principal_classification": "ROLE"
> 																			}
> 																		],
> 																		"rule": {
> 																			"n_out_of": {
> 																				"n": 1,
> 																				"rules": [
> 																					{
> 																						"signed_by": 0
> 																					}
> 																				]
> 																			}
> 																		},
> 																		"version": 0
> 																	}
> 																},
> 																"version": "0"
> 															},
> 															"Readers": {
> 																"mod_policy": "Admins",
> 																"policy": {
> 																	"type": 1,
> 																	"value": {
> 																		"identities": [
> 																			{
> 																				"principal": {
> 																					"msp_identifier": "Org2MSP",
> 																					"role": "ADMIN"
> 																				},
> 																				"principal_classification": "ROLE"
> 																			},
> 																			{
> 																				"principal": {
> 																					"msp_identifier": "Org2MSP",
> 																					"role": "PEER"
> 																				},
> 																				"principal_classification": "ROLE"
> 																			},
> 																			{
> 																				"principal": {
> 																					"msp_identifier": "Org2MSP",
> 																					"role": "CLIENT"
> 																				},
> 																				"principal_classification": "ROLE"
> 																			}
> 																		],
> 																		"rule": {
> 																			"n_out_of": {
> 																				"n": 1,
> 																				"rules": [
> 																					{
> 																						"signed_by": 0
> 																					},
> 																					{
> 																						"signed_by": 1
> 																					},
> 																					{
> 																						"signed_by": 2
> 																					}
> 																				]
> 																			}
> 																		},
> 																		"version": 0
> 																	}
> 																},
> 																"version": "0"
> 															},
> 															"Writers": {
> 																"mod_policy": "Admins",
> 																"policy": {
> 																	"type": 1,
> 																	"value": {
> 																		"identities": [
> 																			{
> 																				"principal": {
> 																					"msp_identifier": "Org2MSP",
> 																					"role": "ADMIN"
> 																				},
> 																				"principal_classification": "ROLE"
> 																			},
> 																			{
> 																				"principal": {
> 																					"msp_identifier": "Org2MSP",
> 																					"role": "CLIENT"
> 																				},
> 																				"principal_classification": "ROLE"
> 																			}
> 																		],
> 																		"rule": {
> 																			"n_out_of": {
> 																				"n": 1,
> 																				"rules": [
> 																					{
> 																						"signed_by": 0
> 																					},
> 																					{
> 																						"signed_by": 1
> 																					}
> 																				]
> 																			}
> 																		},
> 																		"version": 0
> 																	}
> 																},
> 																"version": "0"
> 															}
> 														},
> 														"values": {
> 															"MSP": {
> 																"mod_policy": "Admins",
> 																"value": {
> 																	"config": {
> 																		"admins": [],
> 																		"crypto_config": {
> 																			"identity_identifier_hash_function": "SHA256",
> 																			"signature_hash_family": "SHA2"
> 																		},
> 																		"fabric_node_ous": {
> 																			"admin_ou_identifier": {
> 																				"certificate": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNTakNDQWZDZ0F3SUJBZ0lSQU1NdjVxTmJMdFpVK2VxYVdxblhaOVl3Q2dZSUtvWkl6ajBFQXdJd2J6RUwKTUFrR0ExVUVCaE1DVlZNeEV6QVJCZ05WQkFnVENrTmhiR2xtYjNKdWFXRXhGakFVQmdOVkJBY1REVk5oYmlCRwpjbUZ1WTJselkyOHhGekFWQmdOVkJBb1REbTl5WnpJdVluVnBiR1F1WTI5dE1Sb3dHQVlEVlFRREV4RmpZUzV2CmNtY3lMbUoxYVd4a0xtTnZiVEFlRncweU1qRXhNVGd4TlRBMU1EQmFGdzB6TWpFeE1UVXhOVEExTURCYU1HOHgKQ3pBSkJnTlZCQVlUQWxWVE1STXdFUVlEVlFRSUV3cERZV3hwWm05eWJtbGhNUll3RkFZRFZRUUhFdzFUWVc0ZwpSbkpoYm1OcGMyTnZNUmN3RlFZRFZRUUtFdzV2Y21jeUxtSjFhV3hrTG1OdmJURWFNQmdHQTFVRUF4TVJZMkV1CmIzSm5NaTVpZFdsc1pDNWpiMjB3V1RBVEJnY3Foa2pPUFFJQkJnZ3Foa2pPUFFNQkJ3TkNBQVJJajg5V1BYWksKblh4MTNHWjRXcWxnbTgwbTlQcnJKK3VqTjhUMlRNaXlHVDczZ1RaMDJTbmI4THphZWVjYndocC90YU0xR3ZhNwpxYzRHZHhqQnVlMFdvMjB3YXpBT0JnTlZIUThCQWY4RUJBTUNBYVl3SFFZRFZSMGxCQll3RkFZSUt3WUJCUVVICkF3SUdDQ3NHQVFVRkJ3TUJNQThHQTFVZEV3RUIvd1FGTUFNQkFmOHdLUVlEVlIwT0JDSUVJTDMyYTFxZmVleCsKRmZNTUxXQmxTYy9Ob3BYVXdvUXVOVmN4TkRKaGxUQXhNQW9HQ0NxR1NNNDlCQU1DQTBnQU1FVUNJUUR2S0lMdgpSREJZaVpLbzFlQVFxbmg2WlgwTm1UY3NIbkoxd1p6ais5TWo5UUlnRkVvN2VkbVQxVElGZXpoRWt1SGZQU1o1CkpCZllwd1h6TnAzc3RGQjdobGM9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K",
> 																				"organizational_unit_identifier": "admin"
> 																			},
> 																			"client_ou_identifier": {
> 																				"certificate": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNTakNDQWZDZ0F3SUJBZ0lSQU1NdjVxTmJMdFpVK2VxYVdxblhaOVl3Q2dZSUtvWkl6ajBFQXdJd2J6RUwKTUFrR0ExVUVCaE1DVlZNeEV6QVJCZ05WQkFnVENrTmhiR2xtYjNKdWFXRXhGakFVQmdOVkJBY1REVk5oYmlCRwpjbUZ1WTJselkyOHhGekFWQmdOVkJBb1REbTl5WnpJdVluVnBiR1F1WTI5dE1Sb3dHQVlEVlFRREV4RmpZUzV2CmNtY3lMbUoxYVd4a0xtTnZiVEFlRncweU1qRXhNVGd4TlRBMU1EQmFGdzB6TWpFeE1UVXhOVEExTURCYU1HOHgKQ3pBSkJnTlZCQVlUQWxWVE1STXdFUVlEVlFRSUV3cERZV3hwWm05eWJtbGhNUll3RkFZRFZRUUhFdzFUWVc0ZwpSbkpoYm1OcGMyTnZNUmN3RlFZRFZRUUtFdzV2Y21jeUxtSjFhV3hrTG1OdmJURWFNQmdHQTFVRUF4TVJZMkV1CmIzSm5NaTVpZFdsc1pDNWpiMjB3V1RBVEJnY3Foa2pPUFFJQkJnZ3Foa2pPUFFNQkJ3TkNBQVJJajg5V1BYWksKblh4MTNHWjRXcWxnbTgwbTlQcnJKK3VqTjhUMlRNaXlHVDczZ1RaMDJTbmI4THphZWVjYndocC90YU0xR3ZhNwpxYzRHZHhqQnVlMFdvMjB3YXpBT0JnTlZIUThCQWY4RUJBTUNBYVl3SFFZRFZSMGxCQll3RkFZSUt3WUJCUVVICkF3SUdDQ3NHQVFVRkJ3TUJNQThHQTFVZEV3RUIvd1FGTUFNQkFmOHdLUVlEVlIwT0JDSUVJTDMyYTFxZmVleCsKRmZNTUxXQmxTYy9Ob3BYVXdvUXVOVmN4TkRKaGxUQXhNQW9HQ0NxR1NNNDlCQU1DQTBnQU1FVUNJUUR2S0lMdgpSREJZaVpLbzFlQVFxbmg2WlgwTm1UY3NIbkoxd1p6ais5TWo5UUlnRkVvN2VkbVQxVElGZXpoRWt1SGZQU1o1CkpCZllwd1h6TnAzc3RGQjdobGM9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K",
> 																				"organizational_unit_identifier": "client"
> 																			},
> 																			"enable": true,
> 																			"orderer_ou_identifier": {
> 																				"certificate": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNTakNDQWZDZ0F3SUJBZ0lSQU1NdjVxTmJMdFpVK2VxYVdxblhaOVl3Q2dZSUtvWkl6ajBFQXdJd2J6RUwKTUFrR0ExVUVCaE1DVlZNeEV6QVJCZ05WQkFnVENrTmhiR2xtYjNKdWFXRXhGakFVQmdOVkJBY1REVk5oYmlCRwpjbUZ1WTJselkyOHhGekFWQmdOVkJBb1REbTl5WnpJdVluVnBiR1F1WTI5dE1Sb3dHQVlEVlFRREV4RmpZUzV2CmNtY3lMbUoxYVd4a0xtTnZiVEFlRncweU1qRXhNVGd4TlRBMU1EQmFGdzB6TWpFeE1UVXhOVEExTURCYU1HOHgKQ3pBSkJnTlZCQVlUQWxWVE1STXdFUVlEVlFRSUV3cERZV3hwWm05eWJtbGhNUll3RkFZRFZRUUhFdzFUWVc0ZwpSbkpoYm1OcGMyTnZNUmN3RlFZRFZRUUtFdzV2Y21jeUxtSjFhV3hrTG1OdmJURWFNQmdHQTFVRUF4TVJZMkV1CmIzSm5NaTVpZFdsc1pDNWpiMjB3V1RBVEJnY3Foa2pPUFFJQkJnZ3Foa2pPUFFNQkJ3TkNBQVJJajg5V1BYWksKblh4MTNHWjRXcWxnbTgwbTlQcnJKK3VqTjhUMlRNaXlHVDczZ1RaMDJTbmI4THphZWVjYndocC90YU0xR3ZhNwpxYzRHZHhqQnVlMFdvMjB3YXpBT0JnTlZIUThCQWY4RUJBTUNBYVl3SFFZRFZSMGxCQll3RkFZSUt3WUJCUVVICkF3SUdDQ3NHQVFVRkJ3TUJNQThHQTFVZEV3RUIvd1FGTUFNQkFmOHdLUVlEVlIwT0JDSUVJTDMyYTFxZmVleCsKRmZNTUxXQmxTYy9Ob3BYVXdvUXVOVmN4TkRKaGxUQXhNQW9HQ0NxR1NNNDlCQU1DQTBnQU1FVUNJUUR2S0lMdgpSREJZaVpLbzFlQVFxbmg2WlgwTm1UY3NIbkoxd1p6ais5TWo5UUlnRkVvN2VkbVQxVElGZXpoRWt1SGZQU1o1CkpCZllwd1h6TnAzc3RGQjdobGM9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K",
> 																				"organizational_unit_identifier": "orderer"
> 																			},
> 																			"peer_ou_identifier": {
> 																				"certificate": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNTakNDQWZDZ0F3SUJBZ0lSQU1NdjVxTmJMdFpVK2VxYVdxblhaOVl3Q2dZSUtvWkl6ajBFQXdJd2J6RUwKTUFrR0ExVUVCaE1DVlZNeEV6QVJCZ05WQkFnVENrTmhiR2xtYjNKdWFXRXhGakFVQmdOVkJBY1REVk5oYmlCRwpjbUZ1WTJselkyOHhGekFWQmdOVkJBb1REbTl5WnpJdVluVnBiR1F1WTI5dE1Sb3dHQVlEVlFRREV4RmpZUzV2CmNtY3lMbUoxYVd4a0xtTnZiVEFlRncweU1qRXhNVGd4TlRBMU1EQmFGdzB6TWpFeE1UVXhOVEExTURCYU1HOHgKQ3pBSkJnTlZCQVlUQWxWVE1STXdFUVlEVlFRSUV3cERZV3hwWm05eWJtbGhNUll3RkFZRFZRUUhFdzFUWVc0ZwpSbkpoYm1OcGMyTnZNUmN3RlFZRFZRUUtFdzV2Y21jeUxtSjFhV3hrTG1OdmJURWFNQmdHQTFVRUF4TVJZMkV1CmIzSm5NaTVpZFdsc1pDNWpiMjB3V1RBVEJnY3Foa2pPUFFJQkJnZ3Foa2pPUFFNQkJ3TkNBQVJJajg5V1BYWksKblh4MTNHWjRXcWxnbTgwbTlQcnJKK3VqTjhUMlRNaXlHVDczZ1RaMDJTbmI4THphZWVjYndocC90YU0xR3ZhNwpxYzRHZHhqQnVlMFdvMjB3YXpBT0JnTlZIUThCQWY4RUJBTUNBYVl3SFFZRFZSMGxCQll3RkFZSUt3WUJCUVVICkF3SUdDQ3NHQVFVRkJ3TUJNQThHQTFVZEV3RUIvd1FGTUFNQkFmOHdLUVlEVlIwT0JDSUVJTDMyYTFxZmVleCsKRmZNTUxXQmxTYy9Ob3BYVXdvUXVOVmN4TkRKaGxUQXhNQW9HQ0NxR1NNNDlCQU1DQTBnQU1FVUNJUUR2S0lMdgpSREJZaVpLbzFlQVFxbmg2WlgwTm1UY3NIbkoxd1p6ais5TWo5UUlnRkVvN2VkbVQxVElGZXpoRWt1SGZQU1o1CkpCZllwd1h6TnAzc3RGQjdobGM9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K",
> 																				"organizational_unit_identifier": "peer"
> 																			}
> 																		},
> 																		"intermediate_certs": [],
> 																		"name": "Org2MSP",
> 																		"organizational_unit_identifiers": [],
> 																		"revocation_list": [],
> 																		"root_certs": [
> 																			"LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNTakNDQWZDZ0F3SUJBZ0lSQU1NdjVxTmJMdFpVK2VxYVdxblhaOVl3Q2dZSUtvWkl6ajBFQXdJd2J6RUwKTUFrR0ExVUVCaE1DVlZNeEV6QVJCZ05WQkFnVENrTmhiR2xtYjNKdWFXRXhGakFVQmdOVkJBY1REVk5oYmlCRwpjbUZ1WTJselkyOHhGekFWQmdOVkJBb1REbTl5WnpJdVluVnBiR1F1WTI5dE1Sb3dHQVlEVlFRREV4RmpZUzV2CmNtY3lMbUoxYVd4a0xtTnZiVEFlRncweU1qRXhNVGd4TlRBMU1EQmFGdzB6TWpFeE1UVXhOVEExTURCYU1HOHgKQ3pBSkJnTlZCQVlUQWxWVE1STXdFUVlEVlFRSUV3cERZV3hwWm05eWJtbGhNUll3RkFZRFZRUUhFdzFUWVc0ZwpSbkpoYm1OcGMyTnZNUmN3RlFZRFZRUUtFdzV2Y21jeUxtSjFhV3hrTG1OdmJURWFNQmdHQTFVRUF4TVJZMkV1CmIzSm5NaTVpZFdsc1pDNWpiMjB3V1RBVEJnY3Foa2pPUFFJQkJnZ3Foa2pPUFFNQkJ3TkNBQVJJajg5V1BYWksKblh4MTNHWjRXcWxnbTgwbTlQcnJKK3VqTjhUMlRNaXlHVDczZ1RaMDJTbmI4THphZWVjYndocC90YU0xR3ZhNwpxYzRHZHhqQnVlMFdvMjB3YXpBT0JnTlZIUThCQWY4RUJBTUNBYVl3SFFZRFZSMGxCQll3RkFZSUt3WUJCUVVICkF3SUdDQ3NHQVFVRkJ3TUJNQThHQTFVZEV3RUIvd1FGTUFNQkFmOHdLUVlEVlIwT0JDSUVJTDMyYTFxZmVleCsKRmZNTUxXQmxTYy9Ob3BYVXdvUXVOVmN4TkRKaGxUQXhNQW9HQ0NxR1NNNDlCQU1DQTBnQU1FVUNJUUR2S0lMdgpSREJZaVpLbzFlQVFxbmg2WlgwTm1UY3NIbkoxd1p6ais5TWo5UUlnRkVvN2VkbVQxVElGZXpoRWt1SGZQU1o1CkpCZllwd1h6TnAzc3RGQjdobGM9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K"
> 																		],
> 																		"signing_identity": null,
> 																		"tls_intermediate_certs": [],
> 																		"tls_root_certs": [
> 																			"LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNVVENDQWZhZ0F3SUJBZ0lSQVA0T0xwRFdYYWFqSkVlWkdCdDlXOGN3Q2dZSUtvWkl6ajBFQXdJd2NqRUwKTUFrR0ExVUVCaE1DVlZNeEV6QVJCZ05WQkFnVENrTmhiR2xtYjNKdWFXRXhGakFVQmdOVkJBY1REVk5oYmlCRwpjbUZ1WTJselkyOHhGekFWQmdOVkJBb1REbTl5WnpJdVluVnBiR1F1WTI5dE1SMHdHd1lEVlFRREV4UjBiSE5qCllTNXZjbWN5TG1KMWFXeGtMbU52YlRBZUZ3MHlNakV4TVRneE5UQTFNREJhRncwek1qRXhNVFV4TlRBMU1EQmEKTUhJeEN6QUpCZ05WQkFZVEFsVlRNUk13RVFZRFZRUUlFd3BEWVd4cFptOXlibWxoTVJZd0ZBWURWUVFIRXcxVApZVzRnUm5KaGJtTnBjMk52TVJjd0ZRWURWUVFLRXc1dmNtY3lMbUoxYVd4a0xtTnZiVEVkTUJzR0ExVUVBeE1VCmRHeHpZMkV1YjNKbk1pNWlkV2xzWkM1amIyMHdXVEFUQmdjcWhrak9QUUlCQmdncWhrak9QUU1CQndOQ0FBVEIKeGk1L3BSZC96dHEzbHlqNW1DWnpzTUM2MTdBb3RxU08zNGtsM0RDdW9rblczZHlEc3IyQ2R5eEtWajlsMy9uUApreGw4R01VMDVhbERnYURUNjczTm8yMHdhekFPQmdOVkhROEJBZjhFQkFNQ0FhWXdIUVlEVlIwbEJCWXdGQVlJCkt3WUJCUVVIQXdJR0NDc0dBUVVGQndNQk1BOEdBMVVkRXdFQi93UUZNQU1CQWY4d0tRWURWUjBPQkNJRUlIZ0EKa1ZFOEJLZDJreDZyck0yY0xDRUREZG85ZWczZjZhdHNTSnh2VU5BUE1Bb0dDQ3FHU000OUJBTUNBMGtBTUVZQwpJUUM5WVQ2dUQzN0tFQ2tsTWU5RVNYSzB4ZGNmbnlVdHE0TjlwN2luS013UWR3SWhBTExzK3ZVWUIybFh1S3VLCmd2c3I5cXFaM1hqZmZaay9HNkk4SDZ5SVFtL0kKLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo="
> 																		]
> 																	},
> 																	"type": 0
> 																},
> 																"version": "0"
> 															}
> 														},
> 														"version": "0"
> 													},
> 													"Org3MSP": {
> 														"groups": {},
> 														"mod_policy": "Admins",
> 														"policies": {
> 															"Admins": {
> 																"mod_policy": "Admins",
> 																"policy": {
> 																	"type": 1,
> 																	"value": {
> 																		"identities": [
> 																			{
> 																				"principal": {
> 																					"msp_identifier": "Org3MSP",
> 																					"role": "ADMIN"
> 																				},
> 																				"principal_classification": "ROLE"
> 																			}
> 																		],
> 																		"rule": {
> 																			"n_out_of": {
> 																				"n": 1,
> 																				"rules": [
> 																					{
> 																						"signed_by": 0
> 																					}
> 																				]
> 																			}
> 																		},
> 																		"version": 0
> 																	}
> 																},
> 																"version": "0"
> 															},
> 															"Endorsement": {
> 																"mod_policy": "Admins",
> 																"policy": {
> 																	"type": 1,
> 																	"value": {
> 																		"identities": [
> 																			{
> 																				"principal": {
> 																					"msp_identifier": "Org3MSP",
> 																					"role": "PEER"
> 																				},
> 																				"principal_classification": "ROLE"
> 																			}
> 																		],
> 																		"rule": {
> 																			"n_out_of": {
> 																				"n": 1,
> 																				"rules": [
> 																					{
> 																						"signed_by": 0
> 																					}
> 																				]
> 																			}
> 																		},
> 																		"version": 0
> 																	}
> 																},
> 																"version": "0"
> 															},
> 															"Readers": {
> 																"mod_policy": "Admins",
> 																"policy": {
> 																	"type": 1,
> 																	"value": {
> 																		"identities": [
> 																			{
> 																				"principal": {
> 																					"msp_identifier": "Org3MSP",
> 																					"role": "ADMIN"
> 																				},
> 																				"principal_classification": "ROLE"
> 																			},
> 																			{
> 																				"principal": {
> 																					"msp_identifier": "Org3MSP",
> 																					"role": "PEER"
> 																				},
> 																				"principal_classification": "ROLE"
> 																			},
> 																			{
> 																				"principal": {
> 																					"msp_identifier": "Org3MSP",
> 																					"role": "CLIENT"
> 																				},
> 																				"principal_classification": "ROLE"
> 																			}
> 																		],
> 																		"rule": {
> 																			"n_out_of": {
> 																				"n": 1,
> 																				"rules": [
> 																					{
> 																						"signed_by": 0
> 																					},
> 																					{
> 																						"signed_by": 1
> 																					},
> 																					{
> 																						"signed_by": 2
> 																					}
> 																				]
> 																			}
> 																		},
> 																		"version": 0
> 																	}
> 																},
> 																"version": "0"
> 															},
> 															"Writers": {
> 																"mod_policy": "Admins",
> 																"policy": {
> 																	"type": 1,
> 																	"value": {
> 																		"identities": [
> 																			{
> 																				"principal": {
> 																					"msp_identifier": "Org3MSP",
> 																					"role": "ADMIN"
> 																				},
> 																				"principal_classification": "ROLE"
> 																			},
> 																			{
> 																				"principal": {
> 																					"msp_identifier": "Org3MSP",
> 																					"role": "CLIENT"
> 																				},
> 																				"principal_classification": "ROLE"
> 																			}
> 																		],
> 																		"rule": {
> 																			"n_out_of": {
> 																				"n": 1,
> 																				"rules": [
> 																					{
> 																						"signed_by": 0
> 																					},
> 																					{
> 																						"signed_by": 1
> 																					}
> 																				]
> 																			}
> 																		},
> 																		"version": 0
> 																	}
> 																},
> 																"version": "0"
> 															}
> 														},
> 														"values": {
> 															"MSP": {
> 																"mod_policy": "Admins",
> 																"value": {
> 																	"config": {
> 																		"admins": [],
> 																		"crypto_config": {
> 																			"identity_identifier_hash_function": "SHA256",
> 																			"signature_hash_family": "SHA2"
> 																		},
> 																		"fabric_node_ous": {
> 																			"admin_ou_identifier": {
> 																				"certificate": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNWakNDQWZ5Z0F3SUJBZ0lSQU4vakxGWWc3QTBmRXdyZlJUbVVrdVV3Q2dZSUtvWkl6ajBFQXdJd2RURUwKTUFrR0ExVUVCaE1DVlZNeEV6QVJCZ05WQkFnVENrTmhiR2xtYjNKdWFXRXhGakFVQmdOVkJBY1REVk5oYmlCRwpjbUZ1WTJselkyOHhHakFZQmdOVkJBb1RFVzl5WnpNdWMzVndjR3hwWlhJdVkyOXRNUjB3R3dZRFZRUURFeFJqCllTNXZjbWN6TG5OMWNIQnNhV1Z5TG1OdmJUQWVGdzB5TWpFeE1UZ3hOVEExTURCYUZ3MHpNakV4TVRVeE5UQTEKTURCYU1IVXhDekFKQmdOVkJBWVRBbFZUTVJNd0VRWURWUVFJRXdwRFlXeHBabTl5Ym1saE1SWXdGQVlEVlFRSApFdzFUWVc0Z1JuSmhibU5wYzJOdk1Sb3dHQVlEVlFRS0V4RnZjbWN6TG5OMWNIQnNhV1Z5TG1OdmJURWRNQnNHCkExVUVBeE1VWTJFdWIzSm5NeTV6ZFhCd2JHbGxjaTVqYjIwd1dUQVRCZ2NxaGtqT1BRSUJCZ2dxaGtqT1BRTUIKQndOQ0FBUUxWVWNzVW5ZdG53eXQvSStKbzE5YUVpMWpmNytvV3hWMmg5cVZLSWE1czNQNUdBUHdlYTBBRHNPZQpzU2pPUjZDTHlPZ3h5TDlmNWFJUkJkcTE1Q3lCbzIwd2F6QU9CZ05WSFE4QkFmOEVCQU1DQWFZd0hRWURWUjBsCkJCWXdGQVlJS3dZQkJRVUhBd0lHQ0NzR0FRVUZCd01CTUE4R0ExVWRFd0VCL3dRRk1BTUJBZjh3S1FZRFZSME8KQkNJRUlFb01hdXczRmhPUDdZY0luNEMyTDhJSnU4UFBjalNRZFA4aStqR2J6R0VGTUFvR0NDcUdTTTQ5QkFNQwpBMGdBTUVVQ0lRRDkweC9BV0ZJbkxJeXp1c3ltd2NXRFVDL0dFbTM3bExZaUYvSWRDL3lGaXdJZ1VGb0hidWR6ClAwZEIrYkJsbzBMN0pCMW9RMG02b2pxcm56elJNNW1XdzE0PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==",
> 																				"organizational_unit_identifier": "admin"
> 																			},
> 																			"client_ou_identifier": {
> 																				"certificate": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNWakNDQWZ5Z0F3SUJBZ0lSQU4vakxGWWc3QTBmRXdyZlJUbVVrdVV3Q2dZSUtvWkl6ajBFQXdJd2RURUwKTUFrR0ExVUVCaE1DVlZNeEV6QVJCZ05WQkFnVENrTmhiR2xtYjNKdWFXRXhGakFVQmdOVkJBY1REVk5oYmlCRwpjbUZ1WTJselkyOHhHakFZQmdOVkJBb1RFVzl5WnpNdWMzVndjR3hwWlhJdVkyOXRNUjB3R3dZRFZRUURFeFJqCllTNXZjbWN6TG5OMWNIQnNhV1Z5TG1OdmJUQWVGdzB5TWpFeE1UZ3hOVEExTURCYUZ3MHpNakV4TVRVeE5UQTEKTURCYU1IVXhDekFKQmdOVkJBWVRBbFZUTVJNd0VRWURWUVFJRXdwRFlXeHBabTl5Ym1saE1SWXdGQVlEVlFRSApFdzFUWVc0Z1JuSmhibU5wYzJOdk1Sb3dHQVlEVlFRS0V4RnZjbWN6TG5OMWNIQnNhV1Z5TG1OdmJURWRNQnNHCkExVUVBeE1VWTJFdWIzSm5NeTV6ZFhCd2JHbGxjaTVqYjIwd1dUQVRCZ2NxaGtqT1BRSUJCZ2dxaGtqT1BRTUIKQndOQ0FBUUxWVWNzVW5ZdG53eXQvSStKbzE5YUVpMWpmNytvV3hWMmg5cVZLSWE1czNQNUdBUHdlYTBBRHNPZQpzU2pPUjZDTHlPZ3h5TDlmNWFJUkJkcTE1Q3lCbzIwd2F6QU9CZ05WSFE4QkFmOEVCQU1DQWFZd0hRWURWUjBsCkJCWXdGQVlJS3dZQkJRVUhBd0lHQ0NzR0FRVUZCd01CTUE4R0ExVWRFd0VCL3dRRk1BTUJBZjh3S1FZRFZSME8KQkNJRUlFb01hdXczRmhPUDdZY0luNEMyTDhJSnU4UFBjalNRZFA4aStqR2J6R0VGTUFvR0NDcUdTTTQ5QkFNQwpBMGdBTUVVQ0lRRDkweC9BV0ZJbkxJeXp1c3ltd2NXRFVDL0dFbTM3bExZaUYvSWRDL3lGaXdJZ1VGb0hidWR6ClAwZEIrYkJsbzBMN0pCMW9RMG02b2pxcm56elJNNW1XdzE0PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==",
> 																				"organizational_unit_identifier": "client"
> 																			},
> 																			"enable": true,
> 																			"orderer_ou_identifier": {
> 																				"certificate": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNWakNDQWZ5Z0F3SUJBZ0lSQU4vakxGWWc3QTBmRXdyZlJUbVVrdVV3Q2dZSUtvWkl6ajBFQXdJd2RURUwKTUFrR0ExVUVCaE1DVlZNeEV6QVJCZ05WQkFnVENrTmhiR2xtYjNKdWFXRXhGakFVQmdOVkJBY1REVk5oYmlCRwpjbUZ1WTJselkyOHhHakFZQmdOVkJBb1RFVzl5WnpNdWMzVndjR3hwWlhJdVkyOXRNUjB3R3dZRFZRUURFeFJqCllTNXZjbWN6TG5OMWNIQnNhV1Z5TG1OdmJUQWVGdzB5TWpFeE1UZ3hOVEExTURCYUZ3MHpNakV4TVRVeE5UQTEKTURCYU1IVXhDekFKQmdOVkJBWVRBbFZUTVJNd0VRWURWUVFJRXdwRFlXeHBabTl5Ym1saE1SWXdGQVlEVlFRSApFdzFUWVc0Z1JuSmhibU5wYzJOdk1Sb3dHQVlEVlFRS0V4RnZjbWN6TG5OMWNIQnNhV1Z5TG1OdmJURWRNQnNHCkExVUVBeE1VWTJFdWIzSm5NeTV6ZFhCd2JHbGxjaTVqYjIwd1dUQVRCZ2NxaGtqT1BRSUJCZ2dxaGtqT1BRTUIKQndOQ0FBUUxWVWNzVW5ZdG53eXQvSStKbzE5YUVpMWpmNytvV3hWMmg5cVZLSWE1czNQNUdBUHdlYTBBRHNPZQpzU2pPUjZDTHlPZ3h5TDlmNWFJUkJkcTE1Q3lCbzIwd2F6QU9CZ05WSFE4QkFmOEVCQU1DQWFZd0hRWURWUjBsCkJCWXdGQVlJS3dZQkJRVUhBd0lHQ0NzR0FRVUZCd01CTUE4R0ExVWRFd0VCL3dRRk1BTUJBZjh3S1FZRFZSME8KQkNJRUlFb01hdXczRmhPUDdZY0luNEMyTDhJSnU4UFBjalNRZFA4aStqR2J6R0VGTUFvR0NDcUdTTTQ5QkFNQwpBMGdBTUVVQ0lRRDkweC9BV0ZJbkxJeXp1c3ltd2NXRFVDL0dFbTM3bExZaUYvSWRDL3lGaXdJZ1VGb0hidWR6ClAwZEIrYkJsbzBMN0pCMW9RMG02b2pxcm56elJNNW1XdzE0PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==",
> 																				"organizational_unit_identifier": "orderer"
> 																			},
> 																			"peer_ou_identifier": {
> 																				"certificate": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNWakNDQWZ5Z0F3SUJBZ0lSQU4vakxGWWc3QTBmRXdyZlJUbVVrdVV3Q2dZSUtvWkl6ajBFQXdJd2RURUwKTUFrR0ExVUVCaE1DVlZNeEV6QVJCZ05WQkFnVENrTmhiR2xtYjNKdWFXRXhGakFVQmdOVkJBY1REVk5oYmlCRwpjbUZ1WTJselkyOHhHakFZQmdOVkJBb1RFVzl5WnpNdWMzVndjR3hwWlhJdVkyOXRNUjB3R3dZRFZRUURFeFJqCllTNXZjbWN6TG5OMWNIQnNhV1Z5TG1OdmJUQWVGdzB5TWpFeE1UZ3hOVEExTURCYUZ3MHpNakV4TVRVeE5UQTEKTURCYU1IVXhDekFKQmdOVkJBWVRBbFZUTVJNd0VRWURWUVFJRXdwRFlXeHBabTl5Ym1saE1SWXdGQVlEVlFRSApFdzFUWVc0Z1JuSmhibU5wYzJOdk1Sb3dHQVlEVlFRS0V4RnZjbWN6TG5OMWNIQnNhV1Z5TG1OdmJURWRNQnNHCkExVUVBeE1VWTJFdWIzSm5NeTV6ZFhCd2JHbGxjaTVqYjIwd1dUQVRCZ2NxaGtqT1BRSUJCZ2dxaGtqT1BRTUIKQndOQ0FBUUxWVWNzVW5ZdG53eXQvSStKbzE5YUVpMWpmNytvV3hWMmg5cVZLSWE1czNQNUdBUHdlYTBBRHNPZQpzU2pPUjZDTHlPZ3h5TDlmNWFJUkJkcTE1Q3lCbzIwd2F6QU9CZ05WSFE4QkFmOEVCQU1DQWFZd0hRWURWUjBsCkJCWXdGQVlJS3dZQkJRVUhBd0lHQ0NzR0FRVUZCd01CTUE4R0ExVWRFd0VCL3dRRk1BTUJBZjh3S1FZRFZSME8KQkNJRUlFb01hdXczRmhPUDdZY0luNEMyTDhJSnU4UFBjalNRZFA4aStqR2J6R0VGTUFvR0NDcUdTTTQ5QkFNQwpBMGdBTUVVQ0lRRDkweC9BV0ZJbkxJeXp1c3ltd2NXRFVDL0dFbTM3bExZaUYvSWRDL3lGaXdJZ1VGb0hidWR6ClAwZEIrYkJsbzBMN0pCMW9RMG02b2pxcm56elJNNW1XdzE0PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==",
> 																				"organizational_unit_identifier": "peer"
> 																			}
> 																		},
> 																		"intermediate_certs": [],
> 																		"name": "Org3MSP",
> 																		"organizational_unit_identifiers": [],
> 																		"revocation_list": [],
> 																		"root_certs": [
> 																			"LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNWakNDQWZ5Z0F3SUJBZ0lSQU4vakxGWWc3QTBmRXdyZlJUbVVrdVV3Q2dZSUtvWkl6ajBFQXdJd2RURUwKTUFrR0ExVUVCaE1DVlZNeEV6QVJCZ05WQkFnVENrTmhiR2xtYjNKdWFXRXhGakFVQmdOVkJBY1REVk5oYmlCRwpjbUZ1WTJselkyOHhHakFZQmdOVkJBb1RFVzl5WnpNdWMzVndjR3hwWlhJdVkyOXRNUjB3R3dZRFZRUURFeFJqCllTNXZjbWN6TG5OMWNIQnNhV1Z5TG1OdmJUQWVGdzB5TWpFeE1UZ3hOVEExTURCYUZ3MHpNakV4TVRVeE5UQTEKTURCYU1IVXhDekFKQmdOVkJBWVRBbFZUTVJNd0VRWURWUVFJRXdwRFlXeHBabTl5Ym1saE1SWXdGQVlEVlFRSApFdzFUWVc0Z1JuSmhibU5wYzJOdk1Sb3dHQVlEVlFRS0V4RnZjbWN6TG5OMWNIQnNhV1Z5TG1OdmJURWRNQnNHCkExVUVBeE1VWTJFdWIzSm5NeTV6ZFhCd2JHbGxjaTVqYjIwd1dUQVRCZ2NxaGtqT1BRSUJCZ2dxaGtqT1BRTUIKQndOQ0FBUUxWVWNzVW5ZdG53eXQvSStKbzE5YUVpMWpmNytvV3hWMmg5cVZLSWE1czNQNUdBUHdlYTBBRHNPZQpzU2pPUjZDTHlPZ3h5TDlmNWFJUkJkcTE1Q3lCbzIwd2F6QU9CZ05WSFE4QkFmOEVCQU1DQWFZd0hRWURWUjBsCkJCWXdGQVlJS3dZQkJRVUhBd0lHQ0NzR0FRVUZCd01CTUE4R0ExVWRFd0VCL3dRRk1BTUJBZjh3S1FZRFZSME8KQkNJRUlFb01hdXczRmhPUDdZY0luNEMyTDhJSnU4UFBjalNRZFA4aStqR2J6R0VGTUFvR0NDcUdTTTQ5QkFNQwpBMGdBTUVVQ0lRRDkweC9BV0ZJbkxJeXp1c3ltd2NXRFVDL0dFbTM3bExZaUYvSWRDL3lGaXdJZ1VGb0hidWR6ClAwZEIrYkJsbzBMN0pCMW9RMG02b2pxcm56elJNNW1XdzE0PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg=="
> 																		],
> 																		"signing_identity": null,
> 																		"tls_intermediate_certs": [],
> 																		"tls_root_certs": [
> 																			"LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNYVENDQWdLZ0F3SUJBZ0lSQUtZa2FqN2ZtbFhhUVJTK01SMC9vSEF3Q2dZSUtvWkl6ajBFQXdJd2VERUwKTUFrR0ExVUVCaE1DVlZNeEV6QVJCZ05WQkFnVENrTmhiR2xtYjNKdWFXRXhGakFVQmdOVkJBY1REVk5oYmlCRwpjbUZ1WTJselkyOHhHakFZQmdOVkJBb1RFVzl5WnpNdWMzVndjR3hwWlhJdVkyOXRNU0F3SGdZRFZRUURFeGQwCmJITmpZUzV2Y21jekxuTjFjSEJzYVdWeUxtTnZiVEFlRncweU1qRXhNVGd4TlRBMU1EQmFGdzB6TWpFeE1UVXgKTlRBMU1EQmFNSGd4Q3pBSkJnTlZCQVlUQWxWVE1STXdFUVlEVlFRSUV3cERZV3hwWm05eWJtbGhNUll3RkFZRApWUVFIRXcxVFlXNGdSbkpoYm1OcGMyTnZNUm93R0FZRFZRUUtFeEZ2Y21jekxuTjFjSEJzYVdWeUxtTnZiVEVnCk1CNEdBMVVFQXhNWGRHeHpZMkV1YjNKbk15NXpkWEJ3YkdsbGNpNWpiMjB3V1RBVEJnY3Foa2pPUFFJQkJnZ3EKaGtqT1BRTUJCd05DQUFSQ3BlUVdNRlNsVmtmeldoSy9nM0kyc3VtK1pGb1lZSkNaRlJSamFSZnc3SmNJZTdlLwoyTzR5Tmp5L0x3dTd2U1FPWlMzTHlkQThhaHdvajlpSGg0MlZvMjB3YXpBT0JnTlZIUThCQWY4RUJBTUNBYVl3CkhRWURWUjBsQkJZd0ZBWUlLd1lCQlFVSEF3SUdDQ3NHQVFVRkJ3TUJNQThHQTFVZEV3RUIvd1FGTUFNQkFmOHcKS1FZRFZSME9CQ0lFSUJ1clFldE9ZZ0JSdVpPL2pza2ZSREczeHNBa295cVVQOEJTaTZka3JNYkVNQW9HQ0NxRwpTTTQ5QkFNQ0Ewa0FNRVlDSVFEME1aMTFUVmdkS0N2RHkzaHQyQ1RnS2tqaUN6S2o5MEl0emtvU1dyTXJYd0loCkFQZUp5amdlTC9QSHVtQjEzVkFxbFQ4MU5yVFcrbXVuU1dBSFVjdUVaKzR5Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K"
> 																		]
> 																	},
> 																	"type": 0
> 																},
> 																"version": "0"
> 															}
> 														},
> 														"version": "0"
> 													},
> 													"Org4MSP": {
> 														"groups": {},
> 														"mod_policy": "Admins",
> 														"policies": {
> 															"Admins": {
> 																"mod_policy": "Admins",
> 																"policy": {
> 																	"type": 1,
> 																	"value": {
> 																		"identities": [
> 																			{
> 																				"principal": {
> 																					"msp_identifier": "Org4MSP",
> 																					"role": "ADMIN"
> 																				},
> 																				"principal_classification": "ROLE"
> 																			}
> 																		],
> 																		"rule": {
> 																			"n_out_of": {
> 																				"n": 1,
> 																				"rules": [
> 																					{
> 																						"signed_by": 0
> 																					}
> 																				]
> 																			}
> 																		},
> 																		"version": 0
> 																	}
> 																},
> 																"version": "0"
> 															},
> 															"Endorsement": {
> 																"mod_policy": "Admins",
> 																"policy": {
> 																	"type": 1,
> 																	"value": {
> 																		"identities": [
> 																			{
> 																				"principal": {
> 																					"msp_identifier": "Org4MSP",
> 																					"role": "PEER"
> 																				},
> 																				"principal_classification": "ROLE"
> 																			}
> 																		],
> 																		"rule": {
> 																			"n_out_of": {
> 																				"n": 1,
> 																				"rules": [
> 																					{
> 																						"signed_by": 0
> 																					}
> 																				]
> 																			}
> 																		},
> 																		"version": 0
> 																	}
> 																},
> 																"version": "0"
> 															},
> 															"Readers": {
> 																"mod_policy": "Admins",
> 																"policy": {
> 																	"type": 1,
> 																	"value": {
> 																		"identities": [
> 																			{
> 																				"principal": {
> 																					"msp_identifier": "Org4MSP",
> 																					"role": "ADMIN"
> 																				},
> 																				"principal_classification": "ROLE"
> 																			},
> 																			{
> 																				"principal": {
> 																					"msp_identifier": "Org4MSP",
> 																					"role": "PEER"
> 																				},
> 																				"principal_classification": "ROLE"
> 																			},
> 																			{
> 																				"principal": {
> 																					"msp_identifier": "Org4MSP",
> 																					"role": "CLIENT"
> 																				},
> 																				"principal_classification": "ROLE"
> 																			}
> 																		],
> 																		"rule": {
> 																			"n_out_of": {
> 																				"n": 1,
> 																				"rules": [
> 																					{
> 																						"signed_by": 0
> 																					},
> 																					{
> 																						"signed_by": 1
> 																					},
> 																					{
> 																						"signed_by": 2
> 																					}
> 																				]
> 																			}
> 																		},
> 																		"version": 0
> 																	}
> 																},
> 																"version": "0"
> 															},
> 															"Writers": {
> 																"mod_policy": "Admins",
> 																"policy": {
> 																	"type": 1,
> 																	"value": {
> 																		"identities": [
> 																			{
> 																				"principal": {
> 																					"msp_identifier": "Org4MSP",
> 																					"role": "ADMIN"
> 																				},
> 																				"principal_classification": "ROLE"
> 																			},
> 																			{
> 																				"principal": {
> 																					"msp_identifier": "Org4MSP",
> 																					"role": "CLIENT"
> 																				},
> 																				"principal_classification": "ROLE"
> 																			}
> 																		],
> 																		"rule": {
> 																			"n_out_of": {
> 																				"n": 1,
> 																				"rules": [
> 																					{
> 																						"signed_by": 0
> 																					},
> 																					{
> 																						"signed_by": 1
> 																					}
> 																				]
> 																			}
> 																		},
> 																		"version": 0
> 																	}
> 																},
> 																"version": "0"
> 															}
> 														},
> 														"values": {
> 															"MSP": {
> 																"mod_policy": "Admins",
> 																"value": {
> 																	"config": {
> 																		"admins": [],
> 																		"crypto_config": {
> 																			"identity_identifier_hash_function": "SHA256",
> 																			"signature_hash_family": "SHA2"
> 																		},
> 																		"fabric_node_ous": {
> 																			"admin_ou_identifier": {
> 																				"certificate": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNXakNDQWdDZ0F3SUJBZ0lSQU0rQmdINnpac2ZlWTlUZW9BcU8wZmd3Q2dZSUtvWkl6ajBFQXdJd2R6RUwKTUFrR0ExVUVCaE1DVlZNeEV6QVJCZ05WQkFnVENrTmhiR2xtYjNKdWFXRXhGakFVQmdOVkJBY1REVk5oYmlCRwpjbUZ1WTJselkyOHhHekFaQmdOVkJBb1RFbTl5WnpRdWJHOW5hWE4wYVdOekxtTnZiVEVlTUJ3R0ExVUVBeE1WClkyRXViM0puTkM1c2IyZHBjM1JwWTNNdVkyOXRNQjRYRFRJeU1URXhPREUxTURVd01Gb1hEVE15TVRFeE5URTEKTURVd01Gb3dkekVMTUFrR0ExVUVCaE1DVlZNeEV6QVJCZ05WQkFnVENrTmhiR2xtYjNKdWFXRXhGakFVQmdOVgpCQWNURFZOaGJpQkdjbUZ1WTJselkyOHhHekFaQmdOVkJBb1RFbTl5WnpRdWJHOW5hWE4wYVdOekxtTnZiVEVlCk1Cd0dBMVVFQXhNVlkyRXViM0puTkM1c2IyZHBjM1JwWTNNdVkyOXRNRmt3RXdZSEtvWkl6ajBDQVFZSUtvWkkKemowREFRY0RRZ0FFRUxZZ3hZSlJZZzBtZEgxUk9HZUo3N3F2NlV0S21WWlV1U0tWY0lFekZORFBjS09xcEk5aAp6NmhkeVluKzZXTSt2NElWVVZBSUhLOW9jZDVoVmU0NGtLTnRNR3N3RGdZRFZSMFBBUUgvQkFRREFnR21NQjBHCkExVWRKUVFXTUJRR0NDc0dBUVVGQndNQ0JnZ3JCZ0VGQlFjREFUQVBCZ05WSFJNQkFmOEVCVEFEQVFIL01Da0cKQTFVZERnUWlCQ0EzL0N2VVpVc1VGaUVIUGJVZjZxVW5ycTd6OGF6RmRDZTcyelFEQzhpempEQUtCZ2dxaGtqTwpQUVFEQWdOSUFEQkZBaUVBcTU0L0Y3UCtsbXdVZnJBZnJ2RWJYa0VXK2ZEbENNQ2pqYndWVlJ4S0ZqVUNJQXZHCndYVDdNWjNyVTBCcGdMUXVKVEhCcy9sK3JGVkZReW1LWHgyb21DMVEKLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=",
> 																				"organizational_unit_identifier": "admin"
> 																			},
> 																			"client_ou_identifier": {
> 																				"certificate": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNXakNDQWdDZ0F3SUJBZ0lSQU0rQmdINnpac2ZlWTlUZW9BcU8wZmd3Q2dZSUtvWkl6ajBFQXdJd2R6RUwKTUFrR0ExVUVCaE1DVlZNeEV6QVJCZ05WQkFnVENrTmhiR2xtYjNKdWFXRXhGakFVQmdOVkJBY1REVk5oYmlCRwpjbUZ1WTJselkyOHhHekFaQmdOVkJBb1RFbTl5WnpRdWJHOW5hWE4wYVdOekxtTnZiVEVlTUJ3R0ExVUVBeE1WClkyRXViM0puTkM1c2IyZHBjM1JwWTNNdVkyOXRNQjRYRFRJeU1URXhPREUxTURVd01Gb1hEVE15TVRFeE5URTEKTURVd01Gb3dkekVMTUFrR0ExVUVCaE1DVlZNeEV6QVJCZ05WQkFnVENrTmhiR2xtYjNKdWFXRXhGakFVQmdOVgpCQWNURFZOaGJpQkdjbUZ1WTJselkyOHhHekFaQmdOVkJBb1RFbTl5WnpRdWJHOW5hWE4wYVdOekxtTnZiVEVlCk1Cd0dBMVVFQXhNVlkyRXViM0puTkM1c2IyZHBjM1JwWTNNdVkyOXRNRmt3RXdZSEtvWkl6ajBDQVFZSUtvWkkKemowREFRY0RRZ0FFRUxZZ3hZSlJZZzBtZEgxUk9HZUo3N3F2NlV0S21WWlV1U0tWY0lFekZORFBjS09xcEk5aAp6NmhkeVluKzZXTSt2NElWVVZBSUhLOW9jZDVoVmU0NGtLTnRNR3N3RGdZRFZSMFBBUUgvQkFRREFnR21NQjBHCkExVWRKUVFXTUJRR0NDc0dBUVVGQndNQ0JnZ3JCZ0VGQlFjREFUQVBCZ05WSFJNQkFmOEVCVEFEQVFIL01Da0cKQTFVZERnUWlCQ0EzL0N2VVpVc1VGaUVIUGJVZjZxVW5ycTd6OGF6RmRDZTcyelFEQzhpempEQUtCZ2dxaGtqTwpQUVFEQWdOSUFEQkZBaUVBcTU0L0Y3UCtsbXdVZnJBZnJ2RWJYa0VXK2ZEbENNQ2pqYndWVlJ4S0ZqVUNJQXZHCndYVDdNWjNyVTBCcGdMUXVKVEhCcy9sK3JGVkZReW1LWHgyb21DMVEKLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=",
> 																				"organizational_unit_identifier": "client"
> 																			},
> 																			"enable": true,
> 																			"orderer_ou_identifier": {
> 																				"certificate": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNXakNDQWdDZ0F3SUJBZ0lSQU0rQmdINnpac2ZlWTlUZW9BcU8wZmd3Q2dZSUtvWkl6ajBFQXdJd2R6RUwKTUFrR0ExVUVCaE1DVlZNeEV6QVJCZ05WQkFnVENrTmhiR2xtYjNKdWFXRXhGakFVQmdOVkJBY1REVk5oYmlCRwpjbUZ1WTJselkyOHhHekFaQmdOVkJBb1RFbTl5WnpRdWJHOW5hWE4wYVdOekxtTnZiVEVlTUJ3R0ExVUVBeE1WClkyRXViM0puTkM1c2IyZHBjM1JwWTNNdVkyOXRNQjRYRFRJeU1URXhPREUxTURVd01Gb1hEVE15TVRFeE5URTEKTURVd01Gb3dkekVMTUFrR0ExVUVCaE1DVlZNeEV6QVJCZ05WQkFnVENrTmhiR2xtYjNKdWFXRXhGakFVQmdOVgpCQWNURFZOaGJpQkdjbUZ1WTJselkyOHhHekFaQmdOVkJBb1RFbTl5WnpRdWJHOW5hWE4wYVdOekxtTnZiVEVlCk1Cd0dBMVVFQXhNVlkyRXViM0puTkM1c2IyZHBjM1JwWTNNdVkyOXRNRmt3RXdZSEtvWkl6ajBDQVFZSUtvWkkKemowREFRY0RRZ0FFRUxZZ3hZSlJZZzBtZEgxUk9HZUo3N3F2NlV0S21WWlV1U0tWY0lFekZORFBjS09xcEk5aAp6NmhkeVluKzZXTSt2NElWVVZBSUhLOW9jZDVoVmU0NGtLTnRNR3N3RGdZRFZSMFBBUUgvQkFRREFnR21NQjBHCkExVWRKUVFXTUJRR0NDc0dBUVVGQndNQ0JnZ3JCZ0VGQlFjREFUQVBCZ05WSFJNQkFmOEVCVEFEQVFIL01Da0cKQTFVZERnUWlCQ0EzL0N2VVpVc1VGaUVIUGJVZjZxVW5ycTd6OGF6RmRDZTcyelFEQzhpempEQUtCZ2dxaGtqTwpQUVFEQWdOSUFEQkZBaUVBcTU0L0Y3UCtsbXdVZnJBZnJ2RWJYa0VXK2ZEbENNQ2pqYndWVlJ4S0ZqVUNJQXZHCndYVDdNWjNyVTBCcGdMUXVKVEhCcy9sK3JGVkZReW1LWHgyb21DMVEKLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=",
> 																				"organizational_unit_identifier": "orderer"
> 																			},
> 																			"peer_ou_identifier": {
> 																				"certificate": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNXakNDQWdDZ0F3SUJBZ0lSQU0rQmdINnpac2ZlWTlUZW9BcU8wZmd3Q2dZSUtvWkl6ajBFQXdJd2R6RUwKTUFrR0ExVUVCaE1DVlZNeEV6QVJCZ05WQkFnVENrTmhiR2xtYjNKdWFXRXhGakFVQmdOVkJBY1REVk5oYmlCRwpjbUZ1WTJselkyOHhHekFaQmdOVkJBb1RFbTl5WnpRdWJHOW5hWE4wYVdOekxtTnZiVEVlTUJ3R0ExVUVBeE1WClkyRXViM0puTkM1c2IyZHBjM1JwWTNNdVkyOXRNQjRYRFRJeU1URXhPREUxTURVd01Gb1hEVE15TVRFeE5URTEKTURVd01Gb3dkekVMTUFrR0ExVUVCaE1DVlZNeEV6QVJCZ05WQkFnVENrTmhiR2xtYjNKdWFXRXhGakFVQmdOVgpCQWNURFZOaGJpQkdjbUZ1WTJselkyOHhHekFaQmdOVkJBb1RFbTl5WnpRdWJHOW5hWE4wYVdOekxtTnZiVEVlCk1Cd0dBMVVFQXhNVlkyRXViM0puTkM1c2IyZHBjM1JwWTNNdVkyOXRNRmt3RXdZSEtvWkl6ajBDQVFZSUtvWkkKemowREFRY0RRZ0FFRUxZZ3hZSlJZZzBtZEgxUk9HZUo3N3F2NlV0S21WWlV1U0tWY0lFekZORFBjS09xcEk5aAp6NmhkeVluKzZXTSt2NElWVVZBSUhLOW9jZDVoVmU0NGtLTnRNR3N3RGdZRFZSMFBBUUgvQkFRREFnR21NQjBHCkExVWRKUVFXTUJRR0NDc0dBUVVGQndNQ0JnZ3JCZ0VGQlFjREFUQVBCZ05WSFJNQkFmOEVCVEFEQVFIL01Da0cKQTFVZERnUWlCQ0EzL0N2VVpVc1VGaUVIUGJVZjZxVW5ycTd6OGF6RmRDZTcyelFEQzhpempEQUtCZ2dxaGtqTwpQUVFEQWdOSUFEQkZBaUVBcTU0L0Y3UCtsbXdVZnJBZnJ2RWJYa0VXK2ZEbENNQ2pqYndWVlJ4S0ZqVUNJQXZHCndYVDdNWjNyVTBCcGdMUXVKVEhCcy9sK3JGVkZReW1LWHgyb21DMVEKLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=",
> 																				"organizational_unit_identifier": "peer"
> 																			}
> 																		},
> 																		"intermediate_certs": [],
> 																		"name": "Org4MSP",
> 																		"organizational_unit_identifiers": [],
> 																		"revocation_list": [],
> 																		"root_certs": [
> 																			"LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNXakNDQWdDZ0F3SUJBZ0lSQU0rQmdINnpac2ZlWTlUZW9BcU8wZmd3Q2dZSUtvWkl6ajBFQXdJd2R6RUwKTUFrR0ExVUVCaE1DVlZNeEV6QVJCZ05WQkFnVENrTmhiR2xtYjNKdWFXRXhGakFVQmdOVkJBY1REVk5oYmlCRwpjbUZ1WTJselkyOHhHekFaQmdOVkJBb1RFbTl5WnpRdWJHOW5hWE4wYVdOekxtTnZiVEVlTUJ3R0ExVUVBeE1WClkyRXViM0puTkM1c2IyZHBjM1JwWTNNdVkyOXRNQjRYRFRJeU1URXhPREUxTURVd01Gb1hEVE15TVRFeE5URTEKTURVd01Gb3dkekVMTUFrR0ExVUVCaE1DVlZNeEV6QVJCZ05WQkFnVENrTmhiR2xtYjNKdWFXRXhGakFVQmdOVgpCQWNURFZOaGJpQkdjbUZ1WTJselkyOHhHekFaQmdOVkJBb1RFbTl5WnpRdWJHOW5hWE4wYVdOekxtTnZiVEVlCk1Cd0dBMVVFQXhNVlkyRXViM0puTkM1c2IyZHBjM1JwWTNNdVkyOXRNRmt3RXdZSEtvWkl6ajBDQVFZSUtvWkkKemowREFRY0RRZ0FFRUxZZ3hZSlJZZzBtZEgxUk9HZUo3N3F2NlV0S21WWlV1U0tWY0lFekZORFBjS09xcEk5aAp6NmhkeVluKzZXTSt2NElWVVZBSUhLOW9jZDVoVmU0NGtLTnRNR3N3RGdZRFZSMFBBUUgvQkFRREFnR21NQjBHCkExVWRKUVFXTUJRR0NDc0dBUVVGQndNQ0JnZ3JCZ0VGQlFjREFUQVBCZ05WSFJNQkFmOEVCVEFEQVFIL01Da0cKQTFVZERnUWlCQ0EzL0N2VVpVc1VGaUVIUGJVZjZxVW5ycTd6OGF6RmRDZTcyelFEQzhpempEQUtCZ2dxaGtqTwpQUVFEQWdOSUFEQkZBaUVBcTU0L0Y3UCtsbXdVZnJBZnJ2RWJYa0VXK2ZEbENNQ2pqYndWVlJ4S0ZqVUNJQXZHCndYVDdNWjNyVTBCcGdMUXVKVEhCcy9sK3JGVkZReW1LWHgyb21DMVEKLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo="
> 																		],
> 																		"signing_identity": null,
> 																		"tls_intermediate_certs": [],
> 																		"tls_root_certs": [
> 																			"LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNYekNDQWdXZ0F3SUJBZ0lRT3BncWhTM3pibWJNU1FOWWVKNnFHekFLQmdncWhrak9QUVFEQWpCNk1Rc3cKQ1FZRFZRUUdFd0pWVXpFVE1CRUdBMVVFQ0JNS1EyRnNhV1p2Y201cFlURVdNQlFHQTFVRUJ4TU5VMkZ1SUVaeQpZVzVqYVhOamJ6RWJNQmtHQTFVRUNoTVNiM0puTkM1c2IyZHBjM1JwWTNNdVkyOXRNU0V3SHdZRFZRUURFeGgwCmJITmpZUzV2Y21jMExteHZaMmx6ZEdsamN5NWpiMjB3SGhjTk1qSXhNVEU0TVRVd05UQXdXaGNOTXpJeE1URTEKTVRVd05UQXdXakI2TVFzd0NRWURWUVFHRXdKVlV6RVRNQkVHQTFVRUNCTUtRMkZzYVdadmNtNXBZVEVXTUJRRwpBMVVFQnhNTlUyRnVJRVp5WVc1amFYTmpiekViTUJrR0ExVUVDaE1TYjNKbk5DNXNiMmRwYzNScFkzTXVZMjl0Ck1TRXdId1lEVlFRREV4aDBiSE5qWVM1dmNtYzBMbXh2WjJsemRHbGpjeTVqYjIwd1dUQVRCZ2NxaGtqT1BRSUIKQmdncWhrak9QUU1CQndOQ0FBVFNocTNVeEhkWlkyeCtSQU9LV0VObDNVVE9La2ZRL0pSeENwQzJ6SCtDZnVINgpqMkpjbVRUbGZEbi93SktHelRKbnQwWjZIYi9rWmdVVFRHbklqZENrbzIwd2F6QU9CZ05WSFE4QkFmOEVCQU1DCkFhWXdIUVlEVlIwbEJCWXdGQVlJS3dZQkJRVUhBd0lHQ0NzR0FRVUZCd01CTUE4R0ExVWRFd0VCL3dRRk1BTUIKQWY4d0tRWURWUjBPQkNJRUlPVnM1c21veTNCRHVKUnlrYWs0WHpyRW9ZUkNYd0Y0NnQ2VW15aFVwQ29TTUFvRwpDQ3FHU000OUJBTUNBMGdBTUVVQ0lRQzhEWDMvNmNrRGdYUm12Y2t3aDQ5UUtHMnpBL2pKQkt5TGhsbDZlbnFkCnhRSWdIR05oY2NHUG5qTDlJeFFsYS9pbnZ5VUVPcTQ3dDYwRmd4NlQvOXBwaWpjPQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg=="
> 																		]
> 																	},
> 																	"type": 0
> 																},
> 																"version": "0"
> 															}
> 														},
> 														"version": "0"
> 													}
> 												},
> 												"mod_policy": "/Channel/Orderer/Admins",
> 												"policies": {},
> 												"values": {
> 													"ChannelCreationPolicy": {
> 														"mod_policy": "/Channel/Orderer/Admins",
> 														"value": {
> 															"type": 3,
> 															"value": {
> 																"rule": "ANY",
> 																"sub_policy": "Admins"
> 															}
> 														},
> 														"version": "0"
> 													}
> 												},
> 												"version": "0"
> 											}
> 										},
> 										"mod_policy": "/Channel/Orderer/Admins",
> 										"policies": {
> 											"Admins": {
> 												"mod_policy": "/Channel/Orderer/Admins",
> 												"policy": {
> 													"type": 1,
> 													"value": {
> 														"identities": [],
> 														"rule": {
> 															"n_out_of": {
> 																"n": 0,
> 																"rules": []
> 															}
> 														},
> 														"version": 0
> 													}
> 												},
> 												"version": "0"
> 											}
> 										},
> 										"values": {},
> 										"version": "0"
> 									},
> 									"Orderer": {
> 										"groups": {
> 											"OrdererOrg": {
> 												"groups": {},
> 												"mod_policy": "Admins",
> 												"policies": {
> 													"Admins": {
> 														"mod_policy": "Admins",
> 														"policy": {
> 															"type": 1,
> 															"value": {
> 																"identities": [
> 																	{
> 																		"principal": {
> 																			"msp_identifier": "OrdererMSP",
> 																			"role": "ADMIN"
> 																		},
> 																		"principal_classification": "ROLE"
> 																	}
> 																],
> 																"rule": {
> 																	"n_out_of": {
> 																		"n": 1,
> 																		"rules": [
> 																			{
> 																				"signed_by": 0
> 																			}
> 																		]
> 																	}
> 																},
> 																"version": 0
> 															}
> 														},
> 														"version": "0"
> 													},
> 													"Readers": {
> 														"mod_policy": "Admins",
> 														"policy": {
> 															"type": 1,
> 															"value": {
> 																"identities": [
> 																	{
> 																		"principal": {
> 																			"msp_identifier": "OrdererMSP",
> 																			"role": "MEMBER"
> 																		},
> 																		"principal_classification": "ROLE"
> 																	}
> 																],
> 																"rule": {
> 																	"n_out_of": {
> 																		"n": 1,
> 																		"rules": [
> 																			{
> 																				"signed_by": 0
> 																			}
> 																		]
> 																	}
> 																},
> 																"version": 0
> 															}
> 														},
> 														"version": "0"
> 													},
> 													"Writers": {
> 														"mod_policy": "Admins",
> 														"policy": {
> 															"type": 1,
> 															"value": {
> 																"identities": [
> 																	{
> 																		"principal": {
> 																			"msp_identifier": "OrdererMSP",
> 																			"role": "MEMBER"
> 																		},
> 																		"principal_classification": "ROLE"
> 																	}
> 																],
> 																"rule": {
> 																	"n_out_of": {
> 																		"n": 1,
> 																		"rules": [
> 																			{
> 																				"signed_by": 0
> 																			}
> 																		]
> 																	}
> 																},
> 																"version": 0
> 															}
> 														},
> 														"version": "0"
> 													}
> 												},
> 												"values": {
> 													"MSP": {
> 														"mod_policy": "Admins",
> 														"value": {
> 															"config": {
> 																"admins": [],
> 																"crypto_config": {
> 																	"identity_identifier_hash_function": "SHA256",
> 																	"signature_hash_family": "SHA2"
> 																},
> 																"fabric_node_ous": {
> 																	"admin_ou_identifier": {
> 																		"certificate": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNQVENDQWVTZ0F3SUJBZ0lSQUtaSktteVpwVUpzcFFuMWtZQVNTVDh3Q2dZSUtvWkl6ajBFQXdJd2FURUwKTUFrR0ExVUVCaE1DVlZNeEV6QVJCZ05WQkFnVENrTmhiR2xtYjNKdWFXRXhGakFVQmdOVkJBY1REVk5oYmlCRwpjbUZ1WTJselkyOHhGREFTQmdOVkJBb1RDMlY0WVcxd2JHVXVZMjl0TVJjd0ZRWURWUVFERXc1allTNWxlR0Z0CmNHeGxMbU52YlRBZUZ3MHlNakV4TVRneE5UQTFNREJhRncwek1qRXhNVFV4TlRBMU1EQmFNR2t4Q3pBSkJnTlYKQkFZVEFsVlRNUk13RVFZRFZRUUlFd3BEWVd4cFptOXlibWxoTVJZd0ZBWURWUVFIRXcxVFlXNGdSbkpoYm1OcApjMk52TVJRd0VnWURWUVFLRXd0bGVHRnRjR3hsTG1OdmJURVhNQlVHQTFVRUF4TU9ZMkV1WlhoaGJYQnNaUzVqCmIyMHdXVEFUQmdjcWhrak9QUUlCQmdncWhrak9QUU1CQndOQ0FBVHdvOGRHN01QaENoUXlwamltZGxGa1VpTEQKRUR3OHpiL2ZXNCsyUjBvMjZKc3k3TkkwazIraGtBeUlPaUsyZXErdWJpWU9CNlpXckVzek5yMFhKaWlGbzIwdwphekFPQmdOVkhROEJBZjhFQkFNQ0FhWXdIUVlEVlIwbEJCWXdGQVlJS3dZQkJRVUhBd0lHQ0NzR0FRVUZCd01CCk1BOEdBMVVkRXdFQi93UUZNQU1CQWY4d0tRWURWUjBPQkNJRUlOTmZNS1FCYUZYVUxyNjhvazdKd1NDMy9WZEkKcU01cFhzSUQ1dk5idFFWSk1Bb0dDQ3FHU000OUJBTUNBMGNBTUVRQ0lHRlcveGF5bm5LUnYvV1BBNTlZUmVXcQppb3c2eWpXenNsSGlvcmpnckw1VUFpQjFRaWNGeTV5WG44RktvTXhScVF1RmZnVW9iU3FxM1M5U3U2Y0xBMnNUCklnPT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=",
> 																		"organizational_unit_identifier": "admin"
> 																	},
> 																	"client_ou_identifier": {
> 																		"certificate": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNQVENDQWVTZ0F3SUJBZ0lSQUtaSktteVpwVUpzcFFuMWtZQVNTVDh3Q2dZSUtvWkl6ajBFQXdJd2FURUwKTUFrR0ExVUVCaE1DVlZNeEV6QVJCZ05WQkFnVENrTmhiR2xtYjNKdWFXRXhGakFVQmdOVkJBY1REVk5oYmlCRwpjbUZ1WTJselkyOHhGREFTQmdOVkJBb1RDMlY0WVcxd2JHVXVZMjl0TVJjd0ZRWURWUVFERXc1allTNWxlR0Z0CmNHeGxMbU52YlRBZUZ3MHlNakV4TVRneE5UQTFNREJhRncwek1qRXhNVFV4TlRBMU1EQmFNR2t4Q3pBSkJnTlYKQkFZVEFsVlRNUk13RVFZRFZRUUlFd3BEWVd4cFptOXlibWxoTVJZd0ZBWURWUVFIRXcxVFlXNGdSbkpoYm1OcApjMk52TVJRd0VnWURWUVFLRXd0bGVHRnRjR3hsTG1OdmJURVhNQlVHQTFVRUF4TU9ZMkV1WlhoaGJYQnNaUzVqCmIyMHdXVEFUQmdjcWhrak9QUUlCQmdncWhrak9QUU1CQndOQ0FBVHdvOGRHN01QaENoUXlwamltZGxGa1VpTEQKRUR3OHpiL2ZXNCsyUjBvMjZKc3k3TkkwazIraGtBeUlPaUsyZXErdWJpWU9CNlpXckVzek5yMFhKaWlGbzIwdwphekFPQmdOVkhROEJBZjhFQkFNQ0FhWXdIUVlEVlIwbEJCWXdGQVlJS3dZQkJRVUhBd0lHQ0NzR0FRVUZCd01CCk1BOEdBMVVkRXdFQi93UUZNQU1CQWY4d0tRWURWUjBPQkNJRUlOTmZNS1FCYUZYVUxyNjhvazdKd1NDMy9WZEkKcU01cFhzSUQ1dk5idFFWSk1Bb0dDQ3FHU000OUJBTUNBMGNBTUVRQ0lHRlcveGF5bm5LUnYvV1BBNTlZUmVXcQppb3c2eWpXenNsSGlvcmpnckw1VUFpQjFRaWNGeTV5WG44RktvTXhScVF1RmZnVW9iU3FxM1M5U3U2Y0xBMnNUCklnPT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=",
> 																		"organizational_unit_identifier": "client"
> 																	},
> 																	"enable": true,
> 																	"orderer_ou_identifier": {
> 																		"certificate": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNQVENDQWVTZ0F3SUJBZ0lSQUtaSktteVpwVUpzcFFuMWtZQVNTVDh3Q2dZSUtvWkl6ajBFQXdJd2FURUwKTUFrR0ExVUVCaE1DVlZNeEV6QVJCZ05WQkFnVENrTmhiR2xtYjNKdWFXRXhGakFVQmdOVkJBY1REVk5oYmlCRwpjbUZ1WTJselkyOHhGREFTQmdOVkJBb1RDMlY0WVcxd2JHVXVZMjl0TVJjd0ZRWURWUVFERXc1allTNWxlR0Z0CmNHeGxMbU52YlRBZUZ3MHlNakV4TVRneE5UQTFNREJhRncwek1qRXhNVFV4TlRBMU1EQmFNR2t4Q3pBSkJnTlYKQkFZVEFsVlRNUk13RVFZRFZRUUlFd3BEWVd4cFptOXlibWxoTVJZd0ZBWURWUVFIRXcxVFlXNGdSbkpoYm1OcApjMk52TVJRd0VnWURWUVFLRXd0bGVHRnRjR3hsTG1OdmJURVhNQlVHQTFVRUF4TU9ZMkV1WlhoaGJYQnNaUzVqCmIyMHdXVEFUQmdjcWhrak9QUUlCQmdncWhrak9QUU1CQndOQ0FBVHdvOGRHN01QaENoUXlwamltZGxGa1VpTEQKRUR3OHpiL2ZXNCsyUjBvMjZKc3k3TkkwazIraGtBeUlPaUsyZXErdWJpWU9CNlpXckVzek5yMFhKaWlGbzIwdwphekFPQmdOVkhROEJBZjhFQkFNQ0FhWXdIUVlEVlIwbEJCWXdGQVlJS3dZQkJRVUhBd0lHQ0NzR0FRVUZCd01CCk1BOEdBMVVkRXdFQi93UUZNQU1CQWY4d0tRWURWUjBPQkNJRUlOTmZNS1FCYUZYVUxyNjhvazdKd1NDMy9WZEkKcU01cFhzSUQ1dk5idFFWSk1Bb0dDQ3FHU000OUJBTUNBMGNBTUVRQ0lHRlcveGF5bm5LUnYvV1BBNTlZUmVXcQppb3c2eWpXenNsSGlvcmpnckw1VUFpQjFRaWNGeTV5WG44RktvTXhScVF1RmZnVW9iU3FxM1M5U3U2Y0xBMnNUCklnPT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=",
> 																		"organizational_unit_identifier": "orderer"
> 																	},
> 																	"peer_ou_identifier": {
> 																		"certificate": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNQVENDQWVTZ0F3SUJBZ0lSQUtaSktteVpwVUpzcFFuMWtZQVNTVDh3Q2dZSUtvWkl6ajBFQXdJd2FURUwKTUFrR0ExVUVCaE1DVlZNeEV6QVJCZ05WQkFnVENrTmhiR2xtYjNKdWFXRXhGakFVQmdOVkJBY1REVk5oYmlCRwpjbUZ1WTJselkyOHhGREFTQmdOVkJBb1RDMlY0WVcxd2JHVXVZMjl0TVJjd0ZRWURWUVFERXc1allTNWxlR0Z0CmNHeGxMbU52YlRBZUZ3MHlNakV4TVRneE5UQTFNREJhRncwek1qRXhNVFV4TlRBMU1EQmFNR2t4Q3pBSkJnTlYKQkFZVEFsVlRNUk13RVFZRFZRUUlFd3BEWVd4cFptOXlibWxoTVJZd0ZBWURWUVFIRXcxVFlXNGdSbkpoYm1OcApjMk52TVJRd0VnWURWUVFLRXd0bGVHRnRjR3hsTG1OdmJURVhNQlVHQTFVRUF4TU9ZMkV1WlhoaGJYQnNaUzVqCmIyMHdXVEFUQmdjcWhrak9QUUlCQmdncWhrak9QUU1CQndOQ0FBVHdvOGRHN01QaENoUXlwamltZGxGa1VpTEQKRUR3OHpiL2ZXNCsyUjBvMjZKc3k3TkkwazIraGtBeUlPaUsyZXErdWJpWU9CNlpXckVzek5yMFhKaWlGbzIwdwphekFPQmdOVkhROEJBZjhFQkFNQ0FhWXdIUVlEVlIwbEJCWXdGQVlJS3dZQkJRVUhBd0lHQ0NzR0FRVUZCd01CCk1BOEdBMVVkRXdFQi93UUZNQU1CQWY4d0tRWURWUjBPQkNJRUlOTmZNS1FCYUZYVUxyNjhvazdKd1NDMy9WZEkKcU01cFhzSUQ1dk5idFFWSk1Bb0dDQ3FHU000OUJBTUNBMGNBTUVRQ0lHRlcveGF5bm5LUnYvV1BBNTlZUmVXcQppb3c2eWpXenNsSGlvcmpnckw1VUFpQjFRaWNGeTV5WG44RktvTXhScVF1RmZnVW9iU3FxM1M5U3U2Y0xBMnNUCklnPT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=",
> 																		"organizational_unit_identifier": "peer"
> 																	}
> 																},
> 																"intermediate_certs": [],
> 																"name": "OrdererMSP",
> 																"organizational_unit_identifiers": [],
> 																"revocation_list": [],
> 																"root_certs": [
> 																	"LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNQVENDQWVTZ0F3SUJBZ0lSQUtaSktteVpwVUpzcFFuMWtZQVNTVDh3Q2dZSUtvWkl6ajBFQXdJd2FURUwKTUFrR0ExVUVCaE1DVlZNeEV6QVJCZ05WQkFnVENrTmhiR2xtYjNKdWFXRXhGakFVQmdOVkJBY1REVk5oYmlCRwpjbUZ1WTJselkyOHhGREFTQmdOVkJBb1RDMlY0WVcxd2JHVXVZMjl0TVJjd0ZRWURWUVFERXc1allTNWxlR0Z0CmNHeGxMbU52YlRBZUZ3MHlNakV4TVRneE5UQTFNREJhRncwek1qRXhNVFV4TlRBMU1EQmFNR2t4Q3pBSkJnTlYKQkFZVEFsVlRNUk13RVFZRFZRUUlFd3BEWVd4cFptOXlibWxoTVJZd0ZBWURWUVFIRXcxVFlXNGdSbkpoYm1OcApjMk52TVJRd0VnWURWUVFLRXd0bGVHRnRjR3hsTG1OdmJURVhNQlVHQTFVRUF4TU9ZMkV1WlhoaGJYQnNaUzVqCmIyMHdXVEFUQmdjcWhrak9QUUlCQmdncWhrak9QUU1CQndOQ0FBVHdvOGRHN01QaENoUXlwamltZGxGa1VpTEQKRUR3OHpiL2ZXNCsyUjBvMjZKc3k3TkkwazIraGtBeUlPaUsyZXErdWJpWU9CNlpXckVzek5yMFhKaWlGbzIwdwphekFPQmdOVkhROEJBZjhFQkFNQ0FhWXdIUVlEVlIwbEJCWXdGQVlJS3dZQkJRVUhBd0lHQ0NzR0FRVUZCd01CCk1BOEdBMVVkRXdFQi93UUZNQU1CQWY4d0tRWURWUjBPQkNJRUlOTmZNS1FCYUZYVUxyNjhvazdKd1NDMy9WZEkKcU01cFhzSUQ1dk5idFFWSk1Bb0dDQ3FHU000OUJBTUNBMGNBTUVRQ0lHRlcveGF5bm5LUnYvV1BBNTlZUmVXcQppb3c2eWpXenNsSGlvcmpnckw1VUFpQjFRaWNGeTV5WG44RktvTXhScVF1RmZnVW9iU3FxM1M5U3U2Y0xBMnNUCklnPT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo="
> 																],
> 																"signing_identity": null,
> 																"tls_intermediate_certs": [],
> 																"tls_root_certs": [
> 																	"LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNRekNDQWVtZ0F3SUJBZ0lRRnpNTVcwb200R3VnaVNnOVdVYW9VekFLQmdncWhrak9QUVFEQWpCc01Rc3cKQ1FZRFZRUUdFd0pWVXpFVE1CRUdBMVVFQ0JNS1EyRnNhV1p2Y201cFlURVdNQlFHQTFVRUJ4TU5VMkZ1SUVaeQpZVzVqYVhOamJ6RVVNQklHQTFVRUNoTUxaWGhoYlhCc1pTNWpiMjB4R2pBWUJnTlZCQU1URVhSc2MyTmhMbVY0CllXMXdiR1V1WTI5dE1CNFhEVEl5TVRFeE9ERTFNRFV3TUZvWERUTXlNVEV4TlRFMU1EVXdNRm93YkRFTE1Ba0cKQTFVRUJoTUNWVk14RXpBUkJnTlZCQWdUQ2tOaGJHbG1iM0p1YVdFeEZqQVVCZ05WQkFjVERWTmhiaUJHY21GdQpZMmx6WTI4eEZEQVNCZ05WQkFvVEMyVjRZVzF3YkdVdVkyOXRNUm93R0FZRFZRUURFeEYwYkhOallTNWxlR0Z0CmNHeGxMbU52YlRCWk1CTUdCeXFHU000OUFnRUdDQ3FHU000OUF3RUhBMElBQkloSTltYUw1MTNFUmNGV0sya20KaFU5ZE1kT2FZL0RqaHdjRjc0VGo5Q0ZQUy9pRnlmeW1uaWZiSUkrQzV4eFlrODBFWmk3M3ljWFUzbndpWXV5UApCU2FqYlRCck1BNEdBMVVkRHdFQi93UUVBd0lCcGpBZEJnTlZIU1VFRmpBVUJnZ3JCZ0VGQlFjREFnWUlLd1lCCkJRVUhBd0V3RHdZRFZSMFRBUUgvQkFVd0F3RUIvekFwQmdOVkhRNEVJZ1Fnd09ORWt2Z2JWang2RE5LNEMzbk8KVTliV2s0SHNYcGhNWWVhdnkrYTc3ckl3Q2dZSUtvWkl6ajBFQXdJRFNBQXdSUUloQU5RRGFuQ1gwY1VpQ0cwbQpOSUFiMjBzM3Nhc3VSNURNRVN6Ti90bHRScDhjQWlBeVlNNUZlSlhacHhNa0xPaFdtcG10aGZqd3Q3ZWxnSjI1CktMeVJSc3hrNXc9PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg=="
> 																]
> 															},
> 															"type": 0
> 														},
> 														"version": "0"
> 													}
> 												},
> 												"version": "0"
> 											}
> 										},
> 										"mod_policy": "Admins",
> 										"policies": {
> 											"Admins": {
> 												"mod_policy": "Admins",
> 												"policy": {
> 													"type": 3,
> 													"value": {
> 														"rule": "MAJORITY",
> 														"sub_policy": "Admins"
> 													}
> 												},
> 												"version": "0"
> 											},
> 											"BlockValidation": {
> 												"mod_policy": "Admins",
> 												"policy": {
> 													"type": 3,
> 													"value": {
> 														"rule": "ANY",
> 														"sub_policy": "Writers"
> 													}
> 												},
> 												"version": "0"
> 											},
> 											"Readers": {
> 												"mod_policy": "Admins",
> 												"policy": {
> 													"type": 3,
> 													"value": {
> 														"rule": "ANY",
> 														"sub_policy": "Readers"
> 													}
> 												},
> 												"version": "0"
> 											},
> 											"Writers": {
> 												"mod_policy": "Admins",
> 												"policy": {
> 													"type": 3,
> 													"value": {
> 														"rule": "ANY",
> 														"sub_policy": "Writers"
> 													}
> 												},
> 												"version": "0"
> 											}
> 										},
> 										"values": {
> 											"BatchSize": {
> 												"mod_policy": "Admins",
> 												"value": {
> 													"absolute_max_bytes": 103809024,
> 													"max_message_count": 10,
> 													"preferred_max_bytes": 524288
> 												},
> 												"version": "0"
> 											},
> 											"BatchTimeout": {
> 												"mod_policy": "Admins",
> 												"value": {
> 													"timeout": "2s"
> 												},
> 												"version": "0"
> 											},
> 											"Capabilities": {
> 												"mod_policy": "Admins",
> 												"value": {
> 													"capabilities": {
> 														"V2_0": {}
> 													}
> 												},
> 												"version": "0"
> 											},
> 											"ChannelRestrictions": {
> 												"mod_policy": "Admins",
> 												"value": null,
> 												"version": "0"
> 											},
> 											"ConsensusType": {
> 												"mod_policy": "Admins",
> 												"value": {
> 													"metadata": {
> 														"consenters": [
> 															{
> 																"client_tls_cert": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNXekNDQWdLZ0F3SUJBZ0lRWjdpZm93M0twREVzZXVvWmJ5SVZwVEFLQmdncWhrak9QUVFEQWpCc01Rc3cKQ1FZRFZRUUdFd0pWVXpFVE1CRUdBMVVFQ0JNS1EyRnNhV1p2Y201cFlURVdNQlFHQTFVRUJ4TU5VMkZ1SUVaeQpZVzVqYVhOamJ6RVVNQklHQTFVRUNoTUxaWGhoYlhCc1pTNWpiMjB4R2pBWUJnTlZCQU1URVhSc2MyTmhMbVY0CllXMXdiR1V1WTI5dE1CNFhEVEl5TVRFeE9ERTFNRFV3TUZvWERUTXlNVEV4TlRFMU1EVXdNRm93V1RFTE1Ba0cKQTFVRUJoTUNWVk14RXpBUkJnTlZCQWdUQ2tOaGJHbG1iM0p1YVdFeEZqQVVCZ05WQkFjVERWTmhiaUJHY21GdQpZMmx6WTI4eEhUQWJCZ05WQkFNVEZHOXlaR1Z5WlhJd0xtVjRZVzF3YkdVdVkyOXRNRmt3RXdZSEtvWkl6ajBDCkFRWUlLb1pJemowREFRY0RRZ0FFdTNMZjBIWmMwa1NnWU5wNklYUkZUZDQxL0k3T3JCaUEwRjlVTW8vQlBTVXYKYlR1Tjl2a0tnQ1VMbFhneXNNZkR2MWphYjlodXM1ck5GTCs2bGxhSGpLT0JtRENCbFRBT0JnTlZIUThCQWY4RQpCQU1DQmFBd0hRWURWUjBsQkJZd0ZBWUlLd1lCQlFVSEF3RUdDQ3NHQVFVRkJ3TUNNQXdHQTFVZEV3RUIvd1FDCk1BQXdLd1lEVlIwakJDUXdJb0Fnd09ORWt2Z2JWang2RE5LNEMzbk9VOWJXazRIc1hwaE1ZZWF2eSthNzdySXcKS1FZRFZSMFJCQ0l3SUlJVWIzSmtaWEpsY2pBdVpYaGhiWEJzWlM1amIyMkNDRzl5WkdWeVpYSXdNQW9HQ0NxRwpTTTQ5QkFNQ0EwY0FNRVFDSUU3Sy90V3d1enl4K1NuTGZqbmZTZzhDU3lOUnh3SndOcXBlYzVCUXBzeWZBaUFmClNrRkNyWUxsLzJqSURuSGN2c2grbzJtK3JIaVhRUVR3c0c1L0JlTy9Ndz09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K",
> 																"host": "orderer0.example.com",
> 																"port": 7050,
> 																"server_tls_cert": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNXekNDQWdLZ0F3SUJBZ0lRWjdpZm93M0twREVzZXVvWmJ5SVZwVEFLQmdncWhrak9QUVFEQWpCc01Rc3cKQ1FZRFZRUUdFd0pWVXpFVE1CRUdBMVVFQ0JNS1EyRnNhV1p2Y201cFlURVdNQlFHQTFVRUJ4TU5VMkZ1SUVaeQpZVzVqYVhOamJ6RVVNQklHQTFVRUNoTUxaWGhoYlhCc1pTNWpiMjB4R2pBWUJnTlZCQU1URVhSc2MyTmhMbVY0CllXMXdiR1V1WTI5dE1CNFhEVEl5TVRFeE9ERTFNRFV3TUZvWERUTXlNVEV4TlRFMU1EVXdNRm93V1RFTE1Ba0cKQTFVRUJoTUNWVk14RXpBUkJnTlZCQWdUQ2tOaGJHbG1iM0p1YVdFeEZqQVVCZ05WQkFjVERWTmhiaUJHY21GdQpZMmx6WTI4eEhUQWJCZ05WQkFNVEZHOXlaR1Z5WlhJd0xtVjRZVzF3YkdVdVkyOXRNRmt3RXdZSEtvWkl6ajBDCkFRWUlLb1pJemowREFRY0RRZ0FFdTNMZjBIWmMwa1NnWU5wNklYUkZUZDQxL0k3T3JCaUEwRjlVTW8vQlBTVXYKYlR1Tjl2a0tnQ1VMbFhneXNNZkR2MWphYjlodXM1ck5GTCs2bGxhSGpLT0JtRENCbFRBT0JnTlZIUThCQWY4RQpCQU1DQmFBd0hRWURWUjBsQkJZd0ZBWUlLd1lCQlFVSEF3RUdDQ3NHQVFVRkJ3TUNNQXdHQTFVZEV3RUIvd1FDCk1BQXdLd1lEVlIwakJDUXdJb0Fnd09ORWt2Z2JWang2RE5LNEMzbk9VOWJXazRIc1hwaE1ZZWF2eSthNzdySXcKS1FZRFZSMFJCQ0l3SUlJVWIzSmtaWEpsY2pBdVpYaGhiWEJzWlM1amIyMkNDRzl5WkdWeVpYSXdNQW9HQ0NxRwpTTTQ5QkFNQ0EwY0FNRVFDSUU3Sy90V3d1enl4K1NuTGZqbmZTZzhDU3lOUnh3SndOcXBlYzVCUXBzeWZBaUFmClNrRkNyWUxsLzJqSURuSGN2c2grbzJtK3JIaVhRUVR3c0c1L0JlTy9Ndz09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K"
> 															},
> 															{
> 																"client_tls_cert": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNYRENDQWdLZ0F3SUJBZ0lRUWJGNzV2TGRRTkhzUTY2WmdnL2N0akFLQmdncWhrak9QUVFEQWpCc01Rc3cKQ1FZRFZRUUdFd0pWVXpFVE1CRUdBMVVFQ0JNS1EyRnNhV1p2Y201cFlURVdNQlFHQTFVRUJ4TU5VMkZ1SUVaeQpZVzVqYVhOamJ6RVVNQklHQTFVRUNoTUxaWGhoYlhCc1pTNWpiMjB4R2pBWUJnTlZCQU1URVhSc2MyTmhMbVY0CllXMXdiR1V1WTI5dE1CNFhEVEl5TVRFeE9ERTFNRFV3TUZvWERUTXlNVEV4TlRFMU1EVXdNRm93V1RFTE1Ba0cKQTFVRUJoTUNWVk14RXpBUkJnTlZCQWdUQ2tOaGJHbG1iM0p1YVdFeEZqQVVCZ05WQkFjVERWTmhiaUJHY21GdQpZMmx6WTI4eEhUQWJCZ05WQkFNVEZHOXlaR1Z5WlhJeExtVjRZVzF3YkdVdVkyOXRNRmt3RXdZSEtvWkl6ajBDCkFRWUlLb1pJemowREFRY0RRZ0FFZCtmenpybEwyMmpBVElnWXREUXlKV1JvRTRMeTdWbk56TXJLbFZkSjRSNTQKZEVTcVdwL0lpM1ZqNEhBZEhybWlVcHBub3JtZ3RXNStYNTIxeXJHRmlxT0JtRENCbFRBT0JnTlZIUThCQWY4RQpCQU1DQmFBd0hRWURWUjBsQkJZd0ZBWUlLd1lCQlFVSEF3RUdDQ3NHQVFVRkJ3TUNNQXdHQTFVZEV3RUIvd1FDCk1BQXdLd1lEVlIwakJDUXdJb0Fnd09ORWt2Z2JWang2RE5LNEMzbk9VOWJXazRIc1hwaE1ZZWF2eSthNzdySXcKS1FZRFZSMFJCQ0l3SUlJVWIzSmtaWEpsY2pFdVpYaGhiWEJzWlM1amIyMkNDRzl5WkdWeVpYSXhNQW9HQ0NxRwpTTTQ5QkFNQ0EwZ0FNRVVDSVFDN1MwVGhRQ2UvVE5BTlB6VmtMcWFac2RyMlUwZ3FzR21JWWdRVE5zdGxMUUlnCll3NVZ2cFczUDVweFFPRFZyVXNLbXFVQUhrT0txeWgwOVRLUEhTVVNtTjA9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K",
> 																"host": "orderer1.example.com",
> 																"port": 8050,
> 																"server_tls_cert": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNYRENDQWdLZ0F3SUJBZ0lRUWJGNzV2TGRRTkhzUTY2WmdnL2N0akFLQmdncWhrak9QUVFEQWpCc01Rc3cKQ1FZRFZRUUdFd0pWVXpFVE1CRUdBMVVFQ0JNS1EyRnNhV1p2Y201cFlURVdNQlFHQTFVRUJ4TU5VMkZ1SUVaeQpZVzVqYVhOamJ6RVVNQklHQTFVRUNoTUxaWGhoYlhCc1pTNWpiMjB4R2pBWUJnTlZCQU1URVhSc2MyTmhMbVY0CllXMXdiR1V1WTI5dE1CNFhEVEl5TVRFeE9ERTFNRFV3TUZvWERUTXlNVEV4TlRFMU1EVXdNRm93V1RFTE1Ba0cKQTFVRUJoTUNWVk14RXpBUkJnTlZCQWdUQ2tOaGJHbG1iM0p1YVdFeEZqQVVCZ05WQkFjVERWTmhiaUJHY21GdQpZMmx6WTI4eEhUQWJCZ05WQkFNVEZHOXlaR1Z5WlhJeExtVjRZVzF3YkdVdVkyOXRNRmt3RXdZSEtvWkl6ajBDCkFRWUlLb1pJemowREFRY0RRZ0FFZCtmenpybEwyMmpBVElnWXREUXlKV1JvRTRMeTdWbk56TXJLbFZkSjRSNTQKZEVTcVdwL0lpM1ZqNEhBZEhybWlVcHBub3JtZ3RXNStYNTIxeXJHRmlxT0JtRENCbFRBT0JnTlZIUThCQWY4RQpCQU1DQmFBd0hRWURWUjBsQkJZd0ZBWUlLd1lCQlFVSEF3RUdDQ3NHQVFVRkJ3TUNNQXdHQTFVZEV3RUIvd1FDCk1BQXdLd1lEVlIwakJDUXdJb0Fnd09ORWt2Z2JWang2RE5LNEMzbk9VOWJXazRIc1hwaE1ZZWF2eSthNzdySXcKS1FZRFZSMFJCQ0l3SUlJVWIzSmtaWEpsY2pFdVpYaGhiWEJzWlM1amIyMkNDRzl5WkdWeVpYSXhNQW9HQ0NxRwpTTTQ5QkFNQ0EwZ0FNRVVDSVFDN1MwVGhRQ2UvVE5BTlB6VmtMcWFac2RyMlUwZ3FzR21JWWdRVE5zdGxMUUlnCll3NVZ2cFczUDVweFFPRFZyVXNLbXFVQUhrT0txeWgwOVRLUEhTVVNtTjA9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K"
> 															},
> 															{
> 																"client_tls_cert": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNXekNDQWdLZ0F3SUJBZ0lRR0xSTzY0QlBXNmk5MmExRVBvSzdyakFLQmdncWhrak9QUVFEQWpCc01Rc3cKQ1FZRFZRUUdFd0pWVXpFVE1CRUdBMVVFQ0JNS1EyRnNhV1p2Y201cFlURVdNQlFHQTFVRUJ4TU5VMkZ1SUVaeQpZVzVqYVhOamJ6RVVNQklHQTFVRUNoTUxaWGhoYlhCc1pTNWpiMjB4R2pBWUJnTlZCQU1URVhSc2MyTmhMbVY0CllXMXdiR1V1WTI5dE1CNFhEVEl5TVRFeE9ERTFNRFV3TUZvWERUTXlNVEV4TlRFMU1EVXdNRm93V1RFTE1Ba0cKQTFVRUJoTUNWVk14RXpBUkJnTlZCQWdUQ2tOaGJHbG1iM0p1YVdFeEZqQVVCZ05WQkFjVERWTmhiaUJHY21GdQpZMmx6WTI4eEhUQWJCZ05WQkFNVEZHOXlaR1Z5WlhJeUxtVjRZVzF3YkdVdVkyOXRNRmt3RXdZSEtvWkl6ajBDCkFRWUlLb1pJemowREFRY0RRZ0FFMjRCOE9FTmpzbVhxdXBUODdCRU1oVVdmbWFDdC90YzBSK3hSMkNjdDFwS2kKZ09EdDV6UW85ZlpBTnN1Yi9ZT1V3T0pRSkdGa2FPN041Wmk1Z2RSTUVLT0JtRENCbFRBT0JnTlZIUThCQWY4RQpCQU1DQmFBd0hRWURWUjBsQkJZd0ZBWUlLd1lCQlFVSEF3RUdDQ3NHQVFVRkJ3TUNNQXdHQTFVZEV3RUIvd1FDCk1BQXdLd1lEVlIwakJDUXdJb0Fnd09ORWt2Z2JWang2RE5LNEMzbk9VOWJXazRIc1hwaE1ZZWF2eSthNzdySXcKS1FZRFZSMFJCQ0l3SUlJVWIzSmtaWEpsY2pJdVpYaGhiWEJzWlM1amIyMkNDRzl5WkdWeVpYSXlNQW9HQ0NxRwpTTTQ5QkFNQ0EwY0FNRVFDSUF3NjhsQmpjY25hTDZkVUFQcWErd0kvQjlnZHRZYjdtSFhOamlwWno0UlNBaUJMCmNIbmMvZmVOdi9udGhVOFUzcFZtL0lCTkhmV0NnVG9mc3N2eVpHeHVQdz09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K",
> 																"host": "orderer2.example.com",
> 																"port": 9050,
> 																"server_tls_cert": "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNXekNDQWdLZ0F3SUJBZ0lRR0xSTzY0QlBXNmk5MmExRVBvSzdyakFLQmdncWhrak9QUVFEQWpCc01Rc3cKQ1FZRFZRUUdFd0pWVXpFVE1CRUdBMVVFQ0JNS1EyRnNhV1p2Y201cFlURVdNQlFHQTFVRUJ4TU5VMkZ1SUVaeQpZVzVqYVhOamJ6RVVNQklHQTFVRUNoTUxaWGhoYlhCc1pTNWpiMjB4R2pBWUJnTlZCQU1URVhSc2MyTmhMbVY0CllXMXdiR1V1WTI5dE1CNFhEVEl5TVRFeE9ERTFNRFV3TUZvWERUTXlNVEV4TlRFMU1EVXdNRm93V1RFTE1Ba0cKQTFVRUJoTUNWVk14RXpBUkJnTlZCQWdUQ2tOaGJHbG1iM0p1YVdFeEZqQVVCZ05WQkFjVERWTmhiaUJHY21GdQpZMmx6WTI4eEhUQWJCZ05WQkFNVEZHOXlaR1Z5WlhJeUxtVjRZVzF3YkdVdVkyOXRNRmt3RXdZSEtvWkl6ajBDCkFRWUlLb1pJemowREFRY0RRZ0FFMjRCOE9FTmpzbVhxdXBUODdCRU1oVVdmbWFDdC90YzBSK3hSMkNjdDFwS2kKZ09EdDV6UW85ZlpBTnN1Yi9ZT1V3T0pRSkdGa2FPN041Wmk1Z2RSTUVLT0JtRENCbFRBT0JnTlZIUThCQWY4RQpCQU1DQmFBd0hRWURWUjBsQkJZd0ZBWUlLd1lCQlFVSEF3RUdDQ3NHQVFVRkJ3TUNNQXdHQTFVZEV3RUIvd1FDCk1BQXdLd1lEVlIwakJDUXdJb0Fnd09ORWt2Z2JWang2RE5LNEMzbk9VOWJXazRIc1hwaE1ZZWF2eSthNzdySXcKS1FZRFZSMFJCQ0l3SUlJVWIzSmtaWEpsY2pJdVpYaGhiWEJzWlM1amIyMkNDRzl5WkdWeVpYSXlNQW9HQ0NxRwpTTTQ5QkFNQ0EwY0FNRVFDSUF3NjhsQmpjY25hTDZkVUFQcWErd0kvQjlnZHRZYjdtSFhOamlwWno0UlNBaUJMCmNIbmMvZmVOdi9udGhVOFUzcFZtL0lCTkhmV0NnVG9mc3N2eVpHeHVQdz09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K"
> 															}
> 														],
> 														"options": {
> 															"election_tick": 10,
> 															"heartbeat_tick": 1,
> 															"max_inflight_blocks": 5,
> 															"snapshot_interval_size": 16777216,
> 															"tick_interval": "500ms"
> 														}
> 													},
> 													"state": "STATE_NORMAL",
> 													"type": "etcdraft"
> 												},
> 												"version": "0"
> 											}
> 										},
> 										"version": "0"
> 									}
> 								},
> 								"mod_policy": "Admins",
> 								"policies": {
> 									"Admins": {
> 										"mod_policy": "Admins",
> 										"policy": {
> 											"type": 3,
> 											"value": {
> 												"rule": "MAJORITY",
> 												"sub_policy": "Admins"
> 											}
> 										},
> 										"version": "0"
> 									},
> 									"Readers": {
> 										"mod_policy": "Admins",
> 										"policy": {
> 											"type": 3,
> 											"value": {
> 												"rule": "ANY",
> 												"sub_policy": "Readers"
> 											}
> 										},
> 										"version": "0"
> 									},
> 									"Writers": {
> 										"mod_policy": "Admins",
> 										"policy": {
> 											"type": 3,
> 											"value": {
> 												"rule": "ANY",
> 												"sub_policy": "Writers"
> 											}
> 										},
> 										"version": "0"
> 									}
> 								},
> 								"values": {
> 									"BlockDataHashingStructure": {
> 										"mod_policy": "Admins",
> 										"value": {
> 											"width": 4294967295
> 										},
> 										"version": "0"
> 									},
> 									"Capabilities": {
> 										"mod_policy": "Admins",
> 										"value": {
> 											"capabilities": {
> 												"V2_0": {}
> 											}
> 										},
> 										"version": "0"
> 									},
> 									"HashingAlgorithm": {
> 										"mod_policy": "Admins",
> 										"value": {
> 											"name": "SHA256"
> 										},
> 										"version": "0"
> 									},
> 									"OrdererAddresses": {
> 										"mod_policy": "/Channel/Orderer/Admins",
> 										"value": {
> 											"addresses": [
> 												"orderer0.example.com:7050",
> 												"orderer1.example.com:8050",
> 												"orderer2.example.com:9050"
> 											]
> 										},
> 										"version": "0"
> 									}
> 								},
> 								"version": "0"
> 							},
> 							"sequence": "0"
> 						},
> 						"last_update": null
> 					},
> 					"header": {
> 						"channel_header": {
> 							"channel_id": "fabric-channel",
> 							"epoch": "0",
> 							"extension": null,
> 							"timestamp": "2022-11-21T05:05:43Z",
> 							"tls_cert_hash": null,
> 							"tx_id": "8ddd8de1a83cd72a9d4e13906d6d381228220b35c4b5d6faebb519a7c6e61320",
> 							"type": 1,
> 							"version": 1
> 						},
> 						"signature_header": {
> 							"creator": null,
> 							"nonce": "53P+5Z3IPDa6GB8+6qcCiOeVilNymwqb"
> 						}
> 					}
> 				},
> 				"signature": null
> 			}
> 		]
> 	},
> 	"header": {
> 		"data_hash": "HejKmVEBDJrdSbXZhJFzIVjIiBDXWI0ZIge4vB+z0j4=",
> 		"number": "0",
> 		"previous_hash": null
> 	},
> 	"metadata": {
> 		"metadata": [
> 			"CgIKAA==",
> 			"",
> 			"",
> 			"",
> 			""
> 		]
> 	}
> }
> ```
>
> ![image-20221120172644437](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221120172644437.png)

> 其结构大概如下：
>
> 在 genesis.block 中也包含了相关的证书内容，如下面这段内容：
>
> ​        LS0tLS1CRUxxxxx                                 ],
> 对应的字符串即为经过 base64 编码的 证书。我们可以通过以下命令查看它：
>
> ```bash
> $ echo LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNZekNDQWdxZ0F3SUJBZ0lSQUxWTHhDcWE5aXVEWFJ3OHgrT0dkbW93Q2dZSUtvWkl6ajBFQXdJd2ZERUwKTUFrR0ExVUVCaE1DVlZNeEV6QVJCZ05WQkFnVENrTmhiR2xtYjNKdWFXRXhGakFVQmdOVkJBY1REVk5oYmlCRwpjbUZ1WTJselkyOHhIREFhQmdOVkJBb1RFMjl5WnpFdWMzVndaWEoyYVhOdmNpNWpiMjB4SWpBZ0JnTlZCQU1UCkdYUnNjMk5oTG05eVp6RXVjM1Z3WlhKMmFYTnZjaTVqYjIwd0hoY05Nakl4TVRFNE1UVXdOVEF3V2hjTk16SXgKTVRFMU1UVXdOVEF3V2pCOE1Rc3dDUVlEVlFRR0V3SlZVekVUTUJFR0ExVUVDQk1LUTJGc2FXWnZjbTVwWVRFVwpNQlFHQTFVRUJ4TU5VMkZ1SUVaeVlXNWphWE5qYnpFY01Cb0dBMVVFQ2hNVGIzSm5NUzV6ZFhCbGNuWnBjMjl5CkxtTnZiVEVpTUNBR0ExVUVBeE1aZEd4elkyRXViM0puTVM1emRYQmxjblpwYzI5eUxtTnZiVEJaTUJNR0J5cUcKU000OUFnRUdDQ3FHU000OUF3RUhBMElBQkhZUkpBRzlVNE9qbGlGUW9OWmJyNWh2TU1CY0dLYWVaT1NWNldzcQpBRWFqSjRmM3dZdWdzR1gwYkluczh5aDMrNThLWmJsZlZqN2JJZGtENGh5UUlUNmpiVEJyTUE0R0ExVWREd0VCCi93UUVBd0lCcGpBZEJnTlZIU1VFRmpBVUJnZ3JCZ0VGQlFjREFnWUlLd1lCQlFVSEF3RXdEd1lEVlIwVEFRSC8KQkFVd0F3RUIvekFwQmdOVkhRNEVJZ1FnMkRwcWZrOXBGZVZnZ0F6eWUzaU11QlpSV24vVzVpNmFTRWFpK04vKwpRVmd3Q2dZSUtvWkl6ajBFQXdJRFJ3QXdSQUlnVVNDUGM5ZnlUTFpHUWRuWStudUFFV1Rrak5uV1BkVXdHd1I0CnoxQjR6eVlDSUYvei9KNDhVd0pCL0htMWZGWnYyL1ZJZjYxWXNuQVR6ZjF6eFlVdUJwNlUKLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo= | base64 -d > test.pem 
> 
> $ openssl x509 -in test.pem -text -noout
> ```
>
> ![image-20221120181037732](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221120181037732.png)
>
> *X.509*证书格式：
>
> ```json
> Certificate:
> Data:
>   Version: 3 (0x2)
>   Serial Number:
>       b5:4b:c4:2a:9a:f6:2b:83:5d:1c:3c:c7:e3:86:76:6a
>   Signature Algorithm: ecdsa-with-SHA256
>   Issuer: C = US, ST = California, L = San Francisco, O = org1.supervisor.com, CN = tlsca.org1.supervisor.com
>   Validity
>       Not Before: Nov 18 15:05:00 2022 GMT
>       Not After : Nov 15 15:05:00 2032 GMT
>   Subject: C = US, ST = California, L = San Francisco, O = org1.supervisor.com, CN = tlsca.org1.supervisor.com
>   Subject Public Key Info:
>       Public Key Algorithm: id-ecPublicKey
>           Public-Key: (256 bit)
>           pub:
>               04:76:11:24:01:bd:53:83:a3:96:21:50:a0:d6:5b:
>               af:98:6f:30:c0:5c:18:a6:9e:64:e4:95:e9:6b:2a:
>               00:46:a3:27:87:f7:c1:8b:a0:b0:65:f4:6c:89:ec:
>               f3:28:77:fb:9f:0a:65:b9:5f:56:3e:db:21:d9:03:
>               e2:1c:90:21:3e
>           ASN1 OID: prime256v1
>           NIST CURVE: P-256
>   X509v3 extensions:
>       X509v3 Key Usage: critical
>           Digital Signature, Key Encipherment, Certificate Sign, CRL Sign
>       X509v3 Extended Key Usage: 
>           TLS Web Client Authentication, TLS Web Server Authentication
>       X509v3 Basic Constraints: critical
>           CA:TRUE
>       X509v3 Subject Key Identifier: 
>           D8:3A:6A:7E:4F:69:15:E5:60:80:0C:F2:7B:78:8C:B8:16:51:5A:7F:D6:E6:2E:9A:48:46:A2:F8:DF:FE:41:58
> Signature Algorithm: ecdsa-with-SHA256
>    30:44:02:20:51:20:8f:73:d7:f2:4c:b6:46:41:d9:d8:fa:7b:
>    80:11:64:e4:8c:d9:d6:3d:d5:30:1b:04:78:cf:50:78:cf:26:
>    02:20:5f:f3:fc:9e:3c:53:02:41:fc:79:b5:7c:56:6f:db:f5:
>    48:7f:ad:58:b2:70:13:cd:fd:73:c5:85:2e:06:9e:94
> ```
>
> ![image-20221120181337833](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221120181337833.png)
>
> 
>
> ```bash
> $ echo LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUNYRENDQWdLZ0F3SUJBZ0lRUWJGNzV2TGRRTkhzUTY2WmdnL2N0akFLQmdncWhrak9QUVFEQWpCc01Rc3cKQ1FZRFZRUUdFd0pWVXpFVE1CRUdBMVVFQ0JNS1EyRnNhV1p2Y201cFlURVdNQlFHQTFVRUJ4TU5VMkZ1SUVaeQpZVzVqYVhOamJ6RVVNQklHQTFVRUNoTUxaWGhoYlhCc1pTNWpiMjB4R2pBWUJnTlZCQU1URVhSc2MyTmhMbVY0CllXMXdiR1V1WTI5dE1CNFhEVEl5TVRFeE9ERTFNRFV3TUZvWERUTXlNVEV4TlRFMU1EVXdNRm93V1RFTE1Ba0cKQTFVRUJoTUNWVk14RXpBUkJnTlZCQWdUQ2tOaGJHbG1iM0p1YVdFeEZqQVVCZ05WQkFjVERWTmhiaUJHY21GdQpZMmx6WTI4eEhUQWJCZ05WQkFNVEZHOXlaR1Z5WlhJeExtVjRZVzF3YkdVdVkyOXRNRmt3RXdZSEtvWkl6ajBDCkFRWUlLb1pJemowREFRY0RRZ0FFZCtmenpybEwyMmpBVElnWXREUXlKV1JvRTRMeTdWbk56TXJLbFZkSjRSNTQKZEVTcVdwL0lpM1ZqNEhBZEhybWlVcHBub3JtZ3RXNStYNTIxeXJHRmlxT0JtRENCbFRBT0JnTlZIUThCQWY4RQpCQU1DQmFBd0hRWURWUjBsQkJZd0ZBWUlLd1lCQlFVSEF3RUdDQ3NHQVFVRkJ3TUNNQXdHQTFVZEV3RUIvd1FDCk1BQXdLd1lEVlIwakJDUXdJb0Fnd09ORWt2Z2JWang2RE5LNEMzbk9VOWJXazRIc1hwaE1ZZWF2eSthNzdySXcKS1FZRFZSMFJCQ0l3SUlJVWIzSmtaWEpsY2pFdVpYaGhiWEJzWlM1amIyMkNDRzl5WkdWeVpYSXhNQW9HQ0NxRwpTTTQ5QkFNQ0EwZ0FNRVVDSVFDN1MwVGhRQ2UvVE5BTlB6VmtMcWFac2RyMlUwZ3FzR21JWWdRVE5zdGxMUUlnCll3NVZ2cFczUDVweFFPRFZyVXNLbXFVQUhrT0txeWgwOVRLUEhTVVNtTjA9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K= | base64 -d > test1.pem 
> ```
>
> 
>
> ### 二、查看交易（tx）文件：
>
> 1.生成文件，具体查看generateArtifacts.sh中的步骤
>
> 2.将 transaction 导出到 JSON 文件进行查看：
>
> ```bash
> configtxgen -inspectChannelCreateTx channel-artifacts/channel.tx > channel.tx.json
> configtxgen -inspectChannelCreateTx channel-artifacts/Org1MSPanchors.tx > Org1MSPanchors.tx.json
> configtxgen -inspectChannelCreateTx channel-artifacts/Org2MSPanchors.tx > Org2MSPanchors.tx.json
> ```
>
> ![image-20221120172120664](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221120172120664.png)
>
> 锚节点文件
>
> ![image-20221120172342194](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221120172342194.png)



## 4.编写*docker-compose.yaml*

### 4.1 *docker-compose.yaml* 文件配置

在*raft-test*目录下，新建*docker-compose.yaml*文件。文件内容如下面所示。

> *docker-compose.yaml*文件内容

```yaml
version: '3.7'

volumes:
  orderer0.example.com:
  orderer1.example.com:
  orderer2.example.com:
  peer0.org1.supervisor.com:
  peer1.org1.supervisor.com:
  peer0.org2.build.com:
  peer1.org2.build.com:
  peer0.org3.supplier.com:
  peer1.org3.supplier.com:
  peer0.org4.logistics.com:
  peer1.org4.logistics.com:

networks:
  test:
    name: ordersFabric_test

services:

  orderer0.example.com:
    container_name: orderer0.example.com
    image: hyperledger/fabric-orderer:latest
    labels:
      service: hyperledger-fabric
    environment:
      - FABRIC_LOGGING_SPEC=INFO
      - ORDERER_GENERAL_LISTENADDRESS=0.0.0.0
      - ORDERER_GENERAL_LISTENPORT=7050
      - ORDERER_GENERAL_LOCALMSPID=OrdererMSP
      - ORDERER_GENERAL_LOCALMSPDIR=/var/hyperledger/orderer/msp
      # enabled TLS
      - ORDERER_GENERAL_TLS_ENABLED=true
      - ORDERER_GENERAL_TLS_PRIVATEKEY=/var/hyperledger/orderer/tls/server.key
      - ORDERER_GENERAL_TLS_CERTIFICATE=/var/hyperledger/orderer/tls/server.crt
      - ORDERER_GENERAL_TLS_ROOTCAS=[/var/hyperledger/orderer/tls/ca.crt]
      
      - ORDERER_GENERAL_CLUSTER_CLIENTCERTIFICATE=/var/hyperledger/orderer/tls/server.crt
      - ORDERER_GENERAL_CLUSTER_CLIENTPRIVATEKEY=/var/hyperledger/orderer/tls/server.key
      - ORDERER_GENERAL_CLUSTER_ROOTCAS=[/var/hyperledger/orderer/tls/ca.crt]
      
      - ORDERER_GENERAL_GENESISMETHOD=file
      - ORDERER_GENERAL_GENESISFILE=/var/hyperledger/orderer/orderer.genesis.block
      
      - ORDERER_CHANNELPARTICIPATION_ENABLED=true
      
      - ORDERER_ADMIN_TLS_ENABLED=true
      - ORDERER_ADMIN_TLS_CERTIFICATE=/var/hyperledger/orderer/tls/server.crt
      - ORDERER_ADMIN_TLS_PRIVATEKEY=/var/hyperledger/orderer/tls/server.key
      - ORDERER_ADMIN_TLS_ROOTCAS=[/var/hyperledger/orderer/tls/ca.crt]
      - ORDERER_ADMIN_TLS_CLIENTROOTCAS=[/var/hyperledger/orderer/tls/ca.crt]
      - ORDERER_ADMIN_LISTENADDRESS=0.0.0.0:7053
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric
    command: orderer
    volumes:
      - ./channel-artifacts/genesis.block:/var/hyperledger/orderer/orderer.genesis.block
      - ./crypto-config/ordererOrganizations/example.com/orderers/orderer0.example.com/msp:/var/hyperledger/orderer/msp
      - ./crypto-config/ordererOrganizations/example.com/orderers/orderer0.example.com/tls/:/var/hyperledger/orderer/tls
    ports:
      - 7050:7050
      - 7053:7053
    networks:
      - test

  orderer1.example.com:
    container_name: orderer1.example.com
    image: hyperledger/fabric-orderer:latest
    labels:
      service: hyperledger-fabric
    environment:
      - FABRIC_LOGGING_SPEC=INFO
      - ORDERER_GENERAL_LISTENADDRESS=0.0.0.0
      - ORDERER_GENERAL_LISTENPORT=8050
      - ORDERER_GENERAL_LOCALMSPID=OrdererMSP
      - ORDERER_GENERAL_LOCALMSPDIR=/var/hyperledger/orderer/msp
      # enabled TLS
      - ORDERER_GENERAL_TLS_ENABLED=true
      - ORDERER_GENERAL_TLS_PRIVATEKEY=/var/hyperledger/orderer/tls/server.key
      - ORDERER_GENERAL_TLS_CERTIFICATE=/var/hyperledger/orderer/tls/server.crt
      - ORDERER_GENERAL_TLS_ROOTCAS=[/var/hyperledger/orderer/tls/ca.crt]
     
      - ORDERER_GENERAL_CLUSTER_CLIENTCERTIFICATE=/var/hyperledger/orderer/tls/server.crt
      - ORDERER_GENERAL_CLUSTER_CLIENTPRIVATEKEY=/var/hyperledger/orderer/tls/server.key
      - ORDERER_GENERAL_CLUSTER_ROOTCAS=[/var/hyperledger/orderer/tls/ca.crt]
      - ORDERER_GENERAL_GENESISMETHOD=file
      - ORDERER_GENERAL_GENESISFILE=/var/hyperledger/orderer/orderer.genesis.block
      - ORDERER_CHANNELPARTICIPATION_ENABLED=true
      - ORDERER_ADMIN_TLS_ENABLED=true
      - ORDERER_ADMIN_TLS_CERTIFICATE=/var/hyperledger/orderer/tls/server.crt
      - ORDERER_ADMIN_TLS_PRIVATEKEY=/var/hyperledger/orderer/tls/server.key
      - ORDERER_ADMIN_TLS_ROOTCAS=[/var/hyperledger/orderer/tls/ca.crt]
      - ORDERER_ADMIN_TLS_CLIENTROOTCAS=[/var/hyperledger/orderer/tls/ca.crt]
      - ORDERER_ADMIN_LISTENADDRESS=0.0.0.0:7053
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric
    command: orderer
    volumes:
      - ./channel-artifacts/genesis.block:/var/hyperledger/orderer/orderer.genesis.block
      - ./crypto-config/ordererOrganizations/example.com/orderers/orderer1.example.com/msp:/var/hyperledger/orderer/msp
      - ./crypto-config/ordererOrganizations/example.com/orderers/orderer1.example.com/tls/:/var/hyperledger/orderer/tls
    ports:
      - 8050:8050
      - 7054:7053
    networks:
      - test

  orderer2.example.com:
    container_name: orderer2.example.com
    image: hyperledger/fabric-orderer:latest
    labels:
      service: hyperledger-fabric
    environment:
      - FABRIC_LOGGING_SPEC=INFO
      - ORDERER_GENERAL_LISTENADDRESS=0.0.0.0
      - ORDERER_GENERAL_LISTENPORT=9050
      - ORDERER_GENERAL_LOCALMSPID=OrdererMSP
      - ORDERER_GENERAL_LOCALMSPDIR=/var/hyperledger/orderer/msp
      # enabled TLS
      - ORDERER_GENERAL_TLS_ENABLED=true
      - ORDERER_GENERAL_TLS_PRIVATEKEY=/var/hyperledger/orderer/tls/server.key
      - ORDERER_GENERAL_TLS_CERTIFICATE=/var/hyperledger/orderer/tls/server.crt
      - ORDERER_GENERAL_TLS_ROOTCAS=[/var/hyperledger/orderer/tls/ca.crt]
     
      - ORDERER_GENERAL_CLUSTER_CLIENTCERTIFICATE=/var/hyperledger/orderer/tls/server.crt
      - ORDERER_GENERAL_CLUSTER_CLIENTPRIVATEKEY=/var/hyperledger/orderer/tls/server.key
      - ORDERER_GENERAL_CLUSTER_ROOTCAS=[/var/hyperledger/orderer/tls/ca.crt]
      - ORDERER_GENERAL_GENESISMETHOD=file
      - ORDERER_GENERAL_GENESISFILE=/var/hyperledger/orderer/orderer.genesis.block
      - ORDERER_CHANNELPARTICIPATION_ENABLED=true
      - ORDERER_ADMIN_TLS_ENABLED=true
      - ORDERER_ADMIN_TLS_CERTIFICATE=/var/hyperledger/orderer/tls/server.crt
      - ORDERER_ADMIN_TLS_PRIVATEKEY=/var/hyperledger/orderer/tls/server.key
      - ORDERER_ADMIN_TLS_ROOTCAS=[/var/hyperledger/orderer/tls/ca.crt]
      - ORDERER_ADMIN_TLS_CLIENTROOTCAS=[/var/hyperledger/orderer/tls/ca.crt]
      - ORDERER_ADMIN_LISTENADDRESS=0.0.0.0:7053
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric
    command: orderer
    volumes:
      - ./channel-artifacts/genesis.block:/var/hyperledger/orderer/orderer.genesis.block
      - ./crypto-config/ordererOrganizations/example.com/orderers/orderer2.example.com/msp:/var/hyperledger/orderer/msp
      - ./crypto-config/ordererOrganizations/example.com/orderers/orderer2.example.com/tls/:/var/hyperledger/orderer/tls
    ports:
      - 9050:9050
      - 7055:7053
    networks:
      - test

  peer0.org1.supervisor.com:
    container_name: peer0.org1.supervisor.com
    image: hyperledger/fabric-peer:latest
    labels:
      service: hyperledger-fabric
    environment:
      #Generic peer variables
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=ordersFabric_test
      - FABRIC_LOGGING_SPEC=INFO
      #- FABRIC_LOGGING_SPEC=DEBUG
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_PROFILE_ENABLED=true
      - CORE_PEER_TLS_CERT_FILE=/etc/hyperledger/fabric/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/etc/hyperledger/fabric/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/etc/hyperledger/fabric/tls/ca.crt
      # Peer specific variabes
      - CORE_PEER_ID=peer0.org1.supervisor.com
      - CORE_PEER_ADDRESS=peer0.org1.supervisor.com:7051
      - CORE_PEER_LISTENADDRESS=0.0.0.0:7051
      - CORE_PEER_CHAINCODEADDRESS=peer0.org1.supervisor.com:7052
      - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:7052
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer1.org1.supervisor.com:8051
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.org1.supervisor.com:7051
      - CORE_PEER_LOCALMSPID=Org1MSP
    volumes:
      - /var/run/docker.sock:/host/var/run/docker.sock
      - ./crypto-config/peerOrganizations/org1.supervisor.com/peers/peer0.org1.supervisor.com/msp:/etc/hyperledger/fabric/msp
      - ./crypto-config/peerOrganizations/org1.supervisor.com/peers/peer0.org1.supervisor.com/tls:/etc/hyperledger/fabric/tls
      - /var/run/:/host/var/run/
      - ./:/etc/hyperledger/channel/
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: peer node start
    ports:
      - 7051:7051
    networks:
      - test

  peer1.org1.supervisor.com:
    container_name: peer1.org1.supervisor.com
    image: hyperledger/fabric-peer:latest
    labels:
      service: hyperledger-fabric
    environment:
      #Generic peer variables
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=ordersFabric_test
      - FABRIC_LOGGING_SPEC=INFO
      #- FABRIC_LOGGING_SPEC=DEBUG
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_PROFILE_ENABLED=true
      - CORE_PEER_TLS_CERT_FILE=/etc/hyperledger/fabric/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/etc/hyperledger/fabric/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/etc/hyperledger/fabric/tls/ca.crt
      # Peer specific variabes
      - CORE_PEER_ID=peer1.org1.supervisor.com
      - CORE_PEER_ADDRESS=peer1.org1.supervisor.com:8051
      - CORE_PEER_LISTENADDRESS=0.0.0.0:8051
      - CORE_PEER_CHAINCODEADDRESS=peer1.org1.supervisor.com:8052
      - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:8052
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer0.org1.supervisor.com:7051
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer1.org1.supervisor.com:8051
      - CORE_PEER_LOCALMSPID=Org1MSP
    volumes:
      - /var/run/docker.sock:/host/var/run/docker.sock
      - ./crypto-config/peerOrganizations/org1.supervisor.com/peers/peer1.org1.supervisor.com/msp:/etc/hyperledger/fabric/msp
      - ./crypto-config/peerOrganizations/org1.supervisor.com/peers/peer1.org1.supervisor.com/tls:/etc/hyperledger/fabric/tls
      - /var/run/:/host/var/run/
      - ./:/etc/hyperledger/channel/
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: peer node start
    ports:
      - 8051:8051
    networks:
      - test

  peer0.org2.build.com:
    container_name: peer0.org2.build.com
    image: hyperledger/fabric-peer:latest
    labels:
      service: hyperledger-fabric
    environment:
      #Generic peer variables
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=ordersFabric_test
      - FABRIC_LOGGING_SPEC=INFO
      #- FABRIC_LOGGING_SPEC=DEBUG
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_PROFILE_ENABLED=true
      - CORE_PEER_TLS_CERT_FILE=/etc/hyperledger/fabric/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/etc/hyperledger/fabric/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/etc/hyperledger/fabric/tls/ca.crt
      # Peer specific variabes
      - CORE_PEER_ID=peer0.org2.build.com
      - CORE_PEER_ADDRESS=peer0.org2.build.com:9051
      - CORE_PEER_LISTENADDRESS=0.0.0.0:9051
      - CORE_PEER_CHAINCODEADDRESS=peer0.org2.build.com:9052
      - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:9052
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer1.org2.build.com:10051
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.org2.build.com:9051
      - CORE_PEER_LOCALMSPID=Org2MSP
    volumes:
      - /var/run/docker.sock:/host/var/run/docker.sock
      - ./crypto-config/peerOrganizations/org2.build.com/peers/peer0.org2.build.com/msp:/etc/hyperledger/fabric/msp
      - ./crypto-config/peerOrganizations/org2.build.com/peers/peer0.org2.build.com/tls:/etc/hyperledger/fabric/tls
      - /var/run/:/host/var/run/
      - ./:/etc/hyperledger/channel/
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: peer node start
    ports:
      - 9051:9051
    networks:
      - test

  peer1.org2.build.com:
    container_name: peer1.org2.build.com
    image: hyperledger/fabric-peer:latest
    labels:
      service: hyperledger-fabric
    environment:
      #Generic peer variables
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=ordersFabric_test
      - FABRIC_LOGGING_SPEC=INFO
      #- FABRIC_LOGGING_SPEC=DEBUG
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_PROFILE_ENABLED=true
      - CORE_PEER_TLS_CERT_FILE=/etc/hyperledger/fabric/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/etc/hyperledger/fabric/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/etc/hyperledger/fabric/tls/ca.crt
      # Peer specific variabes
      - CORE_PEER_ID=peer1.org2.build.com
      - CORE_PEER_ADDRESS=peer1.org2.build.com:10051
      - CORE_PEER_LISTENADDRESS=0.0.0.0:10051
      - CORE_PEER_CHAINCODEADDRESS=peer1.org2.build.com:10052
      - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:10052
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer0.org2.build.com:9051
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer1.org2.build.com:10051
      - CORE_PEER_LOCALMSPID=Org2MSP
    volumes:
      - /var/run/docker.sock:/host/var/run/docker.sock
      - ./crypto-config/peerOrganizations/org2.build.com/peers/peer1.org2.build.com/msp:/etc/hyperledger/fabric/msp
      - ./crypto-config/peerOrganizations/org2.build.com/peers/peer1.org2.build.com/tls:/etc/hyperledger/fabric/tls
      - /var/run/:/host/var/run/
      - ./:/etc/hyperledger/channel/
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: peer node start
    ports:
      - 10051:10051
    networks:
      - test

  peer0.org3.supplier.com:
    container_name: peer0.org3.supplier.com
    image: hyperledger/fabric-peer:latest
    labels:
      service: hyperledger-fabric
    environment:
      #Generic peer variables
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=ordersFabric_test
      - FABRIC_LOGGING_SPEC=INFO
      #- FABRIC_LOGGING_SPEC=DEBUG
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_PROFILE_ENABLED=true
      - CORE_PEER_TLS_CERT_FILE=/etc/hyperledger/fabric/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/etc/hyperledger/fabric/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/etc/hyperledger/fabric/tls/ca.crt
      # Peer specific variabes
      - CORE_PEER_ID=peer0.org3.supplier.com
      - CORE_PEER_ADDRESS=peer0.org3.supplier.com:11051
      - CORE_PEER_LISTENADDRESS=0.0.0.0:11051
      - CORE_PEER_CHAINCODEADDRESS=peer0.org3.supplier.com:11052
      - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:11052
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer1.org3.supplier.com:12051
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.org3.supplier.com:11051
      - CORE_PEER_LOCALMSPID=Org3MSP
    volumes:
      - /var/run/docker.sock:/host/var/run/docker.sock
      - ./crypto-config/peerOrganizations/org3.supplier.com/peers/peer0.org3.supplier.com/msp:/etc/hyperledger/fabric/msp
      - ./crypto-config/peerOrganizations/org3.supplier.com/peers/peer0.org3.supplier.com/tls:/etc/hyperledger/fabric/tls
      - /var/run/:/host/var/run/
      - ./:/etc/hyperledger/channel/
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: peer node start
    ports:
      - 11051:11051
    networks:
      - test

  peer1.org3.supplier.com:
    container_name: peer1.org3.supplier.com
    image: hyperledger/fabric-peer:latest
    labels:
      service: hyperledger-fabric
    environment:
      #Generic peer variables
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=ordersFabric_test
      - FABRIC_LOGGING_SPEC=INFO
      #- FABRIC_LOGGING_SPEC=DEBUG
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_PROFILE_ENABLED=true
      - CORE_PEER_TLS_CERT_FILE=/etc/hyperledger/fabric/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/etc/hyperledger/fabric/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/etc/hyperledger/fabric/tls/ca.crt
      # Peer specific variabes
      - CORE_PEER_ID=peer1.org3.supplier.com
      - CORE_PEER_ADDRESS=peer1.org3.supplier.com:12051
      - CORE_PEER_LISTENADDRESS=0.0.0.0:12051
      - CORE_PEER_CHAINCODEADDRESS=peer1.org3.supplier.com:12052
      - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:12052
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer0.org3.supplier.com:11051
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer1.org3.supplier.com:12051
      - CORE_PEER_LOCALMSPID=Org3MSP
    volumes:
      - /var/run/docker.sock:/host/var/run/docker.sock
      - ./crypto-config/peerOrganizations/org3.supplier.com/peers/peer1.org3.supplier.com/msp:/etc/hyperledger/fabric/msp
      - ./crypto-config/peerOrganizations/org3.supplier.com/peers/peer1.org3.supplier.com/tls:/etc/hyperledger/fabric/tls
      - /var/run/:/host/var/run/
      - ./:/etc/hyperledger/channel/
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: peer node start
    ports:
      - 12051:12051
    networks:
      - test

  peer0.org4.logistics.com:
    container_name: peer0.org4.logistics.com
    image: hyperledger/fabric-peer:latest
    labels:
      service: hyperledger-fabric
    environment:
      #Generic peer variables
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=ordersFabric_test
      - FABRIC_LOGGING_SPEC=INFO
      #- FABRIC_LOGGING_SPEC=DEBUG
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_PROFILE_ENABLED=true
      - CORE_PEER_TLS_CERT_FILE=/etc/hyperledger/fabric/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/etc/hyperledger/fabric/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/etc/hyperledger/fabric/tls/ca.crt
      # Peer specific variabes
      - CORE_PEER_ID=peer0.org4.logistics.com
      - CORE_PEER_ADDRESS=peer0.org4.logistics.com:13051
      - CORE_PEER_LISTENADDRESS=0.0.0.0:13051
      - CORE_PEER_CHAINCODEADDRESS=peer0.org4.logistics.com:13052
      - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:13052
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer1.org4.logistics.com:14051
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.org4.logistics.com:13051
      - CORE_PEER_LOCALMSPID=Org4MSP
    volumes:
      - /var/run/docker.sock:/host/var/run/docker.sock
      - ./crypto-config/peerOrganizations/org4.logistics.com/peers/peer0.org4.logistics.com/msp:/etc/hyperledger/fabric/msp
      - ./crypto-config/peerOrganizations/org4.logistics.com/peers/peer0.org4.logistics.com/tls:/etc/hyperledger/fabric/tls
      - /var/run/:/host/var/run/
      - ./:/etc/hyperledger/channel/
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: peer node start
    ports:
      - 13051:13051
    networks:
      - test

  peer1.org4.logistics.com:
    container_name: peer1.org4.logistics.com
    image: hyperledger/fabric-peer:latest
    labels:
      service: hyperledger-fabric
    environment:
      #Generic peer variables
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=ordersFabric_test
      - FABRIC_LOGGING_SPEC=INFO
      #- FABRIC_LOGGING_SPEC=DEBUG
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_PROFILE_ENABLED=true
      - CORE_PEER_TLS_CERT_FILE=/etc/hyperledger/fabric/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/etc/hyperledger/fabric/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/etc/hyperledger/fabric/tls/ca.crt
      # Peer specific variabes
      - CORE_PEER_ID=peer1.org4.logistics.com
      - CORE_PEER_ADDRESS=peer1.org4.logistics.com:14051
      - CORE_PEER_LISTENADDRESS=0.0.0.0:14051
      - CORE_PEER_CHAINCODEADDRESS=peer1.org4.logistics.com:14052
      - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:14052
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer0.org4.logistics.com:13051
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer1.org4.logistics.com:14051
      - CORE_PEER_LOCALMSPID=Org4MSP
    volumes:
      - /var/run/docker.sock:/host/var/run/docker.sock
      - ./crypto-config/peerOrganizations/org4.logistics.com/peers/peer1.org4.logistics.com/msp:/etc/hyperledger/fabric/msp
      - ./crypto-config/peerOrganizations/org4.logistics.com/peers/peer1.org4.logistics.com/tls:/etc/hyperledger/fabric/tls
      - /var/run/:/host/var/run/
      - ./:/etc/hyperledger/channel/
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: peer node start
    ports:
      - 14051:14051
    networks:
      - test

  cli1:
    container_name: cli1
    image: hyperledger/fabric-tools:latest
    tty: true
    stdin_open: true
    environment:
      - GOPATH=/opt/gopath
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - FABRIC_LOGGING_SPEC=INFO
      - CORE_PEER_ID=cli1
      - CORE_PEER_ADDRESS=peer0.org1.supervisor.com:7051
      - CORE_PEER_LOCALMSPID=Org1MSP
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.supervisor.com/peers/peer0.org1.supervisor.com/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.supervisor.com/peers/peer0.org1.supervisor.com/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.supervisor.com/peers/peer0.org1.supervisor.com/tls/ca.crt
      - CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.supervisor.com/users/Admin@org1.supervisor.com/msp
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: /bin/bash
    volumes:
      - /var/run/:/host/var/run/
      - ./chaincode/go/:/opt/gopath/src/github.com/hyperledger/fabric-cluster/chaincode/go
      - ./crypto-config:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/
      - ./channel-artifacts:/opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts
    networks:
      - test

  cli2:
    container_name: cli2
    image: hyperledger/fabric-tools:latest
    tty: true
    stdin_open: true
    environment:
      - GOPATH=/opt/gopath
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - FABRIC_LOGGING_SPEC=INFO
      - CORE_PEER_ID=cli2
      - CORE_PEER_ADDRESS=peer0.org2.build.com:9051
      - CORE_PEER_LOCALMSPID=Org2MSP
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.build.com/peers/peer0.org2.build.com/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.build.com/peers/peer0.org2.build.com/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.build.com/peers/peer0.org2.build.com/tls/ca.crt
      - CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.build.com/users/Admin@org2.build.com/msp
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: /bin/bash
    volumes:
      - /var/run/:/host/var/run/
      - ./chaincode/go/:/opt/gopath/src/github.com/hyperledger/fabric-cluster/chaincode/go
      - ./crypto-config:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/
      - ./channel-artifacts:/opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts
    networks:
      - test

  cli3:
    container_name: cli3
    image: hyperledger/fabric-tools:latest
    tty: true
    stdin_open: true
    environment:
      - GOPATH=/opt/gopath
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - FABRIC_LOGGING_SPEC=INFO
      - CORE_PEER_ID=cli3
      - CORE_PEER_ADDRESS=peer0.org3.supplier.com:11051
      - CORE_PEER_LOCALMSPID=Org3MSP
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.supplier.com/peers/peer0.org3.supplier.com/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.supplier.com/peers/peer0.org3.supplier.com/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.supplier.com/peers/peer0.org3.supplier.com/tls/ca.crt
      - CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.supplier.com/users/Admin@org3.supplier.com/msp
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: /bin/bash
    volumes:
      - /var/run/:/host/var/run/
      - ./chaincode/go/:/opt/gopath/src/github.com/hyperledger/fabric-cluster/chaincode/go
      - ./crypto-config:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/
      - ./channel-artifacts:/opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts
    networks:
      - test

  cli4:
    container_name: cli4
    image: hyperledger/fabric-tools:latest
    tty: true
    stdin_open: true
    environment:
      - GOPATH=/opt/gopath
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - FABRIC_LOGGING_SPEC=INFO
      - CORE_PEER_ID=cli4
      - CORE_PEER_ADDRESS=peer0.org4.logistics.com:13051
      - CORE_PEER_LOCALMSPID=Org4MSP
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org4.logistics.com/peers/peer0.org4.logistics.com/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org4.logistics.com/peers/peer0.org4.logistics.com/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org4.logistics.com/peers/peer0.org4.logistics.com/tls/ca.crt
      - CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org4.logistics.com/users/Admin@org4.logistics.com/msp
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: /bin/bash
    volumes:
      - /var/run/:/host/var/run/
      - ./chaincode/go/:/opt/gopath/src/github.com/hyperledger/fabric-cluster/chaincode/go
      - ./crypto-config:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/
      - ./channel-artifacts:/opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts
    networks:
      - test
```

![image-20221118234203209](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221118234203209.png)

### 4.2 *docker-compose*容器网络运行

利用docker-compose命令来生成镜像：

```sh
$ docker-compose up -d
$ docker ps -a
```

![image-20221118234430203](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221118234430203.png)



![image-20221118234515863](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221118234515863.png)



如果在后面需要删除容器可以使用以下命令：

```bash
$ docker rm $(docker ps -a -q)

$ docker-compose up -d
```



## 5.通道相关配置操作

> ## Fabric源码分析-生成Channel.tx通道初始配置文件
>
> Channel提供了多账本的功能，我们创建Channel时首先就是要配置联盟和Channel中包含的组织，之后使用configtxgen生成Channel.tx文件，这个文件内部其实就是创建账本所需要的配置更新。
>
> ### 1.使用
>
> 从fabric源码中复制一份configtx，对其进行修改
>
> ```bash
> cp $GOPATH/src/github.com/hyperledger/fabric/sampleconfig/configtx.yaml .
> ```
>
> 设置一个生成Channe的配置，OrdererOrg/CoreOrg/SupplierOrg/BankOrg需要进行配置，在此省略
>
> ```yaml
> TestTwoOrgsChannel:
>      Consortium: SampleConsortium
>      Application:
>          Organizations:
>                 - *CoreOrg
>                 - *SupplierOrg
>                 - *BankOrg
> ```
>
> 最后，使用下面的命令就可以生成创世块了
>
> ```sh
> configtxgen -profile TestTwoOrgsChannel -outputCreateChannelTx ./mychannel.tx -channelID mychannel
> ```
>
> ### 2.程序分析
>
> configtxgen的程序在`common/tools/configtxgen/main.go`中，主要的功能是获取配置文件，生成ConfigUpdate，包装为一个ConfigUpdate的交易并写入文件。主流程是：
>
> 1. factory.InitFactories(nil) 初始化BCCSP，用来为加密提供服务。
> 2. 获取指定profile的配置，将其序列化为Profile类型。
> 3. 我们设置了outputChannelCreateTx，因此会执行doOutputChannelCreateTx方法。
> 4. 创建一个用来创建Channel的交易，将其写入文件。
>
> #### 2.1 创建Channel的交易文件
>
> 有了Profile配置，首先创建一个ConfigUpdate，将其包装到ConfigUpdateEnvelope中，之后序列化作为Data，添加Header后形成Payload，最后添加签名，封装为Envelope并返回。
>
> ![img](https://upload-images.jianshu.io/upload_images/4747194-3c3734f17aca8c23.png?imageMogr2/auto-orient/strip|imageView2/2/w/764/format/webp)
>
> channel.tx的内容
>
> 因此，我们需要将主要逻辑聚焦于ConfigUpdate的内容。
>
> #### 2.2 ConfigGroup
>
> `NewChannelCreateConfigUpdate`方法会生成一个ConfigUpdate，ConfigUpdate会被发送到orderer来创建一个新的Channel，该方法根据指定的profile的配置中的Application，创建Application的ConfigGroup，ConfigGroup可以理解为一个配置树型，Application中包含了Organizations，Organizations中是三个Org，因此，最终会形成一个Application的配置树。ConfigGroup的结构可以参考[ConfigGroup](https://www.jianshu.com/p/65230bbffb58)，Application的Groups中是组织的配置，在`NewApplicationOrgGroup`中，会对Org的配置组装，读取并验证MSP的相关证书。
>
> #### 2.3 ConfigUpdate
>
> 上面只是将配置转化为ConfigGroup组成的配置树，接下来就是计算ConfigUpdate了，也就是计算配置有那些更新。计算的程序如下，首先会创建一个新的ConfigGroup，其Groups子配置有一个`Application`，对应这我们之前生成的Application的Config，这是因为在`TestTwoOrgsChannel`中，Application是作为一个子级，需要将其放在一个Root配置下。
>
> ```go
> newChannelGroup = &cb.ConfigGroup{
>   Groups: map[string]*cb.ConfigGroup{
>     channelconfig.ApplicationGroupKey: ag,
>   },
> }
> 
> template = proto.Clone(newChannelGroup).(*cb.ConfigGroup)
> template.Groups[channelconfig.ApplicationGroupKey].Values = nil
> template.Groups[channelconfig.ApplicationGroupKey].Policies = nil
> ```
>
> 之后，克隆了一份新创建的ConfigGroup，将其`Application`的子ConfigGroup的Value和Policy设置为nil，之前我们在2.2节构造ConfigGroup时，ConfigGroup会设置默认的一些Ploicy，如果配置了Capabilities，已经设置Value
>
> ```go
> addImplicitMetaPolicyDefaults(applicationGroup)
> 
> if len(conf.Capabilities) > 0 {
>   addValue(applicationGroup, channelconfig.CapabilitiesValue(conf.Capabilities), channelconfig.AdminsPolicyKey)
> }
> ```
>
> template的修改势必会造成配置发生变化，因此下一步就是计算两个ConfigGroup的更新了，我们以template作为原来的配置，newChannelGroup最为更新之后的配置，计算哪些配置发生了更新。虽然这里我们还容易看出来发生了那些变化，但是为了统一处理，还是进行了计算（如果设置了orderingSystemChannelGroup的话，template的方式就不是这么简单的克隆后修改了）。
>
> **计算方法**
>
> 主要是遍历原ConfigGroup的Policy，Value和ConfigGroup，将未发生变化的部分保存到sameSet，发送变化后的配置保存到writeSet中且版本号加1。最后，如果未发生变化，返回的ConfigUpdate的ReadSet和WriteSet均为nil，如果发生了变化，将sameSet分别赋值给ReadSet和WriteSet，保证未发生变化的部分在两个Set中都存在。这样我们就知道了发生变化前后的配置。
> 根据之前template的生成方式，我们可以查看最后的读写集合
>
> ![img](https://upload-images.jianshu.io/upload_images/4747194-8587a8d48006abcb.png?imageMogr2/auto-orient/strip|imageView2/2/w/1029/format/webp)
>
> ConfigUpdate
>
> 在上图中，ReadSet的Application的version为0,WriteSet中version为1,说明发生了变化，通过观察，我们看到ConfigGroup的Policies发生了变化，这与template的修改相符合。其实最终的ConfigUpdate就是要通知，Application的Policy发生了变化，如果设置了某些配置参数的话，Value也会发生变化。
>
> 计算出来ConfigUpdate，最后封装为Envelope后，写入文件中，整个流程就结束了。
>
> 

### 5.1 创建通道和生成通道配置文件

1.进入cli1容器

```bash
$ docker exec -it cli1 bash

#cli1容器内部操作，特别注意-c mychannel名称被系统默认通道名称占用，不建议用户使用

bash-5.1# peer channel create -o orderer0.example.com:7050 -c appchannel -f channel-artifacts/appchannel.tx --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/msp/tlscacerts/tlsca.example.com-cert.pem

#退出容器，回到宿主机
bash-5.1# exit
```



![image-20221119183825929](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221119183825929.png)

2.将通道文件复制到raft-test项目路径下，然后再复制到其它cli容器内，加入通道

```sh
$ pwd

$ docker cp cli1:/opt/gopath/src/github.com/hyperledger/fabric/peer/appchannel.block .

$ docker cp appchannel.block cli2:/opt/gopath/src/github.com/hyperledger/fabric/peer

$ docker cp appchannel.block cli3:/opt/gopath/src/github.com/hyperledger/fabric/peer

$ docker cp appchannel.block cli4:/opt/gopath/src/github.com/hyperledger/fabric/peer

```

![image-20221119185931190](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221119185931190.png)



> 在Fabric网络中创建通道时报错
>
> Error: got unexpected status: FORBIDDEN -- config update for existing channel did not pass initial checks: implicit policy evaluation failed - 0 sub-policies were satisfied, but this policy requires 1 of the 'Writers' sub-policies to be satisfied: permission denied
>
> ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210107203540579.png)
>
> 原因通过configtxgen命令生成创世区块时使用的通道名与创建通道时使用的通道名一样，如下所示，channelID 与-c通道名均为amops
>
> ```bash
> $ configtxgen -profile SampleMultiNodeEtcdRaft -channelID amops -outputBlock ./channel-artifacts/genesis.block
> 
> $ peer channel create -o agridepartorderer.amops.com:7050 -c amops -f ./channel-artifacts/channel.tx \
> --tls true \
> --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/amops.com/orderers/agridepartorderer.amops.com/msp/tlscacerts/tlsca.amops.com-cert.pem
> ```
>
>
> 将configtxgen指令中的channel修改成不一样的即可
>
> ```bash
> $ configtxgen -profile SampleMultiNodeEtcdRaft -channelID amopsdeploy -outputBlock ./channel-artifacts/genesis.block
> ```
>
> 
>
> 重启Fabric网络后解决如上问题
>
> 成功解决：Error: got unexpected status: FORBIDDEN -- config update for existing channel did not pass initial checks: implicit policy evaluation failed - 0 sub-policies were satisfied, but this policy requires 1 of the 'Writers' sub-policies to be satisfied: permission denied的问题。
> ————————————————
> 版权声明：本文为CSDN博主「余府」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
> 原文链接：https://blog.csdn.net/bean_business/article/details/112335991



### 5.2 加入通道与更新锚点

1.cli1容器操作

```sh
$ docker exec -it cli1 bash

bash-5.1# peer channel join -b appchannel.block

bash-5.1# peer channel update -o orderer0.example.com:7050 -c appchannel -f ./channel-artifacts/Org1MSPanchors.tx \
--tls true \
--cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer0.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

#退出容器，回到宿主机
bash-5.1# exit
```

![image-20221119192046789](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221119192046789.png)



2.cli2容器操作

```sh
$ docker exec -it cli2 bash

bash-5.1# peer channel join -b appchannel.block

bash-5.1# peer channel update -o orderer0.example.com:7050 -c appchannel -f ./channel-artifacts/Org2MSPanchors.tx \
--tls true \
--cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer0.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

bash-5.1# exit
```

![image-20221119192303927](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221119192303927.png)



3.cli3容器操作

```sh
$ docker exec -it cli3 bash

bash-5.1# peer channel join -b appchannel.block

bash-5.1# peer channel update -o orderer0.example.com:7050 -c appchannel -f ./channel-artifacts/Org3MSPanchors.tx \
--tls true \
--cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer0.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

bash-5.1# exit
```



![image-20221119192521834](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221119192521834.png)



***4.cli4容器操作***

```sh
$ docker exec -it cli4 bash

bash-5.1# peer channel join -b appchannel.block

bash-5.1# peer channel update -o orderer0.example.com:7050 -c appchannel -f ./channel-artifacts/Org4MSPanchors.tx \
--tls true \
--cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer0.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

bash-5.1# exit
```

![image-20221119192716507](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221119192716507.png)



## 6.链码调用

### 6.1 复制官方链码

> 注意：这里的sacc.go文件目录取决于自己本地中的文件

```sh
$ cd ~/go/src/github.com/hnuorg/raft-test/chaincode/go
$ sudo cp ~/go/src/github.com/hyperledger/fabric-samples/chaincode/sacc/sacc.go . 
```



### 6.2 安装go依赖并打包链码

```bash
$ docker exec -it cli1 bash

#cli1容器内部执行
bash-5.1# cd /opt/gopath/src/github.com/hyperledger/fabric-cluster/chaincode/go/
bash-5.1# ls
sacc.go

bash-5.1# go env -w GOPROXY=https://goproxy.cn,direct
bash-5.1# go env -w GO111MODULE=auto
bash-5.1# go mod init
bash-5.1# go mod tidy
bash-5.1# go mod vendor

bash-5.1# ls
go.mod   go.sum  sacc.go  vendor

bash-5.1# peer lifecycle chaincode package sacc.tar.gz \
--path github.com/hyperledger/fabric-cluster/chaincode/go/ \
--label sacc_1

bash-5.1# ls
go.mod go.sum  sacc.go  sacc.tar.gz  vendor

bash-5.1# echo $PWD
/opt/gopath/src/github.com/hyperledger/fabric-cluster/chaincode/go

bash-5.1# exit
exit
```

![image-20221119220227229](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221119220227229.png)

![image-20221119220302175](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221119220302175.png)



### 6.3 将打包好的链码复制到四个cli

```sh
$ docker exec -it cli1 bash
bash-5.1# cd /opt/gopath/src/github.com/hyperledger/fabric/peer/
bash-5.1# ls
channel-artifacts  crypto  appchannel.block
bash-5.1# cp /opt/gopath/src/github.com/hyperledger/fabric-cluster/chaincode/go/sacc.tar.gz /opt/gopath/src/github.com/hyperledger/fabric/peer/
bash-5.1# exit
exit

$ sudo docker cp cli1:/opt/gopath/src/github.com/hyperledger/fabric/peer/sacc.tar.gz .

$ ls
chaincode  channel-artifacts  configtx.yaml  crypto-config  crypto-config.yaml  docker-compose.yaml  mychannel.block  sacc.tar.gz

$ sudo docker cp sacc.tar.gz cli2:/opt/gopath/src/github.com/hyperledger/fabric/peer/
$ sudo docker cp sacc.tar.gz cli3:/opt/gopath/src/github.com/hyperledger/fabric/peer/
$ sudo docker cp sacc.tar.gz cli4:/opt/gopath/src/github.com/hyperledger/fabric/peer/

```

![image-20221119221227301](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221119221227301.png)



### 6.4 安装链码、批准链码、查看链码

> **在批准链码时，–package-id 的值需要与安装生成的package identifier:值一致**
>
> - cli1容器操作

```sh
$ docker exec -it cli1 bash

bash-5.1# peer lifecycle chaincode install sacc.tar.gz

2022-11-21 05:25:56.660 UTC 0001 INFO [cli.lifecycle.chaincode] submitInstallProposal -> Installed remotely: response:<status:200 payload:"\nGsacc_1:c365af10a4662d7ee6a8a04cd4303c6d84fdd05039843f742e5018d8a5e72a60\022\006sacc_1" > 
2022-11-21 05:25:56.660 UTC 0002 INFO [cli.lifecycle.chaincode] submitInstallProposal -> Chaincode code package identifier: sacc_1:c365af10a4662d7ee6a8a04cd4303c6d84fdd05039843f742e5018d8a5e72a60

bash-5.1# peer lifecycle chaincode queryinstalled
Installed chaincodes on peer:
Package ID: sacc_1:9ffa6d574bee9027d9ae02ad3c8a6fb3c7d0f65e86dcf887e6e33f7216571af8, Label: sacc_1

bash-5.1# peer lifecycle chaincode approveformyorg --channelID appchannel --name sacc --version 1.0 --init-required --package-id sacc_1:62942f6deb64d0af4aed4bb23f41528deeb7a87d9a4daa14042207d9bed8e816 --sequence 1 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer0.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

2022-11-21 05:28:15.395 UTC 0001 INFO [cli.lifecycle.chaincode] setOrdererClient -> Retrieved channel (appchannel) orderer endpoint: orderer0.example.com:7050
2022-11-21 05:28:17.537 UTC 0002 INFO [chaincodeCmd] ClientWait -> txid [d0e44affd75649d5f5218d57c08f6959d762d49e5c0e5aaf6cab858c5fff789c] committed with status (VALID) at peer0.org1.supervisor.com:7051

bash-5.1# peer lifecycle chaincode checkcommitreadiness --channelID appchannel --name sacc --version 1.0 --init-required --sequence 1 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --output json

bash-5.1# exit
exit
```

![image-20221121133311037](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221121133311037.png)





在cli2、cli3、cli4容器中的操作与上面类似，所以自行依次来进行执行

cli2

```bash
$ docker exec -it cli2 bash

bash-5.1# peer lifecycle chaincode install sacc.tar.gz

2022-11-21 05:35:20.649 UTC 0001 INFO [cli.lifecycle.chaincode] submitInstallProposal -> Installed remotely: response:<status:200 payload:"\nGsacc_1:c365af10a4662d7ee6a8a04cd4303c6d84fdd05039843f742e5018d8a5e72a60\022\006sacc_1" > 
2022-11-21 05:35:20.649 UTC 0002 INFO [cli.lifecycle.chaincode] submitInstallProposal -> Chaincode code package identifier: sacc_1:c365af10a4662d7ee6a8a04cd4303c6d84fdd05039843f742e5018d8a5e72a60

bash-5.1# peer lifecycle chaincode queryinstalled
Installed chaincodes on peer:
Package ID: sacc_1:9ffa6d574bee9027d9ae02ad3c8a6fb3c7d0f65e86dcf887e6e33f7216571af8, Label: sacc_1

bash-5.1# peer lifecycle chaincode approveformyorg --channelID appchannel --name sacc --version 1.0 --init-required --package-id sacc_1:62942f6deb64d0af4aed4bb23f41528deeb7a87d9a4daa14042207d9bed8e816 --sequence 1 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer0.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

2022-11-21 05:37:09.897 UTC 0001 INFO [cli.lifecycle.chaincode] setOrdererClient -> Retrieved channel (appchannel) orderer endpoint: orderer0.example.com:7050
2022-11-21 05:37:12.030 UTC 0002 INFO [chaincodeCmd] ClientWait -> txid [2cad28a3d24ea4fb3f2ebb7712f8513c310b768898bee9252be27ead8017f97a] committed with status (VALID) at peer0.org2.build.com:9051


bash-5.1# peer lifecycle chaincode checkcommitreadiness --channelID appchannel --name sacc --version 1.0 --init-required --sequence 1 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --output json


bash-5.1# exit
exit
```



![image-20221121133815073](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221121133815073.png)



cli3

```bash
$ docker exec -it cli3 bash

bash-5.1# peer lifecycle chaincode install sacc.tar.gz

2022-11-21 05:40:16.322 UTC 0001 INFO [cli.lifecycle.chaincode] submitInstallProposal -> Installed remotely: response:<status:200 payload:"\nGsacc_1:c365af10a4662d7ee6a8a04cd4303c6d84fdd05039843f742e5018d8a5e72a60\022\006sacc_1" > 
2022-11-21 05:40:16.322 UTC 0002 INFO [cli.lifecycle.chaincode] submitInstallProposal -> Chaincode code package identifier: sacc_1:c365af10a4662d7ee6a8a04cd4303c6d84fdd05039843f742e5018d8a5e72a60

bash-5.1# peer lifecycle chaincode queryinstalled
Installed chaincodes on peer:
Package ID: sacc_1:9ffa6d574bee9027d9ae02ad3c8a6fb3c7d0f65e86dcf887e6e33f7216571af8, Label: sacc_1

bash-5.1# peer lifecycle chaincode approveformyorg --channelID appchannel --name sacc --version 1.0 --init-required --package-id sacc_1:62942f6deb64d0af4aed4bb23f41528deeb7a87d9a4daa14042207d9bed8e816 --sequence 1 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer0.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

2022-11-21 05:40:59.501 UTC 0001 INFO [cli.lifecycle.chaincode] setOrdererClient -> Retrieved channel (appchannel) orderer endpoint: orderer0.example.com:7050
2022-11-21 05:41:01.648 UTC 0002 INFO [chaincodeCmd] ClientWait -> txid [c8f56485521bb98ccc58be57c78f179595a7b4b478932a6a50e735dc1638190b] committed with status (VALID) at peer0.org3.supplier.com:11051


bash-5.1# peer lifecycle chaincode checkcommitreadiness --channelID appchannel --name sacc --version 1.0 --init-required --sequence 1 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --output json


bash-5.1# exit
exit
```

![image-20221121134115737](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221121134115737.png)



cli4

```bash
$ docker exec -it cli4 bash

bash-5.1# peer lifecycle chaincode install sacc.tar.gz

2022-11-21 05:42:53.021 UTC 0001 INFO [cli.lifecycle.chaincode] submitInstallProposal -> Installed remotely: response:<status:200 payload:"\nGsacc_1:c365af10a4662d7ee6a8a04cd4303c6d84fdd05039843f742e5018d8a5e72a60\022\006sacc_1" > 
2022-11-21 05:42:53.022 UTC 0002 INFO [cli.lifecycle.chaincode] submitInstallProposal -> Chaincode code package identifier: sacc_1:c365af10a4662d7ee6a8a04cd4303c6d84fdd05039843f742e5018d8a5e72a60

bash-5.1# peer lifecycle chaincode queryinstalled
Installed chaincodes on peer:
Package ID: sacc_1:62942f6deb64d0af4aed4bb23f41528deeb7a87d9a4daa14042207d9bed8e816, Label: sacc_1

bash-5.1# peer lifecycle chaincode approveformyorg --channelID appchannel --name sacc --version 1.0 --init-required --package-id sacc_1:62942f6deb64d0af4aed4bb23f41528deeb7a87d9a4daa14042207d9bed8e816 --sequence 1 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer0.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

2022-11-21 05:44:19.247 UTC 0001 INFO [cli.lifecycle.chaincode] setOrdererClient -> Retrieved channel (appchannel) orderer endpoint: orderer0.example.com:7050
2022-11-21 05:44:21.394 UTC 0002 INFO [chaincodeCmd] ClientWait -> txid [2c2c9dfde7cbedbc0e938cd49470453fe4f8bb5fc98a7cc25d67b2f66c94abbd] committed with status (VALID) at peer0.org4.logistics.com:13051



bash-5.1# peer lifecycle chaincode checkcommitreadiness --channelID appchannel --name sacc --version 1.0 --init-required --sequence 1 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --output json

bash-5.1# exit
exit
```

![image-20221121134420033](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221121134420033.png)

### 6.5 提交链码

cli1容器下来进行提交

```sh
$ docker exec -it cli1 bash

bash-5.1# peer lifecycle chaincode commit -o orderer0.example.com:7050 \
--channelID appchannel \
--name sacc \
--version 1.0 \
--sequence 1 \
--init-required \
--tls true \
--cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer0.example.com/msp/tlscacerts/tlsca.example.com-cert.pem \
--peerAddresses peer0.org1.supervisor.com:7051 \
--tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.supervisor.com/peers/peer0.org1.supervisor.com/tls/ca.crt \
--peerAddresses peer0.org2.build.com:9051 \
--tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.build.com/peers/peer0.org2.build.com/tls/ca.crt \
--peerAddresses peer0.org3.supplier.com:11051 \
--tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.supplier.com/peers/peer0.org3.supplier.com/tls/ca.crt \
--peerAddresses peer0.org4.logistics.com:13051 \
--tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org4.logistics.com/peers/peer0.org4.logistics.com/tls/ca.crt

2022-11-21 05:46:21.295 UTC 0001 INFO [chaincodeCmd] ClientWait -> txid [30cff5a0ec784602af2b6157c322b6945b4b359b698e9acb6d8c2c567ed73499] committed with status (VALID) at peer0.org4.logistics.com:13051
2022-11-21 05:46:21.296 UTC 0002 INFO [chaincodeCmd] ClientWait -> txid [30cff5a0ec784602af2b6157c322b6945b4b359b698e9acb6d8c2c567ed73499] committed with status (VALID) at peer0.org3.supplier.com:11051
2022-11-21 05:46:21.296 UTC 0003 INFO [chaincodeCmd] ClientWait -> txid [30cff5a0ec784602af2b6157c322b6945b4b359b698e9acb6d8c2c567ed73499] committed with status (VALID) at peer0.org2.build.com:9051
2022-11-21 05:46:21.296 UTC 0004 INFO [chaincodeCmd] ClientWait -> txid [30cff5a0ec784602af2b6157c322b6945b4b359b698e9acb6d8c2c567ed73499] committed with status (VALID) at peer0.org1.supervisor.com:7051
```

![image-20221121134633889](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221121134633889.png)

### 6.6 链码初始化

> 接着上面的命令进行执行

```bash
bash-5.1# peer chaincode invoke -o orderer0.example.com:7050 \
--isInit \
--ordererTLSHostnameOverride orderer0.example.com \
--tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer0.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C appchannel -n sacc \
--peerAddresses peer0.org1.supervisor.com:7051 \
--tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.supervisor.com/peers/peer0.org1.supervisor.com/tls/ca.crt \
--peerAddresses peer0.org2.build.com:9051 \
--tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.build.com/peers/peer0.org2.build.com/tls/ca.crt \
--peerAddresses peer0.org3.supplier.com:11051 \
--tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.supplier.com/peers/peer0.org3.supplier.com/tls/ca.crt \
--peerAddresses peer0.org4.logistics.com:13051 \
--tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org4.logistics.com/peers/peer0.org4.logistics.com/tls/ca.crt -c '{"Args":["a","bb"]}' 

022-11-23 07:24:51.993 UTC 0001 INFO [chaincodeCmd] chaincodeInvokeOrQuery -> Chaincode invoke successful. result: status:200 
```

![image-20221123152508957](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221123152508957.png)

### 6.7 查询数据

```bash
bash-5.1# peer chaincode query -C appchannel -n sacc -c '{"Args":["query","a"]}'
bb
```

![image-20221123152613943](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221123152613943.png)

### 6.8 调用链码

```sh
bash-5.1# peer chaincode invoke -o orderer0.example.com:7050 \
--tls true \
--cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer0.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C appchannel -n sacc \
--peerAddresses peer0.org1.supervisor.com:7051 \
--tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.supervisor.com/peers/peer0.org1.supervisor.com/tls/ca.crt \
--peerAddresses peer0.org2.build.com:9051 \
--tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.build.com/peers/peer0.org2.build.com/tls/ca.crt \
--peerAddresses peer0.org3.supplier.com:11051 \
--tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.supplier.com/peers/peer0.org3.supplier.com/tls/ca.crt \
--peerAddresses peer0.org4.logistics.com:13051 \
--tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org4.logistics.com/peers/peer0.org4.logistics.com/tls/ca.crt -c '{"Args":["set","a","cc"]}'

2022-11-23 07:28:40.077 UTC 0001 INFO [chaincodeCmd] chaincodeInvokeOrQuery -> Chaincode invoke successful. result: status:200 payload:"cc" 
```

![image-20221123152840097](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221123152840097.png)

### 6.9 再次查询链码

```shell
bash-5.1# peer chaincode query -C appchannel -n sacc -c '{"Args":["query","a"]}'
cc
```

![image-20221123152942428](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20221123152942428.png)

### 6.10 关闭删除镜像

到这里一个完整的网络就搭建并测试成功了，所以就可以关闭并删除镜像了

```sh
$ docker stop $(docker ps -a -q)
$ docker rm $(docker ps -a -q)
```

## 总结

以上就是全部的安装测试过程，希望大家安装测试成功。
参考资料：

- https://blog.csdn.net/qq_45469783/article/details/119298677

- https://www.whcsrl.com/blog/1029128#4_1055



# Hyperledger Fabric 关于出块时间和区块大小

## 出块时间和区块大小

刚开始接触 fabric 的同学可能会有这样的疑问：为什么每个区块里就只有一笔交易呢？会产生这样的疑问很正常，说明有去认真仔细的观察和思考了。

![img](https://oscimg.oschina.net/oscnet/up-535c76eaf0cb925f85a76513f5f77f1e8cf.png)

## 区块大小的配置

决定区块大小的配置在通道配置文件 `configtx.yaml` 中：

```
Orderer: &OrdererDefaults


    OrdererType: solo

    Addresses:
        # - 127.0.0.1:7050

    # Batch Timeout: The amount of time to wait before creating a batch.
    BatchTimeout: 2s

    # Batch Size: Controls the number of messages batched into a block.
    # The orderer views messages opaquely, but typically, messages may
    # be considered to be Fabric transactions.  The 'batch' is the group
    # of messages in the 'data' field of the block.  Blocks will be a few kb
    # larger than the batch size, when signatures, hashes, and other metadata
    # is applied.
    BatchSize:

        # Max Message Count: The maximum number of messages to permit in a
        # batch.  No block will contain more than this number of messages.
        MaxMessageCount: 500

        # Absolute Max Bytes: The absolute maximum number of bytes allowed for
        # the serialized messages in a batch. The maximum block size is this value
        # plus the size of the associated metadata (usually a few KB depending
        # upon the size of the signing identities). Any transaction larger than
        # this value will be rejected by ordering.
        # It is recommended not to exceed 49 MB, given the default grpc max message size of 100 MB
        # configured on orderer and peer nodes (and allowing for message expansion during communication).
        AbsoluteMaxBytes: 10 MB

        # Preferred Max Bytes: The preferred maximum number of bytes allowed
        # for the serialized messages in a batch. Roughly, this field may be considered
        # the best effort maximum size of a batch. A batch will fill with messages
        # until this size is reached (or the max message count, or batch timeout is
        # exceeded).  If adding a new message to the batch would cause the batch to
        # exceed the preferred max bytes, then the current batch is closed and written
        # to a block, and a new batch containing the new message is created.  If a
        # message larger than the preferred max bytes is received, then its batch
        # will contain only that message.  Because messages may be larger than
        # preferred max bytes (up to AbsoluteMaxBytes), some batches may exceed
        # the preferred max bytes, but will always contain exactly one transaction.
        PreferredMaxBytes: 2 MB
```

`BatchTimeout` : 出块的时间间隔

`MaxMessageCount`: 一个块中最大交易数量

`AbsoluteMaxBytes`: 块的最大大小

`PreferredMaxBytes`: 块的优先选择的大小

这几个配置项决定了出块的时间和块的大小。

看到每个区块中只有一笔交易是因为在 2s 内网络中就只有一笔交易，所以在打包出块的时候这个块中就只有这一笔交易。

当在 2s 内增加了交易的次数后，产生的区块中就会包含多笔交易了。

![img](https://oscimg.oschina.net/oscnet/up-391bc2af649537c4f0484f3f098488f4a76.png)





# Hyperledger Fabric 编写第一个链码（Chaincode）

## 什么是链码？[¶](https://hyperledger-fabric.readthedocs.io/en/latest/chaincode4ade.html#what-is-chaincode)

链码是一个程序，用[Go](https://golang.org/)编写，[Node.js](https://nodejs.org/)， 或实现规定接口的 [Java](https://java.com/en/)。 链码在与对等节点不同的进程中运行，并初始化和管理 账本状态通过应用程序提交的事务。

链码通常处理由 网络，所以它类似于“智能合约”。可以调用链码来更新或查询 建议交易中的分类账。给定适当的权限，链码 可以调用另一个链码，无论是在同一通道还是在不同的通道中，以访问其状态。 请注意，如果被调用链码与调用链码位于不同的通道上， 只允许读取查询。也就是说，不同通道上被调用的链码只是一个， 在后续提交阶段不参与状态验证检查。`Query`

在以下部分中，我们将通过 应用程序开发人员。我们将提供一个资产转移链码示例演练， 以及结构协定 API 中每种方法的用途。如果你 是将链码部署到正在运行的网络的网络运营商， 访问将[智能合约部署到通道](https://hyperledger-fabric.readthedocs.io/en/latest/deploy_chaincode.html)教程和 [Fabric 链码生命周期](https://hyperledger-fabric.readthedocs.io/en/latest/chaincode_lifecycle.html)概念主题。

本教程概述了结构协定 API 提供的高级 API。 要了解有关使用 Fabric 合约 API 开发智能合约的更多信息，请访问 developapps/smartcontract 主题。

## 结构合约接口[¶](https://hyperledger-fabric.readthedocs.io/en/latest/chaincode4ade.html#fabric-contract-api)

它提供了合约接口，这是一个高级API，供应用程序开发人员实现智能合约。 在Hyperledger Fabric中，智能合约也被称为链码。使用此 API 为编写业务逻辑提供了高级入口点。 不同语言的结构合约 API 文档可在以下链接中找到：`fabric-contract-api`

> - [去](https://godoc.org/github.com/hyperledger/fabric-contract-api-go/contractapi)
> - [节点.js](https://hyperledger.github.io/fabric-chaincode-node/main/api/)
> - [爪哇岛](https://hyperledger.github.io/fabric-chaincode-java/main/api/org/hyperledger/fabric/contract/package-summary.html)

请注意，在使用合约 API 时，调用的每个链码函数都会传递一个交易上下文“ctx”，从该上下文中传递 你可以获取链码存根（GetStub（） ），它具有访问账本（例如 GetState（） ）和发出请求的功能 更新账本（例如 PutState（） ）。您可以通过下面的特定语言链接了解更多信息。

> - [去](https://godoc.org/github.com/hyperledger/fabric-chaincode-go/shim#Chaincode)
> - [节点.js](https://hyperledger.github.io/fabric-chaincode-node/main/api/fabric-shim.ChaincodeInterface.html)
> - [爪哇岛](https://hyperledger.github.io/fabric-chaincode-java/main/api/org/hyperledger/fabric/shim/Chaincode.html)

在本教程中使用 Go 链码，我们将演示这些 API 的使用 通过实现管理简单“资产”的资产转移链码应用程序。



## 资产转移链码[¶](https://hyperledger-fabric.readthedocs.io/en/latest/chaincode4ade.html#asset-transfer-chaincode)

我们的应用程序是一个基本的示例链码，用于初始化资产的账本，创建、读取、更新和删除资产，检查以查看 如果资产存在，并将资产从一个所有者转移到另一个所有者。



### 选择代码的位置[¶](https://hyperledger-fabric.readthedocs.io/en/latest/chaincode4ade.html#choosing-a-location-for-the-code)

如果你没有用 Go 编程，你可能要确保 您已经安装了[Go](https://golang.org/)并正确配置了您的系统。我们假设 您正在使用支持模块的版本。

现在，您需要为链码应用程序创建一个目录。

为了简单起见，让我们使用以下命令：

```bash
// atcc is shorthand for asset transfer chaincode
mkdir atcc && cd atcc
```

现在，让我们创建模块和源文件，我们将用代码填充：

```bash
go mod init atcc
touch atcc.go
```



### 家政[¶](https://hyperledger-fabric.readthedocs.io/en/latest/chaincode4ade.html#housekeeping)

首先，让我们从一些内务管理开始。与每个链码一样，它实现了结构-[合约-api接口](https://godoc.org/github.com/hyperledger/fabric-contract-api-go/contractapi)， 因此，让我们为链码的必要依赖项添加 Go import 语句。我们将导入 结构合约 API 包并定义我们的智能合约。

```go
package main

import (
  "fmt"
  "encoding/json"
  "log"
  "github.com/hyperledger/fabric-contract-api-go/contractapi"
)

// SmartContract provides functions for managing an Asset
type SmartContract struct {
  contractapi.Contract
}
```

接下来，让我们添加一个结构来表示账本上的简单资产。 请注意 JSON 注释，该注释将用于将资产封送到存储在账本上的 JSON。 JSON虽然不是一种确定性的数据格式 - 元素的顺序可以改变，同时仍然在语义上表示相同的数据。 因此，挑战在于能够生成一组一致的 JSON。 下面还显示了实现一致性的好方法，该方法包括按照字母顺序创建资产对象结构。`Asset`

```go
// Asset describes basic details of what makes up a simple asset
// Insert struct field in alphabetic order => to achieve determinism accross languages
// golang keeps the order when marshal to json but doesn't order automatically

type Asset struct {
  AppraisedValue int    `json:"AppraisedValue"`
  Color          string `json:"Color"`
  ID             string `json:"ID"`
  Owner          string `json:"Owner"`
  Size           int    `json:"Size"`
}
```

### 初始化链码[¶](https://hyperledger-fabric.readthedocs.io/en/latest/chaincode4ade.html#initializing-the-chaincode)

接下来，我们将实现该函数，用一些初始数据填充账本。`InitLedger`

```go
// InitLedger adds a base set of assets to the ledger
func (s *SmartContract) InitLedger(ctx contractapi.TransactionContextInterface) error {
  assets := []Asset{
    {ID: "asset1", Color: "blue", Size: 5, Owner: "Tomoko", AppraisedValue: 300},
    {ID: "asset2", Color: "red", Size: 5, Owner: "Brad", AppraisedValue: 400},
    {ID: "asset3", Color: "green", Size: 10, Owner: "Jin Soo", AppraisedValue: 500},
    {ID: "asset4", Color: "yellow", Size: 10, Owner: "Max", AppraisedValue: 600},
    {ID: "asset5", Color: "black", Size: 15, Owner: "Adriana", AppraisedValue: 700},
    {ID: "asset6", Color: "white", Size: 15, Owner: "Michel", AppraisedValue: 800},
  }

  for _, asset := range assets {
    assetJSON, err := json.Marshal(asset)
    if err != nil {
        return err
    }

    err = ctx.GetStub().PutState(asset.ID, assetJSON)
    if err != nil {
        return fmt.Errorf("failed to put to world state. %v", err)
    }
  }

  return nil
}
```

接下来，我们编写一个函数，在账本上创建一个尚不存在的资产。在编写链码时，它 在对账本采取行动之前检查账本上是否存在某物是个好主意，如图所示 在下面的函数中。`CreateAsset`

```go
// CreateAsset issues a new asset to the world state with given details.
func (s *SmartContract) CreateAsset(ctx contractapi.TransactionContextInterface, id string, color string, size int, owner string, appraisedValue int) error {
  exists, err := s.AssetExists(ctx, id)
  if err != nil {
    return err
  }
  if exists {
    return fmt.Errorf("the asset %s already exists", id)
  }

  asset := Asset{
    ID:             id,
    Color:          color,
    Size:           size,
    Owner:          owner,
    AppraisedValue: appraisedValue,
  }
  assetJSON, err := json.Marshal(asset)
  if err != nil {
    return err
  }

  return ctx.GetStub().PutState(id, assetJSON)
}
```

现在我们已经用一些初始资产填充了账本并创建了一个资产， 让我们编写一个函数，允许我们从账本中读取资产。`ReadAsset`

```go
// ReadAsset returns the asset stored in the world state with given id.
func (s *SmartContract) ReadAsset(ctx contractapi.TransactionContextInterface, id string) (*Asset, error) {
  assetJSON, err := ctx.GetStub().GetState(id)
  if err != nil {
    return nil, fmt.Errorf("failed to read from world state: %v", err)
  }
  if assetJSON == nil {
    return nil, fmt.Errorf("the asset %s does not exist", id)
  }

  var asset Asset
  err = json.Unmarshal(assetJSON, &asset)
  if err != nil {
    return nil, err
  }

  return &asset, nil
}
```

现在我们的账本上有可以与之交互的资产，让我们编写一个链码函数，允许我们更新允许更改的资产的属性。`UpdateAsset`

```go
// UpdateAsset updates an existing asset in the world state with provided parameters.
func (s *SmartContract) UpdateAsset(ctx contractapi.TransactionContextInterface, id string, color string, size int, owner string, appraisedValue int) error {
  exists, err := s.AssetExists(ctx, id)
  if err != nil {
    return err
  }
  if !exists {
    return fmt.Errorf("the asset %s does not exist", id)
  }

  // overwriting original asset with new asset
  asset := Asset{
    ID:             id,
    Color:          color,
    Size:           size,
    Owner:          owner,
    AppraisedValue: appraisedValue,
  }
  assetJSON, err := json.Marshal(asset)
  if err != nil {
    return err
  }

  return ctx.GetStub().PutState(id, assetJSON)
}
```

在某些情况下，我们需要能够从分类账中删除资产， 因此，让我们编写一个函数来处理该需求。`DeleteAsset`

```go
// DeleteAsset deletes an given asset from the world state.
func (s *SmartContract) DeleteAsset(ctx contractapi.TransactionContextInterface, id string) error {
  exists, err := s.AssetExists(ctx, id)
  if err != nil {
    return err
  }
  if !exists {
    return fmt.Errorf("the asset %s does not exist", id)
  }

  return ctx.GetStub().DelState(id)
}
```

我们之前说过，检查资产是否存在是一个好主意 对它采取行动，所以让我们编写一个调用的函数来实现该要求。`AssetExists`

```go
// AssetExists returns true when asset with given ID exists in world state
func (s *SmartContract) AssetExists(ctx contractapi.TransactionContextInterface, id string) (bool, error) {
  assetJSON, err := ctx.GetStub().GetState(id)
  if err != nil {
    return false, fmt.Errorf("failed to read from world state: %v", err)
  }

  return assetJSON != nil, nil
}
```

接下来，我们将编写一个函数，该函数支持将资产从一个所有者转移到另一个所有者。`TransferAsset`

```go
// TransferAsset updates the owner field of asset with given id in world state.
func (s *SmartContract) TransferAsset(ctx contractapi.TransactionContextInterface, id string, newOwner string) error {
  asset, err := s.ReadAsset(ctx, id)
  if err != nil {
    return err
  }

  asset.Owner = newOwner
  assetJSON, err := json.Marshal(asset)
  if err != nil {
    return err
  }

  return ctx.GetStub().PutState(id, assetJSON)
}
```

让我们编写一个函数，我们将调用该函数，该函数使账本的查询能够 返回账本上的所有资产。`GetAllAssets`

```go
// GetAllAssets returns all assets found in world state
func (s *SmartContract) GetAllAssets(ctx contractapi.TransactionContextInterface) ([]*Asset, error) {
  // range query with empty string for startKey and endKey does an
  // open-ended query of all assets in the chaincode namespace.
  resultsIterator, err := ctx.GetStub().GetStateByRange("", "")
  if err != nil {
    return nil, err
  }
  defer resultsIterator.Close()

  var assets []*Asset
  for resultsIterator.HasNext() {
    queryResponse, err := resultsIterator.Next()
    if err != nil {
      return nil, err
    }

    var asset Asset
    err = json.Unmarshal(queryResponse.Value, &asset)
    if err != nil {
      return nil, err
    }
    assets = append(assets, &asset)
  }

  return assets, nil
}
```

注意

下面提供了完整的链码示例，作为 使本教程尽可能清晰明了。在一个 实际实现，包很可能会被分段 其中包导入链码包以允许轻松进行单元测试。 要了解这是什么样子的，请参阅结构样本中的资产转移 [Go 链码](https://github.com/hyperledger/fabric-samples/tree/main/asset-transfer-basic/chaincode-go)。如果你看，你会发现 它包含和导入中定义的和 位于 。`main``assetTransfer.go``package main``package chaincode``smartcontract.go``fabric-samples/asset-transfer-basic/chaincode-go/chaincode/`

### 齐心协力[¶](https://hyperledger-fabric.readthedocs.io/en/latest/chaincode4ade.html#pulling-it-all-together)

最后，我们需要添加函数，它将调用 [ContractChaincode.Start](https://godoc.org/github.com/hyperledger/fabric-contract-api-go/contractapi#ContractChaincode.Start) 函数。这是整个链码程序源代码。`main`

```go
package main

import (
  "encoding/json"
  "fmt"
  "log"

  "github.com/hyperledger/fabric-contract-api-go/contractapi"
)

// SmartContract provides functions for managing an Asset
type SmartContract struct {
  contractapi.Contract
}

// Asset describes basic details of what makes up a simple asset
type Asset struct {
  ID             string `json:"ID"`
  Color          string `json:"color"`
  Size           int    `json:"size"`
  Owner          string `json:"owner"`
  AppraisedValue int    `json:"appraisedValue"`
}

// InitLedger adds a base set of assets to the ledger
func (s *SmartContract) InitLedger(ctx contractapi.TransactionContextInterface) error {
  assets := []Asset{
    {ID: "asset1", Color: "blue", Size: 5, Owner: "Tomoko", AppraisedValue: 300},
    {ID: "asset2", Color: "red", Size: 5, Owner: "Brad", AppraisedValue: 400},
    {ID: "asset3", Color: "green", Size: 10, Owner: "Jin Soo", AppraisedValue: 500},
    {ID: "asset4", Color: "yellow", Size: 10, Owner: "Max", AppraisedValue: 600},
    {ID: "asset5", Color: "black", Size: 15, Owner: "Adriana", AppraisedValue: 700},
    {ID: "asset6", Color: "white", Size: 15, Owner: "Michel", AppraisedValue: 800},
  }

  for _, asset := range assets {
    assetJSON, err := json.Marshal(asset)
    if err != nil {
      return err
    }

    err = ctx.GetStub().PutState(asset.ID, assetJSON)
    if err != nil {
      return fmt.Errorf("failed to put to world state. %v", err)
    }
  }

  return nil
}

// CreateAsset issues a new asset to the world state with given details.
func (s *SmartContract) CreateAsset(ctx contractapi.TransactionContextInterface, id string, color string, size int, owner string, appraisedValue int) error {
  exists, err := s.AssetExists(ctx, id)
  if err != nil {
    return err
  }
  if exists {
    return fmt.Errorf("the asset %s already exists", id)
  }

  asset := Asset{
    ID:             id,
    Color:          color,
    Size:           size,
    Owner:          owner,
    AppraisedValue: appraisedValue,
  }
  assetJSON, err := json.Marshal(asset)
  if err != nil {
    return err
  }

  return ctx.GetStub().PutState(id, assetJSON)
}

// ReadAsset returns the asset stored in the world state with given id.
func (s *SmartContract) ReadAsset(ctx contractapi.TransactionContextInterface, id string) (*Asset, error) {
  assetJSON, err := ctx.GetStub().GetState(id)
  if err != nil {
    return nil, fmt.Errorf("failed to read from world state: %v", err)
  }
  if assetJSON == nil {
    return nil, fmt.Errorf("the asset %s does not exist", id)
  }

  var asset Asset
  err = json.Unmarshal(assetJSON, &asset)
  if err != nil {
    return nil, err
  }

  return &asset, nil
}

// UpdateAsset updates an existing asset in the world state with provided parameters.
func (s *SmartContract) UpdateAsset(ctx contractapi.TransactionContextInterface, id string, color string, size int, owner string, appraisedValue int) error {
  exists, err := s.AssetExists(ctx, id)
  if err != nil {
    return err
  }
  if !exists {
    return fmt.Errorf("the asset %s does not exist", id)
  }

  // overwriting original asset with new asset
  asset := Asset{
    ID:             id,
    Color:          color,
    Size:           size,
    Owner:          owner,
    AppraisedValue: appraisedValue,
  }
  assetJSON, err := json.Marshal(asset)
  if err != nil {
    return err
  }

  return ctx.GetStub().PutState(id, assetJSON)
}

// DeleteAsset deletes an given asset from the world state.
func (s *SmartContract) DeleteAsset(ctx contractapi.TransactionContextInterface, id string) error {
  exists, err := s.AssetExists(ctx, id)
  if err != nil {
    return err
  }
  if !exists {
    return fmt.Errorf("the asset %s does not exist", id)
  }

  return ctx.GetStub().DelState(id)
}

// AssetExists returns true when asset with given ID exists in world state
func (s *SmartContract) AssetExists(ctx contractapi.TransactionContextInterface, id string) (bool, error) {
  assetJSON, err := ctx.GetStub().GetState(id)
  if err != nil {
    return false, fmt.Errorf("failed to read from world state: %v", err)
  }

  return assetJSON != nil, nil
}

// TransferAsset updates the owner field of asset with given id in world state.
func (s *SmartContract) TransferAsset(ctx contractapi.TransactionContextInterface, id string, newOwner string) error {
  asset, err := s.ReadAsset(ctx, id)
  if err != nil {
    return err
  }

  asset.Owner = newOwner
  assetJSON, err := json.Marshal(asset)
  if err != nil {
    return err
  }

  return ctx.GetStub().PutState(id, assetJSON)
}

// GetAllAssets returns all assets found in world state
func (s *SmartContract) GetAllAssets(ctx contractapi.TransactionContextInterface) ([]*Asset, error) {
  // range query with empty string for startKey and endKey does an
  // open-ended query of all assets in the chaincode namespace.
  resultsIterator, err := ctx.GetStub().GetStateByRange("", "")
  if err != nil {
    return nil, err
  }
  defer resultsIterator.Close()

  var assets []*Asset
  for resultsIterator.HasNext() {
    queryResponse, err := resultsIterator.Next()
    if err != nil {
      return nil, err
    }

    var asset Asset
    err = json.Unmarshal(queryResponse.Value, &asset)
    if err != nil {
      return nil, err
    }
    assets = append(assets, &asset)
  }

  return assets, nil
}

func main() {
  assetChaincode, err := contractapi.NewChaincode(&SmartContract{})
  if err != nil {
    log.Panicf("Error creating asset-transfer-basic chaincode: %v", err)
  }

  if err := assetChaincode.Start(); err != nil {
    log.Panicf("Error starting asset-transfer-basic chaincode: %v", err)
  }
}
```

## 链码访问控制[¶](https://hyperledger-fabric.readthedocs.io/en/latest/chaincode4ade.html#chaincode-access-control)

链码可以利用客户端（提交者）证书进行访问 使用 控制决策。此外 结构协定 API 提供提取客户端标识的扩展 API 来自可用于访问控制决策的提交者的证书， 无论是基于客户端身份本身，还是基于组织身份， 或在客户端标识属性上。`ctx.GetStub().GetCreator()`

例如，表示为键/值的资产可能包括 作为值一部分的客户端标识（例如作为 JSON 属性） 表示资产所有者），并且只有此客户才能获得授权 以在将来更新键/值。客户端标识 库扩展 API 可以在链码中用于检索此 提交者信息以做出此类访问控制决策。



## 管理用 Go 编写的链码的外部依赖关系[¶](https://hyperledger-fabric.readthedocs.io/en/latest/chaincode4ade.html#managing-external-dependencies-for-chaincode-written-in-go)

您的 Go 链码依赖于不是 Go 包（如链码垫片） 标准库的一部分。这些包的源必须包含在 您的链码包，当它安装到对等节点时。如果您已构建 您的链码作为一个模块，最简单的方法是“供应商”的 在打包链码之前依赖关系。`go mod vendor`

```go
go mod tidy
go mod vendor
```

这会将链码的外部依赖项放入本地目录中。`vendor`

一旦依赖项在您的链码目录中被提供，然后操作将包含与 依赖到链码包中。`peer chaincode package``peer chaincode install`

## JSON 确定性[¶](https://hyperledger-fabric.readthedocs.io/en/latest/chaincode4ade.html#json-determinism)

能够可预测地处理数据格式至关重要，并且能够搜索区块链中保存的数据。

### 技术问题[¶](https://hyperledger-fabric.readthedocs.io/en/latest/chaincode4ade.html#technical-problem)

存储在 Fabric 中的数据格式由用户自行决定。 最低级别的 API 接受一个字节数组并存储该数组 - 这代表什么不是 Fabric 关心的问题。 重要的是在模拟交易时，给定相同的输入链码给出相同的字节数组。 否则，背书可能并不完全匹配，交易将不会提交或无效。

JSON 通常用作在账本上存储数据的数据格式，如果使用 CouchDB 查询，则需要 JSON。

JSON虽然不是一种确定性的数据格式 - 元素的顺序可以改变， 同时仍然在语义上表示相同的数据。因此，挑战在于能够生成一组一致的 JSON。

### 一个解决方案[¶](https://hyperledger-fabric.readthedocs.io/en/latest/chaincode4ade.html#a-solution)

生成跨多种语言的一致集合。 每种语言都有不同的功能和库，可用于将对象转换为 JSON。 跨不同语言实现确定性的最佳方法是选择一种规范方式作为格式化 JSON 的通用准则。 为了获得跨语言的一致哈希，您可以按字母顺序格式化 JSON。`JSON`

### 戈朗[¶](https://hyperledger-fabric.readthedocs.io/en/latest/chaincode4ade.html#golang)

在 Golang 中，该包用于将结构对象序列化为 JSON。 更具体地说，使用函数，后者按排序键顺序封送映射，并按声明字段的顺序保留结构。 由于结构是按字段声明顺序封送的，因此在定义新结构时请遵循字母顺序。`encoding/json``Marshal`

### 节点.js[¶](https://hyperledger-fabric.readthedocs.io/en/latest/chaincode4ade.html#id3)

在Javascript中，当将对象序列化为JSON时，通常使用该函数。 但是，为了获得一致的结果，需要一个确定性的 JSON.stringify（） 版本;通过这种方式，可以从字符串化的结果中获得一致的哈希。 是一个很好的库，也可以结合使用以达到字母顺序。[此处](https://www.npmjs.com/package/json-stringify-deterministic)获取更深入的教程。`JSON.stringify()``json-stringify-deterministic``sort-keys-recursive`

### 爪哇岛[¶](https://hyperledger-fabric.readthedocs.io/en/latest/chaincode4ade.html#id4)

Java 提供了多个库来将对象序列化为 JSON 字符串。但是，并非所有它们都提供一致性和排序。 例如，该库不提供任何一致性，因此应避免用于此应用程序。另一方面 该库非常适合我们的目的，因为它按字母顺序生成一致的 JSON。`Gson``Genson`

您可以在[资产转移基本](https://github.com/hyperledger/fabric-samples/tree/main/asset-transfer-basic)链码上找到这种做法的一个很好的例子。

注意

这只是我们认为可以有效的众多方法之一。 序列化时，您可以使用各种方法来实现一致性;不过 考虑到Fabric中使用的编程语言的不同特征， 按字母顺序排列的方法代表了解决问题的简单而有效的方法。 总之，如果最适合您的需求，请随意采用不同的方法。 附言如果您使用其他方法，请不要忘记在评论中告诉我们。





# *Hyperledger Fabric Gateway*网关官方文档

[Hyperledger Fabric-gateway](https://hyperledger.github.io/fabric-gateway/)

[在 GitHub 上查看](https://github.com/hyperledger/fabric-gateway/)

*Fabric gateway*是 Hyperledger Fabric 区块链网络的核心组件，负责协调代表客户端应用程序提交交易和查询账本状态所需的操作。通过使用网关，客户端应用程序只需连接到结构网络中的单个终结点。

## Fabric网关概述

[fabric-samples](https://github.com/hyperledger/fabric-samples) 存储库中有 *Go*、*Node* 和 *Java* 的示例，如果您想尝试新的 *Fabric Gateway*，这是一个很好的起点！

- [结构样本/资产转移基础](https://github.com/hyperledger/fabric-samples/tree/main/asset-transfer-basic)，用于交易提交和评估的示例。
- [结构样本/资产转移](https://github.com/hyperledger/fabric-samples/tree/main/asset-transfer-events)事件，以获取链码事件的示例。
- [结构样本/off_chain_data](https://github.com/hyperledger/fabric-samples/tree/main/off_chain_data)块事件的示例。

如果从其中一个旧版 Fabric 客户端 SDK 迁移现有应用程序，请参阅[迁移指南](https://hyperledger.github.io/fabric-gateway/migration)。

## 客户端接口

结构网关客户端 API 可用于多种编程语言，以支持开发使用网关与结构网络交互的客户端应用程序。

### Go

- [快速入门指南](https://github.com/hyperledger/fabric-gateway/blob/main/pkg/client/README.md)
- [接口文档](https://pkg.go.dev/github.com/hyperledger/fabric-gateway/pkg/client)

### Node

- [快速入门指南](https://github.com/hyperledger/fabric-gateway/blob/main/node/README.md)
- [接口文档](https://hyperledger.github.io/fabric-gateway/main/api/node/)

### Java

- [快速入门指南](https://github.com/hyperledger/fabric-gateway/blob/main/java/README.md)
- [接口文档](https://hyperledger.github.io/fabric-gateway/main/api/java/)

## 兼容性

*Fabric*网关客户端 *API* 的每个次要版本都面向当前支持的 *Go* 版本，以及当前长期支持 （LTS） 版本的 Node 和 Java。网关对等方的特定最低版本的*Hyperledger Fabric*也需要完整的功能。

下表显示了 *Fabric* 版本、编程语言运行时以及经过显式测试并支持与 *Fabric* 网关客户端 API 一起使用的其他依赖项。

| ****       | **测试**         | **支持**         |
| ---------- | ---------------- | ---------------- |
| **Fabric** | **2.5**          | **2.4.4+**       |
| **go**     | 1.18, 1.19, 1.20 | 1.18, 1.19, 1.20 |
| **Node**   | 14, 16, 18       | 14, 16, 18       |
| **Java**   | 8, 11, 17        | 8, 11, 17        |
| **Linux**  | Ubuntu 22.04     |                  |



## 案例一：资产转移基本示例

**fabric-samples/asset-transfer-basic**

[fabric-samples/asset-transfer-basic at main · hyperledger/fabric-samples · GitHub](https://github.com/hyperledger/fabric-samples/tree/main/asset-transfer-basic)

资产转移基本示例演示：

- 将客户端应用程序连接到 *Fabric* 区块链网络。
- 提交智能合约交易以更新账本状态。
- 评估智能合约交易以查询账本状态。
- 处理事务调用中的错误。

### 关于示例

此示例包括多种语言的智能合约和应用程序代码。此示例演示资产的创建、读取、更新、传输和删除。

有关应用程序代码和客户端 API 用法的更详细演练，请参阅主 Hyperledger Fabric 文档中的[运行 Fabric 应用程序教程](https://hyperledger-fabric.readthedocs.io/en/latest/write_first_app.html)。

#### 应用

遵循客户端应用程序代码中的执行流，并在运行应用程序时遵循相应的输出。注意以下顺序：

- 事务调用（控制台输出，如“**-->提交事务**”和“**-->评估事务**”）。
- 事务返回的结果（控制台输出，如“***\** 结果**”）。

#### 智能合约

智能合约（在文件夹中）实现以下功能来支持应用程序：`chaincode-xyz`

- 创建资产
- 读取资产
- 更新资产
- 删除资源
- 转移资产

请注意，智能合约实现的资产转移是一个简化的场景，没有所有权验证，仅用于演示如何调用交易。

### 运行示例

结构测试网络用于部署和运行此示例。请按顺序执行以下步骤：

#### 1.创建测试网络和通道

```bash
$ ./network.sh up createChannel -c mychannel -ca
```

![image-20230527171303912](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20230527171303912.png)

详细信息：

```bash
magpie@DLTFabricer:~/go/src/github.com/hyperledger/fabric-samples/test-network$ ./network.sh up createChannel -c mychannel -ca
Using docker and docker-compose
Creating channel 'mychannel'.
If network is not up, starting nodes with CLI timeout of '5' tries and CLI delay of '3' seconds and using database 'leveldb with crypto from 'Certificate Authorities'
Bringing up network
LOCAL_VERSION=2.5.1
DOCKER_IMAGE_VERSION=2.5.1
CA_LOCAL_VERSION=v1.5.6
CA_DOCKER_IMAGE_VERSION=v1.5.6
Generating certificates using Fabric CA
[+] Running 4/4
 ✔ Network fabric_test   Created                                                                                                                                                  0.1s 
 ✔ Container ca_org2     Started                                                                                                                                                  2.2s 
 ✔ Container ca_org1     Started                                                                                                                                                  2.2s 
 ✔ Container ca_orderer  Started                                                                                                                                                  2.3s 
Creating Org1 Identities
Enrolling the CA admin
+ fabric-ca-client enroll -u https://admin:adminpw@localhost:7054 --caname ca-org1 --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/org1/ca-cert.pem
2023/05/27 08:52:58 [INFO] Created a default configuration file at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/fabric-ca-client-config.yaml
2023/05/27 08:52:58 [INFO] TLS Enabled
2023/05/27 08:52:58 [INFO] generating key: &{A:ecdsa S:256}
2023/05/27 08:52:59 [INFO] encoded CSR
2023/05/27 08:52:59 [INFO] Stored client certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/msp/signcerts/cert.pem
2023/05/27 08:52:59 [INFO] Stored root CA certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/msp/cacerts/localhost-7054-ca-org1.pem
2023/05/27 08:52:59 [INFO] Stored Issuer public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/msp/IssuerPublicKey
2023/05/27 08:52:59 [INFO] Stored Issuer revocation public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/msp/IssuerRevocationPublicKey
Registering peer0
+ fabric-ca-client register --caname ca-org1 --id.name peer0 --id.secret peer0pw --id.type peer --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/org1/ca-cert.pem
2023/05/27 08:52:59 [INFO] Configuration file location: /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/fabric-ca-client-config.yaml
2023/05/27 08:52:59 [INFO] TLS Enabled
2023/05/27 08:52:59 [INFO] TLS Enabled
Password: peer0pw
Registering user
+ fabric-ca-client register --caname ca-org1 --id.name user1 --id.secret user1pw --id.type client --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/org1/ca-cert.pem
2023/05/27 08:52:59 [INFO] Configuration file location: /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/fabric-ca-client-config.yaml
2023/05/27 08:52:59 [INFO] TLS Enabled
2023/05/27 08:52:59 [INFO] TLS Enabled
Password: user1pw
Registering the org admin
+ fabric-ca-client register --caname ca-org1 --id.name org1admin --id.secret org1adminpw --id.type admin --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/org1/ca-cert.pem
2023/05/27 08:52:59 [INFO] Configuration file location: /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/fabric-ca-client-config.yaml
2023/05/27 08:52:59 [INFO] TLS Enabled
2023/05/27 08:52:59 [INFO] TLS Enabled
Password: org1adminpw
Generating the peer0 msp
+ fabric-ca-client enroll -u https://peer0:peer0pw@localhost:7054 --caname ca-org1 -M /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/org1/ca-cert.pem
2023/05/27 08:53:00 [INFO] TLS Enabled
2023/05/27 08:53:00 [INFO] generating key: &{A:ecdsa S:256}
2023/05/27 08:53:00 [INFO] encoded CSR
2023/05/27 08:53:00 [INFO] Stored client certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp/signcerts/cert.pem
2023/05/27 08:53:00 [INFO] Stored root CA certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp/cacerts/localhost-7054-ca-org1.pem
2023/05/27 08:53:00 [INFO] Stored Issuer public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp/IssuerPublicKey
2023/05/27 08:53:00 [INFO] Stored Issuer revocation public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp/IssuerRevocationPublicKey
Generating the peer0-tls certificates, use --csr.hosts to specify Subject Alternative Names
+ fabric-ca-client enroll -u https://peer0:peer0pw@localhost:7054 --caname ca-org1 -M /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls --enrollment.profile tls --csr.hosts peer0.org1.example.com --csr.hosts localhost --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/org1/ca-cert.pem
2023/05/27 08:53:00 [INFO] TLS Enabled
2023/05/27 08:53:00 [INFO] generating key: &{A:ecdsa S:256}
2023/05/27 08:53:00 [INFO] encoded CSR
2023/05/27 08:53:00 [INFO] Stored client certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/signcerts/cert.pem
2023/05/27 08:53:00 [INFO] Stored TLS root CA certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/tlscacerts/tls-localhost-7054-ca-org1.pem
2023/05/27 08:53:00 [INFO] Stored Issuer public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/IssuerPublicKey
2023/05/27 08:53:00 [INFO] Stored Issuer revocation public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/IssuerRevocationPublicKey
Generating the user msp
+ fabric-ca-client enroll -u https://user1:user1pw@localhost:7054 --caname ca-org1 -M /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/org1/ca-cert.pem
2023/05/27 08:53:00 [INFO] TLS Enabled
2023/05/27 08:53:00 [INFO] generating key: &{A:ecdsa S:256}
2023/05/27 08:53:00 [INFO] encoded CSR
2023/05/27 08:53:00 [INFO] Stored client certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp/signcerts/cert.pem
2023/05/27 08:53:00 [INFO] Stored root CA certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp/cacerts/localhost-7054-ca-org1.pem
2023/05/27 08:53:00 [INFO] Stored Issuer public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp/IssuerPublicKey
2023/05/27 08:53:00 [INFO] Stored Issuer revocation public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp/IssuerRevocationPublicKey
Generating the org admin msp
+ fabric-ca-client enroll -u https://org1admin:org1adminpw@localhost:7054 --caname ca-org1 -M /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/org1/ca-cert.pem
2023/05/27 08:53:00 [INFO] TLS Enabled
2023/05/27 08:53:00 [INFO] generating key: &{A:ecdsa S:256}
2023/05/27 08:53:00 [INFO] encoded CSR
2023/05/27 08:53:00 [INFO] Stored client certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/signcerts/cert.pem
2023/05/27 08:53:00 [INFO] Stored root CA certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/cacerts/localhost-7054-ca-org1.pem
2023/05/27 08:53:00 [INFO] Stored Issuer public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/IssuerPublicKey
2023/05/27 08:53:00 [INFO] Stored Issuer revocation public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/IssuerRevocationPublicKey
Creating Org2 Identities
Enrolling the CA admin
+ fabric-ca-client enroll -u https://admin:adminpw@localhost:8054 --caname ca-org2 --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/org2/ca-cert.pem
2023/05/27 08:53:01 [INFO] Created a default configuration file at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/fabric-ca-client-config.yaml
2023/05/27 08:53:01 [INFO] TLS Enabled
2023/05/27 08:53:01 [INFO] generating key: &{A:ecdsa S:256}
2023/05/27 08:53:01 [INFO] encoded CSR
2023/05/27 08:53:01 [INFO] Stored client certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/msp/signcerts/cert.pem
2023/05/27 08:53:01 [INFO] Stored root CA certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/msp/cacerts/localhost-8054-ca-org2.pem
2023/05/27 08:53:01 [INFO] Stored Issuer public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/msp/IssuerPublicKey
2023/05/27 08:53:01 [INFO] Stored Issuer revocation public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/msp/IssuerRevocationPublicKey
Registering peer0
+ fabric-ca-client register --caname ca-org2 --id.name peer0 --id.secret peer0pw --id.type peer --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/org2/ca-cert.pem
2023/05/27 08:53:01 [INFO] Configuration file location: /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/fabric-ca-client-config.yaml
2023/05/27 08:53:01 [INFO] TLS Enabled
2023/05/27 08:53:01 [INFO] TLS Enabled
Password: peer0pw
Registering user
+ fabric-ca-client register --caname ca-org2 --id.name user1 --id.secret user1pw --id.type client --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/org2/ca-cert.pem
2023/05/27 08:53:01 [INFO] Configuration file location: /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/fabric-ca-client-config.yaml
2023/05/27 08:53:01 [INFO] TLS Enabled
2023/05/27 08:53:01 [INFO] TLS Enabled
Password: user1pw
Registering the org admin
+ fabric-ca-client register --caname ca-org2 --id.name org2admin --id.secret org2adminpw --id.type admin --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/org2/ca-cert.pem
2023/05/27 08:53:01 [INFO] Configuration file location: /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/fabric-ca-client-config.yaml
2023/05/27 08:53:01 [INFO] TLS Enabled
2023/05/27 08:53:01 [INFO] TLS Enabled
Password: org2adminpw
Generating the peer0 msp
+ fabric-ca-client enroll -u https://peer0:peer0pw@localhost:8054 --caname ca-org2 -M /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/msp --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/org2/ca-cert.pem
2023/05/27 08:53:01 [INFO] TLS Enabled
2023/05/27 08:53:01 [INFO] generating key: &{A:ecdsa S:256}
2023/05/27 08:53:01 [INFO] encoded CSR
2023/05/27 08:53:02 [INFO] Stored client certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/msp/signcerts/cert.pem
2023/05/27 08:53:02 [INFO] Stored root CA certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/msp/cacerts/localhost-8054-ca-org2.pem
2023/05/27 08:53:02 [INFO] Stored Issuer public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/msp/IssuerPublicKey
2023/05/27 08:53:02 [INFO] Stored Issuer revocation public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/msp/IssuerRevocationPublicKey
Generating the peer0-tls certificates, use --csr.hosts to specify Subject Alternative Names
+ fabric-ca-client enroll -u https://peer0:peer0pw@localhost:8054 --caname ca-org2 -M /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls --enrollment.profile tls --csr.hosts peer0.org2.example.com --csr.hosts localhost --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/org2/ca-cert.pem
2023/05/27 08:53:02 [INFO] TLS Enabled
2023/05/27 08:53:02 [INFO] generating key: &{A:ecdsa S:256}
2023/05/27 08:53:02 [INFO] encoded CSR
2023/05/27 08:53:02 [INFO] Stored client certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/signcerts/cert.pem
2023/05/27 08:53:02 [INFO] Stored TLS root CA certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/tlscacerts/tls-localhost-8054-ca-org2.pem
2023/05/27 08:53:02 [INFO] Stored Issuer public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/IssuerPublicKey
2023/05/27 08:53:02 [INFO] Stored Issuer revocation public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/IssuerRevocationPublicKey
Generating the user msp
+ fabric-ca-client enroll -u https://user1:user1pw@localhost:8054 --caname ca-org2 -M /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/users/User1@org2.example.com/msp --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/org2/ca-cert.pem
2023/05/27 08:53:02 [INFO] TLS Enabled
2023/05/27 08:53:02 [INFO] generating key: &{A:ecdsa S:256}
2023/05/27 08:53:02 [INFO] encoded CSR
2023/05/27 08:53:02 [INFO] Stored client certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/users/User1@org2.example.com/msp/signcerts/cert.pem
2023/05/27 08:53:02 [INFO] Stored root CA certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/users/User1@org2.example.com/msp/cacerts/localhost-8054-ca-org2.pem
2023/05/27 08:53:02 [INFO] Stored Issuer public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/users/User1@org2.example.com/msp/IssuerPublicKey
2023/05/27 08:53:02 [INFO] Stored Issuer revocation public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/users/User1@org2.example.com/msp/IssuerRevocationPublicKey
Generating the org admin msp
+ fabric-ca-client enroll -u https://org2admin:org2adminpw@localhost:8054 --caname ca-org2 -M /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/org2/ca-cert.pem
2023/05/27 08:53:02 [INFO] TLS Enabled
2023/05/27 08:53:02 [INFO] generating key: &{A:ecdsa S:256}
2023/05/27 08:53:02 [INFO] encoded CSR
2023/05/27 08:53:02 [INFO] Stored client certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp/signcerts/cert.pem
2023/05/27 08:53:02 [INFO] Stored root CA certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp/cacerts/localhost-8054-ca-org2.pem
2023/05/27 08:53:02 [INFO] Stored Issuer public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp/IssuerPublicKey
2023/05/27 08:53:02 [INFO] Stored Issuer revocation public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp/IssuerRevocationPublicKey
Creating Orderer Org Identities
Enrolling the CA admin
+ fabric-ca-client enroll -u https://admin:adminpw@localhost:9054 --caname ca-orderer --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/ordererOrg/ca-cert.pem
2023/05/27 08:53:02 [INFO] Created a default configuration file at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/fabric-ca-client-config.yaml
2023/05/27 08:53:02 [INFO] TLS Enabled
2023/05/27 08:53:02 [INFO] generating key: &{A:ecdsa S:256}
2023/05/27 08:53:02 [INFO] encoded CSR
2023/05/27 08:53:02 [INFO] Stored client certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/msp/signcerts/cert.pem
2023/05/27 08:53:02 [INFO] Stored root CA certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/msp/cacerts/localhost-9054-ca-orderer.pem
2023/05/27 08:53:02 [INFO] Stored Issuer public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/msp/IssuerPublicKey
2023/05/27 08:53:02 [INFO] Stored Issuer revocation public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/msp/IssuerRevocationPublicKey
Registering orderer
+ fabric-ca-client register --caname ca-orderer --id.name orderer --id.secret ordererpw --id.type orderer --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/ordererOrg/ca-cert.pem
2023/05/27 08:53:03 [INFO] Configuration file location: /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/fabric-ca-client-config.yaml
2023/05/27 08:53:03 [INFO] TLS Enabled
2023/05/27 08:53:03 [INFO] TLS Enabled
Password: ordererpw
Registering the orderer admin
+ fabric-ca-client register --caname ca-orderer --id.name ordererAdmin --id.secret ordererAdminpw --id.type admin --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/ordererOrg/ca-cert.pem
2023/05/27 08:53:03 [INFO] Configuration file location: /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/fabric-ca-client-config.yaml
2023/05/27 08:53:03 [INFO] TLS Enabled
2023/05/27 08:53:03 [INFO] TLS Enabled
Password: ordererAdminpw
Generating the orderer msp
+ fabric-ca-client enroll -u https://orderer:ordererpw@localhost:9054 --caname ca-orderer -M /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/ordererOrg/ca-cert.pem
2023/05/27 08:53:03 [INFO] TLS Enabled
2023/05/27 08:53:03 [INFO] generating key: &{A:ecdsa S:256}
2023/05/27 08:53:03 [INFO] encoded CSR
2023/05/27 08:53:03 [INFO] Stored client certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/signcerts/cert.pem
2023/05/27 08:53:03 [INFO] Stored root CA certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/cacerts/localhost-9054-ca-orderer.pem
2023/05/27 08:53:03 [INFO] Stored Issuer public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/IssuerPublicKey
2023/05/27 08:53:03 [INFO] Stored Issuer revocation public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/IssuerRevocationPublicKey
Generating the orderer-tls certificates, use --csr.hosts to specify Subject Alternative Names
+ fabric-ca-client enroll -u https://orderer:ordererpw@localhost:9054 --caname ca-orderer -M /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls --enrollment.profile tls --csr.hosts orderer.example.com --csr.hosts localhost --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/ordererOrg/ca-cert.pem
2023/05/27 08:53:03 [INFO] TLS Enabled
2023/05/27 08:53:03 [INFO] generating key: &{A:ecdsa S:256}
2023/05/27 08:53:03 [INFO] encoded CSR
2023/05/27 08:53:03 [INFO] Stored client certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/signcerts/cert.pem
2023/05/27 08:53:03 [INFO] Stored TLS root CA certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/tlscacerts/tls-localhost-9054-ca-orderer.pem
2023/05/27 08:53:03 [INFO] Stored Issuer public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/IssuerPublicKey
2023/05/27 08:53:03 [INFO] Stored Issuer revocation public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/IssuerRevocationPublicKey
Generating the admin msp
+ fabric-ca-client enroll -u https://ordererAdmin:ordererAdminpw@localhost:9054 --caname ca-orderer -M /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/users/Admin@example.com/msp --tls.certfiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/fabric-ca/ordererOrg/ca-cert.pem
2023/05/27 08:53:03 [INFO] TLS Enabled
2023/05/27 08:53:03 [INFO] generating key: &{A:ecdsa S:256}
2023/05/27 08:53:03 [INFO] encoded CSR
2023/05/27 08:53:04 [INFO] Stored client certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/users/Admin@example.com/msp/signcerts/cert.pem
2023/05/27 08:53:04 [INFO] Stored root CA certificate at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/users/Admin@example.com/msp/cacerts/localhost-9054-ca-orderer.pem
2023/05/27 08:53:04 [INFO] Stored Issuer public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/users/Admin@example.com/msp/IssuerPublicKey
2023/05/27 08:53:04 [INFO] Stored Issuer revocation public key at /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/users/Admin@example.com/msp/IssuerRevocationPublicKey
Generating CCP files for Org1 and Org2
WARN[0000] Found orphan containers ([ca_org2 ca_org1 ca_orderer]) for this project. If you removed or renamed this service in your compose file, you can run this command with the --remove-orphans flag to clean it up. 
[+] Running 7/7
 ✔ Volume "compose_orderer.example.com"     Created                                                                                                                               0.0s 
 ✔ Volume "compose_peer0.org1.example.com"  Created                                                                                                                               0.0s 
 ✔ Volume "compose_peer0.org2.example.com"  Created                                                                                                                               0.0s 
 ✔ Container peer0.org2.example.com         Started                                                                                                                               2.2s 
 ✔ Container orderer.example.com            Started                                                                                                                               3.5s 
 ✔ Container peer0.org1.example.com         Started                                                                                                                               2.1s 
 ✔ Container cli                            Started                                                                                                                               3.6s 
CONTAINER ID   IMAGE                               COMMAND                  CREATED          STATUS                    PORTS                                                                                                                             NAMES
4f91f5a4b0f7   hyperledger/fabric-tools:latest     "/bin/bash"              4 seconds ago    Up Less than a second                                                                                                                                       cli
3d96c01673f4   hyperledger/fabric-orderer:latest   "orderer"                4 seconds ago    Up Less than a second     0.0.0.0:7050->7050/tcp, :::7050->7050/tcp, 0.0.0.0:7053->7053/tcp, :::7053->7053/tcp, 0.0.0.0:9443->9443/tcp, :::9443->9443/tcp   orderer.example.com
b9b95ef4b50f   hyperledger/fabric-peer:latest      "peer node start"        4 seconds ago    Up 2 seconds              0.0.0.0:7051->7051/tcp, :::7051->7051/tcp, 0.0.0.0:9444->9444/tcp, :::9444->9444/tcp                                              peer0.org1.example.com
86b8dd0ad170   hyperledger/fabric-peer:latest      "peer node start"        4 seconds ago    Up 1 second               0.0.0.0:9051->9051/tcp, :::9051->9051/tcp, 7051/tcp, 0.0.0.0:9445->9445/tcp, :::9445->9445/tcp                                    peer0.org2.example.com
ed88dffb6341   hyperledger/fabric-ca:latest        "sh -c 'fabric-ca-se…"   14 seconds ago   Up 12 seconds             0.0.0.0:8054->8054/tcp, :::8054->8054/tcp, 7054/tcp, 0.0.0.0:18054->18054/tcp, :::18054->18054/tcp                                ca_org2
56e39157f2eb   hyperledger/fabric-ca:latest        "sh -c 'fabric-ca-se…"   14 seconds ago   Up 12 seconds             0.0.0.0:7054->7054/tcp, :::7054->7054/tcp, 0.0.0.0:17054->17054/tcp, :::17054->17054/tcp                                          ca_org1
941f4e4919ae   hyperledger/fabric-ca:latest        "sh -c 'fabric-ca-se…"   14 seconds ago   Up 11 seconds             0.0.0.0:9054->9054/tcp, :::9054->9054/tcp, 7054/tcp, 0.0.0.0:19054->19054/tcp, :::19054->19054/tcp                                ca_orderer
f1e8a0f68934   portainer/portainer                 "/portainer"             6 days ago       Exited (2) 17 hours ago                                                                                                                                     epic_solomon
Using docker and docker-compose
Generating channel genesis block 'mychannel.block'
/home/magpie/go/bin/configtxgen
+ configtxgen -profile TwoOrgsApplicationGenesis -outputBlock ./channel-artifacts/mychannel.block -channelID mychannel
2023-05-27 08:53:12.511 UTC 0001 INFO [common.tools.configtxgen] main -> Loading configuration
2023-05-27 08:53:12.553 UTC 0002 INFO [common.tools.configtxgen.localconfig] completeInitialization -> orderer type: etcdraft
2023-05-27 08:53:12.554 UTC 0003 INFO [common.tools.configtxgen.localconfig] completeInitialization -> Orderer.EtcdRaft.Options unset, setting to tick_interval:"500ms" election_tick:10 heartbeat_tick:1 max_inflight_blocks:5 snapshot_interval_size:16777216 
2023-05-27 08:53:12.554 UTC 0004 INFO [common.tools.configtxgen.localconfig] Load -> Loaded configuration: /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/configtx/configtx.yaml
2023-05-27 08:53:12.604 UTC 0005 INFO [common.tools.configtxgen] doOutputBlock -> Generating genesis block
2023-05-27 08:53:12.604 UTC 0006 INFO [common.tools.configtxgen] doOutputBlock -> Creating application channel genesis block
2023-05-27 08:53:12.605 UTC 0007 INFO [common.tools.configtxgen] doOutputBlock -> Writing genesis block
+ res=0
Creating channel mychannel
Using organization 1
+ osnadmin channel join --channelID mychannel --config-block ./channel-artifacts/mychannel.block -o localhost:7053 --ca-file /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/tlsca/tlsca.example.com-cert.pem --client-cert /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt --client-key /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.key
+ res=0
Status: 201
{
        "name": "mychannel",
        "url": "/participation/v1/channels/mychannel",
        "consensusRelation": "consenter",
        "status": "active",
        "height": 1
}

Channel 'mychannel' created
Joining org1 peer to the channel...
Using organization 1
+ peer channel join -b ./channel-artifacts/mychannel.block
+ res=0
2023-05-27 08:53:21.472 UTC 0001 INFO [channelCmd] InitCmdFactory -> Endorser and orderer connections initialized
2023-05-27 08:53:21.620 UTC 0002 INFO [channelCmd] executeJoin -> Successfully submitted proposal to join channel
Joining org2 peer to the channel...
Using organization 2
+ peer channel join -b ./channel-artifacts/mychannel.block
+ res=0
2023-05-27 08:53:24.693 UTC 0001 INFO [channelCmd] InitCmdFactory -> Endorser and orderer connections initialized
2023-05-27 08:53:24.725 UTC 0002 INFO [channelCmd] executeJoin -> Successfully submitted proposal to join channel
Setting anchor peer for org1...
Using organization 1
Fetching channel config for channel mychannel
Using organization 1
Fetching the most recent configuration block for the channel
+ peer channel fetch config config_block.pb -o orderer.example.com:7050 --ordererTLSHostnameOverride orderer.example.com -c mychannel --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/organizations/ordererOrganizations/example.com/tlsca/tlsca.example.com-cert.pem
2023-05-27 08:53:25.217 UTC 0001 INFO [channelCmd] InitCmdFactory -> Endorser and orderer connections initialized
2023-05-27 08:53:25.372 UTC 0002 INFO [cli.common] readBlock -> Received block: 0
2023-05-27 08:53:25.391 UTC 0003 INFO [channelCmd] fetch -> Retrieving last config block: 0
2023-05-27 08:53:25.393 UTC 0004 INFO [cli.common] readBlock -> Received block: 0
Decoding config block to JSON and isolating config to Org1MSPconfig.json
+ configtxlator proto_decode --input config_block.pb --type common.Block --output config_block.json
+ jq '.data.data[0].payload.data.config' config_block.json
+ jq '.channel_group.groups.Application.groups.Org1MSP.values += {"AnchorPeers":{"mod_policy": "Admins","value":{"anchor_peers": [{"host": "peer0.org1.example.com","port": 7051}]},"version": "0"}}' Org1MSPconfig.json
Generating anchor peer update transaction for Org1 on channel mychannel
+ configtxlator proto_encode --input Org1MSPconfig.json --type common.Config --output original_config.pb
+ configtxlator proto_encode --input Org1MSPmodified_config.json --type common.Config --output modified_config.pb
+ configtxlator compute_update --channel_id mychannel --original original_config.pb --updated modified_config.pb --output config_update.pb
+ configtxlator proto_decode --input config_update.pb --type common.ConfigUpdate --output config_update.json
+ jq .
++ cat config_update.json
+ echo '{"payload":{"header":{"channel_header":{"channel_id":"mychannel", "type":2}},"data":{"config_update":{' '"channel_id":' '"mychannel",' '"isolated_data":' '{},' '"read_set":' '{' '"groups":' '{' '"Application":' '{' '"groups":' '{' '"Org1MSP":' '{' '"groups":' '{},' '"mod_policy":' '"",' '"policies":' '{' '"Admins":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Endorsement":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Readers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Writers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '}' '},' '"values":' '{' '"MSP":' '{' '"mod_policy":' '"",' '"value":' null, '"version":' '"0"' '}' '},' '"version":' '"0"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '},' '"write_set":' '{' '"groups":' '{' '"Application":' '{' '"groups":' '{' '"Org1MSP":' '{' '"groups":' '{},' '"mod_policy":' '"Admins",' '"policies":' '{' '"Admins":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Endorsement":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Readers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Writers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '}' '},' '"values":' '{' '"AnchorPeers":' '{' '"mod_policy":' '"Admins",' '"value":' '{' '"anchor_peers":' '[' '{' '"host":' '"peer0.org1.example.com",' '"port":' 7051 '}' ']' '},' '"version":' '"0"' '},' '"MSP":' '{' '"mod_policy":' '"",' '"value":' null, '"version":' '"0"' '}' '},' '"version":' '"1"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '}' '}}}}'
+ configtxlator proto_encode --input config_update_in_envelope.json --type common.Envelope --output Org1MSPanchors.tx
2023-05-27 08:53:26.897 UTC 0001 INFO [channelCmd] InitCmdFactory -> Endorser and orderer connections initialized
2023-05-27 08:53:26.914 UTC 0002 INFO [channelCmd] update -> Successfully submitted channel update
Anchor peer set for org 'Org1MSP' on channel 'mychannel'
Setting anchor peer for org2...
Using organization 2
Fetching channel config for channel mychannel
+ peer channel fetch config config_block.pb -o orderer.example.com:7050 --ordererTLSHostnameOverride orderer.example.com -c mychannel --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/organizations/ordererOrganizations/example.com/tlsca/tlsca.example.com-cert.pem
Using organization 2
Fetching the most recent configuration block for the channel
2023-05-27 08:53:27.251 UTC 0001 INFO [channelCmd] InitCmdFactory -> Endorser and orderer connections initialized
2023-05-27 08:53:27.256 UTC 0002 INFO [cli.common] readBlock -> Received block: 1
2023-05-27 08:53:27.256 UTC 0003 INFO [channelCmd] fetch -> Retrieving last config block: 1
2023-05-27 08:53:27.258 UTC 0004 INFO [cli.common] readBlock -> Received block: 1
Decoding config block to JSON and isolating config to Org2MSPconfig.json
+ configtxlator proto_decode --input config_block.pb --type common.Block --output config_block.json
+ jq '.data.data[0].payload.data.config' config_block.json
+ jq '.channel_group.groups.Application.groups.Org2MSP.values += {"AnchorPeers":{"mod_policy": "Admins","value":{"anchor_peers": [{"host": "peer0.org2.example.com","port": 9051}]},"version": "0"}}' Org2MSPconfig.json
Generating anchor peer update transaction for Org2 on channel mychannel
+ configtxlator proto_encode --input Org2MSPconfig.json --type common.Config --output original_config.pb
+ configtxlator proto_encode --input Org2MSPmodified_config.json --type common.Config --output modified_config.pb
+ configtxlator compute_update --channel_id mychannel --original original_config.pb --updated modified_config.pb --output config_update.pb
+ configtxlator proto_decode --input config_update.pb --type common.ConfigUpdate --output config_update.json
+ jq .
++ cat config_update.json
+ echo '{"payload":{"header":{"channel_header":{"channel_id":"mychannel", "type":2}},"data":{"config_update":{' '"channel_id":' '"mychannel",' '"isolated_data":' '{},' '"read_set":' '{' '"groups":' '{' '"Application":' '{' '"groups":' '{' '"Org2MSP":' '{' '"groups":' '{},' '"mod_policy":' '"",' '"policies":' '{' '"Admins":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Endorsement":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Readers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Writers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '}' '},' '"values":' '{' '"MSP":' '{' '"mod_policy":' '"",' '"value":' null, '"version":' '"0"' '}' '},' '"version":' '"0"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '},' '"write_set":' '{' '"groups":' '{' '"Application":' '{' '"groups":' '{' '"Org2MSP":' '{' '"groups":' '{},' '"mod_policy":' '"Admins",' '"policies":' '{' '"Admins":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Endorsement":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Readers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Writers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '}' '},' '"values":' '{' '"AnchorPeers":' '{' '"mod_policy":' '"Admins",' '"value":' '{' '"anchor_peers":' '[' '{' '"host":' '"peer0.org2.example.com",' '"port":' 9051 '}' ']' '},' '"version":' '"0"' '},' '"MSP":' '{' '"mod_policy":' '"",' '"value":' null, '"version":' '"0"' '}' '},' '"version":' '"1"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '}' '}}}}'
+ configtxlator proto_encode --input config_update_in_envelope.json --type common.Envelope --output Org2MSPanchors.tx
2023-05-27 08:53:27.709 UTC 0001 INFO [channelCmd] InitCmdFactory -> Endorser and orderer connections initialized
2023-05-27 08:53:27.728 UTC 0002 INFO [channelCmd] update -> Successfully submitted channel update
Anchor peer set for org 'Org2MSP' on channel 'mychannel'
Channel 'mychannel' joined
magpie@DLTFabricer:~/go/src/github.com/hyperledger/fabric-samples/test-network$ 
```



#### 2.部署其中一个链码

```bash
# To deploy the TypeScript chaincode implementation
./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-typescript/ -ccl typescript

# To deploy the Go chaincode implementation
./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go/ -ccl go

# To deploy the Java chaincode implementation
./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-java/ -ccl java
```

部署链码：

```bash
./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go/ -ccl go
```

部署详细信息：

```bash
magpie@DLTFabricer:~/go/src/github.com/hyperledger/fabric-samples/test-network$ ./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go/ -ccl go
Using docker and docker-compose
deploying chaincode on channel 'mychannel'
executing with the following
- CHANNEL_NAME: mychannel
- CC_NAME: basic
- CC_SRC_PATH: ../asset-transfer-basic/chaincode-go/
- CC_SRC_LANGUAGE: go
- CC_VERSION: 1.0
- CC_SEQUENCE: 1
- CC_END_POLICY: NA
- CC_COLL_CONFIG: NA
- CC_INIT_FCN: NA
- DELAY: 3
- MAX_RETRY: 5
- VERBOSE: false
Vendoring Go dependencies at ../asset-transfer-basic/chaincode-go/
~/go/src/github.com/hyperledger/fabric-samples/asset-transfer-basic/chaincode-go ~/go/src/github.com/hyperledger/fabric-samples/test-network
~/go/src/github.com/hyperledger/fabric-samples/test-network
Finished vendoring Go dependencies
+ peer lifecycle chaincode package basic.tar.gz --path ../asset-transfer-basic/chaincode-go/ --lang golang --label basic_1.0
+ res=0
++ peer lifecycle chaincode calculatepackageid basic.tar.gz
+ PACKAGE_ID=basic_1.0:28a1bfd3d9158bf905bd6a043d6ed9da0fb0e6bf65bb52cef930b6a54e48d5d7
Chaincode is packaged
Installing chaincode on peer0.org1...
Using organization 1
+ peer lifecycle chaincode queryinstalled --output json
+ jq -r 'try (.installed_chaincodes[].package_id)'
+ grep '^basic_1.0:28a1bfd3d9158bf905bd6a043d6ed9da0fb0e6bf65bb52cef930b6a54e48d5d7$'
+ test 1 -ne 0
+ peer lifecycle chaincode install basic.tar.gz
+ res=0
2023-06-02 00:48:50.109 UTC 0001 INFO [cli.lifecycle.chaincode] submitInstallProposal -> Installed remotely: response:<status:200 payload:"\nJbasic_1.0:28a1bfd3d9158bf905bd6a043d6ed9da0fb0e6bf65bb52cef930b6a54e48d5d7\022\tbasic_1.0" > 
2023-06-02 00:48:50.232 UTC 0002 INFO [cli.lifecycle.chaincode] submitInstallProposal -> Chaincode code package identifier: basic_1.0:28a1bfd3d9158bf905bd6a043d6ed9da0fb0e6bf65bb52cef930b6a54e48d5d7
Chaincode is installed on peer0.org1
Install chaincode on peer0.org2...
Using organization 2
+ peer lifecycle chaincode queryinstalled --output json
+ jq -r 'try (.installed_chaincodes[].package_id)'
+ grep '^basic_1.0:28a1bfd3d9158bf905bd6a043d6ed9da0fb0e6bf65bb52cef930b6a54e48d5d7$'
+ test 1 -ne 0
+ peer lifecycle chaincode install basic.tar.gz
+ res=0
2023-06-02 00:49:41.984 UTC 0001 INFO [cli.lifecycle.chaincode] submitInstallProposal -> Installed remotely: response:<status:200 payload:"\nJbasic_1.0:28a1bfd3d9158bf905bd6a043d6ed9da0fb0e6bf65bb52cef930b6a54e48d5d7\022\tbasic_1.0" > 
2023-06-02 00:49:41.985 UTC 0002 INFO [cli.lifecycle.chaincode] submitInstallProposal -> Chaincode code package identifier: basic_1.0:28a1bfd3d9158bf905bd6a043d6ed9da0fb0e6bf65bb52cef930b6a54e48d5d7
Chaincode is installed on peer0.org2
Using organization 1
+ peer lifecycle chaincode queryinstalled --output json
+ jq -r 'try (.installed_chaincodes[].package_id)'
+ grep '^basic_1.0:28a1bfd3d9158bf905bd6a043d6ed9da0fb0e6bf65bb52cef930b6a54e48d5d7$'
+ res=0
basic_1.0:28a1bfd3d9158bf905bd6a043d6ed9da0fb0e6bf65bb52cef930b6a54e48d5d7
Query installed successful on peer0.org1 on channel
Using organization 1
+ peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/tlsca/tlsca.example.com-cert.pem --channelID mychannel --name basic --version 1.0 --package-id basic_1.0:28a1bfd3d9158bf905bd6a043d6ed9da0fb0e6bf65bb52cef930b6a54e48d5d7 --sequence 1
+ res=0
2023-06-02 00:49:44.282 UTC 0001 INFO [chaincodeCmd] ClientWait -> txid [e2b2fe2c0512cb3bf36f219a2199d46e1aef556440f58bbe81339213b1ae3961] committed with status (VALID) at localhost:7051
Chaincode definition approved on peer0.org1 on channel 'mychannel'
Using organization 1
Checking the commit readiness of the chaincode definition on peer0.org1 on channel 'mychannel'...
Attempting to check the commit readiness of the chaincode definition on peer0.org1, Retry after 3 seconds.
+ peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name basic --version 1.0 --sequence 1 --output json
+ res=0
{
        "approvals": {
                "Org1MSP": true,
                "Org2MSP": false
        }
}
Checking the commit readiness of the chaincode definition successful on peer0.org1 on channel 'mychannel'
Using organization 2
Checking the commit readiness of the chaincode definition on peer0.org2 on channel 'mychannel'...
Attempting to check the commit readiness of the chaincode definition on peer0.org2, Retry after 3 seconds.
+ peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name basic --version 1.0 --sequence 1 --output json
+ res=0
{
        "approvals": {
                "Org1MSP": true,
                "Org2MSP": false
        }
}
Checking the commit readiness of the chaincode definition successful on peer0.org2 on channel 'mychannel'
Using organization 2
+ peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/tlsca/tlsca.example.com-cert.pem --channelID mychannel --name basic --version 1.0 --package-id basic_1.0:28a1bfd3d9158bf905bd6a043d6ed9da0fb0e6bf65bb52cef930b6a54e48d5d7 --sequence 1
+ res=0
2023-06-02 00:49:52.592 UTC 0001 INFO [chaincodeCmd] ClientWait -> txid [1ca2aca4b3ddaf9062ffc233d104348b26b36b703b781a116e0dcf330841024d] committed with status (VALID) at localhost:9051
Chaincode definition approved on peer0.org2 on channel 'mychannel'
Using organization 1
Checking the commit readiness of the chaincode definition on peer0.org1 on channel 'mychannel'...
Attempting to check the commit readiness of the chaincode definition on peer0.org1, Retry after 3 seconds.
+ peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name basic --version 1.0 --sequence 1 --output json
+ res=0
{
        "approvals": {
                "Org1MSP": true,
                "Org2MSP": true
        }
}
Checking the commit readiness of the chaincode definition successful on peer0.org1 on channel 'mychannel'
Using organization 2
Checking the commit readiness of the chaincode definition on peer0.org2 on channel 'mychannel'...
Attempting to check the commit readiness of the chaincode definition on peer0.org2, Retry after 3 seconds.
+ peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name basic --version 1.0 --sequence 1 --output json
+ res=0
{
        "approvals": {
                "Org1MSP": true,
                "Org2MSP": true
        }
}
Checking the commit readiness of the chaincode definition successful on peer0.org2 on channel 'mychannel'
Using organization 1
Using organization 2
+ peer lifecycle chaincode commit -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/tlsca/tlsca.example.com-cert.pem --channelID mychannel --name basic --peerAddresses localhost:7051 --tlsRootCertFiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/tlsca/tlsca.org1.example.com-cert.pem --peerAddresses localhost:9051 --tlsRootCertFiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/tlsca/tlsca.org2.example.com-cert.pem --version 1.0 --sequence 1
+ res=0
2023-06-02 00:50:01.229 UTC 0001 INFO [chaincodeCmd] ClientWait -> txid [f68dc92e7b99f6348e2e1e1a73a0954e3f741c4772034f9c848b784f51c7c1ed] committed with status (VALID) at localhost:7051
2023-06-02 00:50:01.264 UTC 0002 INFO [chaincodeCmd] ClientWait -> txid [f68dc92e7b99f6348e2e1e1a73a0954e3f741c4772034f9c848b784f51c7c1ed] committed with status (VALID) at localhost:9051
Chaincode definition committed on channel 'mychannel'
Using organization 1
Querying chaincode definition on peer0.org1 on channel 'mychannel'...
Attempting to Query committed status on peer0.org1, Retry after 3 seconds.
+ peer lifecycle chaincode querycommitted --channelID mychannel --name basic
+ res=0
Committed chaincode definition for chaincode 'basic' on channel 'mychannel':
Version: 1.0, Sequence: 1, Endorsement Plugin: escc, Validation Plugin: vscc, Approvals: [Org1MSP: true, Org2MSP: true]
Query chaincode definition successful on peer0.org1 on channel 'mychannel'
Using organization 2
Querying chaincode definition on peer0.org2 on channel 'mychannel'...
Attempting to Query committed status on peer0.org2, Retry after 3 seconds.
+ peer lifecycle chaincode querycommitted --channelID mychannel --name basic
+ res=0
Committed chaincode definition for chaincode 'basic' on channel 'mychannel':
Version: 1.0, Sequence: 1, Endorsement Plugin: escc, Validation Plugin: vscc, Approvals: [Org1MSP: true, Org2MSP: true]
Query chaincode definition successful on peer0.org2 on channel 'mychannel'
Chaincode initialization is not required
magpie@DLTFabricer:~/go/src/github.com/hyperledger/fabric-samples/test-network$ 
```

部分截图：

![image-20230602090031203](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20230602090031203.png)

#### 3.运行程序

```bash
# To run the Typescript sample application
cd application-gateway-typescript
npm install
npm start

# To run the Go sample application
cd ~/go/src/github.com/hyperledger/fabric-samples/asset-transfer-basic
cd application-gateway-go
go run .

# To run the Java sample application
cd application-gateway-java
./gradlew run
```

运行详细信息：

```bash
cd ~/go/src/github.com/hyperledger/fabric-samples/asset-transfer-basic/application-gateway-go$

magpie@DLTFabricer:~/go/src/github.com/hyperledger/fabric-samples/asset-transfer-basic/application-gateway-go$ go run .
go: downloading github.com/hyperledger/fabric-gateway v1.2.2
go: downloading google.golang.org/genproto v0.0.0-20230216225411-c8e22ba71e44

--> Submit Transaction: InitLedger, function creates the initial set of assets on the ledger 
*** Transaction committed successfully

--> Evaluate Transaction: GetAllAssets, function returns all the current assets on the ledger
*** Result:[
  {
    "AppraisedValue": 300,
    "Color": "blue",
    "ID": "asset1",
    "Owner": "Tomoko",
    "Size": 5
  },
  {
    "AppraisedValue": 400,
    "Color": "red",
    "ID": "asset2",
    "Owner": "Brad",
    "Size": 5
  },
  {
    "AppraisedValue": 500,
    "Color": "green",
    "ID": "asset3",
    "Owner": "Jin Soo",
    "Size": 10
  },
  {
    "AppraisedValue": 600,
    "Color": "yellow",
    "ID": "asset4",
    "Owner": "Max",
    "Size": 10
  },
  {
    "AppraisedValue": 700,
    "Color": "black",
    "ID": "asset5",
    "Owner": "Adriana",
    "Size": 15
  },
  {
    "AppraisedValue": 800,
    "Color": "white",
    "ID": "asset6",
    "Owner": "Michel",
    "Size": 15
  }
]

--> Submit Transaction: CreateAsset, creates new asset with ID, Color, Size, Owner and AppraisedValue arguments 
*** Transaction committed successfully

--> Evaluate Transaction: ReadAsset, function returns asset attributes
*** Result:{
  "AppraisedValue": 1300,
  "Color": "yellow",
  "ID": "asset1685667849677",
  "Owner": "Tom",
  "Size": 5
}

--> Async Submit Transaction: TransferAsset, updates existing asset owner
*** Successfully submitted transaction to transfer ownership from Tom to Mark. 
*** Waiting for transaction commit.
*** Transaction committed successfully

--> Submit Transaction: UpdateAsset asset70, asset70 does not exist and should return an error
*** Successfully caught the error:
Endorse error for transaction 8b30516b9691d27b6c3f4f9e32d9f78dbf582ced7d63354bbaf9c0d58945bd3c with gRPC status Aborted: rpc error: code = Aborted desc = failed to endorse transaction, see attached details for more info
Error Details:
- address: peer0.org1.example.com:7051, mspId: Org1MSP, message: chaincode response 500, the asset asset70 does not exist
magpie@DLTFabricer:~/go/src/github.com/hyperledger/fabric-samples/asset-transfer-basic/application-gateway-go$ 
```

运行截图：

![image-20230602090838419](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20230602090838419.png)

### 收拾

完成后，您可以关闭测试网络（从文件夹中）。该命令将删除测试网络的所有节点，并删除您创建的任何账本数据。`test-network`

```bash
./network.sh down
```



## 案例二：资产转移事件示例

资产转移事件示例演示：

- 从智能合约交易函数发出链码事件。
- 在客户端应用程序中接收链码事件。
- 在客户端应用程序中重放以前的链码事件。

当一个区块提交到账本时，会发布事件。

有关每个通道的事件服务的更多信息，请访问 Fabric 文档中的[基于通道的事件服务](https://hyperledger-fabric.readthedocs.io/en/latest/peer_event_services.html)页面。

### 关于示例

此示例包括多种语言的智能合约和应用程序代码。在类似于基本资产传输的用例中（请参阅资产[传输基本](https://github.com/hyperledger/fabric-samples/blob/main/asset-transfer-basic)文件夹），此示例显示了在创建/更新/删除资产期间以及在将资产转移到新所有者期间发送和接收事件。

#### 应用

遵循客户端应用程序代码中的执行流，并在运行应用程序时遵循相应的输出。注意以下顺序：

- 事务调用（控制台输出，如“**-->提交事务**”）。
- 应用程序接收的事件（控制台输出，如“**<-收到链码事件**”）。

请注意，在应用程序代码提交事务并将其提交到账本之后，但在与事件无关的其他应用程序活动期间，侦听器将接收事件。

#### 智能合约

智能合约（在文件夹中）实现以下功能来支持应用程序：`chaincode-xyz`

- 创建资产
- 读取资产
- 更新资产
- 删除资源
- 转移资产

请注意，智能合约实现的资产转移是一个简化的场景，没有所有权验证，仅用于演示发送和接收事件的使用。

### 运行示例

与其他示例一样，结构测试网络用于部署和运行此示例。请按顺序执行以下步骤：

1.创建测试网络和通道（从文件夹中）

`test-network`

```bash
./network.sh up createChannel -c mychannel -ca
```

详情建示例一。

2.部署其中一个智能合约实现（从文件夹中）

test-network`

```bash
# To deploy the JavaScript chaincode implementation
./network.sh deployCC -ccn events -ccp ../asset-transfer-events/chaincode-javascript/ -ccl javascript -ccep "OR('Org1MSP.peer','Org2MSP.peer')"

# To deploy the Java chaincode implementation
./network.sh deployCC -ccn events -ccp ../asset-transfer-events/chaincode-java/ -ccl java -ccep "OR('Org1MSP.peer','Org2MSP.peer')"
```

详细信息：

```bash
magpie@DLTFabricer:~/go/src/github.com/hyperledger/fabric-samples/test-network$ ./network.sh deployCC -ccn events -ccp ../asset-transfer-events/chaincode-javascript/ -ccl javascript -ccep "OR('Org1MSP.peer','Org2MSP.peer')"
Using docker and docker-compose
deploying chaincode on channel 'mychannel'
executing with the following
- CHANNEL_NAME: mychannel
- CC_NAME: events
- CC_SRC_PATH: ../asset-transfer-events/chaincode-javascript/
- CC_SRC_LANGUAGE: javascript
- CC_VERSION: 1.0
- CC_SEQUENCE: 1
- CC_END_POLICY: OR('Org1MSP.peer','Org2MSP.peer')
- CC_COLL_CONFIG: NA
- CC_INIT_FCN: NA
- DELAY: 3
- MAX_RETRY: 5
- VERBOSE: false
+ peer lifecycle chaincode package events.tar.gz --path ../asset-transfer-events/chaincode-javascript/ --lang node --label events_1.0
+ res=0
++ peer lifecycle chaincode calculatepackageid events.tar.gz
+ PACKAGE_ID=events_1.0:91d63daaf0fc6186b344f3ac1408b326cbfc54df3b1ed30c3ef32758b06af482
Chaincode is packaged
Installing chaincode on peer0.org1...
Using organization 1
+ peer lifecycle chaincode queryinstalled --output json
+ grep '^events_1.0:91d63daaf0fc6186b344f3ac1408b326cbfc54df3b1ed30c3ef32758b06af482$'
+ jq -r 'try (.installed_chaincodes[].package_id)'
+ test 1 -ne 0
+ peer lifecycle chaincode install events.tar.gz
+ res=0
2023-06-02 01:18:13.949 UTC 0001 INFO [cli.lifecycle.chaincode] submitInstallProposal -> Installed remotely: response:<status:200 payload:"\nKevents_1.0:91d63daaf0fc6186b344f3ac1408b326cbfc54df3b1ed30c3ef32758b06af482\022\nevents_1.0" > 
2023-06-02 01:18:13.950 UTC 0002 INFO [cli.lifecycle.chaincode] submitInstallProposal -> Chaincode code package identifier: events_1.0:91d63daaf0fc6186b344f3ac1408b326cbfc54df3b1ed30c3ef32758b06af482
Chaincode is installed on peer0.org1
Install chaincode on peer0.org2...
Using organization 2
+ peer lifecycle chaincode queryinstalled --output json
+ grep '^events_1.0:91d63daaf0fc6186b344f3ac1408b326cbfc54df3b1ed30c3ef32758b06af482$'
+ jq -r 'try (.installed_chaincodes[].package_id)'
+ test 1 -ne 0
+ peer lifecycle chaincode install events.tar.gz
+ res=0
2023-06-02 01:19:10.529 UTC 0001 INFO [cli.lifecycle.chaincode] submitInstallProposal -> Installed remotely: response:<status:200 payload:"\nKevents_1.0:91d63daaf0fc6186b344f3ac1408b326cbfc54df3b1ed30c3ef32758b06af482\022\nevents_1.0" > 
2023-06-02 01:19:10.529 UTC 0002 INFO [cli.lifecycle.chaincode] submitInstallProposal -> Chaincode code package identifier: events_1.0:91d63daaf0fc6186b344f3ac1408b326cbfc54df3b1ed30c3ef32758b06af482
Chaincode is installed on peer0.org2
Using organization 1
+ peer lifecycle chaincode queryinstalled --output json
+ jq -r 'try (.installed_chaincodes[].package_id)'
+ grep '^events_1.0:91d63daaf0fc6186b344f3ac1408b326cbfc54df3b1ed30c3ef32758b06af482$'
+ res=0
events_1.0:91d63daaf0fc6186b344f3ac1408b326cbfc54df3b1ed30c3ef32758b06af482
Query installed successful on peer0.org1 on channel
Using organization 1
+ peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/tlsca/tlsca.example.com-cert.pem --channelID mychannel --name events --version 1.0 --package-id events_1.0:91d63daaf0fc6186b344f3ac1408b326cbfc54df3b1ed30c3ef32758b06af482 --sequence 1 --signature-policy 'OR('\''Org1MSP.peer'\'','\''Org2MSP.peer'\'')'
+ res=0
2023-06-02 01:19:12.762 UTC 0001 INFO [chaincodeCmd] ClientWait -> txid [69e59ae697e76e566f2eb17ec8e847d656d363027672f0cf6916cc6a97f80ddb] committed with status (VALID) at localhost:7051
Chaincode definition approved on peer0.org1 on channel 'mychannel'
Using organization 1
Checking the commit readiness of the chaincode definition on peer0.org1 on channel 'mychannel'...
Attempting to check the commit readiness of the chaincode definition on peer0.org1, Retry after 3 seconds.
+ peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name events --version 1.0 --sequence 1 --signature-policy 'OR('\''Org1MSP.peer'\'','\''Org2MSP.peer'\'')' --output json
+ res=0
{
        "approvals": {
                "Org1MSP": true,
                "Org2MSP": false
        }
}
Checking the commit readiness of the chaincode definition successful on peer0.org1 on channel 'mychannel'
Using organization 2
Checking the commit readiness of the chaincode definition on peer0.org2 on channel 'mychannel'...
Attempting to check the commit readiness of the chaincode definition on peer0.org2, Retry after 3 seconds.
+ peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name events --version 1.0 --sequence 1 --signature-policy 'OR('\''Org1MSP.peer'\'','\''Org2MSP.peer'\'')' --output json
+ res=0
{
        "approvals": {
                "Org1MSP": true,
                "Org2MSP": false
        }
}
Checking the commit readiness of the chaincode definition successful on peer0.org2 on channel 'mychannel'
Using organization 2
+ peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/tlsca/tlsca.example.com-cert.pem --channelID mychannel --name events --version 1.0 --package-id events_1.0:91d63daaf0fc6186b344f3ac1408b326cbfc54df3b1ed30c3ef32758b06af482 --sequence 1 --signature-policy 'OR('\''Org1MSP.peer'\'','\''Org2MSP.peer'\'')'
+ res=0
2023-06-02 01:19:21.125 UTC 0001 INFO [chaincodeCmd] ClientWait -> txid [e089c9a747fca7b1758a3c02ad29b6084180b7727373b36d73c0a69ca3356679] committed with status (VALID) at localhost:9051
Chaincode definition approved on peer0.org2 on channel 'mychannel'
Using organization 1
Checking the commit readiness of the chaincode definition on peer0.org1 on channel 'mychannel'...
Attempting to check the commit readiness of the chaincode definition on peer0.org1, Retry after 3 seconds.
+ peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name events --version 1.0 --sequence 1 --signature-policy 'OR('\''Org1MSP.peer'\'','\''Org2MSP.peer'\'')' --output json
+ res=0
{
        "approvals": {
                "Org1MSP": true,
                "Org2MSP": true
        }
}
Checking the commit readiness of the chaincode definition successful on peer0.org1 on channel 'mychannel'
Using organization 2
Checking the commit readiness of the chaincode definition on peer0.org2 on channel 'mychannel'...
Attempting to check the commit readiness of the chaincode definition on peer0.org2, Retry after 3 seconds.
+ peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name events --version 1.0 --sequence 1 --signature-policy 'OR('\''Org1MSP.peer'\'','\''Org2MSP.peer'\'')' --output json
+ res=0
{
        "approvals": {
                "Org1MSP": true,
                "Org2MSP": true
        }
}
Checking the commit readiness of the chaincode definition successful on peer0.org2 on channel 'mychannel'
Using organization 1
Using organization 2
+ peer lifecycle chaincode commit -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/ordererOrganizations/example.com/tlsca/tlsca.example.com-cert.pem --channelID mychannel --name events --peerAddresses localhost:7051 --tlsRootCertFiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/tlsca/tlsca.org1.example.com-cert.pem --peerAddresses localhost:9051 --tlsRootCertFiles /home/magpie/go/src/github.com/hyperledger/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/tlsca/tlsca.org2.example.com-cert.pem --version 1.0 --sequence 1 --signature-policy 'OR('\''Org1MSP.peer'\'','\''Org2MSP.peer'\'')'
+ res=0
2023-06-02 01:19:29.423 UTC 0001 INFO [chaincodeCmd] ClientWait -> txid [9703937c50afe13cf6adb05aa235f54c1b80087a8bfc642850c29fa74de2a64f] committed with status (VALID) at localhost:7051
2023-06-02 01:19:29.427 UTC 0002 INFO [chaincodeCmd] ClientWait -> txid [9703937c50afe13cf6adb05aa235f54c1b80087a8bfc642850c29fa74de2a64f] committed with status (VALID) at localhost:9051
Chaincode definition committed on channel 'mychannel'
Using organization 1
Querying chaincode definition on peer0.org1 on channel 'mychannel'...
Attempting to Query committed status on peer0.org1, Retry after 3 seconds.
+ peer lifecycle chaincode querycommitted --channelID mychannel --name events
+ res=0
Committed chaincode definition for chaincode 'events' on channel 'mychannel':
Version: 1.0, Sequence: 1, Endorsement Plugin: escc, Validation Plugin: vscc, Approvals: [Org1MSP: true, Org2MSP: true]
Query chaincode definition successful on peer0.org1 on channel 'mychannel'
Using organization 2
Querying chaincode definition on peer0.org2 on channel 'mychannel'...
Attempting to Query committed status on peer0.org2, Retry after 3 seconds.
+ peer lifecycle chaincode querycommitted --channelID mychannel --name events
+ res=0
Committed chaincode definition for chaincode 'events' on channel 'mychannel':
Version: 1.0, Sequence: 1, Endorsement Plugin: escc, Validation Plugin: vscc, Approvals: [Org1MSP: true, Org2MSP: true]
Query chaincode definition successful on peer0.org2 on channel 'mychannel'
Chaincode initialization is not required
magpie@DLTFabricer:~/go/src/github.com/hyperledger/fabric-samples/test-network$ 
```

运行截图：

![image-20230602092709517](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20230602092709517.png)

3.运行应用程序（从文件夹）。`asset-transfer-events`

```bash
# To run the Go sample application
cd application-gateway-go
go run .

# To run the Typescript sample application
cd application-gateway-typescript
npm install
npm start

# To run the Java sample application
cd application-gateway-java
./gradlew run
```

运行详细信息：

```bash
magpie@DLTFabricer:~/go/src/github.com/hyperledger/fabric-samples/asset-transfer-events/application-gateway-go$ ls
app.go  connect.go  go.mod  go.sum
magpie@DLTFabricer:~/go/src/github.com/hyperledger/fabric-samples/asset-transfer-events/application-gateway-go$ go run .

*** Start chaincode event listening

--> Submit transaction: CreateAsset, asset1685669326994 owned by Sam with appraised value 100

*** CreateAsset committed successfully

--> Submit transaction: UpdateAsset, asset1685669326994 update appraised value to 200

<-- Chaincode event received: CreateAsset - {
  "ID": "asset1685669326994",
  "Color": "blue",
  "Size": "10",
  "Owner": "Sam",
  "AppraisedValue": "100"
}

<-- Chaincode event received: UpdateAsset - {
  "ID": "asset1685669326994",
  "Color": "blue",
  "Size": "10",
  "Owner": "Sam",
  "AppraisedValue": "200"
}

*** UpdateAsset committed successfully

--> Submit transaction: TransferAsset, asset1685669326994 to Mary

<-- Chaincode event received: TransferAsset - {
  "ID": "asset1685669326994",
  "Color": "blue",
  "Size": "10",
  "Owner": "Mary",
  "AppraisedValue": "200"
}

*** TransferAsset committed successfully

--> Submit transaction: DeleteAsset, asset1685669326994

*** DeleteAsset committed successfully

<-- Chaincode event received: DeleteAsset - {
  "ID": "asset1685669326994",
  "Color": "blue",
  "Size": "10",
  "Owner": "Mary",
  "AppraisedValue": "200"
}

*** Start chaincode event replay

<-- Chaincode event replayed: CreateAsset - {
  "ID": "asset1685669326994",
  "Color": "blue",
  "Size": "10",
  "Owner": "Sam",
  "AppraisedValue": "100"
}

<-- Chaincode event replayed: UpdateAsset - {
  "ID": "asset1685669326994",
  "Color": "blue",
  "Size": "10",
  "Owner": "Sam",
  "AppraisedValue": "200"
}

<-- Chaincode event replayed: TransferAsset - {
  "ID": "asset1685669326994",
  "Color": "blue",
  "Size": "10",
  "Owner": "Mary",
  "AppraisedValue": "200"
}

<-- Chaincode event replayed: DeleteAsset - {
  "ID": "asset1685669326994",
  "Color": "blue",
  "Size": "10",
  "Owner": "Mary",
  "AppraisedValue": "200"
}

magpie@DLTFabricer:~/go/src/github.com/hyperledger/fabric-samples/asset-transfer-events/application-gateway-go$ 
```

运行截图：

![image-20230602092931649](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20230602092931649.png)

### 收拾

完成后，您可以关闭测试网络（从文件夹中）。该命令将删除测试网络的所有节点，并删除您创建的任何账本数据。`test-network`

```bash
./network.sh down
```



## 案例三、链下数据存储示例

链下数据存储示例演示：

- 在客户端应用程序中接收块事件。
- 使用检查指针在发生故障或应用程序重新启动后恢复事件侦听。
- 从区块事件中提取账本更新，以构建链下数据存储。

### 关于示例

此示例演示如何将区块链网络中的数据复制到链下数据存储。使用链下数据存储，您可以分析来自网络的数据或构建仪表板，而不会降低应用程序的性能。

此示例使用适用于*Fabric v2.4* 及更高版本的[Fabric-gateway client API](https://hyperledger.github.io/fabric-gateway/) 的块事件侦听功能。

#### 应用

客户端应用程序提供了几个可以使用命令行调用的“命令”：

- **getAllAssets**：检索账本上记录的所有资产的当前详细信息。看：
  - TypeScript： [application-typescript/src/getAllAssets.ts](https://github.com/hyperledger/fabric-samples/blob/main/off_chain_data/application-typescript/src/getAllAssets.ts)
  - Java： [application-java/app/src/main/java/GetAllAssets.java](https://github.com/hyperledger/fabric-samples/blob/main/off_chain_data/application-java/app/src/main/java/GetAllAssets.java)
- **listen**：侦听区块事件，并使用它们在链下数据存储中复制账本更新。看：
  - TypeScript： [application-typescript/src/listen.ts](https://github.com/hyperledger/fabric-samples/blob/main/off_chain_data/application-typescript/src/listen.ts)
  - Java： [application-java/app/src/main/java/Listen.java](https://github.com/hyperledger/fabric-samples/blob/main/off_chain_data/application-java/app/src/main/java/Listen.java)
- 交易：提交一组**交易**以创建、修改和删除资产。看：
  - TypeScript： [application-typescript/src/transact.ts](https://github.com/hyperledger/fabric-samples/blob/main/off_chain_data/application-typescript/src/transact.ts)
  - Java： [application-java/app/src/main/java/Transact.java](https://github.com/hyperledger/fabric-samples/blob/main/off_chain_data/application-java/app/src/main/java/Transact.java)

为了使示例代码保持简洁，**listen** 命令将账本更新写入当前工作目录（对于 Java 示例为目录）中命名的输出文件。真正的实现可以将账本更新直接写入所选的链下数据存储。可以在运行示例时检查此文件中捕获的信息。`store.log``application-java/app`

请注意，**监听**命令是可重新启动的，并将在上次成功处理的块/事务后恢复事件侦听。这是使用检查指针来保持当前侦听位置来实现的。检查点状态将保存到当前工作目录中命名的文件中。如果不存在检查点状态，则事件侦听从账本的开头（块号零）开始。`checkpoint.json`

#### 智能合约

资产转移基本智能合约用于生成交易和相关分类账更新。

### 运行示例

结构测试网络用于部署和运行此示例。请按顺序执行以下步骤：

1.创建测试网络和通道（从文件夹中）。`test-network`

```bash
./network.sh up createChannel -c mychannel -ca
```

详情请见示例一。

2.部署资产转移基本智能合约实现之一（从文件夹中）。`test-network`

```bash
# To deploy the TypeScript chaincode implementation
./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-typescript/ -ccl typescript

# To deploy the Go chaincode implementation
./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go/ -ccl go

# To deploy the Java chaincode implementation
./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-java/ -ccl java
```

详见示例一。

3.使用一些资产填充账本，并使用事件捕获账本更新（从文件夹中）。`off_chain_data`

```bash
# To run the TypeScript sample application
cd application-typescript
npm install
npm start transact listen

# To run the Java sample application
cd application-java
./gradlew run --quiet --args='transact listen'
```

运行详细信息：

```bash
magpie@DLTFabricer:~/go/src/github.com/hyperledger/fabric-samples$ cd off_chain_data
magpie@DLTFabricer:~/go/src/github.com/hyperledger/fabric-samples/off_chain_data$ ls
application-java  application-typescript  legacy-application-javascript  README.md
magpie@DLTFabricer:~/go/src/github.com/hyperledger/fabric-samples/off_chain_data$ cd application-typescript
magpie@DLTFabricer:~/go/src/github.com/hyperledger/fabric-samples/off_chain_data/application-typescript$ ls
package.json  src  tsconfig.json
magpie@DLTFabricer:~/go/src/github.com/hyperledger/fabric-samples/off_chain_data/application-typescript$ npm install

> off-chain-data@1.0.0 prepare
> npm run build


> off-chain-data@1.0.0 build
> tsc

added 175 packages in 21s
magpie@DLTFabricer:~/go/src/github.com/hyperledger/fabric-samples/off_chain_data/application-typescript$ npm start transact listen

> off-chain-data@1.0.0 start
> node ./dist/app transact listen

Created asset 0d1d120a-b0ad-421f-838b-5ed33b55704d
Created asset 1b4e2dde-588f-4051-99aa-0a0845fd4bc4
Created asset 18ca8c5b-388b-429e-b2bb-cd98f3d3dad4
Created asset b722d8c5-382a-43fe-80ff-dfcfb36daf09
Created asset 6e6522b9-13be-4dab-ad7c-011d1390ab42
Created asset 4220f424-eb86-4d9f-8cd2-07f0aa16b240
Created asset a937a608-a625-46fe-bde8-a943cadbb514
Created asset 880852ba-55eb-4b46-9cf4-7128c5677043
Created asset 473375fd-b35b-46bf-96e7-50ddcd81c0d3
Created asset 0823b6f2-fb64-45ce-9768-0f479d2090ae
Deleted asset 0823b6f2-fb64-45ce-9768-0f479d2090ae
Transferred asset a937a608-a625-46fe-bde8-a943cadbb514 from charlie to bob
Transferred asset 4220f424-eb86-4d9f-8cd2-07f0aa16b240 from bob to alice
Transferred asset 1b4e2dde-588f-4051-99aa-0a0845fd4bc4 from bob to charlie
Transferred asset 880852ba-55eb-4b46-9cf4-7128c5677043 from charlie to bob
Transferred asset 473375fd-b35b-46bf-96e7-50ddcd81c0d3 from bob to alice
Transferred asset 6e6522b9-13be-4dab-ad7c-011d1390ab42 from bob to alice
Deleted asset 0d1d120a-b0ad-421f-838b-5ed33b55704d
Starting event listening from block 0
Last processed transaction ID within block: undefined

Received block 0

Received block 1

Received block 2

Received block 3
Skipping read-only or system transaction e2b2fe2c0512cb3bf36f219a2199d46e1aef556440f58bbe81339213b1ae3961

Received block 4
Skipping read-only or system transaction 1ca2aca4b3ddaf9062ffc233d104348b26b36b703b781a116e0dcf330841024d

Received block 5
Skipping read-only or system transaction f68dc92e7b99f6348e2e1e1a73a0954e3f741c4772034f9c848b784f51c7c1ed

Received block 6
Process transaction 982eae4998837fd08b3facc2f05fd941dfbfd4d017a1851b7ade1180fda921a6

Received block 7
Process transaction 8f54779af452a36e3ec6cb873dd416567dc2aecd143e8d310fdd1fd8fa022f7b

Received block 8
Process transaction 9cae665314b2825d566f758b40f7fe798ac9e7e1fb90a028e1ae2a3954d39e44

Received block 9
Skipping read-only or system transaction 69e59ae697e76e566f2eb17ec8e847d656d363027672f0cf6916cc6a97f80ddb

Received block 10
Skipping read-only or system transaction e089c9a747fca7b1758a3c02ad29b6084180b7727373b36d73c0a69ca3356679

Received block 11
Skipping read-only or system transaction 9703937c50afe13cf6adb05aa235f54c1b80087a8bfc642850c29fa74de2a64f

Received block 12
Process transaction 30529b97aa3fbeb0e77c5ad7daabe726dabbed2a31022ba9793cc0730cb215b5

Received block 13
Process transaction ea5ba002c7bc61dea7e1effa518dd7501a1f62761a1fe13ba914186e5168cdfe

Received block 14
Process transaction 123be1dc8d65312251456a959f8fdec2a91ab3d06bd634ea44f4e4142021a987

Received block 15
Process transaction 4911ba86f71e1674eee664048851ec796484a7876c73a4c885edbc49d0201612

Received block 16
Process transaction 0d79632cb2891aebdd72c99b0931cf4cd919890479141cb4f5d67beb64b44117

Received block 17
Process transaction ff5a7cb944ad64e524e9021e7b97c27e58247b00283da9496f5c671b9c59241e

Received block 18
Process transaction b645969efae388c1e7c5e9b3122d1edefd76f20bde8176a03de18b67958c681e

Received block 19
Process transaction f0b90487489043d692d5acbe4f8f81c96b3dadbd3458b02f21a14c224713c463

Received block 20
Process transaction 8954cdd074d665f8a28bf363896c6314e5202eaddb470feb57fc2fb7306fa97b
Process transaction 0a4e04c49894141b1f848f69e44b6ae244e364e35aa793f1a7ac2dadbe6b1fe2
Process transaction 63c1d2560371ced1572aeeaa00cec82178265686fecac3153068b59eabc3178d
Process transaction 84a7206a9e424c2dbf3b937ce49ef24d1c8cf600e316099c260728021a5dff46
Process transaction 5a6c046fef6f1d280e681364a7cded6f5d149b23c1fb1d592070fbc4642bd464
Process transaction 23732760cb56d516f5d1a7f13c0583971bc7f44bc89a1a6ef4cec18f752d549e
Process transaction c58b061cc4421e413834e6ad8c6cfd31dfd50544fa09fc456d996d3a47d31a40
Process transaction 7e694f30ae5fb34c3b66385d4794cc3e041d8deed1933063a67e13d905fd479c
Process transaction d629eef6ff262793494d1d1e8c80f27e4df231fa9471bf0ebbbb26be4c5555c6
Process transaction 126b209ee50e95d41a204b3d498030270d4a43c1e5ef7c681b400330279483c2

Received block 21
Process transaction 3892cb26bc6aacf2fd9d24cd30fb50e74acd27653d4f835a7c7b2c49d4f71fcb
Process transaction cf5b18c90588694a7d15bdbfb61c5a743f8c00fa2859743ebc12843fe3105e19
Process transaction af309b42fec714a35449539e377d37817cd1633ef8f557336a9707600441465a
Process transaction 69ed35c57b8dd24fae25e554570d8d188fce4a678f8e32167463430d0a4db732
Process transaction e909cef5c8f00cd4ec5290efe0f460a995db27ba2c40c2404cb10edfcd443734
Process transaction dc0dfc43406335978e14b0050d0eb723469b340518156d506a85399f8d19d174
Process transaction f1bdc52b58fda00ba9da0bc44d2e4d8ed71a9427696df0a754475bdbbf0f4dc7
Process transaction 6a141f936abf7c94143d881576ebe5fd38ffd02cab50eb66aa4cf5b23b5ab577

```

运行截图：

![image-20230602110412008](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20230602110412008.png)



4.使用 **Control-C** 中断侦听器进程。

截图：

![image-20230602110501965](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20230602110501965.png)

5.查看区块链的当前世界状态（从文件夹中）。您可能希望将结果与文件中侦听器捕获的账本更新进行比较。`off_chain_data``store.log`

```bash
# To run the TypeScript sample application
cd application-typescript
npm --silent start getAllAssets

# To run the Java sample application
cd application-java
./gradlew run --quiet --args=getAllAssets
```

详细信息：

```bash
magpie@DLTFabricer:~/go/src/github.com/hyperledger/fabric-samples/off_chain_data/application-typescript$ ls
checkpoint.json  dist  node_modules  package.json  package-lock.json  src  store.log  tsconfig.json
magpie@DLTFabricer:~/go/src/github.com/hyperledger/fabric-samples/off_chain_data/application-typescript$ npm --silent start getAllAssets
[
  {
    "AppraisedValue": 566,
    "Color": "red",
    "ID": "18ca8c5b-388b-429e-b2bb-cd98f3d3dad4",
    "Owner": "charlie",
    "Size": 7
  },
  {
    "AppraisedValue": 993,
    "Color": "blue",
    "ID": "1b4e2dde-588f-4051-99aa-0a0845fd4bc4",
    "Owner": "charlie",
    "Size": 3
  },
  {
    "AppraisedValue": 742,
    "Color": "blue",
    "ID": "4220f424-eb86-4d9f-8cd2-07f0aa16b240",
    "Owner": "alice",
    "Size": 6
  },
  {
    "AppraisedValue": 769,
    "Color": "green",
    "ID": "473375fd-b35b-46bf-96e7-50ddcd81c0d3",
    "Owner": "alice",
    "Size": 10
  },
  {
    "AppraisedValue": 998,
    "Color": "blue",
    "ID": "6e6522b9-13be-4dab-ad7c-011d1390ab42",
    "Owner": "alice",
    "Size": 7
  },
  {
    "AppraisedValue": 439,
    "Color": "green",
    "ID": "880852ba-55eb-4b46-9cf4-7128c5677043",
    "Owner": "bob",
    "Size": 3
  },
  {
    "AppraisedValue": 52,
    "Color": "blue",
    "ID": "a937a608-a625-46fe-bde8-a943cadbb514",
    "Owner": "bob",
    "Size": 8
  },
  {
    "AppraisedValue": 350,
    "Color": "blue",
    "ID": "asset1",
    "Owner": "Tom",
    "Size": 5
  },
  {
    "AppraisedValue": 1300,
    "Color": "yellow",
    "ID": "asset1685667849677",
    "Owner": "Mark",
    "Size": 5
  },
  {
    "AppraisedValue": 400,
    "Color": "red",
    "ID": "asset2",
    "Owner": "Brad",
    "Size": 5
  },
  {
    "AppraisedValue": 500,
    "Color": "green",
    "ID": "asset3",
    "Owner": "Jin Soo",
    "Size": 10
  },
  {
    "AppraisedValue": 600,
    "Color": "yellow",
    "ID": "asset4",
    "Owner": "Max",
    "Size": 10
  },
  {
    "AppraisedValue": 1300,
    "Color": "yellow",
    "ID": "asset413",
    "Owner": "Tom",
    "Size": 5
  },
  {
    "AppraisedValue": 700,
    "Color": "black",
    "ID": "asset5",
    "Owner": "Adriana",
    "Size": 15
  },
  {
    "AppraisedValue": 800,
    "Color": "white",
    "ID": "asset6",
    "Owner": "Michel",
    "Size": 15
  },
  {
    "AppraisedValue": 553,
    "Color": "red",
    "ID": "b722d8c5-382a-43fe-80ff-dfcfb36daf09",
    "Owner": "charlie",
    "Size": 8
  }
]
magpie@DLTFabricer:~/go/src/github.com/hyperledger/fabric-samples/off_chain_data/application-typescript$ 
```

运行截图：

![image-20230602110736404](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20230602110736404.png)

6.进行更多账本更新，然后观察侦听器恢复功能（从文件夹中）。从记录到控制台的事务 ID 中注意，侦听器恰好在上次成功处理事务之后恢复。`off_chain_data`

```bash
# To run the TypeScript sample application
cd application-typescript
npm start transact
SIMULATED_FAILURE_COUNT=5 npm start listen
npm start listen

# To run the Java sample application
cd application-java
./gradlew run --quiet --args=transact

SIMULATED_FAILURE_COUNT=5 ./gradlew run --quiet --args=listen
./gradlew run --quiet --args=listen
```

运行详细信息：

```bash
magpie@DLTFabricer:~/go/src/github.com/hyperledger/fabric-samples/off_chain_data/application-typescript$ npm start transact

> off-chain-data@1.0.0 start
> node ./dist/app transact

Created asset 7870408d-e410-4726-8141-236945931da3
Created asset a58bef54-fff2-48be-b829-11bab70f27cd
Created asset 829e3ac8-c79e-41e8-8ae5-6bb21193248c
Created asset d6a50328-8b23-4459-807d-b001973c6eeb
Created asset 0e1c0bae-0d3e-466d-b80f-bfd91a990954
Created asset 258802ed-7b1b-48bb-9442-c66db917efef
Created asset 6ef10097-2bf7-4e5f-8d2a-5394eecbb886
Created asset c6de616d-392b-4775-aa99-be90ce13eb79
Created asset bf02cac0-f554-4067-b29c-31fcc358e011
Created asset d4c88a80-2330-49f9-b5b0-256e07587157
Transferred asset bf02cac0-f554-4067-b29c-31fcc358e011 from alice to bob
Transferred asset d6a50328-8b23-4459-807d-b001973c6eeb from alice to bob
Transferred asset 7870408d-e410-4726-8141-236945931da3 from alice to bob
Deleted asset a58bef54-fff2-48be-b829-11bab70f27cd
Transferred asset 6ef10097-2bf7-4e5f-8d2a-5394eecbb886 from bob to alice
Transferred asset 258802ed-7b1b-48bb-9442-c66db917efef from charlie to bob
Deleted asset 7870408d-e410-4726-8141-236945931da3
magpie@DLTFabricer:~/go/src/github.com/hyperledger/fabric-samples/off_chain_data/application-typescript$ SIMULATED_FAILURE_COUNT=5 npm start listen

> off-chain-data@1.0.0 start
> node ./dist/app listen

Starting event listening from block 22
Last processed transaction ID within block: undefined
Simulating a write failure every 5 transactions

Received block 22
Process transaction 1895de60d78b211be71c4f2a245a1eb0b475b77ff94502ef80ec44bbb5b03b99
Process transaction b5ff54f04c3f063e1d6fee64254c2a01f981a134f06115f322b4de0e4c07845d
Process transaction 81338573ac90add02de0d1e91ec2488f932fea2143fc4f022eda2bb4237154dc
Process transaction c32dc8915c8f5da8f9bfe3f922ebe7ce5ba0b790910dcf7c3123320959a33cb7
Process transaction ffacaa957961c4d677c51bbaf34ebf0a0c6f2372f2247501715847d1e5ace8bb
Process transaction 6e011a2be49d70d4d6df406d1a73e6f635687d4aa59d38ecb3ab25e48579fe8a
ExpectedError: Simulated write failure
    at simulateFailureIfRequired (/home/magpie/go/src/github.com/hyperledger/fabric-samples/off_chain_data/application-typescript/dist/listen.js:213:15)
    at TransactionProcessor.applyWritesToOffChainStore (/home/magpie/go/src/github.com/hyperledger/fabric-samples/off_chain_data/application-typescript/dist/listen.js:71:5)
    at TransactionProcessor.process (/home/magpie/go/src/github.com/hyperledger/fabric-samples/off_chain_data/application-typescript/dist/listen.js:175:78)
    at BlockProcessor.process (/home/magpie/go/src/github.com/hyperledger/fabric-samples/off_chain_data/application-typescript/dist/listen.js:134:40)
    at async main (/home/magpie/go/src/github.com/hyperledger/fabric-samples/off_chain_data/application-typescript/dist/listen.js:101:17)
    at async main (/home/magpie/go/src/github.com/hyperledger/fabric-samples/off_chain_data/application-typescript/dist/app.js:34:13)
magpie@DLTFabricer:~/go/src/github.com/hyperledger/fabric-samples/off_chain_data/application-typescript$ npm start listen

> off-chain-data@1.0.0 start
> node ./dist/app listen

Starting event listening from block 22
Last processed transaction ID within block: ffacaa957961c4d677c51bbaf34ebf0a0c6f2372f2247501715847d1e5ace8bb

Received block 22
Process transaction 6e011a2be49d70d4d6df406d1a73e6f635687d4aa59d38ecb3ab25e48579fe8a
Process transaction 10ee6f64c13b2bef82d24067f66acbceaa7b7bab77ac62b7e497f0195565d6f9
Process transaction c67b8fa9dcb0ef68061f1481b26d3dc7bf21982fc8d3749f47763be8e6b4b316
Process transaction 50d01699f4fbec22fa1fd637ce8fa8484743cecf0eb0033f92c2f2cb02a0e885
Process transaction 63baf7857ab95a7c614d49ae7b89e64d93dbb0630d665e7eac27b8ce514e111b

Received block 23
Process transaction 791b77422aaf8c99ed95ddbe7cd074ebdefb85464ab9c73d7d0840cc41d6fdd9
Process transaction d85acf44d28b8cd7d95d46a20dd7b44882588eb54bbca55be7ea9d286f17f6a1
Process transaction f4894eb2976e55e471c1cfc31f935374e806a23f91379b142a2a2b18099bdd38
Process transaction 46a07e638f834e51ae2763f00f09b9d1a2f48bed01e36cb00a16131067de36c2
Process transaction 95f21536aae2c3f45b5d59b9ddf9e1271f6884460ddc08a4b3708e3287cd2fc7
Process transaction f0b70e38b3e36232cbb51f17c0196e4cacb5848e8b80225702f5d15de127e75f

Received block 24
Process transaction 027cedd54d2cbfe7fd57ea2cc9add1f160863815a345894742f83d1eb7df9dcd

```

运行截图：



7.使用 **Control-C**![image-20230602111212930](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20230602111212930.png)中断侦听器进程。



### 收拾

通过在侦听器停止时删除文件，可以删除持久化的事件检查点位置。`checkpoint.json`

可以通过删除文件来删除记录的账本更新。`store.log`

完成后，您可以关闭测试网络（从文件夹中）。该命令将删除测试网络的所有节点，并删除您创建的任何账本数据。在尝试使用新网络运行应用程序之前，请务必删除 和 文件。`test-network``checkpoint.json``store.log`

```bash
./network.sh down
```



# *Hyperledger Fabric SDK for Go*官方教程

此SDK使Go开发人员能够构建与[Hyperledger Fabric](http://hyperledger-fabric.readthedocs.io/en/latest/)交互的解决方案。

## 开始

获取适用于结构和结构 CA 的客户端 SDK 包。

```
go get github.com/hyperledger/fabric-sdk-go
```

你很高兴，快乐编码！查看使用演示示例。

### 文档

可以在[GoDoc](https://godoc.org/github.com/hyperledger/fabric-sdk-go)上查看SDK文档。

供最终开发人员使用的软件包与主 SDK 软件包 （pkg/fabsdk） 一起位于 pkg/client 文件夹中。

如果您希望使用结构“网关”编程模型，则 API 位于 pkg/gateway 文件夹中。

### 例子

- [E2E测试](https://github.com/hyperledger/fabric-sdk-go/blob/main/test/integration/e2e/end_to_end.go)：使用SDK查询和执行交易的基本示例
- [账本查询测试](https://github.com/hyperledger/fabric-sdk-go/blob/main/test/integration/pkg/client/ledger/ledger_queries_test.go)：使用 SDK 查询频道底层账本的基本示例
- [多组织测试](https://github.com/hyperledger/fabric-sdk-go/blob/main/test/integration/e2e/orgs/multiple_orgs_test.go)：一个涉及多个组织参与交易的示例
- 动态背书人选择：使用[动态背书人选择](https://github.com/hyperledger/fabric-sdk-go/blob/main/test/integration/pkg/fabsdk/provider/sdk_provider_test.go)的示例（基于链码策略）
- E2E PKCS11 测试：使用 [PKCS2](https://github.com/hyperledger/fabric-sdk-go/blob/main/test/integration/e2e/pkcs11/e2e_test.go) 加密套件和配置 的 E11E 测试
- 需要更多示例！

### 社区

- 讨论正在[火箭聊天](https://chat.hyperledger.org/channel/fabric-sdk-go)中进行。
- 问题跟踪在 [Jira](https://jira.hyperledger.org/secure/RapidBoard.jspa?projectKey=FAB&rapidView=7&view=planning) 中处理。

## 客户端开发工具包

### 当前兼容性

SDK 的集成测试针对三个标记的 Fabric 版本运行：

- 上一页（当前为 v1.4.7）
- 稳定版（当前为 v2.2.0）
- 预发行版（当前已禁用）

此外，出于开发目的，集成测试还会根据需要针对 devstable Fabric 版本运行。

### 已停用的版本

当“prev”代码级别更新时，下面列出了上次测试的fabric-sdk-go提交或标记。

- 结构 v1.3： AC70276
- 结构 v1.2：5e291d3
- 结构 v1.1： f7ae259
- 结构 v1.0： 5ac5226

### 运行测试套件

获取适用于Fabric和Fabric CA 的客户端 SDK 包。

```bash
git clone https://github.com/hyperledger/fabric-sdk-go.git

# In the Fabric SDK Go directory
cd fabric-sdk-go/

# Optional - Automatically install Go tools used by test suite
# make depend

# Running test suite
make

# Clean test suite run artifacts
make clean
```

### 去标签

可以提供以下 Go 标签以启用其他功能：

- 实验性：包括对实验性功能的支持。

## 为 Go SDK 做贡献

如果您想为 Go SDK 做出贡献，请运行测试套件并提交补丁以供审核。有关一般准则，请参阅 Fabric 项目[的贡献页面](http://hyperledger-fabric.readthedocs.io/en/latest/CONTRIBUTING.html)。

你需要：

- 围棋 1.14
- 做
- 码头工人
- 码头工人撰写
- 吉特
- gobin （GO111MODULE=off go get -u github.com/myitcv/gobin）
- 库工具

笔记：

- 依赖关系使用 [Go 模块](https://github.com/golang/go/wiki/Modules)处理。

### 运行测试套件的一部分

```
# In the Fabric SDK Go directory
cd fabric-sdk-go/

# Optional - Automatically install Go tools used by test suite
# make depend

# Optional - Running only code checks (linters, license, spelling, etc)
# make checks

# Running all unit tests and checks
make unit-test

# Running all integration tests
make integration-test
```

### 手动运行包单元测试

```
# In a package directory
go test
```

### 手动运行集成测试

你需要：

- 工作结构和织物-ca设置。建议您使用 中提供的 docker 撰写文件。还建议您使用 中提供的默认 .env 设置。请参阅以下步骤。`test/fixtures/dockerenv``test/fixtures/dockerenv`
- 自定义设置，以防您的Hyperledger Fabric网络未运行或使用不同的端口。`test/fixtures/config/config_test.yaml``localhost`

#### 在 Docker Hub 上使用结构映像进行测试

测试套件默认为 Docker Hub 上结构映像的最新兼容标记。 以下命令启动结构：

```
# In the Fabric SDK Go directory
cd fabric-sdk-go

# Start fabric (stable tag)
make dockerenv-stable-up

# Or more generally, start fabric at a different code level (prev, stable, prerelease, devstable)
# make dockerenv-[CODELEVEL]-up
```

#### 运行集成测试

结构现在应该正在运行。在不同的外壳中，运行集成测试

```
# In the Fabric SDK Go directory
cd fabric-sdk-go

# Use script to setup parameters for integration tests and execute them
# Previously we use to have hostnames like Fabric CA server, orderer and peer pointed to localhost
# Now since we removed this now, We will be using a different configuration
make integration-tests-local

# Or more generally, run integration tests at a different code level (prev, stable, prerelease, devstable)
# and fixture target version
# FABRIC_CODELEVEL_VER=[VER] FABRIC_CODELEVEL_TAG=[CODELEVEL] make integration-tests-local
# Previously we use to have hostnames like Fabric CA server, orderer and peer pointed to localhost
# Now since we removed this now, We will be using a different config file config_test_local.yaml
# which has the Fabric CA server, orderer and peers pointed to localhost
# It is also possible to run integration tests using go test directly. For example:
#cd fabric-sdk-go/test/integration/
#go test -args testLocal=true

#cd fabric-sdk-go/test/integration/orgs
#go test -args testLocal=true 

# You should review test/scripts/integration.sh for options and details.
# Note: you should generally prefer the scripted version to setup parameters for you.
```

#### 使用本地结构构建进行测试（高级）

或者，您可以使用以下命令使用 Fabric 的本地构建：

```
# Start fabric (devstable codelevel with latest docker tags)
make dockerenv-latest-up
```

## 许可证

Hyperledger Fabric SDK Go软件在[Apache许可证版本2.0](https://github.com/hyperledger/fabric-sdk-go/blob/main/LICENSE)下获得许可。

------

本文档根据[知识共享署名 4.0 国际许可协议](http://creativecommons.org/licenses/by/4.0/)进行许可。





# *Hyperledger Fabric-sdk-go*编程实战

首先，在区块链网络搭建完成以及链码编写完成并已经上链后，我们便可编写区块链应用层得编写了，编写应用层需要先编写SDK（请确保自己gopath里面有github.com/hyperledger/fabric-sdk-go包）依赖包， Golang依赖包可以自行下载。那么我们首先编写应用层的sdk。在编写sdk前需要完成sdk的配置文件，尤其是要进行通信的Fabric组件的地址。我们将所有内容放入一个新的配置文件，Fabric SDK Go配置和我们的自定义参数中。
使用如下命令创建config.yaml

```shell
 cd $GOPATH/src/github.com/chainHero/heroes-service && \
 vi config.yaml
```

以下是config.yaml的完整内容。

```yaml
name: "heroes-service-network"
#
# Schema version of the content. Used by the SDK to apply the corresponding parsing rules.
#
version: 1.0.0

#
# The client section used by GO SDK.
#
client:

  # Which organization does this application instance belong to? The value must be the name of an org
  # defined under "organizations"
  organization: org1

  logging:
    level: info

  # Global configuration for peer, event service and orderer timeouts
  # if this this section is omitted, then default values will be used (same values as below)
#  peer:
#    timeout:
#      connection: 10s
#      response: 180s
#      discovery:
#        # Expiry period for discovery service greylist filter
#        # The channel client will greylist peers that are found to be offline
#        # to prevent re-selecting them in subsequent retries.
#        # This interval will define how long a peer is greylisted
#        greylistExpiry: 10s
#  eventService:
#    # Event service type (optional). If not specified then the type is automatically
#    # determined from channel capabilities.
#    type: (deliver|eventhub)
    # the below timeouts are commented out to use the default values that are found in
    # "pkg/fab/endpointconfig.go"
    # the client is free to override the default values by uncommenting and resetting
    # the values as they see fit in their config file
#    timeout:
#      connection: 15s
#      registrationResponse: 15s
#  orderer:
#    timeout:
#      connection: 15s
#      response: 15s
#  global:
#    timeout:
#      query: 180s
#      execute: 180s
#      resmgmt: 180s
#    cache:
#      connectionIdle: 30s
#      eventServiceIdle: 2m
#      channelConfig: 30m
#      channelMembership: 30s
#      discovery: 10s
#      selection: 10m

  # Root of the MSP directories with keys and certs.
  cryptoconfig:
    path: ${GOPATH}/src/github.com/chainHero/heroes-service/fixtures/crypto-config

  # Some SDKs support pluggable KV stores, the properties under "credentialStore"
  # are implementation specific
  credentialStore:
    path: /tmp/heroes-service-store

    # [Optional]. Specific to the CryptoSuite implementation used by GO SDK. Software-based implementations
    # requiring a key store. PKCS#11 based implementations does not.
    cryptoStore:
      path: /tmp/heroes-service-msp

   # BCCSP config for the client. Used by GO SDK.
  BCCSP:
    security:
     enabled: true
     default:
      provider: "SW"
     hashAlgorithm: "SHA2"
     softVerify: true
     level: 256

  tlsCerts:
    # [Optional]. Use system certificate pool when connecting to peers, orderers (for negotiating TLS) Default: false
    systemCertPool: false

    # [Optional]. Client key and cert for TLS handshake with peers and orderers
    client:
      keyfile:
      certfile:

#
# [Optional]. But most apps would have this section so that channel objects can be constructed
# based on the content below. If an app is creating channels, then it likely will not need this
# section.
#
channels:
  # name of the channel
  chainhero:
    # Required. list of orderers designated by the application to use for transactions on this
    # channel. This list can be a result of access control ("org1" can only access "ordererA"), or
    # operational decisions to share loads from applications among the orderers.  The values must
    # be "names" of orgs defined under "organizations/peers"
    # deprecated: not recommended, to override any orderer configuration items, entity matchers should be used.
    # orderers:
    #  - orderer.example.com

    # Required. list of peers from participating orgs
    peers:
      peer0.org1.hf.chainhero.io:
        # [Optional]. will this peer be sent transaction proposals for endorsement? The peer must
        # have the chaincode installed. The app can also use this property to decide which peers
        # to send the chaincode install request. Default: true
        endorsingPeer: true

        # [Optional]. will this peer be sent query proposals? The peer must have the chaincode
        # installed. The app can also use this property to decide which peers to send the
        # chaincode install request. Default: true
        chaincodeQuery: true

        # [Optional]. will this peer be sent query proposals that do not require chaincodes, like
        # queryBlock(), queryTransaction(), etc. Default: true
        ledgerQuery: true

        # [Optional]. will this peer be the target of the SDK's listener registration? All peers can
        # produce events but the app typically only needs to connect to one to listen to events.
        # Default: true
        eventSource: true

      peer1.org1.hf.chainhero.io:

    policies:
      #[Optional] options for retrieving channel configuration blocks
      queryChannelConfig:
        #[Optional] min number of success responses (from targets/peers)
        minResponses: 1
        #[Optional] channel config will be retrieved for these number of random targets
        maxTargets: 1
        #[Optional] retry options for query config block
        retryOpts:
          #[Optional] number of retry attempts
          attempts: 5
          #[Optional] the back off interval for the first retry attempt
          initialBackoff: 500ms
          #[Optional] the maximum back off interval for any retry attempt
          maxBackoff: 5s
          #[Optional] he factor by which the initial back off period is exponentially incremented
          backoffFactor: 2.0


#
# list of participating organizations in this network
#
organizations:
  org1:
    mspid: org1.hf.chainhero.io
    cryptoPath: peerOrganizations/org1.hf.chainhero.io/users/{userName}@org1.hf.chainhero.io/msp
    peers:
      - peer0.org1.hf.chainhero.io
      - peer1.org1.hf.chainhero.io

    # [Optional]. Certificate Authorities issue certificates for identification purposes in a Fabric based
    # network. Typically certificates provisioning is done in a separate process outside of the
    # runtime network. Fabric-CA is a special certificate authority that provides a REST APIs for
    # dynamic certificate management (enroll, revoke, re-enroll). The following section is only for
    # Fabric-CA servers.
    certificateAuthorities:
      - ca.org1.hf.chainhero.io

#
# List of orderers to send transaction and channel create/update requests to. For the time
# being only one orderer is needed. If more than one is defined, which one get used by the
# SDK is implementation specific. Consult each SDK's documentation for its handling of orderers.
#
orderers:
  orderer.hf.chainhero.io:
    url: localhost:7050

    # these are standard properties defined by the gRPC library
    # they will be passed in as-is to gRPC client constructor
    grpcOptions:
      ssl-target-name-override: orderer.hf.chainhero.io
      # These parameters should be set in coordination with the keepalive policy on the server,
      # as incompatible settings can result in closing of connection.
      # When duration of the 'keep-alive-time' is set to 0 or less the keep alive client parameters are disabled
      keep-alive-time: 0s
      keep-alive-timeout: 20s
      keep-alive-permit: false
      fail-fast: false
      # allow-insecure will be taken into consideration if address has no protocol defined, if true then grpc or else grpcs
      allow-insecure: false

    tlsCACerts:
      # Certificate location absolute path
      path: ${GOPATH}/src/github.com/chainHero/heroes-service/fixtures/crypto-config/ordererOrganizations/hf.chainhero.io/tlsca/tlsca.hf.chainhero.io-cert.pem
#
# List of peers to send various requests to, including endorsement, query
# and event listener registration.
#
peers:
  peer0.org1.hf.chainhero.io:
    # this URL is used to send endorsement and query requests
    url: localhost:7051
    # eventUrl is only needed when using eventhub (default is delivery service)
    eventUrl: localhost:7053

    grpcOptions:
      ssl-target-name-override: peer0.org1.hf.chainhero.io
      # These parameters should be set in coordination with the keepalive policy on the server,
      # as incompatible settings can result in closing of connection.
      # When duration of the 'keep-alive-time' is set to 0 or less the keep alive client parameters are disabled
      keep-alive-time: 0s
      keep-alive-timeout: 20s
      keep-alive-permit: false
      fail-fast: false
      # allow-insecure will be taken into consideration if address has no protocol defined, if true then grpc or else grpcs
      allow-insecure: false

    tlsCACerts:
      # Certificate location absolute path
      path: ${GOPATH}/src/github.com/chainHero/heroes-service/fixtures/crypto-config/peerOrganizations/org1.hf.chainhero.io/tlsca/tlsca.org1.hf.chainhero.io-cert.pem

  peer1.org1.hf.chainhero.io:
    # this URL is used to send endorsement and query requests
    url: localhost:8051
    # eventUrl is only needed when using eventhub (default is delivery service)
    eventUrl: localhost:8053

    grpcOptions:
      ssl-target-name-override: peer1.org1.hf.chainhero.io
      # These parameters should be set in coordination with the keepalive policy on the server,
      # as incompatible settings can result in closing of connection.
      # When duration of the 'keep-alive-time' is set to 0 or less the keep alive client parameters are disabled
      keep-alive-time: 0s
      keep-alive-timeout: 20s
      keep-alive-permit: false
      fail-fast: false
      # allow-insecure will be taken into consideration if address has no protocol defined, if true then grpc or else grpcs
      allow-insecure: false

    tlsCACerts:
      # Certificate location absolute path
      path: ${GOPATH}/src/github.com/chainHero/heroes-service/fixtures/crypto-config/peerOrganizations/org1.hf.chainhero.io/tlsca/tlsca.org1.hf.chainhero.io-cert.pem

#
# Fabric-CA is a special kind of Certificate Authority provided by Hyperledger Fabric which allows
# certificate management to be done via REST APIs. Application may choose to use a standard
# Certificate Authority instead of Fabric-CA, in which case this section would not be specified.
#
certificateAuthorities:
  ca.org1.hf.chainhero.io:
    url: http://localhost:7054
    # Fabric-CA supports dynamic user enrollment via REST APIs. A "root" user, a.k.a registrar, is
    # needed to enroll and invoke new users.
    httpOptions:
      verify: false
    registrar:
      enrollId: admin
      enrollSecret: adminpw
    # [Optional] The optional name of the CA.
    caName: ca.org1.hf.chainhero.io
    tlsCACerts:
      # Certificate location absolute path
      path: ${GOPATH}/src/github.com/chainHero/heroes-service/fixtures/crypto-config/peerOrganizations/org1.hf.chainhero.io/ca/ca.org1.hf.chainhero.io-cert.pem

entityMatchers:
  peer:
    - pattern: (\w*)peer0.org1.hf.chainhero.io(\w*)
      urlSubstitutionExp: localhost:7051
      eventUrlSubstitutionExp: localhost:7053
      sslTargetOverrideUrlSubstitutionExp: peer0.org1.hf.chainhero.io
      mappedHost: peer0.org1.hf.chainhero.io

    - pattern: (\w*)peer1.org1.hf.chainhero.io(\w*)
      urlSubstitutionExp: localhost:8051
      eventUrlSubstitutionExp: localhost:8053
      sslTargetOverrideUrlSubstitutionExp: peer1.org1.hf.chainhero.io
      mappedHost: peer1.org1.hf.chainhero.io

  orderer:
    - pattern: (\w*)orderer.hf.chainhero.io(\w*)
      urlSubstitutionExp: localhost:7050
      sslTargetOverrideUrlSubstitutionExp: orderer.hf.chainhero.io
      mappedHost: orderer.hf.chainhero.io

  certificateAuthorities:
    - pattern: (\w*)ca.org1.hf.chainhero.io(\w*)
      urlSubstitutionExp: http://localhost:7054
      mappedHost: ca.org1.hf.chainhero.io
```

接下来我们需要创建一个文件夹

```sh
mkdir $GOPATH/src/github.com/chainHero/heroes-service/blockchain
```


创建一个文件为sdk.go

```sh
vi $GOPATH/src/github.com/chainHero/heroes-service/blockchain/sdk.go
```

便可以编写SDK了，代码如下。

```go
package blockchain

import (
	"github.com/hyperledger/fabric-sdk-go/pkg/client/channel"
	"github.com/hyperledger/fabric-sdk-go/pkg/core/config"
	"github.com/hyperledger/fabric-sdk-go/pkg/fabsdk"
)

var (
	SDK				*fabsdk.FabricSDK//定义一个指向fabsdk.go里面的FabricSDK的结构体
	ChannelName		= "mychannel"//当前将要访问的智能合约所属通道名称
	ChaincodeName	= "mycc"//网络编写时的链码名称
	Org				= "org1"
	User			= "Admin"
	ConfigPath		= "/home/work/go/src/trace-back/application/config.yaml"//configPath配置路径
	TargetEndPoint1 = "node2.organization1.gdzce.cn"//指定背书节点
	TargetEndPoint2 = "node2.organization2.gdzce.cn"//指定背书节点
	TargetEndPoint3 = "node2.organization3.gdzce.cn"//指定背书节点
)

// 初始化
func Init()  { 
	var err error
	SDK, err = fabsdk.New(config.FromFile(ConfigPath))
	if err != nil {
		panic(err)
	}
}

// 交互
func ChannelExecute(fcn string, args [][]byte) (channel.Response, error) {
	// 创建客户端
	ctx := SDK.ChannelContext(ChannelName, fabsdk.WithOrg(Org), fabsdk.WithUser(User))
	cli, err := channel.New(ctx)
	if err != nil {
		return channel.Response{}, err
	}
	resp, err := cli.Execute(channel.Request{
		ChaincodeID:     ChaincodeName,
		Fcn:             fcn,
		Args:            args,
	}, channel.WithTargetEndpoints(TargetEndPoint1, TargetEndPoint2, TargetEndPoint3))
	if err != nil {
		return channel.Response{}, err
	}
	return resp, nil
}

// 查询
func ChannelQuery(fcn string, args [][]byte) (channel.Response, error) {
	// 创建客户端
	ctx := SDK.ChannelContext(ChannelName, fabsdk.WithOrg(Org), fabsdk.WithUser(User))
	cli, err := channel.New(ctx)
	if err != nil {
		return channel.Response{}, err
	}
	return cli.Query(channel.Request{
		ChaincodeID:     ChaincodeName,
		Fcn:             fcn,
		Args:            args,
	}, channel.WithTargetEndpoints(TargetEndPoint1, TargetEndPoint2, TargetEndPoint3))
}
```

这便完成了sdk的编写了，其它的后面再说。





# *Hyperledger fabric-sdk-go*实战教程

## 1.下载源代码

[sxguan/fabric-go-sdk (github.com)](https://github.com/sxguan/fabric-go-sdk)

源代码*clone*下载：

```bash
$ cd ~/go/src/github.com/hnublocklab
$ git clone https://github.com/sxguan/fabric-go-sdk.git
```

![image-20230525164902465](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20230525164902465.png)

## 2.网络启动

分布式账本网络采用*Fabric* 2.5.1版本部署，已经取消了系统通道，因此必须修改 *docker-compose.yaml*文件的相关配置。

```yaml
- ORDERER_GENERAL_GENESISMETHOD` #将启动方式改为 `none`
- ORDERER_CHANNELPARTICIPATION_ENABLED` #无系统通道启动，该字段必须设置为 true
```

 *docker-compose.yaml*文件如下：

```yaml
# Copyright IBM Corp. All Rights Reserved.
#
# SPDX-License-Identifier: Apache-2.0
#

version: '2'

volumes:
  orderer.example.com:
  peer0.org1.example.com:
  peer1.org1.example.com:

networks:
  test:

services:
  orderer.example.com:
    container_name: orderer.example.com
    image: hyperledger/fabric-orderer:latest
    environment:
      - FABRIC_LOGGING_SPEC=DEBUG
      - ORDERER_GENERAL_LISTENADDRESS=0.0.0.0
      - ORDERER_GENERAL_LISTENPORT=7050
      - ORDERER_GENERAL_BOOTSTRAPMETHOD=none
      - ORDERER_CHANNELPARTICIPATION_ENABLED=true
      #- ORDERER_GENERAL_GENESISMETHOD=file
      #- ORDERER_GENERAL_GENESISFILE=/var/hyperledger/orderer/orderer.genesis.block
      - ORDERER_GENERAL_LOCALMSPID=OrdererMSP
      - ORDERER_GENERAL_LOCALMSPDIR=/var/hyperledger/orderer/msp
      # enabled TLS
      - ORDERER_GENERAL_TLS_ENABLED=true
      - ORDERER_GENERAL_TLS_PRIVATEKEY=/var/hyperledger/orderer/tls/server.key
      - ORDERER_GENERAL_TLS_CERTIFICATE=/var/hyperledger/orderer/tls/server.crt
      - ORDERER_GENERAL_TLS_ROOTCAS=[/var/hyperledger/orderer/tls/ca.crt]
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric
    command: orderer
    volumes:
      - ./crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/msp:/var/hyperledger/orderer/msp
      - ./crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/:/var/hyperledger/orderer/tls
      - orderer.example.com:/var/hyperledger/production/orderer
    ports:
      - 7050:7050
    networks:
      - test

  peer0.org1.example.com:
    container_name: peer0.org1.example.com
    image: hyperledger/fabric-peer:latest
    environment:
      #Generic peer variables
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      # the following setting starts chaincode containers on the same
      # bridge network as the peers
      # https://docs.docker.com/compose/networking/
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=fixtures_test
      - FABRIC_LOGGING_SPEC=DEBUG
      #- FABRIC_LOGGING_SPEC=DEBUG
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_PROFILE_ENABLED=true
      - CORE_PEER_TLS_CERT_FILE=/etc/hyperledger/fabric/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/etc/hyperledger/fabric/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/etc/hyperledger/fabric/tls/ca.crt
      # Peer specific variabes
      - CORE_PEER_ID=peer0.org1.example.com
      - CORE_PEER_ADDRESS=peer0.org1.example.com:7051
      - CORE_PEER_LISTENADDRESS=0.0.0.0:7051
      - CORE_PEER_CHAINCODEADDRESS=peer0.org1.example.com:7052
      - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:7052
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer1.org1.example.com:9051
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.org1.example.com:7051
      - CORE_PEER_LOCALMSPID=Org1MSP
      - CORE_LEDGER_STATE_STATEDATABASE=CouchDB
      - CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb0:5984
      - CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME=admin
      - CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD=123456
    volumes:
      - /var/run/:/host/var/run/
      - ./crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp:/etc/hyperledger/fabric/msp
      - ./crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls:/etc/hyperledger/fabric/tls
      - peer0.org1.example.com:/var/hyperledger/production
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: peer node start
    ports:
      - 7051:7051
    depends_on:
      - orderer.example.com
      #- couchdb.org1.example.com
    networks:
      - test

  peer1.org1.example.com:
    container_name: peer1.org1.example.com
    image: hyperledger/fabric-peer:latest
    environment:
      #Generic peer variables
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      # the following setting starts chaincode containers on the same
      # bridge network as the peers
      # https://docs.docker.com/compose/networking/
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=fixtures_test
      - FABRIC_LOGGING_SPEC=DEBUG
      #- FABRIC_LOGGING_SPEC=DEBUG
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_PROFILE_ENABLED=true
      - CORE_PEER_TLS_CERT_FILE=/etc/hyperledger/fabric/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/etc/hyperledger/fabric/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/etc/hyperledger/fabric/tls/ca.crt
      # Peer specific variabes
      - CORE_PEER_ID=peer1.org1.example.com
      - CORE_PEER_ADDRESS=peer1.org1.example.com:9051
      - CORE_PEER_LISTENADDRESS=0.0.0.0:9051
      - CORE_PEER_CHAINCODEADDRESS=peer1.org1.example.com:9052
      - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:9052
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer1.org1.example.com:9051
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer0.org1.example.com:7051
      - CORE_PEER_LOCALMSPID=Org1MSP
      - CORE_LEDGER_STATE_STATEDATABASE=CouchDB
      - CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb1:5984
      - CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME=admin
      - CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD=123456
    volumes:
      - /var/run/:/host/var/run/
      - ./crypto-config/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/msp:/etc/hyperledger/fabric/msp
      - ./crypto-config/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/tls:/etc/hyperledger/fabric/tls
      - peer1.org1.example.com:/var/hyperledger/production
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: peer node start
    ports:
      - 9051:9051
    depends_on:
      - orderer.example.com
      #- couchdb.org1.example.com
    networks:
      - test

  ca.org1.example.com:
    image: hyperledger/fabric-ca:latest
    container_name: ca.org1.example.com
    environment:
      - FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server
      - FABRIC_CA_SERVER_CA_NAME=ca.org1.example.com
      - FABRIC_CA_SERVER_CA_CERTFILE=/etc/hyperledger/fabric-ca-server-config/ca.org1.example.com-cert.pem
      - FABRIC_CA_SERVER_CA_KEYFILE=/etc/hyperledger/fabric-ca-server-config/priv_sk
      - FABRIC_CA_SERVER_TLS_ENABLED=true
      - FABRIC_CA_SERVER_TLS_CERTFILE=/etc/hyperledger/fabric-ca-server-config/ca.org1.example.com-cert.pem
      - FABRIC_CA_SERVER_TLS_KEYFILE=/etc/hyperledger/fabric-ca-server-config/priv_sk
    ports:
      - 7054:7054
    command: sh -c 'fabric-ca-server start -b admin:adminpw -d'
    volumes:
      - ./crypto-config/peerOrganizations/org1.example.com/ca/:/etc/hyperledger/fabric-ca-server-config
    networks:
      - test

  couchdb0:
    container_name: couchdb0
    image: hyperledger/fabric-couchdb:latest
    # Populate the COUCHDB_USER and COUCHDB_PASSWORD to set an admin user and password
    # for CouchDB.  This will prevent CouchDB from operating in an "Admin Party" mode.
    environment:
      - COUCHDB_USER=admin
      - COUCHDB_PASSWORD=123456
    # Comment/Uncomment the port mapping if you want to hide/expose the CouchDB service,
    # for example map it to utilize Fauxton User Interface in dev environments.
    ports:
      - "5984:5984"
    networks:
      - test

  couchdb1:
    container_name: couchdb1
    image: hyperledger/fabric-couchdb:latest
    # Populate the COUCHDB_USER and COUCHDB_PASSWORD to set an admin user and password
    # for CouchDB.  This will prevent CouchDB from operating in an "Admin Party" mode.
    environment:
      - COUCHDB_USER=admin
      - COUCHDB_PASSWORD=123456
    # Comment/Uncomment the port mapping if you want to hide/expose the CouchDB service,
    # for example map it to utilize Fauxton User Interface in dev environments.
    ports:
      - "7984:5984"
    networks:
      - test
```

网络启动：

```bash
cd ~/go/src/github.com/hnublocklab/fabric-go-sdk/fixtures 
docker-compose up -d
```

![image-20230525215131918](C:\Users\acrow\AppData\Roaming\Typora\typora-user-images\image-20230525215131918.png)



# Hyperledger *fabric-SDK-Go*开发实战教程

## 0.概述

在实际部署时，一般将Fabric SDK作为API的提供者，为上层应用或对外服务提供Fabric区块链网络底层服务。Fabric SDK本质是一个Fabric基础对象，功能的封装集合和客户端框架实现，旨在方便开发人员快速开发出符合产品业务需求的服务模块。

获取 Fabric 和 Fabric CA 的客户端 SDK 包，SDK 文档可以在GoDoc中查看：

```bash
go get github.com/hyperledger/fabric-sdk-go
```


下面使用sdk实现创建通道，加入通道，打包链码，安装链码，升级链码等功能。

## 1.基本工作流程

1. 使用配置实例化一个 fabsdk 实例。注意：fabsdk 维护缓存，因此您应该最小化 fabsdk 本身的实例。
2. 使用您的 fabsdk 实例基于用户和组织创建上下文。注意：通道上下文还需要通道 ID。
3. 使用它的 New func 创建一个客户端实例，传递上下文。注意：您为所需的每个上下文创建一个新的客户端实例。
4. 使用每个客户提供的功能来创建您的解决方案！
5. 调用 fabsdk.Close() 释放资源和缓存。



## 2.config.yaml配置文件

配置文件包含client（客户端）、channels（通道）、organizations（组织）、orderers（排序节点列表）、peers（peer列表）、entityMatchers（实例匹配器）六部分信息。

其中client表示当前客户端信息，包含所属组织、密钥和证书文件的路径等， 这是每个client专用的信息。其余部分为fabric网络结构的信息，这些结构信息应当与configytx.yaml中是一致的。这是通用配置，每个客户端都可以拿来使用。另外，这部分信息并不需要是完整fabric网络信息，如果当前client只和部分节点交互，那配置文件中只需要包含所使用到的网络信息。

```yaml
version: 0.0.0

client:

 organization: org1

 logging:
   level: info

 cryptoconfig:
   path: ./fixtures/crypto-config

 credentialStore:

   path: "/tmp/state-store"

   cryptoStore:
     # Specific to the underlying KeyValueStore that backs the crypto key store.
     path: /tmp/msp

 BCCSP:
   security:
     enabled: true
     default:
       provider: "SW"
     hashAlgorithm: "SHA2"
     softVerify: true
     level: 256

 tlsCerts:
   # [Optional]. Use system certificate pool when connecting to peers, orderers (for negotiating TLS) Default: false
   systemCertPool: true

   # [Optional]. Client key and cert for TLS handshake with peers and orderers
   client:
     key:
       path: ./fixtures/crypto-config/peerOrganizations/org1.maakees.com/users/User1@org1.maakees.com/tls/client.key
     cert:
       path: ./fixtures/crypto-config/peerOrganizations/org1.maakees.com/users/User1@org1.maakees.com/tls/client.crt


channels:
 # multi-org test channel
 mychannel:
   peers:
     peer0.org1.maakees.com:
       endorsingPeer: true
       chaincodeQuery: true
       ledgerQuery: true
       eventSource: true
     peer.org1.maakees.com:
       endorsingPeer: true
       chaincodeQuery: true
       ledgerQuery: true
       eventSource: true

   policies:
     queryChannelConfig:
       minResponses: 1
       maxTargets: 1
       retryOpts:
         attempts: 5
         initialBackoff: 500ms
         maxBackoff: 5s
         backoffFactor: 2.0
#
# list of participating organizations in this network
#
organizations:
 org1:
   mspid: Org1MSP

   # This org's MSP store (absolute path or relative to client.cryptoconfig)
   cryptoPath: ./fixtures/crypto-config/peerOrganizations/org1.maakees.com/users/{username}@org1.maakees.com/msp

   peers:
     - peer0.org1.maakees.com
     - peer1.org1.maakees.com
   # failed to create resmgmt client due to context error: user not found
   users:
     Admin:
       cert:
         path: ./fixtures/crypto-config/peerOrganizations/org1.maakees.com/users/Admin@org1.maakees.com/msp/signcerts/Admin@org1.maakees.com-cert.pem
       key:
         path: ./fixtures/crypto-config/peerOrganizations/org1.maakees.com/users/Admin@org1.maakees.com/msp/keystore/priv_sk
     User1:
       cert:
         path: ./fixtures/crypto-config/peerOrganizations/org1.maakees.com/users/User1@org1.maakees.com/msp/signcerts/User1@org1.maakees.com-cert.pem
       key:
         path: ./fixtures/crypto-config/peerOrganizations/org1.maakees.com/users/User1@org1.maakees.com/msp/keystore/priv_sk

 # Orderer Org name
 ordererorg:
   # Membership Service Provider ID for this organization
   mspID: OrdererMSP

   # Needed to load users crypto keys and certs for this org (absolute path or relative to global crypto path, DEV mode)
   cryptoPath: ./fixtures/crypto-config/ordererOrganizations/maakees.com/users/Admin@maakees.com/msp
   # failed to create resmgmt client due to context error: user not found
   users:
     Admin:
       cert:
         path: ./fixtures/crypto-config/ordererOrganizations/maakees.com/users/Admin@maakees.com/msp/signcerts/Admin@maakees.com-cert.pem
       key:
         path: ./fixtures/crypto-config/ordererOrganizations/maakees.com/users/Admin@maakees.com/msp/keystore/priv_sk
#
# List of orderers to send transaction and channel create/update requests to. For the time
# being only one orderer is needed. If more than one is defined, which one get used by the
# SDK is implementation specific. Consult each SDK's documentation for its handling of orderers.
#
orderers:

 orderer.example.com:
   # [Optional] Default: Infer from hostname
   url: localhost:7050

   # these are standard properties defined by the gRPC library
   # they will be passed in as-is to gRPC client constructor
   grpcOptions:
     ssl-target-name-override: orderer0.maakees.com
     # These parameters should be set in coordination with the keepalive policy on the server,
     # as incompatible settings can result in closing of connection.
     # When duration of the 'keep-alive-time' is set to 0 or less the keep alive client parameters are disabled
     keep-alive-time: 0s
     keep-alive-timeout: 20s
     keep-alive-permit: false
     fail-fast: false
     # allow-insecure will be taken into consideration if address has no protocol defined, if true then grpc or else grpcs
     allow-insecure: false

   tlsCACerts:
     # Certificate location absolute path
     path: ./fixtures/crypto-config/ordererOrganizations/maakees.com/tlsca/tlsca.maakees.com-cert.pem

#
# List of peers to send various requests to, including endorsement, query
# and event listener registration.
#
peers:
 peer0.org1.maakees.com:
   # this URL is used to send endorsement and query requests
   # [Optional] Default: Infer from hostname
   url: localhost:7051
   grpcOptions:
     ssl-target-name-override: peer0.org1.maakees.com
     # These parameters should be set in coordination with the keepalive policy on the server,
     # as incompatible settings can result in closing of connection.
     # When duration of the 'keep-alive-time' is set to 0 or less the keep alive client parameters are disabled
     keep-alive-time: 0s
     keep-alive-timeout: 20s
     keep-alive-permit: false
     fail-fast: false
     # allow-insecure will be taken into consideration if address has no protocol defined, if true then grpc or else grpcs
     allow-insecure: false
   #grpcOptions:
   #  ssl-target-name-override: peer0.org1.example.com

   tlsCACerts:
     # Certificate location absolute path
     path: ./fixtures/crypto-config/peerOrganizations/org1.maakees.com/tlsca/tlsca.org1.maakees.com-cert.pem

 peer1.org1.maakees.com:
   # this URL is used to send endorsement and query requests
   # [Optional] Default: Infer from hostname
   url: peer1.org1.maakees.com:8051
   grpcOptions:
     ssl-target-name-override: peer1.org1.maakees.com
     # These parameters should be set in coordination with the keepalive policy on the server,
     # as incompatible settings can result in closing of connection.
     # When duration of the 'keep-alive-time' is set to 0 or less the keep alive client parameters are disabled
     keep-alive-time: 0s
     keep-alive-timeout: 20s
     keep-alive-permit: false
     fail-fast: false
     # allow-insecure will be taken into consideration if address has no protocol defined, if true then grpc or else grpcs
     allow-insecure: false
   #grpcOptions:
   #  ssl-target-name-override: peer0.org1.example.com

   tlsCACerts:
     # Certificate location absolute path
     path: ./fixtures/crypto-config/peerOrganizations/org1.maakees.com/tlsca/tlsca.org1.maakees.com-cert.pem

#
# Fabric-CA is a special kind of Certificate Authority provided by Hyperledger Fabric which allows
# certificate management to be done via REST APIs. Application may choose to use a standard
# Certificate Authority instead of Fabric-CA, in which case this section would not be specified.
#
entityMatchers:
 peer:
   - pattern: peer0.org1.maakees.(\w+)
     urlSubstitutionExp: localhost:7051
     sslTargetOverrideUrlSubstitutionExp: peer0.org1.maakees.com
     mappedHost: peer0.org1.maakees.com

   - pattern: peer1.org1.maakees.(\w+)
     urlSubstitutionExp: localhost:8051
     sslTargetOverrideUrlSubstitutionExp: peer1.org1.maakees.com
     mappedHost: peer1.org1.maakees.com

 orderer:
   - pattern: (\w+).maakees.(\w+)
     urlSubstitutionExp: localhost:7050
     sslTargetOverrideUrlSubstitutionExp: orderer0.maakees.com
     mappedHost: orderer0.maakees.com

 certificateAuthority:
   - pattern: (\w+).org1.maakees.(\w+)
     urlSubstitutionExp: https://localhost:7054
     sslTargetOverrideUrlSubstitutionExp: ca.org1.maakees.com
     mappedHost: ca.org1.maakees.com
```



## 3.代码示例

### 3.1创建SDK实例

configFile：配置文件路径

```go
func InitSdk(configFile string) (*fabsdk.FabricSDK) {
	// 通过config.FromFile解析配置文件，然后通过fabsdk.New创建fabric go sdk的入口实例
	sdk, err := fabsdk.New(config.FromFile(configFile))
	if err != nil {
		fmt.Println("初始化fabric sdk失败，error：%v", err)
		os.Exit(-1)
	}
	fmt.Println("Fabric sdk初始化成功!!!")
	return sdk
}
```

### 3.2 创建通道

channelID：通道名称，例：mychannel
channelConfigPath： 通道文件路径，例：./fixtures/channel-artifacts/channel.tx
orgName: 组织名称，例：org1
orgAdmin：组织管理用户，Admin

```go
func createChannel(sdk *fabsdk.FabricSDK, channelID, channelConfigPath,orgName,orgAdmin string) {
	signIds := []msp.SigningIdentity{}
	// 为组织创建msp客户端
	orgMspClient, err := mspclient.New(sdk.Context(), mspclient.WithOrg(orgName))
	if err != nil {
		fmt.Errorf("创建组织msp客户端失败，error: %v", err)
		os.Exit(-1)
	}
	// 获取组织签名信息
	signId, err := orgMspClient.GetSigningIdentity(orgAdmin)
	if err != nil {
		fmt.Errorf("GetSigningIdentity error: %v", err)
		os.Exit(-1)
	}
	signIds = append(signIds, signId)
	
	ordererClientContext := sdk.Context(fabsdk.WithUser(initInfo.OrdererAdminUser), fabsdk.WithOrg(initInfo.OrdererOrgName))
	chMgmtClient, err := resmgmt.New(ordererClientContext)
	if err != nil {
		fmt.Errorf("根据orderer节点创建通道管理客户端失败，error: %v", err)
		os.Exit(-1)
	}
	// 保存通道请求的参数
	chreq := resmgmt.SaveChannelRequest{
		ChannelID:         channelID,
		ChannelConfigPath: channelConfigPath,
		SigningIdentities: signIds,
	}
	// 创建通道
	_, err := chMgmtClient.SaveChannel(chreq)
	if err != nil {
		fmt.Errorf("SaveChannel error: %v", err)
		os.Exit(-1)
	}
	fmt.Println("创建通道成功!!!")
}

```

### 3.3 加入通道

channelID：通道名称，例：mychannel
orgName: 组织名称，例：org1
orgAdmin：组织用户，Admin
ordererEndpoint：排序节点，例：orderer0.example.com

```go
func joinChannel(sdk *fabsdk.FabricSDK, channelID, orgAdmin, orgName, ordererEndpoint string) {
	clientContext := sdk.Context(fabsdk.WithUser(orgAdmin), fabsdk.WithOrg(orgName))
	// 根据组织创建通道管理客户端实例
	orgResMgmt, err := resmgmt.New(clientContext)
	if err != nil {
		fmt.Errorf("创建通道管理客户端失败，error: %v", err)
		os.Exit(-1)
	}
	if err = orgResMgmt.JoinChannel(channelID, resmgmt.WithRetry(retry.DefaultResMgmtOpts), resmgmt.WithOrdererEndpoint(ordererEndpoint)); err != nil {
		fmt.Errorf("%s组织加入通道失败，error: %v", orgName, err)
		os.Exit(-1)
	}
	fmt.Println("组织加入通道成功")
}

```



### 3.4 打包链码

ccName：链码名称
ccVersion：链码版本号
ccPath：链码路径

```go
func packageCC(ccName, ccVersion, ccPath string) (string, []byte) {
	label := ccName + "_" + ccVersion
	desc := &lifecycle.Descriptor{
		Path:  ccPath,
		Type:  peer.ChaincodeSpec_GOLANG,
		Label: label,
	}
	ccPkg, err := lifecycle.NewCCPackage(desc)
	if err != nil {
		fmt.Errorf("packageCC error: %v", err)
		os.Exit(-1)
	}
	return label, ccPkg
}

```

### 3.5 安装链码

sdk： SDK实例
label：链码名称
ccPkg：安装包
orgAdmin：组织管理员
orgName：组织名称

```go
func installCC(sdk *fabsdk.FabricSDK, label string, ccPkg []byte, orgAdmin, orgName string)  {
	installCCReq := resmgmt.LifecycleInstallCCRequest{
		Label:   label,
		Package: ccPkg,
	}
	clientContext := sdk.Context(fabsdk.WithUser(orgAdmin), fabsdk.WithOrg(orgName))
	orgResMgmt, _ := resmgmt.New(clientContext)
	if _,err := orgResMgmt.LifecycleInstallCC(installCCReq, resmgmt.WithRetry(retry.DefaultResMgmtOpts)); err != nil {
		fmt.Errorf("LifecycleInstallCC error: %v", err)
		os.Exit(-1)
	}
}
```

### 3.6 查询安装的链码

orgResMgmt：根据组织创建的资源管理客户端

```go
func queryInstalledCC(orgResMgmt *resmgmt.Client) {
	installedCC, err := orgResMgmt.LifecycleQueryInstalledCC()
	if err != nil {
		fmt.Errorf("LifecycleQueryInstalledCC error: %v", err)
		os.Exit(-1)
	}
	for _, info := range installedCC {
		fmt.Printf("label：%s, packageID：%s",info.Label, info.PackageID)
	}
}
```

### 3.7 根据packageID获取链码包

packageID：链码包ID
orgResMgmt：根据组织创建的资源管理客户端

```go
func getInstalledCCPackage(packageID string, orgResMgmt *resmgmt.Client) {
	if _, err := orgResMgmt.LifecycleGetInstalledCCPackage(packageID); err != nil {
		fmt.Errorf("LifecycleGetInstalledCCPackage error: %v", err)
		os.Exit(-1)
	}
	fmt.Println("LifecycleGetInstalledCCPackage success")
}

```

### 3.8 批准链码定义

sdk：SDK实例
packageID：链码安装包ID
ccName：链码名称
ccVersion：链码版本号
sequence：链码序列号
channelID：通道名称
orgAdmin：组织管理员
orgName：组织名称
orgMspID：组织MspID
ordererEndpoint： order节点地址

```go
func approveCC(sdk *fabsdk.FabricSDK, packageID, ccName, ccVersion string, sequence int64, channelID string, orgAdmin, orgName, orgMspID, ordererEndpoint string) {
	mspIDs := []string{}
	mspIDs = append(mspIDs, orgMspID)
	ccPolicy := policydsl.SignedByNOutOfGivenRole(int32(len(mspIDs)), mb.MSPRole_MEMBER, mspIDs)
	approveCCReq := resmgmt.LifecycleApproveCCRequest{
		Name:              ccName,
		Version:           ccVersion,
		PackageID:         packageID,
		Sequence:          sequence,
		EndorsementPlugin: "escc",
		ValidationPlugin:  "vscc",
		SignaturePolicy:   ccPolicy,
		InitRequired:      true,
	}
	orgClientContext := sdk.Context(fabsdk.WithUser(orgAdmin), fabsdk.WithOrg(orgName))
	ctx, err := context.NewLocal(orgClientContext)
	orgPeers, err := ctx.LocalDiscoveryService().GetPeers()
	if err != nil {
		fmt.Errorf("GetPeers error: %v", err)
		os.Exit(-1)
	}
	orgResMgmt,_ := resmgmt.New(orgClientContext)
	if _, err := orgResMgmt.LifecycleApproveCC(channelID, approveCCReq, resmgmt.WithTargets(orgPeers...), resmgmt.WithOrdererEndpoint(ordererEndpoint), resmgmt.WithRetry(retry.DefaultResMgmtOpts)); err != nil {
		fmt.Errorf("LifecycleApproveCC error: %v", err)
		os.Exit(-1)
	}
	fmt.Errorf("LifecycleApproveCC success")
}

```

### 3.9 查询批准的链码

sdk：SDK实例
ccName：链码名称
sequence：序列号
channelID：通道ID
orgAdmin：组织管理员
orgName：组织名称

```go
func queryApprovedCC(sdk *fabsdk.FabricSDK,ccName string, sequence int64, channelID string, orgAdmin, orgName string) {
	queryApprovedCCReq := resmgmt.LifecycleQueryApprovedCCRequest{
		Name:     ccName,
		Sequence: sequence,
	}
	orgClientContext := sdk.Context(fabsdk.WithUser(orgAdmin), fabsdk.WithOrg(orgName))
	ctx, _ := context.NewLocal(orgClientContext)
	orgResMgmt,_ := resmgmt.New(orgClientContext)
	orgPeers, _ := ctx.LocalDiscoveryService().GetPeers()
	for _, p := range orgPeers {
		resp, err := orgResMgmt.LifecycleQueryApprovedCC(channelID, queryApprovedCCReq, resmgmt.WithTargets(p))
		if err != nil {
			fmt.Errorf("LifecycleQueryApprovedCC returned error: %v", err)
		}
		fmt.Println(resp.Name)
	}
}

```

### 3.10 检查链码是否已准备好在通道上提交

sdk：SDK实例
ccName：链码名称
ccVersion：链码版本号
sequence：链码序列号
channelID：通道名称
orgAdmin：组织管理员
orgName：组织名称
orgMspID：组织MspID

```go
func checkCCCommitReadiness(sdk *fabsdk.FabricSDK,ccName, ccVersion string, sequence int64, channelID string, orgAdmin, orgName, orgMspID string) {
	mspIds := []string{"orgMspID"}
	ccPolicy := policydsl.SignedByNOutOfGivenRole(int32(len(mspIds)), mb.MSPRole_MEMBER, mspIds)
	req := resmgmt.LifecycleCheckCCCommitReadinessRequest{
		Name:    ccName,
		Version: ccVersion,
		EndorsementPlugin: "escc",
		ValidationPlugin:  "vscc",
		SignaturePolicy:   ccPolicy,
		Sequence:          sequence,
		InitRequired:      true,
	}
	orgClientContext := sdk.Context(fabsdk.WithUser(orgAdmin), fabsdk.WithOrg(orgName))
	ctx, _ := context.NewLocal(orgClientContext)
	orgResMgmt,_ := resmgmt.New(orgClientContext)
	orgPeers, _ := ctx.LocalDiscoveryService().GetPeers()
	for _, p := range orgPeers {
		resp, err := orgResMgmt.LifecycleCheckCCCommitReadiness(channelID, req, resmgmt.WithTargets(p))
		if err != nil {
			fmt.Errorf("LifecycleCheckCCCommitReadiness error: %v", err)
			os.Exit(-1)
		}
		for _, r := range resp.Approvals {
			if !r {
				fmt.Errorf("链码未就绪")
				os.Exit(-1)
			}
		}
	}
	fmt.Errorf("LifecycleCheckCCCommitReadiness success")
}
```

### 3.11 提交链码定义到通道

sdk：SDK实例
ccName：链码名称
ccVersion：链码版本号
sequence：链码序列号
channelID：通道名称
orgAdmin：组织管理员
orgName：组织名称
orgMspID：组织MspID

```go
func commitCC(ccName, ccVersion string, sequence int64, channelID string, orgs appConf.OrgsConfig, orgAdmin, orgName, orgMspID, ordererEndpoint string) {
	mspIDs := []string{"orgMspID"}
	ccPolicy := policydsl.SignedByNOutOfGivenRole(int32(len(mspIDs)), mb.MSPRole_MEMBER, mspIDs)

	req := resmgmt.LifecycleCommitCCRequest{
		Name:              ccName,
		Version:           ccVersion,
		Sequence:          sequence,
		EndorsementPlugin: "escc",
		ValidationPlugin:  "vscc",
		SignaturePolicy:   ccPolicy,
		InitRequired:      true,
	}
	
	orgClientContext := sdk.Context(fabsdk.WithUser(orgAdmin), fabsdk.WithOrg(orgName))
	orgResMgmt,_ := resmgmt.New(orgClientContext)
	_, err := orgResMgmt.LifecycleCommitCC(channelID, req, resmgmt.WithOrdererEndpoint(ordererEndpoint), resmgmt.WithRetry(retry.DefaultResMgmtOpts))
	if err != nil {
		fmt.Errorf("LifecycleCommitCC error: %v", err)
		os.Exit(-1)
	}
	fmt.Errorf("LifecycleCommitCC success")
}

```

### 3.12 初始化链码

ccName：链码名称
orgName：组织名称
orgUser：组织用户

```go
func InitCC(ccName,orgName,orgUser string) {
	clientChannelContext := sdk.ChannelContext(channelID, fabsdk.WithUser(orgUser), fabsdk.WithOrg(orgName))
	client, err := channel.New(clientChannelContext)
	_, err := client.Execute(channel.Request{ChaincodeID: ccName, Fcn: "Init", Args: nil, IsInit: true},
		channel.WithRetry(retry.DefaultChannelOpts))
	if err != nil {
		fmt.Errorf("Failed to initcc: %s", err)
		os.Exit(-1)
	}
	fmt.Println("合约初始化成功 !!!")
}

```



### 3.12 链码升级

升级链码与部署链码的生命周期相同：

重新打包链码
在peer 节点上安装新的链码包
批准新的链码定义
提交链码定义到通道
Fabric 链码在链码定义中使用 sequence 来跟踪升级。所有的通道成员需要把序列号加一并且同意新的定义来升级链码。

————————————————
版权声明：本文为CSDN博主「余生·情诗」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/qq_28052455/article/details/126819799



# *HyperLedger Fabric* 实战—反欺诈系统

注意点
1、用户唯一ID应该是姓名与身份证号组合后的MD5，这样可以防止加盟用户仅通过机器生成的大量身份证号来恶意刷区块链平台；

2、合约不应以用户为核心，而应以用户及小贷公司的合同作为合约核心，以防止加盟方自身的Bug等问题，向联盟平台写入大量无法溯源的用户数据，而导致错误；











————————————————
版权声明：本文为CSDN博主「探路人」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/xuguangyuansh/article/details/83269833





# *Hyperledger Fabric* 2.5 单机部署多节点网络


本博客主要记录搭建一个3个排序节点、4个组织的每个组织各2个节点的fabric区块链网络

单机部署多节点网络
1、相关环境的安装与配置
2、生成相关的证书文件
3、生成相关的通道配置文件
4、生成docker-compose.yaml的启动文件
5、通道相关配置操作
6、链码调用
1、相关环境的安装与配置
官方文档
安装Git
安装CUrl
安装Go
安装Docker
安装Docker-compose
go、docker、docker-compose，fabric的相关docker镜像、官方的fabric-samples文件、相关fabric二进制文件，具体配置之后补充
————————————————
版权声明：本文为CSDN博主「ccccczy_」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/qq_45469783/article/details/119298677







# 从0开始搭建Hyperledger Fabric 2.x(fabric2.5版本)





————————————————
版权声明：本文为CSDN博主「xcSpark」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/u010895512/article/details/131095387



# 基于Hyperledger Fabric的学位学历认证管理系统

项目源码：https://github.com/Pistachiout/Academic-Degree-BlockChain

## 一、选题背景

学历造假、认证造假等是一个全球日益普遍的现象，不仅对社会产生了巨大的负面影响，同时也极大增加了企业和单位的用人成本，造成了无谓的经济消耗；从另一个角度来说，纸质或电子版的证书无论从保存便利性、验证可靠性、可信性等方面，都存在着很大的不足。一种高度可靠、易保存、易证伪同时还顾全隐私保护的学位学历认证管理系统，是一个必然的需求。
区块链是非中心化信任网络，适合作为此类分布式应用的底层架构和基础工具。区块链具有两大核心特点：一是数据难以篡改、二是去中心化。区块链所记录的信息更加真实可靠，可以帮助解决人们互不信任的问题。

## 二、功能分析

由于系统需要保证人才受教育情况真实性，所以对于系统的用户而言，不可能由用户自己添加相应的学历信息，而是由具有一定权限的用户来完成添加或修改的功能。但普通用户可以通过系统溯源功能来确定信息的真伪。所以我们将系统用户的使用角色分为两种：

普通用户
管理员用户
普通用户具有对数据的查询功能 ，但实现查询之前必须经过登录认证：

用户登录：系统只针对合法用户进行授权使用，所以用户必须先进行登录才能完成相应的功能。
查询实现：查询分为两种方式实现
根据证书编号与姓名查询：根据用户输入的证书编号与姓名进行查询。
根据身份证号码查询：根据用户输入指定的身份证号码进行查询，此功能可以实现溯源。
管理员用户除具有普通用户的功能之外，额外添加了两个功能：

添加信息：可以向系统中添加新的学历信息。
修改信息：针对已存在的学历信息进行修改。
最后需要达到的要求

认证颁发时的数据生成和上链流程；
认证验证时的验证方法：核实电子证书文件的哈希值；
用户界面：证书生成、证书上链、证书查询、证书核验；
安全和隐私保护。

————————————————
版权声明：本文为CSDN博主「Pistachiout」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/qq_45808700/article/details/129423030



# *Hyperledger Fabric*使用私有数据的官方教程

本教程将演示如何使用私有数据收集 （PDC） 提供存储以及为授权的对等方（Peer）检索区块链网络上的私人数据的组织。集合是使用包含管理该集合的策略的集合定义文件指定的。

本教程中的信息假定了解私有数据 商店及其用例。有关详细信息，请查看[私有数据](https://hyperledger-fabric.readthedocs.io/en/release-2.5/private-data/private-data.html)。

注意

这些指令使用引入的新 Fabric 链码生命周期 在结构 v2.0 版本中。如果您想使用以前的 生命周期模型 要将私有数据与链码一起使用，请访问 v1.4 在[结构中使用私有数据教程](https://hyperledger-fabric.readthedocs.io/en/release-1.4/private_data_tutorial.html)的版本。

本教程将引导您完成以下步骤来练习定义， 通过 Fabric 配置和使用私有数据：

1. [资产转移私有数据示例用例](https://hyperledger-fabric.readthedocs.io/en/release-2.5/private_data_tutorial.html#pd-use-case)
2. [生成集合定义 JSON 文件](https://hyperledger-fabric.readthedocs.io/en/release-2.5/private_data_tutorial.html#pd-build-json)
3. [使用链码 API 读取和写入私有数据](https://hyperledger-fabric.readthedocs.io/en/release-2.5/private_data_tutorial.html#pd-read-write-private-data)
4. [将私有数据智能合约部署到通道](https://hyperledger-fabric.readthedocs.io/en/release-2.5/private_data_tutorial.html#pd-install-define-cc)
5. [注册身份](https://hyperledger-fabric.readthedocs.io/en/release-2.5/private_data_tutorial.html#pd-register-identities)
6. [在私有数据中创建资产](https://hyperledger-fabric.readthedocs.io/en/release-2.5/private_data_tutorial.html#pd-store-private-data)
7. [以授权对等方身份查询私有数据](https://hyperledger-fabric.readthedocs.io/en/release-2.5/private_data_tutorial.html#pd-query-authorized)
8. [将私有数据作为未经授权的对等方进行查询](https://hyperledger-fabric.readthedocs.io/en/release-2.5/private_data_tutorial.html#pd-query-unauthorized)
9. [转移资产](https://hyperledger-fabric.readthedocs.io/en/release-2.5/private_data_tutorial.html#pd-transfer-asset)
10. [清除私有数据](https://hyperledger-fabric.readthedocs.io/en/release-2.5/private_data_tutorial.html#pd-purge)
11. [对私有数据使用索引](https://hyperledger-fabric.readthedocs.io/en/release-2.5/private_data_tutorial.html#pd-indexes)
12. [其他资源](https://hyperledger-fabric.readthedocs.io/en/release-2.5/private_data_tutorial.html#pd-ref-material)

本教程将资产[传输私有数据示例](https://github.com/hyperledger/fabric-samples/tree/main/asset-transfer-private-data/chaincode-go)部署到 Fabric 测试网络，以演示如何创建、部署和使用 私人数据。 您应该已完成[安装结构和结构示例](https://hyperledger-fabric.readthedocs.io/en/release-2.5/install.html)任务。



## 资产转移私有数据示例用例

此示例演示如何使用三个私有数据收集，以及使用以下用例在 Org1 和 Org2 之间传输资产：`assetCollection``Org1MSPPrivateCollection``Org2MSPPrivateCollection`

Org1 的成员创建一个新资产，此后称为所有者。资产的公开详细信息， 包括所有者的身份，都存储在名为 的私有数据集合中。资产也是使用评估的 所有者提供的值。评估值由每个参与者用于同意资产的转移，并且仅存储在所有者组织的集合中。在我们的例子中，业主同意的初始评估值存储在.`assetCollection``Org1MSPPrivateCollection`

要购买资产，买方需要同意与 资产所有者。在此步骤中，买方（Org2 的成员）创建协议 使用智能合约功能进行交易并同意评估价值。 此值存储在集合中。现在，资产 所有者可以使用智能合约功能将资产转让给买方。 该函数使用通道账本上的哈希值来 确认业主和买家已同意相同的评估价值 在转移资产之前。`'AgreeToTransfer'``Org2MSPPrivateCollection``'TransferAsset'``'TransferAsset'`

在介绍传输方案之前，我们将讨论组织如何在 Fabric 中使用私有数据收集。



## 生成集合定义 JSON 文件

在一组组织可以使用私有数据进行交易之前，所有组织 在通道上需要构建定义私有的集合定义文件 与每个链码关联的数据集合。存储在私有中的数据 数据收集仅分发给某些组织的同行 频道的所有成员。集合定义文件描述了所有 组织可以从链码读取和写入的私有数据集合。

每个集合由以下属性定义：

- `name`：集合的名称。
- `policy`：定义允许保留集合数据的组织对等方。
- `requiredPeerCount`：将私有数据传播为 链码背书的条件
- `maxPeerCount`：出于数据冗余目的，其他对等体的数量 当前认可对等方将尝试将数据分发到的。 如果认可对等方关闭，则这些其他对等方在提交时可用 如果有请求拉取私有数据。
- `blockToLive`：对于非常敏感的信息，例如定价或个人信息， 此值表示数据应在专用数据库中保留多长时间 的块。数据将存在于专用数据库上此指定数量的块中 之后它将被清除，使这些数据从网络中过时。 要无限期保留私有数据，即从不清除私有数据，请将 的属性到 。`blockToLive``0`
- `memberOnlyRead`：值 表示对等方自动 强制仅属于集合成员组织之一的客户端 允许对私有数据进行读取访问。`true`
- `memberOnlyWrite`：值 表示对等方自动 强制仅属于集合成员组织之一的客户端 允许对私有数据进行写入访问。`true`
- `endorsementPolicy`：定义需要满足的背书策略 以便写入私有数据收集。集合级别认可策略 覆盖链码级策略。有关构建策略的详细信息 定义请参阅[背书策略](https://hyperledger-fabric.readthedocs.io/en/release-2.5/endorsement-policies.html)主题。

所有组织都需要部署相同的集合定义文件 使用链码，即使组织不属于任何集合。在 除了在集合文件中显式定义的集合之外， 每个组织都可以访问其对等方上的隐式集合，该集合只能 由他们的组织阅读。对于使用隐式数据收集的示例， 请参阅[结构中的安全资产转移](https://hyperledger-fabric.readthedocs.io/en/release-2.5/secured_asset_transfer/secured_private_asset_transfer_tutorial.html)。

资产传输私有数据示例包含一个 collections_config.json 文件 定义了三个私有数据收集定义：、、 和。`assetCollection``Org1MSPPrivateCollection``Org2MSPPrivateCollection`

```
// collections_config.json

[
   {
   "name": "assetCollection",
   "policy": "OR('Org1MSP.member', 'Org2MSP.member')",
   "requiredPeerCount": 1,
   "maxPeerCount": 1,
   "blockToLive":1000000,
   "memberOnlyRead": true,
   "memberOnlyWrite": true
   },
   {
   "name": "Org1MSPPrivateCollection",
   "policy": "OR('Org1MSP.member')",
   "requiredPeerCount": 0,
   "maxPeerCount": 1,
   "blockToLive":3,
   "memberOnlyRead": true,
   "memberOnlyWrite": false,
   "endorsementPolicy": {
       "signaturePolicy": "OR('Org1MSP.member')"
   }
   },
   {
   "name": "Org2MSPPrivateCollection",
   "policy": "OR('Org2MSP.member')",
   "requiredPeerCount": 0,
   "maxPeerCount": 1,
   "blockToLive":3,
   "memberOnlyRead": true,
   "memberOnlyWrite": false,
   "endorsementPolicy": {
       "signaturePolicy": "OR('Org2MSP.member')"
   }
   }
]
```

定义中的属性指定两者 Org1 和 Org2 可以将集合存储在其对等方上。和参数用于指定只有 Org1 和 Org2 客户端可以读取和写入此集合。`policy``assetCollection``memberOnlyRead``memberOnlyWrite`

该集合仅允许 Org1 的对等方具有 私有数据在其私有数据库中，而集合只能由 Org2 的对等方存储。参数 用于创建特定于集合的认可策略。每次更新或需要认可 由将集合存储在其对等方上的组织提供。我们将看到如何 这些集合用于在本教程过程中传输资产。`Org1MSPPrivateCollection``Org2MSPPrivateCollection``endorsementPolicy``Org1MSPPrivateCollection``Org2MSPPrivateCollection`

当链码定义为 使用 [Peer 生命周期链码提交命令提交](https://hyperledger-fabric.readthedocs.io/en/release-2.5/commands/peerlifecycle.html#peer-lifecycle-chaincode-commit)到通道。 有关此过程的更多详细信息，请参见下文第 3 节。



## 使用链码 API 读取和写入私有数据

了解如何将渠道上的数据私有化的下一步是构建 链码中的数据定义。资产转移私有数据示例划分 私有数据根据数据将如何分为三个单独的数据定义 被访问。

```go
// Peers in Org1 and Org2 will have this private data in a side database
type Asset struct {
       Type  string `json:"objectType"` //Type is used to distinguish the various types of objects in state database
       ID    string `json:"assetID"`
       Color string `json:"color"`
       Size  int    `json:"size"`
       Owner string `json:"owner"`
}

// AssetPrivateDetails describes details that are private to owners

// Only peers in Org1 will have this private data in a side database
type AssetPrivateDetails struct {
       ID             string `json:"assetID"`
       AppraisedValue int    `json:"appraisedValue"`
}

// Only peers in Org2 will have this private data in a side database
type AssetPrivateDetails struct {
       ID             string `json:"assetID"`
       AppraisedValue int    `json:"appraisedValue"`
}
```

具体而言，对私人数据的访问将受到限制，如下所示：

- `objectType, color, size, and owner`存储在中，因此根据收集策略（Org1 和 Org2）中的定义，通道成员可见。`assetCollection`
- `AppraisedValue`的资产存储在集合或 中，具体取决于资产的所有者。只有属于可以存储集合的组织的用户才能访问该值。`Org1MSPPrivateCollection``Org2MSPPrivateCollection`

资产传输私有数据示例创建的所有数据智能 合约存储在 PDC 中。智能合约使用结构链码API 使用 和 函数将私有数据读取和写入私有数据集合。您可以[在此处](https://godoc.org/github.com/hyperledger/fabric-chaincode-go/shim#ChaincodeStub)找到有关这些功能的更多信息。 此私有数据存储在对等方的私有状态数据库中（与公共状态数据库分开），并且 通过八卦协议在授权的同行之间传播。`GetPrivateData()``PutPrivateData()`

下图说明了私有数据使用的私有数据模型 样本。请注意，图中仅显示 Org3 以说明如果 通道上还有任何其他组织，他们无权访问配置中定义*的任何*私有数据收集。

![_images/SideDB-org1-org2.png](https://hyperledger-fabric.readthedocs.io/en/release-2.5/_images/SideDB-org1-org2.png)

### 读取集合数据

智能合约使用链码 API 查询 数据库。 采用两个参数，即**集合名称和**数据键。召回集合允许同行 Org1 和 Org2 将私有数据保存在端数据库中，并且该集合仅允许 Org1 的对等方拥有其 端数据库中的私有数据并允许对等方 的 Org2 将其私有数据保存在端数据库中。 有关实现详细信息，请参阅以下两个[资产转移私有数据功能](https://github.com/hyperledger/fabric-samples/blob/main/asset-transfer-private-data/chaincode-go/chaincode/asset_queries.go)：`GetPrivateData()``GetPrivateData()``assetCollection``Org1MSPPrivateCollection``Org2MSPPrivateCollection`

> - 用于查询属性值的**读取资产**。`assetID, color, size and owner`
> - **ReadAssetPrivateDetails** 用于查询属性的值。`appraisedValue`

当我们在本教程后面使用对等命令发出数据库查询时， 我们将调用这两个函数。

### 写入私有数据

智能合约使用链码API来存储私有数据 到私有数据库中。API 还需要集合的名称。 请注意，资产传输私有数据示例包括三个不同的私有数据收集，但它称为 链码中两次（在这种情况下充当 Org1）。`PutPrivateData()`

1. 使用 名为 的集合。`assetID, color, size and owner``assetCollection`
2. 使用名为 的集合写入私有数据。`appraisedValue``Org1MSPPrivateCollection`

如果我们充当 Org2，我们将替换为 .`Org1MSPPrivateCollection``Org2MSPPrivateCollection`

例如，在函数的以下代码段中，调用两次，每组私有数据调用一次。`CreateAsset``PutPrivateData()`

```
// CreateAsset creates a new asset by placing the main asset details in the assetCollection
// that can be read by both organizations. The appraisal value is stored in the owners org specific collection.
func (s *SmartContract) CreateAsset(ctx contractapi.TransactionContextInterface) error {

    // Get new asset from transient map
    transientMap, err := ctx.GetStub().GetTransient()
    if err != nil {
        return fmt.Errorf("error getting transient: %v", err)
    }

    // Asset properties are private, therefore they get passed in transient field, instead of func args
    transientAssetJSON, ok := transientMap["asset_properties"]
    if !ok {
        //log error to stdout
        return fmt.Errorf("asset not found in the transient map input")
    }

    type assetTransientInput struct {
        Type           string `json:"objectType"` //Type is used to distinguish the various types of objects in state database
        ID             string `json:"assetID"`
        Color          string `json:"color"`
        Size           int    `json:"size"`
        AppraisedValue int    `json:"appraisedValue"`
    }

    var assetInput assetTransientInput
    err = json.Unmarshal(transientAssetJSON, &assetInput)
    if err != nil {
        return fmt.Errorf("failed to unmarshal JSON: %v", err)
    }

    if len(assetInput.Type) == 0 {
        return fmt.Errorf("objectType field must be a non-empty string")
    }
    if len(assetInput.ID) == 0 {
        return fmt.Errorf("assetID field must be a non-empty string")
    }
    if len(assetInput.Color) == 0 {
        return fmt.Errorf("color field must be a non-empty string")
    }
    if assetInput.Size <= 0 {
        return fmt.Errorf("size field must be a positive integer")
    }
    if assetInput.AppraisedValue <= 0 {
        return fmt.Errorf("appraisedValue field must be a positive integer")
    }

    // Check if asset already exists
    assetAsBytes, err := ctx.GetStub().GetPrivateData(assetCollection, assetInput.ID)
    if err != nil {
        return fmt.Errorf("failed to get asset: %v", err)
    } else if assetAsBytes != nil {
        fmt.Println("Asset already exists: " + assetInput.ID)
        return fmt.Errorf("this asset already exists: " + assetInput.ID)
    }

    // Get ID of submitting client identity
    clientID, err := submittingClientIdentity(ctx)
    if err != nil {
        return err
    }

    // Verify that the client is submitting request to peer in their organization
    // This is to ensure that a client from another org doesn't attempt to read or
    // write private data from this peer.
    err = verifyClientOrgMatchesPeerOrg(ctx)
    if err != nil {
        return fmt.Errorf("CreateAsset cannot be performed: Error %v", err)
    }

    // Make submitting client the owner
    asset := Asset{
        Type:  assetInput.Type,
        ID:    assetInput.ID,
        Color: assetInput.Color,
        Size:  assetInput.Size,
        Owner: clientID,
    }
    assetJSONasBytes, err := json.Marshal(asset)
    if err != nil {
        return fmt.Errorf("failed to marshal asset into JSON: %v", err)
    }

    // Save asset to private data collection
    // Typical logger, logs to stdout/file in the fabric managed docker container, running this chaincode
    // Look for container name like dev-peer0.org1.example.com-{chaincodename_version}-xyz
    log.Printf("CreateAsset Put: collection %v, ID %v, owner %v", assetCollection, assetInput.ID, clientID)

    err = ctx.GetStub().PutPrivateData(assetCollection, assetInput.ID, assetJSONasBytes)
    if err != nil {
        return fmt.Errorf("failed to put asset into private data collection: %v", err)
    }

    // Save asset details to collection visible to owning organization
    assetPrivateDetails := AssetPrivateDetails{
        ID:             assetInput.ID,
        AppraisedValue: assetInput.AppraisedValue,
    }

    assetPrivateDetailsAsBytes, err := json.Marshal(assetPrivateDetails) // marshal asset details to JSON
    if err != nil {
        return fmt.Errorf("failed to marshal into JSON: %v", err)
    }

    // Get collection name for this organization.
    orgCollection, err := getCollectionName(ctx)
    if err != nil {
        return fmt.Errorf("failed to infer private collection name for the org: %v", err)
    }

    // Put asset appraised value into owners org specific private data collection
    log.Printf("Put: collection %v, ID %v", orgCollection, assetInput.ID)
    err = ctx.GetStub().PutPrivateData(orgCollection, assetInput.ID, assetPrivateDetailsAsBytes)
    if err != nil {
        return fmt.Errorf("failed to put asset private details: %v", err)
    }
    return nil
}
```

总而言之，上面的策略定义允许 Org1 和 Org2 中的所有对等方存储和交易 随着资产转移私人数据在其 私有数据库。但只有 Org1 中的对等方才能存储和处理 Org1 集合中的关键数据，并且仅对等方 在 Org2 中可以存储和处理 Org2 集合中的关键数据。`collections_config.json``assetID, color, size, owner``appraisedValue``Org1MSPPrivateCollection``appraisedValue``Org2MSPPrivateCollection`

作为额外的数据隐私优势，由于正在使用集合， 只有私有数据*哈希*通过排序器，而不是私有数据本身， 对订购者保密私人数据。

## 启动网络

现在我们准备逐步完成一些演示如何使用的命令 私人数据。

**自己尝试一下**

在安装、定义和使用私有数据智能合约之前， 我们需要启动结构测试网络。为了本教程，我们希望 从已知的初始状态进行操作。以下命令将杀死任何活动 或过时的 Docker 容器并删除以前生成的项目。 因此，让我们运行以下命令来清理任何以前的 环境：

```
cd fabric-samples/test-network
./network.sh down
```

从目录中，您可以使用以下命令启动 使用证书颁发机构和 CouchDB 建立结构测试网络：`test-network`

```
./network.sh up createChannel -ca -s couchdb
```

此命令将部署一个 Fabric 网络，该网络由一个通道组成，该通道由两个组织（每个组织维护一个对等节点）、证书颁发机构和 在使用 CouchDB 作为状态数据库时对服务进行排序。要么是 LevelDB 要么 CouchDB 可以与集合一起使用。选择CouchDB来演示如何 将索引与私有数据一起使用。`mychannel`

注意

要使集合正常工作，跨组织很重要 八卦配置正确。请参阅我们关于[八卦数据传播协议](https://hyperledger-fabric.readthedocs.io/en/release-2.5/gossip.html)的文档， 特别注意“锚点同行”部分。我们的教程 不关注八卦，因为它已经在测试网络中配置， 但是在配置频道时，八卦主播对等点至关重要 配置集合以正常工作。



## 将私有数据智能合约部署到通道

我们现在可以使用测试网络脚本将智能合约部署到通道。 从测试网络目录运行以下命令。

```
./network.sh deployCC -ccn private -ccp ../asset-transfer-private-data/chaincode-go/ -ccl go -ccep "OR('Org1MSP.peer','Org2MSP.peer')" -cccg ../asset-transfer-private-data/chaincode-go/collections_config.json
```

请注意，我们需要传递私有数据收集定义文件的路径 到命令。作为将链码部署到通道的一部分，两个组织 在通道上必须传递相同的专用数据收集定义作为一部分 结构[链码生命周期](https://hyperledger-fabric.readthedocs.io/en/release-2.5/chaincode_lifecycle.html)。我们还在部署智能合约 链码级背书策略为 . 这允许 Org1 和 Org2 创建资产，而无需获得来自 另一个组织。您可以看到部署链码所需的步骤 发出上述命令后打印在日志中。`"OR('Org1MSP.peer','Org2MSP.peer')"`

当两个组织都使用 [peer 生命周期链码 approveformyorg](https://hyperledger-fabric.readthedocs.io/en/release-2.5/commands/peerlifecycle.html#peer-lifecycle-chaincode-approveformyorg) 命令批准链码定义时，链码定义包括私有数据收集的路径 使用标志的定义。您可以在终端中看到打印的以下 approveformyorg 命令：`--collections-config`

```
peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name private --version 1.0 --collections-config ../asset-transfer-private-data/chaincode-go/collections_config.json --signature-policy "OR('Org1MSP.member','Org2MSP.member')" --package-id $CC_PACKAGE_ID --sequence 1 --tls --cafile $ORDERER_CA
```

在通道成员同意将私有数据收集作为链码的一部分之后 定义，数据收集使用[对等生命周期链码](https://hyperledger-fabric.readthedocs.io/en/release-2.5/commands/peerlifecycle.html#peer-lifecycle-chaincode-commit)提交命令提交到通道。如果在日志中查找 commit 命令，可以看到它使用 用于提供集合定义的路径的同一标志。`--collections-config`

```
peer lifecycle chaincode commit -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --channelID mychannel --name private --version 1.0 --sequence 1 --collections-config ../asset-transfer-private-data/chaincode-go/collections_config.json --signature-policy "OR('Org1MSP.member','Org2MSP.member')" --tls --cafile $ORDERER_CA --peerAddresses localhost:7051 --tlsRootCertFiles $ORG1_CA --peerAddresses localhost:9051 --tlsRootCertFiles $ORG2_CA
```



## 注册身份

私有数据传输智能合约支持属于网络的个人身份的所有权。在我们的场景中，资产的所有者将是 Org1 的成员，而买方将属于 Org2。为了突出显示 API 与用户证书中的信息之间的联系，我们将使用 Org1 和 Org2 证书颁发机构 （CA） 注册两个新身份，然后使用 CA 生成每个身份的证书和私钥。`GetClientIdentity().GetID()`

首先，我们需要设置以下环境变量以使用 Fabric CA 客户端：

```
export PATH=${PWD}/../bin:${PWD}:$PATH
export FABRIC_CFG_PATH=$PWD/../config/
```

我们将使用 Org1 CA 创建标识资产所有者。将结构 CA 客户端主页设置为 Org1 CA 管理员的 MSP（此标识由测试网络脚本生成）：

```
export FABRIC_CA_CLIENT_HOME=${PWD}/organizations/peerOrganizations/org1.example.com/
```

您可以使用 fabric-ca-client 工具注册新的所有者客户端标识：

```
fabric-ca-client register --caname ca-org1 --id.name owner --id.secret ownerpw --id.type client --tls.certfiles "${PWD}/organizations/fabric-ca/org1/tls-cert.pem"
```

现在，您可以通过向 enroll 命令提供注册名称和密钥来生成身份证书和 MSP 文件夹：

```
fabric-ca-client enroll -u https://owner:ownerpw@localhost:7054 --caname ca-org1 -M "${PWD}/organizations/peerOrganizations/org1.example.com/users/owner@org1.example.com/msp" --tls.certfiles "${PWD}/organizations/fabric-ca/org1/tls-cert.pem"
```

运行以下命令，将节点 OU 配置文件复制到所有者身份 MSP 文件夹中。

```
cp "${PWD}/organizations/peerOrganizations/org1.example.com/msp/config.yaml" "${PWD}/organizations/peerOrganizations/org1.example.com/users/owner@org1.example.com/msp/config.yaml"
```

现在，我们可以使用 Org2 CA 创建买方身份。将结构 CA 客户端主页设置为 Org2 CA 管理员：

```
export FABRIC_CA_CLIENT_HOME=${PWD}/organizations/peerOrganizations/org2.example.com/
```

您可以使用 fabric-ca-client 工具注册新的所有者客户端标识：

```
fabric-ca-client register --caname ca-org2 --id.name buyer --id.secret buyerpw --id.type client --tls.certfiles "${PWD}/organizations/fabric-ca/org2/tls-cert.pem"
```

我们现在可以注册以生成标识 MSP 文件夹：

```
fabric-ca-client enroll -u https://buyer:buyerpw@localhost:8054 --caname ca-org2 -M "${PWD}/organizations/peerOrganizations/org2.example.com/users/buyer@org2.example.com/msp" --tls.certfiles "${PWD}/organizations/fabric-ca/org2/tls-cert.pem"
```

运行以下命令，将节点 OU 配置文件复制到买家身份 MSP 文件夹中。

```
cp "${PWD}/organizations/peerOrganizations/org2.example.com/msp/config.yaml" "${PWD}/organizations/peerOrganizations/org2.example.com/users/buyer@org2.example.com/msp/config.yaml"
```



## 在私有数据中创建资产

现在我们已经创建了资产所有者的身份，我们可以调用 私有数据智能合约来创建新资产。复制并粘贴以下内容 在测试网络目录中的终端中设置命令：

**自己尝试一下**

```
export PATH=${PWD}/../bin:$PATH
export FABRIC_CFG_PATH=$PWD/../config/
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/owner@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
```

我们将使用该函数创建一个私有资产 data ― 资产 ID，颜色、大小和评估值为 。回想一下，私有数据**评估值**将与私有数据**资产 ID、颜色、大小**分开存储。 因此，该函数调用 API 两次以保存专用数据，每个集合一次。另请注意 私有数据使用标志传递。传递的输入 因为瞬态数据不会持久化在事务中以保持 数据私有。瞬态数据作为二进制数据传递，因此当 使用终端，它必须采用 base64 编码。我们使用环境变量 捕获 base64 编码值，并使用命令去除 Linux base64 命令添加的有问题的换行符。`CreateAsset``asset1``green``20``100``CreateAsset``PutPrivateData()``--transient``tr`

运行以下命令以创建资产：

```
export ASSET_PROPERTIES=$(echo -n "{\"objectType\":\"asset\",\"assetID\":\"asset1\",\"color\":\"green\",\"size\":20,\"appraisedValue\":100}" | base64 | tr -d \\n)
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n private -c '{"function":"CreateAsset","Args":[]}' --transient "{\"asset_properties\":\"$ASSET_PROPERTIES\"}"
```

您应该看到类似于以下内容的结果：

```
[chaincodeCmd] chaincodeInvokeOrQuery->INFO 001 Chaincode invoke successful. result: status:200
```

请注意，上面的命令仅针对 Org1 对等方。事务写入两个集合，并且 . 需要来自 Org1 对等方的认可才能写入集合，而继承链码的背书策略 . 来自 Org1 对等方的认可可以同时满足两种背书策略，并且能够在没有 Org2 认可的情况下创建资产。`CreateAsset``assetCollection``Org1MSPPrivateCollection``Org1MSPPrivateCollection``assetCollection``"OR('Org1MSP.peer','Org2MSP.peer')"`



## 以授权对等方身份查询私有数据

我们的集合定义允许 Org1 和 Org2 的所有对等方 将私有数据保存在其侧数据库中， 但只有 Org1 中的对等方才能拥有 Org1 对其私人数据的意见 数据库。作为 Org1 中的授权对等方，我们将查询两组私有数据。`assetID, color, size, and owner``appraisedValue`

第一个命令调用作为参数传递的函数。`query``ReadAsset``assetCollection`

```
// ReadAsset reads the information from collection
func (s *SmartContract) ReadAsset(ctx contractapi.TransactionContextInterface, assetID string) (*Asset, error) {

     log.Printf("ReadAsset: collection %v, ID %v", assetCollection, assetID)
     assetJSON, err := ctx.GetStub().GetPrivateData(assetCollection, assetID) //get the asset from chaincode state
     if err != nil {
         return nil, fmt.Errorf("failed to read asset: %v", err)
     }

     //No Asset found, return empty response
     if assetJSON == nil {
         log.Printf("%v does not exist in collection %v", assetID, assetCollection)
         return nil, nil
     }

     var asset *Asset
     err = json.Unmarshal(assetJSON, &asset)
     if err != nil {
         return nil, fmt.Errorf("failed to unmarshal JSON: %v", err)
     }

     return asset, nil

 }
```

第二个命令调用作为参数传递的函数。`query``ReadAssetPrivateDetails``Org1MSPPrivateDetails`

```
// ReadAssetPrivateDetails reads the asset private details in organization specific collection
func (s *SmartContract) ReadAssetPrivateDetails(ctx contractapi.TransactionContextInterface, collection string, assetID string) (*AssetPrivateDetails, error) {
     log.Printf("ReadAssetPrivateDetails: collection %v, ID %v", collection, assetID)
     assetDetailsJSON, err := ctx.GetStub().GetPrivateData(collection, assetID) // Get the asset from chaincode state
     if err != nil {
         return nil, fmt.Errorf("failed to read asset details: %v", err)
     }
     if assetDetailsJSON == nil {
         log.Printf("AssetPrivateDetails for %v does not exist in collection %v", assetID, collection)
         return nil, nil
     }

     var assetDetails *AssetPrivateDetails
     err = json.Unmarshal(assetDetailsJSON, &assetDetails)
     if err != nil {
         return nil, fmt.Errorf("failed to unmarshal JSON: %v", err)
     }

     return assetDetails, nil
 }
```

现在**自己尝试一下**

我们可以读取使用 ReadAsset 函数创建的资产的主要详细信息 以 Org1 身份查询资产集合集合：

```
peer chaincode query -C mychannel -n private -c '{"function":"ReadAsset","Args":["asset1"]}'
```

成功后，该命令将返回以下结果：

```
{"objectType":"asset","assetID":"asset1","color":"green","size":20,"owner":"x509::CN=appUser1,OU=admin,O=Hyperledger,ST=North Carolina,C=US::CN=ca.org1.example.com,O=org1.example.com,L=Durham,ST=North Carolina,C=US"}
```

资产的“所有者”是通过调用智能合约创建资产的身份。私有数据智能合约使用 API 读取身份证书的名称和颁发者。您可以在 owner 属性中看到身份证书的名称和颁发者。`GetClientIdentity().GetID()`

查询作为 Org1 成员的私有数据。`appraisedValue``asset1`

```
peer chaincode query -C mychannel -n private -c '{"function":"ReadAssetPrivateDetails","Args":["Org1MSPPrivateCollection","asset1"]}'
```

您应该看到以下结果：

```
{"assetID":"asset1","appraisedValue":100}
```



## 将私有数据作为未经授权的对等方进行查询

现在我们将操作来自 Org2 的用户。Org2 在其端数据库中具有策略中定义的资产传输私有数据，但不存储 组织 1 的资产数据。我们将查询这两组私有数据。`assetID, color, size, owner``assetCollection``appraisedValue`

### 切换到 Org2 中的对等方

运行以下命令以 Org2 成员身份运行并查询 Org2 对等方。

**自己尝试一下**

```bash
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/buyer@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051
```

### 查询私有数据 Org2 被授权

Org2 中的对等方应该在其端数据库中拥有第一组资产传输私有数据 （），并且可以使用与参数一起调用的函数访问它。`assetID, color, size and owner``ReadAsset()``assetCollection`

**自己尝试一下**

```
peer chaincode query -C mychannel -n private -c '{"function":"ReadAsset","Args":["asset1"]}'
```

成功后，应看到类似于以下结果的内容：

```
{"objectType":"asset","assetID":"asset1","color":"green","size":20,
"owner":"x509::CN=appUser1,OU=admin,O=Hyperledger,ST=North Carolina,C=US::CN=ca.org1.example.com,O=org1.example.com,L=Durham,ST=North Carolina,C=US" }
```

### 查询私有数据 Org2 无权

由于资产是由 Org1 创建的，因此关联的资产存储在集合中。值为 不由 Org2 中的对等方存储。运行以下命令以演示 资产未存储在 Org2 对等方中：`appraisedValue``asset1``Org1MSPPrivateCollection``appraisedValue``Org2MSPPrivateCollection`

**自己尝试一下**

```
peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n private -c '{"function":"ReadAssetPrivateDetails","Args":["Org2MSPPrivateCollection","asset1"]}'
```

空响应显示买家中不存在 asset1 私有详细信息 （组织2）私人收藏。

来自 Org2 的用户也无法读取 Org1 私有数据收集：

```
peer chaincode query -C mychannel -n private -c '{"function":"ReadAssetPrivateDetails","Args":["Org1MSPPrivateCollection","asset1"]}'
```

通过在集合配置文件中进行设置，我们 指定只有来自 Org1 的客户端才能从集合中读取数据。Org2 客户端 尝试读取集合的用户只会得到以下响应：`"memberOnlyRead": true`

```
Error: endorsement failure during query. response: status:500 message:"failed to
read asset details: GET_STATE failed: transaction ID: d23e4bc0538c3abfb7a6bd4323fd5f52306e2723be56460fc6da0e5acaee6b23: tx
creator does not have read access permission on privatedata in chaincodeName:private collectionName: Org1MSPPrivateCollection"
```

来自 Org2 的用户将只能看到私有数据的公共哈希。



## 转移资产

让我们看看转移到 Org2 需要什么。在这种情况下，Org2 需要同意 从 Org1 购买资产，他们需要就 .您可能想知道他们如何 同意 Org1 是否将他们的意见保留在其私有端数据库中。对于答案 对此，让我们继续。`asset1``appraisedValue``appraisedValue`

**自己尝试一下**

使用我们的对等 CLI 切换回终端。

要转移资产，买方（接收方）需要通过调用链码函数与资产所有者同意相同的协议。约定的值将存储在 Org2 对等方的集合中。运行以下命令以同意评估值 100 作为 Org2：`appraisedValue``AgreeToTransfer``Org2MSPDetailsCollection`

```
export ASSET_VALUE=$(echo -n "{\"assetID\":\"asset1\",\"appraisedValue\":100}" | base64 | tr -d \\n)
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n private -c '{"function":"AgreeToTransfer","Args":[]}' --transient "{\"asset_value\":\"$ASSET_VALUE\"}"
```

买方现在可以查询他们在 Org2 私有数据收集中同意的值：

```
peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n private -c '{"function":"ReadAssetPrivateDetails","Args":["Org2MSPPrivateCollection","asset1"]}'
```

调用将返回以下值：

```
{"assetID":"asset1","appraisedValue":100}
```

既然买方已同意以评估价值购买资产，所有者可以转让 组织 2 的资产。资产需要由拥有资产的身份转移， 所以让我们充当 Org1：

```
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/owner@org1.example.com/msp
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_ADDRESS=localhost:7051
```

来自 Org1 的所有者可以读取 AgreeToTransfer 交易添加的数据以查看买方身份：

```
peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n private -c '{"function":"ReadTransferAgreement","Args":["asset1"]}'
{"assetID":"asset1","buyerID":"eDUwOTo6Q049YnV5ZXIsT1U9Y2xpZW50LE89SHlwZXJsZWRnZXIsU1Q9Tm9ydGggQ2Fyb2xpbmEsQz1VUzo6Q049Y2Eub3JnMi5leGFtcGxlLmNvbSxPPW9yZzIuZXhhbXBsZS5jb20sTD1IdXJzbGV5LFNUPUhhbXBzaGlyZSxDPVVL"}
```

我们现在拥有转移资产所需的一切。智能合约使用该函数来检查资产评估的哈希值 中的值与 中的评估值的哈希值匹配。如果哈希相同，则确认 所有者和感兴趣的买方已同意相同的资产价值。如果 满足条件，转账函数将获取买家的客户ID 从转让协议中，并使买方成为资产的新所有者。转移 功能还将从前所有者的收藏中删除资产评估值， 以及从 .`GetPrivateDataHash()``Org1MSPPrivateCollection``Org2MSPPrivateCollection``assetCollection`

运行以下命令以传输资产。所有者需要提供 资产 ID 和买方到转移交易的组织 MSP ID：

```
export ASSET_OWNER=$(echo -n "{\"assetID\":\"asset1\",\"buyerMSP\":\"Org2MSP\"}" | base64 | tr -d \\n)
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n private -c '{"function":"TransferAsset","Args":[]}' --transient "{\"asset_owner\":\"$ASSET_OWNER\"}" --peerAddresses localhost:7051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt"
```

您可以查询以查看传输结果：`asset1`

```
peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n private -c '{"function":"ReadAsset","Args":["asset1"]}'
```

结果将显示买方身份现在拥有资产：

```
{"objectType":"asset","assetID":"asset1","color":"green","size":20,"owner":"x509::CN=appUser2, OU=client + OU=org2 + OU=department1::CN=ca.org2.example.com, O=org2.example.com, L=Hursley, ST=Hampshire, C=UK"}
```

资产的“所有者”现在具有买方身份。

您还可以确认转移从 Org1 集合中删除了私有详细信息：

```
peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n private -c '{"function":"ReadAssetPrivateDetails","Args":["Org1MSPPrivateCollection","asset1"]}'
```

查询将返回空结果，因为资产私有数据已从 Org1 私有数据收集中删除。



## 清除私有数据

对于私有数据只需要保留一小段时间的用例， 可以在一定数量的块后“清除”数据，留下 仅作为交易不可变证据的数据哈希值。 如果数据包含敏感数据，组织可以决定清除私有数据 其他事务使用但不再需要的信息，或 如果数据正在复制到链下数据库中。

示例中的数据包含一个专用协议，该协议 组织可能希望在一段时间后过期。因此，它 寿命有限，在 使用该属性的指定数量的块的区块链 在集合定义中。`appraisedValue``blockToLive`

该定义的属性值为 ，这意味着此数据将存在于 三个街区，然后它将被清除。如果我们创建额外的 在通道上被阻止，Org2 最终会同意 被清除。我们可以创建 3 个新块来演示：`Org2MSPPrivateCollection``blockToLive``3``appraisedValue`

**自己尝试一下**

在终端中运行以下命令以切换回以成员身份操作 的组织 2 和目标 Org2 对等体：

```
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/buyer@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051
```

我们仍然可以在 ：`appraisedValue``Org2MSPPrivateCollection`

```
peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n private -c '{"function":"ReadAssetPrivateDetails","Args":["Org2MSPPrivateCollection","asset1"]}'
```

您应该会看到日志中打印的值：

```
{"assetID":"asset1","appraisedValue":100}
```

由于我们需要跟踪在清除私有数据之前添加的块数， 打开一个新的终端窗口并运行以下命令以查看 Org2 对等方。记下最高的块号。

```
docker logs peer0.org1.example.com 2>&1 | grep -i -a -E 'private|pvt|privdata'
```

现在返回到我们作为 Org2 成员的终端并运行以下命令 命令以创建三个新资产。每个命令将创建一个新块。

```
export ASSET_PROPERTIES=$(echo -n "{\"objectType\":\"asset\",\"assetID\":\"asset2\",\"color\":\"blue\",\"size\":30,\"appraisedValue\":100}" | base64 | tr -d \\n)
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n private -c '{"function":"CreateAsset","Args":[]}' --transient "{\"asset_properties\":\"$ASSET_PROPERTIES\"}"
export ASSET_PROPERTIES=$(echo -n "{\"objectType\":\"asset\",\"assetID\":\"asset3\",\"color\":\"red\",\"size\":25,\"appraisedValue\":100}" | base64 | tr -d \\n)
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n private -c '{"function":"CreateAsset","Args":[]}' --transient "{\"asset_properties\":\"$ASSET_PROPERTIES\"}"
export ASSET_PROPERTIES=$(echo -n "{\"objectType\":\"asset\",\"assetID\":\"asset4\",\"color\":\"orange\",\"size\":15,\"appraisedValue\":100}" | base64 | tr -d \\n)
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n private -c '{"function":"CreateAsset","Args":[]}' --transient "{\"asset_properties\":\"$ASSET_PROPERTIES\"}"
```

返回到另一个终端并运行以下命令以确认 新资产导致创建了三个新区块：

```
docker logs peer0.org1.example.com 2>&1 | grep -i -a -E 'private|pvt|privdata'
```

现已从私人数据收集中清除。从 Org2 终端再次发出查询以查看 响应为空。`appraisedValue``Org2MSPDetailsCollection`

```
peer chaincode query -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n private -c '{"function":"ReadAssetPrivateDetails","Args":["Org2MSPPrivateCollection","asset1"]}'
```



## 对私有数据使用索引

索引也可以应用于私有数据收集，方法是将索引打包在 目录 与链码一起。[此处](https://github.com/hyperledger/fabric-samples/blob/main//asset-transfer-private-data/chaincode-go/META-INF/statedb/couchdb/collections/assetCollection/indexes/indexOwner.json)提供了一个示例索引。`META-INF/statedb/couchdb/collections/<collection_name>/indexes`

对于将链码部署到生产环境，建议 定义链码旁边的任何索引，以便链码和支持 一旦链码 安装在对等体上并在通道上实例化。关联的索引是 在通道上实例化链码时自动部署，当 指定标志指向 集合 JSON 文件。`--collections-config`

注意

无法创建用于隐式私有数据收集的索引。 隐式集合基于组织名称并自动创建。名称的格式 是 请参阅 [FAB-17916](https://jira.hyperledger.org/browse/FAB-17916) 了解更多信息。`_implicit_org_<OrgsMSPid>`

## 收拾

当您使用完私有数据智能合约后，您可以降低测试 使用脚本的网络。`network.sh`

```
./network.sh down
```

此命令将关闭网络的 CA、对等节点和排序节点 我们创造的。请注意，账本上的所有数据都将丢失。 如果要再次学习本教程，将从干净的初始状态开始。



## 其他资源

对于额外的私有数据教育，已经创建了一个视频教程。

注意

该视频使用以前的生命周期模型来安装私有数据 带有链码的集合。





# 官方文档教程03-运行外部链码生成器

Fabric v2.4.1 外部链代码构建器提供了一种实用的方法来运行智能合约，使peer能够运行外部（对自身）命令来管理链码。 相比之下，早期将智能合约部署到channel方法需要peer节点来编排链代码的完整生命周期。 这要求peer有权访问 Docker 守护进程以创建映像并启动容器。 Java、Node.js 和 Go 链码框架对同行来说是明确已知的，包括它们应该如何构建和启动。

因此，传统的链码部署方法很难将链码部署到 Kubernetes (K8s) 或其他对 Docker 守护进程的访问受到限制的环境中，并且很难以任何形式的调试模式运行链码。 此外，代码通常由peer重新构建，需要外部互联网连接并引入一些关于已安装依赖项的不确定性。

链码即服务方法确实需要管理员协调链码构建和部署阶段。 尽管这增加了一个步骤，但它为管理员提供了对该过程的控制。 peer仍然需要安装“链码包”，但没有代码——只安装有关链码托管位置的信息（例如主机名、端口和 TLS 配置）。

## 1.Fabric v2.4.1 改进

测试网络使用最新的 Fabric 版本 （v2.4.1），这有助于使用链码‘-as-a-service’：

- peer节点的 Docker 映像包含一个名为“ccaasbuilder”的链码‘-as-a-service’的预配置构建器。这消除了构建自己的外部构建器并重新打包和配置peer的先前要求。
- ccaasbuilder 应用程序包含在二进制 tgz 存档下载中，以供在其他情况下使用。 sampleconfig/core.yaml 更新为引用“ccaasbuilder”。
- Fabric v2.4.1 Java 链码消除了编写自定义引导主类的要求（在 Node.js 链码中实现并计划用于 go 链码）。

**注意：**此核心功能在早期版本中也可用，但需要编写外部链码代码生成器二进制文件并正确配置 core.yaml。

## 2.通过测试网络实现端到端

test-network和一些链码已经更新，以支持链码‘-as-a-service’的运行。下面的命令需要最新的fabric-samples以及最新的fabric docker映像。

首先打开两个终端窗口，一个用于启动Fabric测试网络，另一个用于监视Docker容器。在”monitoring”窗口中，运行以下bash脚本来监视fabric_test网络上Docker容器的活动；这将监视添加到fabric-test网络的所有Docker容器。

test-network通常是通过运行./network.sh up命令来创建的，因此请延迟运行bash脚本，直到创建网络。（注意，可以使用docker network create fabric-test提前创建网络。





bash

```bash
# from the fabric-samples repo
./test-network/monitordocker.sh
```

在”Fabric Network”窗口中，启动测试网络:

```bash
cd test-network
./network.sh up createChannel
```

下一个命令的变体，例如使用 CouchDB 或 CA，可以在不影响链码‘-as-a-service’功能的情况下使用。三个关键步骤如下，顺序不分：

- 生成链码包的Docker映像，其中包含用于确定链码容器（托管一个或多个合同）运行位置的信息。资产转移-basic/chaincode-typescript和资产转移-basic/chaincode-java都用Docker文件进行了更新。
- 安装、批准和提交链码定义；无论是否使用外部链码生成器，都将运行这些命令。链码包只包含连接信息（主机名、端口、TLS证书），没有代码。
- 启动包含合同的Docker容器。

容器必须在peer提交第一个事务之前运行。如果设置了initRequired标志，这可能发生在提交时。

该序列可以按如下方式运行:





bash

```bash
./network.sh deployCCAAS  -ccn basicts -ccp ../asset-transfer-basic/chaincode-typescript
```

这与deployCC命令类似，因为它指定了名称和路径。因为每个容器都在fabric-test网络上，更改端口可以避免与其他链码容器发送冲突。如果运行多个服务，则需要更改端口。

如果成功到这一点，智能合约（链码）应该在监控窗口中启动。应该有两个容器在运行，一个用于org1，一个用于org2。容器名称包含组织名称、peer名称和链码名称。

作为测试，运行“合同原数据”功能，如下所示。（有关作为不同组织进行测试的详细信息，请参阅[与网络交互](https://hyperledger-fabric.readthedocs.io/en/latest/test_network.html#interacting-with-the-network)）。





bash

```bash
# Environment variables for Org1
# Org1的环境变量
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/tlsca/tlsca.org1.example.com-cert.pem
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
export PATH=${PWD}/../bin:$PATH
export FABRIC_CFG_PATH=${PWD}/../config

# invoke the function
# 调用函数
peer chaincode query -C mychannel -n basicts -c '{"Args":["org.hyperledger.fabric:GetMetadata"]}' | jq
```





json

```json
{
  "$schema": "https://hyperledger.github.io/fabric-chaincode-node/main/api/contract-schema.json",
  "contracts": {
    "AssetTransferContract": {
      "name": "AssetTransferContract",
      "contractInstance": {
        "name": "AssetTransferContract",
        "default": true
      },
      "transactions": [
        {
          "tag": [
            "SUBMIT",
            "submitTx"
          ],
          "parameters": [],
          "name": "InitLedger"
        },
        {
          "tag": [
            "SUBMIT",
            "submitTx"
          ],
          "parameters": [
            {
              "name": "id",
              "description": "",
              "schema": {
                "type": "string"
              }
            },
            {
              "name": "color",
              "description": "",
              "schema": {
                "type": "string"
              }
            },
            {
              "name": "size",
              "description": "",
              "schema": {
                "type": "number"
              }
            },
            {
              "name": "owner",
              "description": "",
              "schema": {
                "type": "string"
              }
            },
            {
              "name": "appraisedValue",
              "description": "",
              "schema": {
                "type": "number"
              }
            }
          ],
          "name": "CreateAsset"
        },
        {
          "tag": [
            "EVALUATE"
          ],
          "parameters": [
            {
              "name": "id",
              "description": "",
              "schema": {
                "type": "string"
              }
            }
          ],
          "name": "ReadAsset"
        },
        {
          "tag": [
            "SUBMIT",
            "submitTx"
          ],
          "parameters": [
            {
              "name": "id",
              "description": "",
              "schema": {
                "type": "string"
              }
            },
            {
              "name": "color",
              "description": "",
              "schema": {
                "type": "string"
              }
            },
            {
              "name": "size",
              "description": "",
              "schema": {
                "type": "number"
              }
            },
            {
              "name": "owner",
              "description": "",
              "schema": {
                "type": "string"
              }
            },
            {
              "name": "appraisedValue",
              "description": "",
              "schema": {
                "type": "number"
              }
            }
          ],
          "name": "UpdateAsset"
        },
        {
          "tag": [
            "SUBMIT",
            "submitTx"
          ],
          "parameters": [
            {
              "name": "id",
              "description": "",
              "schema": {
                "type": "string"
              }
            }
          ],
          "name": "DeleteAsset"
        },
        {
          "returns": {
            "type": "boolean"
          },
          "name": "AssetExists",
          "tag": [
            "EVALUATE"
          ],
          "parameters": [
            {
              "name": "id",
              "description": "",
              "schema": {
                "type": "string"
              }
            }
          ]
        },
        {
          "tag": [
            "SUBMIT",
            "submitTx"
          ],
          "parameters": [
            {
              "name": "id",
              "description": "",
              "schema": {
                "type": "string"
              }
            },
            {
              "name": "newOwner",
              "description": "",
              "schema": {
                "type": "string"
              }
            }
          ],
          "name": "TransferAsset"
        },
        {
          "returns": {
            "type": "string"
          },
          "name": "GetAllAssets",
          "tag": [
            "EVALUATE"
          ],
          "parameters": []
        }
      ],
      "info": {
        "title": "AssetTransfer",
        "description": "Smart contract for trading assets",
        "name": "AssetTransferContract",
        "version": ""
      }
    },
    "org.hyperledger.fabric": {
      "name": "org.hyperledger.fabric",
      "contractInstance": {
        "name": "org.hyperledger.fabric"
      },
      "transactions": [
        {
          "name": "GetMetadata"
        }
      ],
      "info": {
        "title": "",
        "version": ""
      }
    }
  },
  "info": {
    "version": "1.0.0",
    "title": "asset-transfer-basic"
  },
  "components": {
    "schemas": {}
  }
}
```

注意，如果没有安装jq，可以省略|jq。但是，原数据在JSON中显示了部署契约的详细信息，因此jq提供了易读性。要确认智能合约正在工作，请重复org2前面的命令。

要运行Java示例，请按如下方式更改deployCCAAS命令，该命令将创建两个新容器:





bash

```bash
./network.sh deployCCAAS  -ccn basicj -ccp ../asset-transfer-basic/chaincode-java
```

## 3.可能存在的错误

如果传递的JSON结构格式不正确，对等日志将包含以下错误:





bash

```bash
::Error: Failed to unmarshal json: cannot unmarshal string into Go value of type map[string]interface {} command=build
```

## 4.如何配置每种语言

每种语言都可以在“即服务”模式下运行。以下方法基于出版时的最新库。启动映像时，可以指定相应链码库的任何TLS选项或其他日志记录选项。

### 4.1Java

使用Fabric v2.4.1Java链码库，没有代码更改或构建更改来实现。如果设置了环境变量CHAINCODE_SERVER_ADDRESS，则将使用“-as-a-service”模式。

下面的示例Docker run命令显示了两个必需的变量，CHAINCODE_SERVER_ADDRESS和CORE_CHAICODE_ID_NAME:





bash

```bash
docker run --rm -d --name peer0org1_assettx_ccaas  \
              --network fabric_test \
              -e CHAINCODE_SERVER_ADDRESS=0.0.0.0:9999 \
              -e CORE_CHAINCODE_ID_NAME=<use package id here> \
               assettx_ccaas_image:latest
```

### 4.2Node.js

对于Node.js（JavaScript或TypeScript）链码，package.json通常使用fabric-chaincode-node start作为主启动命令。要以“-as-a-service”模式运行，请将此启动命令更改为fabric-chaincode-node服务器–链码地址=$CHAINCODE_SERVER_ADDRESS-链码ID=$CHAINCODE_ID

## 5.调试链码

运行在‘-as-a-service’模式提供选项，类似于用于调试代码的Fabric“dev”模式。dev模式的限制不适用于‘-as-a-service’。

deployCCAAS命令可以提供-cca as docker false选项来不生成Docker映像或启动Docker容器。该选项输出将要运行的命令。

命令输出类似于以下示例：





bash

```bash
./network.sh deployCCAAS  -ccn basicj -ccp ../asset-transfer-basic/chaincode-java -ccaasdocker false
#....
Not building docker image; this the command we would have run
docker build -f ../asset-transfer-basic/chaincode-java/Dockerfile -t basicj_ccaas_image:latest --build-arg CC_SERVER_PORT=9999 ../asset-transfer-basic/chaincode-java
#....
Not starting docker containers; these are the commands we would have run
    docker run --rm -d --name peer0org1_basicj_ccaas                    --network fabric_test                   -e CHAINCODE_SERVER_ADDRESS=0.0.0.0:9999                   -e CHAINCODE_ID=basicj_1.0:59dcd73a14e2db8eab7f7683343ce27ac242b93b4e8075605a460d63a0438405 -e CORE_CHAINCODE_ID_NAME=basicj_1.0:59dcd73a14e2db8eab7f7683343ce27ac242b93b4e8075605a460d63a0438405                     basicj_ccaas_image:latest
```

注意：根据目录位置或调试要求，前面的命令可能需要调整。

## 6.构建 Docker 镜像

调试链码的第一个要求是构建Docker映像。只要peer能够连接到connection.json中指定的hostname:port，链码的实际打包对peer就不重要了。可以重新定位下面指定的Docker文件。

手动构建用于资产转移的Docker映像-basic/chaincode-java：





bash

```bash
docker build -f ../asset-transfer-basic/chaincode-java/Dockerfile -t basicj_ccaas_image:latest --build-arg CC_SERVER_PORT=9999 ../asset-transfer-basic/chaincode-java
```

## 7.启动 Docker 容器

下来，必须启动 Docker 容器。例如，在 Node.js 中，容器可以按如下方式启动：





bash

```bash
docker run --rm -it -p 9229:9229 --name peer0org2_basic_ccaas --network fabric_test -e DEBUG=true -e CHAINCODE_SERVER_ADDRESS=0.0.0.0:9999 -e CHAINCODE_ID=basic_1.0:7c7dff5cdc43c77ccea028c422b3348c3c1fb5a26ace0077cf3cc627bd355ef0 -e CORE_CHAINCODE_ID_NAME=basic_1.0:7c7dff5cdc43c77ccea028c422b3348c3c1fb5a26ace0077cf3cc627bd355ef0 basic_ccaas_image:latest
```

例如，在Java中，Docker容器可以按如下方式启动：





bash

```bash
 docker run --rm -it --name peer0org1_basicj_ccaas -p 8000:8000 --network fabric_test -e DEBUG=true -e CHAINCODE_SERVER_ADDRESS=0.0.0.0:9999 -e CHAINCODE_ID=basicj_1.0:b014a03d8eb1898535e25b4dfeeb3f8244c9f07d91a06aec03e2d19174c45e4f -e CORE_CHAINCODE_ID_NAME=basicj_1.0:b014a03d8e
b1898535e25b4dfeeb3f8244c9f07d91a06aec03e2d19174c45e4f  basicj_ccaas_image:latest
```

## 8.调试先决条件

以下先决条件适用于调试所有语言:

- 容器名称必须与peer的connection.json中的名称匹配。
- peer节点通过Docker网络连接到链码容器。因此，端口9999不需要转发到主机。
- 调试器中的单个单步执行可能会触发默认的Fabric事务超时值30秒。通过将CORE_CHAINCODE_EXECUTETIMEOUT=300s添加到test-network/docker/docker-composer-test-net.yml文件中每个peer的环境选项中，将链码完成事务的时间增加到300秒。
- 在上一节中的docker run命令中，test-network -d默认选项已被-it取代。此更改在前台运行Docker容器，而不是在分离模式下运行Docker容器。

以下先决条件适用于调试Node.js:

- 端口9229被转发。但是，这是Node.js使用的调试端口。
- -e DEBUG=true将触发节点运行时以调试模式启动。这是在docker/docker-entrypoint.sh脚本中编码的，出于安全考虑，应该考虑从生产映像中删除该脚本。
- 如果您使用的是TypeScript，请确保TypeScript已使用sourcemaps编译；否则，调试器将难以匹配源代码。

以下先决条件适用于调试 Java：

- 端口8000被转发，这是JVM的调试端口。
- -e DEBUG=true将触发节点运行时以调试模式启动。这是一个在docker/docker-entrypoint.sh脚本中编码的示例，出于安全考虑，应该考虑将其从生产映像中删除。
- 启动调试器的java命令选项是java-agent lib:jdwp=传输=dt_socket，服务器=y，挂起=n，地址=0.0.0.0:8000 -jar/链码.jar。注意0.0.0.0，作为调试端口，必须绑定到所有的网络适配器因此可以从容器外部连接调试器。

## 9.与多个peer一起运行

在前面的方法中，链码被批准的每个peer都有一个运行链码的容器。‘-as-a-service’方法需要实现相同的体系结构。

json包含正在运行的链码容器的地址，因此可以对其进行更新，以确保每个peer连接到不同的容器。但是，与链码包中的connection.json一样，Fabric要求包ID在组织中的所有peer之间保持一致。为了实现这一点，外部构建器支持模板功能。此模板的上下文取自每个peer上设置的环境变量CHAINCODE_AS_A_SERVICE_BUILDER_CONFIG。

将地址定义为connection.json中的模板，如下所示：





json

```json
{
  "address": "{{.peername}}_assettransfer_ccaas:9999",
  "dial_timeout": "10s",
  "tls_required": false
}
```

在peer的环境配置中，为org1的peer1设置以下变量:





bash

```bash
CHAINCODE_AS_A_SERVICE_BUILDER_CONFIG="{\"peername\":\"org1peer1\"}"
```

外部构建器然后将此地址解析为org1peer1_assettransfer_ccaas:9999，供peer使用。

每个peer都可以有自己独立的配置，因此有一个唯一的地址。设置的JSON字符串可以具有任何结构，只要模板（在golang模板语法中）匹配。

connection.json中的任何值都可以模板化，但只能模板化值，不能模板化键。

------

 ***文章作者:\*** [普信摩羯妈宝凤凰下头搞笑男](http://somewei.cn/about)

 ***文章链接:\*** http://junwuqing.github.io/2022/12/05/guan-fang-wen-dang-jiao-cheng-03-yun-xing-wai-bu-lian-ma-sheng-cheng-qi/

 ***版权声明:\*** 本博客所有文章除特別声明外，均采用 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.zh) 许可协议。转载请注明来源 [普信摩羯妈宝凤凰下头搞笑男](http://somewei.cn/about) !


来源: somewei
文章作者: 普信摩羯妈宝凤凰下头搞笑男
文章链接: http://somewei.cn/2022/12/05/guan-fang-wen-dang-jiao-cheng-03-yun-xing-wai-bu-lian-ma-sheng-cheng-qi/
本文章著作权归作者所有，任何形式的转载都请注明出处。



# 部署*Hyperledger Fabric*生产网络



本部署指南是对设置生产 Fabric 网络组件的正确顺序的高级概述，此外还有最佳实践和部署时要牢记的众多注意事项中的一些。请注意，本主题将从单个人的角度将“建立网络”作为一个整体过程进行讨论。更有可能的是，现实世界的生产网络不会由单个人建立，而是由几个人（例如，一组银行各自建立自己的组件）指导的协作努力。

部署 *Fabric* 网络的过程非常复杂，需要了解公钥基础结构和管理分布式系统。如果您是智能合约或应用程序开发人员，则在部署生产级 *Fabric* 网络时不需要这种级别的专业知识。但是，您可能需要了解网络的部署方式，以便开发有效的智能合约和应用程序。

如果您只需要一个开发环境来测试链码、智能合约和应用程序，请查看[使用 Fabric 测试网络](https://hyperledger-fabric.readthedocs.io/en/latest/test_network.html)。您将部署的网络将包括两个组织（每个组织拥有一个对等节点）和一个拥有单个排序节点的排序服务组织。**此测试网络并非旨在提供部署生产组件的蓝图，也不应用作蓝图，因为它会做出生产部署不会做出的假设和决策。**

本指南将概述设置生产组件和生产网络的步骤：

- [第一步：确定网络配置](https://hyperledger-fabric.readthedocs.io/en/latest/deployment_guide_overview.html#dg-step-one-decide-on-your-network-configuration)

- [步骤 2：为资源设置群集](https://hyperledger-fabric.readthedocs.io/en/latest/deployment_guide_overview.html#dg-step-two-set-up-a-cluster-for-your-resources)

- [步骤 3：设置 CA](https://hyperledger-fabric.readthedocs.io/en/latest/deployment_guide_overview.html#dg-step-three-set-up-your-cas)

- [步骤 4：使用 CA 创建标识和 MSP](https://hyperledger-fabric.readthedocs.io/en/latest/deployment_guide_overview.html#dg-step-four-use-the-ca-to-create-identities-and-msps)

- [步骤 5：部署对等节点和排序节点](https://hyperledger-fabric.readthedocs.io/en/latest/deployment_guide_overview.html#dg-step-five-deploy-nodes)

  [创建对等方](https://hyperledger-fabric.readthedocs.io/en/latest/deployment_guide_overview.html#dg-create-a-peer)[创建排序节点](https://hyperledger-fabric.readthedocs.io/en/latest/deployment_guide_overview.html#dg-create-an-ordering-node)



## 第一步：确定网络配置

区块链网络的结构将由它所服务的用例决定。有太多的选项无法在每个点上提供明确的指导，但让我们考虑一些情况。与开发环境或概念证明相比，安全性、资源管理和高可用性在生产中运行时成为优先事项。您需要多少个节点来满足高可用性，您希望在哪些数据中心部署这些节点以满足灾难恢复和数据驻留的需求？您将如何确保您的私钥和信任根保持安全？除上述内容外，以下是在部署组件之前需要做出的决策示例：

- **证书颁发机构配置**。 作为您必须对等方（多少、每个通道上多少节点等）和订购服务（多少节点、谁将拥有它们）做出的总体决策的一部分，您还必须决定如何部署组织的 CA。生产网络应使用传输层安全性 （TLS），这将需要设置 TLS CA 并使用它来生成 TLS 证书。需要在注册 CA 之前部署此 TLS CA。我们将在[步骤 3：设置 CA](https://hyperledger-fabric.readthedocs.io/en/latest/deployment_guide_overview.html#dg-step-three-set-up-your-cas) 中对此进行更多讨论。
- **是否使用组织单位？**某些组织可能会发现有必要建立组织单位，以便在某些标识和单个 CA 创建的 MSP 之间创建分离（例如，制造商可能希望一个组织单位用于其运输部门，另一个组织单位用于其质量控制部门）。请注意，这与“节点 OU”的概念是分开的，在“节点 OU”中，标识可以具有编码到其中的角色（例如，“管理员”或“对等方”）。
- **数据库类型。**网络中的某些通道可能要求以[状态数据库可以理解的CouchDB](https://hyperledger-fabric.readthedocs.io/en/latest/couchdb_as_state_database.html)方式对所有数据进行建模，而其他网络优先考虑速度，可能会决定所有对等体都将使用LevelDB。请注意，通道上不应有同时使用 CouchDB 和 LevelDB 的对等方，因为 CouchDB 对键和值施加了一些数据限制。在 LevelDB 中有效的键和值在 CouchDB 中可能无效。
- **是否创建系统通道。**排序节点可以使用称为“系统通道”（可以从中创建应用程序通道）的管理通道的配置块进行引导，或者根据需要简单地启动并加入应用程序通道。建议的方法是在没有配置块的情况下引导，这是本部署指南假设您将采用的方法。有关创建系统通道创世区块并使用它引导排序节点的更多信息，请查看 Fabric v2.2 文档中的[部署生产网络](https://hyperledger-fabric.readthedocs.io/en/release-2.2/deployment_guide_overview.html#creating-an-ordering-node)。
- **频道和私人数据。**某些网络可能会认为[渠道](https://hyperledger-fabric.readthedocs.io/en/latest/channels.html)是确保某些交易的隐私和隔离的最佳方式。其他人可能会决定减少渠道，必要时辅以[私人数据收集](https://hyperledger-fabric.readthedocs.io/en/latest/private-data/private-data.html)，可以更好地满足他们的隐私需求。
- **容器编排。**不同的用户也可能对他们的容器编排做出不同的决定，为他们的对等进程、日志记录、CouchDB、gRPC 通信和链码创建单独的容器，而其他用户可能决定组合其中一些进程。
- **链码部署方法。**用户可以选择使用内置的构建和运行支持、使用外部构建器和[启动器的自定义构建](https://hyperledger-fabric.readthedocs.io/en/latest/cc_launcher.html)和运行，或使用运行链码[作为外部服务](https://hyperledger-fabric.readthedocs.io/en/latest/cc_service.html)来部署他们的链码。
- **使用防火墙。**在生产部署中，属于一个组织的组件可能需要访问其他组织的组件，因此需要使用防火墙和高级网络配置。例如，使用 Fabric SDK 的应用程序需要访问来自所有组织的所有背书对等方以及所有渠道的排序服务。同样，对等方需要访问他们从中接收新块的通道上的排序服务。

无论您的组件部署在何处，您都需要在您选择的管理系统（例如 Kubernetes）方面具有高度的专业知识，以便有效地运营您的网络。同样，网络结构的设计必须适合业务用例以及网络将在其中运行所在的行业政府的任何相关法律和法规。本部署指南不会介绍每次迭代和潜在的网络配置，但会提供需要考虑的通用准则和规则。



## 步骤 2：为资源设置群集

一般来说，Fabric 与用于部署和管理它的方法无关。例如，可以在笔记本电脑上部署和管理对等方。出于多种原因，这可能是不可取的，但 Fabric 中没有任何内容禁止它。

只要您有能力部署容器，无论是在本地（或防火墙后面）还是在云中，都应该可以建立组件并将它们相互连接。但是，Kubernetes 具有许多有用的工具，使其成为用于部署和管理 Fabric 网络的流行容器管理平台。有关 Kubernetes 的更多信息，请查看 [Kubernetes 文档](https://kubernetes.io/docs)。本主题主要将其范围限制为二进制文件，并提供在使用 Docker 部署或 Kubernetes 时可以应用的说明。

无论您选择在何处部署组件，都需要确保有足够的资源使组件有效运行。您需要的大小在很大程度上取决于您的用例。如果您计划将单个对等方加入多个高容量通道，则与仅计划加入单个通道相比，它将需要更多的 CPU 和内存。粗略估计，当您计划分配给单个排序节点时，计划将大约三倍的资源专用于对等节点（如下所示，建议在排序服务中至少部署三个节点，最好部署五个节点）。同样，对于 CA，您应该需要大约十分之一的资源，就像对等方一样。您还需要向集群添加存储（某些云提供商可能会提供存储），因为如果不先与云提供商设置存储，则无法配置持久卷和持久卷声明。持久存储的使用可确保 MSP、账本和已安装的链码等数据不会存储在容器文件系统上，从而防止它们在容器被销毁时被销毁。

通过部署概念验证网络并在负载下对其进行测试，您将更好地了解所需的资源。

### 管理您的基础架构

您用于管理后端的确切方法和工具将取决于您选择的后端。但是，这里有一些注意事项值得注意。

- 使用机密对象在群集中安全地存储重要的配置文件。有关 Kubernetes 机密的信息，请查看 [Kubernetes 机密](https://kubernetes.io/docs/concepts/configuration/secret/)。您还可以选择使用硬件安全模块 （HSM） 或加密的持久卷 （PV）。同样，在部署 Fabric 组件后，您可能希望在自己的后端连接到容器，例如在 Docker Hub 等服务中使用私有存储库。在这种情况下，您需要以 Kubernetes 密钥的形式对登录信息进行编码，并在部署组件时将其包含在 YAML 文件中。
- 群集注意事项和节点大小调整。在上面的步骤 2 中，我们讨论了如何考虑节点大小的一般大纲。您的用例以及稳健的开发期是您真正了解对等节点、排序节点和 CA 需要多大的唯一途径。
- 您选择如何装载卷。最佳做法是将与节点相关的卷挂载到节点部署位置的外部。这将允许您稍后引用这些卷（例如，重新启动已崩溃的节点或容器），而无需重新部署或重新生成加密材料。
- 您将如何监视资源。建立策略和方法来监视各个节点使用的资源以及通常部署到群集的资源至关重要。当您将对等方加入更多通道时，您可能需要增加其 CPU 和内存分配。同样，您需要确保有足够的存储空间来存储状态数据库和区块链。



## 步骤 3：设置 CA[¶](https://hyperledger-fabric.readthedocs.io/en/latest/deployment_guide_overview.html#step-three-set-up-your-cas)

必须在结构网络中部署的第一个组件是 CA。这是因为必须先创建与节点关联的证书（不仅针对节点本身，还创建标识谁可以管理节点的证书），然后才能部署节点本身。虽然不必使用结构 CA 来创建这些证书，但结构 CA 还会创建正确定义组件和组织所需的 MSP 结构。如果用户选择使用结构 CA 以外的 CA，则必须自行创建 MSP 文件夹。

- 一个 CA（或多个，如果您使用的是中间 CA，则在下面详细介绍中间 CA）用于生成（通过称为“注册”的过程）组织管理员的证书、该组织的 MSP 以及该组织拥有的任何节点。此 CA 还将为任何其他用户生成证书。由于其在“注册”身份中的作用，此 CA 有时称为“注册 CA”或“ecert CA”。
- 另一个 CA 生成用于保护传输层安全性 （TLS） 上的通信的证书。因此，此 CA 通常称为“TLS CA”。这些 TLS 证书附加到操作，以防止“中间人”攻击。请注意，TLS CA 仅用于为节点颁发证书，可以在该活动完成后关闭。用户可以选择使用单向（仅限客户端）TLS以及双向（服务器和客户端）TLS，后者也称为“双向TLS”。由于在部署“注册”CA 之前，应确定是否指定您的网络将使用 TLS（建议使用）（指定此 CA 配置的 YAML 文件具有用于启用 TLS 的字段），因此应首先部署 TLS CA，并在引导注册 CA 时使用其根证书。连接到注册 CA 时，此 TLS 证书也将用于注册用户和节点的标识。`fabric-ca client`

虽然与组织关联的所有非 TLS 证书都可以由单个“根”CA（即，作为其自己的信任根的 CA）创建，但对于添加的安全性，组织可以决定使用其证书由根 CA（或最终导致返回根 CA 的另一个中间 CA）创建的“中间”CA。由于根 CA 中的泄露会导致其整个信任域（管理员、节点及其为其生成证书的任何 CA 的证书）崩溃，因此中间 CA 是限制根 CA 公开的有用方法。是否选择使用中间 CA 将取决于您的使用案例的需求。它们不是强制性的。请注意，还可以配置轻量级目录访问协议 （LDAP） 来管理结构网络上的身份，适用于那些已经具有此实施且不希望向其现有基础架构添加身份管理层的企业。LDAP 有效地预注册目录的所有成员，并允许他们根据给定的条件进行注册。

**在生产网络中，建议为每个组织至少部署一个 CA 用于注册目的，另一个 CA 用于 TLS。**例如，如果部署三个与一个组织关联的对等节点和一个与排序组织关联的排序节点，则至少需要四个 CA。其中两个 CA 将用于对等组织（为对等方、管理员、通信和代表组织的 MSP 的文件夹结构生成注册和 TLS 证书），另外两个将用于排序者组织。请注意，用户通常只会向注册 CA 注册和注册，而节点将同时向注册 CA（节点将获取在尝试对其操作进行签名时标识它的签名证书）和 TLS CA（将获取用于验证其通信的 TLS 证书）进行注册和注册。

有关如何设置组织 CA 和 TLS CA 并注册其管理员身份的示例，请查看[结构 CA 部署指南](https://hyperledger-fabric-ca.readthedocs.io/en/latest/deployguide/ca-deploy.html)。部署指南使用 Fabric CA 客户端注册和注册设置 CA 时所需的身份。

部署生产 CA

- [规划 CA](https://hyperledger-fabric-ca.readthedocs.io/en/latest/deployguide/ca-deploy-topology.html)
- [生产 CA 服务器的清单](https://hyperledger-fabric-ca.readthedocs.io/en/latest/deployguide/ca-config.html)
- [CA 部署步骤](https://hyperledger-fabric-ca.readthedocs.io/en/latest/deployguide/cadeploy.html)



## 步骤 4：使用 CA 创建标识和 MSP

创建 CA 后，可以使用它们为与组织（由 MSP 表示）相关的标识和组件创建证书。对于每个组织，您至少需要：

- 注册并**注册管理员身份并创建 MSP**。创建将与组织关联的 CA 后，可以使用它首先注册用户，然后注册身份（生成网络上所有实体使用的证书对）。第一步，身份的用户名和密码由 CA 的管理员分配。还可以为标识提供属性和隶属关系（例如，组织管理员需要 of ）。注册标识后，可以使用用户名和密码进行注册。CA 将为此身份生成两个证书 — 网络其他成员已知的公共证书（也称为“签名证书”或“公共证书”）和用于对身份执行的操作进行签名的私钥（存储在文件夹中）。CA 还将生成一组称为“MSP”的文件夹，其中包含颁发证书的 CA 的公共证书和 CA 的信任根（这可能是也可能不是同一个 CA）。可以将此 MSP 视为定义与管理员标识关联的组织。如果组织的管理员也将是节点的管理员（这将是典型的），**则必须在创建节点的本地 MSP 之前创建组织管理员身份，因为在创建本地 MSP 时必须使用节点管理员的证书**。`role``admin``keystore`
- 注册**和注册节点标识**。正如注册并注册组织管理员身份一样，节点的身份必须同时注册 CA 和 TLS CA（后者生成用于保护通信的证书）并注册。不要为节点指定角色 或 在向注册 CA 注册节点时，为其指定角色 or 。与管理员一样，也可以分配此标识的属性和隶属关系。节点的 MSP 结构称为“本地 MSP”，因为分配给身份的权限仅在本地（节点）级别相关。此 MSP 在创建节点标识时创建，并在引导节点时使用。`admin``user``peer``orderer`

有关基于 Fabric 的区块链网络中的身份和权限的更多概念性信息，请参阅[身份](https://hyperledger-fabric.readthedocs.io/en/latest/identity/identity.html)和[成员资格服务提供程序 （MSP）。](https://hyperledger-fabric.readthedocs.io/en/latest/membership/membership.html)

有关如何使用 CA 注册和注册身份的详细信息（包括示例命令），请查看[向 CA 注册和注册身份](https://hyperledger-fabric-ca.readthedocs.io/en/latest/deployguide/use_CA.html)。



## 步骤 5：部署对等节点和排序节点

收集完所需的所有证书和 MSP 后，几乎可以创建节点了。如上所述，有许多有效的方法来部署节点。

在部署任何节点之前，必须自定义其配置文件。对于对等方，此文件称为 ，而排序节点的配置文件称为 。`core.yaml``orderer.yaml`

有三个主要选项可用于调整配置。

1. 编辑与二进制文件捆绑在一起的 YAML 文件。
2. 部署时使用环境变量覆盖。
3. 指定 CLI 命令上的标志。

选项 1 的优点是，只要您关闭和恢复节点，就会保留更改。缺点是，在升级到新的二进制版本时，必须将自定义的选项移植到新的 YAML（升级到新版本时应使用最新的 YAML）。

注意

您可以使用所有大写字母、相关短语之间的下划线和前缀，从相关 YAML 文件中的参数推断环境变量。例如，中称为的对等配置变量（这是配置部分中的变量）将呈现为名为 的环境变量，而配置文件部分中的排序服务环境变量将呈现为名为 的环境变量。`peer.localMSPid``localMSPid``peer``core.yaml``CORE_PEER_LOCALMSPID``General.LocalMSPID``General``orderer.yaml``ORDERER_GENERAL_LOCALMSPID`



### 创建对等方

如果您已经通读了对等方的关键概念主题，您应该对[对等方](https://hyperledger-fabric.readthedocs.io/en/latest/peers/peers.html)在网络中扮演的角色以及它们与其他网络组件交互的性质有一个很好的了解。对等方归属于频道成员的组织所有（因此，这些组织有时称为“对等组织”）。它们连接到排序服务和其他对等方，在其上安装了智能合约，并且是存储分类账的地方。

在创建对等方之前，了解这些角色非常重要，因为它们会影响您的自定义和部署决策。要查看需要做出的各种决策，请查看[规划生产对等方](https://hyperledger-fabric.readthedocs.io/en/latest/deploypeer/peerplan.html)。

对等方文件中的配置值必须自定义或使用环境变量覆盖。您可以在 [Hyperledger Fabric 的 sampleconfig 目录中](https://github.com/hyperledger/fabric/blob/main/sampleconfig/core.yaml)找到默认配置文件。此配置文件与对等映像捆绑在一起，也包含在可下载的二进制文件中。有关如何将生产与对等映像一起下载的信息，请查看[部署对等](https://hyperledger-fabric.readthedocs.io/en/latest/deploypeer/peerdeploy.html)。`core.yaml``core.yaml``core.yaml`

虽然默认参数很多，但您只需要自定义其中的一小部分。通常，如果不需要更改优化值，请保留默认值。`core.yaml`

在 中的参数中，有：`core.yaml`

- 标识符：这些**标识符**不仅包括相关本地 MSP 和传输层安全性 （TLS） 证书的路径，还包括对等方的名称（称为“对等 ID”）和拥有对等方组织的 MSP ID。
- 地址和路径：由于对等方本身不是实体，而是与其他对等方和组件交互，因此必须在配置中指定一系列**地址**。这些包括其他组件可以找到对等体本身的地址，以及可以找到链码的地址（如果您使用外部链码）。同样，您需要指定账本的位置（以及状态数据库类型）和外部构建器的路径（同样，如果您打算使用外部链码）。其中包括**操作和指标**，允许您设置通过配置终端节点来监控对等方运行状况和性能的方法。
- **八卦**：Fabric 网络中的组件使用“八卦”协议相互通信。通过此协议，它们可以被发现服务发现，并相互传播块和私有数据。请注意，八卦通信是使用 TLS 保护的。

有关及其特定参数的详细信息，请查看[生产对等方的清单](https://hyperledger-fabric.readthedocs.io/en/latest/deploypeer/peerchecklist.html)。`core.yaml`

当您熟悉对等方的配置方式、卷的挂载方式和后端配置时，可以运行命令来启动对等方（此命令将取决于后端配置）。

部署生产对等体

- [规划生产对等方](https://hyperledger-fabric.readthedocs.io/en/latest/deploypeer/peerplan.html)
- [生产对等方的清单](https://hyperledger-fabric.readthedocs.io/en/latest/deploypeer/peerchecklist.html)
- [部署对等方](https://hyperledger-fabric.readthedocs.io/en/latest/deploypeer/peerdeploy.html)



### 创建排序节点

注意：虽然可以向排序服务添加其他节点，但这些教程中仅介绍创建排序服务的过程。

如果您已经通读了订购服务的关键概念主题，您应该对排序[服务](https://hyperledger-fabric.readthedocs.io/en/latest/orderer/ordering_service.html)在网络中扮演的角色及其与其他网络组件交互的性质有一个很好的了解。排序服务负责将背书交易“排序”到区块中，然后对等方验证并提交到他们的分类账中。

在创建排序服务之前，了解这些角色非常重要，因为它会影响自定义和部署决策。对等和排序服务之间的主要区别之一是，在生产网络中，多个排序节点协同工作以形成通道的“排序服务”（这些节点也称为“同意者集”）。这会产生一系列需要在节点级别和集群级别做出的重要决策。其中一些集群决策不是在单个排序节点文件中做出的，而是在用于为应用程序通道生成创世块的文件中做出的。要查看您需要做出的各种决策，请查看[规划订购服务](https://hyperledger-fabric.readthedocs.io/en/latest/deployorderer/ordererplan.html)。`orderer.yaml``configtx.yaml`

排序节点文件中的配置值必须自定义或使用环境变量覆盖。您可以在 [Hyperledger Fabric 的 sampleconfig 目录中](https://github.com/hyperledger/fabric/blob/main/sampleconfig/orderer.yaml)找到默认配置文件。`orderer.yaml``orderer.yaml`

此配置文件与排序器映像捆绑在一起，也包含在可下载的二进制文件中。有关如何下载产品以及订购者映像的信息，请查看[部署订购服务](https://hyperledger-fabric.readthedocs.io/en/latest/deployorderer/ordererdeploy.html)。`orderer.yaml`

虽然默认参数很多，但您只需要自定义其中的一小部分。通常，如果不需要更改优化值，请保留默认值。`orderer.yaml`

在 中的参数中，有：`orderer.yaml`

- 标识符：这些**标识符**不仅包括相关本地 MSP 和传输层安全性 （TLS） 证书的路径，还包括拥有排序节点的组织的 MSP ID。
- 地址**和路径**：由于排序节点与其他组件交互，因此必须在配置中指定一系列地址。其中包括其他组件可以找到排序节点本身的地址，以及**操作和指标**，这些地址允许您设置通过配置端点来监视排序节点的运行状况和性能的方法。

有关及其特定参数的详细信息，请查看[生产订购节点的清单](https://hyperledger-fabric.readthedocs.io/en/latest/deployorderer/ordererchecklist.html)。`orderer.yaml`

注意：本教程假设在引导排序器时不会使用系统通道创世块。相反，这些节点（或其子集）将使用创建通道的过程加入到[通道](https://hyperledger-fabric.readthedocs.io/en/latest/create_channel/create_channel_participation.html)。有关如何创建将使用系统通道创世块引导的排序器的信息，请查看 Fabric v2.2 文档中的[部署排序服务](https://hyperledger-fabric.readthedocs.io/en/release-2.2/deployment_guide_overview.html)。

部署生产订购节点

- [规划订购服务](https://hyperledger-fabric.readthedocs.io/en/latest/deployorderer/ordererplan.html)
- [生产订购节点的清单](https://hyperledger-fabric.readthedocs.io/en/latest/deployorderer/ordererchecklist.html)
- [部署排序服务](https://hyperledger-fabric.readthedocs.io/en/latest/deployorderer/ordererdeploy.html)

## 后续步骤

区块链网络都是关于连接的，所以一旦你部署了节点，你显然会想要将它们连接到其他节点！如果您有对等组织和对等组织，则需要将组织加入联盟并加入或[创建频道](https://hyperledger-fabric.readthedocs.io/en/latest/create_channel/create_channel_participation.html)。如果您有排序节点，则需要将对等组织添加到您的联盟中。您还需要学习如何开发链码，您可以在主题开发应用程序/场景和[编写您的第一个链码](https://hyperledger-fabric.readthedocs.io/en/latest/chaincode4ade.html)中了解。

连接节点和创建通道的部分过程将涉及修改策略以适应业务网络的用例。有关策略的详细信息，请查看[策略](https://hyperledger-fabric.readthedocs.io/en/latest/policies/policies.html)。结构中的常见任务之一是编辑现有通道。有关该过程的教程，请查看[更新通道配置](https://hyperledger-fabric.readthedocs.io/en/latest/config_update.html)。一种流行的频道更新是将组织添加到现有频道。有关该特定流程的教程，请查看将[组织添加到频道](https://hyperledger-fabric.readthedocs.io/en/latest/channel_update_tutorial.html)。有关在部署节点后升级节点的信息，请查看[升级组件](https://hyperledger-fabric.readthedocs.io/en/latest/upgrading_your_components.html)。


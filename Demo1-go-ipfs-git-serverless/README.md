
![](http://daijiale-cn.oss-cn-hangzhou.aliyuncs.com/djl-blog-pic/ipfs/book/ipfs-git-arch.jpg)

## 版本信息：
 - git version 2.16.0
 - go-ipfs version 0.4.17

## 关键命令梳理：

```sh
$ cd Desktop
//--bare:不包含工作区，直接就是版本的内容
$ git clone --bare https://github.com/daijiale/ipfs-md-wiki
```

选取了一个之前托管在Github上的代码仓库`ipfs-md-wiki`作为本例中的迁移对象。

```
$ cd ipfs-md-wiki.git
$ git update-server-info
```

之后，我们打开git仓库对象包，通过将大的packfile分解成单独的对象，以便git仓库中存在多分支版本情况时，也能一一被IPFS网络识别并添加。

```sh
$ cp objects/pack/*.pack .
$ git unpack-objects < ./*.pack
$ rm ./*.pack
```


```sh
$ ipfs id
{
  "ID": "Qme...FZ",
  ...
}
$ ipfs daemon
$ ipfs add -r .
...
...
...
added QmS...ny ipfs-md-wiki.git

```

我们已经将`ipfs-md-wiki.git`添加到了本地IPFS文件仓库中，并获取其对应的CID信息：`QmS..ny `。

接下来，我们还需要做的就是将CID为`QmS..ny `的内容发布至IPFS网络中的更多节点之上，具体有如下两种方式：

**1.通过新节点pin add**

我们按照之前的方式，再部署一个新的IPFS节点，并启动daemon进程，通过`ipfs pin add QmS..ny`挂载一份Git Remote仓库服务：

```sh
$ ipfs id
{
  "ID": "Qmd...JW",
  ...
}
$ ipfs daemon
$ ipfs pin add QmS..ny
```

**2.利用公共网关**

```
https://cloudflare-ipfs.com/ipfs/QmS..ny

https://ipfs.io/ipfs/QmS..ny

https://ipfs.infura.io/ipfs/QmS..ny
```


当我们将git remote CID信息发布至多个IPFS网络节点后，我们可以通过`ipfs dht findprovs`根据CID信息来反向查询节点信息，从而验证一下Git Remote目前的分布式部署情况：

```
$ ipfs daemon
$ ipfs dht findprovs QmS...ny
Qme...FZ //本地节点id
Qmd...JW //pin节点id
QmS...hm//第三方网关节点id
```

现在，我们用Git工具，对刚才添加进IPFS网络中的Git Remote仓库进行clone操作：

```sh
$ git clone https://cloudflare-ipfs.com/ipfs/QmS...ny ipfs-md-wiki-repo
```

这是一个Go语言的程序块，执行的是导入依赖包的命令，通过本项目所搭建的Git分布式服务模型，用IPFS的CID指纹唯一标识了每个版本的Git源码库，可以规避一些变更风险，需要更新版本时，也可根据CID来自由切换、指定导入。


```go
import (
    "github.com/daijiale/ipfs-md-wiki"
)
```

```go
import (
    mylib "gateway.ipfs.io/ipfs/QmS...ny"
)
```


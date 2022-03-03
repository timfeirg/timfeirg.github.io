---
layout: post
title:  "在 Windows PowerShell 下安装 lain"
date:   2022-03-01 00:00:00
---

WSL 很好用, 但在我们团队的项目中, WSL 2.0 与 EasyConnect 这款垃圾软件不对付, 不少长期使用 Windows 的同事们, 唯一的选择便是在 PowerShell 下使用 lain. 好在 Python / Kubernetes / Helm 在 Windows 下的兼容性都非常好, 因此整个过程比较顺利, 因此在这里记录下安装流程, 以及沿途的一些最佳实践.

## 提前准备: 包管理器, 以及其他生产力工具

接下来要安装不少东西, 一个个下载安装包那可太原始了. 好在 Windows 世界已经出现了 [Chocolatey](https://chocolatey.org/install) 这种好东西, 途中所需的各种软件基本都可通过他来安装, 所以不要犹豫立刻拿去用吧.

无论哪个操作系统, 在 Shell 下切换目录是一个输入量非常大的事情, 一定要趁早优化掉. 在 Linux / Mac 上我一直用 [fasd](https://github.com/clvv/fasd), PowerShell 下我也找到了类似的工具: [z.ps](https://github.com/JannesMeyer/z.ps), 不过项目 README 里的安装流程有些欠维护, 已经有更轻松的安装方式了, 比如[这篇文章](https://githubhot.com/repo/badmotorfinger/z#the-easy-way-using-powershellget).

lain 往往是在企业内网维护发版的, 以 OpenVPN 为例, 也可以用 `choco install openvpn-connect` 来安装.

## 提前准备: lain 依赖的三方软件

以管理员模式打开 PowerShell, 接下来安装的各种东西或多或少都需要管理员权限.

首先[安装 Python Runtime](https://python-docs.readthedocs.io/en/latest/starting/install3/win.html): `choco install python`. 安装完以后请确认环境变量设置正确, 如果你不知道在 Windows 下如何修改环境变量, 随手一搜就有手把手教程.

* 将 Python Package 的可执行文件目录加入到 `$PATH`, 如果你没有使用 virtualenv, 那么这个路径一般是 `c:\users\$UESR\appdata\roaming\python\python310\Scripts`
* 添加环境变量 `PYTHONUTF8=1`, 中文 Windows 系统下默认编码是 GBK, lain 会报错

接下来安装 git, [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/#install-on-windows-using-chocolatey-or-scoop), [helm](https://helm.sh/docs/intro/install/#from-chocolatey-windows):

```powershell
choco install git
# client / server 的版本最好匹配上
choco install kubernetes-cli --version=1.20.4
choco install kubernetes-helm
```

其实完整使用 lain 还需要 docker, 这个情况稍微复杂一些. 长话短说, 不建议使用 docker-desktop, 而是尽量让服务器来帮你构建:

```powershell
# 由于是连接服务器的 docker daemon 做事情, 因此这里只需要安装 client 就够了
choco install docker
lain --remote-docker build --push
```

当然了, 更好的做法就是推到代码仓库, 让 CI 帮你做事情, 直接隔离各种本地环境问题. 不过如果你真的铁了心要在 Windows 本地做 `lain build`, 也可以 `choco install docker-desktop`, 只不过由于 Windows Docker 也是在用 WSL 虚拟机做事情, 出问题做排查的时候要额外注意, 确保虚拟机资源充裕.

这样一来, lain 所需的各种依赖软件就到位了.

## 安装 lain, 以及验证

从你的团队的内网 PyPI 上直接安装即可:

```powershell
pip -i https://pypi.example.com/pypi/simple/ install -U lain_cli
```

安装完毕以后, 用 [dummy](https://github.com/timfeirg/dummy) 来进行一番验证吧:

```powershell
git clone https://github.com/timfeirg/dummy
lain use test
cd dummy
lain init
lain secret show
lain --remote-docker build --push
lain deploy
```

这样一个流程跑通, 就代表 lain 可以在 PowerShell 下正常使用了, 再见.
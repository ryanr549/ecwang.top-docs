# **服务器概况**

我们的服务器是一组高性能计算集群(High Performance Cluster, HPC)，拥有1个管理节点(`manage`，20核，40线程)，以及4个计算节点(`node01-04`，56核×4)，没有GPU。计算节点的cpu算力比管理节点更高。合理利用多核能够显著提升科学计算的效率。

>管理节点开启了**超线程**，这意味着一颗核上可以有两个线程(thread)；在管理节点上使用`htop`等命令查看CPU使用情况时会看到40个核，这是**逻辑CPU核心数**，每个对应一个线程。而在4个计算节点上没有开启超线程，这是由于计算节点需要承担并行计算的任务，而超线程会影响并行计算的效率；在计算节点上使用`htop`可以看到56个核。

# 网络情况

服务器IP可以从公网访问，当前的ipv4地址为211.86.148.33。

不能保证未来IP地址是否会改变，我们申请了域名ecwang.top实时指向服务器的ip。因此，需要使用ip的地方一般可以用ecwang.top来替换。

# 数据存储

- Linux系统中，一切都是文件。
- `/`：根目录
	- `/home/<username>`：自己的家目录，也写作`~`
	- `/data/<username>`：自己的数据目录
	- `~/data`是`/data/<username>`的软链接(快捷方式)
	- `/data/public`：公共数据目录，把数据放在这里来共享给服务器上其他人。

>`/data`挂载在与根目录下其他路径不同的硬盘上：根目录其他路径挂载的硬盘总空间大概512GB，而`/data`挂载的硬盘总空间为256TB(1TB=1024GB)。所以数据请放在自己的`data`下。


# 远程登录

## SSH连接

无论使用什么系统，都可以很方便地通过SSH(Secure Shell)连接到服务器进行操作。

通过终端命令连接：
- Windows: Powershell
- Mac/Linux: 使用Terminal
```bash
ssh <username>@<IP>
```
输入以上命令，(随后输入密码和验证码)后即可进入命令行操作服务器的界面。

>此外也有一些SSH客户端程序可用：
>- Windows: [MobaXterm](https://mobaxterm.mobatek.net/)，[PuTTY](https://www.putty.org/)
>- [Termius](https://termius.com/)(全平台包含移动平台，不过需要注册)
>在ssh客户端中添加主机，指定ip地址、用户名和密码即可登录。

## SSH公钥认证

一般情况下每次登录都需要google authenticator。如果在本地终端生成密钥并上传至服务器，则可以省略这个步骤。

```bash
ssh-keygen -t rsa # 出现交互框按回车即可
ssh-copy-id <username>@<IP>
```

>这个过程实际上是把本机上生成的公钥`.../.ssh/id_rsa.pub`复制粘贴到服务器上`~/.ssh/authorized_keys`文件中，从而实现密钥验证。在一些SSH客户端中也能通过图形化界面完成这一过程。

## VNC远程桌面

下载vnc-viewer ([https://www.realvnc.com/en/connect/download/viewer/](https://www.realvnc.com/en/connect/download/viewer/))，不需要注册帐号，免费的功能足够了。

在服务器上运行如下命令：

```bash
run-vnc
```

>直接运行命令默认在60分钟后关闭vncserver，想要改变时长可以在命令后面带参数
>- `run-vnc 120`：开启120分钟后关闭
>- `run-vnc -1`：负数代表持续开启

一般会输出如下信息：

```text
============== vncserver started on port 1 =============
=== Run the following command in your local terminal ===
ssh -L 59000:localhost:5901 -C -N -l hryu 211.86.148.33
==== Then connect to localhost:59000 with vncviewer ====
=== Your vncserver is to be stopped after 60 minutes ===
```

这意味着你的vncserver在端口1启动。接下来复制刚刚输出的`ssh -L ...`一行，粘贴至本机的终端上运行(自己的电脑上，Powershell/terminal)。

>另外，如果桌面环境出现问题，可以运行命令强制关闭vncserver:
>`vncserver -kill :<port_id>`
>其中`port_id`是上面命令输出的端口编号。

然后在本机的vncViewer上操作，连接`localhost:59000`，密码为`17012ustc`。顺利的话直接能够进入桌面。找到xfceTerminal并打开，输入`ds9`即可运行fits图像浏览软件ds9，当然也能运行其他Linux系统下的图形化界面程序。

>如果自己的账户有正在运行的vncserver时，运行run-vnc不会开启一个新的进程，只是重新输出一次提示信息，类似这样：
>`A vncserver has already been running on port 1`
>这样是为了限制每个用户只能开一个vncserver。

# Python环境配置

Anaconda安装在`/data/software/miniconda3`以供所有用户使用。如果是新用户需要首先在命令行执行以下命令来完成conda初始化。

```bash
conda init bash
```

>[conda](https://conda.io/)是一个python环境管理工具，使用conda可以方便地建立python虚拟环境、安装python包并解决依赖问题。

conda初始化完成后重新登录SSH，你会发现命令行多出了一个`(base)`，这代表着你处于`base`环境下：**`base`环境是所有用户公用的固定环境，包含了基本使用所需要的一些包，你不能在此环境下安装新python包**。

>在`base`环境下请勿使用`pip install`安装python包：`pip`安装的默认位置为`~/.local/lib/pythonx.x/site-packages`，这与`conda`安装的位置不同，可能会导致奇怪的依赖问题。但是在自己创建的虚拟环境中用`pip install`就没问题了，后面会讲。

但是在实际工作时我们往往对环境有一些要求，从而需要安装其他版本的包。这时就需要创建虚拟环境：比如说我要创建一个名为"fitting"的虚拟环境，指定python版本为3.9：

```bash
conda create -n fitting python=3.9
```

>这里的`-n`是指定"name"，很形象。

创建完成后可以浏览当前可用的全部环境以及各自的位置，输出如下：

```bash
conda env list
# conda environments:
#
base                  * /data/software/miniconda3
fitting                 /home/<username>/.conda/envs/fitting
```

使用下面命令激活虚拟环境：

```bash
conda activate fitting
```

安装需要的包，用`conda`和`pip`都行，这时安装的包都会在你自己的家目录下，各个环境之间相互独立，也不会影响其他人。

```bash
conda install <pkg_name>
pip install <pkg_name>
```

>安装包时可以指定版本：
>`conda install <pkg_name>=x.x`，以及`pip install <pkg_name>==x.x`

退出虚拟环境(返回`base`环境)：

```bash
conda deactivate
```

如果虚拟环境出了问题，你可以直接删掉这个环境，结果就是删掉`/home/<username>/.conda/envs/fitting`这个文件夹：

```bash
conda remove -n fitting
```

## 配置Jupyter Notebook/Lab

>不论是对于Python初学者，还是熟练使用者，Jupyter都是一个好用的工具。Jupyter是一个基于iPython的交互式Python编程环境。能够直接显示画出的图让它在数据科学以及研究领域非常实用。

现在推荐优先使用Jupyter Lab。使用如下命令进行配置

```bash
jupyter server --generate-config # 在~/.jupyter/下生成配置文件
jupyter server password # 设置密码
cat ~/.jupyter/jupyter_server_config.json # 复制输出的password
vim jupyter_server_config.py # 用vim编辑配置文件
```

>vim怎么用？
>- 打开文件默认为`NORMAL`模式
>	- 光标移动：•左`h`,右`l`,上`k`,下`j`
>	- 翻页：`<ctrl>+f`向下翻页，`<ctrl>+b`向上翻页
>	- 输入/在文件中搜索：例如搜索字符串remote: `/remote<Enter>`，随后按`n`到下一处，`<shift>+n`到上一处
>- 按i进入`INSERT`模式，方向键可移动光标
>- 更改完文本后按`<esc>`返回正常模式，按`:`进入命令模式
>- 输入`wq<Enter>`，保存文件并退出。`w`为保存，`q`为退出

更改以下4处并取消注释(删除行首的`# `)
- `c.ServerApp.allow_remote_access = True`
- `c.ServerApp.password = ‘先前复制的password’`
- `c.ServerApp.port = 随便一个四位数，第一位不为0`
- `c.ServerApp.ip = ‘*’ 把引号内localhost换成星号`

配置完成，在服务器上启动jupyter lab后即可在远程浏览器上直接使用。

```bash
nohup jupyter lab & 
# 'nohup'将程序的输出保存到当前目录的下的nohup.out
# 命令末尾的'&'让程序在后台运行
```

接下来在浏览器输入网址`<IP>:<port>`，提示输入密码；用先前配置时自己设置的密码即可进入jupyter lab界面。

>Jupyter Lab是Jupyter Notebook的升级版本，提供了类似VSCode的插件功能。如果要使用Jupyter Notebook，则在前面配置的步骤中把'server'全都换成'notebook'，即从`jupyter notebook --generate-config`开始配置。

## 在Jupyter Lab/Notebook中使用虚拟环境

首先需要在虚拟环境(我们在这里假定虚拟环境的名字是"fitting")中安装`ipykernel`包：

```bash
conda activate fitting # 先进入fitting虚拟环境
conda install ipykernel # 再安装ipykernel包
```

接下来保证你在这个虚拟环境中，运行下面的命令

```bash
python -m ipykernel install --name fitting --display-name "Fitting_env"
```

这里`--name`后面跟虚拟环境的名字，`display-name`后面跟你想让他在图形界面显示的名字。

接下来就可以在运行你的jupyter kernel时选择这个虚拟环境了。

# 连接计算节点

在SSH登录管理节点之后再通过`ssh`命令来连接计算节点。

```bash
ssh node01 # 连接计算节点1，同理还可以连2, 3, 4
```

计算节点只能在管理节点的内网访问，因此不能一步连接到计算节点。

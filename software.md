# **服务器可用软件**

为了方便计算节点使用，软件都安装在`/data`路径下。

# Python/Anaconda
- 基本的开源软件。
- 路径：`/data/software/miniconda3`
- 公用环境：用户可以调用，不能更改
	- base：安装了Anaconda所有包的环境，可满足基本使用。
	- emcee：安装了emcee, corner, matplotlib, numpy以及其依赖，用来做MCMC模拟。
- 用户环境：

# MESA
- 模拟恒星结构演化的开源代码。
- 路径：`/data/software/mesa-r23.05.1`
- 使用前先将mesa\_sdk安装到自己的目录：
```bash
tar xvfz /data/public/pkgs/mesasdk_linux.tar.gz -C ~/
```
- 需要添加环境变量：在`~/.bashrc`中添加
```bash
export MESASDK_ROOT=~/mesasdk
export MESA_DIR="/data/software/mesa-r23.05.1"
export OMP_NUM_THREADS=20
source $MESASDK_ROOT/bin/mesasdk_init.sh
```
- 使用步骤，参考[官网](https://docs.mesastar.org/en/release-r23.05.1/using_mesa/running.html)
	- 复制`star/work`目录：`cp -r $MESA_DIR/star/work ~/data/mesa_work`
	- 编译：`cd ~/data/mesa_work && ./mk`
	- 编辑配置文件inlist，inlist\_project，inlist\_pgstar
	- ...
- 配套的读取mesa数据的python包，`/data/software/py_mesa_reader`

# IDL
- 商业软件，自己文章里能不能用？存疑
- 使用方法两种，命令行下输入`idl`即可进入类似IPython的交互式界面；[vnc连接远程桌面](./server_guide?id=vnc远程桌面)可以在xfce\_terminal中输入`idlde`可以进入idl的IDE。
- 安装了NASA的天文包(在`/data/software/IDLAstro`)，例如`mrdfits()`, `readfits()`等可以直接用

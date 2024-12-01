# 虚拟 Debian 环境

LAMDA 可以通过一个模块创建可在安卓内使用的完整 Debian 环境，你可以使用 apt 安装软件以及进行代码编译，同样，你可以在此环境中自行编译及使用 bpf 相关程序。你可以在 release 页面中找到 `lamda-mod-debian-arm64-v8a.tar.gz`（请根据你的机器架构下载对应的安装包）。
然后通过远程桌面或者 内置 adb 等方式，将 lamda-mod-debian-arm64-v8a.tar.gz 上传到设备，随后进行如下安装操作。

> 注：该 debian 环境只包含基础的软件包，你需要使用 apt 自行安装 git、python3 等常用命令。

```bash
# 切换到用户模块目录
cd /data/usr/modules
# 假设，该文件被你上传到了 /data/local/tmp
tar -xzf /data/local/tmp/lamda-mod-debian-arm64-v8a.tar.gz
# 解包完成
```

解包完成后，当前目录下将会存在一个 `debian` 目录，这时，你已经完成了基本的安装，下面介绍如何进入系统。

```bash
# 首先我们需要获知绝对目录，在以上情况下，绝对目录为 /data/usr/modules/debian
# 注意：每个根（debian 根系统）同时只支持一个终端实例使用
# 执行以下命令进入 debian interactive shell
debian /bin/bash
# 执行一次 id 命令
debian /bin/bash -c id
#
# 如果你并没有将模块安装于 /data/usr/modules，则需要指定模块位置
debian --root /path/to/debian /bin/bash
```

下面介绍进阶使用方法，你可以使用此 debian 环境自行建立一个 ssh 服务器，或者在此 debian 环境中运行 Python 脚本。
由于每个独立的 debian 环境只支持一个终端实例使用，我们建议用 ssh 的方式，这样，你可以接入多个 shell 到此 debian 环境。
什么是 `只支持一个终端实例使用`？就是当你执行 `debian /bin/bash` 后并保持使用状态，如果你在其他地方继续执行此命令，
该命令将会返回错误，使你无法再次进入此根系统，除非你将之前的 `debian /bin/bash` 退出。

现在，我们介绍如何在此 debian 环境中运行一个 ssh 服务以及安装基础的 Python 环境。

```bash
# 执行命令进入 debian shell
debian /bin/bash
# 现在，你应该在 debian shell 中，执行以下命令
root@localhost: apt update
root@localhost: apt install -y openssh-server procps python3 python3-pip python3-dev
root@localhost: echo 'PermitRootLogin yes' >>/etc/ssh/sshd_config
root@localhost: echo 'StrictModes no' >>/etc/ssh/sshd_config
root@localhost: mkdir -p /run/sshd
root@localhost: # 修改 root 密码
root@localhost: echo root:lamda|chpasswd
root@localhost: # 退出 debian 环境
root@localhost: exit
# 现在，你已经进入了 lamda 的 shell 环境，执行以下命令来启动 debian 环境中的 ssh 服务
debian /usr/sbin/sshd -D -e
```

如果你想此 debian ssh 环境随 lamda 一同启动，请执行 `crontab -e`，并在其中写下如下规则并重启即可。

```bash
@reboot debian /usr/sbin/sshd -D -e >/data/usr/sshd.log 2>&1
```

现在，获取该设备的 IP 地址，随后在你的电脑上执行如下命令并输入密码 lamda 即可登录该 debian shell

```bash
ssh root@192.168.x.x (手机IP)
```

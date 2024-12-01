# 安装内置 ADB 秘钥

无线连接内置的 WIFI ADB，此 ADB 非全功能 adb，仅支持 shell,pull,push,forward,reverse 等常用功能通过此功能你将**无需开启开发者模式**即可连接最高权限的 adb。

> 注：jdwp 调试相关功能具有唯一性，与系统内置存在冲突所以此 adb 目前不支持。

```python
# LAMDA 内置的 adb 服务完全独立于系统本身提供的 adb 服务
# 所以在使用之前需要先手动调用以下接口将你的 adb 公钥安装至设备上
# 否则直接连接将会显示未授权（系统设置开发者模式中授权的秘钥与内置 adb 并不通用）
#
# tools 目录下的 adb_pubkey.py 封装了下面接口的安装过程
# 你可以使用该脚本一键授权，允许本机连接，请查看其 README，以下代码仅做参考说明
#
# 这个秘钥文件位于你电脑上的 ~/.android 或者 C:\\Users\xxxx\.android，文件名为 adbkey.pub
# 如果不存在这个文件但是存在文件 adbkey，请切换到该目录并执行命令
# adb pubkey adbkey >adbkey.pub 来生成 adbkey.pub
#
# 随后使用 python 代码来拼接这个生成的 adbkey.pub 路径
import os
keypath = os.path.join("~", ".android", "adbkey.pub")
abs_keypath = os.path.expanduser(keypath)
print (abs_keypath)
#
# 然后安装这个 adbkey.pub 到 LAMDA
d.install_adb_pubkey(abs_keypath)
# 随后通过命令 adb connect 192.168.0.2:65000 连接设备
#
# 或者如果你需要从内置 adb 移除这个公钥
d.uninstall_adb_pubkey(abs_keypath)
```

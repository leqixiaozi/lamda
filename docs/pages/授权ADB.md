# 连接内置 ADB

用于安装本机的 ADB pubkey 到 LAMDA 的脚本，否则 ADB 连接将会显示未授权，这是使用 LAMDA 接口编写的脚本。

```bash
# 安装 adb pubkey
python3 -u adb_pubkey.py install 192.168.1.2
# 卸载 adb pubkey
python3 -u adb_pubkey.py uninstall 192.168.1.2
```

安装后，执行
```bash
adb kill-server
adb connect 192.168.1.2:65000
adb -s 192.168.1.2:65000 shell
```
来连接内置 ADB。

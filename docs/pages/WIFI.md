## WIFI 相关功能

目前WIFI操作部分功能由于可能导致设备异常未实现，仅介绍部分实现的功能

```python
wifi = d.stub("Wifi")

# 获取wifi bssid,ssid ip 等相关信息
wifi.status()

# 获取黑名单中的所有 bssid
wifi.blacklist_get_all()

# 将bssid加入黑名单(将不会显示在wifi列表)
wifi.blacklist_add("3c:06:aa:8a:55:66")

# 清空所有黑名单
wifi.blacklist_clear()

# 执行 wifi 扫描
wifi.scan()

# 获取周边 wifi 扫描结果
wifi.scan_results()

# 获取当前wifi的mac地址
wifi.get_mac_addr()

# 获取 wifi 信号强度，链接速率
wifi.signal_poll()
```

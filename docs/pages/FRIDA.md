# Frida Hook

## 通过代码使用

> **注意：** LAMDA 自 7.18 版本开始，内置 FRIDA 需要提供 token 参数才可连接，当然客户端库已经为你处理好了一切，如果你使用的是 7.18 之前的版本，请查看[旧版本的文档](https://github.com/rev1si0n/lamda/tree/5.0#连接内置的-frida)获取使用方法。

```python
# 使用 LAMDA 时的做法
device = d.frida
device.enumerate_processes()
```

等效原生代码

```python
# 获取 token，会动态变化请勿保存，每次需要连接时均需重新获取
token = d._get_session_token()

manager = frida.get_device_manager()
device = manager.add_remote_device("192.168.0.2:65000", token=token)
device.enumerate_processes()
```

## 通过命令使用

首先，你仍然需要先使用API获取当前 token，该 token 为固定 16 位字符串，例如 `czvpyqg82dk0xrnj`。

```python
token = d._get_session_token()
print (token)
```

对于所有 frida 官方命令行工具，加上参数 `-H 192.168.0.2:65000` 以及 `--token xxxxxxxxxxxxxxxx` 即可。

```bash
frida -H 192.168.0.2:65000 -f com.android.settings --token xxxxxxxxxxxxxxxx
# 如果服务端启动时使用了 certificate 选项，也需要在此加入 --certificate 参数
frida -H 192.168.0.2:65000 -f com.android.settings --certificate /path/to/lamda.pem --token xxxxxxxxxxxxxxxx
```

对于其他工具如 objection 等，通常也会有参数提供，这些参数都大同小异。

## 暴露应用接口

使用 FRIDA 暴露 Java 接口功能需要你能熟练编写 frida 脚本，tools 目录包含本次文内描述的脚本及工具。示例中使用的脚本为 test-fridarpc.js 文件，编程注意: frida 脚本中 `rpc.exports` 定义的函数参数以及返回值只能为任意 Javascript 中可以被JSON序列化的值。

执行以下命令注入 com.android.settings，注意查看是否有报错。

```bash
python3 fridarpc.py -f test-fridarpc.js -a com.android.settings -d 192.168.0.2
```

如果正常结束，那接口已经暴露出来了，现在请求 
```bash
http://192.168.0.2:65000/fridarpc/myRpcName/getMyString?args=["A","B"]
```
> 两个字符串参数 A，B 为注入脚本中的方法 `getMyString(paramA, paramB)` 的位置参数，注意参数的提供格式，不要手写或者字符串拼接参数，务必使用 `json.dumps(["A", "B"])` 生成。

即可得到方法的返回结果，接口同时支持 POST 以及 GET，参数列表也可以同时使用多个参数，空列表代表无参数，注意 args 参数序列化后的字符串最长不能超过 `32KB`，并且在使用了 --certificate 的情况下，链接需要改为 https 方式访问。

> 用 requests 调用
```python
import json
import requests
url = "http://192.168.0.2:65000/fridarpc/myRpcName/getMyString"
# 请求接口
res = requests.post(url, data={"args": json.dumps(["A","B"])})
print (res.status_code, res.json()["result"])

#* 状态码 200 一切正常
#* 状态码 410 需要重新注入脚本或者脚本未注入（目前不支持自动重新注入）
#* 状态码 500 脚本或参数异常
#* 状态码 400 参数错误
```

响应结果的格式是固定的，可在浏览器打开查看。

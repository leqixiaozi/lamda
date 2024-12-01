# OCR 自动化

## OCR 后端设置

> 目前OCR识别方式仅支持 检查元素是否存在、点击、截图、等操作。OCR 后端支持使用 paddleocr、easyocr 以及自定义 OCR 后端。


使用OCR识别方式前，需要先设置 OCR 后端，根据实际情况选择你需要的后端并提前安装依赖库。这里提醒如果你的情景是集群，即你需要在同一台电脑上进行多个设备的控制操作，请务必使用自定义后端，直接使用 paddleocr 或者 easyocr 会占用大量本机资源！


> 自定义OCR后端，主要用于大量设备控制或者本机无GPU加速等情况，可以将自己编写的OCR识别后端部署为HTTP服务，并在自定义后端内请求进行远程识别。

```python
import requests
from lamda.client import CustomOcrBackend

class MyCustomOcrBackend(object):
    def __init__(self, url, auth):
        self.url = url
        # 请在此 url 实现 OCR 识别接口
        self.auth = auth
    def ocr(self, image):
        # image 是图像的 binary data
        r = requests.post(url, headers={"X-Auth": self.auth}, data=image)
        # 你的接口，或者你应该把接口的返回值格式化成如下列表
        # 列表的每个元素为一个 3 个值的 tuple，其中第一位为识别区域，第二位是识别的文字，第三位为置信度
        # [([[189, 75], [469, 75], [469, 165], [189, 165]], '我的', 0.9754989504814148), ]
        return r.json()
```

初始化OCR识别

```python
# setup_ocr_backend 中的额外参数应为初始化该实例的参数
# 例如，你可以对比两个库在实际使用和作为 setup_ocr_backend 使用的参数之间的区别
paddleocr.PaddleOCR(use_gpu=True, drop_score=0.85, use_space_char=True)
easyocr.Reader(["ch_sim", "en"])

# 开始使用，根据实际情况选择，请提前安装相关依赖

# 使用 paddleocr 作为后端，截图用于识别的质量为80，使用GPU（有GPU）
d.setup_ocr_backend("paddleocr", quality=80, use_gpu=True, drop_score=0.85, use_space_char=True)
# 使用 paddleocr 作为后端，截图用于识别的质量为80，使用CPU（无GPU）
d.setup_ocr_backend("paddleocr", quality=80, use_gpu=False, drop_score=0.85, use_space_char=True)
# 使用 easyocr 作为后端，截图用于识别的质量为80，识别简体中文及英文
d.setup_ocr_backend("easyocr", ["ch_sim", "en"], quality=80)

# 使用远程的自定义OCR（以上面的 MyCustomOcrBackend 为例）
d.setup_ocr_backend(MyCustomOcrBackend, "http://my-backend.server/api/ocr", "mySecret")
```

## OCR 操作

```python
element = d.ocr(text="我的")
element = d.ocr(textContains="我的")
element = d.ocr(textMatches=".*?我的") # 正则

# 点击
element.click()
# 存在才点击
element.click_exists()

# 是否存在
element.exists()
# 截图
element.screenshot(100, ).save("element.png")
# 获取匹配信息
element.info()
```
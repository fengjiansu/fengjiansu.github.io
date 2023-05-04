# 用python实现天气预报助手

## 背景
今年天气变化莫测，一会儿30多度一会儿10几度，就想着要不写个自动脚本，监测明后两天是否有雨，或者气温是否极度变化，有的话就发个邮件通知下
## 思路
通过查询，了解到[和风天气api ](https://dev.qweather.com/docs/api/weather/weather-daily-forecast/)免费版本支持每日预报(3-7天)请求次数1000次对于个人用户远超所需了，因此只需要借助api获取返回的json文件，再根据获取到的信息进行处理即可
## 实现

### 获取天气
[和风天气API使用方法](https://dev.qweather.com/docs/api/weather/weather-daily-forecast/)
url为"https://devapi.qweather.com/v7/weather/3d"
参数只需确定城市[城市搜索服务](https://dev.qweather.com/docs/api/geoapi/)和key[如何获取你的key](https://dev.qweather.com/docs/configuration/project-and-key/)
通过python的request包(需要额外安装)的get方法得到


``` python
import requests

import json
def getWeather(url, params):
    response = requests.get(url, params, timeout=30)
    response.encoding = response.apparent_encoding
    html = response.text
    js = json.loads(html)
    daily = js["daily"]
    state1 = isSend(daily[0], daily[1])
    if ("True" in state1):
        send(state1["True"], "明天", daily[1])
        return

    state2 = isSend(daily[0], daily[2])
    if ("True" in state2):
        send(state2["True"], "明天", daily[2])
```

###  判断是否需要发送
初始化一个字典，通过setdefault()方法，返回键值如果键值不存在，插入键，且对应的value默认为None
判断温差，雨雪，风力，紫外线是否需要注意，是的话添加True的键值，并且对应的value要通过累加字符串的方式更新
``` python
def isSend(today, tomorrow):
    state = {}
    if (abs(int(today["tempMax"])-int(tomorrow["tempMax"])) > 6):
        state["True"] = state.setdefault("True", "") + "温度"
    if ("雨" in tomorrow["textDay"] or "雪" in tomorrow["textDay"] or "雨" in tomorrow["textNight"] or "雪" in tomorrow["textNight"]):
        state["True"] = state.setdefault("True", "") + "天气"
    if (int(tomorrow["windScaleDay"][0]) > 4):
        state["True"] = state.setdefault("True", "") + "风"
    if (int(tomorrow["uvIndex"]) > 4):
        state["True"] = state.setdefault("True", "") + "紫外线"
    else:
        state['False'] = "没有特殊情况"
    return state
```

### 发送信息

根据之前返回的四种情况加上提示语句

``` python
def getMessage(state, day):
    message = "您的天气小助手提醒您：\n"
    message += day
    if ("温度" in state):
        message += "的气温变化幅度大，记得增添衣服哟(提前准备好)\n"
    if ("天气" in state):
        message += "可能会出现雨雪天气，记得带伞\n"
    if ("风" in state):
        message += "风力过大,出门小心\n"
    if ("紫外线" in state):
        message += "紫外线强度过高，出门做好防晒\n"
    message += day+"具体天气如下：\n"
    return message
```
把获取到的天气转化为字符串，自己定义一个字典key_str key为天气里面的key，value则是对应的字符串
这样通过遍历key_str里的key值就可以把对应的字符串即相应的数值组合到一起
``` python
key_str = {
    "tempMax": "最高温度",
    "tempMin": "最低温度",
    "textDay": "白天天气",
    "textNight": "夜间天气",
    "windDirDay": "白天风向",
    "windScaleDay": "白天风力",
    "windSpeedDay": "白天风速",
    "windDirNight": "夜间风向",
    "windScaleNight": "夜间风力",
    "windSpeedNight": "夜间风速",
    "precip": "总降水量",
    "uvIndex": "紫外线强度指数",
    "humidity": "相对湿度",
    "vis": "能见度",

}
def wheather_msg(wheather):
    message = wheather["fxDate"]+"\n"
    for key in key_str.keys():
        key_str[key] += ":"+wheather[key]
        if ("温度" in key_str[key]):
            key_str[key] += "度"
        if ("风力" in key_str[key]):
            key_str[key] += "级"
        if (("风速" in key_str[key])):
            key_str[key] += "公里/h"
        if ("降水量" in key_str[key]):
            key_str[key] += "毫米"
        if ("能见度" in key_str[key]):
            key_str[key] += "公里"
        key_str[key] += "\n"
        message += key_str[key]
    return message
```

## 发邮件
需要QQ邮箱开通SMTP服务
```
from email.mime.text import MIMEText
import smtplib
import time
def sendEmail(message):
    msg_from = ''  # 发送方邮箱
    passwd = ''  # 发送方邮箱的授权码
    msg_to = ''  # 收件人邮箱

    subject = "天气预报"  # 主题
    msg = MIMEText(message)
    msg['Subject'] = subject
    msg['From'] = msg_from
    msg['To'] = msg_to
    try:
        s = smtplib.SMTP_SSL("smtp.qq.com", 465)  # 邮件服务器及端口号
        s.login(msg_from, passwd)
        s.sendmail(msg_from, msg_to, msg.as_string())
        print('[' + str(time.strftime('%Y-%m-%d %H:%M:%S',
                                      time.localtime(time.time()))) + "]邮件发送成功,邮件内容：" + message)
    except s.SMTPException:
        print('[' + str(time.strftime('%Y-%m-%d %H:%M:%S',
                                      time.localtime(time.time()))) + "]邮件发送失败,邮件内容：" + message)
    finally:
        s.quit()
```

[借助github的Action服务]
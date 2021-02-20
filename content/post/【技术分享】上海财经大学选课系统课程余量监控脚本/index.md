---
title: 【技术分享】上海财经大学选课系统课程余量监控脚本
subtitle: 最近正临选课，想选的课没选上，就写了一个课程余量监控的脚本，在有人退课的时候发送系统提醒，最后成功抢到课。这篇 blog 会分享这个课程余量监控脚本。
date: 2020-01-27T14:54:59.548Z
summary: "\n"
draft: false
featured: false
image:
  filename: featured
  focal_point: Smart
  preview_only: false
---
## 使用提示与重要声明

- <b>该脚本只是通过正常频率的模拟操作以实现课程容量监控的目的，调用的是上海财经大学教学管理信息系统的官方API，不对该系统进行任何形式的攻击与破坏；</b>
- <b>本脚本完全开源，且保证初始参数设置不会对服务器造成显著影响，请在使用脚本的过程中遵守中华人民共和国网络安全法和上海财经大学的相关规章制度，请勿通过高频率访问对服务器造成压力过载。因使用者滥用该脚本造成的一切责任，由使用者个人承担；</b>
- <b>为保证选课的公平、有序，本脚本不提供自动选课功能，仅提供余量监控与提醒功能，后续也不会更新提供抢课功能;</b>
- <b>Coockies 可能记录了你的用户ID和密码等隐私信息，请勿随意分享。因 coockies 随意分享造成的一切个人损失，由使用者个人承担。</b>

## 源代码
```python
import requests
import json
import time
from win10toast import ToastNotifier

# 输入你的设置
profileId = 0000 
wanted_course_ids = ['000000','000000','000000'] 
cookies_raw = '' 
request_frequence = 20 

COURSE_URL = "http://eams.sufe.edu.cn/eams/stdElectCourse!queryStdCount.action?profileId="

cookies={}
cookies_arr =  cookies_raw.split(';')
for ck in cookies_arr:
    name,value=ck.strip().split('=',1)
    cookies[name]=value

request_count = 0
while True:
    time.sleep(request_frequence)
    request_count += 1
    print("正在监控课程余量，已请求",request_count,"次")

    response = requests.get(COURSE_URL+str(profileId),cookies=cookies)
    student_count_text = response.text.split('window.lessonId2Counts=')[1]

    replace_list = ['sc', 'lc']
    for replace_text in replace_list:
        student_count_text = student_count_text.replace(
            '{}:'.format(replace_text), '"{}":'.format(replace_text))

    student_count_text = student_count_text.replace("'", '"')
    student_count_list = json.loads(student_count_text)

    for wanted_course_id in wanted_course_ids:
        course_count = student_count_list[wanted_course_id]
        if course_count['sc'] != course_count['lc']:
            toaster = ToastNotifier()
            toaster.show_toast(u'快抢课', u'课程有余量了！')
    break
```

## 使用提示
1. 确保 `requests、json、time、win10toast` 这四个包都安装好了，如果没有安装先 `pip install` ；
2. 设置参数 `profileId、wanted_course_ids、cookies_raw、request_frequence`:
    - `profileId` 是你的个人Id，可以进入选课系统从选课页面的链接上获取；
    - `wanted_course_ids` 是你希望检测课程余量的课程 Id，可以进入选课系统从课程介绍页面的链接上获取；
    - `coockies_raw` 是你的coockies，可以按F12从 Network 里的 Request Headers 获取,或者搜索如何获取 coockies，coockies 可能包含隐私数据，建议不要随意分享；
    - `request_frequence` 是你的请求频率，默认设置是20秒，选课系统本身的刷新频率为12秒左右，所以20秒请求一次完全不会对服务器造成影响，请勿过分修改该参数设置；
3. 如果当前监测课程有余量，则会发送系统通知进行提醒。该通知提醒目前还只在 Win10 上测试过，不一定有用。
![](https://raw.githubusercontent.com/JinChengneng/images/master/notification.png)

最后我靠着这个简单的小脚本一边打着王者荣耀一边抢到了最想要的课，祝大家也都选课顺利！因为抢课时间紧，这个小脚本大概只用了不到一个小时的时间完成，所以可能会有一些使用上的问题或者实现上不成熟的地方，欢迎多多交流！
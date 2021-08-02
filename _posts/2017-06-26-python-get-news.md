---
title: python科技新闻爬取
tags: 人生苦短，就用python
description: python自动爬取新闻发送到邮箱，每天自动获取早间新闻
---


从[python线上巡检](https://huhansome.github.io/_posts/2017-06-16-python-check-server/) 中尝到甜头之后，觉得python这门语言还真是实在，于是想了想，每天公交车上刷科技新闻，要是能主动把新闻整理好发送给我多好，于是撸起袖子就是干，搞了一个科技新闻爬虫。可以设置固定的时间去爬去，还可以自己写一些算法去筛选自己想要的新闻，代码简单，没有使用复杂的库，连bs都没用上。


代码实现：

```python
#请求readhub
def readhubRequest(url, params, headers = None, method = 'POST'):
    status_code = 0
    json = 'no json'
    method = method.upper()
    
    try:
        if len(url) == 0:
            return (status_code, json)
        if method == 'POST':
            if headers != None:
                resp = requests.post(url = url, params = params, headers = headers)
            else:
                resp = requests.post(url = url, params = params)
            status_code = resp.status_code
            json = resp.json()
        elif method == 'GET':
            if headers != None:
                resp = requests.get(url = url, params = params, headers = headers)
            else:
                resp = requests.get(url = url, params = params)
            status_code = resp.status_code
            json = resp.json()

    except Exception as e:
        print e
    #print json  #打印看看成果
    return (status_code, json)
    pass


#发送到邮箱 查看科技新闻吧
def sendmail(content):
    # 第三方 SMTP 服务
    mail_host = "smtp.qq.com"  #设置服务器
    mail_user = ""   #用户名,
    mail_pass = ""   #口令
    
    sender = '发送者'
    receivers = [] #接收者
    
    message = MIMEText('\n'.join(content), 'plain', 'utf-8')
    message['From'] = Header("发送人", 'utf-8')
    message['To'] =  Header("接收人", 'utf-8')
    
    subject = '科技新闻--Python爬虫'
    message['Subject'] = Header(subject, 'utf-8')
    
    try:
        smtpObj = smtplib.SMTP_SSL(mail_host, 465)
        #smtpObj.connect(mail_host, 25)
        smtpObj.login(mail_user, mail_pass)
        smtpObj.sendmail(sender, receivers, message.as_string())
    #print "发送成功"
    except smtplib.SMTPException as e:
        print e
    pass


def getnews():
    #请求数据,这里是从readhub爬取，可以换为今日头条什么的
    (code, json) = readhubRequest("https://api.readhub.me/topic", {"lastCursor" : "", "pageSize" : 20}, None, 'GET')
    #数据拿到了，整理一下，发送邮件或者干其他
    news = []
    if "data" in json:
        for new in json["data"]:
            if "title" in new:
                news.append(new["title"])
    #print news
    if (len(news) > 0):
        sendmail(news)
    pass
	
#schedule.py 时间脚本，控制爬取时间
schedulelist = [
                  {
                  "hour":00,
                  "minute":01,
                  "second":20
                  },
                  {
                  "hour":00,
                  "minute":02,
                  "second":20
                  },
                  {
                    "hour":9,
                    "minute":30,
                    "second":00
                  },
                  {
                  "hour":23,
                  "minute":59,
                  "second":30
                  }
                  ]

def addCount(count,total):
    count = count + 1
    if count == total:
        count = 0
    return count

def nextTime(item):
    curTime = datetime.now()
    hour = item["hour"]
    minute = item["minute"]
    second = item["second"]
    desTime = curTime.replace(hour = hour, minute = minute, second = second, microsecond = 0)
    return  desTime
    pass

def run():
    index = 0
    while True:
        try:
            curTime = datetime.now()
            
            total_count = len(schedulelist)
            
            item = schedulelist[index]
            #print "当前时间" + str(curTime)
  
            desTime = nextTime(item)
            delta = desTime - curTime
            skipSeconds =  delta.total_seconds()
            #print ("距离下次还有%d秒" % skipSeconds)
            
            if skipSeconds < 0 :
                #配置为明天第一个任务
                index = 0
                #今天任务做完，睡到第二天
                curTime = datetime.now()
                tmptime = curTime.replace(hour = 23, minute = 59, second = 59, microsecond = 0)
                skipSeconds = (tmptime - curTime).total_seconds()
                #print ("距离明天还有 %d 秒" % skipSeconds)
                
                item = schedulelist[index]
                desTimet = nextTime(item)
                #print "明天第一个任务的时间:" + str(desTimet)
                #print ("距离明天第一个任务还有 %d 秒" % (desTimet.hour * 3600 + desTimet.minute * 60 + desTimet.second))
                #print "要睡觉了"
                time.sleep(skipSeconds + 1)
                #print "不会到这来"
                continue
        
            #print ("skipSeconds = %d" % skipSeconds)
            time.sleep(skipSeconds)
            index = addCount(index,total_count)
            #print "这次任务已经完成,开始下个任务"
            #到点了，该做事了  0-6 星期一至星期日
            today = datetime.now().weekday()
            if (today == 5 or today == 6):
                #老子双休不干活
                pass
            else:
                #周末耍完了 上班了
                readhub.getnews()
                pass
        except Exception as e:
            print e
```

这只是相当初级的内容爬取，甚至连header都不用去模拟，更别说UA，IP限制等等
---
title: python自动化的使用 
tags: 人生苦短，就用python
description: python实现发送邮件，巡检线上运行状态
---

#### **背景**
之前在某公司，发现测试人员每天固定时间点都要发一些巡检报告，有些时间点很早，并且感觉时时都要知道线上情况，看着真的为他们感觉到累。于是写代码的就不安分了，这么无聊的重复性工作为啥要人来做，让代码去做不是很好吗？

#### **初窥python**
作为这么一门网红语言，不会真是有点说不过去。出于上面的需求，直接不管青红皂白就创建了一个python项目，虽然不知道python的相关api，但是程序思路是有的。大概思路就是写一个无限循环的程序，让他定时去check服务器的相关接口，然后检查接口的返回情况来决定服务器状态是否处于正常运行状态，至于check的频率以及返回情况的判断就和自己需求有关了，最后拿到结果可以用webhook的形式发送到钉钉群，或者用stmp发送邮件到相关人邮箱。

#### **技术实现**
本项目会用到python一些流行的三方库，至于三方库的安装，直接使用'pip install xxxx'即可，pip和iOS开发的cocoapods异曲同工（类似的安卓的gradle, 前端的npm，iOS的加特鸡什么的），有的库用pip安装不了的话，使用'easy_install xxxx' 即可。本次主要使用到requests框架，简直是python界的AFN，安卓界的OKHttp，功能作用强大到啥样子就不说了。
技术细节:
```python
#发送请求，检查接口是否正常
def my_request(url,params,headers=None,method='POST'):
    status_code = 0
    json ='no json'
    method = method.upper()

    try:
        if len(url) == 0:
            return (status_code, json)
        if method == 'POST':
           	if headers != None:
                req = requests.post(url=url, params=params,headers = headers)
            else:
                req = requests.post(url=url, params=params)
            status_code = req.status_code
            steplog(req.text)
            json = req.json()
        elif method == 'GET':
            if headers != None:
                req = requests.get(url=url, params=params, headers=headers)
            else:
                req = requests.get(url=url, params=params)

            status_code = req.status_code
            json = req.json()

    except Exception as e:
		#做异常处理
        #steplog(url+":网络请求异常\t" + str(e))
        #steplog("错误日志:"+EVERY_LOG)
    return (status_code, json)
    pass

#检查接口是否正常
def check_response(name,code,json):
    try:
        # server是否响应正常
        response_success = False
        # json返回结果
        if 'status' in json:
            status = int(json['status'])
        elif "STATUS" in json:
            status = int(json['STATUS'])
        else:
            status = 0

        if "ERROR_CODE" in json:
            error_code = json["ERROR_CODE"]
        else:
            error_code = -1;

        if code == 200:
            # server响应正常
            response_success = True
            pass

        # print(response_success)
        # print(status == 1)

        if response_success:

            if status == 1:
                return name + '正常'
                pass
            elif status == 2 and error_code == 0:
                tip = '【服务器响应正常，但接口做加密处理，无法读取json，请单独验证】'
                #steplog(tip)
                return name + '正常'
                # return name+'正常' +  "\t"+tip
                pass
            else:
                return name + '\t' + 'json数据返回异常'
                pass
            pass

        else:
            return name + '\t' + '服务器响应异常，错误码：' + str(code)

            pass

    except Exception as e:  # 如果 try 中的语句没有执行，则会执行下面的 ret=False
        #steplog(name+":异常\t" + str(e));

    pass

#发送到钉钉群
def sendDingDing(content,phone = None):

    url = '你的钉钉群webhook地址' #参见钉钉相关webhook接口说明
    headers = {
        "Content-Type": "application/json ;charset=utf-8 "
    }

    text_message = {
        "msgtype": "text",
        "text": {"content": content}
    }

    if phone != None:
        text_message["at"]= {
            #"atMobiles": [phone],
            #"isAtAll": False
        }
        pass


    text_message = json.dumps(text_message)
    res = requests.post(url, data=text_message, headers=headers)
    reponse_json = res.json()
    if reponse_json["errcode"] == 0 and reponse_json["errmsg"] == "ok" :
        print("钉钉消息发送成功")
    pass
	
	
#发送到邮箱
def sendmail(content):
    # 第三方 SMTP 服务
    mail_host = "smtp.qq.com"  #设置服务器
    mail_user = ""   #用户名
    mail_pass = ""   #口令,邮箱设置里面打开stmp之后可以设置一个口令
    
    sender = '发送者邮箱'
    receivers = [] #接收者邮箱
    
    message = MIMEText(content, 'plain', 'utf-8')
    message['From'] = Header("发送人", 'utf-8')
    message['To'] =  Header("接收人", 'utf-8')
    
    subject = '这里是邮箱标题'
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

```
#### **吐槽**
那个邮箱发送啊，试了好久一直报错，真的很心碎。原因好像之前把密码当口令使用，还有stmp端口使用错了。关于python 发送邮件[这里](http://www.runoob.com/python/python-email.html)更详细，关于端口，[这里](https://blog.csdn.net/zhangpan19910604/article/details/45065629)讲解也还不错.
如果发现大量重复性的没有多大意义的工作，要想办法用代码去做。~，毕竟人生苦短，不能浪费。

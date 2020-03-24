---
layout:     post
title:      "TelegramBot"
subtitle:   " \"Telegram Bot监控深度学习训练\""
date:       2019-08-04 21:51:00
author:     "HeShuai"
header-img: "img/banner/telegram_bot.png"
catalog: true
tags:
    - 瞎折腾
    - 深度学习
---

# Telegram bot实现的简单告警系统-训练监控
##### 0. 前言
 作为一个称职的人工稚嫩调参侠，需要时时把脉训练进程，确保喂进去的这剂参药管用，还要防止程序或者平台资源引起的猝死。Logger是一个好东西，但是实时性不够，如下班回家或者周末在家loss飞了或者程序死了都不知道。那我能不能雇佣一位管家，实时帮我传递最新消息，而且能够在出现异常的时候报告给我呢？
可以的。之前在家里路由用过[server酱](http://sc.ftqq.com/3.version)，可以通过微信进行相关信息报送，监控路由和联网设备的相关情况。但是有几个问题：
1. 首先server酱通过方糖公众号实现信息报送，但是使用受限。
2. 微信聊天记录的多设备云共享垃圾，且不保留全部历史信息，不方便后续和管理。
3. 微信不允许多设备共存登录，不方便多设备查看。
4. 微信不够geek。

作为日常在telegram搜集和分享小视频的油脂青年，**Telegram Bot**刚好可以完美弥补**微信+server酱**方案的缺点。

---
>**但是！！！**
>**由于telegram服务器在大环境下是不能访问的，因此以下所有操作都假设你能够科学接触太平洋彼岸。如果不能，请先学相关技能。**

---
##### 1. Telegram Bot
###### 1.1 Bot简介
[telegram bot](https://core.telegram.org/bots)翻译过来就是telegram机器人，是telegram平台上的虚拟用户，类似小冰或者siri。用户可以在telegram聊天窗口中，像@一个正常用户一样对bot发送各种指示和消息，bot则按照其内部规则进行回应。
> Bots are third-party applications that run inside Telegram. Users can interact with bots by sending them messages, commands and inline requests. You control your bots using HTTPS requests to our bot API.

Bot可以实现很多功能，如作为新闻管家给你推送相关信息，作为群管家进行群管理，还可以实现以inline的方式在对话框内实时反馈，如进行实时翻译、wiki查询和表情搜索等。官方有一些预设的bots，如Gmail Bot, Image Bot, GIF bot, IMDB bot, Wiki bot, Music bot, Youtube bot, GitHub bot等等等等。
![](https://raw.githubusercontent.com/mightycatty/mightycatty.github.io/master/img/20190730235909.png)

在与Bot交互中，存在三种关键不同身份：常规telegram用户，bot机器人和bot控制者。
>**常规telegram用户**：正在使用telegram聊天的任意自然人。在聊天窗口中与bot进行交互。<br>
>**Bot机器人本体**：所有bots集中存活在telegram的官方服务器。<br>
>**Bot控制者**：bot控制端，相当于bot的大脑，一般以服务的形式存在。bot所有的行为都在这里规定，决定了你的bot能实现什么功能。通过**https请求**的方式控制在telegram服务器上的bot机器人。

![](https://raw.githubusercontent.com/mightycatty/mightycatty.github.io/master/img/tele_bot.png)
上图为三者之间的沟通关系。一般用户在聊天窗口与bot机器人进行交互；bot机器人获知交互信息和记录它自身的当前状态；bot控制者，即bot大脑，以https请求的方式与bot机器人之间进行信息交互。控制者/大脑可以通过POST和GET来控制bot机器人。

###### 1.2 Bot控制

Bot的行为由bot控制者决定，可以内部自定各种规则，以实现不同的功能。Telegrame官方定义bot 控制者通过https请求的方式来实现bot机器人控制，支持常规的GET和POST操作。https请求本质上跟你每天访问google的请求是一样的，相当于向bot机器人发送一次指令。其格式是：

    https://api.telegram.org/bot<token>/METHOD_NAME

 token为你创建bot时的token，唯一指向需要控制的某一个bot机器人，METHOD_NAME则是控制方法，控制bot的动作，有一系列的官方预设的方法。在浏览器中输入控制网址，bot机器人会返回一个json作为回应。
 常用方法有以下，更多方法见[官方api说明](https://core.telegram.org/bots/api)：

| 方法        | 作用                                                         |
| ----------- | ------------------------------------------------------------ |
| getMe       | 获取机器人的标识信息                                         |
| getUpdates  | 获取机器人当前的状态，返回一个update实例列表；表明机器人当前跟谁在对话，对话的内容等等 |
| sendMessage | 向某个用户发送文字信息                                       |
| sendPhoto   | 向某个用户发送图片信息                                       |

---
##### 2. 实操

###### 2.1 交互逻辑
当你想实现bot与用户之间的双向交互时，你的Bot大脑需要保持24X7的运行，以保证能随时回应用户请求，因此需要一台云服务器运行你的大脑程序。我的需求只是实现一个传话管家，帮我时刻盯着训练程序状态，交互指向大多数情况下是bot->我，因此bot大脑只需附着于我的被监控代码中，必要时进行传话，不需要一个24X7的服务器。
前面blabla了一大堆，其实流程很简单：
![](https://raw.githubusercontent.com/mightycatty/mightycatty.github.io/master/img/tele_bot_mlserver.png)
###### 2.2 Bot创建和token/chat id获取
在利用BOT向你的telegram发送消息之前，你有两个关键前提需要：token和chat_id。

    TOKEN：创建你的bot时获取，唯一指向你的bot。
    Chat_id: 你创建的bot和某个用户的对话id，这里特指和你的对话窗口id。

TOKEN和chat_id的获取都很简单，移步[这里](https://www.assistanz.com/get-server-notification-telegram-app/)。

###### 2.3 快餐代码
 BOT担负的是实时远程logger的作用，因此可以和logger进行结合，下面是一份简单的快餐代码。
```python
"""
整合telegram bot的简单logger，可以实现本地文件log出，也可以实现通过bot向你的telegram发送实时message
使用：
1. 去telegram @botFather创建你的bot，获取token
2. 与你自己创建的bot发起对话，获取chat id
3. 实例化下面的MyLog，然后就ready to go了。
"""
import logging
import requests


class MyLog(object):
    def __init__(self, log_file='output.log', tele_bot_token=None, chat_id=None):
        # telegram bot token, 默认唯一指向你需要控制的bot
        if tele_bot_token is not None:
            self.token = tele_bot_token
        else:
            self.token = -- #你创建的bot token
        # bot与你的聊天的chat id，默认是你自己
        if chat_id is not None:
            self.chat_id = chat_id
        else:
            self.chat_id = -- # 你的bot与你之间的chat id
        # create logger
        self.logger = logging.getLogger(__name__)
        self.logger.setLevel(logging.DEBUG)
        self.log_file = log_file
        ch = logging.FileHandler(self.log_file)
        ch.setLevel(logging.DEBUG)
        # create formatter
        formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
        ch.setFormatter(formatter)
        self.logger.addHandler(ch)

    def debug(self, message):
        self.logger.debug(message)

    def info(self, message):
        self.logger.info(message)

    def warn(self, message):
        self.logger.warning(message)

    def error(self, message):
        self.logger.error(message)

    def clear_logfile(self):
        with open(self.log_file, 'w'):
            pass
        return
    @staticmethod
    def telegram_bot_send_text(token, chat_id, message):
        send_text = 'https://api.telegram.org/bot' + token + '/sendMessage?chat_id=' + chat_id + '&parse_mode=Markdown&text=' + message
        response = requests.get(send_text)
        return response.json()

    def fire_message_via_bot(self, message, chat_id=None):
        """
        向指定的chat id发送文字信息
        注意：由于采用markdown解析，字符串中不能出现markdown的语义符，否则报bad request错误
        :param message:
        :param chat_id:
        :return:
        """
        response = None
        if chat_id is None:
            chat_id = self.chat_id
        if (self.token is None) or (chat_id is None):
            self.logger.warning('token or chat_id is required!')
        else:
            response = MyLog.telegram_bot_send_text(self.token, chat_id=self.chat_id, message=message)
        return response


if __name__ == '__main__':
    logger = MyLog()
    logger.fire_message_via_bot('hello from other side')
```
可以看到其实代码很简单，关键的是<u>fire_message_via_bot</u>函数。该函数向你的telegram发送消息，你可以在你程序的任意合适的地方调用，如使用一般的logging方法一样。

当你按照2.2获取了token和chat id，运行上面的代码，你可以收到来自你的bot的亲切问候 ;-)
![](https://raw.githubusercontent.com/mightycatty/mightycatty.github.io/master/img/bot_test_sucess.png)

###### 2.4 Keras中监控训练进程
上面的代码只是一个简单的使用例子引导，更复杂的应用相信各位聪明的家伙能自己鼓捣出来的。这里再分享一个自己在keras中的应用。

众所周知，**keras**中的**callback**是极其重要的，其在每个epoch之后会进行调用，可以实现checkpoint\logging和其他操作。因此写一个TelegramBot callback就可以轻松监控每个epoch的训练数据。

~~煎饼果子~~代码快餐整一套：
```python
class TelegramBot(Callback):
    """
     通过telegram bot监控训练进程
     """
    def __init__(self, logger):
        super(TelegramBot, self).__init__()
        self.logger = logger

    def on_epoch_end(self, epoch, logs=None):
        try:
            model_name = self.model.name
            message = 'Model:{}\n epoch:{}\n'.format(model_name, epoch).replace('_', '-')
            for key, value in logs.items():
                message += '{}:{}\n'.format(key, round(float(value), 2)).replace('_', '-')
            result = self.logger.fire_message_via_bot(message)
            print (message)
            print(result)
        except Exception as e:
            print('error with Telegram bot callback:{}'.format(e))
```
作为callback送入你的model compile中。训练一个epoch之后，你又被你的bot问候了:-)
![](https://raw.githubusercontent.com/mightycatty/mightycatty.github.io/master/img/bot_callback.png)

##### 3. 后话与TODOs
上述只是一个简单的传话管家，把服务器中的训练进展通过telegram实时送到你脸上。在交互形式上是单向的。事实上，可以拓展为双向。
双向的Bot可以实现以下功能：

>1. 实时监控训练进展，精准把脉。
>2. 利用keras callback，可以在不需重启训练程序的情况下，通过telegram远程调整训练超参，如lr等。
>3. 更复杂的自定义的双向交互。

要实现双工通讯，你的Bot需要：
>1. 独立的进程，随时监控用户发来的讯息。
>2. 双向交互的handle定义。

从0开始写一个双向BOT会重复造很多轮子，因此需要借助一些优秀的开源工具：
>[python-telegram-bot](https://github.com/python-telegram-bot/python-telegram-bot)

>[pyTelegramBotAPI](https://github.com/eternnoir/pyTelegramBotAPI)

上述为两个python封装的telegram api，可以实现很复杂的BOT构建，具体和进阶应用可以查看其提供的文档。后续我会试过自己搞一个，彼时有进展了再更新此文。
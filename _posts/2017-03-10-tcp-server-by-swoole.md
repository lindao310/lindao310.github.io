---
layout: post
title:  "基于swoole开发的一个长连接项目"
date:   2017-03-10 15:14:54
categories: php
tags: swoole 长连接
---

* content
{:toc}


### 为什么会有这个东西

- 部门打算做类似狼人杀这种社交游戏，领导要求我们先做个DEMO，技术们先练练手， 于是就有了这个猜歌游戏 --- 大家进入一个房间，游戏开始播放10首歌，每首歌在播放的时候大家猜歌名，根据猜的正确与否及先后顺序计分，每首歌间歇可以使用道具，游戏结束要展示结果报告并对失败者做惩罚。。。。

### 技术选型

- 一般这种带聊天和即时推送的项目方案大体3个，轮训、long-polling、长连接（我理解的叫持久连接更准确点），我决定做长连接，不就弄个tcp server， 然后定好消息协议吗， 写了好几年api了， 还是做点新鲜的事情吧

- 第一个想到的是openresty，我本人十分佩服和仰慕章亦春的， 一直有志于学习和使用openresty。 看了下这个[https://github.com/openresty/stream-lua-nginx-module](https://github.com/openresty/stream-lua-nginx-module),  觉得以我目前的知识储备，驾驭起来没有把握； 而这个DEMO是个涉及部门转型的大事，2周内必须做出来， 所以放弃了

- 然后周末花了2天时间看了下go(需求是周三到我这边的，周四周五主要理解需求，罗列功能点）， 我的自信也受了点打击，觉得go这个到处都是 `err ！=nil` 实在太丑了。。。然后不管是变量的声明初始化，函数的声明，参数都太多了，我觉得很不好理解，都没法一眼看懂，还必须的脑子里绕一下。。。感觉用这个就更没把握了

- 所以最后还是用已经比较熟悉的swoole来做这个事情呢

### 消息格式
- 只是个内部演示的DEMO，本着一切从简的原则，数据结构就定成文本格式的json_encode 串来做，例如 `["{$cmd}", {"param1":"","param2":""}] `, 包含2个元素的list，第一个元素是字符串，表示cmd， 第二个参数是个map，表示该命令需要的参数
 
- 由于TCP是个面向流的协议，它的一个传输单元不一定刚好是一个完整的消息帧，在开发完联调的时候，ios就发现了这个粘包问题，对于这个问题，解决方案通常2个
	- 约定开始结束符
	- 自定义包头和包体， 比如常用的就是设定前4个字节是包头，从包头里读取包的长度信息，然后从tcp conn 按长度读具体消息帧
	
	
### Ios开发遇到的消息边界问题
- 开发完是跟安卓联调了2天，ios才开始接，然后讨论组里ios说他解析不了数据让我改，我想安卓联调2天都没问题啊， 就回他换个姿势试试，他就很不爽的反问我‘啥姿势能满足你，你数据格式有问题，赶紧改吧“。
- 真是气不打一处来啊， 你啥证据不提供，直接给结论 "数据格式有问题"就不合理； 然后就这个问题来讲，你从socket read出来的东西解析不出来，起码给我看下你read到的内容是啥吧； 
- 我让他打印出来看了下， 发现我推送的2条消息粘一起呢， 这个显然就是大家所说的粘包问题。我不开心，所以没跟他解释。。。
- 然后到了第二天他才截了个网页，发个粘包的解释出来，说可能是粘包了。。。原来他不知道这个啊
- 我平复了下心情， 过去跟他讲怎么回事&解决方案，约定结束符你需要如何如何读数据，自动义长度包头你要如何处理 。。。然后他站起来理直气壮的怼了我， ”你的问题你让我改底层，ios就我一个人啊，哪有时间弄啊。。。", 我被`改底层`这3个字彻底击溃。。。
- tcp是字节流的形式，没有边界的。边界问题都是通过应用层自己编码处理的哈。 我觉得你之前不知道粘包这个概念我还勉强能理解，没学过嘛。 但是你都上网查了大半天，然后我跟你讲具体逻辑打算让你选哪个办法解决的时候你竟然完全听不懂，我只能心里默念"这个**是怎么进公司的"...

### 争论经验总结
- 要尽量选择跟聪明人一起工作； 
- 不善言辞，沉默寡言，没有威信很容易被怼啊。 
- 2个条件都不符合，遇到这种问题真是没处说理啊。
- 我要多努力系统的学习计算机基础知识； 因为tcp这个我也是去年因为查问题才学的，不然我也不知道要处理这个边界问题

### 服务端实现

- 这个基于swoole开发的聊天室肯定有现成的， 网上找了下， 最后决定用这个[https://github.com/hsinlu/swoole-chatroom](https://github.com/hsinlu/swoole-chatroom) ，在这个基础上按我们的需求进行改造

- 房间的概念，没有离线消息， swoole_table 结构太单一了，改用redis来存储数据

- 增加定时器，workStart回调的时候一个，定时输出当前server status； 房间达到人数和每轮结束后自动开始下一轮

```php

swoole_timer_tick(10000, function($timer_id) use ($app, $server, $fd) {
            echo "监听是否新一轮游戏开始 " . date("Y-m-d H:i:s")." \n";
            $checkRoomDoing = $app->room->checkRoomDoing();
            if ($checkRoomDoing) {
                swoole_timer_clear($timer_id);
                echo "游戏已经开始 清空计时器 " . date("Y-m-d H:i:s")." \n";
                return ;
            }

            $lastMsgTime = $app->room->getRoomLastMsgTime();
            $nums = $app->users->getCount();

            if ((time() - $lastMsgTime) > TIMER_NEXT_ROUND && $nums >= GAME_START_NUMBERS) {
                //符合条件的话，清除定时器并开始下一轮游戏
                swoole_timer_clear($timer_id);

                room($server, $fd, '房间内已经30秒没人说话，开始新一轮游戏', REFEREE, 'common');

                echo "游戏开始2 " . date("Y-m-d H:i:s")." \n";
                $round = $app->room->generateRoundID();
                $app->room->setRoomStatus(2); //设置房间为正在游戏中
                room($server, $fd, $round, REFEREE, 'game_start');
            }
        });
        
        
```

- server增加shutdown回调，需要 `kill -15 pid ` 才能触发这个回调， 用来清理房间信息，方便联调

- server 增加start 回调， 通过`cli_set_process_title('swoole-chatroom');` 设置进行名字，再写个shell 脚本来完成重新启动工作


```sh
 #!/bin/bash
 # 拉取最新代码并重启服务
 # wu.daolin
 # 2017-03-20

 date "+%Y-%m-%d %H:%M:%S";

 path='/home/wu.daolin/swoole-chatroom';
 branch_name='master';
 process_name='swoole-chatroom';

 cd ${path};

 # 拉取新代码
 git pull origin ${branch_name};

 # 停掉服务
 pid=`/sbin/pidof $process_name`
 echo -e "$process_name pid: $pid\n\n";

 kill -15 $pid;

 echo -e "$process_name stopped！！！！\n";
 #启动服务
 php serve.php -d;

```
 
### 遇到的难关
 
 在联调的第二天我们的客户端反馈这个socket连接经常断掉， 我想我设置的是60s检查一次，如果当前连接180s没有收到数据就主动断开连接啊
 
 - 我加了日志看了下，确实是完全没有规律的就Closed , 从连上几秒到3分钟都有

 - 确定有问题，先在自己的电脑上启动server，让他们连我的电脑联调；然后我开始艰难的排查这个按常理不该出现的问题。。。 
 
 - 怀疑我们的办公网不稳定，我启动server后，从本机，跳板机，自己的电脑同时connect server 观察，发现都是会断，而且是同一时间

 - 既然是同一时间，而且本地也会断，就不是办公网的问题了， 怀疑是server有问题，比如swoole 出现异常，worker重启了之类的， 增加的register_shutdown_function ， 也没捕获到任何问题

 - tcpdump 抓了下包再看看, 看第四行是server主动断的，应该是server端出了问题

```sh
 
 17:37:52.722906 IP 172.16.68.10.57652 > vm-game-dev002.vm.momo.com. 19535: Flags [S], seq 2383924617, win 65535, options [mss   1460,nop,wscale 5,nop,nop,TS val 720270677 ecr 0,sackOK,eol], length 0

 17:37:52.722928 IP vm-game-dev002.vm.momo.com.19535 > 172.16.68.10.57652: Flags [S.], seq 3377700123, ack 2383924618, win 14600, options [mss 1460,nop,nop,sackOK,nop,wscale 9], length 0

 17:37:52.744399 IP 172.16.68.10.57652 > vm-game-dev002.vm.momo.com.19535: Flags [.], ack 1, win 8192, length 0

 17:39:15.997490 IP vm-game-dev002.vm.momo.com.19535 > 172.16.68.10.57652: Flags [F.], seq 11, ack 11, win 29, length 0

 17:39:16.029284 IP 172.16.68.10.57652 > vm-game-dev002.vm.momo.com.19535: Flags [.], ack 12, win 8192, length 0

 17:39:16.029462 IP 172.16.68.10.57652 > vm-game-dev002.vm.momo.com.19535: Flags [F.], seq 11, ack 12, win 8192, length 0

 17:39:16.029475 IP vm-game-dev002.vm.momo.com.19535 > 172.16.68.10.57652: Flags [.], ack 12, win 29, length 0

```
 
- 再试， 如果swoole没问题的话，可能是这个机器有问题，用stream_socket_server 在server上写个 server 做实验， 结果没有问题

- 现在只剩下2个办法了，一个是重装或者升级swoole， 另一个是找sa申请换台测试服务器

- 还好，我把swoole升级到1.9.6之后，这个问题好了


### 其他总结

- 维持长连接是很耗资源的，而且这个socket谁都能连， 所以通常的策略你连上之后n秒内要发送鉴权信息，否则server要主动断开
- 如果client长时间没有发消息，server也应该主动断掉
- client如果发送不合法的消息内容，应该主动断掉
- 长连接的地址应该通过api下发，域名优先（最好区分运营商），ip地址也最好有，防止域名劫持
- 长连接理论上就是很不稳定，很容易因为各种网络&设备的原因断掉的， 所以很多业务的数据还是应该走http；tcp长连负责推送简短的通知比较好
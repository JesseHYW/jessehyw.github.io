---
layout: post
title: 小爱音箱接入ChatGPT
categories: [技术, ChatGPT, 小米]
description: 尝试将小爱音响接入chatgpt
keywords: blog, 小爱, chatgpt
---

前几天偶然在[小众软件](https://meta.appinn.net/)公众号看见了 `xiaogpt` 项目的推荐推文。扫了一眼感觉部署难度不大，加上自己和室友也都还挺有兴趣的，今天（周五）就立马着手配置。

项目所需：（预先下载或 fork 前两个项目）
>+ [xiaogpt - 让小爱接入GPT的主要部件](https://github.com/yihong0618/xiaogpt)
>+ [MiService - 获取小爱音箱的设备ID](https://github.com/yihong0618/MiService)
>+ Python 3.8+
>+ ChatGPT 账号 （有API更好）
>+ 小爱音箱（of course）

以下操作均在 Windows 11 环境下进行。

### 配置环境变量
有两种方法可以添加我们所需的环境变量，在任何项目中都通用。
- 在 cmd 里运行以下命令：
```
set MI_USER=<Username>
set MI_PASS=<Password>
set MI_DID=<DID>
set OPENAI_API_KEY=<api>
```
其中 `<Username>` 和 `<Password>` 替换成你的小米账号和密码。`<DID>` 在后文会提到如何获取，可以稍后再来配置它。`<api>` 就是你的 OpenAI API，如果没有就忽略这条。

---

- 打开 `系统设置-系统信息-高级系统设置-环境变量`
在 用户变量 或 系统变量 任一分类中，新建以下变量：
```
1. 变量名 MI_USER 变量值 <Username>
2. 变量名 MI_PASS 变量值 <Password>
3. 变量名 MI_DID 变量值 <DID>
4. 变量名 OPENAI_API_KEY 变量值 <api>
```
变量值内容同上。

### 安装依赖库
在 `MiService` 的项目根目录打开 cmd，运行命令：`pip3 install .`

### 获取音箱设备ID（DID）
在同上位置运行命令：`micli.py list`

在输出内容中找到你要配置的音箱，记下它的信息，形如以下内容：
```
"name": "小米AI音箱",
"model": "xiaomi.wifispeaker.s12",
"did": "123456",
"token": "1145151237xjse00112233j33"
```
再将 did 加入环境变量，方法见上文。
> 这里我碰见了整个部署过程中遇到的最大的问题。
当我尝试在 cmd 里运行这一命令时，出现了以下报错：
```
D:\Github\MiService-main>
[main 2023-04-07T07:16:10.277Z] [SharedProcess] using utility process
[main 2023-04-07T07:16:10.418Z] update#setState idle
[main 2023-04-07T07:16:11.575Z] [UtilityProcess type: shared-process, pid: <none>]: creating new...
[main 2023-04-07T07:16:11.579Z] [UtilityProcess id: 1, type: fileWatcher, pid: <none>]: creating new...
[main 2023-04-07T07:16:11.586Z] [UtilityProcess id: 1, type: extensionHost, pid: <none>]: creating new...
[main 2023-04-07T07:16:11.596Z] [UtilityProcess type: shared-process, pid: 16240]: successfully created
[main 2023-04-07T07:16:11.617Z] [UtilityProcess id: 1, type: fileWatcher, pid: 896]: successfully created
[main 2023-04-07T07:16:11.652Z] [UtilityProcess id: 1, type: extensionHost, pid: 14276]: successfully created
[main 2023-04-07T07:16:40.424Z] update#setState checking for updates
[main 2023-04-07T07:16:40.542Z] update#setState idle
[4756:0407/152247.433:ERROR:gpu_init.cc(481)] Passthrough is not supported, GL is disabled, ANGLE is
```
> 通过提问 new bing 以及自行google，我发现问题大概是 chromedriver 的调用参数有问题造成的。然而由于技术有限，我实在是找不出解决办法。折腾了半天，我决定换个角度来处理。   
于是我打开了 **VS Code**， 用里面的终端来执行这一命令。出乎意料，程序完美地跑了起来。希望能有大佬解释一下这是为什么😂（因为毫无头绪差点打算放弃的我：呼~）   
**注意**：在 VS Code 中运行这一命令时，不需要 .py 后缀，即 `micli list` 即可。

### 获取音箱的硬件型号
- 直接看你的音箱的屁股/底下。找到形如 `LX06` `S12` `X08E` 等的标识就是型号了。
- 或者使用 `micli mina` 命令来查询：
运行命令后，在输出内容里找到你要配置的音箱名称，再往下几行找到形如 `"hardware": "S12"` 的参数，后面引号的内容就是我们需要的硬件型号，记录下来。

### 运行 xiaogpt
在 xiaogpt 根目录下运行命令：
`xiaogpt --hardware <hardware>`   
或 `xiaogpt --hardware <hardware> --use_chatgpt_api` （若有API）   
`<hardware>` 替换为刚才获得的硬件型号。

如果配置无误，终端会显示：
```
Running xiaogpt now, 用`帮我/请回答`开头来提问
或用`开始持续对话`开始持续对话
```
此时，你就可以开始愉快地用小爱与 ChatGPT 对话了！注意开头要加上提示词，并且电脑要保持 VPN 连接、终端持续运行。

> 命令中的其它常用参数：
>- `--mute_xiaoai` - 单次禁用小爱自己的回答
>- `--use_gpt3` - 使用 GPT-3 引擎
>- `--enable_edge_tts` - 使用 Edge 浏览器的语音合成引擎
>- `--stream` - 使用流式响应，加快响应速度（效果不明）
>- `--config xiao_config.json` - 使用配置文件启动（需要先修改目录中的 json 配置文件，实现一次做好设定，之后一键使用）

### 后记
这个项目部署难度不大，除了中途碰见的 cmd 里的错误之外，全程基本上没有遇见什么阻碍。只可惜我没有可用的 API ，导致 GPT 的反应比较慢。但用来娱乐一下，问一些有的没的，这个程度也差不多了。项目原理感觉有点意思，不需要申请小米官方的开发者平台，就可以实现命令行控制小爱音箱，`MiService` 还是挺牛的。

之前就看见大佬把 Siri 和 ChatGPT 连接了起来，这次又亲手实现了小爱和 GPT 的对接。总给我一种感觉，就是科幻电影里，那种又能做事又能聊天、上知天文下知地理的高智能 AI 助理已在眼前。

What will come next?

> 附：项目作者录制的 [部署全流程](https://www.youtube.com/watch?v=K4YA8YwzOOA) (Youtube)
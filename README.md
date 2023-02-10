# Info

> 基于 ChatGPT 的微信聊天机器人，通过 [OpenAI] 接口生成对话内容，实现微信消息的接收和自动回复。已实现的特性如下：

- [x] **文本对话：** 接收私聊及群组中的微信消息，使用 ChatGPT 生成回复内容，完成自动回复
- [x] **规则定制化：** 支持私聊中按指定规则触发自动回复，支持对群组设置自动回复白名单
- [x] **图片生成：** 支持根据描述生成图片，并自动发送至个人聊天或群聊
- [x] **上下文记忆**：支持多轮对话记忆，且为每个好友维护独立的上下会话

>引入 [itchat-uos](https://github.com/why2lyj/ItChat-UOS)，解决由于不能登录网页微信而无法使用的问题，解决 **Python3.9** 的兼容问题;
支持根据描述生成图片并发送，**OpenAI** 版本需大于 **0.25.0**；
建议 **Python** 版本在 **3.7.1~3.9.X** 之间，**3.10** 及以上版本在 **MacOS** 可用，其他系统上不确定能否正常运行；
**itchat-uos** 使用指定版本 **1.5.0.dev0**。

---

# 部署

## 注册账号

- 登录 [OpenAI 注册页面](https://beta.openai.com/signup) 注册账号，可参考[教程（需要连接网络）](https://mirror.xyz/0x8869a2E79c1A792fD4f3c041978568aDd4D20857/5HtM3r8395wzdbxFh4ayhxiqPuGqIEkvgTwCWcMzYXQ)并通过 5Sim 虚拟手机号平台来接收验证码，最好选英国跟荷兰的号码。
- 完成注册账号后登入 [API 管理页面](https://beta.openai.com/account/api-keys) 创建一个 API Key 并保存下来，后面需要在根目录的文件 `config.json` 配置这个 key。


## 服务器准备

- 支持 Windows / MacOS / Linux 系统（可在 Linux 服务器上长期运行)，同时需安装 `Python`、


- 测试所使用的是 [DigitalOcean](https://m.do.co/c/9de664fa6fad) 的服务器，配置如下：


<div align="center">
	<img src="/../main/doc/image/digitalocean.png" alt="Editor" width="700">
</div>

```bash
Singapore                                  # 区域
Ubuntu 22.10 x64                           # image OS 系统
Basic                                      # Droplet Type
Regular 512M/1CPU/10GB SSD/500GB  $4/mo    # CPU options
```

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.cdn.digitaloceanspaces.com/WWW/Badge%201.svg)](https://www.digitalocean.com/?refcode=9de664fa6fad&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)

## 开始安装

- 克隆项目代码
```bash
git clone https://github.com/0xXPunkX/wechatgpt
cd wechatgpt
```

- 重装 pip，看自己需求情况，不过建议先跑一遍
```bash
sudo apt-get remove python3-pip
sudo apt-get install python3-pip
```

- 安装依赖 itchat
```bash
pip3 install itchat-uos==1.5.0.dev0
```

- 一般情况下需要 upgrade
```bash
python3 -m pip install --upgrade pip
```

- 安装依赖 openai
```bash
pip3 install --upgrade openai
```


## 配置

配置文件的模板在根目录的`config-template.json`中，通过以下命令复制该模板创建最终生效的 `config.json` 文件：

```bash
cp config-template.json config.json

vi config.json
```

在`config.json`中填入配置，以下是对默认配置的说明，可根据需要进行自定义修改：

```bash
# config.json文件内容示例
{ 
  "open_ai_api_key": "YOUR API KEY"             # 注册 OpenAI 后登录个人中心获取
  "single_chat_prefix": ["bot", "@bot"],        # 私聊时文本需该前缀才能触发机器人回复
  "single_chat_reply_prefix": "🤖️",             # 私聊时自动回复的前缀，用于区分真人
  "group_chat_prefix": ["@bot"],                # 群聊时包含该前缀则会触发机器人回复
  "group_name_white_list": ["Qun1", "Qun2"],    # 开启自动回复的群名称列表，请修改为群名，或者直接使用 ALL_GROUP 解除限制
  "image_create_prefix": ["Photo", "Design"],   # 开启图片回复的前缀
  "conversation_max_tokens": 1000,              # 支持上下文记忆的最多字符数，超过了会删除最早的信息
  "character_desc": "我是人工智能 GPT, 一个由 OpenAI 训练的大型语言模型，版本为 GPT-3, 目前支持对话、搜索、画图。"
  # 人格描述
}
```
## 配置说明

**个人聊天**

+ 个人聊天中，需要以 "bot"或"@bot" 为开头的内容触发机器人，对应配置项 `single_chat_prefix` (如果不需要以前缀触发可以填写  `"single_chat_prefix": [""]`)
+ 机器人回复的内容会以 "[bot] " 作为前缀， 以区分真人，对应的配置项为 `single_chat_reply_prefix` (如果不需要前缀可以填写 `"single_chat_reply_prefix": ""`)

**群组聊天**

+ 群组聊天中，群名称需配置在 `group_name_white_list ` 中才能开启群聊自动回复。如果想对所有群聊生效，可以直接填写 `"group_name_white_list": "ALL_GROUP"`
+ 默认只要被人 @ 就会触发机器人自动回复；另外群聊天中只要检测到以 "@bot" 开头的内容，同样会自动回复（方便自己触发），这对应配置项 `group_chat_prefix`
+ 可选配置: `group_name_keyword_white_list` 配置项支持模糊匹配群名称，`group_chat_keyword`配置项则支持模糊匹配群消息内容，用法与上述两个配置项相同。（Contributed by [evolay](https://github.com/evolay))

**其他配置**

+ 对于图像生成，在满足个人或群组触发条件外，还需要额外的关键词前缀来触发，对应配置 `image_create_prefix `
+ 关于OpenAI对话及图片接口的参数配置（内容自由度、回复字数限制、图片大小等），可以参考 [对话接口](https://beta.openai.com/docs/api-reference/completions) 和 [图像接口](https://beta.openai.com/docs/api-reference/completions)  文档直接在 [代码](https://github.com/0xXPunkX/wechatgpt/blob/main/bot/openai/open_ai_bot.py) `wechatgpt/bot/openai/open_ai_bot.py` 中进行调整。
+ `conversation_max_tokens`：表示能够记忆的上下文最大字数（一问一答为一组对话，如果累积的对话字数超出限制，就会优先移除最早的一组对话）
+ `character_desc` 配置中保存着你对机器人说的一段话，他会记住这段话并作为他的设定，你可以为他定制任何人格


## 运行

- 如果是开发机 **本地运行**，直接在项目的根目录下执行：

```bash
python3 app.py
```

终端输出二维码后，使用微信进行扫码，当输出 `Start auto replying` 时表示自动回复程序已经成功运行了（注意：用于登录的微信需要在支付处已完成实名认证）。扫码登录后，就可以在微信手机端通过配置的关键词触发自动回复了。


- 如果是 **服务器部署**，则使用nohup命令在后台运行：

```bash
touch nohup.out                             # 首次运行需要新建日志文件
nohup python3 app.py & tail -f nohup.out    # 在后台运行程序并通过日志输出二维码
```
扫码登录后程序即可运行于服务器后台，此时可通过 `ctrl+c` 关闭日志，不会影响后台程序的运行。

- 重新登录服务器 / 日志关闭后如果想要再次打开只需输入：

```bash
tail -f nohup.out
```

- 可使用该命令可查看运行于后台的进程，如果想要重新启动程序可以先 `kill` 掉对应的进程。

```bash
ps -ef | grep app.py | grep -v grep
```



## 演示
N/a

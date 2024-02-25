---
title: Ollama的介绍和使用
date: 2024-02-25 10:32:44
tags:
categories: Python
---

Ollama是一个开源工具，它能帮助我们在本地搭建和运行Llama2、Mistral、Gemma等开源LLM服务，方便我们构建自己的 LLM 应用。

今天介绍下它的使用。

<!--more-->

# Ollama的安装

Linux环境下，Ollama的安装方式为：

```txt
curl -fsSL https://ollama.com/install.sh | sh
```

Mac环境下，去官网下载安装包进行安装，下载地址：

```txt
https://ollama.com/download/mac
```

安装完成后，我们可以执行 ollama 命令查看到如下信息：

```txt
Usage:
  ollama [flags]
  ollama [command]

Available Commands:
  serve       Start ollama
  create      Create a model from a Modelfile
  show        Show information for a model
  run         Run a model
  pull        Pull a model from a registry
  push        Push a model to a registry
  list        List models
  cp          Copy a model
  rm          Remove a model
  help        Help about any command

Flags:
  -h, --help      help for ollama
  -v, --version   Show version information

Use "ollama [command] --help" for more information about a command.

```

# Ollama的使用

通过查看 Ollama 的命令，可以发现它很像 Docker。

需要拉取模型到本地时，我们可以使用 ollama pull 命令：

```txt
ollama pull gemma
ollama pull llama2
ollama pull mistral
...
```

查看本地已存在的模型时，可以运行：ollama list

在本地运行一个模型时，可以执行：ollama run gemma

运行模型后，会出现这样的对话界面：

```txt
zanwang@zan ~ % ollama run gemma
>>> hello
Hello, hello! 👋
It's nice to hear from you. What would you like to talk about today?

>>> who are you
I am an AI language model, designed to provide you with information and 
engage in conversation on various topics. I am still under development, 
but I am constantly learning new things.
Would you like me to tell you more about myself or would you like to ask 
me a question?
```

我们可以输入问题，然后等待模型的回答。

 

我们还可以使用 ollama create 命令来创建一个 LLM.

首先要编写一个 Modefile 文件，文件内容如下：

```txt
FROM qwen

# set the temperature to 1 [higher is more creative, lower is more coherent]
PARAMETER temperature 1
PARAMETER max_tokens 10

# set the system message
SYSTEM """
You are Tracy, a execllent teacher. Answer as Tracy, the assistant, only.
"""
```

我上面基于通义千问qwen模型创建了一个LLM，设置它的 temperature 为1。

执行以下命令创建该模型：

```txt
ollama create tracy(模型名称) -f ./Modefile
```

创建完成后，我们可以执行 ollama list 查看模型列表：

```txt
zanwang@zan ollamaproject % ollama list
NAME                 	ID          	SIZE  	MODIFIED      
codellama:latest     	8fdf8f752f6e	3.8 GB	47 hours ago 	
gemma:latest         	430ed3535049	5.2 GB	47 hours ago 	
llama2:latest        	78e26419b446	3.8 GB	2 days ago   	
llama2-chinese:latest	cee11d703eee	3.8 GB	47 hours ago 	
mistral:latest       	61e88e884507	4.1 GB	47 hours ago 	
qwen:latest          	d53d04290064	2.3 GB	46 hours ago 	
tracy:latest         	e941b7cd9ad8	2.3 GB	9 seconds ago
```

然后我们就可以使用它了：ollama run tracy

# Ollama API的使用

除了直接聊天，ollama 还提供了 REST API 供我们使用，我们可以像调用 OpenAI 接口那样调用 ollama 的接口。启动 ollama 服务以后，我们可以直接调用接口获取 LLM 的响应，

我们可以调用聊天接口：

```txt
zanwang@zan blog % curl http://localhost:11434/api/chat -d '{
  "model": "qwen",
  "messages": [
    { "role": "user", "content": "你好，你是谁？" }
  ]
}'
```

api/chat 接口可以传入以下参数：

```txt
model: 模型名称
messages: 消息内容
format: 响应数据的格式，默认是json
template: prompt的模板
stream: 是否流式输出, 默认true
```



也可以调用 generate 接口：

```txt
curl http://localhost:11434/api/generate -d '{
  "model": "llama2-chinese",
  "prompt":"你好，你是谁"
}'
```



ollama rest api文档：

```txt
https://github.com/ollama/ollama/blob/main/docs/api.md
```

# Ollama UI的使用

使用 Ollama 创建好模型后，我们离创建一个聊天应用就差web界面了，下面是几个基于 ollama 的chat ui，有请兴趣的同学可以试一下：

```txt
chatbot ollama：https://github.com/ivanfioravanti/chatbot-ollama
open webui：https://github.com/open-webui/open-webui
ollama ui：https://github.com/ollama-webui/ollama-webui-lite
```





参考链接：

Ollama官网：https://ollama.com/

Ollama Github：https://github.com/ollama/ollama

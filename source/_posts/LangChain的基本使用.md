---
title: LangChain的基本使用
date: 2024-02-22 11:46:53
tags:
categories: Python
---

LangChain 是一个开源框架，它的出现是为了简化和加速大语言模型相关应用的开发，比如开发基于OpenAI的应用，开发基于百度千帆大模型的应用。

<!--more-->

那你可能会说，平时我们开发基于大模型的应用或者功能时，比如开发基于OpenAI的的应用时，我们都是直接调用OpenAI的chat接口或者assistants接口，并没有使用LangChain框架和它的接口。是的，平时我们开发时，可以直接调用相应大模型的接口，但当我们考虑到易用性、扩展性，使用LangChain是一个更好地选择，并且LangChain中还整合了很多组件和功能，包括：支持多种大语言模型、支持多种数据源连接、支持文本处理数据分析agent、提供部署和运维工具等。

下面基于Python和OpenAI介绍下LangChain的基本使用方式。

安装LangChain：pip install langchain

执行上面的命令后，会安装langchain、langchain-community、langchain-core等langchain相关的包。

安装openai：pip install openai

# 创建一个llm

安装好后，我们就可以使用LangChain就行开发了，先看一个简单的例子：

```python
from langchain.llms import OpenAI


OPENAI_API_KEY = 'xxx'


def call(prompt=''):
    # 创建一个基于OpenAI的llm模块
    llm = OpenAI(openai_api_key=OPENAI_API_KEY, temperature=1, max_tokens=10, model_name='gpt-3.5-turbo-instruct')
    # 调用模块的方法进行提问
    resp = llm.predict(prompt)
    return resp


if __name__ == '__main__':
    prompt = '你是谁？'
    resp = call(prompt)
    print(resp)
```

上面的例子中，我们使用 OpenAI 创建了一个 llm 对象，创建 llm 对象时传入的参数和直接调用 OpneAI Chat 接口时传入的参数几乎是一样的，具体有哪些参数，可以查看Python源码或者 LangChain 文档。



对于上面的例子，我们也可以使用 langchain-openai 库来实现。

首先安装 langchain-openai：pip install langchain-openai

安装好后，使用 langchain-openai 开发：

```python
from langchain_openai import OpenAI


OPENAI_API_KEY = 'xxx'


def call(prompt=''):
    llm = OpenAI(openai_api_key=OPENAI_API_KEY, temperature=1)
    resp = llm.predict(prompt)
    return resp


if __name__ == '__main__':
    prompt = '你是谁？'
    resp = call(prompt)
    print(resp)
```

使用 langchain.llms 的 OpenAI 和 langchain_openai 的 OpenAI 输出的效果是一样的，通过查看源码可以发现，这两个类最终都是继承的 BaseLLM 类。

# 创建一个chat llm

在 langchain 中，chat消息有四种类型，分别是：AIMessage, HumanMessage, SystemMessage、ChatMessage，

其中，AIMessage是 llm 的消息，HumanMessage是用户输入的消息，SystemMessage是系统的消息，ChatMessage是自定义的消息。

我们使用 langchain.chat_models 中的 ChatOpenAI 创建一个 chat llm：

```python
from langchain.chat_models import ChatOpenAI


OPENAI_API_KEY = 'xxx'


def call(prompt=''):
    # 创建一个基于Chat OpenAI的llm模块
    llm = ChatOpenAI(openai_api_key=OPENAI_API_KEY, temperature=1)
    resp = llm.predict(prompt)
    return resp


if __name__ == '__main__':
    prompt = '今天晚饭吃什么'
    resp = call(prompt)
    print(resp)
```

当然，也可以将上面的 langchain.chat_models 替换成 langchain_openai，效果是一样的。



创建chat llm后，除了简单的调用 predict() 聊天之外，还可以通过使用 HumanMessage、SystemMessage 进行聊天，具体代码如下：

```python
from langchain.chat_models import ChatOpenAI
from langchain.schema import HumanMessage, SystemMessage


OPENAI_API_KEY = 'xxx'


def call(prompt=''):
    llm = ChatOpenAI(openai_api_key=OPENAI_API_KEY, temperature=1)
    system_msg = '你是一个非常有经验的高级厨师，请帮我推荐有益健康的菜谱！'

    messages = [SystemMessage(content=system_msg), HumanMessage(content=prompt)]
    resp = llm.predict_messages(messages)
    return resp


if __name__ == '__main__':
    prompt = '今天晚饭吃什么'
    resp = call(prompt)
    print(resp)
```



我们也可以通过 generate() 方法进行多组chat：

```python
from langchain.chat_models import ChatOpenAI
from langchain.schema import HumanMessage, SystemMessage


OPENAI_API_KEY = 'xxx'


def call(prompt1='', prompt2=''):
    llm = ChatOpenAI(openai_api_key=OPENAI_API_KEY, temperature=1)
    system_msg1 = '你是一个非常有经验的高级厨师，轻轻帮我推荐有益健康的菜谱！'
    system_msg2 = '你是一个资深训练师，请给出如何保持身材的建议！'

    batch_messages = [
        [
            SystemMessage(content=system_msg1),
            HumanMessage(content=prompt1)
        ],
        [
            SystemMessage(content=system_msg2),
            HumanMessage(content=prompt2)
        ]
    ]
    resp = llm.generate(batch_messages)
    return resp


if __name__ == '__main__':
    prompt1 = '今天晚饭吃什么'
    prompt2 = '今天有什么锻炼计划'
    resp = call(prompt1, prompt2)
    print(resp.generations[0])
    print("===================")
    print(resp.generations[1])
```

# 使用prompt模板

上面的几个例子中，都是使用的固定的prompt，如果我们想更加灵活的使用prompt，可以使用prompt模板，看一个例子：

```python
from langchain.llms import OpenAI
from langchain.prompts import PromptTemplate
from langchain.chains import LLMChain


OPENAI_API_KEY = 'xxx'


def call(who):
    llm = OpenAI(openai_api_key=OPENAI_API_KEY, temperature=1)

    # 创建一个prompt模板
    prompt = PromptTemplate.from_template("如果{who}生气了，我该怎么哄她？")

    # 创建一个llm chain对象
    chain = LLMChain(llm=llm, prompt=prompt)
    # 运行chain时，将prompt模板的参数传入
    resp = chain.run(who=who)
    return resp


if __name__ == '__main__':
    resp = call("女朋友")
    print(resp)
    print("=========================")
    resp = call("小猫")
    print(resp)
```

代码中使用 PromptTemplate 创建了一个 prompt 模板，通过使用 LLMChain 模块，将 prompt 和 llm 串联起来，最后执行并获取结果。

上面的例子中我们使用到了 LangChain中的 chain 模块：LLMChain。chain是 LangChain 中非常重要的一部分知识，通过使用 chain，我们将以模块化的方式调用大模型，即把 prompt 分成一个模块，调用 llm 分成一个模块，这两个模块之间互不影响，我们修改 prompt 时不会影响 llm，切换 llm 时也不会影响 prompt。chain 通过把调用大模型的过程切分成几个部分，降低了这几个部分之间的依赖关系，从而更加灵活，降低了后期更新和修改的成本。

LangChain中不只有 LLMChain，还有LLMRouterChain、LLMCheckerChain等，感兴趣的可以查看官方文档了解更多。

# 创建一个ConversationChain

在 LangChain 中，我们可以使用 ConversationChain 创建一个用于对话的应用。使用 ConversationChain 的好处在于，它会缓存聊天记录的上下文，使我们在对话时，大模型能够知道之前的聊天数据，从而给出更好地回答。下面看一个例子：

```python
from langchain.llms import OpenAI
from langchain.chains import ConversationChain

OPENAI_API_KEY = 'xxx'

llm = OpenAI(openai_api_key=OPENAI_API_KEY)
conversion = ConversationChain(llm=llm)
conversion.run("你好呀")
conversion.run("我叫小明")
conversion.run("帮我写一份简历")
conversion.run("我是谁")
```

代码中使用了 ConversationChain ，查看源码会发现，ConversationChain 中有一个 memory 参数，这个参数指定了聊天记录的缓存信息，使大模型能够知道之前对话的内容。memory 参数的默认值就是 ConversationBufferMemory。

我们可以使用下面的方式查看 memory 中缓存的所有内容：

```python
print(conversation.memory.load_memory_variables({}))
```



memory 除了使用 ConversationBufferMemory之外，还可以使用 ConversationBufferWindowMemory，它可以指定缓存最后 n 轮的对话内容。

```python
from langchain.llms import OpenAI
from langchain.chains import ConversationChain
from langchain.memory import ConversationBufferWindowMemory


OPENAI_API_KEY = 'xxx'


def call():
    # 创建一个llm对象
    llm = OpenAI(openai_api_key=OPENAI_API_KEY, temperature=1)
    # 只缓存最近一轮的对话内容
    memory = ConversationBufferWindowMemory(k=1)
    # 创建一个ConversationChain对象
    conversation = ConversationChain(llm=llm, memory=memory)
    # 运行conversation chain
    resp = conversation.run("你好呀")
    print(resp)
    print("===========")
    resp = conversation.run("我是小明")
    print(resp)
    print("===========")
    resp = conversation.run("你能做什么")
    print(resp)
    print("===========")

    # 查看memory中的内容
    print(conversation.memory.load_memory_variables({}))


if __name__ == '__main__':
    call()
```



以上就是本文介绍的 LangChain 的一些基本用法。



LangChain 官方文档：https://python.langchain.com/docs/get_started/introduction

LangChain GitHub仓库：https://github.com/langchain-ai/langchain

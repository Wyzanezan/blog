---
title: AutoGen的介绍和使用
date: 2024-02-27 12:36:59
tags:
---

AutoGen 是由微软、宾夕法尼亚州立大学和华盛顿大学合作开发的一个框架，它支持使用多个代理来开发 LLM 应用程序，这些代理可以相互对话来完成任务。

<!--more-->

AutoGen 中的代理是可定制的、可对话的，并且无缝地允许人类参与。AutoGen 有如下特点：

1. 可以轻松构建基于多代理对话的下一代 LLM 应用程序。它简化了复杂的 LLM 工作流程的编排、自动化和优化。它最大限度地提高了 LLM 模型的性能并克服了它们的弱点。

1. 它支持复杂工作流程的多种对话模式。借助可定制和可对话的代理，开发人员可以使用 AutoGen 构建各种涉及对话自主性、代理数量和代理对话拓扑的对话模式。

1. 它提供了一系列具有不同复杂性的工作系统。这些系统涵盖各种领域和复杂性的广泛应用。

1. 它提供增强的 LLM 推理。它提供 API 和缓存等实用程序，以及错误处理、多配置推理、上下文编程等高级使用模式。

AutoGen 的功能类似于 LangChain 中的 agents。

# AutoGen的安装

首先，我们需要安装 AutoGen 的 Python 包：

```txt
pip install pyautogen docker
```

安装完成后，我们就可以使用了。

# AutoGen的使用

## 加载配置数据

使用 AutoGen 时，我们首先需要加载配置，代码如下：

```python
import os
import autogen

# config_list = [
#     {
#         "model": "gpt-4-32k",
#         "api_key": os.environ["OPENAI_API_KEY"],
#         "tags": ["gpt-4-32"]
#     },
#     {
#         "model": "gpt-4",
#         "api_key": os.environ["OPENAI_API_KEY"],
#         "tags": ["gpt-4"]
#     },
#     {
#          "model": "llama2",
#          "tags": ["llama2"]
#     }
# ]

# 选择需要使用的模型
filter_dict = {
    "model": "gpt-4"
}

config_list = autogen.config_list_from_json(
    "OAI_CONFIG_LIST",
    filter_dict=filter_dict,   # 指定需要使用的模型
)

# agent访问 LLM 的相关配置
llm_config = {
    "timeout": 600,
    "cache_seed": 42,
    "config_list": config_list,   # LLM 配置列表
    "temperature": 0,
}
```

在 config_list 中，可以配置很多模型供不同的 task 使用。config_list 可以直接是一个列表，也可以使用 autogen.config_list_from_json() 从 OAI_CONFIG_LIST 中加载配置数据，OAI_CONFIG_LIST 可以是一个json文件，内容如下：

```txt
[
    {
        "model": "gpt-4-32k",
        "api_key": "xxx",
        "tags": ["gpt-4-32"]
    },
    {
        "model": "gpt-4",
        "api_key": "xxx",
        "tags": ["gpt-4"]
    },
    {
        "model": "llama2",
        "tage": ["llama2", "local"]
    }
]
```

agent 会使用 config_list 中可用的第一个模型，并针对该模型进行 LLM 调用。如果第一个模型调用失败，agent 将针对第二个模型重试请求，依此类推，直到收到提示完成消息。



如果我们有多个 LLM，但是想指定使用其中的一个，可以使用 filter_dict 进行过滤，比如要指定使用 gpt-4 这个模型，就可以这样配置：

```python
# 选择需要使用的模型
filter_dict = {
    "model": "gpt-4"
}

config_list = autogen.config_list_from_json(
    "OAI_CONFIG_LIST",
    filter_dict=filter_dict,   # 指定需要使用的模型
)
```

当然，我们也可以使用 tags 进行过滤：

```python
filter_dict = {
    "tags": ["gpt-4"]
}
```

使用 tags 进行过滤时，tags 的值是一个列表，列表中可以指定多个 LLM。

## 创建并运行agent

创建 agent 时，上面的 llm_config 配置需要作为参数传入：

```python
assistant = autogen.AssistantAgent(
    name="assistant",   # agent名称
    llm_config=llm_config,   # agent配置
)

# 创建一个用户代理，该代理可以执行代码，并将结果反馈给其他代理
user_proxy = autogen.UserProxyAgent(
    name="user_proxy",
    human_input_mode="NEVER",
    max_consecutive_auto_reply=10,
    is_termination_msg=lambda x: x.get("content", "").rstrip().endswith("TERMINATE"),
    code_execution_config={
        "work_dir": "web",
        "use_docker": False,
    }, 
    llm_config=llm_config,
    system_message="""Reply TERMINATE if the task has been solved at full satisfaction.
Otherwise, reply CONTINUE, or the reason why the task is not solved yet.""",
)
```

创建 UserProxyAgent 对象时，参数的含义为：

name：指定用户代理的名称

human_input_mode：用户输入的模式，即是否在每次收到消息时请求人工输入。可能的值有 "ALWAYS"，"NEVER"，"TERMINATE"：

```txt
ALWAYS：表示每次收到消息时，代理都会提示人工输入，这种模式下，当用户输入 exit 或者 is_termination_msg 参数为 True 时，对话会退出。
NEVER：表示代理永远不会提示人工输入，这种模式下，当自动回复次数达到 max_consecutive_auto_reply 设置的值或者 is_termination_msg 为 True 时，对话会停止。
TERMINATE：表示仅当收到终止消息或自动回复数量达到 max_consecutive_auto_reply 设置的值时，代理才会提示人工输入。
```

max_consecutive_auto_reply：一个 int 值，设置最大连续自动回复数，默认为 None，没有限制。该限制仅在 human_input_mode 不是“ALWAYS”时起作用。

is_termination_msg：该参数值是一个函数，它接受字典形式的消息并返回一个布尔值，指示收到的消息是否是终止消息，该字典可以包含以下键：“content”、“role”、“name”、“function_call”。

code_execution_config：是一个字典或者 False，表示执行的代码需要的配置。字典中的配置项有：

```txt
work_dir（可选，str）：代码执行的工作目录。如果没有，将使用默认工作目录。默认工作目录是“path_to_autogen”下的“extensions”目录。
use_docker（可选，list、str 或 bool）：用于执行代码的 docker 容器。默认值为 True，这意味着代码将在 docker 容器中执行。将使用默认的镜像列表。如果提供了镜像名称的列表或字符串，则代码将在 docker 容器中执行，并成功拉取第一个镜像。如果为 False，则代码将在当前环境中执行。
timeout（可选，int类型）：最大执行时间（以秒为单位）。
last_n_messages（可选，int类型）：要回顾代码执行的消息数。默认为 1。
```

llm_config：LLM 的配置信息

system_message：str 或者 list 类型，用于 ChatCompletion 聊天的系统消息，仅当 llm_config 不为 False 时使用。

最后，执行 user agent 初始化聊天：

```python
user_proxy.initiate_chat(
    assistant,   # 接收消息的 agent，就是我们上面创建的 assistant agent
    message="""帮我分析一下过去一个月A股中药明康德这只股票的变化情况""",   # 传递的消息内容
)
```

下面是它的输出结果：

````txt
[user_proxy[0m (to assistant):


帮我分析一下过去一个月A股中药明康德这只股票的变化情况


--------------------------------------------------------------------------------
[assistant[0m (to user_proxy):

首先，我们需要从网络上获取股票数据。我们可以使用Python的pandas_datareader库来获取这些数据。以下是获取过去一个月药明康德（股票代码：603259.SH）股票数据的代码。

```python
# filename: stock_analysis.py

import pandas_datareader as pdr
import datetime

# Define the stock to be analyzed
stock_code = '603259.SS'

# Define the time period
end_date = datetime.datetime.now()
start_date = end_date - datetime.timedelta(days=30)

# Get the stock data
stock_data = pdr.get_data_yahoo(stock_code, start_date, end_date)

# Print the stock data
print(stock_data)
```

请运行这段代码，它将输出过去一个月药明康德的股票数据，包括每天的开盘价、最高价、最低价、收盘价和成交量。

--------------------------------------------------------------------------------
[31m
>>>>>>>> EXECUTING CODE BLOCK 0 (inferred language is python)...[0m
[33muser_proxy[0m (to assistant):

exitcode: 1 (execution failed)
Code output: 
Traceback (most recent call last):
  File "stock_analysis.py", line 14, in <module>
    stock_data = pdr.get_data_yahoo(stock_code, start_date, end_date)
                 ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/Users/zanwang/dev/AutoGenproject/pyautogen/lib/python3.11/site-packages/pandas_datareader/data.py", line 80, in get_data_yahoo
    return YahooDailyReader(*args, **kwargs).read()
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/Users/zanwang/dev/AutoGenproject/pyautogen/lib/python3.11/site-packages/pandas_datareader/base.py", line 253, in read
    df = self._read_one_data(self.url, params=self._get_params(self.symbols))
         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/Users/zanwang/dev/AutoGenproject/pyautogen/lib/python3.11/site-packages/pandas_datareader/yahoo/daily.py", line 153, in _read_one_data
    data = j["context"]["dispatcher"]["stores"]["HistoricalPriceStore"]
           ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^
TypeError: string indices must be integers, not 'str'


--------------------------------------------------------------------------------
[33massistant[0m (to user_proxy):

看起来我们在获取Yahoo财经数据时遇到了问题。这可能是由于Yahoo财经的API发生了变化或者是由于网络问题。我们可以尝试使用另一个库，如yfinance，它专门用于从Yahoo财经获取数据。

以下是使用yfinance库获取过去一个月药明康德（股票代码：603259.SS）股票数据的代码。

```python
# filename: stock_analysis.py

import yfinance as yf
import datetime

# Define the stock to be analyzed
stock_code = '603259.SS'

# Define the time period
end_date = datetime.datetime.now()
start_date = end_date - datetime.timedelta(days=30)

# Get the stock data
stock_data = yf.download(stock_code, start_date, end_date)

# Print the stock data
print(stock_data)
```

请运行这段代码，它将输出过去一个月药明康德的股票数据，包括每天的开盘价、最高价、最低价、收盘价和成交量。

--------------------------------------------------------------------------------
[31m
>>>>>>>> EXECUTING CODE BLOCK 0 (inferred language is python)...[0m
[33muser_proxy[0m (to assistant):

exitcode: 0 (execution succeeded)
Code output: 
                 Open       High        Low      Close  Adj Close     Volume
Date                                                                        
2024-01-29  63.000000  63.980000  59.360001  59.360001  59.360001  106097922
2024-01-30  56.400002  58.599998  56.180000  56.849998  56.849998   87396491
2024-01-31  56.000000  56.549999  53.900002  54.200001  54.200001   65697194
2024-02-01  54.560001  56.939999  53.730000  55.799999  55.799999   58337298
2024-02-02  56.910000  57.430000  50.220001  50.220001  50.220001   65707485
2024-02-05  46.279999  51.730000  45.200001  48.230000  48.230000  159302508
2024-02-06  46.860001  52.810001  45.369999  52.250000  52.250000  113259142
2024-02-07  53.610001  56.380001  53.610001  55.250000  55.250000  111295644
2024-02-08  53.650002  54.500000  52.000000  53.119999  53.119999   99007432
2024-02-19  50.500000  51.279999  48.619999  50.049999  50.049999  132806402
2024-02-20  49.849998  51.639999  49.330002  51.570000  51.570000  100319873
2024-02-21  51.000000  55.500000  50.820000  53.740002  53.740002  108874573
2024-02-22  53.090000  54.360001  52.880001  54.110001  54.110001   62886204
2024-02-23  54.040001  54.180000  52.549999  53.580002  53.580002   69410659
2024-02-26  53.840000  55.299999  53.750000  55.000000  55.000000   50604459


--------------------------------------------------------------------------------
[assistant[0m (to user_proxy):

根据输出的数据，我们可以看到过去一个月药明康德（股票代码：603259.SS）的股票每天的开盘价、最高价、最低价、收盘价和成交量。这些数据可以帮助我们了解股票的变化情况。

接下来，我们可以对这些数据进行一些基本的分析，例如计算平均收盘价、最高价和最低价，以及成交量。以下是进行这些计算的代码：

```python
# filename: stock_analysis.py

import yfinance as yf
import datetime

# Define the stock to be analyzed
stock_code = '603259.SS'

# Define the time period
end_date = datetime.datetime.now()
start_date = end_date - datetime.timedelta(days=30)

# Get the stock data
stock_data = yf.download(stock_code, start_date, end_date)

# Calculate the average close price
average_close = stock_data['Close'].mean()

# Calculate the highest price
highest_price = stock_data['High'].max()

# Calculate the lowest price
lowest_price = stock_data['Low'].min()

# Calculate the average volume
average_volume = stock_data['Volume'].mean()

# Print the results
print(f'Average close price: {average_close}')
print(f'Highest price: {highest_price}')
print(f'Lowest price: {lowest_price}')
print(f'Average volume: {average_volume}')
```

请运行这段代码，它将输出过去一个月药明康德的平均收盘价、最高价、最低价和平均成交量。

--------------------------------------------------------------------------------
[31m
>>>>>>>> EXECUTING CODE BLOCK 0 (inferred language is python)...[0m
[33muser_proxy[0m (to assistant):

exitcode: 0 (execution succeeded)
Code output: 
Average close price: 53.55533345540365
Highest price: 63.97999954223633
Lowest price: 45.20000076293945
Average volume: 92733552.4


--------------------------------------------------------------------------------
[assistant[0m (to user_proxy):

根据输出的结果，我们可以得到以下分析：

- 过去一个月药明康德的平均收盘价为53.56元。
- 过去一个月药明康德的最高价为63.98元。
- 过去一个月药明康德的最低价为45.20元。
- 过去一个月药明康德的平均成交量为92733552.4股。

这些数据可以帮助我们了解药明康德在过去一个月的股票表现。然而，这只是基本的分析，如果需要更深入的分析，可能需要考虑更多的因素，如市场环境、公司基本面、行业动态等。

TERMINATE

--------------------------------------------------------------------------------
````

从上面的输出可以看出，assistant agent 编写了数据抓取代码和数据分析代码并执行了它们，并且可以处理遇到的错误，还是非常强大的。这个问题如果直接问 ChatGPT，它是不会直接给出答案的，但是通过使用 AutoGen 就可以获取答案，用来获取和分析实时数据是非常不错的。

我们还可以创建一个 chat group，这个 chat group 中可以有多个 assistant agent，每个 assistant agent 可以有不同的功能。代码如下：

```python
user_proxy = autogen.UserProxyAgent(
    name="User_proxy",
    system_message="A human admin.",
    code_execution_config={
        "last_n_messages": 2,
        "work_dir": "groupchat",
        "use_docker": False,
    },
    human_input_mode="TERMINATE",
)
coder = autogen.AssistantAgent(
    name="Coder",
    llm_config=llm_config,
)
pm = autogen.AssistantAgent(
    name="Product_manager",
    system_message="Creative in software product ideas.",
    llm_config=llm_config,
)

# 创建一个聊天组
groupchat = autogen.GroupChat(agents=[user_proxy, coder, pm], messages=[], max_round=12)
# 创建一个管理员
manager = autogen.GroupChatManager(groupchat=groupchat, llm_config=llm_config)

# 开始聊天
user_proxy.initiate_chat(
    manager, message="Find a latest paper about gpt-4 on arxiv and find its potential applications in software."
)
```



关于 AutoGen 使用的更多例子，可以查看官方文档：https://microsoft.github.io/autogen/docs/Examples

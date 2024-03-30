---
title: LLM相关应用和开发工具整理
date: 2024-03-26 20:12:15
tags:
---

最近使用了很多大语言模型（LLM）相关的应用和开发工具，在这篇博客中将它们汇总一下。

<!--more-->

LLM 相关的应用和工具大致可以分为以下几类：

- AI Agent 开发平台
- LLM 应用开发平台
- LLM 管理工具
- LLM Python 开发库

下面分别介绍下每个分类都有哪些应用和工具。

# AI Agent开发平台

AI Agent 应用是 LLM 比较火的一个应用方向，也是一个在未来非常有潜力的方向。

AI Agent 是指能够在其环境中自动执行任务或达成目标的软件实体。这些代理能够通过感知环境并在此基础上进行决策来执行动作，以实现特定的目标或任务。AI Agent的工作原理和目的可以根据其设计和应用场景而有很大差异，但一般来说，它们都具备以下几个关键特性：

1. **感知能力（Perception）**：AI Agent能够通过内置或外部的传感器感知其所处的环境。这包括从环境中收集数据和信息，如图像、声音、温度等，以帮助代理理解当前的状况。
2. **决策能力（Decision Making）**：基于对环境的感知，AI Agent能够评估不同的行动方案，并选择最佳的方案来执行。这个过程通常涉及到某种形式的智能算法，如机器学习、深度学习或逻辑推理等。
3. **行动能力（Action）**：AI Agent可以通过执行器对环境产生影响，执行各种动作。这些执行器可以是物理的（如机器人的手臂）或软件上的（如网络请求）。
4. **自主性（Autonomy）**：AI Agent具有一定程度的自主性，意味着它们能够在没有人类直接干预的情况下，根据自己的判断和策略进行操作和决策。
5. **学习能力（Learning）**：许多AI Agent具备学习能力，可以通过经验改进其行为。这通常是通过机器学习和深度学习实现的，使得代理能够基于过去的行为和结果来优化其未来的决策和动作。
6. **适应性（Adaptability）**：与学习能力密切相关，适应性是指AI Agent能够适应环境的变化，并据此调整其行为的能力。

目前比较流行的 AI Agent 应用开发平台有：AutoGen、MetaGPT、CrewAI、XAgent、AutoGPT、OpenAgents 等。

## AutoGen

AutoGen 是由微软、宾夕法尼亚州立大学和华盛顿大学合作开发的一个框架，它支持使用多个代理来开发 LLM 应用程序，这些代理可以相互对话来完成任务。AutoGen 中的代理是可定制的、可对话的，并且无缝地允许人类参与。AutoGen 有如下特点：

```txt
1. 可以轻松构建基于多代理对话的 LLM 应用程序。它简化了复杂的 LLM 工作流程的编排、自动化和优化。它最大限度地提高了 LLM 模型的性能并克服了它们的弱点。
2. 它支持复杂工作流程的多种对话模式。借助可定制和可对话的代理，开发人员可以使用 AutoGen 构建各种涉及对话自主性、代理数量和代理对话拓扑的对话模式。
3. 它提供了一系列具有不同复杂性的工作系统。这些系统涵盖各种领域和复杂性的广泛应用。
4. 它提供增强的 LLM 推理。它提供 API 和缓存等实用程序，以及错误处理、多配置推理、上下文编程等高级使用模式。
```

Github：https://github.com/microsoft/autogen

官网：https://microsoft.github.io/autogen/

## MetaGPT

MetaGPT 是一个 Multi-Agent 框架，它为GPT分配不同的角色，形成一个协作实体来完成复杂的任务。MetaGPT 将一行需求作为输入并输出用户故事/竞争分析/需求/数据结构/API/文档等。在内部，MetaGPT 包括产品经理/架构师/项目经理/工程师。它提供了软件公司的整个流程以及精心策划的 SOP。

Code=SOP(Team)是核心理念。我们具体化SOP并将其应用于由 LLM 组成的团队。

LLM 是大脑，Agent 是手、脚、眼睛、耳朵等。

MetaGPT 一个集产品经理、架构师、项目经理、程序员于一体的 AI Agent 工具。

Github：https://github.com/geekan/MetaGPT

## XAgent

XAgent 是由清华大学开发的一个开源的基于大型语言模型（LLM）的自主智能体，可以自动解决各种任务。 它被设计为一个通用的智能体，可以应用于各种任务。它具有以下特点：

```txt
自主性：XAgent可以在没有人类参与的情况下自动解决各种任务。
安全性：XAgent被设计为安全运行。所有的行为都被限制在一个docker容器内。不用担心你的主机环境受到影响
可扩展性：XAgent被设计为可扩展的。您可以轻松地添加新的工具来增强智能体的能力，甚至是新的智能体！
GUI：XAgent为用户提供了友好的GUI来与智能体交互。您也可以使用命令行界面与智能体交互。
与人类的合作：XAgent可以与您合作解决任务。它不仅有能力在行进中遵循您的指导来解决复杂的任务，而且在遇到挑战时还可以寻求您的帮助。
```

XAgent由三部分组成：

```txt
调度器 负责动态实例化和分派任务给不同的智能体。它允许我们添加新的智能体和改进智能体的能力。
规划器 负责为任务生成和校正计划。它将任务分解为子任务，并为它们生成里程碑，使智能体能够逐步解决任务。
行动者 负责采取行动实现目标和完成子任务。行动者利用各种工具来解决子任务，它也可以与人类合作来解决任务。
```

Github：https://github.com/OpenBMB/XAgent

## CrewAI

CrewAI 是用于编排角色扮演、自主人工智能代理的框架。通过促进协作智能，CrewAI 使代理能够无缝协作，处理复杂的任务。

GIthub：https://github.com/joaomdmoura/crewAI

## AutoGPT

AutoGPT 的宗旨是为每个人提供易于使用和构建的人工智能。它们提供工具，以便我们可以专注于重要的事情。

AutoGPT 项目由四个主要部分组成：

```txt
Agent：是 AutoGPT 的核心以及启动这一切的项目：由 LLM 支持的半自主 agent，可以为我们执行任何任务
Benchmark（基准）：又名 agbenchmark，衡量你的 agent 的表现！ agbenchmark 可与任何支持 agent 协议的 agent 一起使用，并且与项目的 CLI 集成使得与 AutoGPT 和基于 forge 的代理一起使用变得更加容易。
Forge（熔炉）：打造你自己的 agent！ Forge 是适合您的 agent 应用程序的现成模板。所有样板代码都已处理完毕，让您可以将所有创造力投入到使您的代理与众不同的事情中。
Frontend（前端）：适用于任何符合 agent 协议的 agent 的易于使用的开源前端。
```

文档：https://docs.agpt.co/

Github：https://github.com/Significant-Gravitas/AutoGPT

## OpenAgents

由于当前的语言代理框架旨在促进构建概念证明语言智能体（Language Agent）的搭建，但是同时忽视了非专家用户的使用，对应用级设计也关注较少。 OpenAgents 是一个用于在日常生活中使用和托管语言智能体的开放平台。

OpenAgents中实现了三个智能体：

```txt
1. 数据智能体-用于用Python/SQL和数据工具进行数据分析；
2. 插件智能体-具有200多个日常工具，并且可供拓展；
3. 网络智能体-用于自动上网。
```

OpenAgents可以分析数据，调用插件，像ChatGPT Plus一样控制浏览器，并且它：

```txt
1. 易于部署
2. 全栈代码
3. 聊天Web UI
4. 代理方法
```

OpenAgents 使普通用户通过为快速响应和常见失败进行优化的web UI与智能体功能进行交互，同时为开发人员和研究人员在本地设置上提供无缝部署体验，为制作创新的语言代理和实现现实世界评估提供了基础。 

Github：https://github.com/xlang-ai/OpenAgents

# LLM 应用开发平台

LLM 应用开发平台允许我们创建自己的知识库，并基于这些知识库快速开发个人或者企业的 RAG （检索增强生成）应用。这类平台应用有 Dify、Flowise、FastGPT、TaskingAI、ChatGPT-Next-Web、LobeChat、PrivateGPT 等。

## Dify

Dify 是一个 LLM 应用程序开发平台，它集成了 BaaS （后端即服务）和 LLMOps，涵盖了构建生成式 AI 原生应用程序的基本技术堆栈，包括内置的 RAG 引擎。Dify 允许我们基于任何 LLM 部署自己版本的 Assistants API 和 GPT。

Dify 有以下特点：

```txt
LLM支持：与 OpenAI 的 GPT 系列模型集成,或者与开源的 Llama2 系列模型集成。事实上，Dify支持主流的商业模型和开源模型(本地部署或基于 MaaS)。
Prompt IDE：和团队一起在 Dify 协作，通过可视化的 Prompt 和应用编排工具开发 AI 应用。 支持无缝切换多种大型语言模型。
RAG引擎：包括各种基于全文索引或向量数据库嵌入的 RAG 能力，允许直接上传 PDF、TXT 等各种文本格式。
AI Agent：基于 Function Calling 和 ReAct 的 Agent 推理框架，允许用户自定义工具，所见即所得。Dify 提供了十多种内置工具调用能力，如谷歌搜索、DELL·E、Stable Diffusion、WolframAlpha 等。
持续运营：监控和分析应用日志和性能，使用生产数据持续改进 Prompt、数据集或模型。
```

官网：https://dify.ai/

Github：https://github.com/langgenius/dify

## Flowise

Flowise 是一种低代码/无代码拖放工具，可以让我们轻松可视化和构建 LLM 应用程序。

文档：https://docs.flowiseai.com/

Github：https://github.com/FlowiseAI/Flowise

## FastGPT

FastGPT 是一个基于 LLM 大语言模型的知识库问答系统，提供开箱即用的数据处理、模型调用等能力。同时可以通过 Flow 可视化进行工作流编排，从而实现复杂的问答场景！

FastGPT 有如下特点：

```txt
独特的 QA 结构：针对客服问答场景设计的 QA 结构，提高在大量数据场景中的问答准确性。
可视化工作流：通过 Flow 模块展示了从问题输入到模型输出的完整流程，便于调试和设计复杂流程。
无限扩展：基于 API 进行扩展，无需修改 FastGPT 源码，也可快速接入现有的程序中。
便于调试：提供搜索测试、引用修改、完整对话预览等多种调试途径。
支持多种模型：支持 GPT、Claude、文心一言等多种 LLM 模型，未来也将支持自定义的向量模型。
```

官网：https://fastgpt.run/

Github：https://github.com/labring/FastGPT

## TaskingAI

TaskingAI 将 Firebase 的简单性带入 AI 原生应用程序开发中。该平台支持使用来自不同提供商的各种 LLM 来创建类似 GPT 的多租户应用程序。它具有独特的模块化功能，例如推理、检索、助手和工具，无缝集成以增强开发过程。TaskingAI 的凝聚力设计确保了人工智能应用程序开发中的高效、智能和用户友好的体验。

TaskingAI 有如下特点：

```txt
1. 一体化 LLM 平台：通过统一的 API 访问数百个 AI 模型。
2. 直观的 UI 控制台：简化项目管理并允许控制台内工作流程测试。
3. BaaS 启发的工作流程：将 AI 逻辑（服务器端）与产品开发（客户端）分开，提供从基于控制台的原型设计到使用 RESTful API 和客户端 SDK 的可扩展解决方案的清晰途径。
4. 可定制的集成：通过可定制的工具和先进的检索增强生成（RAG）系统增强 LLM 的功能。
5. 异步效率：利用Python FastAPI的异步特性实现高性能、并发计算，增强应用程序的响应能力和可扩展性。
```

TaskingAI 的架构设计以模块化和灵活性为核心，能够与各种 LLM 兼容。这种适应性使其能够轻松支持各种应用程序，从简单的演示到复杂的多租户人工智能系统。TaskingAI 以开源原则为基础，融合了众多开源工具，确保该平台不仅多功能，而且可定制。

文档：https://docs.tasking.ai/

Github：https://github.com/TaskingAI/TaskingAI

## ChatGPT-Next-Web

ChatGPT-Next-Web 可以让我们一键免费部署你的私人 ChatGPT 网页应用，支持 GPT3, GPT4 & Gemini Pro 模型。

它有如下特点：

```txt
1. 只需在 1 分钟内即可在 Vercel 上一键免费部署
2. 提供Linux/Windows/MacOS 上的紧凑型客户端下载
3. 与自行部署的LLM完全兼容，推荐与RWKV-Runner或LocalAI一起使用
4. 隐私第一，所有数据都存储在浏览器本地
5. Markdown 支持：LaTex、mermaid、代码高亮等。
6. 响应式设计、深色模式和 PWA 
7. 首屏加载速度快（~100kb），支持流式响应
8. v2 中的新增功能：使用提示模板（掩码）创建、共享和调试您的聊天工具 
9. 由 Awesome-chatgpt-prompts-zh 和 Awesome-chatgpt-prompts 提供支持的很棒的 prompt
10. 自动压缩聊天历史记录以支持长时间对话，同时保存您的令牌
11. 国际化：英语、西班牙语、意大利语、土耳其语、德语、Tiếng Việt、Русский、Čeština、한국어、印度尼西亚
```

Github：https://github.com/ChatGPTNextWeb/ChatGPT-Next-Web

## LobeChat

Lobe Chat - 一个开源、现代设计的 LLM / 人工智能聊天框架。支持多AI提供商（OpenAI / Claude 3 / Gemini / Perplexity / Bedrock / Azure / Mistral / Ollama），多模态（Vision / TTS）和插件系统。一键免费部署您的私人 ChatGPT 聊天应用程序。

Lobe Chat 有以下特点：

```txt
1. 多模式服务提供商支持，支持了AWS Bedrock、Google AI、Anthropic、ChatGLM、Moonshot AI、Groq等模型服务商
2. 本地大语言模型 (LLM) 支持，LobeChat 基于 Ollama 支持了本地模型的使用，让用户能够更灵活地使用自己的或第三方的模型
3. 模型视觉识别，LobeChat 已经支持 OpenAI 最新的 gpt-4-vision 支持视觉识别的模型，这是一个具备视觉识别能力的多模态应用。 用户可以轻松上传图片或者拖拽图片到对话框中，助手将能够识别图片内容，并在此基础上进行智能对话，构建更智能、更多元化的聊天场景
4. TTS 和 STT 语音会话，LobeChat 支持文字转语音（Text-to-Speech，TTS）和语音转文字（Speech-to-Text，STT）技术，这使得我们的应用能够将文本信息转化为清晰的语音输出，用户可以像与真人交谈一样与我们的对话助手进行交流。 用户可以从多种声音中选择，给助手搭配合适的音源。 同时，对于那些倾向于听觉学习或者想要在忙碌中获取信息的用户来说，TTS 提供了一个极佳的解决方案
5. 文本生成图像，支持最新的文本到图片生成技术，LobeChat 现在能够让用户在与助手对话中直接调用文生图工具进行创作。 通过利用 DALL-E 3、MidJourney 和 Pollinations 等 AI 工具的能力， 助手们现在可以将你的想法转化为图像。 同时可以更私密和沉浸式地完成你的创作过程
6. 插件系统（函数调用），LobeChat 的插件生态系统是其核心功能的重要扩展，它极大地增强了 ChatGPT 的实用性和灵活性。通过利用插件，ChatGPT 能够实现实时信息的获取和处理，例如自动获取最新新闻头条，为用户提供即时且相关的资讯
7. Agent 市场 (GPTs)，在 LobeChat 的助手市场中，创作者们可以发现一个充满活力和创新的社区，它汇聚了众多精心设计的助手，这些助手不仅在工作场景中发挥着重要作用，也在学习过程中提供了极大的便利。 我们的市场不仅是一个展示平台，更是一个协作的空间。在这里，每个人都可以贡献自己的智慧，分享个人开发的助手
8. 使用渐进式Web应用程序 (PWA)，采用了渐进式 Web 应用 PWA 技术， 这是一种能够将网页应用提升至接近原生应用体验的现代 Web 技术。通过 PWA，LobeChat 能够在桌面和移动设备上提供高度优化的用户体验，同时保持轻量级和高性能的特点。 在视觉和感觉上，我们也经过精心设计，以确保它的界面与原生应用无差别，提供流畅的动画、响应式布局和适配不同设备的屏幕分辨率
9. 移动设备适配，针对移动设备进行了一系列的优化设计，以提升用户的移动体验
10. 可以自定义主题，LobeChat 在界面设计上充分考虑用户的个性化体验，因此引入了灵活多变的主题模式，其中包括日间的亮色模式和夜间的深色模式。 除了主题模式的切换，还提供了一系列的颜色定制选项，允许用户根据自己的喜好来调整应用的主题色彩。无论是想要沉稳的深蓝，还是希望活泼的桃粉，或者是专业的灰白，用户都能够在 LobeChat 中找到匹配自己风格的颜色选择
```

Github：https://github.com/lobehub/lobe-chat

## PrivateGPT

PrivateGPT 是一个可投入生产的 AI 项目，可让您利用大型语言模型 (LLM) 的功能提出有关文档的问题，即使在没有互联网连接的情况下。 100% 私有，任何数据都不会离开您的执行环境。该项目提供了一个 API，提供构建私有的、上下文感知的 AI 应用程序所需的所有原语。它遵循并扩展了 OpenAI API 标准，支持普通响应和流式响应。这意味着，如果您可以在您的工具之一中使用 OpenAI API，则可以使用您自己的 PrivateGPT API，无需更改代码，并且如果您在本地设置中运行 privateGPT，则免费。

PrivateGPT 是一项服务，它将一组 AI RAG 原语包装在一组全面的 API 中，提供私有、安全、可定制且易于使用的 GenAI 开发框架。

它使用FastAPI和LLamaIndex作为其核心框架。这些可以通过更改代码库本身来定制。

它支持各种本地和远程的 LLM 提供商、嵌入提供商和向量存储。这些可以轻松更改，而无需更改代码库。

API 分为两个逻辑块： 

```txt
1. 高级 API，抽象了 RAG（检索增强生成）管道实现的所有复杂性：
1）文档摄取：内部管理文档解析、分割、元数据提取、嵌入生成和存储。
2）使用所摄取文档中的上下文进行聊天和完成：抽象上下文检索、提示工程和响应生成

2. 低级 API，允许高级用户实现自己的复杂管道：
1）嵌入生成：基于一段文本。
2）上下文块检索：给定查询，从摄取的文档中返回最相关的文本块。
```

除此之外，还提供了一个可用的 Gradio UI 客户端来测试 API，以及一组有用的工具，例如批量模型下载脚本、摄取脚本、文档文件夹监视等。

Github：https://github.com/zylon-ai/private-gpt

文档：https://docs.privategpt.dev/overview/welcome/introduction

# LLM 管理工具

LLM 管理工具可以帮助我们管理开源大语言模型，它们可以下载、删除、运行开源大模型，并且提供调用 LLM 服务的API，使我们可以很容易的构建基于本地的 LLM 应用。

这类管理工具有 Ollama、LM Studio、LocalAI 等。

## Ollama

Ollama是一个开源工具，它能帮助我们在本地搭建和运行Llama2、Mistral、Gemma等开源LLM服务，Ollama也允许我们基于一个 LLM 去创建另一个LLM，就像使用 Dockerfile 那样，它还提供了 REST API 供我们调用。

官网：https://ollama.com/

Github：https://github.com/ollama/ollama

## LM Studio

LM Studio 可以帮我们发现、下载 LLM，并在本地运行它，它还能帮我们做以下工作：

```txt
1. 通过应用内聊天 UI 或 OpenAI 兼容的本地服务器使用模型
2. 从 HuggingFace 存储库下载任何兼容的模型文件
3. 在应用程序的主页中发现新的和值得注意的 LLM
```

官网：https://lmstudio.ai/

## LocalAI

LocalAI 允许我们使用消费级硬件在本地或本地运行 LLM 、生成图像、音频（不仅如此），支持多种模型系列和架构，并且不需要 GPU。LocalAI 致力于让任何人都能使用人工智能。

文档：https://localai.io/

# LLM Python开发库

LLM Python 开发库提供了通用的 API 让我们来调用各种大模型服务，除了 LangChain、LlamaIndex 这这些常用的库，还有下面一些很好用的库：

```txt
1. 提供统一接口调用 LLM 的库：transformers、vllm、litellm 等
2. 开发 web 界面的库：streamlit、chainlit 等
```



以上就是 LLM 相关的应用和开发工具，后面还会持续补充 ...

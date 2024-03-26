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

目前比较流行的 AI Agent 应用开发平台有：AutoGen、MetaGPT、CrewAI、XAgent、AutoGPT等。

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

# LLM 应用开发平台

LLM 应用开发平台允许我们创建自己的知识库，并基于这些知识库快速开发个人或者企业的 RAG （检索增强生成）应用。这类平台应用有 Dify、Flowise、FastGPT、TaskingAI等。

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



以上就是 LLM 相关的应用和开发工具，后面还会持续补充。

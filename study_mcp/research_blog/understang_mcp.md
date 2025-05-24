# 一文带你 "看见" MCP 的过程，彻底理解 MCP 的概念

Original 悟鸣 [阿里云开发者](javascript:void(0);)

_2025年05月23日 08:31_ _浙江_

![Image](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLrU7fFaF1s5cc9SMEzqmQzayQARaCYvxm2ekA9O3aK6sM76LLZmHoGSrxfvib5oEzyamIn0TJ2dMA/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

一、前言

说实话，当我看到使用 MCP 服务还需要手动复制粘贴 JSON 的时候，包括现在很多 MCP 服务只有工具，没有资源和提示时，我认为 MCP 还不太成熟。

随着今年“智能体”的大爆发，使用工具的诉求越来越强烈。随着 MCP 服务市场、MCP 自动配置功能的出现，MCP 的使用门槛正在降低，越来越多的服务宣布支持 MCP 协议，开始要爆发的趋势。

网上也有很多介绍 MCP 的文章，但很多文章要么太理论化，动辄就上一堆 Python 代码，对于非开发者来说比较枯燥，要么就是直接介绍 MCP 如何配置使用，太偏实践，知其然，不知其所以然。

本文将先尽量用相对通俗的语言和图形介绍基本的理论，然后以 ModelScope 的 MCP 市场，Cherry Studio 客户端为例，为大家展示 MCP 的过程，让大家更加有体感，更好地理解和掌握 MCP ,最后分享一点资料和观点。

二、理论基础

RAG 是为了让大模型获取足够的上下文，Function Calling 是为了让模型使用工具，他们和 MCP 有或多或少的联系，因此，接下来，我先介绍前面两个概念再介绍 MCP。

**2.1 先看 RAG**

RAG（Retrieval Augmented Generation ，检索增强生成），我们不需要训练和微调大模型，只需要提供和用户提问相关的额外的信息到提示词中，从而可以获得更高质量的回答。

![Image](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL2bakmHG7gRicob7WwaDUzetkfS7QVBiaOgh03gC9vVib0opP6BG82U8FlPsZBZnsAwHGicAl5MM9icBg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

图片来源：https://www.dailydoseofds.com/16-techniques-to-supercharge-and-build-real-world-rag-systems-part-1/

通常，需要将资料通过嵌入模型生成服务转化为向量，然后存储到向量数据库中。

当用户提问时，将用户的问题向量化从向量数据库中进行相似度匹配出 TOP N 个片段，拼接成新的提示词发送给大模型，大模型就可以结合你的资料更好地回答问题了。

当然， RAG 还有很多变种，如 Agentic RAG 、Modular RAG、Graph RAG 等多种高级形式，这里就不展了。

2.2 再看 Function Calling

Function Calling (函数调用) 是一种允许大型语言模型(LLM)根据用户输入识别它需要的工具并决定何时调用该工具的机制。

![Image](https://mmbiz.qpic.cn/mmbiz_gif/Z6bicxIx5naL2bakmHG7gRicob7WwaDUzeEOaOmlYcCrqIG37TO4Vub6uLFW2BiacbZaMc0ZXzkY5qY93b9Rcys4w/640?wx_fmt=gif&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

图片来源：ailydoseofds

基本工作原理如下：LLM 接收用户的提示词，LLM 决定它需要的工具，执行方法调用，后端服务执行实际的请求给出处理结果，大语言模型根据处理结果生成最终给用户的回答。

不同的 API 需要封装成不同的方法，通常需要编写代码，很难在不同的平台灵活复用。

参见百炼函数调用文档：https://help.aliyun.com/zh/model-studio/tool-calling-overview/

**2.3 来看 MCP** 

模型上下文协议（Model Context Protocol，简称MCP）是一个由 Anthropic 在 2024 年 11 月 25 日开源的新标准。MCP 是一个开放标准，旨在连接AI助手与数据所在的系统，包括内容存储库、业务工具和开发环境。其目标是帮助前沿模型产生更好、更相关的响应。

![Image](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL2bakmHG7gRicob7WwaDUzeUYSgX7IwZibUjkEztMibdicria5QTn6VdUibpOo7G3YLDJGoovrlXV9U5vg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

图片来源：https://zhuanlan.zhihu.com/p/598749792

MCP 可以看作是 AI 应用程序的 "USB-C端口"。就像 USB-C 为连接设备与各种外设提供了标准化方式，MCP为 AI 模型连接不同数据源和工具提供了标准化方法。

![Image](https://mmbiz.qpic.cn/mmbiz_gif/Z6bicxIx5naL2bakmHG7gRicob7WwaDUzede4g2iaegGLvY4QyrgqoTia43BiaguU0T53No4YKd4umbakKyaAIgeY8g/640?wx_fmt=gif&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

图片来源：ailydoseofds

#### **2.3.1 MCP 的核心**

从技术角度看，MCP 遵循客户端-服务器架构，其中主机应用可以连接到多个服务器。

MCP 有三个关键组件：

-   主机(Host)
    
-   客户端(Client)
    
-   服务器(Server)
    

#### **2.3.2 组件详解**

-   **主机：代表任何提供 AI 交互环境的应用程序(如 Claude 桌面版、Cursor)，它能访问工具和数据，并运行 MCP 客户端。**
    

-   **MCP 客户端：在主机内运行，使其能与 MCP 服务器通信。**
    

-   **MCP 服务器：暴露特定功能并提供数据访问，例如：**
    

-   工具：使 LLM 能通过服务器执行操作
    
-   资源：向 LLM 公开服务器中的数据和内容
    
-   提示：创建可重用的提示模板和工作流
    

#### **2.3.3 MCP 相比传统 API 的优势**

传统 API ：

![Image](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL2bakmHG7gRicob7WwaDUzeAtgpnupxxehPgRfrSvargNiaOHc6bIiab1WScAQZiaZugNHVbTdqjaBog/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

图片来源：ailydoseofds

-   如果 API 最初需要两个参数(例如天气服务的位置和日期)，用户集成应用程序会发送带有这些确切参数的请求。
    
    ![Image](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL2bakmHG7gRicob7WwaDUzeEJCXUSdmLylE6EmTbLOH23sDYibykoV73DR6gAZp95PG4YmPrBQCaBQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)
    

图片来源：ailydoseofds

-   如果后来添加第三个必需参数(例如温度单位，如摄氏度或华氏度)，API的合约就会改变。
    
-   这意味着所有 API 用户必须更新代码以包含新参数。如果不更新，请求可能失败、返回错误或提供不完整结果。
    

MCP 的设计解决了这个问题：

-   MCP 引入了与传统 API 截然不同的动态灵活方法。
    
-   当客户端(如 Claude 桌面版)连接到 MCP 服务器(如您的天气服务)时，它发送初始请求以了解服务器的能力。
    
    ![Image](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL2bakmHG7gRicob7WwaDUze36VwXG9MUzYORhXgZpzib4CVHtial6icMWiapw9FJeyVnuqhicSO5DeBLPg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)
    

图片来源：ailydoseofds

-   服务器响应其可用工具、资源、提示和参数的详细信息。例如，如果您的天气 API 最初支持位置和日期，服务器会将这些作为其功能的一部分进行通信。
    
    ![Image](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL2bakmHG7gRicob7WwaDUzefDrmQg2yPhnLgMKPRQwFKUDQJa50Q2xGdFgPBhV5LTNzFJt2jE94xg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)
    

图片来源：ailydoseofds

-   如果您稍后添加单位参数，MCP 服务器可以在下一次交换期间动态更新其功能描述。客户端不需要硬编码或预定义参数—只需查询服务器的当前功能并相应地适应。
    
-   这样，客户端可以及时调整其行为，使用更新的功能(例如在请求中包含单位)，无需重写或重新部署代码。
    

![Image](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL2bakmHG7gRicob7WwaDUzeXME2VRibj8ENiarb5Hh4wRfwxc2ib1u3aUoShucqzqVI8Eq10fpBQrtBQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

图源：https://www.latent.space/p/why-mcp-won

其实，MCP 也是用了 “中间层”的思想，让大模型使用工具标准化，让大模型调用工具更方便，焕发出新的生机。

#### **2.3.4 MCP 的过程**

![Image](https://mmbiz.qpic.cn/mmbiz_gif/Z6bicxIx5naL2bakmHG7gRicob7WwaDUzeDuv7637icicdQBAuz4hg95bFibD3FjaPCJdAb5yNdPSetowNTj5lGw03Q/640?wx_fmt=gif&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

图片来源：ailydoseofds

首先需要在主机上自动或手动配置 MCP 服务，当用户输入问题时， MCP 客户端让 大语言模型选择 MCP 工具，大模型选择好 MCP 工具以后， MCP 客户端寻求用户同意（很多产品支持配置自动同意），MCP 客户端请求 MCP 服务器， MCP 服务调用工具并将工具的结果返回给 MCP 客户端， MCP 客户端将模型调用结果和用户的查询发送给大语言模型，大语言模型组织答案给用户。

**其实 RAG 、Function Call 和 MCP 本质上都是一样，都是为了借助外部工具帮助大模型完成更复杂的事情。**

三、“看见” MCP 的过程

![](https://mp.weixin.qq.com/s/UfzmzJ3zTOQ45ppRy5D6kA)

已关注

Follow

Replay Share Like

Close

**观看更多**

更多

_退出全屏_

[](javascript:;)

_切换到竖屏全屏__退出全屏_

阿里云开发者已关注

[](javascript:;)

Share Video

，时长01:06

0/0

00:00/01:06

切换到横屏模式

继续播放

进度条，百分之0

[Play](javascript:;)

00:00

/

01:06

01:06

_全屏_

倍速播放中

[0.5倍](javascript:;) [0.75倍](javascript:;) [1.0倍](javascript:;) [1.5倍](javascript:;) [2.0倍](javascript:;)

[超清](javascript:;) [流畅](javascript:;)

Your browser does not support video tags

继续观看

一文带你 "看见" MCP 的过程，彻底理解 MCP 的概念

观看更多

转载

,

一文带你 "看见" MCP 的过程，彻底理解 MCP 的概念

阿里云开发者已关注

Share点赞Wow

Added to Top Stories[Enter comment](javascript:;)

[Video Details](javascript:;)

**3.1 ModelScope**

![Image](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL2bakmHG7gRicob7WwaDUze3HricibocF9sc8lKVfytmuPy3vCh85B43YiavUwbPia26DGdQ69VMeqmeg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

我们来到魔搭的 MCP 市场：https://modelscope.cn/mcp，这里只是为了演示，其他的 MCP 广场也是一样的。

![Image](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL2bakmHG7gRicob7WwaDUzeaLDia8q10Fq14L8YibT7EbFFFot2eqaTx96pSR80tUUB3tgGg4hBnD3w/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

登录以后，我们选择一个 MCP 服务，点击右侧的“连接”按钮，就可以 获取 MCP服务的 SSE URL。

**3.2 Cherry Studio**

Cherry Studio 官网: https://cherry-ai.com/

![Image](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL2bakmHG7gRicob7WwaDUzeMhIBwANnS3Aa6cicDsKNb5hkkbt0kYn0ic63lfgcCZRR0RDZL8TQCKow/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

在右上角点击开启，就可以看到该 MCP 服务提供的工具、提示和资源。

![Image](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL2bakmHG7gRicob7WwaDUzep3EfGz41ZqUIbJiaia8VL6EpQb1KjMGjFYoXIVCYFMuWcHpzia2kgMPCA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

可以单独开启和关闭每个可用的工具。

![Image](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL2bakmHG7gRicob7WwaDUze2yZPgSFyRYonDOw7ADibtm8oiaKVibsu04DU0d84cicnYiayh3xLHPzJqAQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

每个 MCP 服务手动配置太麻烦，大家也可以点击“同步服务器”，配置 ModelScope 的 API 令牌，配置之后就可以一键同步 ModelScope 上开启的 MCP 服务了。

**3.3 开发者模式**

![Image](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL2bakmHG7gRicob7WwaDUzeRrsZLPfhE3Ctz0eF8e4ITD7tT0jOVgTAYzqPjibvduoxiaprNHc9JjPQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

输入想要问的问题，开启需要的 MCP 服务。

为了看见 MCP 的执行过程，我们还需要通过 Ctrl+Shift+I（Mac端：Command+Option+I）打开开发者模式。

![Image](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL2bakmHG7gRicob7WwaDUzesR9WW72jHibDMQxQ1Uxv4sI6XLJ5DWDhYoias6tA1qmZPysR1iajL4t9A/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

第一次请求，发送系统提示词，包括大模型可以选择的工具和要求等，然后给出用户的问题。

![Image](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL2bakmHG7gRicob7WwaDUzeMwNPG3J8bgg5pVsukjICO7ib4f9nLDCTiaXeZICG8zN7LnuqriafzV2PQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

可以看到第二次调用，模型选择使用 infoq 工具获得资讯后组织语言展示给用户。

index =0 的系统提示词的内容：

    {

主要包括：

-   工具使用格式 - 使用XML风格的标签来调用工具，包括工具名称和JSON格式的参数；
    
-   工具使用示例 - 展示了如何使用各种假设工具的例子，如文档问答、图像生成、Python解释器等；
    
-   可用工具列表 - 提供了多个实际可用的信息获取工具，主要是各种中英文新闻和内容平台的热门榜单；
    
-   工具使用规则 - 强调了正确使用参数、仅在需要时调用工具、不重复调用等规则；
    
-   最后，提示词以一个有趣的激励结束，表示如果任务正确完成，将获得一百万美元奖励。
    

index =1 的用户的内容

    {

index =2 的助手的内容

    {

index =3 的 用户的内容

    {

![Image](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL2bakmHG7gRicob7WwaDUzeFNwdQ6BJXEhSV3xwlv4IgYSYZoWfXMmJwhAyZMJ3Ut2rPWPzSs91GQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

第三次调用主要是为了给话题起标题。

大家，可以多开启几个工具或者多问几次，结合理论部分的讲解，会对 MCP 的过程会有更深刻的体会。

四、MCP 市场和 MCP 学习经验

**4.1 MCP 市场**

MCP 市场有很多，国内常见的有 ModelScope 和 阿里云百炼。

![Image](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL2bakmHG7gRicob7WwaDUzeS9nUjzGJ4ql0ybQK3fHSPmIEYPxiaDxjaZRIibKpqH3giayW6mMqxKaibA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

ModelScope：https://modelscope.cn/mcp

![Image](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL2bakmHG7gRicob7WwaDUzeST4qhWtFYqI2m0LJ6jppZXkIrZr8ArL7HtQcsjxTyPmvCB1oDeEmWQ/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

百炼 MCP 市场：https://bailian.console.aliyun.com/?tab=mcp#/mcp-market

**4.2 MCP 学习技巧**

![Image](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL2bakmHG7gRicob7WwaDUzeLumaWC4KlIia5aWBe46NiaibcBRiafhVZQF18eYsHfPofqbgIajP7Q5p0g/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

大家还可以使用 Qwen （https://chat.qwen.ai）深入研究功能来系统学习 MCP。

![Image](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL2bakmHG7gRicob7WwaDUzeA1eib7kYxMVhDMjpOPzttRhZu9d8L1MCGH2JrYIHzT6YMLMJEumfSvg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

我们感兴趣可以 Clone Cherry Studio 的源码，使用通义灵码、 Cursor 等 AI 编程工具，打开仓库：https://github.com/CherryHQ/cherry-studio 直接对你关心的问题进行提问，学习效果更好。

![Image](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL2bakmHG7gRicob7WwaDUzeUnF4n7EgDJIvxyjpBYhVCg17t1He9QjU81IhRqtD1Y7j6uFRwm8mwA/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

大家还可以通过我们之前分享的“通俗讲解专家智能体”，先用生活化的例子快速 GET 到意思，然后专业讲解，再给出易记的方法，最后通过 SVG 图解的方式帮助我们更直观理解知识。详情参见：《[我是如何基于 DeepSeek-R1 构建出高效学习Agent的？](https://mp.weixin.qq.com/s?__biz=MzIzOTU0NTQ0MA==&mid=2247546064&idx=1&sn=eff1383fbf75c57b85ebd4f66ba31da6&scene=21#wechat_redirect)》  

五、观点

**5.1 我发现的一些问题**

**问题1：MCP 服务的配置、开关通常需要手动操作，使用方式还不够智能。这有点和 ChatGPT 的插件功能很相似，用户需要手动选择想要开启的插件，AI 还不能自动选择。**在大模型上下文和能力没有完全解决之前， MCP 下一步应该需要支持场景化（剧本式）开启和关闭，而不是全局的手动开启和关闭，开启太多 Tokens消耗大，开启太少可能影响功能的使用，某个场景在特定环节能够用到的 MCP 是相对确定的，需要类似工作流编排的形式来解决。

**问题2：如果开启大量的 MCP 服务，客户端如果第一次将所有工具信息都发给大模型让大模型来抉择，会浪费大量 Tokens。未来这部分可能需要用本地模型来实现。**

**问题3：MCP 只是解决了协议的问题，工具的稳定性很重要，调用工具时服务不可用非常影响用户体验。应该有个工具可用性检测机制，不可用时及时下线。**

**问题4：现在 MCP 服务的封装主流还是前端框架和 Python，Java 来封装 MCP 似乎不太方便或者说上手门槛略高，未来可能需要再增加一个中间层来吃掉这个复杂度。**

**问题5：现在很多模型使用 MCP 的能力还不太强，有些场景哪怕适合调用某个 MCP 服务，有些模型也只会在需要明确告知使用 MCP 才会选择使用 MCP。**

**5.2 关于 MCP 只是一种“过渡”的说法**

很多人认为 MCP 只是一个过渡，因为目前 MCP 设计存在一些局限性，而且，尽管 MCP 获得了 OpenAI、Google 等大厂的支持，但其成为永久行业标准仍存在不确定性。

我认为，MCP 确实存在一些缺点，但，MCP 确实为大模型更好地调用工具提供了一个非常好的协议。大模型想要做更复杂发挥出更大价值必然要学会使用工具，模型上下文协议非常有必要，只是未必一定是 Anthropic 这个版本。

MCP 可能是一个过渡，但这不是问题。其实 RAG 也存在诸多问题，可能也是一个过渡。其实，从更大尺度来看，我们以前用的电话、功能机、现在用的这些传统软件都是一种过渡。人，总是有时代局限性，比如我们手动挡的汽车和现在的自动驾驶没法比，也是一种过渡，但不妨碍它能够真正帮助到我们。

**5.3 展望**

当前，大模型现在还很难通过 MCP 实现人类一样的操作。比如 PPT 制作、视频剪辑，大模型还不能通过 MCP 实现类似真正和人类一样的智能的 “RPA”操作。 

随着越来越多的服务支持 MCP，随着模型的服务不断增强，面向 MCP 的业务流程编排会或许成为主流。随着 AI 使用工具的能力不断增强，未来，能够带来的帮助也会更大。

![Image](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naL2bakmHG7gRicob7WwaDUze1iaTcia1VOyKyj9VvwnGuDwXRx3BX43buMsUAfG5I5bNX1tCtibsrc1Lg/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1)

图源：https://www.dailydoseofds.com/p/a-visual-guide-to-agent2agent-a2a-protocol/

MCP 试图在解决模型使用工具的问题（就像人可以使用扳手、可以开车、可以使用的电脑等）。但是，想要完成非常复杂的事情，智能体之间必然需要协同（就像上下级之间的管理和执行，同事之间分工合作），智能体之间的标准协议非常重要。谷歌最近发布的 A2A 协议则试图解决智能体之间通信的协议问题。正如上图所示，两者可以结合使用，MCP 用户服务的注册和发现， A2A 用户智能体之间的通信。

AI 技术的发展比想象地要更快，正如 ChatGPT 刚出来时，很多人说它永远不会取代我们，它的知识不会更新，现在 AI 搜索、RAG 已经成为常态。

现在，模型虽然存在速度慢的问题，存在上下文长度限制问题，存在幻觉的问题，存在好用模型价格高的问题等，这些都将逐渐得到解决。 AI 将会更好地使用工具，Agent 也将更加智能， Agent 之间的协同也将更加通畅。
+++
title = "最近用 LLM 产品写代码的一些感想"
date = 2025-08-28T00:00:00Z
tags = ["LLM", "AI"]
categories = ["notes"]
+++

# 前言

在 ChatGPT 刚出来的时候，也就是 GPT-3 就开始用了，那个时候免费用，也勉强能用来写代码。它能在提供的沙箱环境执行 Python 代码，任何微小的校验方法编写，例如验证身份证方法直接丢给它，让它写，提示词就写：根据你学到的知识，帮我编写下面的代码—— 就是要它和训练的数据输出一模一样的内容。到后面 ChatGPT4 出来，让它转 Python 后端项目到 Java 项目，DAG 构建族谱，计算同一个祠堂下两个人的关系，两千行的代码，也能正常转，只不过提示词要多写点，还得一点点手动拆分代码。那个时候是和 Claude 一起用，即使两个都是 Pro Plan 但是架不住代码比较多消耗 Token 很快。现在只有 Claude 写的代码能勉强用于生产，因为简单的代码 Cursor 就能自动补全，GPT-5 也就那样，甚至开源的 Qwen3-Coder-480B-A35B-Instruct 写代码能力都比 GPT-5 强。

在当前阶段，LLM 根本替代不了高级开发，尤其是一些大模型，例如 Qwen3 或者 GPT5 完全比不上 Claude4.1。但是把一些琐碎的，低层级的任务丢给大模型，它会做得很好，不要一句话，“帮我把这个文档弄一下”，这种没有明确的目的废话。

正确的话术应该：编写一个 Python 程序，输入的是 Markdown 格式的文档，将文档中表格数据输出成 SQL INSERT 语句，碰到复杂的表格则跳过处理，对于 HTML 代码编写的表格不予处理，对 Markdown 语法的表格，先获取表格头部信息提取对应英文字段，创建合适的表名，转成 MySQL8 的 SQL 语法创建表，并且生成 INSERT 语句插入数据。执行 Python 程序，输出建表和插入数据的 SQL，输出文件是 init.sql。

就是有着**明确的问题，明确的处理方式，以及预期结果**，毕竟 **Garbage in, garbage out**。你不可能指望垃圾数据，片汤话就能解决问题，传统的监督学习还要一个个标注数据，大模型能做到零样本输出看似它学会了思考，学会了表达，实际不是。在它训练的数据中，如果这部分存在过，就得让它过度拟合，回答。就像做数据治理，这家厂商给的性别字段叫 gender:(male,female,unknown)，另一个厂商上报的数据是 sex(0,1,-1) 这些都可以做影射到符合国标 GB/T 2261.1-2003 的字段上，统一叫 gender(0,1,2,9)，但是有些他上报 g:(x, y, trans, plastic bag, walmart bag)，这样就输入的是垃圾信息， 别想治理出能用的数据。

下面比较的三个大模型就是 Claude4.1，GPT5，Qwen3。
# 只输入有用的

由于大语言模型本身就是根据前面的需要 attention 的 token 猜下一个 token，有着上下文限制，导致很长的代码重构完全做不了。Claude 能给个大概的规划然后写出一些能用的，符合《重构》的代码，（在没有刻意提示的情况下）例如将超过 50 行的代码拆分成一个个小的方法。像那种很复杂判断的，封装到一个可读的函数中，例如《重构》里面的例子，将复杂的日期判断，重构成  `isSummerTime`  这种可读的函数，完全就能理解这个一长串的表达式是干什么的。后面两个大语言模型在没有非常好的提示词的前提下，让它们重构，只是从一坨答辩，变成另一坨答辩+一坨答辩。当然，超过 3000 行的代码重构，Claude 也会力不从心，不能很好输出高质量的重构，只能一点点看过去。重构最重要的就是更新测试代码，或者补全测试代码，如果你不提示，后面两个几乎也不会去主动写，Claude 有时候会自己补充，即使我没有写明改完代码后写单元测试并执行。
# 无法完美解决完全陌生领域问题

如果不了解编程，不了解各种协议本身，例如 WebSocket 各种 frame，和 TCP 什么关系，可变长的 head 设计，HTTP 101 升级到该协议的过程，直接去调试或者抓这个消息，会很吃力。LLM 只能提高本来就很擅长自己领域的工作效率，做不到任何领域都从零给你由浅入深解决问题。那些给你展示什么“一人公司”，不懂代码就写出了 XXX 项目，他们只是想卖你课而已，不是所有人都有这运气和这能力的，这是不可复制的。*认识你自己*

Cursor 也不是真的完全不懂代码的人，输入他的想法，就能完成一个代码的变更。我写了个很简单的 Python GUI 程序，就是为了解决 CharlesProxy 抓不到使用不走安卓应用 WebView HTTP 代理升级的 WebSocket，HTTPToolkit 通过 VPN 形式能抓到数据，但是查看每秒几十条消息过来，看不清啊。只能问 Claude，然后推荐我使用 Frida 安卓逆向，代理应用的指定的 WebSocket 类收到消息和发送消息的方法，将数据抓取输出到日志文件，然后 Python GUI 程序类似 tail 命令过滤出收到的消息。这个 GUI 代码是我用 Cursor + Claude 4.1 MaxMode 写的（很费钱）。

这个 GUI 程序有这几个功能，自动滚动，用两个队列分别接收发送和接收到的程序， 然后过滤的时候显示出来，后面还加了个根据 contract symbol 区分是哪几个，全部都是由 Cursor + Claude 完成，我只需要简单过一遍代码，然后执行程序即可。测试不是特别了解代码，不了解代码如何编写，用自然语言描述他的需求就很困难，比如要加个 Button，清除当前 TextArea 上的所有消息，把这个需求详细描述给 Cursor，这个没练过，还真不好写一个比较合适的提示词。没有写过或者了解过 GUI 的，Button、Panel、TextArea、InputField、Label、Slider、Checkbox 这些概念的去描述新的需求，大模型只能给你个很烂的回答。
# 命令行编程

Claude Code 任何操作，都可以通过自然语言描述完成，比如让它自己给自己装个操作 Redis 数据库的 MCP，它自己就能装，也不用你费劲巴拉的从零开始，让它用 uv 创建一个 Langchain 的演示项目，也非常快，非常简单，你没有 uv，它就会去下载，也不需要你提示，全自动。理论上来说，确实是可以 24 小时不间断给你打工，但我不是 200 美元一个月的 Max 会员，Token 限制很严重，而且 Pro Plan 的账号 Claude Code 用不了 Claude 4.1。

Qwen-Code + 魔搭提供的每天免费用 2000 次，如果深度使用肯定是不够的，如果只是改改项目代码，绝对是够用的。用户协议没看，不知道是否会被魔搭社区拿去做训练数据。还有种用法就是 Claude Code Router 把 Claude Code 转成使用你自己的大模型，例如魔搭免费提供的最新的 Qwen 最强 Instruct 模型。或者更改环境变量，达到变更 Claude Code 调用 Qwen 模型。

1. 不友好的对话保存设计，Qwen-Coder 你必须输入 `/chat save` 才会主动保存你的对话，下次再进来的时候才能输入 `/chat resume` 使用上次对话。
2. 超长时间的回复，也不知道是不是我网络的问题，同样的问题，Qwen-Code 处理速度是 Claude 接近两倍左右。也就是 Claude Code 一分钟解决了，Qwen-Code 要两分钟。还不能按 Ctrl+B 让它以 background 形式运行。
3. 除非建立 .env 文件，把 BaseUrl，密钥，模型名称设置好，不然基本每次都要去 Qwen 国际版登录。
4. MCP 工具，是初始化的时候，就一个个加载，建立一个个子进程，工具一多，启动巨慢。完全可以先配置，再去通过命令 `/mcp up xx` 这样按需打开，或者用 `/mcp up *` 这样的命令打开所有的，而不是等到所有 MCP 工具准备好了再启动，这样会让别人觉得 Qwen Coder 卡死了。
5. 没有 subagent，一些完全可以独立完成的任务，非要挤在一个对话中，白白浪费了宝贵的上下文。我在用的时候还没有，刚看了最新的代码，加上了这个功能。

Qwen Code 评价，免费，但是不是那么好用，对比 Claude Code 体验差挺多的。

如果想要免费使用 Claude Code，可以更改环境变量，或者使用 Claude Code Router，把 Claude 改成魔搭社区提供的 Qwen3 模型，地址和密钥，这样就能完全免费使用 Claude Code。但是，Claude Code 的核心竞争力不在于工具，而是背后的 Claude4 或者 Claude4.1 大语言模型。
# Grok 和 Claude

最新免费使用的 grok-code-fast-1 非常快，但是写代码的能力不如 Claude，更别说真人了。唯快不破，调用起来就是快，但是不支持多模态，也就是你截图给它，它不支持。但是它免费，生成 Token 的速度最快。

免费到 9 月 10 号。尽量把简单、重复的工作丢给这个模型，复杂，难处理的，重构的，丢给 Claude，自己手动充当 GPT-5 Router。

好垃圾啊，没有哪个企业会选择使用这个大模型来协助开发人员开发的，还在优化，xAI 招了一堆大牛，后面再看吧。
# DeepSeek 官网做了很多优化

以前你直接问 DeepSeek 今天杭州天气如何，不把联网搜索打开，它就不会回答，或者乱回答。现在加了对应的 Tool 在系统提示词里面，现在即使你不打开联网搜索，它也会主动去联网搜索今天的天气。这是后端的优化。

之前画图，例如画架构图，时序图等等，它只会输出符合 Mermaid 语法的代码块，现在检测到是 Mermaid 的代码块，前端会直接渲染成图。这是前端的优化。
# 如何编写合适的问题

看下面几个视频和链接，以及样例就知道怎么写了。
1. 吴恩达提示词课程（小破站搜）
2. [Claude System Prompts](https://docs.anthropic.com/en/release-notes/system-prompts)
3. [Cline Prompts](https://github.com/cline/cline/blob/v3.25.2/src/core/prompts/system.ts)
4. [LangChain ReAct](https://smith.langchain.com/hub/hwchase17/react)

不需要像最早期的 DALL·E 3 一样，编写一系列眼花缭乱的 positive prompts 和 negative prompts 从而来实现在文生图的时候，在图片里生成准确的文字那样高深。简单，但是完整阐述你的需求，大语言模型就能精准猜出你的下一段话是什么。

23 年的时候，小破站还有所谓的“提示词工程师”推广，这种太容易被取代了，纯纯割韭菜的。模仿别人说话不会吗，有那么多可以模仿的，甚至还有这个 [Prompt Engineering](https://www.promptingguide.ai/) 给大家抄。也不需要看别人论文 [Chain-of-Thought Prompting](https://arxiv.org/pdf/2201.11903)，就能写出一手比较好的提示词。

![](https://s2.loli.net/2025/09/10/TG5xtd846IrEXUl.png)

这个网站上写的其实和我总结的一样，至于分隔符的，在吴恩达提示词课程那里有，也就是用训练的时候，\`\`\` 这三个单引号开始，三个单引号结束，作为一块内容，大模型训练的时候大部分用的都是转成 Markdown 格式的数据，所以用这个会更好识别。
## 样例

Claude System Prompts、Cline 的大泥球 Prompts、LangChain ReAct 提示词。
### Claude 

其实就是写文章一样，结构化思维。[Claude System Prompts](https://docs.anthropic.com/en/release-notes/system-prompts)
###  Cline Prompts

内容太多了，就不粘出来了。

可以看到 tag 为 3.25.2 是最后一个版本在一个文件中看见这个大泥球，后续版本优化掉了。里面还有 MCP Tool 使用的样例代码。

在这 `mcpHub.getServers().length > 0`，可以看到 System Prompts 这么长，本质上它就是拼接字符串来实现结构化返回，然后执行结构化返回内容。这么长的提示词，可见使用 Cline，Token 消耗会非常快，每次对话都要带上这样一坨。
### LangChain ReAct

让 LLM 先思考，执行，观察循环。
```plaintext
Answer the following questions as best you can. You have access to the following tools:

{tools}

Use the following format:

Question: the input question you must answer
Thought: you should always think about what to do
Action: the action to take, should be one of [{tool_names}]
Action Input: the input to the action
Observation: the result of the action
... (this Thought/Action/Action Input/Observation can repeat N times)
Thought: I now know the final answer
Final Answer: the final answer to the original input question

Begin!

Question: {input}
Thought:{agent_scratchpad}
```

总结就是思考 -> 执行 -> 观察 -> 继续决定是否继续走这一个流程，最终输出回答。
## 技巧
### 举例子（Few-shot-learning）

上面 Cline 的提示词就是很好的举例子样例，从列出 Tools 的使用，MCP Tool，再到 TODO 列表的更新全是样例，事无巨细。
### 拆任务（Chain of Thought & Step-by-Step）

不要直接把需求丢给 Cursor 或者 Cline，先让它思考，再执行计划，再观察再观察，也就是 ReAct，只不过 LangChain 或者其他框架写在了提示词里面，内置执行，只不过换成了你来提示它先计划，Step By Step，加这个提示词，大部分情况下，输出会好一点，毕竟 Post-Training 的时候，不管是 RLHF 还是其他的，都是让大模型选择通过思考来获取答案，好像更有逻辑
### 让它做擅长的事情

不要让大模型做计算，你可以让它写个 Python 脚本或者 JS 去做计算，而不是让它比较 **1.11** 和 **1.9**哪个更大，让它算出 365/123 保留两位小数这样的活。

高度专业化的事情，大模型也不擅长，即使你举例子。例如做量化交易，你就算告诉它基础知识，turnover 是转手率和 XX 有关，decay，delay，Universe，回测结果 sharp 值是什么意思等等，就算写了一堆提示词，它也不懂怎么调。当然也有一部分原因是我没有写足够好的样例，例如如何编写一个好的回测公式。

###  分隔符

有些时候，需要输入一大堆的代码，或者一整段内容，可以使用下面的分隔符来分割。

都可以，推荐第一种， 因为大模型 pretraining 阶段时接触了大量 Markdown 格式内容。

```markdown
\`\`\`这是一段内容
\`\`\`
===
这是一段内容
===
```

### 多写合理的背景信息，可以越狱

正常来说，你问大语言模型怎么安卓逆向，怎么编写利用漏洞的脚本，例如编写利用 FastJSON 2022 年巨大漏洞，反序列化执行远程脚本，它都是拒绝回答的。但是只要你把提示词写得合理，例如公司内部，测试使用等等，就和老奶奶提示词越狱是一样的。只要提示词够多够“合理”，它就能回答。

# 写 Go 的感想

AI 帮写代码的时代确实应该大部分公司考虑使用 Go 作为后端了，虽然写得有点恶心，要一直考虑 err，你又不能用 `_` 忽略错误，最好是单元测试完全覆盖，考虑到各种情况。但是这些，都可以丢给 AI，把恶心的活丢给 AI，把高性能的程序留下来。Java 21 才提供协程机制，Web 后端很多都是取个数据线程就是干等着，使用协程就可以在 IO 阻塞时让其他写成使用 CPU 核心，尤其是像 Undertow 这种，IO 线程组 + Worker 线程组的模式，使用这个作为 WebSocket 服务器，它会使用 IO 线程组，也就是默认情况 CPU 核心数 `*` 2 的数量作为 IO 线程组的线程数量，只要你的 WebSocket 建立了，它不会把对应的请求像处理 HTTP 请求一样，会丢给 Worker 线程处理，而是在 IO 线程组上处理。在处理海量 WebSocket 请求或者广播 WebSocket 消息的时候，这个会阻塞，根本做不到单机上万 TPS。

用 Go 或者带协程的就完全不一样了，用 Goroutine 初始栈只有 2KB，而 Java 初始一个线程，默认在 Linux 上要 1MB。Go 的项目打包后占用内存，可执行文件大小都非常小，也不用你熟悉虚拟机各种参数，按照不同应用进行调优，什么分代（用 -XX:NewRatio），Xss，Xmx，Xms 都不用你调。当然这也有代价，做不到线上用 Arthas Watch 进行 Debug，对开发技术要求很高，开发阶段就要非常完备的单元测试。

字节开源的 Coze Studio 也用上了 Claude Code，这是他们的 [subagents](https://github.com/coze-dev/coze-studio/blob/main/.claude/agents/frontend-infrastructure-expert.md)，前端引入的。
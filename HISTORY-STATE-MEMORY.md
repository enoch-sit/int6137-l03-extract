# 对话记录 · 状态 · 记忆

**读者：** 教师备课笔记。不发给学生。口吻按课堂，方便以后贴进 L03 PPT。  
**日期：** 2026-09-17（检索）· 第 3 周备课  
**对照：** [`../INT6137-L02/INT6137-L02.md`](../INT6137-L02/INT6137-L02.md) · L02 玻璃箱 [`https://enoch-sit.github.io/int6137-l02-chatbot/l02-chatbot.html`](https://enoch-sit.github.io/int6137-l02-chatbot/l02-chatbot.html) · **抽取演示（剧本）：** [`state-extract-demo.html`](./state-extract-demo.html) · **真抽取（全靠密钥）：** [`state-extract-live.html`](./state-extract-live.html)  
**公开页：** [真抽取](https://enoch-sit.github.io/int6137-l03-extract/state-extract-live.html) · [剧本](https://enoch-sit.github.io/int6137-l03-extract/state-extract-demo.html) · [English live](https://enoch-sit.github.io/int6137-l03-extract/state-extract-live.en.html) · [English scripted](https://enoch-sit.github.io/int6137-l03-extract/state-extract-demo.en.html)  
**本周课堂钟点仍是：** [`timeline-L03.md`](./timeline-L03.md)（Tavily / 翻译工作流）。本文件不改星期六跑场。

若只记得一句：**磁盘上的记录不是这一发。这一发不是记忆。摘要只是有损的代替。**

第 2 周带走：API 没有记忆。`messages[]` 才是对话。本笔记把那句话拆开——因为「对话」其实是四件不同的东西叠在一起，现代做法不再假装它们是同一个数组。

---

## L02 对照

玻璃箱是 L02 标本。抽取更新状态，用下面那一页单独演示。

| 格子 | 课上叫 | KEEP_EN | 它实际是 |
|---|---|---|---|
| `#messages-json` / 气泡 | 对话记录的可编辑视图 | history / transcript | 应用里那份 log。你可以存着给 UI 看。 |
| `#clipboard`（状态） | 状态 | state | 本线程你不肯丢的事实。普通文本即可，不必是 JSON。这一发可以拷进 POST。 |
| L03 [`state-extract-demo.html`](./state-extract-demo.html) | 抽取演示（剧本） | extract | **另一页。** 下一句走写好的订位；空白密钥也填卡片。不要和十五变量玻璃箱混在一页。 |
| L03 [`state-extract-live.html`](./state-extract-live.html) | 真抽取 | extract (live) | **再一页。** 必须有密钥。你打字 → chat POST；抽取 → 第二次 POST。没有剧本。黄标只在 value 对上原句时出现。 |
| L03 [`state-extract-live.en.html`](./state-extract-live.en.html) / [`state-extract-demo.en.html`](./state-extract-demo.en.html) | English class | extract (en) | Same two pages in English. Same IDs and pipeline. Booking example uses Chen / two → three. |
| 视窗策略「摘要 / 混合」 | 占位摘要 | summary（占位） | `【摘要·占位】` 只演示**槽位**。它不是真的摘要，更不是记忆。 |
| 页面里的 JS `state` | 应用状态 | — | 整页控件的对象。课上不要叫「记忆」。 |

L02 第 13 点四个动作仍然成立：写入、选择、压缩、隔离。本笔记只把「压缩之后还剩什么」说清楚。

向量库、跨周长期记忆产品，仍是第 5 周。本课的 **memory** 不是那个。

---

## 一张图

```text
磁盘 / 页面上的对话记录     history（可以很长；模型看不见磁盘）
        │
        │  选择 · 压缩 · 丢掉
        ▼
这一发真正送出的 messages[]   工作集（视窗里的 RAM）
        │
        │  模型只读这一发
        ▼
回复 + usage

旁边两条，不走「整段 log」：

  状态 state     本线程事实 → 需要时写进这一发（L02 的 <state>…</state>）
  记忆 memory    视窗装不下时，先放到数组外面；下一发再注入
  摘要 summary   被丢掉的旧轮次的有损代替 → 通常变成一条 system
```

Karpathy / LangChain 的比方（课堂可说）：模型像 CPU，上下文视窗像 RAM。上下文工程 = 决定 RAM 里放什么。硬盘上的 history 不会自动进 RAM。

---

## 1 · 什么是 message

一条 **message** 是信封上的一格：`{ role, content }`。Provider 会把它换成模型训练过的 chat 标记。

| 现在就要会指 | 以后才会多一格 |
|---|---|
| `system` / `user` / `assistant` | `tool`（课表第 3 周；本笔记不练） |

它**不是**：

- 屏幕上的气泡（气泡可以藏 `<state>`，JSON 里仍在）
- 整段对话记录（那是数组，不是一条）
- 状态对象本身（除非你选择把状态序列化写进某条 `content`）

现代接口里，一轮里还可能出现「工具结果」「压缩块」这类条目。课堂只需记住：**能进这一发、会被模板化的，才算这一发的 message。** 气泡里有、请求里没有 = 模型没看见。

---

## 2 · 对话记录 history

**history**（transcript）是应用保存的 log：谁说过什么、按时间排。

- UI 可以显示全部。导出 JSON 可以是全部。
- **模型每一发都是失忆的。** 没写进这一发 body，等于没发生。
- 所以 history 可以在磁盘上很长，而这一发只带最近 N 轮。那不是 bug，是选择。

L02 第 12 点：UX 有状态，API 无状态。history 属于 UX / 磁盘。它不是 API 的记忆。

长记录的三件事：

1. 越长越贵（每轮 `prompt_tokens` 把旧账再付一次）。
2. 视窗是硬边界，满了会爆或被截。
3. 就算塞得下，旧的、跑题的、工具长文也会把模型带偏。所以「全送」不是默认正确答案。

---

## 3 · 状态 state

**state** 不是另外编的一套故事。它是你从 **history** 里**抽出来**的事实：丢掉下一发就会做错或伤人的那些。L02 玻璃箱叫它 **状态**。

- 普通文本即可。JSON 只是你偶尔选用的写法，不是类型。
- 卡片**独立存放**，但内容**来自对话**。改气泡不该偷偷改状态（L02 已教）；该改的是你抽错了、或用户更正了。
- 它怎么进模型？只有一条路：这一发拷进去（接到 `system` 或最新 `user`）。关掉注入 = 这一发模型看不见。

发明状态，是为了：**对话可以变长、被压缩、被丢掉；抽出来的那一页还在。** 摘要会说错名字、丢掉「不要花生」。状态是你拒绝交给运气的那一页。

### 例子 · 对话里加粗 = 抽进状态的词

一段平常的订位闲聊。**加粗**是关键词：同一批词写进状态。没加粗的（寒暄、客套、跑题）不进卡片。以**最后一次更正**为准（**两位** → **三位**）。

课堂：每拍先读对话、指加粗，再看卡片多了哪一行。可点版本：[`state-extract-demo.html`](./state-extract-demo.html)（下一句 → 抽取；空白密钥走剧本）。自己打字、两次都是真 POST：[`state-extract-live.html`](./state-extract-live.html)（必须填密钥）。

**第 1 拍 · 还没抽出**

同学：嗨，今晚想订个位子～  
店员：欢迎。请问怎么称呼、几位？

```text
stage: 问姓名
name:
party:
```

**第 2 拍 · 抽出 陈、两位**

同学：我姓 **陈**。先订 **两位** 吧。  
店员：陈先生，两位。想吃粤菜还是日料？

```text
stage: 问菜系
name: 陈
party: 2
```

**第 3 拍 · 抽出 日料、花生过敏**

同学：**日料** 好了。对了 **花生过敏**，千万别花生。  
店员：菜单会避开花生。要靠窗吗？

```text
stage: 问座位
name: 陈
party: 2
cuisine: 日料
allergy: 花生 — 不要
```

**第 4 拍 · 抽出 靠窗；两位被更正成三位**

同学：要 **靠窗**。哦改成 **三位**，小孩也来。  
店员：改成三位。要儿童椅吗？

```text
stage: 问儿童椅
name: 陈
party: 3
cuisine: 日料
allergy: 花生 — 不要
window: 是
```

**第 5 拍 · 抽出 不用椅；好评那句丢掉**

同学：**不用** 椅。你们店好评好像很多啊……  
店员：谢谢。今晚 **7 点** 可以吗？

```text
stage: 问时间
name: 陈
party: 3
cuisine: 日料
allergy: 花生 — 不要
window: 是
highchair: 否
```

**第 6 拍 · 抽出 7 点；最后一句在考记忆**

同学：**7 点**。那我刚才说过敏了吗？

```text
stage: 确认
name: 陈
party: 3
cuisine: 日料
allergy: 花生 — 不要
window: 是
highchair: 否
time: 19:00
```

同一段对话，只看同学侧、加粗即关键词（PPT 用荧光笔）：

> 嗨，今晚想订个位子～  
> 我姓 **陈**。先订 **两位** 吧。  
> **日料** 好了。对了 **花生过敏**，千万别花生。  
> 要 **靠窗**。哦改成 **三位**，小孩也来。  
> **不用** 椅。你们店好评好像很多啊……  
> **7 点**。那我刚才说过敏了吗？

词 → 卡片（更正只留最后一版）：

| 对话里的加粗 | 状态里写成 |
|---|---|
| **陈** | `name: 陈` |
| **两位** → 后来 **三位** | `party: 3`（2 被覆盖） |
| **日料** | `cuisine: 日料` |
| **花生过敏** | `allergy: 花生 — 不要` |
| **靠窗** | `window: 是` |
| **不用**（儿童椅） | `highchair: 否` |
| **7 点** | `time: 19:00` |
| 嗨 / 好评好像很多 / 谢谢 | （不写） |

问全班：如果现在只留最后一句「那我刚才说过敏了吗？」，没有状态卡片，模型还知不知道 **花生**、**三位**、**陈**？

对照三种坏法：

```text
坏 1  把整段 history 每轮重发
      → 贵，而且「两个人」和「三个」同时在，模型可能订错

坏 2  只做摘要：「客人聊了订位，改过人数，提到饮食。」
      → 情节还在，花生没了，三位可能变成「几位」

坏 3  凭空发明状态（还没人说花生，卡片上先写过敏）
      → 那不是抽取，是瞎编。状态没有的事实，不要写
```

好的做法：每出现一句**更正或硬约束**，立刻改卡片。history 仍可留下给 UI；这一发带 **短摘要 + 状态卡片 + 最近原文**。

L02 玻璃箱默认那几行，就是空卡片等对话来填：

```text
stage: ask_name
name:
topic:
```

### 玻璃箱演示 · 抽取是另一次 POST

口令：聊天是一次请求；**更新卡片是另一次请求。** 结构在卡片上，不在闲聊里。

1. 学生用玻璃箱聊订位（或任何会说出姓名的话）。**不要**用手改 状态。
2. 指 `messages[]`：那是对话。指卡片：还是空的 `name:`。
3. 点 **抽取**（在 状态 旁边）。请求体变成**另一组** `messages[]`：system 写「只做抽取 + 卡片结构」，user 带着对话记录。温度 0，不开 stream。
4. 回来之后：卡片填上了；气泡没有新的助手格；HTTP 状态写「抽取 HTTP …」。这就是独立 API。
5. 再点发送：请求体又变回聊天那一组，并可把新卡片注入 `<state>`。

不要说「模型自己会改状态」。模型只回这一发的 `content`。是**你的程序**把抽取结果写进卡片。

`name:` 空着，是因为用户还没说。他说了「我姓陈」，程序（抽取 POST，或你当老师用手填）才写上。状态是抽取器的输出，不是第二份小说。

不要和另外两个「state」混：

| 别人也叫 state | 本课怎么说 |
|---|---|
| 玻璃箱 JS `state` | 应用状态，页面对象 |
| 图工作流整份检查点（含 messages 和其他字段） | 应用把线程存盘；第 4 周再碰。不要叫记忆 |

---

## 4 · 摘要不是记忆

工业界现在把三件事拆开。课上用三个动词就够。

| 做法 | KEEP_EN | 干什么 | 丢不丢得起 |
|---|---|---|---|
| 把旧轮次压成一段话，塞回数组 | summary | 让对话在视窗里继续 | **有损。** 细节可能没了。磁盘上的 history 仍可另存 |
| 丢掉可以再取回的长文（旧搜索结果、整篇网页） | 丢掉 / 清掉 | 腾 RAM；需要时再调工具 | 丢的是副本，不是唯一事实 |
| 把必须活下去的事实放到数组**外面** | memory | 压缩之后、甚至下一场会话，还能再注入 | 这才是「记住」；责任在你的程序 |

**summary** 是压缩 **history** 的方法。它不是 memory。

- 摘要住在下一发的 `messages[]` 里（常常是一条 `system`）。
- 记忆住在数组外面（一张小卡片、一行状态、一个文件）。下一发你再决定写不写进去。
- 把摘要当成唯一档案 = 没有档案。审计、评分、回放，要看 history 或你另存的记录。

L02 视窗三种策略对照：

| L02 策略 | 现在怎么讲 |
|---|---|
| 滑动 | 只留最近原文。旧的直接丢。没有摘要，也没有记忆。 |
| 摘要占位 | 假装有一条 summary。玻璃箱没有真的读懂旧轮次。 |
| 混合 | 摘要 + 最近原文。现代聊天/代理的常见形状。 |
| （本笔记补的）记忆 | 混合之后仍怕丢的：放进 状态 / 记忆，不要只靠那条摘要。 |

第 5 周的向量检索，是另一种「数组外面」。本周不要把 memory 讲成「做一个向量库」。先会：一张明文卡片，能再注入。

---

## 5 · 这一发 messages[] 里有什么

每一发是一次**组装**，不是把磁盘倒进去。

典型工作集（课堂默认混合）：

```text
1. 短的 system（人设、规则；不要把整本手册塞这里）
2. 一条摘要（若已经压缩过旧轮次）
3. 状态 / 记忆里这一发需要的事实（拷进来，或本来就在 system）
4. 最近 N 轮 user / assistant 原文
5. 最新这一句 user
```

四格预算仍是 L02 的：`system` / 历史（含摘要）/ 最新 user / 模型输出。多出来的状态注入，算进 `system` 或最新 user，不要假装它不占 token。

所以：

- **history** = 你可能拥有的全部。
- **messages[] 这一发** = 你决定让模型看见的切片。
- 「模型记得」= 你把切片重发了。

---

## 6 · 什么必须留下

问的不是「什么有意思」，是：**丢掉之后，下一发会做错或伤人吗？**

L02 第 13 点已经给过隔离：寒暄可丢；过敏 / 订单号不能丢。下面是同一条规则的现代表。

| 必须留下（写进 状态 / 记忆，或留在最近原文） | 可压成摘要 | 可丢掉 |
|---|---|---|
| 用户更正、「不要再做 X」 | 已完成支线的结局一句 | 寒暄、「好的」「Sure」 |
| 当前目标 + 阶段（例如还在问姓名） | 旧脑暴过程 | 可再搜一次的长文、整页 HTML |
| ID、路径、URL、姓名、过敏、订单号 | 很久以前的闲聊情节 | 失败重试的中间过程 |
| 仍有效的报错原文 | — | 下一发会重新注入的那段 system |
| 最近 N 轮原文（正在做的事） | — | 已被新摘要取代的旧摘要 |

优先级（必须砍的时候）：**用户更正 > 仍有效的错误 > 正在做的工作 > 已经做完的工作。** 最近的原文权重大；开头的寒暄最先丢。

不要留给摘要的：数字、名字、否定句。摘要擅长情节，不擅长「花生过敏」「订单 A-1042」「不要再用红色」。那些进 状态 / 记忆。

工具还没作为本周作业。但 keep/drop 现在就要说：工具吐出来的长结果，默认是「可再取回的副本」，不是对话记录的永久正文。

---

## 7 · 常见误读

| 误读 | 纠正 |
|---|---|
| 气泡还在 = 模型记得 | 气泡是 UX。没进这一发 `messages[]` = 没发生。 |
| 状态是另外编的档案 | 状态从对话**抽取**。没说过的事实不要写；说过又更正的，以最后为准。 |
| 状态必须是 JSON | 状态是事实文本。JSON 是可选写法。信封才是 JSON。 |
| 摘要 = 记忆 | 摘要是有损代替，住在数组里。记忆住在数组外面，靠你再注入。 |
| 把全部 history 每轮重发 = 负责 | 又贵又吵。选择 / 压缩 / 丢掉也是负责。 |
| 应用把线程存盘 = 模型有记忆 | 存盘让*程序*续跑。模型仍只看见这一发。 |
| ChatGPT / Claude 产品里的 Memory 开关 = 本课的 memory | 产品名。本课指：你自己把事实放到数组外面。 |
| 视窗更大 = 不用再管 | 视窗是 RAM。RAM 变大，乱塞照样偏、照样贵。 |
| 第 5 周向量库 = 现在的作业 | 先会明文卡片。向量是以后的检索，不是本周的 keep。 |

---

## 8 · 教师自测

不看厂商文档，要能口头答：

1. **history 是什么？** 应用保存的对话 log；可以比这一发长；模型不读磁盘。
2. **state 是什么？** 从对话抽出的、本线程不肯丢的事实（L02 状态）；文本；这一发才拷进去。不是另编的故事。
3. **memory 是什么？** 放到 `messages[]` 外面、压缩之后还能再注入的事实。不是摘要，不是向量作业。
4. **一条 message 是什么？** `{ role, content }`（以及后来的工具格）。不是气泡，不是整段记录。
5. **什么必须留下？** 更正、正在做的目标、标识与过敏订单、仍有效的报错、最近原文。寒暄、可再取的长文、失败过程可丢。情节可摘要。数字与否定句不要只靠摘要。

日后做 PPT：一页图、一页三名词、一页订位抽取例子、一页 keep/drop、一页误读。本文件就是那几页的正文。

---

## 来源

检索日期 **2026-09-17**。英文原句只出现在这里。正文不写厂商路径名。

1. LangChain, *Context Engineering* — https://www.langchain.com/blog/context-engineering-for-agents  
   > “The LLM is like the CPU and its context window is like the RAM.”  
   同文：runtime state 可以把 `messages` 以外的字段隔开，需要时再给模型。

2. Anthropic Claude Cookbook, *Context engineering tools* — https://github.com/anthropics/claude-cookbooks/blob/main/tool_use/context_engineering/context_engineering_tools.ipynb  
   > “compaction compresses the whole window when it grows too large, clearing drops stale re-fetchable data inside the window, and memory moves information out of the window so it survives across sessions.”  
   摘要是有损的整段压缩；可再取的工具结果该丢掉，而不是当圣经。

3. Anthropic Cookbook, *Session memory compaction* — https://platform.claude.com/cookbook/misc-session-memory-compaction  
   必须留下：identifiers、values、user corrections、errors、active work。可省略 pleasantries，以及「反正下一发会再注入的 system」。砍的时候：corrections > errors > active work > completed work。

4. OpenAI, *Compaction* — https://developers.openai.com/api/docs/guides/compaction  
   > “reduce context size while preserving state needed for subsequent turns.”  
   压缩之后的窗口是下一发的工作集，不是完整档案。

5. LangGraph, *Add memory* — https://docs.langchain.com/oss/python/langgraph/add-memory  
   线程内：`messages` 旁边可以另有一个 `summary` 字段；旧消息删掉，摘要留下。检查点是程序续跑，不是模型记忆。

6. （对照，非厂商）Tamar Peretz, *LLM Agent Memory Architecture* — https://www.andyagentlab.com/articles/agent-architecture/llm-memory-architecture/  
   > “Compacted or summarized state [is not] a complete source-of-record history or a general long-term memory architecture.”

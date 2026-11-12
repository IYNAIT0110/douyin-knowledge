# 深度总结（中英对照）· LLM Wiki 取代 RAG：把"检索"挪到入库时

# Deep Dive (Bilingual) · LLM Wiki over RAG: Moving Retrieval to Ingest Time

> 原视频：https://v.douyin.com/s2lsiCV7XlY/ （作者「顶级AI宗宗主」，3:51）
> 分享时间：2026-11-12 07:45 ｜ 逐字稿 `transcripts/2026-11-12/7706491315030101267.txt`
> 姊妹篇：想直接搞懂内容，看 `2026-11-12-LLMWiki取代RAG-深度总结.md`（纯中文知识向）；本文面向"顺便练英语"。
> 素材为**中文原声**，英文为**译写**（not a literal gloss）：按原意用地道英文重写，保留原句的信息结构，不逐字硬译。
> 逐字稿系 ASR，专名误听已订正：LLM Vicky→LLM Wiki、Android Carpathity→Andrej Karpathy、在线围机→在线维基、RAJ/LAG→RAG、相量→向量、可毒性→可读性、净品调研→竞品调研、翻文见贵→翻文件柜、全天后→全天候、越耗补→越厚实（共 53 处，全表见主报告）。
> 视频中 53–60s 为引流口播（学习路线 PDF），已剔除不译。

---

## 怎么用这份对照版 / How to Use This File

**EN** This file uses a **fading scaffold**: dense support at the start, thinning out as you go.
> **中** 这份文件用**渐隐支架**：开头扶梯最密，越往后越少。

**EN** **Part A** is line by line — read the English first, and only glance at the Chinese when you are genuinely stuck.
> **中** **A 段**逐句对照：先读英文，真的卡住才看下面那行中文。

**EN** **Part B** gives a whole English paragraph before the Chinese. Survive a paragraph, then check.
> **中** **B 段**先整段英文，再整段中文：撑完一整段再去核对。

**EN** **Part C** is English only, with the Chinese folded away. Use it to check yourself, not to read along.
> **中** **C 段**只有英文，中文折叠起来：用来自查，不是用来对照着读。

**EN** Every Chinese line sits in a quote block, so English stays the default thing your eyes land on.
> **中** 所有中文都放在引用块里，让英文始终是眼睛的默认落点。

---

# Part A · 核心论点（逐句对照）
# Part A · Core Arguments (Sentence by Sentence)

## A1. "RAG is dead" and the question behind it

**EN** "Nobody uses RAG any more — it is completely obsolete, and everyone has moved to LLM Wiki."
> **中** "RAG 已经没人用、彻底过时，现在全都上 LLM Wiki。"

**EN** That claim has been spreading fast lately.
> **中** 最近网上这个论调越传越火。

**EN** So what exactly is an LLM Wiki? One short video, and you will have it down cold.
> **中** 那到底什么是 LLM Wiki？今天一条视频把它讲透。

**EN** In AI application development, RAG — retrieval-augmented generation — has always been the default recipe for knowledge bases.
> **中** 在之前的 AI 应用开发里，RAG（检索增强生成）一直是知识库的标配。

**EN** You upload a pile of documents; when the user asks a question, the system retrieves the relevant fragments, hands them to the model, and the model writes the answer.
> **中** 上传一堆文档，用户提问的时候，系统检索相关片段交给大模型参考，再输出答案。

## A2. The weakness is structural

**EN** But this recipe has a weakness it was born with.
> **中** 但这套方案有个天生短板。

**EN** Fragments are fetched on the fly, only when a question arrives — so the knowledge never truly settles.
> **中** 每次提问才临时捞取碎片，知识没法真正沉淀。

**EN** The model passively pulls up slices; it never actively digests the material or connects the new with the old.
> **中** 模型只是被动调取切片，不会主动消化、串联新旧资料。

**EN** In 2026, Andrej Karpathy proposed LLM Wiki, and a wave of open-source implementations followed — and together they replaced that old paradigm.
> **中** 而 2026 年 Andrej Karpathy 提出的 LLM Wiki，加上后续一系列开源实现，直接换掉了这套老范式。

## A3. The shift: the model becomes the editor

**EN** It no longer waits for the user to ask a question before digging through the raw documents.
> **中** 它不再等到用户提问的时候才翻原始文档。

**EN** Instead, the large model acts as the editor and curator of the knowledge base.
> **中** 而是让大模型充当知识库的编辑与管理员。

**EN** The moment a document lands in the repository, it gets a deep read — summarised, organised, filed.
> **中** 文档入库的那一刻就深度精读、归纳、梳理。

**EN** What grows out of it is a living wiki whose pages link to one another, cross-reference, and cite.
> **中** 持续搭建一份可以互相跳转、关联引用的在线维基。

## A4. Four differences, one variable

**EN** Put traditional RAG and LLM Wiki side by side, and the gap is obvious at a glance.
> **中** 把传统 RAG 和 LLM Wiki 放在一起对比，差距一眼就能看懂。

**EN** First, when the knowledge gets processed: RAG assembles retrieved fragments on the fly, at query time, and throws them away after use.
> **中** 第一，知识处理时机：传统 RAG 提问时临时组装检索出来的片段，用完即抛。

**EN** LLM Wiki front-loads the computation to the ingest stage — the knowledge is compiled ahead of time and kept.
> **中** LLM Wiki 把计算前置到素材录入阶段——提前编译知识，长久保存沉淀。

**EN** Second, the form of the information: RAG stores chunks and vector fragments; LLM Wiki produces structured wiki pages with bidirectional links, plus a graph of how concepts relate.
> **中** 第二，信息组织形式：RAG 存的是切分后的文本块、向量碎片；LLM Wiki 产出结构化双向链接的维基页面，附带知识关联图谱。

**EN** Third, how new material is handled: RAG just chunks the new document and files it — old and new content never touch.
> **中** 第三，新增资料的处理逻辑：传统 RAG 只是把新文档切块入库，新旧内容互不打扰。

**EN** LLM Wiki actively compares the newcomer against what it already knows: it creates pages for new concepts, updates the old ones, and flags where the information conflicts.
> **中** LLM Wiki 会主动比对已有知识：生成新概念页面、更新旧内容，还能识别标注信息冲突。

**EN** Fourth, readability: vectors and slices are nearly impossible for a human to read; the wiki outputs tidy, note-style pages that a person can read, browse, and trace back to the source.
> **中** 第四，可读性：RAG 的向量和文本切片人很难直接看懂；LLM Wiki 输出的是规整的笔记式页面，人可以直接阅读、查阅、溯源。

## A5. The closing metaphor

**EN** In one sentence: traditional RAG is you, at question time, rushing to a search engine and digging through a filing cabinet.
> **中** 最后一句话总结：传统 RAG 相当于你提问的时候临时找搜索引擎翻文件柜。

**EN** LLM Wiki is hiring a scholar who is online around the clock, reading everything you own, and compiling it into an encyclopedia that cross-checks itself and keeps growing.
> **中** 而 LLM Wiki 等于雇了一位全天候在线的学者，不停阅读你所有资料，帮你编纂一部互相印证、持续生长的百科全书。

**EN** The more material you feed in, the more dimensional — and the thicker — this knowledge base becomes.
> **中** 喂进去的素材越多，这份知识库就越立体、越厚实。

---

# Part B · 论证展开（整段英文 → 整段中文）
# Part B · The Argument in Full (English Paragraph First)

## B1. Why "Use It and Throw It Away" Is Structural

**EN**
The weakness is not a bug someone forgot to fix; it follows directly from where the computation happens. If the system only touches your documents at query time, then everything it does is disposable by construction: fragments are fetched, stitched into a prompt, consumed, and discarded. Nothing that happened in this query makes the next one cheaper or better answered. The model never gets to digest anything — it is a courier, not a reader. And when the second document arrives, the first one is still just a pile of slices in a drawer; the two never meet. Knowledge that never settles cannot compound. That is the real cost of "on the fly": not slow answers, but an answer that is exactly as hard to produce the hundredth time as it was the first.

> **中**
这个短板不是谁忘了修的 bug，它直接由"计算发生在哪一刻"决定。如果系统只在提问时才碰你的文档，那它做的一切天然都是一次性的：片段被捞出来、拼进 prompt、用掉、扔掉。这次问答里发生的一切，不会让下一次更便宜、也不会答得更好。模型从来没有机会消化任何东西——它是快递员，不是读者。而第二份文档进来时，第一份在抽屉里还是一堆切片，两者永远不相遇。沉淀不下来的知识没法复利。这才是"临时捞取"真正的代价：不是答案慢，而是第一百次提问和第一次一样费力。

## B2. Three Layers, and Why the Source of Truth Must Not Move

**EN**
A full LLM Wiki stack splits into three layers. The bottom is the raw-material layer: original PDFs, web pages, meeting notes — kept as an immutable source of fact, so that everything can be traced back. The middle is the wiki layer, maintained entirely by the model: summary pages, entity pages, topic pages; it adds pages, deletes pages, updates content, and keeps the bidirectional hyperlinks intact. The top is the rules layer — a preset set of prompts telling the model what structure the knowledge base should follow, how to classify a new concept, and what to do when two pieces of information contradict each other. The point of the split is that the bottom layer never moves. The wiki layer can be wrong, deleted, rebuilt at will — because the facts are still lying underneath, untouched. Separating truth from rendition is what makes an AI-maintained knowledge base safe to trust: you are never more than one link away from the original document.

> **中**
整套 LLM Wiki 一般分为三层。底层是原始素材层：原始 PDF、网页、会议纪要——作为不可篡改的事实源存放，所有内容都可以回溯。中间是维基呈现层，由大模型全权维护：摘要页、实体页、主题页；负责增删页面、更新内容、维护双向超链接。顶层是规则约定层——预设的提示词规范，告诉模型知识库该遵循什么样的结构、怎么归类新概念、如何处理互相矛盾的信息。分层的要点在于底层永远不动。维基层可以出错、可以被删掉、可以随意重建——因为事实还原封不动地躺在底下。把"真理"和"转述"分开，才是 AI 维护的知识库敢让人信的底气：你永远离原始文档只差一个链接。

## B3. The Two-Phase Ingest, and Why Conflicts Get Annotated, Not Overwritten

**EN**
Drop a new document in, and it walks through two phases. First, analysis: the model reads the whole thing, extracts the core claims, then searches the existing knowledge base to sort out what is genuinely new and what conflicts with the record. Then, generation: it writes and revises pages like a wiki editor — creates entries for new entities, fills in details on existing ones, and adds cross-links between related pages. The subtle part is what happens when a real conflict shows up: it does not overwrite. Instead it writes the disagreement into the entry, with the sources labelled — source A says this number, the newer source B concludes that. This single design choice carries a whole epistemology: a knowledge base is not a collection of truths, it is a collection of claims with provenance. A system that silently overwrites will quietly lose the counter-evidence the day the second document arrives.

> **中**
丢进去一份新资料，它会走两大流程。先是分析阶段：大模型通读全文、提炼核心论点，然后检索现有知识库，区分哪些是全新信息、哪些和过往记录存在冲突。再是生成阶段：像维基编辑一样撰写、修订页面——给新实体建词条、补充原有词条的细节、给相关页面加上关联。微妙的地方在真正遇到冲突时怎么处理：它不覆盖。而是把分歧写进词条里，标好来源——来源 A 给出的数据是这个数，最新的来源 B 结论又是那个。这一个设计选择背后是一整套知识观：知识库不是真理的集合，是**带出处的论断**的集合。一个只会静默覆盖的系统，会在第二份资料到来那天，悄悄丢掉第一份里的反例。

## B4. Two Scenarios Where This Actually Pays

**EN**
It fits two kinds of scenario especially well. One is the internal company knowledge base: team knowledge is scattered across chat logs, requirement docs and meeting recordings, and nobody wants to organise it by hand. Plug this in, and daily communication settles on its own into a systematic encyclopaedia — a new hire gets a coherent internal manual that is continuously updated, instead of six months of archaeology. The other is competitive research and complex intelligence work: data from different channels often disagrees, and traditional RAG, which retrieves a random chunk, easily generalises from one side of the story. LLM Wiki catches the contradiction at ingest time and spells it out in the entry — what source A reported, and what the latest source B concludes — which is exactly how you avoid walking into an information trap. Notice what the two scenarios share: the value is not faster answers, but answers that stay trustworthy when the sources fight each other.

> **中**
它尤其适合两类场景。一类是企业内部知识库：团队知识散落在聊天记录、需求文档、会议录音里，没人愿意手动整理。接入之后，日常沟通自动沉淀成体系化的百科——新人上手直接拿到一份持续更新、逻辑连贯的内部手册，而不是做六个月的考古。另一类是竞品调研这种复杂情报：不同渠道的数据经常打架，传统 RAG 随机捞出一段，很容易以偏概全。LLM Wiki 在入库阶段就识别出矛盾，并在词条里写清楚——来源 A 报的是什么数、最新的来源 B 结论又是什么——这正好帮你避开信息陷阱。注意这两个场景的共同点：价值不在"答得快"，而在**信息源互相打架时答案仍然可信**。

---

# Part C · 延伸（纯英文，中文折叠自查）
# Part C · Extras (English Only — Chinese Folded for Self-Check)

## C1. One Variable That Generates All Four Differences

*Terms: **front-load** = do the heavy work early; **compound** = grow on itself, like interest; **disposable** = used once, then discarded.*

The four-row comparison table looks like four features. It is really one variable with four consequences: when does the knowledge get processed? If the answer is "at query time", then everything downstream is forced — the system has milliseconds, so it can only slice and fetch; slices have no structure, so old and new never meet; and the artefacts are vectors, which no human can read. If the answer is "at ingest time", the opposite of each line follows for free: there is time to build structure, the newcomer meets the archive, conflicts get detected because someone is actually reading, and the output is pages a person can open. This is worth memorising as a pattern: when you meet any new tool in this space, do not study its feature list — ask where the computation happens. Feature lists expire; variables do not.

<details>
<summary>中文自查（读完英文再看）</summary>

那张四行对比表看着是四个 feature，其实是一个变量和它的四个后果：**知识在什么时候被加工？**如果答案是"提问时"，下游全是被逼出来的——系统只有几毫秒，所以只能切片捞取；切片没有结构，所以新旧永不见面；产物是向量，人读不懂。如果答案是"入库时"，每一行的反面都免费跟上：有时间建结构、新来的能碰上存量的、冲突能被发现（因为真的有人在读）、产物是人能打开的页面。这一招值得记住当成模式：以后遇到这个领域的任何新工具，别研究它的 feature 清单，问"计算发生在哪一刻"。feature 清单会过期，变量不会。

</details>

## C2. "X Is Dead" Is a Marketing Sentence — the Honest Architecture Is Hybrid

*Terms: **substrate** = the underlying layer something sits on; **latency** = delay before a response; **multi-tenant** = one system serving many isolated customers.*

External fact-checking (done after the video, not in it): Karpathy did propose this in April 2026 — an "idea file" on GitHub, with his own research wiki at roughly a hundred articles and four hundred thousand words. But every serious write-up adds the same three caveats the video skips. Hallucination baking: a wrong synthesis becomes a wrong page that every future query trusts — organised, persistent mistakes are harder to spot than scattered ones, so the lint pass must hunt for plausible-but-wrong claims, not just broken links. Ingest is not free: the heavy work moved, it did not vanish; with fast-changing data, re-compiling costs more than retrieving. Scale ceiling: once the wiki outgrows the context window, you need a retrieval substrate underneath — and the cheapest one is, ironically, vector embeddings. RAG also still wins outright at millions of documents, hard latency limits and multi-tenant serving. The honest summary circulating in the field is three words: complement, not replace.

<details>
<summary>中文自查（读完英文再看）</summary>

外部核实（视频之后做的，不是视频里的）：Karpathy 确实在 2026 年 4 月提出了这套东西——GitHub 上一份 "idea file"，他自己的研究 wiki 约 100 篇文章、40 万词。但每一篇认真的文章都补上了视频略过的三个坑。**幻觉烘焙**：编译时写错的综合会变成一页错的知识，之后每次提问都信任它——有组织、持久化的错误比零散的更难发现，所以 lint pass 要专门猎杀"看着很像真的"的论断，不只是断链。**入库不免费**：重活是搬家了，不是消失了；数据变得快时，反复重编译比检索更贵。**规模天花板**：wiki 长到超出上下文窗口后，底下还得垫一层检索 substrate——最便宜的恰好又是向量嵌入。RAG 在百万级文档、硬延迟、多租户场景也仍然完胜。业界流传的诚实总结是三个词：**complement, not replace（互补，不是替代）**。

</details>

## C3. The Five-Minute Test You Can Run on Any System You Own

*Terms: **recurring** = appearing again and again; **entity page** = one page per concept or thing; **payoff** = the return you get for the effort.*

Here is the portable version of the whole video, for any knowledge system you personally run. One question: do the same things keep showing up? If the same class of problem, the same error, the same question recurs across your records, then a query-time-only system is quietly wasting it — every recurrence is retrieved from scratch, and the common thread across the records lives nowhere except in your head. That is the signal to hand-write one entity page: what it looks like, what causes it, how it was fixed, which records it links to. Do it by hand once, before automating anything — fifteen minutes of manual labour tells you whether the pattern fits your data or fights it, which no tutorial can. And run the three caveats from C2 as eligibility checks first: if your corpus changes by the hour, or you need answers in fifty milliseconds, or it has outgrown any context window — stop, the wiki layer is the wrong tool. Otherwise, the smallest honest experiment is one page.

<details>
<summary>中文自查（读完英文再看）</summary>

这是整个视频的可搬运版，适用于你自己在用的任何知识系统。一个问题：**同样的东西是不是反复出现？**如果同一类问题、同一个报错、同一个提问在你的记录里反复出现，那纯"提问时"系统正在悄悄浪费它——每次重现都从零检索，而横跨这些记录的那条主线，除了你脑子里哪儿都不存在。这就是该手写一张实体页的信号：它长什么样、根因是什么、当时怎么修的、关联哪些记录。动手之前先**手工**写一张——十五分钟的人肉劳动能告诉你这个模式在你的数据上是顺是逆，任何教程都给不了这个。先拿 C2 的三个坑当资格检查：如果你的语料每小时都在变、或你需要 50 毫秒内出答案、或它已经大到塞不进任何上下文窗口——停下，维基层是错的工具。否则，最小诚实的实验就是：**先写一页**。

</details>

---

# 词块表 / Chunk Table

> 不要背单词，背这 18 个可直接搬用的搭配。Don't memorise words — memorise these 18 ready-to-use chunks.

| # | Chunk（词块） | 意思 | 可以直接这么说 |
| --- | --- | --- | --- |
| 1 | **the claim is spreading fast** | 论调越传越火 | *That claim has been spreading fast lately.* |
| 2 | **at query time** | 提问时（检索发生的一刻） | *Fragments are fetched at query time.* |
| 3 | **at ingest time** | 入库时 | *The computation is front-loaded to ingest time.* |
| 4 | **a weakness it was born with** | 天生短板 | *This recipe has a weakness it was born with.* |
| 5 | **fetch fragments on the fly** | 临时捞取碎片 | *It fetches fragments on the fly, only when a question arrives.* |
| 6 | **use it and throw it away** | 用完即抛 | *Retrieve at query time, use it and throw it away.* |
| 7 | **knowledge never settles** | 知识没法沉淀 | *If it never settles, it cannot compound.* |
| 8 | **the model acts as the editor and curator** | 模型充当编辑与管理员 | *The model acts as the editor and curator of the knowledge base.* |
| 9 | **the moment X lands** | X 一入库的那一刻 | *The moment a document lands, it gets a deep read.* |
| 10 | **front-load the computation** | 把计算前置 | *Front-load the computation to the ingest stage.* |
| 11 | **compile the knowledge ahead of time** | 提前编译知识 | *The knowledge is compiled ahead of time and kept.* |
| 12 | **chunks and vector fragments** | 文本块与向量碎片 | *RAG stores chunks and vector fragments.* |
| 13 | **bidirectional links** | 双向链接 | *Wiki pages connected by bidirectional links.* |
| 14 | **an immutable source of fact** | 不可篡改的事实源 | *The raw layer is kept as an immutable source of fact.* |
| 15 | **trace back to the source** | 回溯到来源 | *Everything can be traced back to the source.* |
| 16 | **flag the conflict** | 标注冲突 | *It flags where the information conflicts.* |
| 17 | **rather than overwriting** | 而不是覆盖 | *It annotates the disagreement rather than overwriting it.* |
| 18 | **data from different channels disagree** | 不同渠道数据打架 | *Data from different channels often disagrees.* |
| 19 | **a scholar online around the clock** | 全天候在线的学者 | *It is a scholar online around the clock, reading everything you own.* |
| 20 | **the more you feed it, the thicker it gets** | 越喂越厚实 | *The more material you feed in, the thicker it becomes.* |

---

# 金句（只给英文，自己啃）
# Quotes, No Chinese — Chew on Them

> "Fragments are fetched on the fly, only when a question arrives — so the knowledge never truly settles."

> "The model passively pulls up slices; it never actively digests the material or connects the new with the old."

> "It no longer waits for the user to ask a question before digging through the raw documents."

> "The moment a document lands in the repository, it gets a deep read."

> "Front-load the computation to the ingest stage — the knowledge is compiled ahead of time and kept."

> "When a real conflict shows up, it does not overwrite. It writes the disagreement into the entry, with the sources labelled."

> "A knowledge base is not a collection of truths — it is a collection of claims with provenance."

> "Traditional RAG is you, at question time, rushing to a search engine and digging through a filing cabinet."

> "LLM Wiki is a scholar online around the clock, compiling an encyclopedia that cross-checks itself and keeps growing."

> "The more material you feed in, the more dimensional — and the thicker — this knowledge base becomes."

---

## 落地清单（中文速览 / 无英文）

1. **记住一个变量**：计算发生在提问时还是入库时——这一个变量能现场推出全部四维差异，不用背表
2. **问自己一遍**：我的知识系统里，同样的东西是不是反复出现？是 → 它值得一张实体页
3. **手写一张词条页**（现象 / 根因 / 解法 / 关联哪些记录），15 分钟，先手工再谈自动化
4. **冲突要标注、不要覆盖**：把"来源 A 说 vs 来源 B 说"写进词条，反例不许被悄悄丢掉
5. **三层分离**：原始层只读不动，维基层随便重建，规则层人定——真理和转述必须分开
6. **再刷到"X 已死"类视频**，先跑三个资格检查：语料变得快不快、延迟硬不硬、规模到天花板没有——三个坑（幻觉烘焙 / 入库成本 / 规模上限）在 `2026-11-12-LLMWiki取代RAG-深度总结.md` 第六节
7. **跟读 Part A**：`at query time / at ingest time` 这组对立是本期最值得练进嘴里的句式

# 抖音 AI Agent 收藏夹 — 每日学习摘要

## 用途
每周一至周五上午 10:00 自动跑：读取飞书"抖音收藏"群当天分享的抖音视频链接，**自动下载 + Whisper 转写 + 生成摘要报告 + 学习待办**。

## 方案一（已跑通）：游客 cookie 自动下载

抖音下载的难点不在登录，而在"新鲜 cookie"（yt-dlp 报错原文 `Fresh cookies (not necessarily logged in) are needed`）。用无头浏览器访问一次 douyin.com 就能拿到含 `__ac_signature`、`ttwid` 的游客 cookie，无需登录。

✅ 已实测：游客 cookie 成功下载 237MB 720p 视频（走视频流地址，**绕开 App 里"禁止下载"的限制**）。

### ⚠️ 坑位：必须访问「目标视频页」刷 cookie（2026-09-18 固化）

只访问 `douyin.com` 首页刷 cookie 时，部分视频仍会报 `Fresh cookies needed`（2026-09-11 与 2026-09-18 均踩过）。

**解法**：让 Playwright **直接 goto 目标视频页**再取 cookie，可稳定拿到 36-38 条（首页只有 23-30 条）。

已在 `download_transcribe.py` 中固化：`refresh_cookies(target_url)` 会先访问目标视频页，cookie 数 ≥20 就不再回退首页；单链接时自动传入该链接，多链接走首页。

### ⚠️ 坑位：多条视频不要并行下载（2026-09-21 修复）

`download()` 原先靠"取 `downloads/` 里 mtime 最大的文件"来定位下载结果。两条视频并行跑时会互相串台 —— **A 转写完其实是 B，而且常常是被截断的 B**（文件还在写入就被读）。2026-09-21 首次踩到：串行重跑后才发现第一份"逐字稿"是第二份视频的前 3 分钟。

已修复：`download()` 改用 yt-dlp `--print after_move:filepath` 返回的真实输出路径（失败才回退排序近似）。

**规则：一次只有一条视频在跑。脚本已经安全了，但 yt-dlp + cookie 刷新仍共享同一份 `douyin_cookies.txt`，串行更省事。**

### ⚠️ 坑位：`make_shadowing.py --no-words` 会覆盖成品（2026-09-21 修复）

`--no-words` 本意是"只想确认一下 Part 分布"，但旧实现把词库置空，`--no-words` 跑一次就把已经做好的 400KB 词典版 HTML 冲成一个 85KB 的空壳。

已修复：先无条件从 `wordcache/` 装载本地缓存，再决定要不要联网补缺漏。**现在 `--no-words` 只影响联网行为，不影响已有词典。**

## 你要做的（只 1 步）

手机抖音里点"分享" → 飞书 → "抖音收藏"群 → 发送链接。

（可选）看完视频后复制文案或口述 2-3 句要点粘到群里，作为转写失败时的兜底素材。

## 自动化要做的事

每个工作日 10:00 自动触发：
1. `lark-cli im +chat-messages-list` 读群**「上次跑完后 → 本次」窗口**内所有消息（默认滑动 24h），提取抖音短链
   - **不再以"自然日 00:00 - 23:59"为界**——避免上午 10 点跑时漏掉前晚 23:5x 新增的视频
   - 首次运行时窗口为 24h；后续每天向前滑动
   - ⚠️ **必须做一次「不限时间」全量拉取复核**（2026-09-18 教训）：用户在运行时刻**之后**分享的视频，任何时窗查询都查不到。全量拉取（群消息总量很小，`has_more=false`）是唯一可靠的兜底 —— 9-17 曾据此误判"无新增"，实际当天下午和晚上各分享了 1 条，直到 9-18 全量复核才发现并补处理
   - 注：`--start/--end` 必须用 **ISO 8601 带 T 分隔符**（`2026-09-18T00:00:00+08:00`），空格分隔会报 validation 错误
   - ⚠️ **lark-cli 是 shell 脚本，必须用 `bash` 调用，不能用 `node`**（2026-09-23 踩坑）。用 `node <path>` 会在第 2 行 `basedir=$(dirname ...)` 处报 `SyntaxError: missing ) after argument list`。正确写法：
     ```bash
     bash "C:/Users/tianyi.bu/.workbuddy/binaries/node/cli-connector-packages/lark-cli" \
       im +chat-messages-list --chat-id "oc_55515e4b712c572e6f69c8c839e2aadc" \
       --start "2026-09-23T00:00:00+08:00" --end "2026-09-23T23:59:59+08:00" --page-size 50
     ```
   - ⚠️ **重定向时不要把 stderr 并进 stdout**：lark-cli 会往 **stderr** 打 `warning: reactions_partial_failed: 1 message(s) failed (...)`（`om_x100b651565a584a0b1b47efed0039bd` 这条已删除消息恒定触发）。用 `2>&1` 会把 warning 顶在 JSON 前面，导致 `json.load` 报 `Expecting value: line 1 column 1`。要 `2>/dev/null`。
   - ⚠️ 跨 bash / Windows Python 传文件时**不要用 `/tmp`**：Git Bash 的 `/tmp` 对 Windows 版 Python 不可见（FileNotFoundError）。写到仓库内的临时文件再读。
2. `download_transcribe.py` 逐条处理：
   - Playwright 无头刷新游客 cookie（无需登录）
   - yt-dlp 下载视频
   - faster-whisper 转写中文逐字稿 → `transcripts/YYYY-MM-DD/<id>.txt`
3. 读逐字稿生成摘要；转写失败的用口述/文案兜底
4. 生成 `notes/YYYY-MM-DD.md`（报告）+ `notes/YYYY-MM-DD-todos.md`（待办）
5. （满足条件时）生成**两篇**深度总结：`notes/YYYY-MM-DD-<主题>-深度总结.md`（纯中文知识向）+ `notes/YYYY-MM-DD-<主题>-中英对照.md`（英语学习向）
6. 生成跟读页 `shadowing/YYYY-MM-DD-<主题>-跟读.html`
7. present_files 展示

### 深度总结的判定规则（2026-09-18 确立）

**不要按视频长度决定，要按"主报告是否已经装不下"决定。** 满足以下任意一条即单开深度总结：

| 判据 | 说明 |
| --- | --- |
| **信息被主报告压缩掉了** | 主报告篇幅有限，若某条视频有 ≥3 个成体系的论点/用例在主报告里只能一句话带过，就单开 |
| **存在可迁移的方法层** | 视频除了事实，还给出了一个可以复用的分析框架或思维工具 —— 这类内容值得单独沉淀 |
| **视角独特、无法与其他视频合并** | 即使信息量小，若它是当日唯一的某个视角（如人文/哲学视角），也单开 |

> 反面判据：**长度本身不是理由**。短视频如果有方法层，也要单开；长视频如果主报告已完整覆盖，也不必硬凑。
> 2026-09-18 的两条视频（37:51 的吴恩达访谈 + 4:02 的 AI 命名）**两条都单开了深度总结**，就是因为后者虽短，但"命名 = 团队的自我期许"是一个可复用的分析工具。

### ⚠️ 深度总结一律出「两篇 + 一个跟读页」（2026-09-19 用户要求，强制）

用户要在学知识的同时学英语。**每个单开深度总结的视频都要产出两份 md + 一个 HTML：**

| 文件 | 定位 | 要求 |
| --- | --- | --- |
| `notes/YYYY-MM-DD-<主题>-深度总结.md` | **知识向**（原风格） | 纯中文，逻辑清晰、论证完整的单篇文章。面向"我要搞懂这件事" |
| `notes/YYYY-MM-DD-<主题>-中英对照.md` | **英语向** | 中英对照，面向"我要顺便练英语" |
| `shadowing/YYYY-MM-DD-<主题>-跟读.html` | **口语向** | 单文件自带数据，任何电脑双击可开 |

两份 md 内容可以有侧重差异，不是简单互译：知识向那份要讲透论证链；对照版那份要照顾可读性和语言学习节奏。

#### 中英对照版的写法：渐隐支架（fading scaffold）

不要全程逐句对照 —— 中英视觉权重相同时，眼睛会直接读中文、跳过英文；逐句孤立对照又学不到篇章连接。按"扶梯由密到疏"分三段：

| 段 | 对照粒度 | 规则 |
| --- | --- | --- |
| **Part A** 核心论点 | 逐句（EN 一行 → 中一行） | 英文用 `**EN**` 开头放正文；中文放进 `>` 引用块，视觉弱化，逼读者先读英文 |
| **Part B** 论证展开 | 整段（英文整段 → 中文整段） | 撑完一整段英文才给中文，训练篇章级阅读 |
| **Part C** 延伸/用例 | 纯英文 + 少量术语注 | 中文用 `<details>` 折叠，只用于**读完英文后自查**，不用于对读 |

固定附件（每篇都要有）：

- **词块表 Chunk Table**（15-20 条）：可复用搭配（如 `lean in` / `the bottleneck has shifted` / `be in a position to do sth`），**不是单词表**
- **原声金句**（视频为英文时）：英文原句，**不配中文**
- 文末**中文速览落地清单**（无英文）

#### 跟读页 = 跟读 + 点词词典

```bash
python make_shadowing.py "notes/YYYY-MM-DD-<主题>-中英对照.md"      # 抓词 + 出页（默认）
python make_shadowing.py "notes/xxx.md" --no-words                   # 跳过词库，快速重排
python make_shadowing.py "notes/xxx.md" --refresh                    # 忽略缓存重抓
```

点一句 → 朗读，**右侧面板同时把这句话拆成单词 chip**；再点任意单词弹出释义卡：

- 音标 / 词性 + 中文释义 / 词形变化（单复数、时态）/ **双语真实例句**（例句可点朗读）
- ⭐ 可收藏进**生词本**，存 localStorage，关掉浏览器再开还在
- 带绿色底线的 chip = 词库已收录；未收录的（多为专有名词）会明确提示

词库在**生成 HTML 时**一次性从有道词典 `dict.youdao.com/jsonapi` 抓取，缓存到 `wordcache/`，之后**永久离线可用**（单篇约 660 词条，句内词覆盖率约 98%）。网络不通也不会失败，会沿用已有缓存。

**同样必须保证"任何电脑都能出声"**，语音三档自动降级：

1. **本地语音** —— 浏览器 speechSynthesis，离线、零延迟（有英文语音包时首选）
2. **在线·有道** —— `dict.youdao.com/dictvoice`，国内可达，不需要本机语音包
3. **在线·Google** —— `translate.google.com/translate_tts`，境外兜底

打开时若检测不到英文语音包，自动切换在线档并在页面上提示。`shadowing/` 的 HTML **要上传 GitHub**，方便换机器下载后直接打开。

> 参考样板：`notes/2026-09-18-对话吴恩达-AI恐惧与机会-深度总结.md`（知识向）+ `-中英对照.md`（英语向）+ `shadowing/…-跟读.html`
> 注：`2026-09-18-AI命名里的小巧思-深度总结.md` 目前只有知识向一篇，尚未补对照版。

## 环境配置（已就绪）

| 组件 | 路径 | 状态 |
|------|------|------|
| Python venv | `C:\Users\tianyi.bu\.workbuddy\binaries\python\envs\douyin\Scripts\python.exe` | ✅ |
| faster-whisper | venv 内（含 PyAV，无需 ffmpeg） | ✅ |
| yt-dlp | venv 内 `Scripts\yt-dlp.exe` | ✅ |
| playwright + chromium | venv 内，chromium 在 `%LOCALAPPDATA%\ms-playwright` | ✅ |

Whisper 模型（首次转写自动下载到 `C:\Users\tianyi.bu\.cache\huggingface\`）：

| 模型 | 大小 | 中文质量 | 转写速度（CPU） |
|------|------|---------|----------------|
| `tiny` | ~75MB | 一般 | 最快 |
| `small`（默认） | ~500MB | 较好 | 约 0.3-0.5 倍实时 |
| `medium` | ~1.5GB | 好 | 约 0.1-0.2 倍实时 |

## 时长与速度（重要）

- 短视频（1-10 分钟干货）：small 模型转写几分钟内完成，体验好
- 长视频（30 分钟以上访谈）：CPU 转写可能 30-60 分钟，自动化会降级用 tiny 或只靠口述
- 建议优先收藏短视频干货；长视频如确需深学，可接受较长转写时间

## 核心脚本

| 脚本 | 用途 |
|------|------|
| `download_transcribe.py` | 整合脚本：刷 cookie → 下载 → 转写（自动化主调用） |
| `make_shadowing.py` | 从中英对照版抽英文 → 跟读 HTML（含点词词典、生词本、三档语音） |
| `shadow_tpl.py` | 上面的 HTML 模板（拆出来避免单文件过长） |
| `get_douyin_cookies.py` | 单独刷游客 cookie |
| `transcribe_video.py` | 单独转写本地视频文件 |

## 手动跑一次

```bash
# 处理单条链接（自动下载+转写）
C:\Users\tianyi.bu\.workbuddy\binaries\python\envs\douyin\Scripts\python.exe \
  D:\workbuddy\dy要点整理\.workbuddy\douyin_collection\download_transcribe.py \
  "https://v.douyin.com/xxxx/" --model small

# 从飞书群读当天链接批量处理
...download_transcribe.py --date 2026-09-10
```

## 目录结构

| 目录 | 用途 |
|------|------|
| `notes/` | **知识产出**，按日期命名：报告 `YYYY-MM-DD.md`、待办 `YYYY-MM-DD-todos.md`、深度总结 `…-深度总结.md`（知识向）+ `…-中英对照.md`（英语向） |
| `shadowing/` | 跟读 HTML（单文件自带数据，任何电脑可开），随仓库上传 |
| `transcripts/` | 逐字稿 txt/json，按日期分组（不上传） |
| `wordcache/` | 词典抓取缓存（构建时用，不上传；删了会自动重抓） |
| `downloads/` | 视频临时目录（转写后自动删，7 天清理） |
| `douyin_cookies.txt` | 自动刷新的游客 cookie（不上传） |

## GitHub 同步

每天整理完 `notes/` 下的 md 文件后，自动 git commit + push 到：

🔗 **https://github.com/IYNAIT0110/douyin-knowledge**

- 认证：gh CLI 已登录（账号 `IYNAIT0110`），git credential 已指向 gh
- 上传内容：`notes/`、`shadowing/` 的 HTML、`README.md`、`.gitignore`
- 不上传：视频、cookie、逐字稿、脚本（`*.py`）、mp3（见 `.gitignore`）
- ⚠️ 该仓库为**公开**，任何人可访问。若想保密，去 GitHub 把仓库改为 Private

## 已知限制

- 抖音风控可能升级，游客 cookie 若失效，`get_douyin_cookies.py` 会自动重刷，通常能恢复；同一条链接连刷失败 2 次以上就直接重试一次整个流程（2026-09-21：第一次 24 条 cookie 失败，重跑拿到 37 条即成功）
- 转写走 CPU，长视频慢；有 NVIDIA 显卡可改 `device=cuda` 提速 5-10 倍
- 极少数视频（需登录才能观看的私密内容）游客态可能下不到，此时靠口述兜底

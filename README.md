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
2. `download_transcribe.py` 逐条处理：
   - Playwright 无头刷新游客 cookie（无需登录）
   - yt-dlp 下载视频
   - faster-whisper 转写中文逐字稿 → `transcripts/YYYY-MM-DD/<id>.txt`
3. 读逐字稿生成摘要；转写失败的用口述/文案兜底
4. 生成 `notes/YYYY-MM-DD.md`（报告）+ `notes/YYYY-MM-DD-todos.md`（待办）
5. （可选）生成 `notes/YYYY-MM-DD-<主题>.md` 深度总结
6. present_files 展示

### 深度总结的判定规则（2026-09-18 确立）

**不要按视频长度决定，要按"主报告是否已经装不下"决定。** 满足以下任意一条即单开深度总结：

| 判据 | 说明 |
| --- | --- |
| **信息被主报告压缩掉了** | 主报告篇幅有限，若某条视频有 ≥3 个成体系的论点/用例在主报告里只能一句话带过，就单开 |
| **存在可迁移的方法层** | 视频除了事实，还给出了一个可以复用的分析框架或思维工具 —— 这类内容值得单独沉淀 |
| **视角独特、无法与其他视频合并** | 即使信息量小，若它是当日唯一的某个视角（如人文/哲学视角），也单开 |

> 反面判据：**长度本身不是理由**。短视频如果有方法层，也要单开；长视频如果主报告已完整覆盖，也不必硬凑。
> 2026-09-18 的两条视频（37:51 的吴恩达访谈 + 4:02 的 AI 命名）**两条都单开了深度总结**，就是因为后者虽短，但"命名 = 团队的自我期许"是一个可复用的分析工具。

### ⚠️ 深度总结必须是「中英对照版」（2026-09-19 用户要求，强制）

用户在学知识的同时要学英语，所以**深度总结文件一律写成中英对照，不再单独出纯中文版**（主报告 `YYYY-MM-DD.md` 保持纯中文，用于速览）。

对照方式采用**渐隐支架（fading scaffold）**，而不是全程逐句对照。原因是：中英视觉权重相同时，眼睛一定会先读中文、直接跳过英文；逐句对照又容易把文章拆成孤立的句子，学不到篇章连接。所以按"扶梯由密到疏"分三段：

| 段 | 对照粒度 | 规则 |
| --- | --- | --- |
| **Part A** 核心论点 | 逐句（EN 一行 → 中一行） | 英文用 `**EN**` 开头放在正文；中文放进 `>` 引用块，视觉弱化，逼读者先读英文 |
| **Part B** 论证展开 | 整段（英文整段 → 中文整段） | 撑完一整段英文才给中文，训练篇章级阅读 |
| **Part C** 延伸/用例 | 纯英文 + 少量术语注 | 中文用 `<details>` 折叠，只用于**读完英文后自查**，不用于对读 |

固定附件（每篇都要有）：

- **词块表 Chunk Table**（15-20 条）：给可复用搭配（如 `lean in` / `the bottleneck has shifted` / `be in a position to do sth`），**不是单词表** —— 词块能直接搬进口语和写作
- **原声金句**（视频为英文时）：英文原句，**不配中文**，留出自己啃的空间
- 文末保留一份**中文速览落地清单**（无英文），方便快速执行

**每篇还必须生成跟读页**（`make_shadowing.py`）：

```bash
python make_shadowing.py "notes/YYYY-MM-DD-<主题>-深度总结.md" --part all
# → shadowing/<同名>-Partall.html
```

浏览器本地语音朗读，离线可用，中文默认折叠；点句即读，可调语速与句间停顿。用 present_files 展示给用户。
（`--format mp3` 走 edge-tts 在线合成，本机网络连不上 `speech.platform.bing.com`，默认不用。）

> 参考样板：`notes/2026-09-18-对话吴恩达-AI恐惧与机会-深度总结.md`
> 注：2026-09-18 的 `AI命名里的小巧思` 那条仍是纯中文版，属历史文件，不作为后续样板。

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
| `make_shadowing.py` | 从中英对照版深度总结抽英文，生成跟读 HTML（默认）/ mp3 |
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
| `notes/` | **所有产出**，按日期命名：报告 `YYYY-MM-DD.md`、待办 `YYYY-MM-DD-todos.md`、深度总结 `YYYY-MM-DD-<主题>.md` |
| `transcripts/` | 逐字稿 txt/json，按日期分组（不上传） |
| `downloads/` | 视频临时目录（转写后自动删，7 天清理） |
| `douyin_cookies.txt` | 自动刷新的游客 cookie（不上传） |

## GitHub 同步

每天整理完 `notes/` 下的 md 文件后，自动 git commit + push 到：

🔗 **https://github.com/IYNAIT0110/douyin-knowledge**

- 认证：gh CLI 已登录（账号 `IYNAIT0110`），git credential 已指向 gh
- 上传内容：仅 `notes/`、`README.md`、`.gitignore`
- 不上传：视频、cookie、逐字稿、脚本（见 `.gitignore`）
- ⚠️ 该仓库为**公开**，任何人可访问。若想保密，去 GitHub 把仓库改为 Private

## 已知限制

- 抖音风控可能升级，游客 cookie 若失效，`get_douyin_cookies.py` 会自动重刷，通常能恢复
- 转写走 CPU，长视频慢；有 NVIDIA 显卡可改 `device=cuda` 提速 5-10 倍
- 极少数视频（需登录才能观看的私密内容）游客态可能下不到，此时靠口述兜底

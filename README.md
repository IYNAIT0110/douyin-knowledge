# 抖音 AI Agent 收藏夹 — 每日学习摘要

## 用途
每周一至周五上午 10:00 自动跑：读取飞书"抖音收藏"群当天分享的抖音视频链接，**自动下载 + Whisper 转写 + 生成摘要报告 + 学习待办**。

## 方案一（已跑通）：游客 cookie 自动下载

抖音下载的难点不在登录，而在"新鲜 cookie"（yt-dlp 报错原文 `Fresh cookies (not necessarily logged in) are needed`）。用无头浏览器访问一次 douyin.com 就能拿到含 `__ac_signature`、`ttwid` 的游客 cookie，无需登录。

✅ 已实测：游客 cookie 成功下载 237MB 720p 视频（走视频流地址，**绕开 App 里"禁止下载"的限制**）。

## 你要做的（只 1 步）

手机抖音里点"分享" → 飞书 → "抖音收藏"群 → 发送链接。

（可选）看完视频后复制文案或口述 2-3 句要点粘到群里，作为转写失败时的兜底素材。

## 自动化要做的事

每个工作日 10:00 自动触发：
1. `lark-cli im +chat-messages-list` 读群**「上次跑完后 → 本次」窗口**内所有消息（默认滑动 24h），提取抖音短链
   - **不再以"自然日 00:00 - 23:59"为界**——避免上午 10 点跑时漏掉前晚 23:5x 新增的视频
   - 首次运行时窗口为 24h；后续每天向前滑动
2. `download_transcribe.py` 逐条处理：
   - Playwright 无头刷新游客 cookie（无需登录）
   - yt-dlp 下载视频
   - faster-whisper 转写中文逐字稿 → `transcripts/YYYY-MM-DD/<id>.txt`
3. 读逐字稿生成摘要；转写失败的用口述/文案兜底
4. 生成 `notes/YYYY-MM-DD.md`（报告）+ `notes/YYYY-MM-DD-todos.md`（待办）
5. present_files 展示

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

每天整理完 `reports/` 和 `todos/` 的 md 文件后，自动 git commit + push 到：

🔗 **https://github.com/IYNAIT0110/douyin-knowledge**

- 认证：gh CLI 已登录（账号 `IYNAIT0110`），git credential 已指向 gh
- 上传内容：仅 `notes/`、`README.md`、`.gitignore`
- 不上传：视频、cookie、逐字稿、脚本（见 `.gitignore`）
- ⚠️ 该仓库为**公开**，任何人可访问。若想保密，去 GitHub 把仓库改为 Private

## 已知限制

- 抖音风控可能升级，游客 cookie 若失效，`get_douyin_cookies.py` 会自动重刷，通常能恢复
- 转写走 CPU，长视频慢；有 NVIDIA 显卡可改 `device=cuda` 提速 5-10 倍
- 极少数视频（需登录才能观看的私密内容）游客态可能下不到，此时靠口述兜底

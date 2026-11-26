# 预渲染视频 RU 中文字幕重新打轴（2026-09-30）

> [!NOTE]
> 这是 2026-09-30 FMV 字幕重新打轴的发布记录，成果已合并至仓库 `video_subtitles/` 与最新版汉化包。完整安装与构建方式请见根目录 [README.md](../../README.md) 与 [BUILDING.md](../../BUILDING.md)。

## 概述

本次修订处理了 `video/` 目录下的全部 **15 个** `*_RU.srt`，共 **222 条**字幕：

- 仅替换 ASCII 时间码；所有 222 条字幕正文的原始字节、载体字库编码、标点、顺序、条目数与单条长度均保持逐字节一致。
- 视频、音轨、其他语言字幕、DLL 与 POD 均未改动，构建过程未启动游戏。
- 彻底消除了原版字幕中的 **78 处相邻重叠**与 **7 条尾部超时越界**（现为 0 重叠、0 越界）。

## 打轴方法

从当前 BIK 视频的第一路英语音轨提取 16 kHz PCM 音频，使用 `faster-whisper` (`medium`) 提取词级时间戳；对配乐较重或低声片段进行无 VAD 二次分析，并对 4 处疑难窗口使用 `large-v3` 交叉复核。随后人工按既有中文译文对齐英语对白，收紧首尾静音并保留短暂可读余量，同时将前一条字幕截断在后一条出现之前。ASR 仅用于辅助对齐时间轴，不改动译文。完整音轨、分词数据及人工校对区间均保存在本地发布目录。

## 特殊条目说明

- **`A1S01P06:1`**：“硫磺会总部，法国，1939年”为画面标题，按镜头出现时机保留，尾部避开首句对白。
- **`A2S02P16:1`**：“莱恩，刚才那是埃菲梅拉，对吧？”——英语音轨经 `medium` 与 `large-v3` 两次独立识别，均仅检出“Rayne? Where'd you end up?”，未听到完整的埃菲梅拉问句。为保留既有译文，该条安排在对白前的镜头空档至称呼结束，并在 `timing_review.tsv` 中标注为 `reference_text_no_full_audio_match`（非逐句英语音频匹配）。
- **`A2S02P16:2`**：按同一英语句中“Where'd you end up?”的词边界对齐显示。

## 文件清单与使用

- `payload/video`：供游戏直接使用的 15 个载体编码 RU 字幕文件。
- `source/*_ZH.utf8.srt`：仅供中文阅读复核，请勿直接覆盖游戏内的 RU 文件。
- `timing_review.tsv`：记录每条字幕的原时间、新时间、位移量、正文与证据类别。
- `baseline.json`：记录全部视频目录文件及关键游戏文件的原始 SHA-256。
- `manifest.json`：记录 15 个字幕的新旧 SHA-256、时长与例外条目。
- `install.ps1`：预检全部原文件哈希并备份后安装，失败时自动恢复。
- `ROLLBACK.sh`（Git Bash）或 `rollback.ps1`：恢复安装前的 15 个字幕。
- `MODIFIED_FILE.zip`：包含载体字幕、清单及安装/回滚脚本的独立安装包（游戏内观感需人工实机测试）。

## 本地复现

```powershell
python transcribe.py
python review_windows.py
python review_critical.py
python build.py
python verify.py modified
```

若直接使用已保留的音频与 ASR 结果，仅需运行 `python build.py`。完整重跑依赖本机 Python 环境中的 `PyAV`、`faster-whisper`、`imageio-ffmpeg` 及已缓存模型。

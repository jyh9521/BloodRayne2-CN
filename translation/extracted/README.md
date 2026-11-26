# BloodRayne 2: Terminal Cut 文本提取说明

本目录由 `tools/export_translation_catalog.py` 直接从 Steam 纯净基线生成。编辑时请勿改动 `id`、`kind`、`file`、`line`、`key`、`source_en` 或 `source_ru`；仅填写 `translation_zh` 并将 `status` 设为 `translated`。

所有 TSV 文件均采用 **UTF-8 BOM** 编码、制表符分隔与 **CRLF** 行尾，可直接使用支持 UTF-8 的表格软件打开编辑。

## 目录文件

| 文件 | 条目数 | 内容说明 |
| --- | ---: | --- |
| `dialogue.tsv` | 2,598 | 剧情对白、字幕、任务目标与游戏提示 |
| `menu.tsv` | 740 | 737 条语言表菜单文本 + 3 条新版 EXE 内置菜单文本 |
| `moves.tsv` | 94 | 招式表文本 |
| `credits.tsv` | 283 | 当前 Terminal Cut 英文演职人员表可见文本 |

上述数据合并后的总表位于上级目录 `catalog.tsv`，共计 **3,715 条**。

> [!NOTE]
> 截至 2026-07-31：
> - `menu.tsv`（739 条有效译文 + 1 条原始空白键值）与 `moves.tsv`（94 条，已校验 `@@...@@` 控制标记和格式占位符）均已翻译完成。
> - 演职人员表按计划不翻译，构建时继续使用英文原件。
> - 剧情对白已拆分为 [../dialogue_deduplicated/dialogue_unique.tsv](../dialogue_deduplicated/dialogue_unique.tsv)（1,706 条唯一翻译单元）与位置映射表（2,598 条），后续对白翻译请直接编辑去重表，构建器 `tools/build_full_translation.py` 会自动回填全部实装位置。

## 原文来源与特殊情况

- **剧情英文与俄文**：纯净 `_cn_project/baseline/LANGUAGE.POD`。
- **菜单英文键与俄文值**：同一纯净 `LANGUAGE.POD`。
- **招式英文原文**：纯净 `COMMON.POD` 内的 `DATA\MOVELIST.TXT`。
- **演职人员英文原文**：纯净 `COMMON.POD` 内的 `DATA\CREDITS.TXT`。

提取时发现的特殊条目：

- **对白对齐**：2,598 条剧情文本中有 2,590 条自动匹配到英文原文；其余 8 条在 `notes` 中标记为仅俄语版存在，翻译时以俄语语义为准；另有 3 条英文显示文本本身为空（状态 `blank`），导入时清空俄语残留。
- **招式表结构差异**：第 72、74、98、100 行的俄文版本将重复按键缩写为“3 次”，导致控制标记数量与英文不一致。这 4 行已在 `notes` 中标出，翻译时以英文结构和完整的 `@@...@@` 标记序列为准。

## 不可改动的内容

翻译与校对时必须原样保留以下内容：

- `@@...@@` 控制标记
- `%d`、`%s` 等格式占位符
- 对白键、WAV 文件名与脚本资源名
- 演职人员表行首的 `/b` 格式指令

> [!NOTE]
> 本文件为 **2026-09-30 仓库重构为公开可重建项目之前的根目录 `README.md` 历史快照**，保留用于追溯早期打包与发布方式（当时采用 `INSTALL.cmd` 与本地 `dist/` 目录）。当前最新文档请查看根目录 [README.md](../../README.md) 与 [BUILDING.md](../../BUILDING.md)。

# BloodRayne 2: Terminal Cut 简体中文补丁

本仓库保存汉化测试版、翻译表、构建工具与历次修订记录。补丁使用俄语文本槽显示中文，配音保持英语，游戏由玩家自行启动与测试。

## 安装 Release v1.0.0（2026-09-30）

1. 将 `dist/BloodRayne2_CN_v1.0.0_20260930.zip` 完整解压到独立目录，双击 `INSTALL.cmd` 并输入 Steam 游戏根目录。
2. 安装器会校验游戏版本、自动备份被覆盖文件并记录新增文件，然后安装 17 个汉化文件。
3. 从 Steam 启动游戏，将文本语言设为 **Russian** 并开启字幕（配音仍为英语）。

- 仅安装 `LANGUAGE.POD`、`dinput8.dll` 和 15 个 `video/*_RU.srt`，不替换 `W32ART.POD`、`W32ENSND.POD`、`COMMON.POD`、视频或存档，无需源工程、Python 或系统字体。
- 若游戏目录已存在其他 `dinput8.dll`，安装器会自动中止而不覆盖。
- 卸载时运行 `UNINSTALL.cmd`，即可从 `_cn_backup_release_20260930` 还原原文件。
- 压缩包内 `README.zh-CN.txt` 包含干净安装、回滚、测试检查点与已知待复核项。`dist/BloodRayne2_CN_current.zip` 为同版本别名（请勿沿用更早期的直接覆盖安装步骤）。

## 源文件与版本记录

- `translation/`：翻译表、术语表与视频中文源文本。
- `video_subtitles/`：适配游戏字体载体的视频中文字幕文件。
- `proxy/`：`dinput8.dll` 代理源代码。
- `tools/`：POD 分析、导出、构建与校验工具。
- `releases/`：历次修订脚本、差异与验证记录（不包含庞大的游戏原始归档与临时副本）。
- `dist/`：当时打包好的可安装测试版压缩包。

缺失字幕修订的范围与未确认音频清单见 `releases/missing-dialogue-20260926/VERIFICATION.txt`（离线脚本统计不等同于全流程实机通关验证）。

在工程根目录运行 `python build_release_zip.py` 可从本机游戏目录重新打包 `dist/BloodRayne2_CN_current.zip` 及校验清单（只读游戏文件，不启动游戏）。

## 历史本地推送流程

当时工作流为先在本地提交，再手动推送至 `origin main`：

```powershell
git -C 'C:\Program Files (x86)\Steam\steamapps\common\BloodRayne 2 Terminal Cut\_cn_project' push origin main
```

每次改动后需重新构建并校验安装包，再一并提交源文件、差异记录与对应 `dist/` 产物，避免仓库代码与实装包不同步。

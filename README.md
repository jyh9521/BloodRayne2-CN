# 《吸血莱恩 2：终极剪辑版》简体中文汉化

**BloodRayne 2: Terminal Cut** 的简体中文汉化补丁与源码重建工具，适用于 **Steam 终极剪辑版**。汉化借用俄语（Russian）文本槽显示简体中文，游戏配音完整保留原版英语。

## 当前状态

目前已发布 **[v1.0](https://github.com/jyh9521/BloodRayne2-CN/releases/tag/v1.0)**，包含以下内容：

- 完整的界面菜单、招式说明、游戏提示与实时剧情字幕
- 15 部预渲染过场动画（FMV）的重新打轴中文字幕
- 为部分原版无字幕的脚本对白和战斗台词补全中文显示

> [!NOTE]
> 本汉化仍处于测试阶段，尚未对全部关卡和触发场景完成逐一实机通关验收。如果你在游玩中遇到漏字幕、排版异常或文本问题，欢迎提交 Issue 反馈。

## 下载与安装

普通玩家只需下载发布包即可，无需安装器、Python、系统字体或编译工具。

1. 前往 [v1.0 发布页](https://github.com/jyh9521/BloodRayne2-CN/releases/tag/v1.0) 下载 `BloodRayne2_CN_v1.0.zip`。
2. 退出游戏，建议先备份游戏根目录中将被覆盖的同名文件。
3. 将压缩包内的全部文件解压到游戏根目录，合并文件夹并覆盖同名文件。
4. 启动游戏，在设置中将文本语言切换为 **Russian（俄语）**，并开启字幕。

> [!TIP]
> 在 Steam 库中右键点击游戏 → **管理 → 浏览本地文件**，即可直接打开游戏根目录。

解压完成后，游戏目录结构应包含以下汉化文件：

```text
BloodRayne 2 Terminal Cut/
├─ rayne2.exe
├─ dinput8.dll
├─ LANGUAGE.POD
└─ video/
   └─ *_RU.srt
```

## 模组兼容与卸载

- **资源兼容**：本补丁仅替换 `LANGUAGE.POD`、`dinput8.dll` 和 `video/` 下的 `*_RU.srt`，不会修改 `W32ART.POD`、`W32ENSND.POD`、视频文件或存档。仅修改后两者的材质、模型或音频模组可以直接共存。
- **DLL 冲突**：如果其他模组也自带 `dinput8.dll`，会占用同一代理入口，无法通过直接覆盖的方式叠加使用。
- **恢复与卸载**：退出游戏，还原安装前备份的原文件，并删除原本不存在的汉化文件（如 `dinput8.dll` 和新增的 `*_RU.srt`）即可。

## 从源码重建

本仓库保存了译文数据、代理 DLL 源码、汉化资源快照与重建脚本，支持在本地从零重新编译并打包补丁。

**环境要求**：Windows 10/11、Python 3.10+（仅使用标准库）、Visual Studio 2022 C++ Build Tools，以及哈希匹配的 Steam 原版 `LANGUAGE.POD`。

```powershell
git clone https://github.com/jyh9521/BloodRayne2-CN.git
Set-Location BloodRayne2-CN
python rebuild/build.py --base-pod "D:\Games\BloodRayne 2 Terminal Cut\LANGUAGE.POD"
```

构建完成后会在 `build/reproducible/BloodRayne2_CN_rebuilt.zip` 生成可解压覆盖的补丁包，整个过程不会修改你的游戏目录。详细的环境配置、原版文件哈希校验和资源验证说明请阅读 [BUILDING.md](BUILDING.md)。

> [!IMPORTANT]
> 重建脚本基于 `rebuild/assets/` 中的定稿资源快照打包，**不会自动导入 `translation/` 下任意 TSV 的修改**。如需修改译文，请同步更新对应资源、字库与清单后再执行重建；此外，不同版本的 MSVC 编译器生成的 DLL 不保证逐字节完全一致。

## 仓库结构与文档

| 路径 | 说明 |
| --- | --- |
| `translation/` | 可编辑译文、术语表与字幕源稿 |
| `rebuild/runtime/` | 中文显示与字幕补全代理 DLL 的 C++ 源码 |
| `rebuild/assets/` | 定稿汉化资源快照、字库与清单 |
| `video_subtitles/` | 15 个已完成时间轴校准的游戏视频字幕 |
| `tools/`、`releases/` | 文本提取工具、历史脚本与各阶段修订记录 |

延伸阅读：

- [BUILDING.md](BUILDING.md) — 从源码重建与修改译文指南
- [HANHUA_NOTES.md](HANHUA_NOTES.md) — 引擎逆向分析与早期开发记录

## 问题反馈与版权声明

如遇漏字幕、乱码或流程卡顿，欢迎在 [Issues](https://github.com/jyh9521/BloodRayne2-CN/issues) 中反馈，并尽量附上补丁版本、所在关卡、触发位置和对应台词（附带截图或日志会更有帮助）。

原版游戏及商业素材版权归原权利人所有。本仓库仅包含汉化工程文件与资源快照，不提供完整游戏，使用与重建均需自行准备正版 Steam 游戏文件。仓库内涉及的第三方组件保留其原有版权与许可，本说明不另行授予统一的开源许可证。

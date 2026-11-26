# 从源码重建汉化

本文档介绍如何在本地环境重新编译代理 DLL、重建汉化版 `LANGUAGE.POD` 并打包最终的覆盖版补丁。

## 构建方式概览

- **普通玩家**：直接从 [v1.0 发布页](https://github.com/jyh9521/BloodRayne2-CN/releases/tag/v1.0) 下载 `BloodRayne2_CN_v1.0.zip` 解压覆盖即可，无需编译器或源码。
- **开发者 / 源码重建**：运行下方的重建脚本会自动编译 x86 代理 DLL，结合 Steam 原版 `LANGUAGE.POD` 与仓库内的定稿汉化资源重新生成 POD，并打包 15 个已校准的视频字幕。整个过程完全独立，不读取本机已安装的汉化文件，也不依赖任何历史工作目录。

## 环境准备

1. **基础环境**：Windows 10/11、Git、Python 3.10 或更高版本（仅依赖标准库，无需 `pip install`）。
2. **编译工具**：Visual Studio 2022 Build Tools，需勾选“使用 C++ 的桌面开发”、MSVC x86/x64 生成工具和 Windows SDK。构建脚本会通过 `vswhere` 自动定位编译器。
3. **Steam 原版文件**：从 Steam 纯净安装的 *BloodRayne 2: Terminal Cut* 中获取未修改的 `LANGUAGE.POD`。其 SHA-256 必须严格匹配：
   ```text
   2e3e0797e147e58b7308d3fcedbdf0a06cb433fb099fc24b8fb38943d8b358e4
   ```
   > [!IMPORTANT]
   > 游戏原始资源由 Steam 提供，本仓库不包含原版归档。请勿将已打过汉化补丁的 `LANGUAGE.POD` 作为构建输入。

## 执行构建

```powershell
git clone https://github.com/jyh9521/BloodRayne2-CN.git
Set-Location BloodRayne2-CN
python rebuild/build.py --base-pod "C:\Program Files (x86)\Steam\steamapps\common\BloodRayne 2 Terminal Cut\LANGUAGE.POD"
```

- **默认输出**：`build/reproducible/BloodRayne2_CN_rebuilt.zip`。
- **自定义输出目录**：可使用 `--output "D:\BR2-build"` 指定其他输出路径。
- **原版 POD 路径**：`--base-pod` 接受任意位置存放的、哈希匹配的原版 `LANGUAGE.POD`。

构建过程不会修改 Steam 游戏目录，也不会启动游戏。生成的压缩包同样为解压覆盖版，直接解压至游戏根目录即可使用。

## 源码与资源说明

| 路径 | 说明 |
| --- | --- |
| `rebuild/runtime/` | 当前代理 DLL 的完整 C++ 源码、导出定义、字幕与布局头文件 |
| `rebuild/assets/localized_resources.zip`、`resources.json` | 定稿汉化资源快照（64 个替换项、7 个新增项，内含字形图集，无需系统字体）及最终 POD 全部 393 个条目的哈希清单 |
| `video_subtitles/` | 2026-09-30 重新打轴后的 15 个游戏载体编码 `*_RU.srt` |
| `releases/overlay-20260930/` | 覆盖版打包脚本与说明 |
| `translation/`、`releases/` | 可编辑译文、术语表及各阶段历史修订记录 |

> [!NOTE]
> 根目录下的历史脚本 `build_release_zip.py` 仅用于从本机已安装的游戏目录打包快照，并非源码重建入口；如需从源码重建，请始终使用 `rebuild/build.py`。

## 验证标准与产物差异

重建脚本会按内容逐项校验输出产物：

- **POD 内容**：全部 393 个资源条目须与 `resources.json` 清单逐条一致，且 CRC 校验通过。由于重建过程不会累积历史废弃数据，新 POD 的容器总哈希会与早期实装版不同。
- **代理 DLL**：须为 x86 架构且包含 5 个代理导出。受编译器小版本差异影响，不同环境下编译出的 DLL 不保证逐字节哈希一致。
- **视频字幕**：15 个 SRT 文件保持原始字节不变。

## 修改译文

`translation/` 目录保存了可编辑的译文表与术语表。需要注意的是，`rebuild/build.py` 直接读取 `rebuild/assets/` 中的定稿资源快照，**不会自动导入对 TSV 文件的修改**。

如果你需要修改译文或调整功能：

1. 参考 `tools/` 和 `releases/` 中的导入与处理工具，将新译文重新导入对应资源并更新字形图集。
2. 同步更新 `rebuild/assets/` 下的资源快照与 `resources.json` 清单（如涉及代码改动，同步更新 `rebuild/runtime/`）。
3. 重新运行 `rebuild/build.py` 验证并打包，切勿只提交 TSV 改动。

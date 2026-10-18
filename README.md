# 《吸血莱恩 2：终极剪辑版》简体中文汉化

BloodRayne 2: Terminal Cut 的简体中文汉化补丁与重建工具，适用于 **Steam 终极剪辑版**。中文借用俄语文本槽，配音保留英语。

## 当前状态

已发布 **v1.0**。覆盖菜单、招式说明、剧情字幕、游戏提示与预渲染动画字幕，并为部分原本没有字幕的对白和战斗台词补上中文。汉化仍在测试中，发布不等于所有显示与对白场景均已验收。

## 下载

[v1.0 发布页](https://github.com/jyh9521/BloodRayne2-CN/releases/tag/v1.0)，附件为 `BloodRayne2_CN_v1.0.zip`。

玩家使用发布包即可；本仓库保存译文、代理源码、汉化资源快照与重建工具。

## 安装 / 使用

1. 退出游戏，先备份将被覆盖的同名文件。
2. 将压缩包的全部内容解压到游戏根目录，合并并覆盖。
3. 启动游戏，将文本语言设为 **Russian（俄语）**，并开启字幕。

Steam → 游戏属性 / 管理 → 浏览本地文件，可找到安装目录。无需安装器、Python、字体或编译工具。

```text
BloodRayne 2 Terminal Cut/
├─ rayne2.exe
├─ dinput8.dll
├─ LANGUAGE.POD
└─ video/
   └─ *_RU.srt
```

## 兼容与恢复

补丁替换 `LANGUAGE.POD`、`dinput8.dll` 和 `video/` 下的 RU 字幕，不替换 `W32ART.POD`、`W32ENSND.POD`、视频文件或存档。材质、模型和声音模组若仅修改后两种 POD，不会直接覆盖汉化。

其他模组若也使用 `dinput8.dll`，会占用同一代理入口，不能直接覆盖叠加。恢复时还原安装前备份，并删除原先不存在的汉化文件。

## 从源码重建

需要 Windows 10/11、Python 3.10+、Visual Studio 2022 C++ Build Tools，以及指纹匹配的 Steam 原版 `LANGUAGE.POD`。Python 部分仅使用标准库。

```powershell
git clone https://github.com/jyh9521/BloodRayne2-CN.git
Set-Location BloodRayne2-CN
python rebuild/build.py --base-pod "D:\Games\BloodRayne 2 Terminal Cut\LANGUAGE.POD"
```

输出为 `build/reproducible/BloodRayne2_CN_rebuilt.zip`，构建不直接修改游戏目录。详细依赖、输入指纹和资源验证见 [BUILDING.md](BUILDING.md)。

重建入口使用 `rebuild/assets/` 中的定稿资源快照，**不会自动导入任意 TSV 改动**。修改译文后须同步更新对应资源、字库与清单，再验证重建；不同编译器的 DLL 不承诺逐字节一致。

## 仓库结构与文档

| 路径 | 内容 |
| --- | --- |
| `translation/` | 译文、术语和字幕源稿 |
| `rebuild/runtime/` | 当前中文显示与字幕代理源码 |
| `rebuild/assets/` | 定稿汉化资源、字库与清单 |
| `video_subtitles/` | 15 个已校准的视频字幕 |
| `tools/`、`releases/` | 提取工具、历史脚本与修订记录 |

- [构建与修改译文](BUILDING.md)
- [技术调查与历史记录](HANHUA_NOTES.md)

## 问题反馈与来源

遇到漏字幕、乱码或流程卡住，请在 [Issues](https://github.com/jyh9521/BloodRayne2-CN/issues) 中附上版本、关卡、触发位置和台词；截图或日志有助于复现。

原始游戏与商业资源归原权利人所有。本仓库的汉化资源快照不等于完整游戏，使用与重建均需自行准备支持版本；各第三方组件保留其原有归属，本说明不另行授予统一开源许可。

> [!NOTE]
> 本文件是 **2026-09-30 修订阶段保存的 `README.md` 快照**（归档于 `releases/readme-20260930/`）。当前最新版项目首页请查看根目录 [README.md](../../README.md)。

# 《吸血莱恩 2：终极剪辑版》简体中文汉化

BloodRayne 2: Terminal Cut 的简体中文汉化补丁，适用于 Steam 终极剪辑版。

包含菜单、招式说明、剧情字幕、游戏提示和预渲染动画字幕，也为部分原本没有字幕的对白、战斗台词补上了中文。配音保留英语。

## 下载与使用

**[下载覆盖版汉化补丁](https://github.com/jyh9521/BloodRayne2-CN/raw/refs/heads/main/dist/BloodRayne2_CN_current.zip)**

1. 退出游戏，把压缩包里的全部内容解压到游戏根目录，覆盖同名文件。
2. 启动游戏，将文本语言设为 **Russian（俄语）**，并开启字幕。
3. 正常游玩即可。中文借用了俄语文本槽，配音仍然是英语。

没有安装器，不需要运行脚本，也不用安装字体或其他工具。

解压后，目录结构如下：

```text
BloodRayne 2 Terminal Cut/
├─ rayne2.exe
├─ dinput8.dll
├─ LANGUAGE.POD
└─ video/
   └─ *_RU.srt
```

Steam 里右键游戏 → **管理 → 浏览本地文件**，即可找到游戏目录。

## 模组兼容与卸载

汉化只替换 `LANGUAGE.POD`、`dinput8.dll` 和 `video` 里的 RU 字幕，**不替换 `W32ART.POD`、`W32ENSND.POD`、视频文件或存档**。

因此，覆盖这两个 POD 的材质、模型或声音模组不会直接把汉化覆盖掉。不过，如果模组也带有 `dinput8.dll`，两者会占用同一个文件，不能直接叠加。

覆盖前建议自行备份同名文件。需要卸载时，把备份还原，再删除原本没有的汉化文件即可。

## 仓库目录结构

| 目录 | 内容 |
| --- | --- |
| `translation/` | 译文、术语表和字幕源稿 |
| `rebuild/runtime/` | 中文显示与字幕补全的代理 DLL 源码 |
| `rebuild/assets/` | 用于重建的汉化资源和字库 |
| `video_subtitles/` | 已调好时间轴的游戏视频字幕 |
| `tools/` | 文本提取、资源处理和构建工具 |
| `releases/` | 历次修订的脚本与技术记录 |
| `dist/` | 打包好的覆盖版补丁（历史快照期路径） |

## 从源码重建

需要 Windows、Python 和 Visual Studio C++ Build Tools，以及一份 Steam 原版 `LANGUAGE.POD`。无需保留之前的本地工作目录。

```powershell
git clone https://github.com/jyh9521/BloodRayne2-CN.git
cd BloodRayne2-CN
python rebuild/build.py --base-pod "你的原版LANGUAGE.POD路径"
```

生成的 `build/reproducible/BloodRayne2_CN_rebuilt.zip` 同样是解压覆盖版。

详细环境要求、构建步骤和修改译文的方法见 **[BUILDING.md](../../BUILDING.md)**。

## 问题反馈

汉化仍在测试中。如果遇到漏字幕、乱码或卡住，请到 [Issues](https://github.com/jyh9521/BloodRayne2-CN/issues) 反馈，尽量说明关卡、触发位置和当时的台词；附上截图或日志会更有帮助。

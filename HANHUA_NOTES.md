# BloodRayne 2: Terminal Cut 技术调查与开发记录

> [!NOTE]
> 本文档整理自 2026-07-31（Asia/Tokyo）前后的早期逆向与架构探索阶段（v1–v11），记录了原版文件基线、POD3 格式、字体渲染机制、英语语音分支补丁及全量文本提取方案。如需了解当前最新版的安装与源码重建方式，请查看 [README.md](README.md) 与 [BUILDING.md](BUILDING.md)。

## Steam 纯净版基线

本项目统一以 2026-07-30 19:03:05 重新安装的 Steam 纯净版作为构建基线，禁止使用旧版本遗留的导出文件、探针或已修改 POD 作为输入。

| 文件 | SHA-256 |
| --- | --- |
| `rayne2.exe` | `EAE3925A344668500F3533EBEC146376C8B1BBDB168ED2C3B1D4DECB0ABF0952` |
| `COMMON.POD` | `1CC9A191D478AE87F97B454FD1D0419CE06A31709F8CB5B9D011D75EF83E3F18` |
| `LANGUAGE.POD` | `2E3E0797E147E58B7308D3FCEDBDF0A06CB433FB099FC24B8FB38943D8B358E4` |
| `W32ART.POD` | `8B9A06FEEA6881026C485100BA24AADFD1C8ADA60400327C7E0F87098FF84631` |

所有构建与安装脚本在执行前均须校验上述基线哈希。

## 格式与引擎结构结论

- `LANGUAGE.POD` 与 `W32ART.POD` 均为 **POD3** 格式。
- POD3 目录项长度固定为 20 字节：名称偏移、文件大小、文件偏移、时间戳、CRC（算法为 **CRC-32/MPEG-2**）。
- 英语关卡文本位于 `LANGUAGE.POD` 的 `WORLD\EN\*.TXT`；俄语完整菜单与关卡文本位于 `WORLD\RU\*.TXT`。
- 菜单文件 `WORLD\RU\MSGLIST.TXT` 采用键值对格式（键为英文，值为俄文）。
- 关卡文本为逗号分隔结构，最后一列为显示文本，控制标记与音频文件名必须原样保留。
- 俄语菜单文件头包含 `Double byte support` 和 `Double byte font` 字段；原版默认关闭双字节支持，但保留了 CP932 编码的日文字体名。
- `rayne2.exe` 导入了 `IsDBCSLeadByte`、`CreateFontA`、`TextOutA`、`ExtTextOutA` 和 `GetTextExtentPoint32A`，表明引擎内部具备 GDI 双字节字形路径。

## 中文语言槽选择

采用俄语（`RU`）槽作为技术入口的原因如下：

- **英语槽（EN）**：虽有 70 个关卡文本条目，但没有独立的 `MSGLIST.TXT`、`MOVELIST.TXT` 和 `CREDITLIST.TXT`，界面高度依赖默认内置资源，不适合作为独立覆盖目标。
- **德/西/法/意语槽**：虽具备本地化菜单与招式表，但共享西欧扩展字体资源，修改会互相影响。
- **俄语槽（RU）**：拥有完整的菜单、招式表、职员表和关卡文本，且具备独立的 524,320 字节 `GOTHICTITLE_RU.TEX` 与独立 FNT 文件。覆盖俄语槽不会影响英语及其他西欧语言，是最干净的中文承载方案。
- **配音解耦**：`rayne2.exe` 同时引用 `W32RUSND.POD` 与 `W32ENSND.POD`，直接选择 RU 会联动切换俄语配音，因此需通过运行时补丁将配音强制保持为英语。

## 渲染、语音与字库架构演进（v1–v11）

### 1. 双字节与位图字体探针（v1–v3）

- 保持 POD 总长度不变并正确更新目录项大小与 CRC 后，游戏可正常挂载修改后的 `LANGUAGE.POD`。
- 开启双字节支持并填入原 CP932 字体名后，`日本語`、`漢字` 等探针能够进入主菜单并按双字节长度切分，但渲染为错位位图字形。同屏对照确认：主菜单与实时演出字幕均不走系统 GDI 字体，而是共用 `W32ART.POD` 中的 `GOTHICTITLE_RU` 语言专用位图字体。
- 静态分析确认：双字节解析器会保留前导字节，但 `GOTHICTITLE_RU` 渲染器仅取尾字节查表。早期测试使用 `0x81` 作为前导字节、`0xC0` 起的尾字节作为 63 个可控槽位，先生成 13 字主菜单字库（`新游戏 / 载入游戏 / 选项 / 额外内容 / 退出`）。
- `dinput8.dll` 代理将 `IsDBCSLeadByte` 固定至代码页 936，并将四个 GDI ANSI 字体/文本 API 转发至宽字符路径；运行日志确认五个目标导入全部挂钩（`patch_mask=0x1F`）。
- `W32ART.POD` 局部补丁器在临时副本上通过了安装与恢复的逐字节往返验证（恢复后精确回到原 SHA-256）。
- **2026-07-31 实机确认**：五项主菜单中文显示正常（间距、描边、高亮与纵向布局无异常），但直接写入原生 GBK 的字幕测试句会按尾字节误命中菜单字形，未替换俄文经清理后显示为问号。
- 将字幕同样改为 `0x81 + 尾字节` 载体编码后，菜单与两句字幕合计 37 个唯一汉字/标点排入纹理底部三行：
  - `LANGUAGE.POD` SHA-256：`8385183FA1BBC27781B857EA930F95DA22147C80871BB827CACB6DCCD27566B8`（386 个条目 CRC 全部通过）
  - `GOTHICTITLE_RU.TEX`：524,320 / 524,320 字节；`GOTHICTITLE_RU.FNT`：3,053 / 3,104 字节（均未超出原物理槽）
- 第二轮实机测试出现主菜单与中文句全部空白（仅 ASCII 问号可见），排查发现原因为压缩 FNT 时误删了冒号后的必需空格，导致老解析器忽略全部新增映射。恢复 `字节: 空格坐标` 格式后（FNT 为 3,090 / 3,104 字节，构建 `menu-subtitle-v3-space-fix`），第三轮实机截图确认“莱恩：字幕编码路径正常。”完整正常显示。

### 2. 英语配音分支逆向（v4–v5）

- 资源比对显示：`W32RUSND.POD` 的 6,803 个资源名全部存在于 `W32ENSND.POD`（英语包额外多 756 个资源）；两包共有 3,749 个 `.LPB` 口型条目且 CRC 完全一致，3,053 个 WAV 中有 1,663 个内容不同。
- **v4（已弃用）**：尝试在 `dinput8.dll` 中挂钩 `CreateFileA/W` IAT，将 `W32RUSND.POD` 重定向至 `W32ENSND.POD`（构建 `menu-subtitle-v4-en-voice`，代理 SHA-256：`2CF6AEF37C4F26672D08E469ED65B8AA8038DE6528B5C5A327ABAE9716EB4954`）。实机测试发现角色无配音且出现敌人服装异常，整包文件句柄重定向方案随即彻底弃用。
- **v5（采用）**：静态分析定位到纯净 EXE 的原生语音选择分支：
  - VA `0x00549151`：`cmp esi, 7`
  - VA `0x00549154`：原字节 `75 45`（RU 时走俄语包，非 RU 时跳往英语分支 `0x0054919B`）
- v5 代理移除文件 API 挂钩，仅在内存字节精确匹配 `75 45` 时改为 `EB 45`，强制走原生英语语音分支，同时保持文本槽为 RU/中文（日志显示 `patch_mask=0x3F ... voice=EN`）。
  - 构建编号：`menu-subtitle-v5-en-voice-branch`
  - 代理 SHA-256：`BF00F99D8EEFC03985F9B8705E94C8909751B62B2CC6E3DEDB8921EF131C0955`
  - **2026-07-31 实机确认**：英语配音成功恢复。

### 3. 菜单批量字库与载体槽排错（v6–v9）

- **v6（`menu-ui-v6-source-han-regular`，已弃用）**：改用思源黑体同源字体 `NotoSansSC-VF.ttf`（Regular 400、30px、无描边），将字体纹理从 512×256 扩至 2048×1024（原纹理保留在左上角），并改为 `W32ART.POD` 归档末尾追加式安装（在 1,024,554,218 字节纯净副本上通过往返恢复验证）。导入 `translation/menu_ui_zh.tsv` 共 219 条 PC 菜单文本（补入 5 个能力名与环形摇杆提示，排除 6 条 Xbox 专用存储提示），按可变宽词组字形去重后占用 188 个槽位：
  - `LANGUAGE.POD` SHA-256：`88F6CDDB9C3AF3FA087B57E230D0D1E0F2F808EE20880128268E8A98D1F21E2E`（386 条目 CRC 通过）；TEX 8,388,640 字节，FNT 4,648 字节。
  - **问题与废止**：实机测试显示自定义按键右栏、音频状态区与显示说明区出现串字与重叠。根因是 v6 复用了 `0x40–0x7E` 英文字母槽位，导致引擎动态生成的 ASCII 按键名和状态值误命中中文词组。该方案判定无效。
- **v7（`menu-ui-v7-ascii-safe`，已迭代）**：完整保留 A–Z、a–z、数字、斜杠及 `[`，仅复用 9 个未用标点槽（`40/5D/5E/5F/60/7B/7C/7D/7E`）与 `0x80–0xFE` 高位槽（共 136 个载体）。菜单范围收敛至 152 条核心文本，其余 585 条回退英文键名以消除俄文高字节冲突：
  - `LANGUAGE.POD` SHA-256：`17300FFCE31843BAF55CEAF02A7B7A7EABB5FB7934845EF029951B2E2E4D19D2`
  - TEX SHA-256：`DB60E707214B3D80838BAD126B0788C91EDC687C3AFAE4D9BDF2573F6F7AFB9A`
  - FNT SHA-256：`BC0640E1D067E7AAC418A400778446808F035324E50462333AF15FBD088ADEDE`
  - **问题**：实机测试发现每个中文选项前都多出“锁定目标”。排查确认“锁定目标”分配在 `0x81`，而 `0x81` 同时是前导字节——只要 FNT 为 `0x81` 定义字形，渲染器就会在每个中文词组前多画一次它。
- **v8（`menu-ui-v8-high-slots-only`）**：彻底禁止复用可打印 ASCII（`0x21–0x7F` 原样保留）和前导字节 `0x81`，载体严格限定在 `0x80` 与 `0x82–0xFE`（共 126 槽）；核心菜单收敛为 142 条，其余 595 条回退英文键名：
  - `LANGUAGE.POD` SHA-256：`00EA822DB513070EA2D0894FC41E37D5F78E37B61F68E0AF27FD9890859F1550`
  - TEX SHA-256：`475680D6028DF8471E18EA30DF3E3AD6F88268BD2B8D8FB6032CAE19F326292B`
  - FNT SHA-256：`80C5C413B69A193092772EBC8269FC3EC8D174526F25E867327B433855363F34`
  - **实机反馈**：前缀串字与 ASCII 冲突完全消失；同时发现 3 处漏翻（`FMV Presentation`、`Widescreen`、`Original` 为新版 EXE 内置键，原俄语 `MSGLIST.TXT` 中不存在；另发现 `Enter Cheat` / `Enter Cheat Code` 未翻译），且演职员表因 `WORLD\RU\CREDITLIST.TXT` 残留 CP1251 俄文高字节而错显中文长串。
- **v9（`menu-ui-v9-missing-labels-credits-safe`）**：追加 3 条 EXE 内置键并补译秘籍相关键（共 148 条中文菜单值，通过共享词块“自定义”保持在 126 槽内；592 条回退英文）；演职员表替换为 `COMMON.POD` 自带的英文原件 `DATA\CREDITS.TXT`（6,999 字节、363 行、0 高位字节，SHA-256：`D8539B6D58F1B84FFAF325CB4610614EA7C0968CFA538DA17EC4C945CAD83079`）：
  - `LANGUAGE.POD` SHA-256：`6D7607E95DBF0D34646DA0FF63F2B8F6B3BA0AB3EC0BDA610658E9718E7FE56D`
  - TEX SHA-256：`3D9E965008269D56BDD43BE8F8611B4A1D75D69542C5608EB08EC6E87E586992`
  - FNT SHA-256：`74DBF7C14A5AD466C5C1844F5FA8F45A9A3042A56DC09CD2607183A28EDE4D8C`
  - `W32ART.POD` 安装后哈希：`C69412DE9937BBD1D591916CF9B1A61316AD9B76B2594D38165AF0B44BA60B22`（通过往返恢复验证）
  - **2026-07-31 实机确认**：漏翻与演职员表异常全部解决。

### 4. 自包含 POD 挂载与动态字形架构（v10–v11）

- **v10（`menu-ui-v10-language-font-overlay`）**：
  - 静态分析确认纯净 EXE 基础归档挂载顺序为 `W32ART, W32MODEL, W32SET, W32SOUND, LANGUAGE, COMMON, W32ART2, W32ART3`（同名资源优先取靠前者），且 `W32ART.POD` 与其余 7 个归档无任何重名条目。
  - 将中文字体 TEX/FNT 直接内置进 `LANGUAGE.POD`（共 388 个条目），并由代理在内存中仅交换 `LANGUAGE.POD` 与 `W32ART.POD` 的挂载顺序，使汉化不再修改 `W32ART.POD`，从而与材质/模型 Mod 彻底解耦。
  - `LANGUAGE.POD` SHA-256：`DAC807D9045EC6B15B8DFC6AF45DAEFE989186036E82626853D16D762A6AE04D`
  - 代理 SHA-256：`67AFAD6D8F3EE13FB5C36632B57EF2B9B98E572EB3D82E11AFD8BC815190901C`（保留英语语音分支补丁）
  - **2026-07-31 实机确认**：同时覆盖 Mod 版 `W32ART.POD` 与 `W32ENSND.POD` 后，中文显示与英语配音均正常。
- **v11（`full-ui-v11-dynamic-glyphs`）**：
  - 完整菜单（740 条）与招式表（94 条）即使复用词块也需要约 699 个槽位，超出 126 槽静态上限。v11 改为**自定义双字节字码 + 运行时动态字形装载**：
    - 文本载体采用互不重叠的前导范围 `0x81–0x9F` 与尾字节范围 `0xA1–0xFE`，最大可编码 **2,914** 个非 ASCII 字符。
    - 前导槽保持零宽，94 个尾槽在排版阶段统一提供单字宽度；代理挂钩纯净 EXE 的 `CBitFont::drawText`（RVA `0xE6AB0`），每次绘制前按当前字符串动态重写尾槽 UV 坐标，完整保留原引擎的排版、阴影、高亮与批处理，且不占用任何可打印 ASCII 字形。
  - v11 实际装载 1,001 个动态字形（最长单句含 40 个不同非 ASCII 字形，低于单次绘制 94 槽上限），字体纹理扩至 2048×4096（`Noto Sans SC` Regular 400、30px、无描边）。
  - 导入内容：菜单 739 条中文 + 1 条结构空行、招式表 94 条中文、114 个已确认对白单元回填至 145 个位置、2 个空白单元对应的 3 个位置清空，其余 2,450 个位置暂回退英文（8 条俄语独有文本正常承载），英文演职员表不变。
  - `LANGUAGE.POD` SHA-256：`6A8CDEDBBD78BA169C904620A16A057184195ACE31AF7CAD563BD36FB10EC9C5`（388 条目 CRC 全部通过）
  - 代理 SHA-256：`6D3CE1EB2BB9435810B6EA509E28762261C50BABF52A40D67644220E8EF981F1`

## 全量文本目录概况（2026-07-31 快照）

- 使用 `tools/export_translation_catalog.py` 从纯净基线直接导出 `translation/catalog.tsv`，总计 **3,715 条**：
  - 剧情、字幕与提示：2,598 条（拆分至 `translation/extracted/dialogue.tsv`）
  - 菜单：740 条（`translation/extracted/menu.tsv`）
  - 招式表：94 条（`translation/extracted/moves.tsv`）
  - 演职人员可见文本：283 条（`translation/extracted/credits.tsv`）
- **对白对齐情况**：2,598 条中有 2,590 条自动对齐同名英语条目（其中 3 条英语原文为空，构建时清空俄语残留）；剩余 8 条为俄语版独有（7 条补充任务目标 + 1 条独有对白），以俄语语义为准。英语槽另多出 9 个其他语言没有的旧版/合并关卡文件，未纳入俄语槽构建。
- **统计与规则**：基线哈希与覆盖统计记录在 `translation/inventory.json`；字段规范及 4 处招式表结构差异见 [translation/extracted/README.md](translation/extracted/README.md)。
- **早期进度快照**：
  - 菜单（739 条有效译文 + 1 条空键值，含补齐的 2 条 `%s` PS2 存储卡错误及 `acquired/acquired2`）与招式表（94 条）已全部完成并校验 `@@...@@` 控制标记；演职人员保持英文原件。
  - 61 个 `WORLD\RU\*.TXT` 中的 2,598 条对白按“英文 + 俄文”完全一致去重后归并为 **1,706** 个唯一翻译单元（454 个单元存在重复，减少 892 条重复工作），分别存放在 `translation/dialogue_deduplicated/dialogue_unique.tsv` 与 `dialogue_occurrences.tsv`。该时间点状态为：114 条已翻译、1,465 条待翻译、125 条重复译文冲突待复核、2 条应清空。

## 工程与测试原则

1. **构建与测试分离**：所有分析、导出、构建和离线校验均在独立工程目录完成，脚本不自动启动或操控游戏进程，实机视觉验证由人工完成。
2. **可逆安装**：测试阶段替换原版文件前必须先生成可校验备份，确保可精确还原。
3. **保留控制标记**：所有控制标记（如 `@@newline@@`、按钮占位符、`$`、`@角色名`）与格式占位符一律不得翻译或破坏。
4. **每次构建必检**：每次构建均校验 POD 结构、条目范围、CRC 与基线哈希。

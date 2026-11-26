# BloodRayne 2 对白去重翻译表

> [!NOTE]
> 本文档中的进度数字（如 114 条已翻译）记录的是 2026-07-31 去重初始化阶段的状态快照；字段格式、映射机制与导入规则仍然适用。

## 翻译入口与字段规则

正式翻译和校对只需编辑 `dialogue_unique.tsv`：

- 填写或修改 `translation_zh` 列。
- 定稿后将 `status` 改为 `translated`。
- **`needs_review`**：同一句原文在原始表中存在多个不同译文，当前默认保留最早出现的版本，需人工统一措辞。
- **`blank`**：英语原文显示文本本身为空，构建时会自动清空俄语残留。

> [!IMPORTANT]
> 请勿修改 `unit_id`、`source_en`、`source_ru` 以及出现次数（occurrence）统计列。`dialogue_review.xlsx` 仅作为便于筛选冲突的审阅副本；自动导入以 TSV 为准，如果在 XLSX 中做了修改，请同步回 `dialogue_unique.tsv`。

## 去重与回填机制

- 纯净 `LANGUAGE.POD` 中包含 61 个 `WORLD\RU\*.TXT` 关卡文本文件，共计 **2,598 条**原始显示文本。
- 按“英文显示文本 + 俄文显示文本”完全一致归并后，得到 **1,706 条**唯一翻译单元（其中 454 个单元出现多次，减少了 892 条重复翻译工作）。
- `dialogue_occurrences.tsv` 完整保留了全部 2,598 个实际资源位置（文件、行号与键）。

去重不会删减任何游戏资源。最终构建时，`tools/build_full_translation.py` 会按 `unit_id` 将每条唯一译文分别回填到 `dialogue_occurrences.tsv` 对应的全部位置。未定稿的 `needs_review` 与 `pending` 条目在构建时暂回退为英语，定稿后重新构建即可自动覆盖所有重复位置。

## 初始去重快照统计

| 状态 | 唯一单元数 | 说明 |
| --- | ---: | --- |
| `translated` | 114 | 对应 145 个实际资源位置 |
| `needs_review` | 125 | 已有重复译文措辞不一致（均未丢失 `@@...@@` 控制标记） |
| `pending` | 1,465 | 待翻译 |
| `blank` | 2 | 英语原文为空，导入时清空 |
| **合计** | **1,706** | — |

其他文本策略：

- `extracted/menu.tsv`：739 条有效菜单文本（外加 1 条原始空键值），已完成。
- `extracted/moves.tsv`：94 条招式表文本，已完成并校验控制标记。
- 演职人员表：保持不翻译，构建时直接使用当前 Terminal Cut 英文原件。

## 同步与校验命令

重新校验或同步表格时运行：

```powershell
python _cn_project/tools/prepare_translations.py
```

脚本会按输入文件的 SHA-256 在 `translation/backups` 下自动备份表格。已在 `dialogue_unique.tsv` 中标记为 `translated` 的人工定稿拥有最高优先级，下次同步时不会被旧的重复行冲突覆盖。

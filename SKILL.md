---
name: study-review-builder
description: 复习资料整合生成器（Review Material Builder）。把课件/PPT/PDF/Word/真题等多源资料整理成成套期末复习资料、培训考核或资格考试复习材料（学生期末/考研/四六级、企业内训、职场培训等场景）：自动划分知识点、真题数据驱动星级（1-5星）、原卷画像适配（按原卷题型/题量/难度/分值出预测卷）、可点击目录 Word（PDF 可选）、思维导图、四视图交互 HTML、两套预测试卷；支持联网补全素材、MCP/外部题库接入、语言类学科出完整试卷。适用于多份资料成套整合的场景（单独画一张思维导图或单独出一道题不属于本技能）。触发词：整理复习资料、整合课件、考前复习、考前突击、划重点、知识点划分、重要度星级、按原卷出卷、适配真题结构、原卷画像、出套题、出试卷、思维导图、知识图谱、自测、预测试卷、期末冲刺、培训考核、四六级模拟卷、英语阅读理解、导出 Word/PDF/HTML；English triggers - organize review materials, generate practice exams from past papers, consolidate lecture notes.
name_cn: 复习资料整合生成器
description_cn: 把课件/PPT/真题等多源资料整理成复习资料全套：Word（PDF 可选）、思维导图、四视图交互 HTML（复习/图谱/自测/预测试卷，日夜主题）、两套预测试卷；适用于期末/考研/四六级与企业培训考核。支持联网补全素材与 MCP/外部题库接入，语言类学科出题带完整材料；v3.5 支持原卷画像适配、题目结构自检、个人信息检测与 doctor 自检。触发词：整理复习资料、考前突击、划重点、出套题、思维导图、自测。
metadata:
  version: 3.5.1
create_source: super-agent-skill-creator
---

# 复习资料整合生成器（Study Review Builder）

## Overview

把多份零散课程资料（课件 PPT、PDF 教材、Word 笔记、真题等）整合成**结构化复习资料**，一次产出全套交付物：

1. **Word 文档**（.docx，PDF 需用户选择）：可点击目录、思维导图页、星级正文、真题命中标注
2. **交互式 HTML**（单文件、零依赖、双击即用）：复习资料版（星级筛选）+ 知识图谱 + 自测版（列表式 + 模糊揭晓）+ 预测试卷四视图，支持**日夜双主题、亮度与冷暖色调调节、护眼模式**
3. **思维导图 PNG**：大气简洁的水平树（Word/PDF 嵌入用）；可选力导向图谱
4. **两套考试预测试卷**（Word，PDF 可选）：A/B 卷 + 参考答案与解析

**v3 工程化增强**（相比 v2 的核心改进）：
- 🛡️ **JSON 校验与自动修复**：新增 `check_review.py`，流水线前先体检；`--fix` 三类自动修复——字符串内部裸引号转「」、字符串内裸控制字符/换行替换为空格、无效转义（如 \0）反斜杠转义，杜绝"一处坏、四脚本连环报错"；
- 🔍 **答案草稿痕迹检测**（v3.2）：校验阶段自动扫描试卷答案/解析中的"修正/重算/待验证/TODO/？——"等草稿标记并 WARN，防止草稿文案流进交付物；
- 🧩 **大 JSON 分片构建**（v3.2）：章节数多（如 10 章 76 课件）时分片编写再合并，每片先 `--syntax-only` 语法校验，合并后全量校验；
- 📦 **依赖一键安装**：新增 `setup_deps.py`，自动检查并安装全部依赖；
- 📂 **多源统一提取**：新增 `extract_sources.py`，PPT/PDF/Word/文本一条命令全部提取，无需逐个装依赖；
- ✅ **交付前体检**：每个生成脚本输出内容统计（章节/知识点/题量/分值/文件大小），总分与题分合计等约束自动校验；
- 🧪 **边界回归检查**：可运行 `python scripts/regression_check.py` 验证校验器对非法结构和依赖缺失的处理；
- 🧭 **真题智能识别**：自动区分"真题"与"复习笔记"，无真题时按明确准则评估星级，星级区分度有依据。

> 版本迭代记录见 [references/VERSION.md](references/VERSION.md)（v1→v2→v3 改进说明，可作参赛答辩材料）。

## 一句话用法（用户只需这样说）

> 「帮我把这些课件和真题整理成复习资料，标重点，再出两套预测试卷」（附上文件即可）

只要话里带「整理复习资料 / 考前复习 / 划重点出套题」这类意图（且涉及多份资料或成套产出）时自动调用本 Skill，无需复杂指令；若只要单张思维导图或单独一道题，直接对话即可，不必调用本 Skill。

## 何时使用本 Skill

- 用户提供课件/资料/真题，说「整理成复习资料」「出思维导图/图谱」「出两套预测试卷」「我要自测」
- 需要可打印、可交互、可自测的复习材料

## 工作流程（Workflow）

### 何时应主动追问用户（v3.5 新增，贯穿全程）

遇到以下信息不全或冲突时，**先用一句话向用户确认再继续，不要擅自假设**：

1. **原卷画像信息不全**：原试卷只有题目没有分值/题量标注 → 追问「总分和每题分值是多少」；
2. **画像与口头要求冲突**：原卷 100 分但用户说「出 50 分的卷」→ 确认以哪个为准（保留原卷结构等比缩放，还是按用户要求重配题型）；
3. **满分未知**：学科满分非 100 且无原卷（如考研专业课 150 分）→ 追问实际满分与题型；
4. **资料明显不足**：仅 1-2 份零散资料且未说明课程范围 → 确认是否还有更多课件/真题；
5. **星级无依据**：无真题且课件无重点标记 → 确认考试侧重点（概念/计算/综合）。

其余情况用合理默认值继续，并在交付时说明所做假设。

### 第 0 步：依赖检查与健康自检（v3 新增，一次即可）

```bash
python scripts/setup_deps.py --with-extract   # 一键安装/检查全部依赖
python scripts/setup_deps.py --doctor         # 健康自检：依赖/字体/输出目录可写
python scripts/setup_deps.py --doctor --format json   # 机读报告（agent 自动化用）
```

- `--with-extract` 安装输入提取依赖（python-pptx、pymupdf）；只生成不提取时可省略。
- `--doctor`（v3.4 新增）：分级 PASS/WARN/FAIL 检查运行环境——依赖可导入、matplotlib 中文字体可用、输出目录可写；FAIL 时非零退出码，`--format json` 供 agent 机读。交付前跑一次可提前暴露环境问题。
- 依赖缺失时各脚本会给出友好提示，不会直接崩溃。

### 第 1 步：收集输入文件

**统一提取（v3 推荐）**：

```bash
python scripts/extract_sources.py 课件1.pptx 课件2.pdf 复习.docx --out .temp/sources
python scripts/extract_sources.py "F:\课件目录" --recursive --out .temp/sources   # 目录直接作为位置参数
```

一次提取 PPTX/PDF/DOCX/TXT/MD 为统一文本，输出到 `--out` 目录（不存在时自动创建），AI 直接读取即可。生成类脚本（docx/试卷/HTML/导图）的输出路径父目录不存在时同样会自动创建。

| 源文件类型 | 提取方式 |
|---|---|
| .pptx（课件） | `extract_sources.py`（python-pptx，含表格） |
| .ppt（旧式二进制课件） | 需先转换为 `.pptx`；脚本会明确提示不支持 |
| .pdf（教材/资料/真题） | `extract_sources.py`（pymupdf）；复杂排版（公式/多栏）用 PDF 技能或 PaddleOCR |
| .docx / .txt / .md | `extract_sources.py`（python-docx，段落+表格） |
| 图片（板书、截图） | 用视觉模型识别文字 |
| 网页 / 链接 | 用 webfetch 抓取正文 |

> 先完整读取所有源文件内容，再进行知识点划分，避免遗漏。

### 第 1.5 步：素材补全（联网搜索 / MCP / 外部题库）

**原则：先用户资料，后外部补全。** 当用户资料不完整（缺真题、缺阅读原文、缺范文、缺听力原文）时，主动按以下顺序补全素材：

1. **联网搜索优先**：用 `online_search` 检索缺失素材（如「四级英语 阅读理解 真题 原文」「XX 课程 名词解释」等），命中后经 `webfetch` 抓取正文，**必须记录 `source`**；不可用时改为 AI 原创拟写（`source` 标注"AI 原创模拟"）。
2. **MCP / 外部数据源接入**：若环境提供 MCP 工具（外部题库、词典、学科数据库等），**优先调用 MCP 工具**取数并写入 JSON（带来源）；不可用则跳过，用联网搜索兜底。
3. **本地题库文件接入**：用户提供的题库文件（.json/.csv/.txt/.docx，含题目与答案）按统一结构并入素材库，用于出题与真题统计。

> 版权与真实性说明：联网抓取的真题/范文仅作教学素材，`source` 字段必须记录出处；无法核实的题目一律标注「模拟题」，不得暗示与真实考试一致。

### 第 2 步：确定组织结构（自动 + 可选用户大纲）

- **默认（自动）**：AI 根据内容自动划分章节与知识点。
- **可选（用户介入）**：用户提供自定义大纲/重点清单时，优先按用户大纲组织。

### 第 3 步：划分知识点并生成匹配词

- 把每个知识点要点提炼为通顺简洁的段落，补充典型例题/考点（`examples`）。
- 为每个知识点生成 3-6 个 `match_terms` 匹配词（**选真题中实际出现的术语**），用于真题统计与图谱考点关联（跨章节共享词会连成虚线）。

### 第 4 步：真题考点量化（数据驱动星级）⭐ 核心差异化

**4.1 先判断用户提供的是真题还是复习笔记（v3 新增）**：

| 判据 | 真题 | 复习笔记 |
|---|---|---|
| 是否含「一、单项选择题」等题型结构/题号 | 是 | 否 |
| 是否含"得分/每题 X 分/考试时间"等字样 | 是 | 否 |
| 是否含答案与解析 | 可能（后面附答案） | 通常有 |

- 判为**真题**（.txt/.md/.docx）：运行真题统计引擎：
```bash
python scripts/exam_stats.py --json review.json --exams 真题1.txt 真题2.docx --mode blend --update
```
- **数据星级公式**：`score = hits / max_hits`，`data_stars = 1 + round(score × 4)`；融合模式 `--mode`：`blend`(默认 0.6数据+0.4AI) / `data` / `ai` / `max`；`--update` 写回 review.json。
- **模式选择指南（v3.2 实践经验）**：`blend` 的关键词匹配含子串噪声——"图""树"等常用字会拉高低频知识点的命中、压低真实重点的占比；真题集中且为选拔性考试（考研/期末统考）时建议 `--mode max`（数据星与 AI 星取高值），既保留数据证据又避免噪声压分。曾实测：同一份数据结构 review，blend 下高频知识点占比 19% 被压到不足 10%，max 下恢复至 19% 且与重邮 802 真题分布一致。

- 判为**复习笔记 / 无真题**：按 **AI 星级评估准则**（v3 新增）综合打分：
  1. 课件篇幅占比大（页数多）→ 加星；
  2. 跨章节重复出现的关键词/考点 → 加星（如"加密""鉴别"在多章反复出现）；
  3. 笔记/课件中出现"重点""必考""常考""记忆"等强调标记 → 加星；
  4. 老师课堂强调/作业布置内容（如本次第二章作业"归纳嗅探、截获和拒绝服务攻击方法"）→ 加星。
  星级要求有区分度（避免全是 3 星），5 星 ≤ 全部知识点的 1/3。

> 把两者结合：即使笔记，也把笔记文本当作语料跑一遍统计，作为辅助参考。

### 第 5 步：编写两套预测试卷题目（出题逻辑完整性）

在 review.json 中编写 `mock_exams` 字段（两套 A/B 卷，结构见 [references/review-schema.md](references/review-schema.md)）。

**5.0 原卷适配画像（有原试卷时必须先做）**：先从用户提供的原试卷提取顶层 `exam_profile`，不要凭经验套用 100 分模板。画像至少记录：总分、考试时长、总题量、各题型题量、每题分值/分值分布，以及易/中/难题量。生成 A/B 卷时，每套卷都必须带 `profile_id`/`question_type`，并逐题填写 `difficulty`（`easy`/`medium`/`hard`）。`check_review.py` 会逐套检查题型数量、分值分布、总分和难度数量；不通过时不得进入生成阶段。

原卷没有明确标注难度时，按认知要求估计：直接记忆/单步识别为 easy，概念辨析/两步推理为 medium，综合分析/计算/开放应用为 hard，并在 `exam_profile.notes` 说明依据。只有在确实没有原卷时，才使用默认题型和 100 分兜底模板。

**5.1 通用题型体系（题型多变性）**

| 题型 `type` | 说明 | 必备字段 |
|---|---|---|
| `choice` | 单项选择题（A-D） | `options` + `answer`(字母) |
| `judge` | 判断题（正确/错误） | `options`(A.正确 B.错误) + `answer` |
| `fill` | 填空题 | `q` 中留空（____），`answer` 填答案 |
| `short` | 简答题 | `answer` 为参考答案要点 |
| `reading` | 材料阅读题（先给材料再做题） | **`stimulus` 必须含材料原文** + 多道 `question` |
| `listening` | 听力题 | `stimulus` 为听力原文（注明"音频由 TTS/人工朗读"）+ 题目 |
| `translation` | 翻译题 | `stimulus` 为待译文本，`answer` 为参考译文 |
| `writing` | 写作题 | `q` 为题目要求，`answer` 为参考范文/要点 |

- 建议覆盖：选择/判断/填空 → 简答 → 材料阅读 → 综合应用，难度梯度递进；
- **满分规则**：默认总分 100；用户学科满分非 100（如考研 408/802 满分 150）时，按真实满分设置 `total_score`，并按原卷画像的题型/题量/分值配比生成。`check_review.py` 校验的是 **total_score 与各题分值合计一致**，以及原卷画像的题型数量、分值分布和难度分布，而非固定等于 100；
- 每题含题干、选项、答案、解析（对应知识点 kp_id）；A/B 卷题目不得重复；
- 若用户提供了真题，模拟真题风格出题。

**5.2 语言类学科出题铁律——杜绝"只有问题没有材料"**

1. 阅读理解：先给完整原文（英语 250~450 词/语文 500~900 字，段落分明），再附 3~6 道题；解析写清"对应原文第几段"；
2. 听力：`stimulus` 放听力原文，题面注明音频需另行生成；
3. 翻译：`stimulus` 给待译文本，`answer` 给参考译文，`analysis` 说明句法/词汇考点；
4. 写作：`q` 给写作要求，`answer` 给参考范文/提纲；
5. 素材优先第 1.5 步补全的原文，其次 AI 原创模拟（务必标注 `source`）。

**5.3 出题引擎（三级保障）**

| 引擎 | 触发条件 | 说明 |
|---|---|---|
| ① AI 内置出题（默认） | 始终可用 | AI 按 5.1/5.2 规则编写 `mock_exams`，质量最高 |
| ② API 模型出题（可选） | 用户配置了模型 API | `python scripts/llm_exam.py --json review.json --api-key ...` |
| ③ 确定性脚本兜底 | `mock_exams` 缺失 | `build_mock_exams.py` 内置 `_auto_papers` 自动组卷，保证一定有输出 |

**5.4 出题答案自检（v3.2 新增，交付前必做）⭐**

计算类题目的答案**必须逐题验算**，AI 生成的数值答案（哈夫曼编码/WPL、AVL 旋转过程、数组地址、ASL、排序趟数等）经常出错且不易察觉：
1. **重算核对**：每道计算题独立验算一遍（不是复述自己的推导，而是按公式重算）；易错点：森林棵数=根右链结点数（易误为数孩子）、AVL 每插一个结点都须立即检查是否失衡（可能失衡多次）、WPL 按新树深度逐叶重算；
2. **草稿清理**：答案/解析中不得残留"？——修正：""重算应为""待验证"等草稿标记，`check_review.py` 会自动扫描并 WARN；
3. **修复后重新生成**：JSON 中修过的答案，必须重新跑生成脚本刷新 docx/HTML（旧生成物不会自动更新）。

> 实测案例：数据结构 802 试卷曾检出 7 处答案缺陷（森林棵数、哈夫曼编码与 WPL、AVL 两次失衡漏一次、数组地址等），全部在交付前自检环节修复。

### 第 2.5 步（重要，写在 review.json 之后）：JSON 校验与自动修复（v3 新增）⭐

```bash
python scripts/check_review.py --json review.json --fix --strict
```

- 校验：JSON 语法 → 顶层字段 → 章节/知识点必填 → match_terms → **试卷总分与分值合计、kp_id 引用、A/B 卷不重复、材料题 stimulus 齐全、答案草稿痕迹扫描（v3.2）、题目结构自检与解析重复度检测（v3.4）**；
- `--fix` 三类自动修复（v3.2 增强）：
  1. 字符串值内部裸 ASCII 引号 → 转换为「」；
  2. 字符串内裸控制字符/换行（JSON 非法）→ 替换为空格；
  3. 无效转义序列（如 `\0`）→ 反斜杠转义为 `\\0`，保留字面量；
- `--strict`：把警告当错误（适合正式交付前）；
- `--syntax-only`（v3.2 新增）：只校验 JSON 语法、跳过结构校验，**用于分片文件的快速校验**；
- **通过后再进入第 6 步生成**，避免四个脚本连环报错。

> 📌 **JSON 书写规范（v3 硬性要求）**：review.json 中所有字符串内容**禁止使用 ASCII 双引号 `"` 包裹中文内容**（如 `"q": "下列属于"被动攻击"的是"` 会破坏 JSON）；中文引用一律用「」或中文引号。若出现该问题，`--fix` 可自动修复。

**题目结构自检（v3.4 新增，校验器自动执行）**：`check_review.py` 会按题型校验结构合法性，编写 mock_exams 时直接遵守可避免返工：
- `choice`：`answer` 必须是 `options` 中存在的选项键（A/B/C/D...），且选项 ≥2 个、每项含 `A. ` 前缀；
- `judge`：`options` 必须恰好为「A. 正确」「B. 错误」（v3.5 起校验器强制）；`answer` 推荐 A/B（渲染端直接显示原文），也兼容 对/错/正确/错误/TRUE/FALSE；
- `fill`：`q` 题干必须含留空符 `____`；
- **解析重复度**：同一套卷内 analysis 非空去重占比 < 60% 且题目数 ≥ 8 时 WARN（提示解析模板化，需逐题写实解析）。

> 违例将以 ERR 拦截（解析重复度为 WARN），`--strict` 下 WARN 也阻断交付。

**大 JSON 分片构建模式（v3.2 新增）**：章节数多、内容量大（如 10 章 76 课件 + 12 年真题）时，一次性写完整个 review.json 易超限出错。推荐分片：

```bash
# ① 分片编写：part1.json（第 1-5 章）、part2.json（第 6-10 章）、exams.json（试卷）
# ② 每片先做语法校验（片段无完整顶层结构，用 --syntax-only）
python scripts/check_review.py --json part1.json --syntax-only
# ③ 合并成完整 review.json（脚本或代码拼接顶层字段 + chapters + mock_exams）
# ④ 合并后全量校验
python scripts/check_review.py --json review.json --fix --strict
```

> 分片文件只需是合法 JSON（对象或数组均可），合并在内存中完成后再写盘；合并脚本放 `.temp/`。

### 第 6 步：生成全套交付物

```bash
# ① 思维导图 PNG（水平树，Word/PDF 嵌入用；--mode graph 可选力导向图谱）
python scripts/mindmap_image.py --json review.json --out mindmap.png --mode tree

# ② Word 文档（PDF 可选：用户需要时加 --to-pdf）
python scripts/build_review_docx.py --json review.json --mindmap mindmap.png --out 复习资料.docx [--to-pdf]

# ③ 两套预测试卷 Word（PDF 可选）
python scripts/build_mock_exams.py --json review.json --out-a 预测试卷A.docx --out-b 预测试卷B.docx [--to-pdf]

# ④ 交互式 HTML（四视图 + 主题系统）
python scripts/build_review_html.py --json review.json --out 复习资料.html
```

每个脚本末尾自动输出 `[STATS]` 交付前体检（章节数/知识点数/题量/分值合计/文件大小）。

**重要：PDF 是用户可选的**——默认只生成 docx；仅当用户明确要 PDF（如"导出 PDF""可打印版本"）时才加 `--to-pdf`。

**PDF 转换策略**（docx_to_pdf，docx 与试卷共用）：优先 LibreOffice(soffice)，不可用时自动降级 **Word COM**（Windows），两者都不可用则提示用户手动另存为 PDF。

### 第 7 步：交付

向用户交付全套文件并说明用法：
1. **Word**（及用户选择的 PDF）：可打印；Word 中按住 Ctrl 点击目录跳转；内嵌思维导图页
2. **HTML**：双击打开；四个标签页——📖 复习资料版（星级筛选）、🕸️ 知识图谱（拖拽/缩放/悬停高亮/点击跳转）、📝 自测版（知识点随机乱序、模糊遮盖、点击揭晓、标记掌握/再看看）、📋 预测试卷（A/B 选卷在线作答、交卷得分与错题清单）；右上角日夜主题、⚙️ 亮度/冷暖调节、护眼模式（自动记忆）
3. **思维导图 PNG**：可单独嵌入任何文档
4. **预测试卷 A/B**：打印后闭卷自测，答案与解析在每份试卷末页

**目录规范（v3 最佳实践）**：最终交付物放用户工作目录根；中间产物（提取文本、校验中间、临时 JSON）放 `.temp/`。

## 脚本速查

| 脚本 | 用途 | 关键参数 |
|---|---|---|
| `setup_deps.py` | **v3 新增**：依赖检查与一键安装；`--doctor` 健康自检（v3.4） | `--check` / `--with-extract` / `--doctor [--format json]` |
| `extract_sources.py` | **v3 新增**：多源文件统一提取为文本 | `paths...` / `--out` / `--recursive` |
| `check_review.py` | **v3 新增**：JSON 校验 + 三类自动修复 + 草稿检测（v3.2）+ 题目结构自检（v3.4）+ 个人信息检测（v3.5） | `--json --fix --strict --syntax-only` |
| `exam_stats.py` | 真题考点量化引擎（数据驱动星级） | `--json --exams --mode --update` |
| `mindmap_image.py` | 思维导图/图谱 PNG | `--json --out --mode tree\|graph --dpi` |
| `build_review_docx.py` | Word 文档（PDF 可选） | `--json --mindmap --out --to-pdf` |
| `build_mock_exams.py` | 两套预测试卷（PDF 可选，支持材料渲染，v3.1 起共享材料自动去重） | `--json --out-a --out-b --to-pdf` |
| `build_review_html.py` | 交互式 HTML（四视图+主题+材料渲染，v3.1 起共享材料自动去重） | `--json --out` |
| `llm_exam.py` | **可选**：API 模型出题引擎（未配置自动跳过） | `--json --api-key --base-url --model --out` |

> 依赖：`python-docx`、`matplotlib`（必需）；`python-pptx`、`pymupdf`（输入提取）。一键安装：`python scripts/setup_deps.py --with-extract`。

## 最佳实践

- `match_terms` 选真题中**实际出现的术语**（如"SYN 泛洪""令牌桶"），跨章节合理共享术语可生成考点关联虚线。
- **JSON 引号规范**：所有字符串内容用「」包裹中文引用，禁止 ASCII 双引号嵌套（`check_review.py --fix` 可自动修复）。
- 重要度星级要有**区分度**，有真题数据时优先 `blend` 模式；无真题时按第 4 步的 AI 星级准则打分；**真题集中且为选拔性考试时用 `max` 模式**（子串匹配噪声会压低 blend 下的真实重点）。
- 知识点标题用「序号 + 标题」形式，便于目录与图谱阅读。
- 预测试卷的解析要写清对应知识点，帮助学生回溯复习。
- 试卷难度建议覆盖：基础概念（选择）→ 理解辨析（简答）→ 综合应用（大题）。
- **语言类学科**：出题先补材料——阅读/听力必须带原文，翻译带待译文本，写作带参考范文；每条外部素材标注 `source`。
- **素材补全顺序**：用户资料 → MCP 外部工具 → 联网搜索 → AI 原创（标注模拟）。
- **内容量大时**：分片构建 review.json（每片 `--syntax-only` 校验 → 合并 → 全量校验），避免单次写入超限。
- **出题后必自检**（5.4 节）：计算类答案逐题重算，草稿标记清理，答案修复后重新生成 docx/HTML。
- **交付前**：必跑 `check_review.py`；交付物列在对话末尾（Word/HTML/PNG/试卷）。

## 数据与隐私边界（v3.4 新增）

- 本 Skill **全部本地处理**：提取、校验、生成均在用户机器完成，不将用户资料上传至任何未声明的外部服务；
- 联网搜索补全素材（第 1.5 步）仅发送**检索关键词**，不发送用户资料原文；命中内容经平台授权的 webfetch 抓取并标注 `source`；
- `llm_exam.py` 仅在用户**显式配置 API Key** 时调用外部模型：密钥优先读环境变量，命令行参数兜底且不落盘明文；未配置自动跳过；
- 中间产物（提取文本/校验中间/临时 JSON）统一放 `.temp/`，交付物放用户工作目录；任务后一次性中间产物可清理；
- **源文件个人信息（v3.5）**：真题/笔记/讲义可能含考生姓名、学号、手机号、身份证号、邮箱等个人信息。`check_review.py` 会自动扫描并以 WARN 提示（报告中已脱敏预览），交付前与用户确认是否需要脱敏；脚本不擅自改动原文内容，避免破坏题目。

## References

- 详细 JSON 结构（含 `mock_exams` 试卷规范）见 [references/review-schema.md](references/review-schema.md)
- 端到端示例数据集（可直接运行验证流水线）见 [references/examples/mini_review.json](references/examples/mini_review.json) 与 [references/examples/README.md](references/examples/README.md)；企业内训/培训考核场景专属示例见 [references/examples/training_review.json](references/examples/training_review.json)
- 版本迭代记录与改进说明见 [references/VERSION.md](references/VERSION.md)

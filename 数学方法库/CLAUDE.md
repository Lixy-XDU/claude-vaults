# CLAUDE.md — 数学方法库项目约束

本文件是数学方法库（Obsidian Vault）项目级约束，与全局 `~/.claude/CLAUDE.md` 互补。冲突时以全局文件为准，除非本文件明确覆盖。

---

## 版本与自动更新

本文件及配套配置（`数学方法库模板.md`、`数学方法库-AI扩展提示词.md`）托管于 GitHub：

```text
github.com:Lixy-XDU/claude-vaults
```

- **本机为全网核心**：所有 vault 配置的更新维护以本机为准（`c:\Users\81402\Desktop\数学方法\`），向外推送至 GitHub。
- **其他机器**：在会话启动时，通过 `git ls-remote` 检查远端仓库是否有新提交。若有更新，询问用户是否同步；无更新则静默跳过。
- **修改后必须同步**：每次修改本项目 CLAUDE.md、模板或扩展提示词后，须在 `D:\git仓库\claude-vaults\` 中 `git add -A && git commit` 提交；**推送前必须向用户确认，得到授权后再 `git push`** 至 GitHub。
- **分支策略**：禁止直接向 `master` 提交。每次修改从 `master` 拉出新分支（`feat/`、`fix/`、`chore/`），完成后合并回 `master` 再推送。
- **同步前须按全局 CLAUDE.md 的脱密处理清单逐项检查**，确保无用户名、绝对路径、API 密钥等敏感信息。

---

## markdown-to-html 输出路径

**硬性规则：所有 `.md` → `.html` 转换产物只能出现在 `html/` 子文件夹下，严禁 HTML 文件散落在其他任何目录中。**

调用 `/markdown-to-html` 转换本库中的 `.md` 文件时：

1. 必须使用 `--out html` 参数，直接将 HTML 输出到 `html/` 子文件夹（无论 md 在哪个子目录）
2. 示例：`npx -y bun main.ts "理论工具/布朗桥.md" --theme default --out html`
3. 每次转换完成后，必须确认输出路径为 `html/<文件名>.html`，不得落在 `方法库/`、`理论工具/`、`文献笔记/` 等源文件目录
4. 转换脚本已自动注入 MathJax v3（`$...$` 行内 + `$$...$$` 块公式），严禁手动再次注入，防止重复或截断

示例：转换 `K-S检验.md` 后，HTML 应位于 `html/K-S检验.html` 且公式可渲染。
5. **转换前预处理**：baoyu 转换器会对 `$...$` 内的 `*` 和 `_` 误解析为 Markdown 斜体标记。转换前须将行内数学模式中的 `^*` 替换为 `^{\ast}`、`_*` 替换为 `_{\ast}`，避免 HTML 标签污染数学块。转换后还须用脚本剥离 `$$` 和 `$` 内残留的 `<br>` 等 HTML 标签

---

## Obsidian 双链规则

生成任何 Obsidian 笔记中的 `[[双链]]` 时必须遵守：

1. **文件名完全匹配**：`[[目标]]` 必须与目标文件的实际文件名（不含 `.md`）完全一致，否则 Obsidian 会创建空文件
2. **完整含英文名**：本库文件名统一格式为 `<中文名>（<English Name>）.md`（如 `ADMM（Alternating Direction Method of Multipliers，交替方向乘子法）.md`），双链必须含括号内的英文名
3. **跨子目录加路径**：引用 `理论工具/` 子文件夹中的文件时，格式为 `[[理论工具/完整文件名|显示别名]]`
4. **写链前先确认**：生成双链前必须先确认目标文件在磁盘上的实际名称，**禁止**用简写假设文件名
5. **表格内双链转义**：在 Markdown 表格中使用 `[[...|别名]]` 时，管道符 `|` 会被解析为列分隔符。改为 `[[...\|别名]]` 转义
6. **理论工具互链检查**：理论工具条目之间若存在"互为表里"或"同一定理的不同版本"关系（如 Carathéodory-Fejér 定理与 Vandermonde 分解），必须在各自的「被哪些方法依赖」表中添加互相引用。新建或修改理论工具后，**务必将同目录下其他理论工具的双链也一并补齐**——理论工具往往是彼此的数学前提或等价表述，单向链接不足以保证可追溯性

---

## Obsidian 写入模板

AI 生成的笔记默认只写入用户指定的收件箱路径（会话中询问），禁止直接写入正式分类目录。

凡是往 Obsidian 写 Markdown，遵守此 front-matter 骨架：

```yaml
---
title: <标题>
type: literature | method | project | idea | daily
tags: [auto-generated, review-needed]
created: <YYYY-MM-DD>
source: <文献 / 代码 / 会话链接>
status: draft
---
```

- `tags` 必含 `review-needed`，提醒 AI 生成内容需人工审核
- 内容末尾保留 `## 🔗 相关` 区域，列出候选双链（`[[...]]`），不替用户决定最终链接结构

---

## 条目组织规则

- **独立方法**（算法型/概念型）：放在 `方法库/` 子文件夹，按 16 节模板完整撰写
- **理论工具**（定理/引理级工具，如 Carathéodory-Fejér 定理、Vandermonde 分解）：放入 `理论工具/` 子文件夹，写精简版（含定义、角色说明、被哪些方法依赖）
- **交叉引用**：
  - 引用理论工具：`[[理论工具/完整文件名|显示别名]]`
  - 引用独立方法（从其他目录）：`[[方法库/完整文件名|显示别名]]`
  - 同一目录内互相引用：`[[完整文件名|显示别名]]`
- **不跳过理论工具**：论文若依赖某定理完成核心证明，该定理应入库，不能只提不建
- **`方法库/` 和 `文献笔记/` 的路径交由 `/ml-research-coder` 管理**：当任务涉及从这两个目录读取笔记并转为 MATLAB 代码实现或论文复现时，委托给该 skill 处理。本文件不再重复声明这两个目录的路径约定

---

## matlab-research-coder 代码输出路径

**硬性规则：`/ml-research-coder` 生成的 MATLAB 代码一律输出到 `code-workspace/` 子文件夹下。**

1. 方法实现（Workflow A）→ `code-workspace/<方法名>/`
2. 论文复现（Workflow B）→ `code-workspace/<论文简称>-reproduction/`
3. 每次生成后确认输出落在 `code-workspace/` 内，不得散落到笔记目录

---

## paper-note-to-ppt-template 输入输出路径

**硬性规则：`/paper-note-to-ppt-template` 的输入只能来自本库内部目录，输出一律落在 `ppt/<任务缩写>/` 下。**

调用 `/paper-note-to-ppt-template` 时：

1. **笔记输入**：从 `文献笔记/` 子文件夹读取论文阅读笔记（`.md`），作为内容计划的主要来源
2. **PDF 输入**：从 `paper/` 子文件夹读取原始 PDF 论文源文件，用于图表提取；若笔记中声明的 PDF 路径不在 `paper/` 下，应先提示用户将 PDF 复制到 `paper/` 中
3. **任务子文件夹命名**：每次任务在 `ppt/` 下新建子文件夹，命名规则为 `<论文简称>-<场景>`，使用中文（如 `PCL-PET异构定位-组会`、`线谱估计-journal-club`），由 Claude 根据论文和场景自动拟定，首次执行时向用户确认
4. **输出路径**：所有产物（`deck.pptx`、`content_plan.json`、`layout_plan.json`、`speaker_notes.md` 等）一律写入 `ppt/<任务缩写>/`
5. **公式与图表资产**：公式 PNG 写入 `ppt/<任务缩写>/assets/formulas/`，提取的图表写入 `ppt/<任务缩写>/assets/figures/`
6. 每次生成完成后，确认输出文件均落在 `ppt/<任务缩写>/` 内，不得散落到 `文献笔记/`、`方法库/` 等源文件目录

### PPT 项目输出目录结构

```text
<任务缩写>/
├── clean_notes.md          # 论文笔记
├── deck-template.pptx      # 最终 PPT（从 outputs/ 复制到顶层方便打开）
├── source_map.md           # 内容 → 论文来源映射
├── speaker_notes.md        # 讲稿
├── assets/
│   ├── figures/            # 论文原图
│   └── formulas/           # 公式渲染 PNG
└── outputs/
    └── paper-note-to-ppt-template/
        ├── deck.pptx
        ├── content_plan.json
        ├── layout_plan.json
        ├── validation_report.json
        ├── final_report.md
        ├── formula_report.json
        └── assets/
            ├── figures/    # 原图副本（渲染时需要）
            └── formulas/
```

顶层只保留 5 类文件，旧 skill 的中间产物（`content_plan.json`、`layout_plan.json` 等）不留在顶层。

---

## MATLAB 项目编码规范

以下规范适用于 `code-workspace/` 下所有 MATLAB 项目，`/ml-research-coder` 生成代码时默认遵循。

### 1. 目录结构：不超过两层

```text
<project>/
├── <ModuleName>.m      # 核心类/模块，直接放项目根
├── <algorithm>.m       # 算法函数，也放项目根
├── tests/              # 测试
├── examples/           # 示例
├── docs/               # 文档
├── data/               # 数据
├── results/            # 输出
└── README.md
```

- **禁止**出现 `src/core/`、`src/algorithms/`、`src/utils/` 等三层以上嵌套
- 核心代码直接放项目根目录，仅 `tests/`、`examples/`、`docs/`、`data/`、`results/` 可建子目录

### 2. 文件粒度：变体算法合并

- 算法变体（如和-积/最大-积、批量GD/随机GD）**合并到一个文件**，用参数或 mode 字符串区分
- 规则：迭代框架相同、仅内部某步操作不同 → 合并，不要为每个微小变体单独建文件

### 3. 工具函数：优先内联

- 少于 50 行且**仅被一个文件调用**的工具函数，内联为该文件的局部函数
- 单独建文件的条件：被 ≥2 个文件调用，或超过 80 行
- 不要为 `enumerateConfigs`、`checkConvergence` 这类小工具单独建文件

### 4. 参数校验：从简

- 科研代码**不用** `arguments` 块做详细校验
- 用 `inputParser` 或 `if nargin < 2, param = default; end` 即可

### 5. 可视化：克制

- 可视化函数可独立建文件（通常较长、可能被多文件调用）
- 一个项目最多 1 个可视化文件

### 检查清单

新建或重构 MATLAB 项目后检查：
- [ ] 核心 `.m` 文件 ≤ 5 个（小型 ≤3）
- [ ] 目录不超过两层
- [ ] 无单调用方的独立工具文件
- [ ] 无 `arguments` 块防御式校验

---

## 目录结构说明

本库各子目录用途如下，新增目录须在此登记：

| 目录 | 用途 | 内容说明 |
| ---- | ---- | ------- |
| `方法库/` | 独立数学方法条目 | 按 16 节模板撰写的完整方法笔记（算法型/概念型） |
| `理论工具/` | 定理/引理级工具 | 精简版笔记，含定义、角色说明、被哪些方法依赖 |
| `文献笔记/` | 论文阅读笔记 | `/literature-paper-reading` 产出的结构化阅读笔记 |
| `html/` | Markdown→HTML 导出 | `/baoyu-markdown-to-html` 转换产物，可随时从 `.md` 重新生成 |
| `paper/` | 原始 PDF 论文源文件 | 待处理/已处理的论文 PDF，供 `/pdf-extract` 读取；处理完成后保留源文件备查 |
| `_extracted/` | PDF 文本提取中间产物 | `/pdf-extract` 输出的 `.txt` + `.meta.json`，为后续阅读/提炼提供文本输入；可随时从 PDF 重新生成 |
| `code-workspace/` | MATLAB 代码工作区 | `/ml-research-coder` 生成的 MATLAB 项目代码，按方法名或论文简称分子目录 |
| `ppt/` | 论文转 PPT 输出 | `/paper-note-to-ppt-template` 的生成产物，每次任务在 `ppt/<任务缩写>/` 下建独立子文件夹，含 `assets/formulas/` 和 `assets/figures/` |

根目录仅存放：

- `CLAUDE.md`（项目约束）
- `数学方法库模板.md`（16 节模板）
- `数学方法库-AI扩展提示词.md`（AI 生成规范）

禁止在根目录或子目录中存放 `.bak` 备份文件、第三方工具配置（`.codebuddy/`、`.workbuddy/` 等）、或与方法库无关的文档。

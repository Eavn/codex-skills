---
name: 归档
description: 当用户明确要求“归档”“整理上下文并归档”“提取本次任务有用内容写入记忆”“写归档笔记”“收尾归档”“archive this session”或把本次任务沉淀为长期记录时使用。作为唯一用户入口/facade，一次完成 Obsidian engineering note、agent-brain wiki、必要 ad-hoc memory note，并自动做轻量相关旧记忆检查；不要因普通复杂任务自动触发。若用户只要求整理/清理/压缩旧记忆，改用“记忆治理”。
---

# 归档 Skill

## 目标

把一次任务、排障、发布、数据清理、设计/实现收口或会话上下文，整理成可复用、可审计、低噪音的归档产物。

本 skill 是用户触发归档时的**唯一入口 / facade**：用户只需要说一次“归档”，本 skill 就负责完成本次归档，并在内部执行最小必要的旧记忆检查。AGENTS、普通记忆、项目笔记只保留事实和结论，不维护归档工作流细则。

## 触发边界

仅在用户明确表达以下意图时使用：

- “归档”
- “整理上下文并归档”
- “提取本次任务有用内容写入记忆”
- “写归档笔记”
- “收尾归档”
- “archive this session”
- 用户直接要求把某次任务沉淀为长期记录

不要因为任务复杂、多步、部署、数据库、测试、规约、并发或排障自动触发归档。没有用户明确归档意图时，只正常完成当前任务。

如果用户的主要目标是“整理旧记忆 / 清理记忆 / 压缩记忆 / shrink MEMORY.md / 删除过期记忆 / 记忆治理”，而不是归档本次任务，使用 `记忆治理` skill，不要套用本归档流程。

## 核心原则

1. **一次入口**：不要让用户先说“归档”再说“记忆治理”。归档内部自动完成轻量相关旧记忆检查。
2. **职责分层**：归档负责沉淀本次任务；`记忆治理` 负责大范围旧记忆清理、冲突消解、重复规则压缩和 shrink 方案。
3. **事实优先**：只归档已发生、已验证、可复验的事实；推测必须标注为推测。
4. **低噪音**：只沉淀未来会复用的结论、命令、路径、风险、验收信号和后续边界；删除流水账。
5. **单一入口**：归档流程只写在本 skill；不要把归档步骤复制进 AGENTS、普通记忆或其他 skill。
6. **分层存放**：
   - Obsidian engineering note：放完整中文复盘、操作记录、验证证据，默认仍写入 `/mnt/c/Users/hardy/Documents/Obsidian Vault/engineering/`。
   - LLM Wiki / agent-brain：默认同时调用 wiki 系列技能语义，把可复用知识沉淀到 `OBSIDIAN_VAULT_PATH` 指向的 wiki（当前为 `/mnt/c/Users/hardy/Documents/Obsidian Vault/agent-brain`）；优先写摘要/决策/规则，不写聊天流水账。
   - ad-hoc memory note：只放需要进入 Codex 长期记忆索引的短规则/事实/修正请求。
   - 仓库文档/AGENTS：仅在用户明确要求时更新，且只写稳定事实，不写归档流程。
7. **不直接改 MEMORY.md**：记忆系统要求通过 `extensions/ad_hoc/notes/` 写新增/修正/删除请求；不要直接编辑 `MEMORY.md` 或历史 rollout summary。
8. **不隐式治理**：归档不是任务规约评估；不要自动调用“任务规约治理”或生成合规评分，除非用户同时明确点名对应 skill。

## 归档流程

### 1. 定义归档范围

先从当前会话和用户最近指令中确定：

- 归档对象：代码变更、线上排障、数据库清理、发布、设计决策、会话行为、记忆治理等。
- 时间边界：本轮、本日、某个 session id、某个提交、某个线上事件。
- 需要保留的证据：命令、SQL、测试、HTTP 返回、提交 hash、PID、日志路径、文件路径、截图/设计产物路径。

边界不清时，优先做合理最小归档；只有在会误写长期记忆或误删内容时才追问。

### 2. 提取可长期复用内容

归档内容按以下优先级筛选：

1. 已验证的最终结论。
2. 关键路径、命令、SQL、接口、文件、提交、部署/回滚方式。
3. 下次遇到同类问题时应优先检查的信号。
4. 明确废弃、冲突、过期或需要修正的旧记忆/旧文档。
5. 未完成事项和风险边界。

不要归档：

- 普通中间猜测。
- 已被最终结论推翻的临时方案，除非它本身是重要坑点。
- 大段聊天原文。
- 敏感凭据、token、密码、完整私密配置。

### 3. 写 Obsidian 归档笔记

> 归档时默认还要沉淀一份到 LLM Wiki / agent-brain。Obsidian engineering note 负责完整复盘，agent-brain wiki 负责可查询的长期知识页。两者都在同一个 Obsidian Vault 下，但目录和用途不同。

归档笔记默认必须写入用户的 Obsidian Vault，而不是仓库内 `obs/` 临时目录；除非用户明确要求仓库内同步副本，否则不要把归档笔记落到 repo `obs/`。

默认落点：

- Vault 根路径：`/mnt/c/Users/hardy/Documents/Obsidian Vault`
- 归档目录：`engineering/`
- 相对路径格式：`engineering/YYYY-MM-DD-中文归档标题.md`

写入方式：

- 优先调用现有 Obsidian 写入 CLI/脚本，不要手写复制到仓库目录。
- SmartQuiz 仓库内可复用命令：

```bash
python3 .codex/skills/obs-log/scripts/write_note.py \
  --relative-path "engineering/YYYY-MM-DD-中文归档标题.md" \
  --title "中文归档标题" \
  --body-file /tmp/archive-note.md \
  --overwrite
```

- 如果当前工作目录没有 `.codex/skills/obs-log/scripts/write_note.py`，先查找可用的 `obs-log` skill 脚本或用户环境中的 Obsidian CLI；仍要写到 Vault 的 `engineering/` 相对路径。
- 只有当用户明确指定其他目录时，才改写 `--relative-path`。

推荐笔记结构：

```markdown
# 标题

## 背景
- 用户目标：
- 时间边界：

## 最终结论
- 

## 关键证据
- 命令/接口/SQL/测试：
- 产物路径/提交/PID/日志：

## 决策与边界
- 

## 后续建议
- 

## 可复用规则
- 
```

### 4. 写 LLM Wiki / agent-brain 条目（默认执行）

归档完成 Obsidian engineering note 后，默认把本次归档中的“未来会复用的知识”同步进 wiki。当前 wiki 配置由 `~/.obsidian-wiki/config` 决定，默认应为：

`OBSIDIAN_VAULT_PATH="/mnt/c/Users/hardy/Documents/Obsidian Vault/agent-brain"`

执行原则：

1. **优先使用 wiki skill 语义**：如果当前任务是“保存本次会话/结论”，按 `wiki-capture` 的方式写；如果是“同步当前项目知识”，按 `wiki-update` 的方式写；如果是“从 Codex 历史批量挖掘”，按 `codex-history-ingest` 的方式写。
2. **不要全量加载 wiki**：只读取 `$OBSIDIAN_VAULT_PATH/hot.md`、`index.md` 和可能相关的少数页面；不要因为归档而扫描整个 vault。
3. **不要复制归档全文**：wiki 页面写 declarative knowledge、决策、规则、流程、坑点、路径和引用；完整过程仍留在 engineering note。
4. **保持 provenance**：新/改页面必须带 `summary`、`provenance`、`base_confidence`、`lifecycle: draft`、`lifecycle_changed`，并用 `^[extracted]` / `^[inferred]` / `^[ambiguous]` 标注关键 claim。
5. **更新 wiki 特殊文件**：如果写入或修改 wiki 页面，同步更新 `.manifest.json`、`index.md`、`hot.md`、`log.md`；若 `QMD_WIKI_COLLECTION` 未设置，记录 `QMD skipped: QMD_WIKI_COLLECTION unset`。
6. **可跳过条件**：用户明确说“不写 wiki / 只写 engineering 归档”时跳过；如果本次归档完全没有可长期复用知识，也可以跳过并在回报里说明。

### 5. 写 ad-hoc memory note（仅必要时）

当归档中包含未来需要全局检索的稳定事实、偏好、轻量修正请求或路由修正时，写一个短 note 到：

`/home/hardy/.codex/memories/extensions/ad_hoc/notes/`

要求：

- 文件名：`<YYYYMMDD-HHMMSS>-<short-slug>.md`
- 只写一件事。
- 明确是新增、修正还是删除请求。
- 删除旧记忆时列出精确文件/行号/主题；不要直接删 `MEMORY.md`。
- 如果只是一次性任务结果，不需要进长期记忆，就不要写 memory note。

### 6. 轻量相关旧记忆检查（归档内置）

归档默认做**轻量**旧记忆检查，目的是避免本次归档刚沉淀的稳定事实立刻被同主题旧记忆误导；不要把它升级成全量记忆治理。

执行方式：

1. **限定范围**：只围绕本次归档关键词、仓库、模块、提交、线上事件或 session id 检索 `MEMORY.md` 和少数明确相关 ad-hoc note；不要默认全量扫所有历史。
2. **最小分类**：只标记三类：
   - `keep`：旧事实仍正确或只是历史证据。
   - `supersede`：旧事实已被本次归档取代，但保留历史价值。
   - `fix-request`：旧事实会误导未来路由/操作，需要写 ad-hoc 修正或删除请求。
3. **最小写入**：只有发现明确冲突、过期 latest/current、错误路由或重复高噪音规则时，才写 ad-hoc 修正/删除请求。
4. **不直接治理**：不要直接编辑 `MEMORY.md`、历史 rollout summary 或已生成的记忆索引；不要在普通归档里做大规模 shrink、去重、重写 taxonomy。
5. **自动升级条件**：如果发现大量冲突、重复规则、过期 latest 状态、跨项目系统性污染，停止在轻量检查边界，读取并使用 `记忆治理` skill；若用户只是要求普通归档，则记录“需要完整记忆治理”的待办/建议，不要求用户再输入一次。

### 7. 记忆治理边界

`记忆治理` 是独立能力，但不是第二个用户入口要求。归档内只在必要时调用它的轻量/建议模式。

- 普通归档：只执行 Step 6 的轻量相关检查。
- 归档中发现明确冲突：写最小 ad-hoc 修正/删除请求。
- 归档中发现系统性问题：在最终回报中列出“建议后续完整记忆治理”的范围；如果用户本次已经明确要求“整理旧记忆/清理所有记忆/shrink”，直接转入 `记忆治理` 完整流程。
- 用户单独说“整理旧记忆 / 清理记忆 / 压缩记忆”：不要走归档，直接使用 `记忆治理`。

### 8. 登记归档流程的散落来源冲突（仅发现明确冲突时）

如果在轻量检查中发现 AGENTS、普通记忆、旧 skill、legacy prompt 中仍有“归档流程步骤/强制归档规则/自动归档触发”，归档内只做**登记和最小修正请求**，不要直接展开来源清理。

处理方式：

1. AGENTS / 旧 skill / legacy prompt：记录冲突文件、关键词、为什么会误导归档路由；不要在普通归档中直接改写或删除这些来源，除非用户本次明确要求编辑该文件。
2. 记忆：如该冲突会影响未来 session，写一条最小 ad-hoc 修正/删除请求；不要直接编辑记忆主文件。
3. 升级边界：如果冲突来源超过一个文件、涉及多个项目/agent、或需要实际删除/迁移旧 skill/prompt，停止在登记边界并转入或建议 `记忆治理`。

### 9. 验证与回报

收尾时至少回报：

- 写了哪些 Obsidian engineering 归档文件。
- 写了哪些 LLM Wiki / agent-brain 页面，或为何跳过 wiki 写入。
- 写了哪些 memory note 或轻量修正/删除请求。
- 轻量旧记忆检查范围，以及 keep / supersede / fix-request 结论。
- 是否触发或建议完整 `记忆治理`。
- 是否更新了仓库文档/AGENTS（默认不更新）。
- 有没有未处理的旧来源或需要用户确认的删除项。

## 禁止事项

- 不要把归档流程复制到 AGENTS 或普通记忆中。
- 不要直接编辑 `/home/hardy/.codex/memories/MEMORY.md`。
- 不要把敏感凭据写入笔记或记忆。
- 不要把归档当成规约评估。
- 不要因为用户只说“继续”而自动归档，除非前文明确正在执行归档。
- 不要要求用户为了完成普通归档再输入一次“记忆治理”。
- 不要在普通归档中做全量记忆 shrink；完整治理交给 `记忆治理`。
- 不要提交、推送或部署，除非用户明确要求。

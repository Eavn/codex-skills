---
name: 记忆治理
description: 当用户明确要求“整理旧记忆”“清理记忆”“删除过期记忆”“压缩记忆”“shrink MEMORY.md”“memory cleanup”“memory governance”“去重记忆”“修正旧记忆”“清理旧 skill/旧 prompt 记忆污染”，或归档流程发现大量冲突/重复/过期记忆需要完整治理时使用。负责审计 Codex memory/ad-hoc notes/相关旧来源并写新增、修正、删除请求；不要直接编辑 MEMORY.md 或历史 rollout summary。
---

# 记忆治理 Skill

## 目标

把旧记忆、冲突记忆、重复规则、过期 latest/current 状态和散落旧来源治理成低噪音、可检索、不会误导未来 session 的状态。

本 skill 是“记忆治理”的 source of truth；`归档` 只是用户入口/facade，并在普通归档内做轻量相关检查。完整记忆治理由本 skill 负责。

## 触发边界

使用本 skill 的情况：

- 用户明确要求整理、清理、删除、压缩、去重、修正旧记忆。
- 用户提到 `MEMORY.md` 膨胀、memory_summary 噪音、shrink、stale memory、memory cleanup/governance。
- 用户要求清理旧 skill、legacy prompt、AGENTS 中残留的过期规则。
- `归档` 过程中发现大量冲突、重复规则、过期 latest 状态或跨项目系统性污染，需要超出轻量检查。

不要因为普通任务复杂、部署、数据库、排障、并发或代码审查自动触发本 skill。没有明确治理目标时，保持正常任务路径。

## 核心原则

1. **主文件只读**：不要直接编辑 `/home/hardy/.codex/memories/MEMORY.md`、`memory_summary.md`、历史 rollout summary 或自动生成索引。
2. **通过 ad-hoc 请求治理**：新增、修正、删除都写到 `/home/hardy/.codex/memories/extensions/ad_hoc/notes/`，交给记忆合并器/后续清理流程处理。
3. **精确范围优先**：先按用户给出的项目、主题、路径、时间或 session id 限定范围；除非用户明确要求“所有记忆”，不要全量扫描。
4. **保留历史，降噪当前**：旧事实若有审计价值，标记为 historical/superseded，而不是无脑删除。
5. **删除请求要可执行**：列出目标文件、行号/主题、建议动作和理由；不要写“清理一下旧记忆”这种不可执行请求。
6. **避免重复请求**：如果同主题删除/修正请求已经存在且尚未被合并器处理，不要再写一条重复 note；更新治理报告或写 shrink plan 指向既有请求。
7. **敏感信息保护**：不要把 token、密码、完整私密配置写入治理 note。
8. **区分知识与状态**：Obsidian/wiki 负责知识层；`.omx/state`、`.omc/state`、`.omo`、OMP backend 等运行时状态不要混进 Codex memory 清理，除非用户明确要求。

## 治理流程

### 1. 定义治理范围

从用户请求或归档上游传入信息中确定：

- 主题：例如 SmartQuiz、OCR、Sub2API、archive skill、provider、某个 release。
- 路径：例如 `/home/hardy/code/smart-quiz`、`/home/hardy/.codex/memories`。
- 时间：某天、某个 rollout、某个 ad-hoc note。
- 问题类型：过期 latest、重复规则、错误路由、冲突事实、legacy skill 污染、摘要噪音。

范围不清但风险低时，先做关键词限定的只读审计；只有在可能误删或写入长期删除请求时才追问。

### 2. 只读检索证据

优先检索：

1. `/home/hardy/.codex/memories/MEMORY.md`
2. `/home/hardy/.codex/memories/memory_summary.md`
3. 明确相关的 `rollout_summaries/*.md`
4. 明确相关的 `extensions/ad_hoc/notes/*.md`
5. 用户点名的 AGENTS、skill、legacy prompt 或项目文档

避免默认全量打开所有 rollout；先用 `grep`/`rg` 定位，再读少数相关文件。

### 3. 分类发现

把每条发现归到以下类别：

- `keep`：仍正确，保留。
- `historical`：已不是当前状态，但作为历史证据有价值。
- `supersede`：被较新事实取代，应在摘要/active memory 中降权。
- `merge`：重复内容应合并成一条稳定规则。
- `fix`：错误、冲突、会误导当前行为，需要修正。
- `delete-request`：无用、过期、污染路由或包含不该保留内容，需要删除请求。
- `needs-human`：删除可能影响审计、凭据、安全或跨工具行为，需要用户确认。

### 4. 写 ad-hoc 治理 note

若需要新增、修正或删除请求，写入：

`/home/hardy/.codex/memories/extensions/ad_hoc/notes/<YYYYMMDD-HHMMSS>-<short-slug>.md`

每个 note 尽量只处理一个主题。推荐结构：

```markdown
# 记忆治理请求：<主题>

- 日期：YYYY-MM-DD
- 类型：add / update / delete-request / merge-request / shrink-request
- 范围：<文件/主题/关键词>

## 当前结论
- 

## 证据
- `MEMORY.md:line-line` — 
- `extensions/ad_hoc/notes/...` — 

## 建议处理
- keep：
- historical/supersede：
- merge：
- fix：
- delete-request：

## 期望合并后行为
- 
```

删除/修正请求必须可执行：给出文件、行号或稳定主题名；如果行号可能漂移，同时给关键词。

### 5. 合并器不可用或请求堆积时

如果发现同主题 ad-hoc 删除/修正请求已经存在、`MEMORY.md` 仍未变化、或无法确认记忆合并器会及时执行：

1. **不要重复写同主题删除请求**：引用既有 note 路径和主题，说明仍待合并。
2. **输出 shrink plan**：把待处理项合并成一份可执行计划，列出优先级、目标主题、建议删除/合并动作和证据锚点。
3. **只在有新增事实时写新 note**：新 note 应说明它补充了什么，不要只是重复旧请求。
4. **回报阻塞点**：明确说明“ad-hoc 请求已写但主索引未收缩”，不要声称 MEMORY.md 已被清理。

### 6. 可选：wiki/engineering 记录

记忆治理通常只写 ad-hoc 请求即可。只有满足以下条件才写 Obsidian/wiki：

- 用户要求治理复盘或长期方法论。
- 治理结果形成跨 agent / 跨项目的稳定知识。
- 本轮治理改变了归档、wiki、memory 架构边界。

如果写 wiki/engineering，遵循对应 wiki/归档 skill；不要把普通删除清单复制成长文档。

### 7. 验证与回报

收尾至少回报：

- 审计范围：检索了哪些文件/关键词。
- 分类结果：keep / historical / supersede / merge / fix / delete-request / needs-human 的数量或要点。
- 写了哪些 ad-hoc governance note。
- 没有直接编辑哪些自动生成文件（尤其 `MEMORY.md`）。
- 哪些内容仍需人工确认或后续合并器执行。

## 与其他 skill 的边界

- `归档`：用户说“归档”时仍是唯一入口；归档内部只做轻量相关旧记忆检查，发现大问题才调用/建议本 skill。
- `note`：写 `.omx/notepad.md`，用于当前 OMX 工作区/会话续航和 compaction 防丢；不是 Codex 长期 memory 治理入口。
- `wiki-*`：负责 Obsidian/agent-brain 知识页；本 skill 只在治理结果需要长期知识化时调用。
- `任务规约治理`：只处理用户明确点名的任务规约评估；不要因记忆治理自动触发。

## 禁止事项

- 不要直接修改 `MEMORY.md`、`memory_summary.md` 或历史 rollout summary。
- 不要删除历史 session / rollout 原始日志。
- 不要把所有旧事实都删除；先区分 historical 和 active-current。
- 不要把凭据、token、密码写入治理 note。
- 不要把记忆治理流程复制到 AGENTS 或普通记忆中。

# KCD Codex Skills — 上下文归档与记忆治理套件

> **本套件专为 [Codex / OMX](https://github.com/openai/codex) 设计，需配合 Codex 的 Skill 系统与 Memory 体系使用。**

## 功能概述

本仓库提供两套 **Codex Skill**，用于解决 AI 编程助手在长时间、多任务会话中产生的**上下文膨胀**与**记忆噪音**问题：

| Skill | 用途 | 触发时机 |
|-------|------|----------|
| **归档** | 将单次任务/排障/会话的上下文沉淀为可复用的长期记录 | 用户明确说“归档”“收尾归档”“archive this session” |
| **记忆治理** | 清理、压缩、去重旧记忆，修正过期规则与冲突事实 | 用户明确要求“整理旧记忆”“清理记忆”“shrink MEMORY.md” |

## 为什么需要 Codex / OMX

Codex（及 OMX 工作流）在持续运行时会产生两类负担：

1. **上下文膨胀**：多轮任务后，当前会话携带大量历史中间猜测、排障日志，导致后续推理 token 浪费、路由混乱。
2. **记忆噪音**：`MEMORY.md` 与 ad-hoc notes 长期累积，过期规则、重复技能、冲突事实混入长期记忆，误导新会话行为。

**本套件不提供独立 CLI 工具**，而是作为 **Codex Skill 指令**注入到 Codex 的系统提示中，由 Codex 自身在适当时机调用执行。Skill 的设计完全遵循 Codex Memory 架构（`MEMORY.md` + `extensions/ad_hoc/notes/`），确保与原生记忆系统无缝兼容。

## 安装方式

### 1. 放入 Codex Skill 目录

将本仓库 clone 到 Codex 的 skills 目录：

```bash
# 默认路径
$HOME/.codex/skills/

# OMX 工作流路径
$PROJECT/.omx/skills/
```

目录结构示例：
```
.skills/
├── kcd-codex-skills/
│   ├── 归档/
│   │   └── SKILL.md
│   └── 记忆治理/
│       └── SKILL.md
```

### 2. 确认 Codex 已加载

在 Codex 会话中，Skill 的 `description` 会自动进入系统提示。你只需在需要时说：

- **归档当前任务**：→ 触发「归档」Skill，自动写入 Obsidian / Wiki / Memory
- **整理旧记忆**：→ 触发「记忆治理」Skill，自动审计并生成修正请求

## 核心设计

- **单一入口（Facade）**：用户只说一次“归档”，Skill 内部自动完成三层输出（Obsidian 笔记 + Wiki 知识页 + ad-hoc memory note），无需重复指令。
- **主文件只读**：所有删除/修正请求都写入 `extensions/ad_hoc/notes/`，不直接篡改 `MEMORY.md`，符合 Codex Memory 合约。
- **自动升级**：归档过程中若发现大量记忆冲突/重复，自动建议升级到「记忆治理」完整流程，不强迫用户二次输入。

## 文件说明

- `归档/SKILL.md` — 归档 Skill 源码，包含完整的触发边界、流程、分层存放规则与禁止事项。
- `记忆治理/SKILL.md` — 记忆治理 Skill 源码，包含 7 类分类（keep/historical/supersede/merge/fix/delete-request/needs-human）与治理流程。

## 配合 OMX 工作流

在 OMX 分层架构中，本套件主要作用于：

- **Planning 层** → 归档产生的决策记录与 PRD 收口知识
- **Execution 层** → 治理 `$ultrawork` / `$ultragoal` 多轮任务后的记忆残留
- **Verification 层** → 归档 `$code-review` / `$ultraqa` 的审计结论

通过 `omx state write/read` 与 `.omx/state` 文件解耦，Skill 只治理 Codex Memory，不侵入 OMX 运行时状态（除非用户明确要求）。

## 版本与更新

- 当前版本：v1.0（基于 Codex 归档 v3 与记忆治理 v1 重构）
- 更新方式：本仓库 `git pull` 后重启 Codex 会话即可热加载新 Skill 规则

---

**维护者**：kcd-dev
**许可证**：MIT

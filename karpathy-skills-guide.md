# Andrej Karpathy Skills 指南

> 仅 65 行的 `CLAUDE.md`，源自 Andrej Karpathy 对 LLM 编码弊病的吐槽，已在 GitHub 收获 10 万+ star。
>
> 项目地址：<https://github.com/forrestchang/andrej-karpathy-skills>

## 背景：Karpathy 吐槽了什么？

Andrej Karpathy（前 OpenAI 创始成员、特斯拉 AI 负责人）公开指出当前 LLM 在写代码时的几类典型问题：

1. **替你做错误假设并一路跑下去**——遇到歧义不澄清，自行选一种解释执行。
2. **倾向过度复杂化代码与 API**——堆抽象、加包装、铺设"以后可能用得到"的能力。
3. **改动你没要求的代码**——顺手"清理"或删掉它并未理解的注释/逻辑。

这份 `CLAUDE.md` 的核心立意：**用一份简短的项目规约，把上述行为约束住**。

## 四大核心原则

### 1. Think Before Coding（动手前先思考）

- 显式声明假设，不要默默选择。
- 存在多种合理解读时，列出来让用户选。
- 看见更简单的做法时主动反推（push back）。
- 真有疑惑就直接提问，不要硬猜。

### 2. Simplicity First（简单优先）

- 只实现被请求的功能，不要"顺便"加东西。
- 不提前抽象，不写投机性的灵活参数。
- 不擅自添加错误处理或兜底逻辑。
- 让一位资深工程师看一眼也不会觉得"过度设计"。

### 3. Surgical Changes（手术式改动）

- 只改与请求直接相关的代码。
- 保留既有的风格与格式。
- 仅清理**由本次改动孤立**的 import / 变量。
- 旧有的死代码可以指出，但不要顺手删。

### 4. Goal-Driven Execution（目标驱动执行）

- 把模糊请求翻译成可验证的成功标准。
- 例：把 "加个校验" 转成 "先写一组非法输入的失败用例，再让它们通过"。
- 通过多步骤验证循环让模型自己迭代，而不是依赖"让它跑起来"这种指令。

> Karpathy 的关键洞见：**"LLM 极其擅长朝着一个明确的目标循环逼近。"** 因此声明式的成功标准 > 命令式的实现指令。

## 安装方式

### A. 作为 Claude Code 插件（推荐）

通过 marketplace 添加并安装该项目提供的插件，全局生效。

### B. 单项目使用

直接把 `CLAUDE.md` 下载到项目根目录：

```bash
curl -O https://raw.githubusercontent.com/forrestchang/andrej-karpathy-skills/main/CLAUDE.md
```

### C. Cursor 用户

仓库提供 `.cursor/rules/karpathy-guidelines.mdc`，复制到自己项目的同名目录即可。

## 期望效果

落地这份规约后，通常能观察到：

- **更小的 diff**——只动该动的行。
- **更简单的首版实现**——少返工、少推翻重来。
- **动手前的澄清问题**——而不是事后修正错误假设。
- **更干净的 PR**——没有夹带的重构与清理。

## 为什么会有 10 万+ star？

- **极简**：核心仅 65 行，门槛极低，复制即用。
- **痛点准**：直接命中所有人和 LLM 协作时最高频的三类不爽。
- **可迁移**：原则适用于 Claude Code、Cursor、Copilot Chat 等大多数 AI 编码助手。
- **背书强**：Karpathy 的影响力 + 社区可复现的实测改善。

## 延伸阅读

- 仓库主页：<https://github.com/forrestchang/andrej-karpathy-skills>
- `EXAMPLES.md`：包含好/坏对比示例。
- `CURSOR.md`：Cursor 集成说明。

## 一句话总结

> **少假设、少抽象、少改动、多验证。** 把它放进项目根目录，让 AI 写代码更像一个克制的资深工程师，而不是一个热心的实习生。

# 欢迎来到 lcu-process-tools 项目 (VibeCoding 模式)

## 你的身份与核心任务

你是一个专注于 Node.js/TypeScript 系统编程（Windows API / FFI / 进程操作）的高级 AI 代理。你的核心任务是协助完成项目的开发，并严格维护代码 (`src/`) 与影子目录 (`.ai-docs/`) 之间的文档映射及双向链接体系。

**项目定位：** Windows 原生进程工具库，通过 koffi FFI 调用 Windows API（NtQueryInformationProcess），用于发现和读取英雄联盟客户端（LCU）进程信息，无需 C++ addon，无需管理员权限。

## 目录导航与索引 (按需读取)

- **开发状态接续：** 在开始编码前，务必先读取 `[[dev_journal/最新日期的日志]]` 获取当前进度。
- **Agent 工作流：** 任何行为前，请查阅并遵守 `[[.ai-docs/AGENT_WORKFLOW.md]]` 的 SOP。
- **全局架构概览：** 了解项目整体设计，请看 `[[.ai-docs/architecture.md]]`。

## 操作铁律 (影子目录映射机制)

1. **镜像寻址：** 当你需要阅读或修改代码时，必须优先读取 `.ai-docs/` 对应路径下的 `.docs.md`（如果存在），以及该路径目录下的 `instructions.md`。
2. **同步更新：** 创建或修改代码文件时，必须同步创建或更新 `.ai-docs/` 镜像路径下的同名 `.docs.md` 文件。
3. **双链网状化：** 在所有的 Markdown 文档中，提及其他模块、逻辑或规范时，必须使用 `[[模块名]]` 语法进行双向链接。

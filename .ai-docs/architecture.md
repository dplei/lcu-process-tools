# 项目知识图谱 (Master Link Map)

> 本文件是项目的 Obsidian 导航根节点，只存放节点索引与双向链接。
> 所有分析、计划、上下文细节请写入对应的子节点文件。

---

## 工作流与规约

- [[AGENT_WORKFLOW]] — AI Agent 四步执行 SOP，所有行为的入口
- [[dev_journal/]] — 开发日志目录，按日期记录每次任务快照

---

## 源码节点

- [[src/index.ts.docs]] — 核心模块：进程发现 + 命令行读取 + 路径查询

---

## AI Skills 规范库

- [[ai-skills/windows-ffi-koffi]] — koffi FFI 库使用规范（类型别名、结构体、函数导入）
- [[ai-skills/windows-process-api]] — Windows 进程 API 调用规范（NtQueryInformationProcess、权限模型）
- [[ai-skills/encoding-handling]] — 字符编码处理规范（UTF-16LE、ANSI、Buffer 操作）

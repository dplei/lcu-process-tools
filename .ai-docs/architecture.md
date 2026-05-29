# 🌐 项目全局架构与源码大地图

## 📁 物理目录快照 (生成时间: 2026-05-29)

```text
提示：未检测到 tree 命令，使用 find 导出资产清单：
dist/index.d.ts
dist/index.js
src/index.ts
```

## 🔍 项目架构诊断分析

### 项目概览

本项目是一个 **单文件 Node.js/TypeScript 工具库**，专注于 Windows 进程操作。核心实现集中在 `src/index.ts`（228 行），通过 koffi FFI 库调用 Windows API（kernel32.dll 和 ntdll.dll）。

### 技术栈与架构特征

**核心技术：**
- **FFI 层：** koffi（纯 JS FFI 库）用于调用 Windows 原生 API
- **目标 API：** NtQueryInformationProcess (ProcessCommandLineInformation = 60)
- **权限模型：** 仅需 PROCESS_QUERY_LIMITED_INFORMATION (0x1000)，无需管理员权限

**架构模式：**
- 单一职责：进程发现 + 命令行读取 + 路径查询
- 无依赖注入，直接调用 Windows API
- 导出三个公共函数：`getPidsByName`、`getCommandLine`、`getProcessImagePath`

### 热点模块优先级（懒加载逆向工程建议）

根据代码复杂度和业务关键性，建议按以下顺序进行文档化：

1. **🔥 优先级 1：`getCommandLine` 函数** (src/index.ts:152-200)
   - **原因：** 核心功能，涉及复杂的 Windows API 调用和 UNICODE_STRING 结构解析
   - **技术难点：** 两阶段查询（先获取长度，再读取数据）、UTF-16LE 编码处理
   - **文档需求：** 需详细说明 NtQueryInformationProcess 的调用约定和返回值结构

2. **🔥 优先级 2：FFI 类型定义与常量** (src/index.ts:17-99)
   - **原因：** 基础设施层，所有函数依赖这些定义
   - **技术难点：** Win32 类型映射、PROCESSENTRY32 结构体对齐
   - **文档需求：** 需建立 Skill 文档说明 koffi 类型系统和 Windows API 映射规范

3. **优先级 3：`getPidsByName` 和 `getProcessImagePath`** (src/index.ts:111-138, 208-227)
   - **原因：** 辅助功能，逻辑相对简单
   - **技术难点：** CreateToolhelp32Snapshot 快照遍历、字符串编码处理

### 待建立的 Skill 规范

根据代码分析，建议在 `.ai-docs/ai-skills/` 下创建以下规范文件：

1. **`windows-ffi-koffi.md`** - koffi FFI 库使用规范
   - 类型别名定义规范（koffi.alias）
   - 结构体定义规范（koffi.struct）
   - 函数导入规范（koffi.load + func）
   - 内存管理与 Buffer 操作

2. **`windows-process-api.md`** - Windows 进程 API 调用规范
   - NtQueryInformationProcess 调用模式
   - 权限标志选择（PROCESS_QUERY_LIMITED_INFORMATION）
   - 错误处理与资源清理（CloseHandle）

3. **`encoding-handling.md`** - 字符编码处理规范
   - UTF-16LE 与 UTF-8 转换
   - ANSI 字符串处理（PROCESSENTRY32.szExeFile）
   - Buffer 操作最佳实践

### 项目健康度评估

**优势：**
- ✅ 代码结构清晰，单一职责明确
- ✅ 无外部运行时依赖（除 koffi）
- ✅ 完善的错误处理（try-catch + null 返回）
- ✅ 资源管理严格（finally 块中 CloseHandle）

**改进空间：**
- ⚠️ 缺少单元测试（无 test/ 目录）
- ⚠️ 缺少示例代码（可在 examples/ 目录补充）
- ⚠️ 缺少内联文档（TSDoc 注释可以更详细）

### 下一步行动建议

1. **立即行动：** 为 `src/index.ts` 创建影子文档 `.ai-docs/src/index.ts.docs.md`
2. **短期目标：** 建立上述三个 Skill 规范文件
3. **中期目标：** 补充单元测试和使用示例
4. **长期目标：** 考虑支持更多进程操作 API（如环境变量读取）

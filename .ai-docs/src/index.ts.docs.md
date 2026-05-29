# src/index.ts — 影子文档

> 对应源文件：`src/index.ts`（228 行）
> 关联规范：[[windows-ffi-koffi]] · [[windows-process-api]] · [[encoding-handling]]

---

## 模块职责

Windows 原生进程工具库的唯一入口。通过 koffi FFI 调用 Windows API，实现三项能力：

1. 按名称查找进程 PID 列表
2. 读取进程完整命令行字符串
3. 读取进程可执行文件路径

**约束：** 仅支持 Windows x64，Windows 8.1+。无需管理员权限。

---

## 层次结构

```
src/index.ts
├── Win32 类型别名 (L22-26)          → [[windows-ffi-koffi#类型别名]]
├── kernel32.dll 函数导入 (L31-48)   → [[windows-ffi-koffi#函数导入]]
├── ntdll.dll 函数导入 (L53-57)      → [[windows-process-api#NtQueryInformationProcess]]
├── PROCESSENTRY32 结构体 (L66-77)   → [[windows-ffi-koffi#结构体定义]]
├── 常量定义 (L82-99)
├── getPidsByName() (L111-138)
├── getCommandLine() (L152-200)
└── getProcessImagePath() (L208-227)
```

---

## 公共 API

### `getPidsByName(exeName: string): number[]`

- **输入：** 可执行文件名，大小写不敏感（如 `"LeagueClientUx.exe"`）
- **输出：** 匹配的 PID 数组，无匹配时返回 `[]`
- **机制：** `CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS)` 遍历进程快照
- **编码：** `szExeFile` 为 ANSI `char[260]`，用 `utf8` 解码 → [[encoding-handling#ANSI字符串]]

### `getCommandLine(pid: number): string | null`

- **输入：** 目标进程 PID
- **输出：** 完整命令行字符串，失败返回 `null`
- **机制：** 两阶段 `NtQueryInformationProcess(ProcessCommandLineInformation=60)` → [[windows-process-api#两阶段查询模式]]
  1. 空 buffer 调用，从 `returnLength` 获取所需字节数
  2. 分配正确大小的 buffer，执行实际查询
- **结构解析：** 返回 buffer 头部为 `UNICODE_STRING`（offset 0-15），字符串数据从 offset 16 开始 → [[encoding-handling#UNICODE_STRING解析]]
- **权限：** 仅需 `PROCESS_QUERY_LIMITED_INFORMATION (0x1000)`，无需管理员

### `getProcessImagePath(pid: number): string | null`

- **输入：** 目标进程 PID
- **输出：** 可执行文件完整路径，失败返回 `null`
- **机制：** `QueryFullProcessImageNameW`，buffer 预分配 1024 字节（512 宽字符）
- **编码：** UTF-16LE，实际字符数由 `sizeBuf` 回填 → [[encoding-handling#UTF-16LE]]

---

## 关键常量

| 常量 | 值 | 说明 |
|------|-----|------|
| `TH32CS_SNAPPROCESS` | `0x00000002` | 快照标志，仅包含进程列表 |
| `PROCESS_QUERY_LIMITED_INFORMATION` | `0x1000` | 最小权限标志 |
| `ProcessCommandLineInformation` | `60` | NtQueryInformationProcess 信息类 |

---

## 资源管理模式

所有 `OpenProcess` / `CreateToolhelp32Snapshot` 获取的句柄，均在 `finally` 块中调用 `CloseHandle` 释放，防止句柄泄漏。

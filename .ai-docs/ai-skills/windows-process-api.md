# Skill: Windows 进程 API 调用规范

> 适用范围：所有调用 kernel32.dll / ntdll.dll 进程相关 API 的代码
> 关联文档：[[src/index.ts.docs]] · [[ai-skills/windows-ffi-koffi]] · [[ai-skills/encoding-handling]]

---

## 权限模型

本项目统一使用最小权限原则：

| 标志 | 值 | 用途 |
|------|-----|------|
| `PROCESS_QUERY_LIMITED_INFORMATION` | `0x1000` | 查询进程信息（命令行、路径），无需管理员 |

**不要使用** `PROCESS_ALL_ACCESS` 或 `PROCESS_VM_READ`，这些需要更高权限，且对 WeGame 启动的进程无效。

---

## NtQueryInformationProcess

### 两阶段查询模式

`ProcessCommandLineInformation (class=60)` 的标准调用模式：

```ts
const returnLenBuf = Buffer.alloc(4);

// 阶段 1：传空 buffer，获取所需字节数
// 返回非零 status (STATUS_INFO_LENGTH_MISMATCH)，但 returnLenBuf 被填入正确长度
NtQueryInformationProcess(hProcess, 60, null, 0, returnLenBuf);
const requiredLen = returnLenBuf.readUInt32LE(0);

// 阶段 2：分配正确大小的 buffer，执行实际查询
const infoBuf = Buffer.alloc(requiredLen);
const status = NtQueryInformationProcess(hProcess, 60, infoBuf, requiredLen, returnLenBuf);
if (status !== 0) return null; // 0 = STATUS_SUCCESS
```

**注意：** 阶段 1 的非零返回值是预期行为，不代表错误。

### 返回 buffer 结构（ProcessCommandLineInformation）

返回的 buffer 头部是 `UNICODE_STRING` 结构（x64 布局）：

```
offset 0-1:  USHORT Length        — 字符串字节长度（UTF-16LE）
offset 2-3:  USHORT MaximumLength
offset 4-7:  ULONG  Padding       — x64 对齐填充
offset 8-15: PVOID  Buffer        — 指针（指向 offset 16 处的数据）
offset 16+:  实际字符串数据（UTF-16LE）
```

读取方式：
```ts
const strByteLen = infoBuf.readUInt16LE(0);
const cmdLine = infoBuf.toString('utf16le', 16, 16 + strByteLen);
```

→ 详见 [[ai-skills/encoding-handling#UNICODE_STRING解析]]

---

## CreateToolhelp32Snapshot 进程遍历

```ts
const hSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
// TH32CS_SNAPPROCESS = 0x00000002，第二参数 0 表示全局快照

const size = koffi.sizeof(PROCESSENTRY32);
const buf = Buffer.alloc(size);
buf.writeUInt32LE(size, 0); // 必须先写入 dwSize

let ok = Process32First(hSnapshot, buf);
while (ok !== 0) {
  const entry = koffi.decode(buf, PROCESSENTRY32);
  // ... 处理 entry
  ok = Process32Next(hSnapshot, buf);
}
CloseHandle(hSnapshot);
```

---

## 资源清理规范

所有通过 `OpenProcess` / `CreateToolhelp32Snapshot` 获取的句柄，**必须**在 `finally` 块中调用 `CloseHandle`：

```ts
const hProcess = OpenProcess(PROCESS_QUERY_LIMITED_INFORMATION, 0, pid);
if (!hProcess) return null;
try {
  // ... 操作
} finally {
  CloseHandle(hProcess); // 无论成功失败都执行
}
```

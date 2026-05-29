# Skill: koffi FFI 库使用规范

> 适用范围：本项目所有通过 koffi 调用 Windows 原生 API 的代码
> 关联文档：[[src/index.ts.docs]] · [[ai-skills/windows-process-api]] · [[ai-skills/encoding-handling]]

---

## 类型别名

用 `koffi.alias` 将 Win32 类型名映射到 koffi 内置类型，提升代码可读性：

```ts
koffi.alias('HANDLE', 'void *');
koffi.alias('PVOID', 'void *');
koffi.alias('DWORD', 'uint32_t');
koffi.alias('ULONG', 'uint32_t');
koffi.alias('NTSTATUS', 'int32_t');
```

**规则：** 别名必须在任何 `koffi.load` 或 `koffi.struct` 调用之前定义。

---

## 结构体定义

用 `koffi.struct` 定义 C 结构体，字段顺序必须与 Win32 头文件完全一致：

```ts
const PROCESSENTRY32 = koffi.struct('PROCESSENTRY32', {
  dwSize: 'DWORD',
  // ... 字段按原始顺序
  szExeFile: 'char[260]',
});
```

**内存对齐：** koffi 默认遵循平台对齐规则（x64 上指针字段 8 字节对齐）。`PVOID` 类型字段会自动处理 x64 对齐。

**使用方式：**
```ts
const size = koffi.sizeof(PROCESSENTRY32);
const buf = Buffer.alloc(size);
buf.writeUInt32LE(size, 0); // dwSize 必须在首次调用前手动写入
const entry = koffi.decode(buf, PROCESSENTRY32);
```

---

## 函数导入

```ts
const lib = koffi.load('kernel32.dll');
const Func = lib.func('ReturnType __stdcall FuncName(ParamType paramName, ...)');
```

**调用约定：** Windows API 统一使用 `__stdcall`。

**输出参数标注：**
- `_Out_` — 纯输出参数（函数写入，调用方读取）
- `_Inout_` — 输入输出参数（调用方写入初始值，函数更新）

```ts
// 示例：QueryFullProcessImageNameW 的 lpdwSize 是 _Inout_
const QueryFullProcessImageNameW = kernel32.func(
  'int __stdcall QueryFullProcessImageNameW(HANDLE hProcess, DWORD dwFlags, _Out_ void *lpExeName, _Inout_ DWORD *lpdwSize)'
);
```

---

## Buffer 与内存操作

koffi 函数调用时，`void *` 参数直接传入 Node.js `Buffer`：

```ts
const buf = Buffer.alloc(requiredSize);
NtQueryInformationProcess(hProcess, infoClass, buf, requiredSize, returnLenBuf);
```

传 `null` 表示空指针（用于两阶段查询的第一阶段）：

```ts
NtQueryInformationProcess(hProcess, infoClass, null, 0, returnLenBuf);
```

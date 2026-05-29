# Skill: 字符编码处理规范

> 适用范围：所有涉及 Windows API 字符串读写的代码
> 关联文档：[[src/index.ts.docs]] · [[ai-skills/windows-ffi-koffi]] · [[ai-skills/windows-process-api]]

---

## UTF-16LE

Windows 宽字符 API（`W` 后缀函数）返回 UTF-16LE 编码字符串。

**读取方式：**
```ts
// charCount 为字符数（非字节数），字节数 = charCount * 2
const str = buf.toString('utf16le', 0, charCount * 2);
```

**QueryFullProcessImageNameW 示例：**
```ts
const buf = Buffer.alloc(1024);          // 512 宽字符 × 2 字节
const sizeBuf = Buffer.alloc(4);
sizeBuf.writeUInt32LE(512, 0);           // 传入最大字符数
QueryFullProcessImageNameW(hProcess, 0, buf, sizeBuf);
const charCount = sizeBuf.readUInt32LE(0); // 函数回填实际字符数
const path = buf.toString('utf16le', 0, charCount * 2);
```

---

## UNICODE_STRING 解析

`NtQueryInformationProcess(ProcessCommandLineInformation)` 返回的 buffer 头部是 `UNICODE_STRING`（x64 布局）：

```
offset 0-1:  Length        (USHORT) — 字符串字节长度，不含 null 终止符
offset 2-3:  MaximumLength (USHORT)
offset 4-7:  Padding       (ULONG)  — x64 对齐
offset 8-15: Buffer        (PVOID)  — 指针，实际数据紧跟在结构体之后
offset 16+:  实际 UTF-16LE 字符串数据
```

**读取方式：**
```ts
const strByteLen = infoBuf.readUInt16LE(0); // Length 字段
const cmdLine = infoBuf.toString('utf16le', 16, 16 + strByteLen);
```

---

## ANSI 字符串（PROCESSENTRY32.szExeFile）

`Process32First/Next` 使用 ANSI 版本，`szExeFile` 是 `char[260]`，以 null 字节终止。

**读取方式：**
```ts
const rawBytes = Buffer.from(entry.szExeFile as unknown as Uint8Array);
const nullIdx = rawBytes.indexOf(0);
const name = rawBytes.toString('utf8', 0, nullIdx !== -1 ? nullIdx : rawBytes.length);
```

**注意：** koffi decode 后 `szExeFile` 的类型是 `Uint8Array`，需要用 `Buffer.from` 包装后才能调用 `toString`。

---

## 编码选择速查

| 场景 | 编码 | Node.js 方法 |
|------|------|-------------|
| `W` 后缀 API 返回值 | UTF-16LE | `buf.toString('utf16le', start, end)` |
| UNICODE_STRING 结构体 | UTF-16LE | 从 offset 16 读，长度取 `readUInt16LE(0)` |
| PROCESSENTRY32.szExeFile | ANSI/UTF-8 | `buf.toString('utf8', 0, nullIdx)` |

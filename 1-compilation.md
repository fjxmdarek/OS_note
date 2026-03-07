# 编译的四个步骤

## 概览

| 步骤 | 英文 | 命令标志 | 输入 → 输出 | 负责程序 |
|------|------|----------|------------|----------|
| 预处理 | Preprocessing | `-E` | `.c` → `.i` | `cpp` |
| 编译 | Compilation | `-S` | `.i` → `.s` | `cc1` |
| 汇编 | Assembly | `-c` | `.s` → `.o` | `as` |
| 链接 | Linking | 无 | `.o` → 可执行文件 | `ld` |

`gcc` 是驱动程序，根据文件后缀自动调用以上四个工具。

---

## 第一步：预处理（Preprocessing）

```bash
gcc -E mem.c -o mem.i
```

- 展开所有 `#include`，把头文件内容原样粘贴进来
- 展开所有 `#define` 宏
- 处理 `#ifdef` / `#ifndef` 等条件编译
- 插入 `# 行号 "文件名"` 行号标记，供后续报错定位

结果：纯 C 代码，没有任何预处理指令。`mem.c` 22 行 → `mem.i` 3405 行（头文件全部展开）。

头文件里大量 `typedef` 是 Linux 为跨平台兼容做的类型层层别名，例如：
```c
typedef unsigned char __u_char;
typedef __u_char      uint8_t;
```

---

## 第二步：编译（Compilation）

```bash
gcc -S mem.i -o mem.s
```

- 词法分析、语法分析、语义分析
- 生成中间表示（IR）
- 优化
- 输出 x86-64 汇编代码

结果：人类可读的汇编文件 `.s`。

### 汇编里的关键概念

**本地标签（.L 开头）**：编译器自动生成的跳转目标，不出现在符号表里，对应 C 里的 if/while/for 等控制流。

```asm
cmpl  $2, -36(%rbp)   ; argc == 2 ?
je    .L8             ; 相等跳过错误处理
call  fwrite@PLT      ; 打印 usage
.L8:
    ...
```

**RIP 相对寻址**：所有地址访问用相对偏移，不用绝对地址，保证 ASLR 下代码正确运行。

```asm
leaq  .LC6(%rip), %rdi   ; 从当前指令位置偏移，不写死地址
```

**@PLT**：外部库函数的调用占位符，等链接阶段填入真实地址。

**栈金丝雀（Stack Canary）**：`movq %fs:40, %rax`，防止栈溢出攻击，函数返回前检查是否被改写。

---

## 第三步：汇编（Assembly）

```bash
gcc -c mem.s -o mem.o
```

- 把汇编指令翻译成机器码（二进制）
- 生成目标文件 `.o`，包含：
  - 机器码
  - 符号表（我提供什么、我需要什么）
  - 重定位表（哪些地址需要链接器填写）

此时 `call printf` 的地址是 0 或占位符，尚未解析。

---

## 第四步：链接（Linking）

```bash
gcc mem.o -o mem                    # 单文件
gcc shell1.o error.o -o shell1      # 多文件
```

- 合并多个 `.o` 文件
- 找到所有外部符号（`printf`、`malloc` 等）的真实地址
- 把所有空的 `call` 地址填上
- 生成最终可执行文件

### PLT / GOT 机制

外部库函数通过 PLT（Procedure Linkage Table）和 GOT（Global Offset Table）调用：

```
call printf@PLT
    → PLT 跳板
        → GOT 表（存着 printf 真实地址）
            → printf 实现
```

程序启动时，动态链接器（`ld.so`）把各库函数的真实地址填进 GOT。

---

## 多文件编译

分步（保存中间文件，改一个文件只需重新编译对应 .o）：
```bash
gcc -E shell1.c -o shell1.i && gcc -E error.c -o error.i && \
gcc -S shell1.i -o shell1.s && gcc -S error.i -o error.s && \
gcc -c shell1.s -o shell1.o && gcc -c error.s -o error.o && \
gcc shell1.o error.o -o shell1
```

一步（不保存中间文件）：
```bash
gcc shell1.c error.c -o shell1
```

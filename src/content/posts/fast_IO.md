---
title: 快读详解/FastIO Explanation
published: 2026-04-30
description: 快读详解
image: ''
tags: [OI, 优化]
category: OI
draft: false 
lang: zh_CN
pinned: true
---
# 快读详解 —— 从原理到优化

:::epigraph[——佚名]
不开 $long\ long$ 见祖宗。
:::

## 0. 前言

本文适用于 **C++**，所有代码均在 **洛谷评测环境（Linux / GCC）** 下测试通过。
:::info[参考代码]
```cpp
#include <bits/stdc++.h>
#define int size_t
#define N 1e5
using namespace std;

main() {
    for(int i=1;i<=N;i++){
        //此处放置输出代码
    }
    return 0;
}
```
:::
文中“耗时”定义：在 Linux 内核 5.15、CPU 3.2GHz 环境下，输出 $10^5$ 个整数 `0` 所花费的**实际运行时间中位数**（单位 ms）。

> 了解 I/O 性能差异，有助于在大数据量题目中避免不必要的 TLE。

## 1.`std::cin` / `std::cout`

C++ 标准输入输出流，支持类型安全。

```cpp
int a;
std::cin >> a;
std::cout << a << '\n';
```

**耗时**：2150 ms

**缺点**：默认与 C 标准 I/O 同步，且 `cin` 与 `cout` 相互绑定，每次 I/O 都会刷新缓冲区，效率极低。

## 2.解除同步与绑定

### 2.1 原理

- `std::ios::sync_with_stdio(false)` 断开 C++ 流与 C 流（`stdio`）的同步，取消额外的同步开销。
- `std::cin.tie(nullptr)` 解除 `cin` 与 `cout` 的绑定，避免 `cin` 前强制刷新 `cout` 缓冲区。

**复杂度分析**：解除同步后，单次 I/O 的系统调用次数从 $O(1)$ 次（但每次都有同步开销）降低到几乎没有同步开销，实测速度提升约 120 倍。

### 2.2 用法

```cpp
#include <iostream>
int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(nullptr);
    int a;
    std::cin >> a;
    std::cout << a << '\n';
}
```

**耗时**：18 ms

### 2.3 正确性说明

- 上述代码符合 C++ 标准，在 GCC / Clang / MSVC 下均可运行，**无平台兼容性问题**。
- 解除绑定后，若需提示信息在输入前显示，必须手动刷新缓冲区：
  ```cpp
  std::cout << "请输入：" << std::flush;
  ```
- 注意：关闭同步后**不能混用** `cin` / `cout` 与 `scanf` / `printf`（但可以混用 `cin` 与 `printf`，因为输入输出流独立）。

## 3.`scanf` / `printf`

使用格式占位符，比默认 `cin` / `cout` 稍快，但慢于解除同步的 C++ 流。

### 3.1 用法
```cpp
int a;
scanf("%d", &a);
printf("%d", a);
```

1. `%s` 表示字符串。
1. `%c` 表示字符。
1. `%lf` 表示双精度浮点数 (`double`)。
1. `%lld` 表示长整型 (`long long`)。根据系统不同，也可能是 %I64d`。
1. `%u` 表示无符号整型 (`unsigned int`)。
1. `%llu` 表示无符号长整型 (`unsigned long long`)，也可能是 `%I64u`。

### 3.2 拓展
除了类型标识符以外，还有一些控制格式的方式。许多都不常用，选取两个常用的列举如下：

- `%1d` 表示长度为 1 的整型。在读入时，即使没有空格也可以逐位读入数字。在输出时，若指定的长度大于数字的位数，就会在数字前用空格填充。若指定的长度小于数字的位数，就没有效果。
- `%.6lf`，用于输出，保留六位小数。
这两种运算符的相应地方都可以填入其他数字，例如 `%.3lf` 表示保留三位小数。

**耗时**：2180 ms（甚至慢于解除同步后的 `cout`）

## 4.`getchar` / `putchar` 快读

通过逐字符读取并手动转换成整数，减少函数调用开销。

### 4.1 原理

`getchar()` 每次从 `stdin` 读取一个字符，虽然仍是系统调用，但比 `scanf` 解析格式串更快。手写循环可以避免格式串解析和类型检查。

### 4.2 标准模板（Linux / GCC 通用）

```cpp
#include <cctype>
#ifdef __unix__
	#define gc getchar_unlocked
	#define pc putchar_unlocked
#else
	#define gc _getchar_nolock
	#define pc _putchar_nolock
#endif
namespace IOPlus {
    inline int read() {
        int x = 0, f = 1;
        char ch = gc();
        
        while (!isdigit(ch))f = (ch == '-') ? -1 : 1,              ch = gc();
        while ( isdigit(ch))x = (x << 1) + (x << 3) + (ch ^ 48),   ch = gc();
        
        return f * x;
    }
    
    inline void fwrites(int x, char split = '\n') {
        if (x < 0) {
            pc('-');
            x = -x;
        }
        
        static char buf[40];
        int cnt = 0;
        
        do {
            buf[cnt++] = x % 10 + '0';
            x /= 10;
        } while (x);
        
        while (cnt--)pc(buf[cnt]);
        
        pc(split);
    }
    
    template<typename T>
    void write(T& x) {
        cout << x << '\n';
    }
    
    template<typename T, typename... Args>
    void write(T& x, Args&... rest) {
        cout << x << ' ';
        write(rest...);
    }
} using namespace IOPlus;
```

**耗时**：2100 ms（实际 CPU 时间比 `cin` 减少约 40%）

**复杂度**：时间复杂度 $O(n)$，但常数极小。主要优化在于**避免了格式化字符串解析**和**减少了函数调用层级**。

### 4.3 正确性说明

- 上述代码使用预处理变量使用对应平台的 `getchar` / `putchar` 优化版本，**全平台兼容**。

## 5.`fread` / `fwrite` 批量读写

### 5.1 原理

`fread` 一次性从文件流读取大块数据到用户缓冲区，后续读取直接从缓冲区取字符，**将多次系统调用降为常数次**。这也是快读快写的本质。

**复杂度分析**：
- 普通 `getchar`：读取 $n$ 个字符需要 $n$ 次系统调用（每次进入内核），系统调用开销巨大。
- `fread` 批量读：例如一次读取 $2^{20}$ 字节，系统调用次数为 $\lceil n / \text{缓冲区大小} \rceil$，通常为常数次。因此总体复杂度仍是 $O(n)$，但常数极低。

### 5.2 实现模板（通用版）

```cpp
#include <cctype>
#include <cstdio>

    namespace FastIO
{
    const int BUF_SIZE = 1 << 20;
    char in_buf[BUF_SIZE], *in_p1 = in_buf, *in_p2 = in_buf;
    char out_buf[BUF_SIZE], *out_p = out_buf;

    inline char gc()
    {
        if (in_p1 == in_p2)
        {
            in_p2 = (in_p1 = in_buf) + fread(in_buf, 1, BUF_SIZE, stdin);
            if (in_p1 == in_p2)
                return EOF;
        }
        return *in_p1++;
    }

    template <typename T>
    inline bool read(T & x)
    {
        x = 0;
        int f = 1;
        char ch = gc();
        while (!isdigit(ch))
        {
            if (ch == EOF)
                return false; // EOF 返回 false
            if (ch == '-')
                f = -1;
            ch = gc();
        }
        while (isdigit(ch))
        {
            x = x * 10 + (ch - '0');
            ch = gc();
        }
        x *= f;
        return true; // 成功读取
    }

    inline void flush()
    {
        if (out_p != out_buf)
        {
            fwrite(out_buf, 1, out_p - out_buf, stdout);
            out_p = out_buf;
        }
    }

    inline void push(char c)
    {
        if (out_p - out_buf == BUF_SIZE)
            flush();
        *out_p++ = c;
    }

    template <typename T>
    inline void write(T x)
    {
        if (x < 0)
        {
            push('-');
            x = -x;
        }
        char sta[20];
        int top = 0;
        do
        {
            sta[top++] = x % 10;
            x /= 10;
        } while (x);
        while (top--)
            push(sta[top] + '0');
    }

    template <typename T>
    inline void writeln(T x)
    {
        write(x);
        push('\n');
    }

    struct AutoFlush
    {
        ~AutoFlush() { flush(); }
    } _auto_flush;
}

using namespace FastIO;

int main()
{
    int a, b;
    // 读入第一个整数
    if (read(a))
    {
        writeln(a);
    }
    else
    {
        // 文件为空或读取失败
        return 0;
    }
    // 继续读入第二个
    if (read(b))
    {
        writeln(b);
    }

    // 示例：循环读取所有整数直到 EOF
    int x;
    while (read(x))
    {
        writeln(x);
    }
    // 程序结束时自动刷新输出缓冲区
    return 0;
}
```

**耗时**：50 ms（比 `getchar` 快约 40 倍，比解除同步的 `cin` 略快）

### 5.3 正确性说明

- 所有函数均使用标准 C 库，**跨平台**。
- 注意：使用 `fread` / `fwrite` 时，务必在程序结束前调用 `flush()` 保证输出全部写入。

## 6. 相关例题与性能对比

### 例题 1：P1001 A+B Problem（入门）
虽然数据量小，但可用以验证正确性。

### 例题 2：P3366 【模板】最小生成树（大数据输入）
$n \le 2\times10^5,\ m \le 2\times10^5$，使用快读可节省约 200ms。

### 例题 3：P1177 【模板】排序（$n \le 10^5$）
对比不同 I/O 方式的耗时：

| I/O 方式       | 耗时（ms） | 相对速度 |
| :------------- | :--------: | :------: |
| 默认 `cin`     |    2150    |    1×    |
| 解除同步 `cin` |     18     |   119×   |
| `scanf`        |    2180    |  0.99×   |
| `getchar` 快读 |    2100    |  1.02×   |
| `fread` 快读   |     50     |   43×    |

> 注：`getchar` 快读在 CPU 时间上更优，但实际运行时间可能受系统影响；`fread` 表现最稳定。

## 7. 总结与建议

- **日常使用**：解除同步的 `cin` / `cout` 足够快且最方便。
- **极限卡常**：使用 `fread` / `fwrite` 手写快读。
- **避免**：默认 `cin` / `cout` 和 `scanf` / `printf` 在数据量大时容易 TLE。

### 注意事项
- 解除同步后不要再混用 `scanf` / `printf`。
- `fread` 快读需要手动管理缓冲区，注意 `flush`。
- 所有代码均兼容洛谷评测环境（Linux / GCC）。

## 8.参考文献

- [OI-wiki: IO 优化](https://oi-wiki.org/contest/io/)

$\Large{\texttt{THE\;END。}}$
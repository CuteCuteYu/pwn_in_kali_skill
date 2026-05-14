# Pwn 二进制漏洞利用指南

一个完整的 GDB 与 Pwntools 学习和参考指南，包含环境配置、调试命令、漏洞利用模板等内容。

## 前置要求

### 操作系统
- **Kali Linux** (推荐 2025 或更新版本)
- 本指南所有工具和命令均已针对 Kali Linux 环境进行测试

### 必需工具

#### Kali 自带工具（无需额外安装）
- **GDB** (GNU Debugger) - 调试器
- **Pwndbg** - GDB 增强插件（Kali 自带）
- **Objdump** - 反汇编工具
- **Readelf** - ELF 文件分析工具
- **NM** - 符号提取工具
- **Strings** - 字符串提取工具
- **File** - 文件类型识别
- **Checksec** - 安全保护检查工具
- **Radare2 / Rabin2** - 高级静态分析工具

#### 需要安装的工具
- **Python 3** - Kali 自带
- **UV** - Python 包管理器（需要安装）
- **Pwntools** - Python CTF 框架（通过 uv 安装）

## 环境配置

### 1. 安装 UV（如果未安装）

```bash
# 使用官方脚本安装
curl -LsSf https://astral.sh/uv/install.sh | sh

# 或使用 pip 安装
pip install uv
```

### 2. 在当前项目目录初始化环境

```bash
cd /path/to/pwn_ai

# 初始化 uv 项目（如果尚未初始化）
uv init

# 安装 pwntools
uv add pwntools
```

### 3. 验证安装

```bash
# 检查 uv 版本
uv --version

# 检查 pwntools 安装
uv run python -c "from pwn import *; print(pwn.__version__)"

# 检查 GDB 和 pwndbg
gdb --version
gdb -ex "pi pwndbg.version" -batch
```

## 使用方式

### 查看技能文档

```bash
# 使用 Skill 工具查看完整指南
# (在支持 Claude Code 的环境中)
/pwn
```

### 运行 Pwntools 脚本

```bash
# 使用 uv 运行脚本
uv run python solve.py

# 或激活虚拟环境后运行
source .venv/bin/activate
python solve.py
```

### GDB 调试二进制文件

```bash
# 基础调试
gdb ./binary

# 使用 pwndbg 增强（Kali 默认已加载）
gdb ./binary
# pwndbg 命令: stack, heap, vmmap, rop, 等
```

## 技能文档内容概览

| 章节 | 内容 |
|------|------|
| 第一部分 | 静态分析工具详解（file, checksec, readelf, objdump, nm, strings, r2） |
| 第二部分 | GDB 调试详解（基础命令、pwndbg 增强、实战场景） |
| 第三部分 | Pwntools 详解（数据打包、进程连接、ROP 工具、shellcraft） |
| 第四部分 | GDB + Pwntools 联合调试 |
| 第五部分 | 常见漏洞类型与利用（栈溢出、ret2libc、格式化字符串、堆利用） |
| 第六部分 | 调试技巧汇总 |

## 常用命令速查

### 静态分析

```bash
# 检查二进制保护机制
checksec --file=./binary

# 查看函数符号
readelf -s ./binary | grep FUNC

# 反汇编函数
objdump -d ./binary | grep -A 30 "main>"

# 提取字符串
strings ./binary | grep -i flag
```

### Pwntools

```python
from pwn import *

# 加载二进制
elf = ELF('./binary')

# 获取地址
win_addr = elf.symbols['win']

# 构造 payload
payload = b'A' * 40 + p64(win_addr)

# 连接并发送
p = process('./binary')
p.sendline(payload)
p.interactive()
```

### GDB 调试

```bash
# 设置断点
b main
b *0x401196

# 查看栈和寄存器
x/20gx $rsp
info registers

# 使用 pwndbg
stack
vmmap
rop --grep "pop rdi"
```

## 项目结构

```
pwn_ai/
├── .claude/
│   └── skills/
│       └── pwn/
│           └── SKILL.md          # 完整技能文档
├── .venv/                         # UV 虚拟环境
├── pyproject.toml                 # UV 项目配置
├── uv.lock                        # UV 锁文件
└── README.md                      # 本文件
```

## 学习路径建议

1. **初学者**: 先阅读技能文档的第一部分（静态分析）和第二部分（GDB 基础）
2. **进阶**: 学习第三部分（Pwntools）和第四部分（联合调试）
3. **实战**: 参考第五部分的漏洞类型，配合第六部分的调试技巧进行练习

## 常见问题

### Q: 为什么需要使用 UV？
A: UV 是一个快速的 Python 包管理器，能更好地管理依赖和虚拟环境。相比传统的 pip + venv，UV 更快且更可靠。

### Q: 可以不使用 UV 吗？
A: 可以，但本指南默认使用 UV 管理环境。如果你想使用传统方式：
```bash
python3 -m venv .venv
source .venv/bin/activate
pip install pwntools
```

### Q: Pwndbg 命令不可用怎么办？
A: 确保 GDB 已加载 Pwndbg：
```bash
gdb ./binary
# 在 GDB 中执行
pi pwndbg.version
```

## 参考资源

- [Pwntools 官方文档](https://docs.pwntools.com/)
- [Pwndbg GitHub](https://github.com/pwndbg/pwndbg)
- [CTF Wiki](https://ctf-wiki.org/)

---

**最后更新**: 2025-05-14
**适用环境**: Kali Linux, Python 3.13+, UV, Pwntools 4.15.0

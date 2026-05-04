- **Ubuntu 16.04**：`2.23-0ubuntu10_amd64` 和 `2.23-0ubuntu10_i386`
    
- **Ubuntu 18.04**：`2.27-3ubuntu1_amd64` 和 `2.27-3ubuntu1_i386`
    
- **Ubuntu 20.04**：`2.31-0ubuntu9_amd64` 和 `2.31-0ubuntu9_i386`
    
- **Ubuntu 22.04**：`2.35-0ubuntu3_amd64` 和 `2.35-0ubuntu3_i386`

# 不同版本对于堆利用造成的影响
| glibc版本     | 典型Ubuntu      | 核心变化                                 | 被“杀死”的经典手法              |
| ----------- | ------------- | ------------------------------------ | ----------------------- |
| ≤ 2.23      | 16.04         | 传统堆管理，无tcache                        | –                       |
| 2.24 – 2.26 | 17.04 – 17.10 | 引入tcache（性能优化）                       | –                       |
| 2.27 – 2.31 | 18.04 – 20.04 | tcache关键检查强化                         | tcache dup（double free） |
| 2.32 – 2.33 | 20.10 – 21.04 | **safe-linking**（tcache/fastbin指针加密） | 传统tcache poisoning      |
| 2.34 – 2.35 | 22.04         | 移除`__malloc_hook`/`__free_hook`      | 直接hook劫持                |
| 2.36+       | 22.10+        | 更多加固（如large bin检查）                   | 复杂化利用                   |
### 1. glibc ≤ 2.23（Ubuntu ≤16.04）

- **特点**：无tcache，只有fastbin、unsorted bin、small/large bin。
    
- **常用手法**：
    
    - Fastbin dup / double free
        
    - Unsorted bin attack（libc地址泄露 + 任意地址写）
        
    - `__malloc_hook` / `__free_hook` / `__realloc_hook` 劫持
        
    - House of Spirit / House of Force / House of Orange
        
- **影响**：最“自由”的时代，几乎所有经典攻击都可用。
    

### 2. glibc 2.24 – 2.26（Ubuntu 17.04-17.10）

- **新增tcache**（per-thread cache，每个大小桶最多7个chunk）。
    
- **影响**：
    
    - tcache使fastbin/small bin利用变得“简单”——无需伪造size，不检查key。
        
    - 但**tcache stashing**等新手法诞生。
        
    - 仍然保留hook，所以利用仍然直接。
        

### 3. glibc 2.27 – 2.31（Ubuntu 18.04-20.04）

- **关键补丁**：tcache添加 `key` 字段（位于bk位置），检查double free。
    
    - 不能再直接 `free` 同一个tcache chunk两次。
        
    - 绕过方法：先free进fastbin/unsorted，再通过其他方式重新进入tcache。
        
- **其他变化**：
    
    - tcache per-thread count上限可自定义（但默认7）。
        
    - `_int_free` 中对top chunk size检查更严。
        
- **实战影响**：tcache double free被禁，但仍可通过**fastbin dup**或**unsorted bin**配合完成。
    

### 4. glibc 2.32 – 2.33（Ubuntu 20.10-21.04）—— **转折点**

- **safe-linking 机制**：
    
    - tcache和fastbin的 `fd` 指针被异或加密：`P ^ (P >> 12)`。
        
    - 直接覆写 `fd` 做任意地址分配**不再有效**，需要泄漏堆地址才能构造密文。
        
- **影响**：
    
    - 传统tcache poisoning / fastbin poisoning需要额外信息（堆地址）。
        
    - 但若能泄露一个堆地址，仍可计算密文绕过。
        
    - hook依然存在，所以最终劫持仍可能。
        

### 5. glibc 2.34 – 2.35（Ubuntu 22.04）—— **hook移除**

- **移除符号**：`__malloc_hook`, `__free_hook`, `__realloc_hook`, `__memalign_hook`。
    
- **影响**：
    
    - 无法直接通过写hook获得代码执行。
        
    - 需要寻找新的函数指针目标，例如：
        
        - `_IO_wide_data` 中的 `_IO_wfile_jumps`（FSOP）
            
        - `exit` 函数中调用的 `__call_tls_dtors` / `__libc_atexit`
            
        - `__after_morecore_hook`（较冷门）
            
        - 栈迁移 + ROP（需要泄露栈地址或libc内gadget）
            
- **堆利用变得更间接**：通常需要结合**FILE结构体攻击**或**exit流程劫持**。
    

### 6. glibc 2.36+（Ubuntu 22.10+）

- **进一步加固**：
    
    - Large bin 插入检查加强（防止 `large bin attack` 任意地址写）。
        
    - `_int_realloc` 更严格。
        
    - safe-linking扩展到small bin?（部分版本）。
        
- **影响**：一些经典手法失效，需要更精细的构造。
## 三、实战建议对照表

|目标版本|推荐攻击路径|
|---|---|
|2.23 – 2.26|任意手法均可，优先tcache dup + hook|
|2.27 – 2.31|tcache dup需要绕过key（可fastbin dup中转），仍可hook|
|2.32 – 2.33|先泄露堆地址，再构造加密fd，最后hook|
|2.34 – 2.35|放弃hook，转向FSOP（如`_IO_2_1_stdout_`泄露，`_IO_list_all`劫持）或exit回调|
|2.36+|需要研究最新bypass（通常利用`_dl_open`、`libc.so`中的其他函数指针）|

## 四、一个小技巧

当你拿到一个libc文件，可以快速查看版本和防御特性：

bash

strings libc.so.6 | grep "GNU C Library"
readelf -s libc.so.6 | grep " malloc_hook"   # 如果有输出则hook存在
objdump -d libc.so.6 | grep "safe-linking"   # 不一定有字符串，但可看`_int_free`中异或指令

**结论**：不同libc版本不仅影响偏移，更决定哪些漏洞组合能被利用。维护本地libc数据库，并在写exp前先确认远程版本，是避免浪费时间的关键。

This response is AI-generated, for reference only.
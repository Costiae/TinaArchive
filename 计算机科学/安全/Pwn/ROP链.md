是一种通过拼尸块已达到借尸还魂的超级拼装。

根据作用方式存在一下几种方向，难度由低到高，所需信息由多到少。
+ [[Ret2LibC]]
+ [[Ret2DLResolve]]
---
# 方法速查
### gadget
+ `pop rdi ; ret`
+ `leave ; ret`
+ `ret`
### 64位
```python
pl+=p64(rsi) #pop rsi ; ret
pl+=p64(arg1)
pl+=p64(rdi) #pop rdi ; ret
pl+=p64(arg0)
pl+=(func) # func(arg0, arg1)
pl+=(ret_to) #program flow where to go
```
>注意[[栈对齐]]。
### 32位
```python
pl+=p32(func) # func(arg0, arg1)
pl+=p32(ret_to)
pl+=p32(arg0)
pl+=p32(arg1)
```
---
# 基础原理
在[[栈溢出]]之后，理论上拥有了将程序流跳转到任何地址的能力，那么，如何将任意运行的能力转换为运行shell呢？参见[[调用约定]]，我们可以模仿这种方式启动函数。
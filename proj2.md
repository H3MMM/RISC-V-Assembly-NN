# 中文乱码问题

../../src/relu.s:26: **got 4 arguments but expected 3** add s0, a0, x0 #å½åè¦æä½çåç´ æé

把中文当成下一个参数了，所以注释最好写英文

# 跳转分支标签和调用函数的区别

```assembly
loop_start:
    add s0, a0, x0  # copy a0 to s0
    add s2, x0, x0 

    beq s1, s2, loop_end  # if s1==s2 jump
    lw a0, 0(s0)     # load word from memory
    ble a0, zero, change 

loop_continue:
    addi s2, s2, 1 #count++
    addi a0, s0, 4 #i++
    j loop_start
```

为什么在这里用   j loop_start  可以但是   jal loop_start 不行

因为代码是一个 **循环体**，而不是函数调用

- 用 `j loop_start` → 程序跳回 `loop_start`，无限循环，**符合循环逻辑**。
- 如果用 `jal loop_start` → 会把返回地址存到 `ra`（默认 `jal loop_start` = `jal ra, loop_start`），每次循环都把 `ra` 覆盖掉。

- 如果你不使用 `ra` 返回，就没关系，但如果后面还有 `ret` 或者其他函数调用，`ra` 被覆盖会导致 **返回地址错误/栈破坏**。循环里用 `jal` 没意义，因为你不打算 `ret` 回去。

### 什么时候用 `jal`？

- **函数调用**时用 `jal`，跳转到函数入口并保存返回地址。
- 循环/条件跳转 **不需要返回地址** → 用 `j`、`beq`、`bne` 等即可。

# 什么时候用s寄存器，什么时候用t寄存器

### 一句话总结

> **如果你的代码不会调用别的函数，就全用 t 寄存器。**
> **如果你的代码要调用别的函数，而且想在调用后保留某些值，就用 s 寄存器并保存/恢复。**

# 用s寄存器时要注意什么

### 1️⃣ 函数入口要“保存”

在你的函数开始时，如果要使用某个 `sX` 寄存器，**必须先把它原本的值保存到栈上**，防止破坏调用者的数据。

```
addi sp, sp, -8     # 为保存两个寄存器腾出空间
sw s0, 0(sp)
sw s1, 4(sp)
```

------

### 2️⃣ 函数退出要“恢复”

返回前，要把原来的值取回来。

```
lw s0, 0(sp)
lw s1, 4(sp)
addi sp, sp, 8
ret
```

💡 **如果你只用了 s0**，那就只保存和恢复 s0 即可。不要多，也不要少。

------

### 3️⃣ 注意栈平衡

函数入口 `sp` 减多少（分配栈空间），退出就要加回同样的值。
 比如入口 `addi sp, sp, -8`，那退出时一定要 `addi sp, sp, 8`。
 否则，返回后 `sp` 出错，会破坏上层函数的栈。

# ReLU函数错误(循环案例)

### 源代码

```assembly
relu:
    addi sp, sp, -12
    sw s0, 8(sp)
    sw s1, 4(sp)
    sw s2, 0(sp)
    add s1, a1, x0 #数组长度

loop_start:
    add s0, a0, x0  # copy a0 to s0
    add s2, x0, x0 
    beq s1, s2, loop_end  # if s1==s2 jump
    lw a0, 0(s0)     # load word from memory
    ble a0, x0, change

loop_continue:
    addi s2, s2, 1 #count++
    addi a0, s0, 4  #i++
    j loop_start

loop_end:
    lw s0, 8(sp)
    lw s1, 4(sp)
    lw s2, 0(sp)
    addi sp, sp, 12
	ret
    
change:
    add a0, x0, x0
    sw a0, 0(s0)
    j loop_continue
```

#### 1.注意跳转到分支change怎么写

一般来说，跳转到分支并不会像jal一样还要返回调用地址，而是一直往下走，由于还需要回到循环，所以必须j loop_continue

#### 2.注意进入循环的量，会不会在循环中被重置

```assembly
add s0, a0, x0  # copy a0 to s0
add s2, x0, x0 
beq s1, s2, loop_end  # if s1==s2 jump
```

在循环中，s0(保存当前要操作的元素指针)，s2(计数器)一直被重置，导致循环一直出不去，s0一直+4，访问非法内存

![image-20251011111902672](attachments/image-20251011111902672.png)

#### 3.跳转分支和函数调用不一样，跳转分支比较简单

函数调用需要:

开栈保存ra和要用到的s寄存器

调用结束后恢复

# 调用函数时ra是在开栈的时候就保存还是在函数调用的前一行保存

> ra是在 **调用函数那一行（jal）执行时** 自动保存的。不是在被调函数的开头，也不是提前保存。

### 那为什么我们在函数开头又要“保存 ra”？

因为：
 **被调函数可能会再次调用别的函数**（嵌套调用）。
 而新的 `jal` 会覆盖掉 `ra`(因为ra只能存一个，不能存两个)

所以，函数的“序言部分”（Prologue）要手动保存 `ra`

### 如果是多层嵌套呢？ra怎么保存好几个返回地址

![image-20251011112648732](attachments/image-20251011112648732.png)

![image-20251011112659897](attachments/image-20251011112659897.png)

![image-20251014212104403](attachments/image-20251014212104403.png)

而且ra要存在栈的最底部，这才符合函数调用的约定

# Argmax函数问题

循环计数混乱，没有合理迁移

![image-20251011184154108](attachments/image-20251011184154108.png)

这里的t1相当于高级语言中的int i，应该学会迁移，特别是边界条件，分清自己从数组的第几个元素开始遍历，遍历到最后应该等于数组的元素个数(int i = 0; i < n; i++)



# DotProduct函数

### 怎么把i*stride(步长)这种变量作为要访问的地址的偏移量？

```assembly
# 假设 a0 = &arr[0]
# t0 = i
# t1 = stride

mul t2, t0, t1       # t2 = i * stride
slli t2, t2, 2        # ×4，因为int占4字节（或用 mul t2, t2, 4 也行）
add t2, a0, t2        # t2 = a0 + i*stride*4
lw  t3, 0(t2)         # t3 = arr[i * stride]
```

需要一个临时寄存器，提前计算出地址

# Matmul函数

### 在需要调用函数时，不可使用s0寄存器

为什么

### 要搞清楚函数调用的逻辑和书写格式

![image-20251013204845091](attachments/image-20251013204845091.png)

函数调用后不需要lw(为什么？)

![image-20251013194532964](attachments/image-20251013194532964.png)

函数调用前将值存入s寄存器之后，就不要再用a寄存器了，因为会在下一次循环中被覆盖，一定要用s

![image-20251013195453337](attachments/image-20251013195453337.png)

### 外层循环忘记更新行指针

![image-20251013203721142](attachments/image-20251013203721142.png)

### 写双层循环的一些注意点：

- 外层是退出条件判断+更新行指针+重置列指针为0
- 内层是退出条件判断+更新列指针

2025.10.13

这几天一直被汇编折磨，matmul写了一整天才终于在人工智能导论课上通过了

happyhappy

![image-20251013205405170](attachments/image-20251013205405170.png)

# readMatrix函数

### mv，beq等函数针对的都是寄存器，不能使用常数

![image-20251015142050107](attachments/image-20251015142050107.png)

必须先

```
addi t0, x0, -1
beq a0, t0, fopenError
```

循环条件设置错误

![image-20251015145643473](attachments/image-20251015145643473.png)

在循环一开始设置a0 != 4时退出，但是一开始a0并不是4，需要初始化其为0以启动循环



# writeMatrix函数

### 怎么取一个int类型数据的指针

![image-20251015204431513](attachments/image-20251015204431513.png)

答案是！暂存到栈上

![image-20251015204643907](attachments/image-20251015204643907.png)

# Classify函数

### 怎么把栈指针地址传给函数

![image-20251016194743630](attachments/image-20251016194743630.png)

### 参数数量设置错误

这里main.s也是一个参数，应该是5

![image-20251016143602901](attachments/image-20251016143602901.png)

### 寄存器用之前没有初始化

![image-20251016144610617](attachments/image-20251016144610617.png)

### 对寄存器所存的东西认识混乱

![image-20251016184453568](attachments/image-20251016184453568.png)

这种暂存在栈上的一定要分清存的是什么，画个图一目了然

### 传入参数错误

一开始传成a0了

![image-20251016185831658](attachments/image-20251016185831658.png)

注意设计系统调用的，一般a0是系统调用码，a1才是真正的参数



25/10/16

![image-20251016190857854](attachments/image-20251016190857854.png)

### 不能用s寄存器存指针

调用约定规定，s寄存器要在函数调用前后保持一致，如果用s寄存器存指针，则会通过指针直接修改其值，导致约定被破坏

![image-20251016200436104](attachments/image-20251016200436104.png)

### *最大问题：如何存储一个指针

这个问题关系到内存空间管理的结构是否规范

既然s寄存器要在前后保持一致，假设我需要一个指针传入函数（此时他是垃圾值），函数过后他又是我需要我的值



### lw和sw写反

![image-20251017112350209](attachments/image-20251017112350209.png)

报这个错误一般就是忘记加回栈指针sp / 写入和读出写反，这里本来应该是把a0写入8(sp)的

### mnist数据集出错

![image-20251017113647508](attachments/image-20251017113647508.png)

结果发现是这个错误码2147483482 = 0x8000005A，也就是exit(90)，是fopen报错，进一步发现是他找不到test_minst_main这个文件夹，创建了之后就好了

# The End

![image-20251017120410327](attachments/image-20251017120410327.png)

10.17完成！！看到ok那一刻心都放松了很多，看到自己画出来的8被识别真的很欣慰

11-17耗时一星期终于做完了，这一周确实是被汇编代码折磨了许久，第一次接触如此底层的代码，不禁感慨以前过的都是什么好日子

从一开始什么都不会，到对调用约定慢慢熟悉，确实切身感受到了自己的进步

继续前进！

![image-20251016192430066](attachments/image-20251016192430066.png)

![image-20251016192443742](attachments/image-20251016192443742.png)
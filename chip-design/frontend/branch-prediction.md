> 分支预测学习笔记

# C910 I-Cache

![I-Cache physical implementation](/Users/fujie/Pictures/typora/icache1.jpg)
![I-Cache logical design ](/Users/fujie/Pictures/typora/icache2.jpg)

# C910 处理器中的分支预测设计

[C910 微架构学习：分支预测](https://zhuanlan.zhihu.com/p/460942331)

> 为什么需要使用分支预测技术：随着处理器流水级数的增加，如果等到 decode、execute 之后才得到分支指令准确的跳转结果，那么代价是很高的。**我们必须要在取指单元通过分支预测来预测跳转指令的地址。**

C910 中一共使用来 5 个模块来做分支预测：

1. 分支历史表(BHT)
2. 快速跳转目标缓冲器(L0 BTB)
3. 分支跳转目标缓冲器(BTB)
4. 间接跳转目标缓冲器(IND BTB)
5. 短循环减速器(Loop Buffer)

![branch prection of C910](/Users/fujie/Pictures/typora/bp01.jpg)

1. pcgen 模块用于 pc 的重定向，它接收各个预测器发来的 pc 值，对当前 pc 进行更新
2. icache 中的 predecode
   - 得到指令包中的“绝对跳转指令”和“条件跳转指令”在指令包中的**位置**
3. ipdecode：在 icache 的 predecode 的基础上，额外译码：
   - 间接跳转指令：用于间接分支预测；跳转指令的 offset：用于判断跳转地址是否预测正确
   - call 指令、return 指令：用于堆栈操作以及 L0 BTB 更新
   - load/store 指令
   - auipc 指令、一些向量指令等
4. L0 BTB：
   - 若命中则在 IF 级就可以得到目的 pc
   - L0 BTB 的数据在 IP 级维护：在 IP 级将 LO BTB 跟 BTB 进行比较，如果二者的跳转目标不一致，则说明 L0 BTB 预测错误，更新 L0 BTB 并且刷新 IF 级的数据
   - 对 return 指令的维护也在 IP 级：和 ras 返回给 IP 级的栈顶 pc 进行比较，不一致则更新 L0 BTB
5. addressgen 模块在 IB 后一级，将指令译码之后得到的 pc 发给 pcgen 和 BTB 模块
6. BHT：用于记录分支跳转历史信息，C910 尽可能在更早的流水线预测**条件分支**的跳转方向，C910 中根据*当前指令的部分 bit*和*前几条条件分支指令的跳转信息*作为索引，到 BHT 取出预测的结果。

   - 采用 2bits 的预测状态位，预测出错两次之后才会改变预测的跳转判断: 采用当前 pc 作为索引可以在 BHT 得到 2bits 的跳转预测信息，高 bit 为 1 则预测跳转
     ![使用PC对BHT进行索引](/Users/fujie/Pictures/typora/bht01.jpg)

   - <u>局部预测器</u>：只参考了当前 pc 的历史，没有参考分支指令之间的相关性. 例如下面的 C 代码中：第二条分支就依赖于前一条分支的结果

     ```c++
     if(b)
         a=0;
     if(a)
         S;
     ```

   - <u>相关预测器</u>：引入了 GHR(global history register), BHR(branch history register)来分析跳转指令之间的相关性，例如:

     - Gselect(global predictor with index selection)预测器：它将<u>PC 和 GHR 做位拼接</u>之后，用于索引 BHT。
       ![Gselect predictor](/Users/fujie/Pictures/typora/gselect.jpg)
     - GShare 预测器: <u>PC 跟 GHR 做异或</u>操作后用来索引 BHT。
       ![GShare predictor](/Users/fujie/Pictures/typora/gshare.jpg)
     - 混叠预测器(BI-Mode predictor)：可以规避<u>分支别名</u>问题

       分支别名(branch aliasing)：当 BHT 相对于 PC 的范围来说不够大时，BHT 只能存储部分 PC 的内容，当 BHT 更新之后，同一个 PC 索引得到的 BHT entry 的行为可能发生变化(该 entry 可能存放的是其他 PC 的行为)。
       ![BI-Mode predictor](/Users/fujie/Pictures/typora/bi-mode.jpg)

   - **C910 Predictor**：是一种 BI-Mode predictor
     ![C910 predictor](/Users/fujie/Pictures/typora/C910BHT.jpg)
     - predict array: 2\*1024\*32
     - select array: 128\*16, select array 会给出 2bits 的数据，该数据高位用于指明选择*taken array*还是*not taken array*
     - GHR: 22bits, 其更新数据来源有 RTU, BJU, loop buffer, BHT

7. BTB：预测跳转的地址

   > PC width set to be 40 bits

   - L0 BTB：为了在 IF 级就对下一条 pc 进行预测，避免 bubble
     - 16 entry
     - 全相连
     - 以 pc[14:0]作为 tag 访问 L0 BTB
     - 由于 L0 BTB 只有 16 个 entry，故应该将最容易跳转的 pc 存放到其中，如：strongly taken 的条件跳转指令、绝对跳转指令、return 指令
     - L0 BTB 预测错误或者未命中的时候，都需要更新，在 IB 级对其进行更新
   - BTB：main BTB
     - 跟 L0 BTB 并行访问
     - 跟 L0 BTB 是包含的关系，存在与 L0 BTB 中的 pc 一定存在于 BTB 中。
     - 比较 L0 BTB 的预测值跟 BTB 的预测值，可以在 IP 级判断 IF 级的预测是否正确，如果错误则需要冲刷流水线
     - 4 路组相连
   - ras：进过 ipdecode 预译码之后，会判断出 call 指令，并且将该指令压入到 IB 级的 ras 中
     - 12entry->C910 最多支持 12 级函数嵌套

8. Loop Buffer 短循环加速器：降低功耗&减少 bubble

   当一个循环体指令比较少，且循环次数比较多的时候，我们如果每次取指都从 i-cache，通过 IFU 取指，则会浪费很多的功耗；此时可以将循环体内的指令放到 IBuf(容量为 32B)中，每次直接从 IBuf 中获取指令送到 ID，其操作流程如下：

   - 检测后向跳转指令->判断短循环
   - 将循环指令装载到 IBuf 中，等待 ibuf 清空
   - 关闭 i-cache 和 IF，从 IBuf 中输出指令到 ibuf 再到 ID

   IFU 每次最少取 4 条指令(避免取指成为处理器瓶颈)、ID 可以并行译码 3 条指令(C910 是 3 发射)

9. Indirect BTB 间接跳转目标缓冲器

   - 256 entry
   - each entry 23 bits(valid: 1 bit, privilege mode: 2 bits, target pc lsb: 20 bits)

   > 间接跳转分支指令相较于条件分支、绝对分支出现的频率较低，间接跳转分支指令大量应用于<u>函数指针</u>，或者<u>switch-case</u>语句，随着面向对象语言程序、动态链接库以及虚拟机等技术的普遍应用，在程序运行过程中动态地确定转移目标地址的特性越来越显著，间接跳转分支指令的使用将越来越频繁。因此， 间接跳转分支指令的预测准确度对处理器性能的影响将越来越大。

   | 预测器       | 预测的指令                             |
   | ------------ | -------------------------------------- |
   | L0 BTB       | 条件分支、绝对分支                     |
   | BTB          | 条件分支、绝对分支、return             |
   | RAS          | call、return                           |
   | Loop Buffer  | 短循环                                 |
   | Indirect BTB | 除了 call，return 之外的所有 jalr 指令 |

   在 C910 中，若不使用间接跳转目标缓冲器，需要读寄存器后才能得到间接跳转指令的跳转结果进行 PC 的重定向，而这至少需要 8 个时钟周期，在预测正确就可以在 3 个时钟周期获得指令执行的结果

# 香山处理器中的分支预测设计

# Todo List

- [x] 补充 BP 的历史：BP 是怎么提出的，是为了解决什么问题，它带了了什么好处，它怎么发展到今天的

# TAGE-Predictor
![TAGE Hardware](/Users/fujie/Pictures/typora/tage-hardware.jpeg)

TAGE 全名为 Tagged Geometric History Branch Predictor，在 2006 年 Journal of Instruction Level Parallelism 上发表的一篇工作中被提出，并且连续赢下了两年的 Competition Branch Predictor 比赛. 其特点有：
1. 混合式预测器
2. 根据不同长度的历史分支序列进行预测，并且对其准确率进行评估，选择准确率最高的

相比于传统的混合式预测器，有如下优点：
1. 每个表项entry添加了tag：出现aliasing（多个pc指向同一个entry）,当BHT短的时候，影响更大。
2. 分支历史表长度呈几何级数变化：
3. 每个表项添加了useful计数器，避免了整个分支历史表填充时间太长。


> 预测器数目超过8个之后，对准确度影响不大

Nanhu TAGE 没周期预测2个指令

# References

1. [Seznec, André. “A new case for the TAGE branch predictor.”2011 44th Annual IEEE/ACM International Symposium on Microarchitecture (MICRO)(2011): 117-127.](https://link.zhihu.com/?target=https%3A//dl.acm.org/doi/abs/10.1145/2155620.2155635)
2. [Zhao, Jerry. “SonicBOOM: The 3rd Generation Berkeley Out-of-Order Machine.” (2020).](https://link.zhihu.com/?target=https%3A//carrv.github.io/2020/papers/CARRV2020_paper_15_Zhao.pdf)
3. [从TAGE看分支预测器设计](https://blog.maxxsoft.net/index.php/archives/129/)
4. [TAGE分支预测器算法](https://blog.csdn.net/weixin_41418994/article/details/103905122)
5. [TAGE预测器的设计原理与在Sonic BOOM中的实现](https://zhuanlan.zhihu.com/p/397105511)

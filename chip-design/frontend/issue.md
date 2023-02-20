> 比较各个处理器微架构的发射路数、对应的性能和硬件复杂度等数据

需要比较的架构有：

- [ ] ARM M Series
- [x] ARM A Series
- [ ] ARM 鲲鹏 Series
- [x] RISC-V 香山
- [ ] RISC-V 玄铁

Other Tasks:

- [ ] 总结不同的跳转指令需要的预测器类型
- [ ] 总结 issue 和 bp 的发展历史

TBD: 总结不同类型的跳转指令，如：直接跳转、寄存器直接跳转、条件跳转、普通指令。\_部分 jalr 指令的跳转地址相对固定，仅靠 FTB 就足以达到很高的预测准确率；函数返回是 jalr 指令中较为常见的应用场景，它和函数调用指令有着显著的配对性，可以使用具有栈结构的 RAS 进行预测；不符合以上特征的 jalr 指令交由 ITTAGE 预测。

# 鲲鹏 920(2019)

> 鲲鹏 920 是目前业界领先的 ARM-based 处理器。该处理器采用 7nm 制造工艺，基于 ARM 架构授权，由华为公司自主设计完成。
> 通过<u>**优化分支预测算法**</u>、提升运算单元数量、<u>**改进内存子系统架构**</u>等一系列微架构设计，大幅提高处理器性能。典型主频下， SPECint Benchmark 评分超过 930，超出业界标杆 25%。同时，能效比优于业界标杆 30%。鲲鹏 920 以<u>**更低功耗为数据中心**</u>提供更强性能。

## Issue

1. 7nm, 基于 ARM 架构授权
2. 主频可达 2.6GHz，单芯片最高支持 64 核
3. 业内首个内置直出 100GE 网络能力的通用处理器

| 组件     | 参数                               |
| -------- | ---------------------------------- |
| ISA 架构 | ARM v8.2                           |
| 核心数   | 最高 64 核                         |
| 频率     | 2.6G,3.0G                          |
| 内存     | 8 DDR4 通道                        |
| L1 $     | 64KB                               |
| L2 $     | 512KB                              |
| L3 $     | 1MB, core shared                   |
| IO       | PCIe 4.0, CCIX, 100G, SAS/SASA 3.0 |
| 功耗     | 180W                               |
| 制成     | TSMC 7nm                           |

# Cortex-A76(2018+)

> 4-way superscalar out-of-order processor

![A76 block diagraph](/Users/fujie/Pictures/typora/a76_block_diagram.svg.webp)
![A76 micro architecture](/Users/fujie/Pictures/typora/a76_micro_arch.webp)

## Issue

| 组件     | 参数                                             |
| -------- | ------------------------------------------------ |
| Renaming | $\leq$ 4 mOP/cycle , ROB 128 entry               |
| Issue    | 8 issue backend , issue queue 120 entry          |
| EXE      | 4 int pipeline, 2 float pipeline, 2 lsu pipeline |

## Branch Prediction

| 组件         | 参数                                                     |
| ------------ | -------------------------------------------------------- |
| NanoBTB      | 16 entry                                                 |
| MicroBTB     | 64 entry                                                 |
| Main BTB     | 6K entry                                                 |
| L1 $         | 64KB, 4 way set associative, 16B/cycle(4~8 instructions) |
| l2 $         | 256K(1 bank) or 512K(2 banks)                            |
| Return Stack |                                                          |
| decode width | 4                                                        |

![a76_branch_predictor](/Users/fujie/Pictures/typora/a76bp.png)

The pipeline is 13 stages with an <u>11-cycle misprediction penalty</u>:

1. Up to 16B instructions are fetched from L1 I$, with BP's help.
2. Each cycle 4 instructions can be decoded into 4 mOP in the decode stage.
3. In rename stage, up to 4 mOP can be renamed in each cycle, break into 8 uOP.

## A76 References

1. [From A76 to A78](https://blog.csdn.net/weixin_45264425/article/details/128360519)
2. [Cortex-A76 Microarchitectures from ChipWiki](https://en.wikichip.org/wiki/arm_holdings/microarchitectures/cortex-a76#:~:text=The%20Cortex%2DA76%20is%20a%20complex%2C%204%2Dway%20superscalar,11%2Dcycle%20branch%20misprediction%20penalty)

# Intel Skylake

TBD

## Skylake References

1. [Intel Skylake Microarchitectures from ChipWiki](<https://en.wikichip.org/wiki/intel/microarchitectures/skylake_(server)#Architecture>)

# 香山处理器-南湖架构(2022)

![nanhu block diagraph](/Users/fujie/Pictures/typora/nanhu_block_diagraph.png)

## Issue

| 组件              | 参数                 |
| ----------------- | -------------------- |
| rename width      | 6                    |
| ROB               | 256 entry            |
| Physical Register | 192(int), 192(float) |
| Physical RF       | 192\*64 bits, 14R8W  |
| Load Queue        | 80                   |
| Store Queue       | 64                   |

> 在指令的发射阶段，涉及到的主要模块是保留站 Reservation Station（或称为发射队列)，主要涉及到的操作有**入队、选择、读数据、出队、<u>监听指令的写回，负责等待指令的唤醒</u>**。

## Branch Prediction

![Nanhu fontend](/Users/fujie/Pictures/typora/nanhu_frontend.png)
![Nanhu branch prediction](/Users/fujie/Pictures/typora/nanhu_bpu.svg)

| 组件           | 参数           |
| -------------- | -------------- |
| pipeline stage | 11             |
| decode width   | 6              |
| L1 $           | 64K, 4/8 way   |
| L2 $           | 512K/1M, 8 way |
| L3 $           | 2M~8M, 8 way   |

**混合预测器**

| 组件   | 参数    |
| ------ | ------- |
| NLP    | 1 cycle |
| FTB    | 2 cycle |
| TAGE   | 2 cycle |
| SC     | 3 cycle |
| ITTAGE | 3 cycle |

**BTB->FTB**(Fetch Target Buffer)：对每一条指令进行预测->对预测块(几条指令)进行预测

## Nanhu References

1. [XiangShan Official docs](https://xiangshan-doc.readthedocs.io/zh_CN/latest/arch/)

# 玄铁 C910

## Issue

> 玄铁 C910 为**乱序 8 发射处理器**，即每周期最多可以发射 8 条指令到执行单元中进行执行。

Issue 中主要解决**RAW**数据冒险：该指令所需要的源操作数是否可以从 RF(Register File)或者 Forward 得到.jjj

1. C910 的(IQ)Issue Queue 每个周期最多载入 2 条数据(每个 IQ 有 2 个写入端口)，一共有 7 条 IQ

   | IQ   | instructions Type  |
   | ---- | ------------------ |
   | aiq0 | ALU, IDIV, special |
   | aiq1 | ALU, IMUL          |
   | biq  | branch             |
   | lsiq | load, store        |
   | sdiq | store data         |
   | viq0 | vector, float      |
   | viq1 | vector, float      |

2. 指令队列中每条指令有以下标志位：
   - agevec 年龄向量：表示 IQ 中指令的优先级，当指令具有发射条件时，根据 agevec 优先发射优先级高的
   - frz：指令发射之后，将 frz 置 1 避免重复发射

## Branch Prediction

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

## C910 References

1. [玄铁 C910 微架构——指令发射](https://zhuanlan.zhihu.com/p/556158655)

# Branch Prediction History

TBD: need some pictures in PPT.

- IBM 7030 Stretch(1950s)

  - pre execute: unconditional branch & conditional branch with index register
  - use **lookahead unit** for recover, the misprediction recover time is very long.

> Microprogrammed computers are popular from 1969s to 1980s, <u>generally didn't relay on BP</u>.

- 2-bit predictor(1977): used in National Lab S-1 supercomputer in 1977; CDC in 1979.
- Burroughs B4900(1982): - microprogrammed machine, COBOL - pipelined and use BP - **4 state branch prediction**: **93%** hit rate
  > early in order RISC processors don't implement BP, they add <u>branch delay slot</u>, they just predict _branch is not taken_. Like MIPS R2000, R3000, SPARC.

> 随着流水线超标量处理器的引入,分支预测变得更加重要。

- <u>**一位或简单的双模态预测器**</u>: Intel Pentium in 1993、 DEC Alpha 21064 in 1994、 MIPS R8000 和 IBM POWER 系列
- NLP: Alpha 21264 in 1998, by a combined _local predictor_ and _global predictor_, where the combining choice is made by a _bimodal predictor_.

## Branch Prediction References

1. [Branch Preditor on WiKipedia](https://en.wikipedia.org/wiki/Branch_predictor)

# Final Compare

|                | M55                                                          | A76                                            | C910                                                         | Nanhu                                                        |
| -------------- | ------------------------------------------------------------ | ---------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 是否有分支预测 | ❌                                                            | ✅ 16 nano-BTB,64 entry-MicroBTB,6k MainBTB,RAS | ✅ L0 BTB, BTB, IndBTB(256 entry), Loop Buffer(32B), RAS      | ✅uBTB: 1 cycle; TAGE: 2 cycle，主预测器，条件分支; SC: 3 cycle，校正预测器，对 TAGE 取反; ITTAGE: 3 cycle，jalr 间接跳转指令; RAS：call/return 指令 |
| 上市年份       | 2020+                                                        | 2018+                                          | 2019                                                         | 2022                                                         |
| 流水线级数     | 4, 5                                                         | 13+                                            | 9~12                                                         | 11                                                           |
| Issue 路数     | 1, 2(16 bit inst)                                            | 8                                              | 8                                                            | 6                                                            |
| 最大核心数     | 4                                                            | 8                                              | 4                                                            | 2                                                            |
| 处理器特点     | 顺序四级流水线处理器：应用于微控制器、超级功耗芯片等嵌入式系统 | 乱序 4 路超标量处理器：应用于低功耗移动场景    | 主要面向对性能要求严格的边缘计算领域, 如边缘服务器,移动智能终端、5G 基站 | 对标 ARM A-76，工业控制、汽车、通信                          |
| ISA            | ARM v8.1-M                                                   | ARM v8.2                                       | RISC-V64GCV                                                  | RISC-V GCBK                                                  |
| 功耗           | ❓                                                            | 750 mW                                         | ❓                                                            | < 5W                                                         |
| 制程           | 5 nm                                                         | 5 nm                                           | 12 nm                                                        | 14 nm                                                        |

| 性能表现 | | | | |
| 预计功耗 | | | | |
Beside： 鲲鹏 920 Max Power 180W

## References

1. [ARM M55 Documentation](https://developer.arm.com/documentation/101051/0101/Technical-overview/Cortex-M55-processor-components/Cortex-M55-processor-core)
2. [ARM M55 Wikichip](https://en.wikichip.org/wiki/arm_holdings/microarchitectures/cortex-m55)
3. [ARM A76 Documentation](https://developer.arm.com/Processors/Cortex-A76)
4. [ARM A76 Wikichip](https://en.wikichip.org/wiki/arm_holdings/microarchitectures/cortex-a76)
5. [ARM M3 Documentation]()
6. [ARM M3 Wikichip](https://en.wikichip.org/wiki/samsung/microarchitectures/m3)
7. [C910 用户手册](https://bbs.16rd.com/thread-581442-1-1.html)
8. [玄铁 C910 总览-山东大学 RISCV 实验室](https://zhuanlan.zhihu.com/p/581155611)
9. [Seznec, André. “A new case for the TAGE branch predictor.”2011 44th Annual IEEE/ACM International Symposium on Microarchitecture (MICRO)(2011): 117-127.](https://link.zhihu.com/?target=https%3A//dl.acm.org/doi/abs/10.1145/2155620.2155635)
10. [Zhao, Jerry. “SonicBOOM: The 3rd Generation Berkeley Out-of-Order Machine.” (2020).](https://link.zhihu.com/?target=https%3A//carrv.github.io/2020/papers/CARRV2020_paper_15_Zhao.pdf)
11. [从TAGE看分支预测器设计](https://blog.maxxsoft.net/index.php/archives/129/)
12. [TAGE分支预测器算法](https://blog.csdn.net/weixin_41418994/article/details/103905122)
13. [TAGE预测器的设计原理与在Sonic BOOM中的实现](https://zhuanlan.zhihu.com/p/397105511)
14. [Reinman G, Austin T, Calder B. A scalable front-end architecture for fast instruction delivery[J]. ACM SIGARCH Computer Architecture News, 1999, 27(2): 234-245](https://dl.acm.org/doi/10.1145/307338.300999)

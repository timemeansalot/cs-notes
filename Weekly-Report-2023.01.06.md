# Weekly-Report 2023.01.06

## PC 的足迹-计算机发展史

### Intel 4004

- 4 位寄存器和数据总线
- 12 位地址总线, 最多寻址 4KB 内存
- 最高主频 740KHz：一般作为 MCU 使用, 无需配备操作系统(放不下 😂)
- 例如 Busicom 公司生产的计算器 141-PF

指令特点

- 46 条指令
- 每条指令 8 位
- 每条基本指令执行 8 个周期、复杂指令执行的周期更长
  [4004 指令集手册](https://web.archive.org/web/20110601032753/http://www.intel.com/Assets/PDF/DataSheet/4004_datasheet.pdf)

TODO: 添加指令集手册相关到图片

### Intel 8086

- 16 位寄存器和 16 位数据总线
- 20 位地址总线, 可寻址 1MB 内存
- 主频 5~10MHz
- 1981 年, IBM PC 机发售, 搭载 Intel 8088 处理器(8 位数据总线版本的 8086)以及微软的 MS-DOS 操作系统

1. 约 256 条指令，指令长度 1 ～ 6 字节
2. 通过指令队列实现总线接口单元和执行单元的并发工作(流水)

[8086 指令集手册](http://matthieu.benoit.free.fr/cross/data_sheets/8086_family_Users_Manual.pdf)

TODO：page16 pictures

## ARM(Advanced RISC Machine)

1. CISC vs RISC: RISC ISA 通常一条指令完成简单独立的一种运算或控制，指令长度固定，格式较为统一。而 CISC ISA 指令的功能要复杂的多，指令长度可变，格式复杂。由于 RISC 指令编码和功能上非常有利于硬件实现，**处理器发展到今天，不论 X86 还是 ARM，其硬件执行核心已经都是 RISC 架构了**，而 X86 多了一层将 CISC 指令 翻译成 RISC 类微指令的步骤，因此译码部分不但增加了额外的流水线级，实现上也复杂许多，这也是 X86 处理器功耗大于同级别的 RISC 处理器的一个重要因素。

### Cortex-A77

1. Cortex A77 面向移动高性能领域，采用 ARMv8.2 64 位指令集架构。
2. 没有在频率控制上向 INTEL 看齐，而更加追求单位功耗下的性能比
3. 流水线结构没有太大变化，还是标准的**physical register Out-of-Order machine**
4. 前端流水线，最重要的改动之一就是增加了 Mop cache:
   - 假设其宽度和 decode 数目一致，1.5K 的 entry 就可以存储 9 千条 32 位指令，应该可以覆盖大多数移动领域的应用场景
   - 在 Mop cache warmup 后，指令可以不经 Icache 通路，直接从 Mop cache 发送到 decode 级，这样整个 fetch 单元都可以进入低功耗状态
   - 同时其中存储指令译码后的信息，包括分支和循环的预测结果，可以实现 zero cycle 的 hardware loop，进一步提高了循环的执行效率
   - Mop 取指相当于减少了 fetch 流水线的长度，这样出现 branch misprediction 之后，如果新的 target 也在 Mop 中，flush 流水线的 penalty 也会降低不少
5. Branch prediction 是另一个重点优化的手段
   - 拓宽了 branch prediction 的 bandwidth 到 64B，这样理论上可以同时预测 16 条 32 位指令的分支结果
   - 大幅增加了 BTB 的 size
6. Decode 级主要是增加了 50%的 dispatch 宽度:
   - 6 条指令的并发理论上可以提供更大的并行执行能力。随之 ROB 的 entry 数也增加到 160 个。
   - ROB 的 entry 数也增加到 160 个
   - 加快 renaming table 在 branch misprediction 后的 update 速度
7. Load-store 流水线，同样采用了统一的 Issue Queue。可以看到 A77 有 2 个 address 的 path 和 2 个 st-date 的 path，可以同时执行 2 条存储指令。组合可能是 2 条 load，2 条 store，1 load+1 store。
   - 这里 ARM 着重介绍了其**pre-fetching**的机制。数据的 prefetch 可以很好的隐藏系统内存的访问延迟，在高性能处理器上非常重要。

总结：

1. _通用处理器的微架构已经趋于稳定，大的方面玩不出新的花样了_，都在细节上做进一步的提升，比如更宽的发射通路，更多的执行单元，更大的预测器等。这给了后来者一个很好的追赶机会。
2. ARM 的移动处理器的设计正在逐步趋同于 INTEL 和 AMD 在桌面处理器的结构，除了核心频率较低以外，其他的指标已经差距不大了，很多 X86 上独有的技术正在逐步出现在 ARM 的设计中
3. 工艺尺寸的缩小带来功耗密度的大幅增加，成为制约核心频率的重要因素。由于移动端散热条件更为苛刻，可能通常的应用会采用多核心工作在较低频率，而对于游戏等大型应用，会提高单核的频率，其他核心降频或进入低功耗(暗硅），保持整个芯片的功耗在可控范围内。

参考资料：
[移动的王者：深入分析 ARM 最强处理器 Cortex A77](https://zhuanlan.zhihu.com/p/76045830)

## RISC-V

### Nutshell

### XS

### 910

### Chipyard(if have time)

## 知识点补充

### 分支预测(Branch prediction, BP)

在ARM中,使用全局分支预测器,该预测器由转移目标缓冲器(Branch Target Buffer,BTB)、全局历史缓冲器(Global History Buffer,GHB) MicroBe,以及 Return Stack 组成

1. 分支预测器的组件
    - 静态预测器(Static Predictor, SP)：分支在BTB中找不到记录时,可以使用 The Static Predictor(静态预测器)。人们将分支指令的执行情况做了大量的统计,从中总结出一些特征,并将这些特征总结为一些固定的策略,这就是静态预测器.
    - 转移目标缓冲(Branch Target Buffer, BTB)：其结构为*指令地址+预测的跳转地址+是否跳转*
    - 返回地址栈(Retern Stack, RS)：在处理函数调用时，将返回地址存储到RS中，当函数返回时，从RS中取PC
      ![BTB结构](https://s2.loli.net/2023/01/04/B9laq4M5scAoWXp.png)


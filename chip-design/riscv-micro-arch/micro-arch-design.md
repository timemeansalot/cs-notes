> RV 32I 处理器微架构设计

设计步骤：

1. data path（数据通路），包括：ALU、memory、registers 和 multiplexer 等
2. control unit（控制单元）：输出“数据通路“部件的**控制信号**.

单周期微架构有如下三个缺点：

1. instruction 和 data 分开存储
2. 处理器的速度由**速度最慢**的部件决定，因此加法指令的速度跟 load 指令的速度是一样的
3. 需要三个加法器

多周期处理器在单周期处理器上的改进：不同的指令有不同的周期，如加法指令只需要 1 个周期，分支指令需要 3 个周期。  
多周期处理器在单周期处理器上的具体改进有：

1. 由于读 instruction 和 data 是在不同的时刻，所以其实可以将二者存放到一起，在不同的时刻读取
2. 在各个模块之间添加寄存器，并且更改控制器的逻辑（由简单的组合逻辑电路变成 FSM）
3. 一个在 ALU 里，两个用于计算 PC：复用加法器

流水线处理器：将单周期处理器的一条指令拆分成 5 个阶段（IF, ID, EXE, MEM, WB)  
需要插入 4 个寄存器(下降沿写入、上升沿读出)，从而将流水线分成 5 个阶段.  
流水线处理器需要处理以下问题：

1. WB 的地址在 ID 阶段得到，在 WB 的时候的地址是后续第三条指令的地址，因此需要将 ID 阶段的地址顺着流水线传递到 WB 阶段。同理 PC 也需要顺着流水线传递。
2. control signal 也需要顺着流水线传递
3. 处理 data hazard: forwarding & stall
4. 处理 control hazard: 增加 pc 错误的判断信号、增加流水线刷新的信号

![single-to-pipeline](/Users/fujie/Pictures/typora/riscv-micro-arch/single-to-pipeline.jpg)

# ToastCore

[ToastCore Website](https://toast-core.readthedocs.io/en/latest/pipeline_details.html)
![toast_architecture](/Users/fujie/Pictures/typora/riscv-micro-arch/toast_architecture.jpg)

## IF

![instruction fetch](/Users/fujie/Pictures/typora/riscv-micro-arch/toast_IF_simple.jpg)

以下两种跳转指令发生的时候，需要处理流水线刷新

1. conditional branch: `Branch PC Dest`, `Branch Taken`
2. unconditional jump: `Jump PC Dest`, `Jump Taken`

## ID

1. Decoder: used to calculate imm, regfile addr, ALU control, signals, Branch Gen(used for unconditional jump), memory signal.  
   ID is made of **combinational logic**.
2. Register File: make of 32\*32 registers, implemented by RAM.  
   2 read ports, 1 write ports.

## EXE

1. ALU, used to do calculation for: Arithmetic, Logical, Shifts, Sets.  
   Make of **combinational logic**.
2. Branch Gen: final decide if conditional branch is taken or not, and calculate the branch address.

# RV32IM Pipeline Implementation

[RV32IM Pipeline Implementation Website](https://cepdnaclk.github.io/e16-co502-RV32IM-pipeline-implementation-group1/)  
[Github Repository](https://github.com/cepdnaclk/e16-co502-RV32IM-pipeline-implementation-group1)

![toast_architecture](/Users/fujie/Pictures/typora/riscv-micro-arch/forwarding_datapath.png)

## Requirements

1. iVerilog for compile and simulate
2. gtkWave for observing the waveform

## Design Notes
1. Pipeline Registers: 
    - 一共有4组带复位的同步寄存器，将流水线分成5个stage
    - 每组寄存器在时钟上升沿写入数据，在下降沿读出数据

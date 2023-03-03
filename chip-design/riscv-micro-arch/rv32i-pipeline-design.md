> RISC-V 32I 5 级流水线设计

# 1 Control Logic & ISA 分析

![instructions_and_control_signals](/Users/fujie/Pictures/typora/riscv-micro-arch/instructions_and_control_signals.png)

## 1.1 IMM_SEL

1. 6 种类型的指令，其中 R 类型的指令不会有立即数，I 类型的指令会有 3 种类型的立即数，因此立即数共有 7 种类型，如下表所示:
2. Immediate Value Generation Unit 用于将指令种的立即数根据规则拓展为 32bits

![imm_gen](/Users/fujie/Pictures/typora/riscv-micro-arch/imm_gen.png)

| Immediate Type | IMM_SEL[2] | IMM_SEL[1] | IMM_SEL[0] |
| -------------- | ---------- | ---------- | ---------- |
| U              | 0          | 0          | 0          |
| J              | 0          | 0          | 1          |
| S              | 0          | 1          | 0          |
| B              | 0          | 1          | 1          |
| I_signed       | 1          | 0          | 0          |
| I_unsigned     | 1          | 0          | 1          |

| I_shift        | 1          | 0          | 1          |
![imm_format](/Users/fujie/Pictures/typora/riscv-micro-arch/imm_format.png)

TBD: 为什么会有 I_shift 类型的 imm 存在?

## 1.2 ALU operation

ALU 有两个输入数据，其中：

1. rs1 可以是 PC 或者是寄存器的值
2. rs2 可以是立即数或者是寄存器的值

> **当选择了 bypass forwarding 的时候，ALU 的 rs1 和 rs2 的输入数据变成了 4 种可能，对应的 2x1 选择起也升级成了 4x1 选择器**.

![alu](/Users/fujie/Pictures/typora/riscv-micro-arch/alu.png)

因此分别使用 OP1_sel, OP2_sel 来选择 ALU 对应的输入数

ALU 一共可能进行 18 种运算，因此需要 5bit 的选择信号 ALU_OP 来指定 ALU 需要运算的类型，如下表所示：

| ALU operation | ALU_OP[4] | ALU_OP[3] | ALU_OP[2] | ALU_OP[1] | ALU_OP[0] |
| ------------- | --------- | --------- | --------- | --------- | --------- |
| ADD           | 0         | 0         | 0         | 0         | 0         |
| SUB           | 0         | 0         | 0         | 1         | 0         |
| SLL           | 0         | 0         | 1         | 0         | 0         |
| SLT           | 0         | 1         | 0         | 0         | 0         |
| SLTU          | 0         | 1         | 1         | 0         | 0         |
| XOR           | 1         | 0         | 0         | 0         | 0         |
| SRL           | 1         | 0         | 1         | 0         | 0         |
| SRA           | 1         | 0         | 1         | 1         | 0         |
| OR            | 1         | 1         | 0         | 0         | 0         |
| AND           | 1         | 1         | 1         | 0         | 0         |
| MUL           | 0         | 0         | 0         | 0         | 1         |
| MULH          | 0         | 0         | 1         | 0         | 1         |
| MULHU         | 0         | 1         | 0         | 0         | 1         |
| MULHSU        | 0         | 1         | 1         | 0         | 1         |
| DIV           | 1         | 0         | 0         | 0         | 1         |
| DIVU          | 1         | 0         | 1         | 0         | 1         |
| REM           | 1         | 1         | 0         | 0         | 1         |
| REMU          | 1         | 1         | 1         | 0         | 1         |

## 1.3 Branch_Jump

![bj_unit](/Users/fujie/Pictures/typora/riscv-micro-arch/bj_unit.png)

| BranchType | BRANCH_JUMP[2] | BRANCH_JUMP[1] | BRANCH_JUMP[0] |
| ---------- | -------------- | -------------- | -------------- |
| BEQ        | 0              | 0              | 0              |
| BNE        | 0              | 0              | 1              |
| NO         | 0              | 1              | 0              |
| J          | 0              | 1              | 1              |
| BLT        | 1              | 0              | 0              |
| BGE        | 1              | 0              | 1              |
| BLTU       | 1              | 1              | 0              |
| BGEU       | 1              | 1              | 1              |

**BRANCH_JUMP**信号用于输入给<u>Branch and Jump Detection Unit</u>，该单元用于判断跳转是否发生，输出 1bit 的 PC_SEL 信号，用于控制 PC 更新时选择 PC+4 还是 ALU 计算得到的 PC.  
![bj_truth_table](/Users/fujie/Pictures/typora/riscv-micro-arch/bj_truth_table.png)
PC 在上升沿更新，reset 时 PC 置为-4.

![pc_reg](/Users/fujie/Pictures/typora/riscv-micro-arch/pc_reg.png)

## 1.4 Read Write

RV-32I 中一共有 5 种类型的 load 指令、3 种类型的 store 指令，再加上非访存指令，一共有 9 种情况，需要 4bit 来区分

| Load_Store Type | READ_WRITE[3] | READ_WRITE[2] | READ_WRITE[1] | READ_WRITE[0] |
| --------------- | ------------- | ------------- | ------------- | ------------- |
| No load/store   | 0             | 0             | 0             | 0             |
| LB              | 1             | 0             | 0             | 0             |
| LH              | 1             | 0             | 0             | 1             |
| LW              | 1             | 0             | 1             | 0             |
| LBU             | 1             | 1             | 0             | 0             |
| LHU             | 1             | 1             | 0             | 1             |
| SB              | 1             | 0             | 1             | 1             |
| SH              | 1             | 1             | 1             | 0             |
| SW              | 1             | 1             | 1             | 1             |

## 1.5 WB_SEL

写入到寄存器种的数据来源一共有 4 种：

1. 来自 ALU 的结果：AUIPC, I type and R type.
2. 来自 mem: load.
3. 来自立即数：LUI
4. 来自 PC: jal, ~~jalr is I type~~

只有当`REG_WRITE_EN==1`的时候，才可以往 regfile 里写入

| Writeback Source      | WB_SEL[1] | WB_SEL[0] |
| --------------------- | --------- | --------- |
| ALU result            | 0         | 0         |
| Data from data memory | 0         | 1         |
| Immediate value       | 1         | 0         |
| PC + 4                | 1         | 1         |

![reg_file](/Users/fujie/Pictures/typora/riscv-micro-arch/reg_file.png)

# 2 解决 Hazard

## 2.1 Data Hazard: 通过 forwarding unit 来解决，需要添加的硬件单元如下：

1. 在需要提前得到数据的功能部件如 ALU 之前添加复用器，将提前算好的数据传入到复用器里
2. 在 forwarding unit 里计算复用器的选择信号 sel, forwarding unit 判断是否需要 forwarding 的原则是：在 decode 阶段得到源操作数的地址，将该地址与 EXE、MEM、和 WB 的目的地址比较，如果匹配则可以进行 forwarding.

下面以 EXE->EXE 的 forwarding 为例：

1. 在 ALU 之前添加复用器，并且将 ALU 的输出结果输入到该复用器
2. 在 forwarding unit 里根据规则计算该复用器的选择信号 sel，因此当存在匹配的 data hazard 的时候，该 forwarding 机制将会被激活

## 2.2 Control Hazard

1. Flush pipeline when branch is taken, reset all IF/ID register, ID/EXE register.



# 最终的数据通路

![forwarding_datapath](/Users/fujie/Pictures/typora/riscv-micro-arch/forwarding_datapath.png)






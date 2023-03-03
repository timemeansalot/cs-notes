> ALU 设计

RV-32IM 涉及到的指令如下图所示：
![RV-32IM Instructions](/Users/fujie/Pictures/typora/riscv-micro-arch/instructions_and_control_signals.png)
![RV-32IM Instructions](/Users/fujie/Pictures/typora/riscv-micro-arch/rv32I-referenceCards.jpg)
ALU 可能执行的操作一共有如下 18 种：

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

其中我需要关心的 ALU 操作有如下 8 种：

| ALU operation | Related ISA | Description              |
| ------------- | ----------- | ------------------------ |
| SLT           | SLT, SLTI   | set less than            |
| SLTU          | SLTU, SLTIU | set less than unsigned   |
| SLL           | SLL, SLLI   | shift left logical       |
| SRL           | SRL, SRLI   | shift right logical      |
| SRA           | SRA, SRAI   | shift right arithmatical |
| OR            | OR, ORI     | or                       |
| AND           | AND, ANDI   | and                      |
| XOR           | XOR, XORI   | xor                      |

可以区分为如下三个类别：

1. set: SLT, SLTU
2. shift: SLL, SRL, SRA
3. logical: OR, AND, XOR  
   **Very intuitive, don't need much optimization**

![RV-32IM Instructions](/Users/fujie/Pictures/typora/riscv-micro-arch/addi.jpg)
![RV-32IM Instructions](/Users/fujie/Pictures/typora/riscv-micro-arch/shiftISA.jpg)
![RV-32IM Instructions](/Users/fujie/Pictures/typora/riscv-micro-arch/R-typeISA.jpg)

## SLT , SLTU

| ALU operation | Related ISA | Description            |
| ------------- | ----------- | ---------------------- |
| SLT           | SLT, SLTI   | set less than          |
| SLTU          | SLTU, SLTIU | set less than unsigned |

`SLT`和`SLTU`需要 ALU 做如下工作：

1. 判断两个软操作数的大小是否满足：$rs1 < rs2$
2. 若判断为真则返回 0x00000001，否则返回 0x00000000  
   计算结果 0 和 1 的时候，可以直接对 sum[31]做 0 扩展得到 32 位的结果

**判断大小**需要以下步骤：

1. 复用 ALU 中的减法器，计算 rs1-rs2
2. 结合 overflow 和 sum[31]判断大小:

   |     | signed | unsigned |
   | --- | ------ | -------- |
   | <   | N⊕V    | $\bar C$ |

   - N: negative=sum[31]
   - V: overflow

![compare](/Users/fujie/Pictures/typora/riscv-micro-arch/compare.jpg)

## SLL, SRL, SRA

RV-32IM 需要实现的移位操作不包括循环移位，只包括：<u>逻辑左移</u>、<u>逻辑右移</u>和<u>算数右移</u>。  
若使用**移位寄存器**来实现移位，每个周期移位是固定的，因此需要多个周期才可以完成移位操作。
![shift01](/Users/fujie/Pictures/typora/riscv-micro-arch/shift01.png)
上图是一个由 4 个 D 触发器构成的简单向右移位寄存器，数据从移位寄存器的左端输入，每个触发器的内容在时钟的上升沿将数据传到下一个触发器。

在 ALU 种需要多数据进行多位移位的操作，采用移位寄存器一次只能移动移位，效率太低；**桶形移位器**采用组合逻辑的方式来实现同时移动多位，在效率上优势极大。因此桶形移位器常被用在于 ALU 中实现移位。

![barrlShifter](/Users/fujie/Pictures/typora/riscv-micro-arch/barrlShifter.jpg)

1. din：待移位待输入数据
2. shift: 移位待位数，有效范围为$[0, \log N-1]$
3. Left/Right: 左移或者右移
4. Arith/Logic：算数右移/逻辑右移
5. dout：移位后的数据

以 4bits din 的 barrlShifter 为例，其 Schematic 如下：

![barrlShift4bits](/Users/fujie/Pictures/typora/riscv-micro-arch/barrlShift4bits.jpg)

- 对于 Nbits 的输入数据，其需要的选择器一共有$\log N$层，每一层共有 N 个选择器，其中第一层选择是否移位 1bit，第二层选择是否移位 2bits，...。
- 对每一个 4 选 1 选择器，其 00 和 10 输入选择未移位后的数据；01 选择右移的数据、11 选择左移的数据。

## OR, AND, XOR

| ALU operation | Related ISA | Description |
| ------------- | ----------- | ----------- |
| OR            | OR, ORI     | or          |
| AND           | AND, ANDI   | and         |
| XOR           | XOR, XORI   | xor         |

针对**逐比特**的 or、and 和 xor，RV-32IM 中需要使用 32 个或门、与门和异或门来实现。

|      | or  | and | xor |
| ---- | --- | --- | --- |
| pmos | 3   | 2   | 4   |
| nmos | 3   | 2   | 4   |

![andgate](/Users/fujie/Pictures/typora/riscv-micro-arch/andgate.png)
![orgate](/Users/fujie/Pictures/typora/riscv-micro-arch/orgate.png)
![xorGate](/Users/fujie/Pictures/typora/riscv-micro-arch/xorGate.png)

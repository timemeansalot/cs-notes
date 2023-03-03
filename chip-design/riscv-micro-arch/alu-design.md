> ALU 设计

RV-32IM 涉及到的指令如下图所示：
![RV-32IM Instructions](/Users/fujie/Pictures/typora/riscv-micro-arch/instructions_and_control_signals.png)
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

其中我需要关心的有如下 8 种：

| ALU operation | Description              | ALU_OP[4] | ALU_OP[3] | ALU_OP[2] | ALU_OP[1] | ALU_OP[0] |
| ------------- | ------------------------ | --------- | --------- | --------- | --------- | --------- |
| SLT           | set less than            | 0         | 1         | 0         | 0         | 0         |
| SLTU          | set less than unsigned   | 0         | 1         | 1         | 0         | 0         |
| SLL           | shift left logical       | 0         | 0         | 1         | 0         | 0         |
| SRL           | shift right logical      | 1         | 0         | 1         | 0         | 0         |
| SRA           | shift right arithmatical | 1         | 0         | 1         | 1         | 0         |
| OR            | or                       | 1         | 1         | 0         | 0         | 0         |
| AND           | and                      | 1         | 1         | 1         | 0         | 0         |
| XOR           | xor                      | 1         | 0         | 0         | 0         | 0         |

可以区分为如下三个类别：
1. set: SLT, SLTU
2. shift: SLL, SRL, SRA
3. logical: OR, AND, XOR  
    **Very intuitive, don't need much optimization**

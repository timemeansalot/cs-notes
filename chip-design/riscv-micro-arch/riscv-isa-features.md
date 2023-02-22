> RISC-V 指令集的学习笔记

# RISC-V 指令集特点

1. 六种指令格式，全部 32 位
   ![risc-isa-types](/Users/fujie/Pictures/typora/riscv-micro-arch/riscv-isa-type.png)
2. 指令里包含 3 个地址:dest, rs1, rs2，两地址的地址会导致更大的 code size，并且效率也更低
3. 操作数在指令里的固定位置->方便解码

# RV 32I

1. 40 条指令
2. 可以模拟除了 RISC-V A 之外的所有指令集，A（原子指令集）需要额外的硬件支持。
3. 内部有 32 个寄存器，寄存器`x0`的置恒为 0，其余`x1~x31`以补码形式存放数据，可以是 signed, unsigned.
4. 在 RV 32I 中只有 PC 寄存器, 没有专门的栈指针或返回地址寄存器(使用 31 个基础寄存器来完成这些功能)。通常情况下：
   - 使用 x1 作为 call 函数的返回地址
   - 使用 x2 作为栈指针
   - 使用 x5 作为链接寄存器

## RV 32I 指令格式

1. RISC-V 的 6 种指令：都是 32bits 的指令，它们必须以 4B 对其，当跳转指令跳转的地址不是 4B 对其的时候，会触发`instruction address misaligned` Exception。
   - R, I, S, U: 所有立即数都是符号扩展，且其扩展的时候**与指令第 31bit**同号
     ![R, I, S, U](/Users/fujie/Pictures/typora/riscv-micro-arch/isa-RISU.jpg)
   - B, J 类型的指令：B 类型在 S 类型基础上对 imm 做了不同的排列、J 类型在 U 类型的基础上对 imm 做了不同的排列
     ![R, I, S, U](/Users/fujie/Pictures/typora/riscv-micro-arch/isa-BJ.jpg)
2. 算数运算：RV-32I 中的算数运算涉及到 I、R、U 两种类型的指令：

   - I type: rs1 来自寄存器, rs2 来自立即数

     > RISC-V 中的算数运算没有**溢出判断，溢出判断是通过 branch 指令来做的**，例如:  
     > `addi t0, t1, imm; blt t0, t1, overflow`

     1. `ADDI`: rd<-rs1+imm  
        可以实现`MV`指令: `MV rd, rs1 <= ADDI rd, rs1, 0`  
        可以实现`NOP`指令：`NOP <= ADDI x0, x0, 0`
     2. `SLTI`: set less than imm, 如果 sr1$\leq$imm，则 rd 置 1  
        `SLTIU`: imm 是以”无符号数“的形式进行符号拓展
     3. `ANDI`  
        `ORI`  
        `XORI`: 可以用来实现`NOT`: `NOT rd, rs1 <= XORI rd, sr1, -1`
        ![ADDI, SLTI, SLTIU](/Users/fujie/Pictures/typora/riscv-micro-arch/addi.jpg)
     4. `SLLI`: shift left logical imm  
        `SRLI`: shift right logical imm  
        `SRAI`: shift right arithmatic imm
        ![SLLI, SRLI, SRAI](/Users/fujie/Pictures/typora/riscv-micro-arch/slli.jpg)

   - U type：没有 rs1, rs2，只有 imm
     1. `LUI`: load upper imm, `rd=imm<<12+0x0000`
     2. `AUIPC`: add upper imm to PC, `rd=imm<<12+PC`, `AUIPC`可以用于获取当前的 PC, `AUIPC rd, 0`
        ![LUI](/Users/fujie/Pictures/typora/riscv-micro-arch/lui.jpg)
   - R type：rs1, rs2 都来自寄存器
     1. `ADD`  
        `SUB`
     2. `SLT`  
        `SLTU`
     3. `AND`  
        `OR`  
        `XOR`
     4. `SLL`: shift left logical  
        `SRL`: shift right logical  
        `SRA`: shift right arithmatic

3. 控制运算：RV-32I 支持*直接跳转*和*条件跳转*

   - 直接跳转

     1. `JAL`: jump and link  
        J type, imm 的单位是 2B  
        20 bit 的 imm 是有符号数，其符号拓展为 32 位后跟 pc 相加可以得到目的地址，寻址范围为$\pm1MB$  
        将 pc+4 存入到 rd 中，一般采用 x1 作为 rd
        ![JAL](/Users/fujie/Pictures/typora/riscv-micro-arch/jal.jpg)
     2. `JALR`: jump and link regist, 常与`LUI`或者`AUIPC`搭配使用  
        I type, imm 的单位不是 2B  
        20 bit 的 imm 是有符号数，其符号拓展为 32 位后跟 rs1 相加并且将 lsb 清零，可以得到目的地址  
        将 pc+4 存入到 rd 中，一般采用 x1 作为 rd
        ![JALR](/Users/fujie/Pictures/typora/riscv-micro-arch/jalr.jpg)

     > `JAL`和`JALR`在 rd, rs1 的选择，会影响到 RAS 的 pop 和 push 操作.

   - 条件跳转: 都是 B type 指令

     1. `BEQ`, `BNE`: branch if equal, branch if not equal
     2. `BLT`, `BLTU`: branch if less than, branch if less than unsigned
     3. `BGE`, `BGEU`: branch if greater equal, branch if greater equal unsigned
        ![BEQ](/Users/fujie/Pictures/typora/riscv-micro-arch/beq.jpg)
        > 对于条件跳转，前向跳转默认不跳、后向跳转默认跳

4. Load, Store 指令

   - load

     1. `Load`: addr=rs1+imm, rd=mem[addr], I type
     2. `LH`, `LHU`: load hexadecimal, load hexadecimal unsigned
     3. `LB`, `LBU`: load byte, load byte unsigned

   - store
     1. `Store`: addr=rs1+imm, mem[addr]=rs2, S type
     2. `SW`, `SH`, `SB`: store word, store hexadecimal, store byte

5. 内存访问控制：控制访问内存的指令的顺序
   - `FENCE`：FENCE 指令之前的指令必须全部都执行完，才可以执行 FENCE 指令之后的指令  
     I type
     ![FENCE](/Users/fujie/Pictures/typora/riscv-micro-arch/fence.jpg)
6. 系统指令
   - `ECALL`: 系统服务申请
   - `EBREAK`: 进入调试器
   - `HINT`: 没啥用的指令，类似于`NOP`指令
     ![ECALL](/Users/fujie/Pictures/typora/riscv-micro-arch/ecall.jpg)

## Conclusion
![rv32i reference and encoding](/Users/fujie/Pictures/typora/riscv-micro-arch/rv32-isa-reference-and-encoding.jpg)
# References

1. [RV32I Base Integer Instruction Set, Version 2.1](https://five-embeddev.com/riscv-isa-manual/latest/rv32.html#)
2. [RISC-V Official Specifications](https://riscv.org/technical/specifications/)
3. [RISC-V Assemble Language Programming](https://github.com/johnwinans/rvalp)


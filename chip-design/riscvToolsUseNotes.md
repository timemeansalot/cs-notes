> Unix 环境下，RISCV 工具链使用笔记，包括如下内容：

1. RISCV 编译工具：gcc, objdump, ...
2. RISCV 调试工具：spike, pk, openOCD, GDB, llvm
3. RISCV assemble

# YouTube 开源学习资料

Contents to study:

- [x] RISCV Tutorial: Spike & Proxy Kernel from Source to Hello World
- [ ] RISCV Instruction and Assembly Tutorial
- [x] RISCV Assembly Tutorial: Practice with LED and Switch on Simulator
- [ ] RISCV Tutorial: Binary Instrumentation Technique using LLVM/CLANG Machine Instruction Pass
- [x] RISCV Tutorial: Running FEDORA on RISCV using QEMU
- [ ] RISCV Tutorial: How to Setup LLVM / CLANG for RISC-V
- [x] RISCV Tutorial: Spike Debugging, OpenOCD, GDB
- [x] RISCV Tutorial: Setup GCC Toolchain & SiFive Prebuilt Toolchain

## Tool Install

### RISCV GNU Toolchain

Linux 下有 2 种方法安装**编译工具链**：

1. 从[GitHub 克隆源代码](https://github.com/riscv-collab/riscv-gnu-toolchain)，然后按照 GitHub 中 readme 的步骤编译源代码，进行安装
2. 从[SiFive/freedom-tools](https://github.com/sifive/freedom-tools/releases)直接下载对应平台已经编译好的可执行文件，支持 Linux/Mac/Windows 等平台
   ![](https://s2.loli.net/2023/03/07/h5dNlWv3RjFTmXI.png)

安装完成之后，可以直接在终端中看到对应的工具：
![](https://s2.loli.net/2023/03/07/RVdZFGAjoOBY2az.png)

### 仿真工具：spike, pk, openocd, QEMU

1. [Spike](https://github.com/riscv-software-src/riscv-isa-sim)和[QEMU](https://github.com/qemu/qemu)都是模拟器，可以模拟 RISCV 指令的运行
2. [pk](https://github.com/riscv-software-src/riscv-pk) 是 proxy kernel，用于在 spike 中模拟一个简单的操作系统和 bbl，这样我们才可以加载程序到 spike 中模拟运行
3. [openocd](https://github.com/riscv/riscv-openocd) 是负责远程调试使用

上述工具都可以在 GitHub 中下载对应的源代码，按照 readme 中的步骤编译安装。

### MacOS 下安装工具链和调试工具

macOS 下安装 riscv-gnu-toolchain，spike，pk 和 openocd 都可以[直接通过 homebrew 安装](https://github.com/riscv-software-src/homebrew-riscv):

```bash
brew tap riscv-software-src/riscv
# 安装工具链
brew install riscv-tools
brew install spike
brew install riscv-pk
brew install riscv-openocd
```

测试是否安装成功: `brew test riscv-tools`, 该命令会测试工具链、riscv-isa-sim 和 riscv-pk.

> 总结: MacOS 下配置环境相比于 Linux 更加简单，直接使用 homebrew 就可以全部搞定。

## Spike 使用

### Spike 内部调试

1. 进入 Spike debug: `spike --isa=rv32im -d /user/local/bin/pk hello`
2. spike 内部 debug 命令：
   - `until 0 pc xx`: 一直运行到 pc=xx 的指令为止, 指令的地址可以搭配反汇编得到的汇编文件得到, 使用 objdump
   - `r 1`: 运行一条指令
   - `reg 0 a0`: 查看 core0 中寄存器 a0 的值

### 使用 openOCD 远程调试

参考[Debugging with Gdb](https://github.com/riscv-software-src/riscv-isa-sim#debugging-with-gdb).  
Spike <-> openOCD <-> GDB

# Referrences

1. [RISCV Tutorial on Youtube by Derry Pratama](https://www.youtube.com/watch?v=zZUtTplVHwE&list=PLgzAvj2cYr3qGvecT_PSnKzl5SxECZmI3)

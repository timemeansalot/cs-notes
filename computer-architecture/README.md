> 计算机体系结构学习笔记

# Study Contents

- [x] lecture1-intro-afterlecture.pdf
- [ ] lecture2a-tradeoffs-metrics-mindset-afterlecture.pdf
- [ ] lecture2b-mysteries-afterlecture.pdf
- [ ] lecture3a-mysteries-ii-afterlecture.pdf
- [ ] lecture3b-fpga_labs-afterlecture.pdf
- [ ] lecture4-combinational-logic-i-afterlecture.pdf
- [ ] lecture5-combinational-logic-ii-afterlecture.pdf
- [ ] lecture6-sequential-logic-afterlecture.pdf
- [ ] lecture7-hdl-verilog-afterlecture.pdf
- [ ] lecture8-timing-and-verification-beforelecture.pdf
- [ ] lecture9-vonneumann-isa-lc3andmips-afterlecture.pdf
- [ ] lecture10a-isa-ii-afterlecture.pdf
- [ ] lecture10b-assembly-programming-afterlecture.pdf
- [ ] lecture11-microarchitecture-fundamentals-afterelecture.pdf
- [ ] lecture12-microarchitecture-fundamentals-ii-afterlecture.pdf
- [ ] lecture13-pipelining-afterlecture.pdf
- [ ] lecture14-pipelined-processor-design-afterlecture.pdf
- [ ] lecture15-precise-exceptions-afterlecture.pdf
- [ ] reorder-buffer-example.pdf
- [ ] lecture16-out-of-order-execution-afterlecture.pdf
- [ ] lecture16b-handling-out-of-order-execution-of-loads-and-stores-afterlecture.pdf
- [ ] lecture17a-dataflow-superscalar-afterlecture.pdf
- [ ] lecture17b-branch-prediction-i-afterlecture.pdf
- [ ] lecture18-branch-prediction-ii-afterlecture.pdf
- [ ] lecture19a-vliw-beforelecture.pdf
- [ ] lecture19b-systolicarrays-beforelecture.pdf
- [ ] lecture19c-dae-beforelecture.pdf
- [ ] lecture20-simd-afterlecture.pdf
- [ ] lecture21-gpu-afterlecture.pdf
- [ ] lecture22-memory-organization-afterlecture.pdf
- [ ] lecture23-memory-hierarchy-and-caches-afterlecture.pdf
- [ ] lecture24-advanced-caches-afterlecture.pdf
- [ ] lecture24a-multi-core-caches-afterlecture.pdf
- [ ] lecture25-prefetching-afterlecture.pdf
- [ ] lecture26-virtual-memory-afterlecture.pdf
- [ ] lecture26a-virtual-memory-ii-afterlecture.pdf
- [ ] lecture27-epilogue-afterlecture.pdf
- [ ] problem-solving-i-beforelecture.pdf
- [ ] problem-solving-ii-beforelecture.pdf
- [ ] problem-solving-iii-beforelecture.pdf
- [ ] problem-solving-iv-beforelecture.pdf

# Study Notes

## Introduction Lectures

当前火热的体系结构研究话题有：

1. data move vs data computation：数据移动的功耗已经远大于数据计算的功耗，一次 load 的功耗是 add 的功耗的上百到上千倍，因此出现了 PIM(process in memory),PNM(process near memory)
2. 数据安全问题：
   - raw hammer：访问 sram 的时候，多次访问同一个 word line 会导致其相邻的 word line 的数据发生变化  
     Rawhammer 解决办法: 增加刷新频率，导致功耗增加性能下降(苹果的做法)；根据生产 SRAM 的规矩，对脆弱的行刷新频繁，对稳健的行刷新不频繁，需要额外用 2kB 的存储来存储 32GB 的 SRAM 信息；用 bloom filter 来估计哪些行是脆弱的，bloom 说某一个集合不存在脆弱的行，则其一定不存在
   - spectre：由于分支预测的时候会预测下一个 pc 的地址，在 pc 预测出错的时候，可能会导致敏感数据被载入到 cache 中<-在没有分支预测的情况下，敏感数据肯定是不会被载入到 cache 中的
3. 基因测序 genome sequencing
4. 多核系统性能不如预期，因为多核需要一起访问SRAM，因此SRAM会成为瓶颈，可能导致某些核的访问出现饥饿

> ABI(Application Binary Interface): In computer software, an application binary interface (ABI) is an interface between two binary program modules. Often, one of these modules is a library or operating system facility, and the other is a program that is being run by a user.
### FPGA(Field Programmable Gate Array)
1. Software Reconfigurable:
    - functions
    - connections
    - IOs
2. key parts of FPGA
    - LUT(look up table)
    - Switch(interconnect)
    

# 参考资料

1. [Digital Design and Computer Architecture Spring 2022](https://safari.ethz.ch/digitaltechnik/spring2022/doku.php?id=schedule) by [Onur Mutlu](https://people.inf.ethz.ch/omutlu/)
2. [Application Binary Interface from Wikipedia](https://en.wikipedia.org/wiki/Application_binary_interface)

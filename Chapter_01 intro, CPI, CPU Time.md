# 8 Great Ideas in Computer Architecture
- Design for Moore's Law
	面向摩尔定理的设计
- Use abstraction to simplify disign
	使用抽象简化设计
- Make the common case fast
	加速经常性事件
- Performance via **parallelism**
	通过并行提高性能
- Performance via **pipelining**
	通过流水线提高性能
- Performance via **prediction**
	通过预测提高性能
- Hierarchy of memories
	存储层次
- Dependability via **redundancy**
	通过冗余提高可靠性


# 计算机组成
- 数据通路：进行算数逻辑运算
- 控制器：在数据通路、存储器、I/O之间调度工作流
- 存储器：包括静态随机存储器和动态随机存储器
- 输入设备：如键盘
- 输出设备：如显示器


# Performance
**Performance = 1 / Execution Time**

"X is n time faster than Y" => 性能反比于执行时间
	Performance of x / Performance of Y = Execution time of Y / Execution time of X


# Execution Time
## Elapsed Time（实际时间）
Total response time, including **all aspects**
- processing
- I/O
- OS overhead
- idle time
**Determines system performance**

## CPU Time
影响因素：
- 算法：IC,CPI
- 编程语言：IC,CPI
- 编译器：IC,CPI
- 指令集：IC,CPI,T

计算：
- CPU时间
	- 总时间 = IC * CPI * T = IC * CPI / Rate
- 平均CPI
	- 总cycle数 = CPI * IC 加权求和
	- 平均CPI = 总cycle数 / IC = CPI加权求和
Time spent **processing a given job**
- Discounts I/O time, other job's shares
Comprises **user CPU time and system CPU time**
Different programs are affected diffrently by CPU and system performance

CPU Time = CPU Clock Cycles * Clock Cycle Time = CPU Clock Cycles / Clock Rate

Performance improved by
- Reducing number of **clock cycles**
- Increasing **clock rate**
- Hardware designer must often trade off clock rate against cycle count（许多技术再减少时钟周期的个数时，会增加时钟周期长度）

Clock Cycles = Instruction Count * Cycles Per Instruction

**CPU Time = IC * CPI * Clock Cycle Time = IC * CPI / Clock Rate**

# Pitfall：Amdahl's Law(阿姆达尔定律)
#阿姆达尔定律
![[Pasted image 20240926192613.png]]

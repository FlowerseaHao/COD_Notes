# Instruction Set(指令集)
The repertoire(全体) of instruction of a computer

## Arithmetic Operations
- Add and subtract, three operands
	- **Two sources and one destination**
	- `add a, b, c  // a gets b + c `
- All arithmetic operations have this form

## Register Operands
- Arithmetic instructions use register operands
- RISC-V has a 32 * 32-bit register file 
	- use for **frequently accessed data**
	- 32-bit data is called a "word"
		- 32 * 32-bit general purpose registers **x0 to x31**
		- 64-bit data is called a "doubleword"
		- 16-bit data is called a "halfword"

## Immediate Operands
- Constant data specified in an instruction
	`addi x22, x22, 4`
- *Make the Common case fast*
	- small constants are common
	- Immediate operand avoids a load instruction
		立即操作数可以避免load指令操作

## The Constant Zero
- RISC-V register x0 is the constant 0
	- Cannot be overwritten
- Useful for common operatons
	- E.g. negate a value in a register by sub zero by the value
		使用该寄存器进行取反操作
	- `sub x9, x0, x8`
# RISC-V Registers
- x0: the constant value 0 （存常数0）
- x1: return address（返回地址）
- x2: stack pointer (栈指针)
- x3: global pointer（全局指针）
- x4: thread pointer
- x5 - x7, x28 - x31: temporaries（临时变量）
- x8: frame pointer (帧指针)
- x9, x18 - x27: saved registers（保存寄存器）
- x10 - x11: function arguments/results（函数返回值）
- x12 - x17: function arguments（函数参数）


# Design Principle
- Simplicity favours **regularity**
	简单源于规整
- **Smaller** is faster
	小则快
- Good design demands good **compromises**

# Endianness (端序)

小端字节序，**高字节存放于内存高地址，低字节存于内存低地址**，大端字节反之


# Sign Extension
简单来说就是将一个数扩展到更高的位数，而为了保持本来这个数的值，则按照最高位向前扩展。
- Representing a number using more bits
	- Preserve the numeric value
- Replicate the sign bit to the left
	- c.f. unsigned values: extend with 0s
	- e.g. `addi x11, x0, 0x8f5`
		- Immediate is always **sign-extended to 32-bits** before use in an arithmetic operation
	- Examples: 8-bit to 16-bit
		- +2：0000 0010 => 0000 0000 0000 0010
		- -2： 1111 1110=> 1111 1111 1111 1110
- In RISC-V instruction set
	- lb: sign-extend loaded byte (符号扩展)
	- lbu: zero-extend loaded byte (零扩展)


# RISC-V Instruction Formats
#Instruction_Formats 
![[Pasted image 20240926195621.png]]

## R-format 
同时具有rs1,rs2,rd
#Instruction_Formats/R-format 
![[Pasted image 20240926195717.png]]
- Instruction field
	- opcode: operation code
	- rd: destination register number
	- funct3: 3-bit function code(additional opcode)
	- rs1: first source register number
	- rs2: second source register number
	- funct7: 7-bit function code(additional opcode)


## I-format
#Instruction_Formats/I-format 
立即数为12位字段，即[11:0],同时注意符号位扩展，从12位扩展到32位
代表指令：`addi`,`load`,`slli`,`slri`,`jalr`
![[Pasted image 20240926195946.png]]
- Immediate arithmetic and load instructions
	- rs1: **source or base address** register number
	- immediate: constant operand, or **offset** added to base address

- `addi x15, x1, -50`
	- ![[Pasted image 20240926200321.png]]
	- Immediate is always **sign-extended to 32-bits** before use in an arithmetic operation

- Load Instruction
	- `lw x9, 240(x10)`
	- ![[Pasted image 20240926200433.png]]
	- The 12-bit signed immediate is added to the base address in register **rs1 to form the memory address**
		- This is very similar to add-immediate operation but used to **create address not to create final result**
	- The value loaded from memory is stored in register **rd**

## S-format
#Instruction_Formats/S-format
有rs1, rs2，rd变为立即数的[4:0]字段，立即数仍然为12位[11:0]，但是是隔断的
代表指令：`store`
![[Pasted image 20240926200727.png]]
- rs1: **base address** register number
- rs2: **source operand** register number
- immediate: **offset** added to base address
	- Split so that rs1 and rs2 field always in the same place
		- ![[Chapter_2#Design Principle]]

## U-format
#Instruction_Formats/U-format
没有rs1, rs2，只有rd，也没有func字段，立即数为20位[31:12]（低位字段清零），（常与addi连用，注意符号位扩展）
代表指令:`lui`

![[Pasted image 20241005224236.png]]
### 32-bit Constants (大立即数)
- For the occasional 32-bit constant
	- `lui rd, constant`
	- Copies 20-bit constant to bits [31:12] of rd
	- Clear bits [11:0] of rd to 0

U-format for "Upper Immediate" instructions
- Has 20-bit immediate in upper 20 bits of 32-bit instruction word
- One destination resgister, rd
- Used for **two instructions**
	- LUI - Load Upper Immediate
	- AUIPC - Add Upper Immediate to PC
- LUI to create long immediates
- LUI **Copies 20-bit constant to bits[31:12] of rd**, **Clear bits [11:0] of rd to 0**.（*高20位加载，低12位置0*）
- Together with an `addi` to set low 12 bits, can create any 32-bit value in a register using two instructions (`lui, addi`)（*使用`lui`和`addi`两条指令来构建一个完整的加载大立即数的操作*）
#Create_long_immediates
![[Pasted image 20241005224853.png]]
### 思考：如何得到0xDEADBEEF ？
错误解答（简单的拼接）：
```RISC-V
lui x10, 0xDEADB # x10 = 0xDEADB000
addi x10, x10, 0xEEF # x10 = 0xDEADAEEF
```
Point:
addi 12-bit immediate is always sign-extended, **if top bit is set, will subtract -1 from upper 20 bits**
一定要注意低12位的符号位，addi指令总是进行**符号扩展**，如果符号位为1，会从高20位中减去1

正确解答：
```RISC-V
lui x10, 0xDEADC
addi x10, x10, 0xEEF
```

## SB-format
有rs1, rs2 (进行算术运算),没有rd，12位立即数字段被分为[12],[10:5],[4:1],[11]，所以立即数为[12:1]，以半字为单位，这点要非常注意
代表指令：`beq`
- Branch instructions specify
	- Opcode, two registers, target address
- Most branch targets are near branch
	- Forward or backward
- SB format, mostly same as S-format
#Instruction_Formats/SB-format 
![[Pasted image 20241005230449.png]]
- But now immediate represents values -4096 to +4094 in **2-byte increments**
- The 12 immediate bits encode even **13-bit** signed byte offsets(**lowest bit of offset is always zero**, so no need to store it)
#Instruction_Formats/SB-format/Branch_Example
![[Pasted image 20241005230920.png]]

## UJ-format
#Instruction_Formats/R-format/UJ-format
有rd，没有rs1, rs2,立即数字段被拆分为[20],[10:1],[11],[19:12]，主要用于跳转到指定地址。（注意也是半字为单位）可以向前后各跳转2^18个指令

如果要进行大范围跳转，那使用lui和jalr的组合指令（注意jalr就不是半字为单位了）

代表指令：`jal`
![[Pasted image 20241005231011.png]]
- `jal` target uses **20-bit immediate for larger range**
- For long jumps, eg, to 32-bit absolute address
	- `lui`: load address [31:12] to temp register
	- `jalr`: add address [11:0] and jump to target
Uses of `jal`:
- j pseudo-instruction
	- `j Label = jal x0, Lable #Discard return address`
- Call function within +-2^18 instructions of PC
	- `jal ra, FuncName`
	- `jal`的imm是2^(20+1)个byte
	- 由于一个指令对应**4个byte，所以2^19对应+-2^18条指令**

## jal 与 jalr 对比
![[Pasted image 20241005232531.png]]

#logic_operation/and 
## AND Operation 
- 选择性置零
- and 0

#logic_operation/or
## OR Operation
- 选择性置1
- or 1

#logic_operation/xor
## XOR Operation
- 选择性翻转
- xor 1

# 边界检查
#边界检查
- 当使用角标**大于等于数组长度或为负数**发生**数组下标越界**
- 将有符号数当作无符号数处理
	- 最高有效位在有符号数中表示符号位，在无符号数中表示数的很大一部分
	- 无符号比较在检测x<y时，也检测了x是否为负数
	- bgeu x, y, IndexOutofBounds
		- case  1: x >= y (索引号>数组长度)
		- case  2: x < 0(索引号为负)


# The "ABI"(Application Binary Interface)
A **contract** between functions


# Caller-saved register and Callee-saved register

## The difference between two strategies
![[Pasted image 20240927220832.png]]

## The RISC-V Register and Convention
![[Pasted image 20240927220951.png]]
To Summerize:
caller 保护：
- x1 返回地址
- x10 - x17 参数
- x5 - x7, x28 - x31  临时变量

## why identify the caller/callee saved
example:
```c
int a, b, c, d;
a = b - c + d;
```
对于上面的例子，a,b,c,d 是声明出来的变量（存储在r0 - r3中），而在计算a 的值的时候，无法通过一条指令完成，需要借助临时变量，比如tmp（分配t0寄存器）

首先计算 `tmp = b - c`
`sub r1, r2, t0`
然后计算`a = tmp + d`
`add t0, r3, r0`
tmp的生命周期到此为止，它的生命周期非常短，但又有专门的寄存器来保存这个临时变量，**这种寄存器里面的值总是易变的**，用完即废

所以在函数调用过程中，对于两类寄存器有着不同的责任保护方，第一类寄存器（r）如果在callee里面改了，**不做保护的话，caller里面声明的变量值都改变了**，这类寄存器，**callee用了一个就要保护一个**；
对于第二类寄存器（t），临时变量用完即废，**callee破坏了也可能没啥影响，callee没有义务保护**，但如果caller确实希望保存该临时变量，也必须由caller声明进行保护。


# Change 64-bit to 32-bit
![[Pasted image 20240927222312.png]]

cut all the labeled red half:
- doubleword to word
- 16 to 8
- 8 to 4


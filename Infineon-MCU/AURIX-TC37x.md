# AURIX Infineon-TC37xx-32-Bit Single-Chip Microcontroller 学习笔记

## 1.概述
- **功能概述**：高性能微控制器、三核
三个**32 位超标量 TriCore 处理器**（TC1.6.2P）
  - 完全集成的 DSP（数字信号处理）功能
  - 乘加单元（MAC），每周期可执行 2 次 MAC 操作
  - 完整流水线的浮点运算单元（FPU）
  - 在全温度范围内支持高达 300 MHz 的运行频率
  - 高达 240/96 KB 的数据专用高速 RAM（DSPR）
  - 高达 64 KB 的指令专用高速 RAM（PSPR）
  - 32 KB 的指令缓存（ICACHE）
  - 16 KB 的数据缓存（DCACHE）

- 多种片上**存储器**
  - 所有嵌入式 NVM（非易失性存储）与 SRAM 都具备 ECC 错误检测与纠正功能
  - 高达 6 MB 的程序 Flash（PFLASH）
  - 高达 256 KB 的数据 Flash（DFLASH 0），可用于 EEPROM 仿真
  - 引导 ROM（BootROM / BROM）  

- 片上总线架构
  - 64 位交叉开关互连总线（SRI），提供 CPU、主控器与存储器之间的高速并行访问
  - 32 位系统外设总线（SPB），用于片上外设与功能模块连接
  - SRI 到 SPB 的总线桥接器（SFI Bridge）

- 片上**外设**单元
  - 12 个异步/同步串行通道（ASCLIN），支持 **LIN** 协议（V1.3、V2.0、V2.1 和 J2602），速率最高达 50 MBaud
  - 5 个队列式 **SPI** 接口通道（QSPI），支持主机和从机模式，速率最高达 50 Mbit/s
  - 1 个高速串行链路（HSSL），用于处理器间串行通信，速率最高可达 320 Mbit/s
  - 2 个微秒级串行总线接口（MSC），用于与外部电源设备的串行端口扩展
  - 2 个 MCMCAN 模块，含 4 个 **CAN** 节点，通过 FIFO 缓冲区实现高效数据处理
  - 15 个单边半字传输（SENT）通道，用于与传感器连接
  - 1 个 **FlexRay** 模块，包含 2 个通道（E-Ray），支持 FlexRay 协议 V2.1
  - 1 个通用定时器模块（GTM），提供强大的数字信号滤波和定时功能，可实现自主且复杂的输入/输出管理
  - 1 个捕获/比较模块 CCU6（包括两个核心：CCU60 和 CCU61）
  - 1 个通用 12 位定时器单元（GPT120）
  - 2 通道外设传感器接口（PSI5），符合 V1.3 标准
  - 1 个带串行物理层的外设传感器接口（PSI5-S）
  - 1 个 **I²C** 总线接口，符合 I²C V2.1 标准
  - 1 个 IEEE802.3 以太网 MAC 接口，支持 RMII 和 MII 连接（ETH）

### 1.2.缩写与定义

|术语缩写|缩写定义                                                       |
|-------|---------------------------------------------------------------|
|BE     |总线访问因错误而终止                                             |
|ok     |允许总线访问并执行                                               |
|16     |16位和32位宽度总线访问允许并执行                                  |
|32     |32位宽度总线访问允许并执行                                       |
|Access |允许总线访问并执行                                               |

### 1.3.系统资源连接器总线
  - **SRI**（**System Resource Interconnect**）是AURIX™ TC37x中的**高性能内部总线**，连接CPU（TriCore™）、存储器（SRAM、Flash）、外设（CAN、GPIO）和DMA等模块。
它像TC37x的“高速公路”，允许“总线主控”（Bus Masters，如CPU、DMA）快速访问存储器和寄存器。

  - Table 2（**Address Map as seen by Bus Masters on Bus SRI**）是SRI总线的内存映射（MEMMAP），定义了存储器和外设的地址范围，以及读写权限。
    - 见**Infineon-AURIX_TC37x-UserManual-v02_00-EN**用户手册**P11**。

### 1.4.SCU_STMEMx寄存器相关内容
  - 该寄存器存储**系统控制单元的状态标识**，而**Table5**展示了各种设备复位之后，如果所有检查都通过了，那么系统控制单元中的 SCU_STMEM3 到 SCU_STMEM6 寄存器中将分别保存什么值。这些值可以用于调试或验证系统启动是否正常。
    - 见**Infineon-AURIX_TC37x-UserManual-v02_00-EN**用户手册**P22**。

### 1.5.DOM0配置参数定义访问权限、地址范围和接口数量
  - DOM（Domain Control）是AURIX™ TC37x中的片上模块，负责管理特定存储器区域或外设的访问权限和数据传输。
  - 在TC37x中，DOM0是一个特定的域（Domain），用于控制与SRI总线（System Resource Interconnect）和**OLDA（Online Data Acquisition）**相关的访问。
  - 生活化比喻：DOM0是TC37x的“门卫”，管理“高速公路”（SRI）和“数据仓库”（OLDA）的访问。Table 7是“门禁规则”，规定谁（CPU、HSM）、何时（Endinit）、如何（Supervisor Mode）能进。
   - 见**Infineon-AURIX_TC37x-UserManual-v02_00-EN**用户手册**P23**。
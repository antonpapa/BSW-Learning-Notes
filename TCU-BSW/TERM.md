# 专业名词补充

**MEMMAP**: **Memmory MAP**(内存映射)
**PSPR**  ：**Program Scratch Pad SRAM**(程序暂存静态随机访问存储器)
    - **SRAM**是**Static Random Access Memory**（静态随机访问存储器）的缩写，是一种**易失性存储器**（Volatile Memory），用于在芯片运行时快速存储数据或代码。
    - 定义：静态随机访问存储器，易失性，TC37x包括PSPR（64KB）、DSPR（240KB/96KB）、LMU（64KB）、DAM（32KB）。
    - 特点：高速（零等待）、低功耗、ECC保护、专属/共享、容量有限。
    - 用途：存储MCAL代码（PSPR，如Can_Init()）、数据（DSPR，如gear=0x01）、共享缓冲区（LMU，如CAN RX）、全局变量（DAM）。

**DSPR**  : **Data Scratch Pad SRAM**(数据暂存静态随机访问存储器)

|特性	      |PSPR	                     |DSPR                     |
|全称         |	Program Scratch Pad SRAM|	Data Scratch Pad SRAM  |
|用途         |	存储程序代码（指令、函数）|	存储数据（变量、缓冲区、栈）|
|大小（TC37x）|	64KB/CPU             |	240KB (CPU0/1), 96KB (else)|
|地址         |	0x70100000+	          |0x70000000+                 |
|BSW场景     |	MCAL函数（如Can_Init）|	CAN变量（如gear=0x01）       |
|速度        |	高速，零等待周期      |	高速，零等待周期              |
|专属性      |	每个CPU独占          |	每个CPU独占                  |

**DCACHE**: **Data Cache**（数据缓存）->高速存储数据的缓存内存
  - **加速数据访问**：DCACHE存储频繁访问的数据（如MCAL变量、gear=0x01），比从主存储器（SRAM、Flash）读取快，减少CPU等待时间。
  - **工作原理**：当CPU需要数据时，先查DCACHE；若命中（数据在DCACHE），直接读取；若未命中，从DSPR或DAM加载，并缓存到DCACHE。
  - **TCU BSW场景**：在变速器TCU开发中，DCACHE加速CAN消息处理（比如msg_id、gear），提高MCAL性能（如Can_Write()）。
  - **注意**：DCACHE需管理一致性（确保缓存与主存储器同步），尤其在多核（TC37x有3个主核）或DMA操作时，MCAL开发可能需刷新DCACHE。

**PFlash**: **Program Flash**(程序闪存) 
  - 指的是AURIX™ TC37x中用于**存储程序代码**的**非易失性存储器**（**Non-Volatile Memory**, NVM）。
    - **非易失性**：掉电后数据保留，适合存储固件。
    - 用途：
          存储MCAL驱动代码（如Can_Init()、Can_Write()）。
          存储AUTOSAR操作系统和TCU应用代码（如换挡逻辑）。
          存储配置数据（如EB tresos生成的CAN配置）。

**BROM**：**Boot ROM**（启动只读存储器）
  - BROM是**Boot Read-Only Memory**（启动只读存储器）的缩写，指的是AURIX™ TC37x中一块**只读**的**非易失性存储器**，用于存储**启动代码**（Bootloader）。
    - 只读：出厂固化，防止篡改。
    - 用途： 
          包含启动代码，负责芯片上电后的初始化（设置时钟、内存、HSM）。
          执行引导逻辑，决定从PFlash、外部存储还是调试接口加载固件。
          支持安全启动（Secure Boot），验证PFlash代码的签名（ASIL-D）。

**LMU**：**Local Memory Unit**(本地存储单元)
  - **LMU**是**Local Memory Unit**（本地存储单元）的缩写，指的是AURIX™ TC37x中一种高速SRAM存储器，用于存储数据和代码，供多个CPU或其他模块（如DMA）共享访问。
    - **共享性**：不像DSPR/PSPR（每个CPU独占），LMU可被多核CPU、DMA、HSM等访问，**适合跨模块数据交换**。
    - 用途：
          存储共享数据，如CAN消息缓冲区（msg_id、gear）供多核处理。
          存储临时代码，如MCAL的辅助函数。
          支持DMA传输，比如将CAN接收数据从LMU搬到DSPR。

**EMEM**：**Extension Memory**（扩展存储器）
  - **EMEM**是**Extension Memory**（扩展存储器）的缩写，指的是AURIX™系列中用于扩展存储的额外存储器模块，通常是SRAM或类似高速存储器，支持灵活的数据或代码存储。
    - 速度：高速SRAM，接近LMU，适合大容量数据。
    - 用途：
          存储大数据缓冲区，如以太网数据、复杂算法的中间结果。
          支持外部设备访问，如通过HSSL（高速串行链路）。
          存储调试数据或实时日志。

**SRI Bus**： **System Resource Interconnect BUS**（系统资源连接器总线）
  - **SRI**（**System Resource Interconnect**）是AURIX™ TC37x中的**高性能内部总线**，连接CPU（TriCore™）、存储器（SRAM、Flash）、外设（CAN、GPIO）和DMA等模块
  - **Infineon-AURIX_TC37x-UserManual-v02_00-EN**用户手册P11有BUS Faberic SRI 的 **address map**

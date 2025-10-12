# BSW-Learning-Notes
My learning notes for AUTOSAR BSW development, Infineon MCU
笔者个人的学习笔记，笔者会将实习和项目中遇到的相关知识整理以后发在这个note中，由于笔者也是个新手小白，所以笔记中不可避免的会出现一些谬误，还请大家理解。
## BSW 技术栈大纲

---

### 一、BSW开发概述
BSW（Basic Software）是AUTOSAR（AUTomotive Open System ARchitecture）架构中的基础软件层，负责提供底层驱动、操作系统、通信协议栈（如CAN、LIN、Ethernet）、诊断服务（如UDS）、内存管理等功能。C语言是BSW开发的主流语言，因为它在嵌入式系统中具有高性能、低开销和直接硬件访问能力。

#### 关键特点：
1. **资源约束**：ECU通常具有有限的RAM（几KB到几MB）、Flash（几十KB到几MB）和低功耗CPU，代码必须高效。
2. **实时性**：BSW代码需满足硬实时或软实时要求（如任务调度、中断处理）。
3. **可靠性**：汽车系统需高可靠性，代码必须健壮，避免崩溃或未定义行为。
4. **标准化**：AUTOSAR规范（如MISRA C，这里笔者由于学习时间有限，相关的编写规范已经有一点点忘了，不过笔者会在后续尽可能的回忆自己实习时的工作内容）
              对代码风格、安全性有严格要求。

#### 笔者目前的技术优势：
- **C++背景**：C++的面向对象思想和内存管理经验可帮助笔者理解BSW的模块化设计和资源管理。
- **SLAM+深度学习**：SLAM的实时计算和优化经验可迁移到BSW的实时任务处理；Python经验有助于笔者快速上手BSW工具链的脚本开发（如配置工具）。
- **车载嵌入式软件开发经验**：项目的毒打让笔者对数据手册的阅读不在那么陌生，可以比较快速的理解各种寄存器和当前芯片的各种功能。
- **底层软件测试开发实习**：在头部Tier1的实习经历让笔者对ASPICE有了些许的认知，最起码知道从需求到实现的大致过程，同时对底软开发发软件编写规范有了基础的认知。
                          还有就是笔者在实习时接触了很多工具，比如INCA、Canoe、Labcar、AEEE（一种基于eclipse实现的ECU开发工具，笔者只是把它当作arxml配置工具和代码版本管理工具）、SVN、Doors等等。这些工具的上手经历也加深了笔者对于现代车企开测试开发流程的理解。

---

### 二、BSW相关的C语言核心知识点
以下是笔者总结的BSW开发中C语言的核心知识点（这里只是简单提及，后续会有详细的笔记）：

#### 1. **C语言基础与嵌入式特性**
- **基本语法强化**：
  - **指针与内存管理**：熟练掌握指针操作（如指针算术、函数指针），因为BSW常需直接操作硬件寄存器。例如：
    ```c
    volatile uint8_t * const pReg = (uint8_t *)0x4000; // 指向硬件寄存器
    *pReg = 0xFF; // 写入寄存器
    ```
    **注意**：使用`volatile`防止编译器优化关键硬件访问。
  - **位操作**：BSW中常需操作硬件寄存器，位操作（如`&`、`|`、`<<`、`>>`）是核心技能。例如：
    ```c
    #define SET_BIT(reg, bit) (reg |= (1 << bit)) // 设置寄存器某位
    #define CLEAR_BIT(reg, bit) (reg &= ~(1 << bit)) // 清除寄存器某位
    ```
  - **结构体与联合**：使用`struct`组织数据（如CAN帧），用`union`实现字节对齐和内存复用。例如：
    ```c
    typedef union {
        uint32_t rawData;
        struct {
            uint8_t signal1 : 4; // 4位信号
            uint8_t signal2 : 4; // 4位信号
        } signals;
    } CanFrame;
    ```
- **嵌入式C特性**：
  - **固定宽度类型**：使用`stdint.h`中的`uint8_t`、`int16_t`等，确保跨平台一致性。
  - **中断处理**：熟悉中断服务例程（ISR），避免在ISR中调用复杂函数。例如：
    ```c
    void __interrupt ISR_Timer0(void) {
        // 简单操作，避免复杂逻辑
        TimerFlag = 1;
    }
    ```
  - **裸机与RTOS**：BSW常运行在裸机或RTOS（如OSEK/VDX、AUTOSAR OS）上，需掌握任务调度和同步机制（如信号量、互斥锁）。

#### 2. **资源优化开发技巧**
由于BSW开发中，资源（CPU、RAM、Flash）极为宝贵（毕竟车规级芯片追求的就是简单快速安全嘛），所以对于资源的管理极为重要（要善于使用NVM合理分配资源）：

- **代码大小优化**：
  - **避免冗余代码**：使用宏和内联函数减少重复代码。例如：
    ```c
    #define READ_SENSOR(id) (SensorData[id] = *(volatile uint16_t *)(SENSOR_BASE + id * 2))
    static inline void ToggleLed(void) { LED_PORT ^= LED_MASK; }
    ```
  - **条件编译**：用`#ifdef`/`#endif`根据硬件配置裁剪代码，减少Flash占用。
    ```c
    #ifdef FEATURE_X
    void FeatureX_Init(void) { /* 功能X初始化 */ }
    #endif
    ```
  - **查表法**：用查找表代替复杂计算，节省CPU周期。例如：
    ```c
    const uint8_t LookupTable[256] = { /* 预计算值 */ };
    uint8_t result = LookupTable[input];
    ```
- **内存优化**：
  - **静态分配优先**：避免动态内存分配（如`malloc`），使用静态数组或结构体。例如：
    ```c
    uint8_t CanBuffer[8]; // 静态分配CAN帧缓冲区
    ```
  - **内存对齐**：确保结构体对齐，减少填充字节。例如：
    ```c
    #pragma pack(push, 1) // 1字节对齐
    typedef struct {
        uint8_t id;
        uint16_t value;
    } SensorData;
    #pragma pack(pop)
    ```
  - **堆栈优化**：最小化函数调用深度，减少栈使用；避免大局部变量。
- **执行效率优化**：
  - **循环展开**：对于小型循环，手动展开以减少跳转开销。例如：
    ```c
    // 替代 for (i = 0; i < 4; i++) { buf[i] = 0; }
    buf[0] = buf[1] = buf[2] = buf[3] = 0;
    ```
  - **避免浮点运算**：嵌入式系统中浮点运算开销大，使用定点运算替代。例如：
    ```c
    int32_t fixed_point = (int32_t)(float_value * 1000); // 定点化
    ```
  - **中断优化**：将ISR逻辑尽量简化，复杂处理移到主循环或任务中。

#### 3. **优雅与模块化设计**
- **模块化开发**：遵循AUTOSAR的分层架构（BSW分为MCAL、OS、COM、MEM等模块），每个模块独立实现，接口清晰。例如：
  ```c
  // CanIf.h
  typedef struct {
      uint8_t *data;
      uint8_t len;
  } CanIf_PduType;

  void CanIf_Transmit(CanIf_PduType *pdu);
  ```
- **状态机设计**：用状态机实现复杂逻辑，代码清晰且易维护。例如：
  ```c
  typedef enum { IDLE, INIT, RUNNING, ERROR } State;
  State currentState = IDLE;
  void ProcessState(void) {
      switch (currentState) {
          case IDLE: /* 逻辑 */ break;
          // ...
      }
  }
  ```
- **错误处理**：集中式错误管理，使用枚举或宏定义错误码。例如：
  ```c
  typedef enum { OK, ERR_NULL_PTR, ERR_TIMEOUT } ErrorCode;
  ErrorCode InitModule(void) {
      if (!isInitialized) return ERR_NULL_PTR;
      // ...
      return OK;
  }
  ```

#### 4. **AUTOSAR与MISRA C规范**
- **AUTOSAR要求**：
  - 熟悉BSW模块（如MCAL、COM、DCM、NVM）及其接口。
  - 使用AUTOSAR工具链（如Vector DaVinci、EB Tresos）生成配置代码。
  - 理解RTE（Run-Time Environment）与BSW的交互。
- **MISRA C合规**：
  - 遵循MISRA C:2012（如规则1.1：避免未定义行为；规则8.6：声明与定义一致）。
  - 避免指针算术复杂操作，优先用数组索引。
  - 静态分析工具（如LDRA、QAC）检查代码合规性。

#### 5. **调试与测试**
- **调试技巧**：
  - 使用调试器（如JTAG、SWD）查看寄存器和内存。
  - 插入日志点（通过UART或CAN输出），但避免影响实时性。例如：
    ```c
    #ifdef DEBUG
    void Log(const char *msg) { Uart_Transmit(msg); }
    #else
    #define Log(msg) ((void)0) // 空操作
    #endif
    ```
- **单元测试**：使用工具如测试BSW模块（笔者使用的是公司提供的UDE（集烧录调试为一体的软件，需配合pls硬件设备使用）），确保功能正确。
- **HIL测试**：在硬件在环（HIL）环境中验证BSW与硬件交互。

---

### 三、BSW开发中的C语言最佳实践（这一块主要是笔者写给自己的学习笔记，所以和个人技术栈紧耦合的内容可能比较多~S）
由于实习已经结束，老想着使用公司的项目锻炼自己的开发能力也不现实，毕竟笔者目前还是学生，因此笔者整理了一下可以用于自己在学校练习的C语言案例：

1. **从C++迁移到C的思维转换**：
   - **摒弃面向对象**：C没有类，改用结构体+函数实现模块化。例如，C++的`class Sensor`可转化为：
     ```c
     typedef struct {
         uint16_t value;
         bool isActive;
     } Sensor;
     void Sensor_Init(Sensor *self);
     ```
   - **手动管理资源**：C++的RAII不可用，需显式初始化和清理资源。
   - **异常处理替代**：用返回值或错误码替代C++的`try-catch`。

2. **SLAM经验的迁移**：
   - **实时优化**：SLAM中的实时算法优化经验可用于BSW任务调度和中断处理。
   - **矩阵运算**：SLAM中的矩阵操作若在BSW中需要（如传感器数据处理），用定点运算或查表法实现。
   - **Python辅助**：用Python编写脚本生成配置代码或解析CAN日志，提升开发效率。

3. **优雅代码风格**：
   - **命名规范**：使用清晰的命名（如`CanIf_Transmit`、`Nvm_WriteBlock`），遵循AUTOSAR命名规则。
   - **注释充分**：为每个函数和关键逻辑添加注释，符合MISRA要求。
   - **代码复用**：通过宏、函数和头文件复用通用逻辑，减少冗余。

4. **资源最小化开发**：
   - **最小化全局变量**：全局变量增加耦合，优先使用局部变量或传递指针。
   - **精简数据结构**：选择合适的数据类型（如`uint8_t`代替`int`），减少内存占用。
   - **低功耗设计**：在空闲状态进入低功耗模式（如关闭外设时钟）。

---

### 四、推荐学习资源与工具
1. **书籍**：
   - 《Embedded C Programming》：深入讲解嵌入式C开发。
   - 《AUTOSAR Compendium》：了解AUTOSAR架构和BSW模块。
   - 《MISRA C:2012 Guidelines》：学习安全编码规范。

2. **在线资源**：
   - **B站C语言教程**：如《郝斌C语言自学教程》（https://www.bilibili.com/video/BV1os411h77o），适合快速复习C语言基础。[](https://blog.csdn.net/sinat_33921105/article/details/107074763)
   - **Vector AUTOSAR Academy**：提供AUTOSAR相关培训。
   - **代码巴士**：分享C语言嵌入式开发案例（https://codebus.cn）。[](https://codebus.cn/)

3. **工具**：
   - **编译器**：IAR Embedded Workbench、Keil uVision，S32 DS（nxp提供的开源软件，可以花点小钱买一块s32k的板子配合使用）。
   - **调试工具**：Lauterbach TRACE32、J-Link。
   - **配置工具**：Vector DaVinci Configurator、EB Tresos。
   - **静态分析**：QAC、PC-lint（检查MISRA合规性）。

4. **实践项目**：
   - **小型BSW模块开发**：实现一个简单的CAN驱动或NVM模块，熟悉AUTOSAR接口。
   - **开源项目**：参考RTEMS或FreeRTOS，学习嵌入式C开发。


---

### 五、总结
BSW开发的C语言编程需要在资源受限的环境中追求高效、可靠和优雅。重点是：
- 掌握嵌入式C的特性（如指针、位操作、中断）。
- 遵循AUTOSAR和MISRA规范，确保代码安全性。
- 优化资源占用（Flash、RAM、CPU），通过静态分配、查表法、循环展开等技巧实现。
- 利用状态机、模块化设计保持代码优雅。
- 结合Python经验，自动化配置和测试流程。

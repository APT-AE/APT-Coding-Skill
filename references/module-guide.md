# 模块处理指南：先分类，后套范例

**模块无穷无尽**(GPIO/UART/SPI/I2C/CAN/ADC/DAC/定时器/PWM/比较器/DMA/RTC/触摸/电机/协议栈…)。
不要试图枚举全集。任何模块**先归入下面的处理类别**，套该类的通用原则；再参考已沉淀的适配层范例。

## 核心手法：适配层(保接口语义)
业务逻辑照搬时，**底座对外暴露的函数/宏名保持源工程原样，只换内部实现**。
业务代码调用处零改动，改动集中在薄适配层。

- 源常是"单步调用"，APT 常是"多步配置" → 用适配层把 APT 的多步**包成源的同名单函数**。
- 多语句宏必须包 `do{}while(0)`，否则在 `if/else` 里出错。

### ★ 从 8-bit MCU 迁移陷阱：寄存器不可 byte 寻址
APT 芯片控制寄存器不支持 byte 寻址，**不能像 8051 那样直接赋值寄存器**(`P1=0xFF`、`TRISB|=0x02` 等)。
→ 必须通过 APT 驱动 API 函数操作，详见 `general/寄存器访问约束.md`。

## test.c 纪律：从建立时就为"最终删除"设计

验证代码集中在 `test.c`，但 **test.c 最终必须删除**(交付不带验证痕迹)。因此从建立时就要守住：

- **test.c 只放验证逻辑，绝不定义业务实体**。任何业务变量/函数(如加热开关 `g_bHeatOn`)必须定义在正式业务文件里(如 porting.c / work.c)，**不能寄生在 test.c**。
  否则删 test.c 时业务代码引用的符号未定义 → 链接失败(实测踩坑：`g_bHeatOn` 曾寄生 test.c，删除时 display.c/work.c 全报 undefined)。
- **test.c 靠 main.c 一两句调用挂载**(`test_run()` / `test_all_init()`)，删除时只需去掉这几句 + cdkproj 注册 + 删文件，不牵动业务。
- **交付前清理清单**(去掉所有 test/调试痕迹)：
  1. 删 `test.c` 文件
  2. cdkproj 去掉 `<File Name="test.c">` 注册 → cdk-make 重新生成 mk
  3. main.c 去掉 test 的 extern 声明、调用、相关注释
  4. 去掉非项目所需的打印(`my_printf`/`printf` 调试语句)
  5. 确认业务变量都不在 test.c 里(在 §上一条已保证)
  6. 整工程 rebuild 0 error，功能不变

## 模块分类处理原则

| 类别 | 通用原则 | 例子 |
|---|---|---|
| 纯 GPIO 类 | 位操作/sbit → APT `gpio_` 驱动，适配层保名 | 继电器、LED、蜂鸣、按键输入 |
| 带时钟的外设 | **物理量守恒**，按主频重算分频/重载/波特率 | 定时器、PWM、串口、ADC 采样时钟 |
| 通信接口类 | 换 APT 对应外设驱动；协议逻辑(纯C)照搬 | UART/SPI/I2C/CAN/LIN |
| 存储类 | 换 APT Flash/EEPROM 驱动，注意擦写单位 | EEPROM、片上 Flash |
| APT 特色/专有库类 | 用 APT 库重配，**参数重新标定，不抄源数值** | 触摸、LED 矩阵驱动、运放 |
| 纯软件/业务逻辑 | 纯C照搬，经适配层调底座 | 状态机、算法、协议栈 |
| **★ APT 没有的外设** | **禁止臆造。停下告诉客户，给替代方案，由客户/FAE 定** | 源有而 APT 无的外设 |
| 架构级映射(ETCB) | 见文末。当前仅识别标记，不实现 | 定时器触发ADC、ADC触发DMA |

## 已沉淀的适配层范例(APT32F104，编译验证过)

### GPIO(✅编译验证)
源(8051)常见：`sbit` 位操作，`Pin^=1` 翻转、`Pin=0` 置位、`TRIS` 寄存器控方向。
APT：`gpio_reverse(port,pin)` / `gpio_set_low/high(port,pin)` / `gpio_configure(port,pin,PIN_OUTPUT/PIN_INPUT)`。
```c
#define HEAT1_PORT GPIOB0
#define HEAT1_PIN  0
#define HEAT1ON()  gpio_reverse(HEAT1_PORT, HEAT1_PIN)   // 源: Pin_HEAT1^=1
#define HEAT1OFF() gpio_set_low(HEAT1_PORT, HEAT1_PIN)   // 源: Pin_HEAT1=0
#define FANON()    do{ gpio_configure(FAN_PORT,FAN_PIN,PIN_OUTPUT); gpio_set_low(FAN_PORT,FAN_PIN); }while(0)
```
坑：① 多语句宏必须 `do{}while(0)`；② **引脚不能假设同名**，源 PB0 ≠ APT PB0，问客户实际接线；③ 方向切换(开漏/推挽)语义要理解对。

### 时钟(✅上板验证)
**不翻译源时钟，直接用 APT demo 的配置**(demo 已验证；源目标频率本就不同)。
`syscon_osc_enable(...)` + `syscon_hclk_pclk_configure(源, HCLK分频, PCLK分频, 频段)`。
**登记主频**(如 48MHz)写入移植笔记，后续所有带时钟模块据此重算。
验证：串口打印不乱码 = 时钟对。

### 定时器(✅编译验证，物理量守恒范例)
两边机制不同，**寄存器值不可搬，只保周期时间**：
- 源(8051 Timer0)：`重载值 = 65536 - (us × SYSCLK/12)`，125us@12MHz → 65411。
- APT(BT)：分频 + prdr 两级。
```c
void Timer0_Init(unsigned int us) {     // 保源接口名
    bt_software_reset(BT1);
    bt_configure(BT1, BTCLK_EN, 47, BT_IMMEDIATE, BT_CONTINUE, BT_PCLKDIV); // 47+1=48分频@48M→1MHz
    bt_set_prdr_cmp(BT1, us, us/2);      // prdr=us个计数=us微秒
    bt_int_enable(BT1, BT_INT_PEND);
    bt_start(BT1);
}
```
源 65411 与 APT prdr=125 **无数值关系**，只有 125us 守恒。派生计数(`500000/Timer0Us` 之类)纯C照搬。

### ADC(✅编译验证)
源业务层一句 `Get_ADC(ch)` 读值，外包采样滤波算法(纯C照搬)。APT 要 配→启→等→读 四步，用适配层包成单函数：
```c
uint16_t Get_ADC(uint8_t ch) {          // 保源接口名
    adc_ain_configure(ADC0, (adc_ainsel_e)ch, 0);
    adc_control(ADC0, ADC_START);
    adc_wait_eoc(ADC0);
    return adc_get_data(ADC0, 0);
}
```
**★ 通道号不是数字对数字**：源"通道24" ≠ APT 第24号枚举。要看这路信号(如 NTC)在 APT 接哪个物理脚 → 对应 `ADC_ADCINAx`，**问客户**。

### 电容触摸(若有，最难，护城河)
- **解耦点**：源与 APT 都在"按键位掩码"这层解耦(源 `keys_flag` ↔ APT `wKeyMap`)。
  策略：业务代码照搬，只要 APT tkey 库产出等价按键掩码。
- 触摸库是闭源 `.a`，只能通过头文件 API + 全局数组交互(`wSamplingData`采样/`wBaselineData`基线/`nOffsetData`偏移/`wKeyMap`按键位图)。
- **参数不能抄源数值**(阈值/电流源依赖硬件电路)，必须在 APT 上重新标定。
- 验证：上位机(demo 自带 `tkey_uart_serialplot()` 经 UART 送 SerialPlot)看波形，边看边调阈值。
- 调参完整流程待硬件实证沉淀。

## 架构级映射(ETCB)—— 当前仅识别标记，不实现
源芯片无 ETCB，"A 触发 B"靠软件中断(A中断→CPU→ISR启动B)。APT 用 ETCB 硬件直连，不经 CPU。
**当前动作**：发现"中断里触发另一模块"的模式，**记录并提示客户"此处后续可用 ETCB 硬件互连优化"**，
但当前仍按源的中断方式迁移，不自动改(用不用是设计决策)。完整实现留后续版本。


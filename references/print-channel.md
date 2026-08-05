# 调试打印通道：硬件 UART vs 半主机

打印是客户"看见"芯片状态的窗口，必须第一步建立(后续所有验证靠它)。有两种方式。

## 隐藏开关：iostring.c 的 _DEBUG_UART_IO
打印函数 `my_printf` → `__putchar__`，其走向由 `iostring.c` 里 `#define _DEBUG_UART_IO` 控制：
```c
void __putchar__ (char ch) {
#ifdef _DEBUG_UART_IO
    uart_write_byte(UART0,ch);          // 定义 → 走硬件 UART0
#else
    volatile unsigned int *pdata = (unsigned int *)LDCC_DATA_P;  // 注释掉 → 走半主机 LDCC
    while (*pdata & LDCC_BIT_STATUS);
    *pdata = ch;
#endif
}
```

## 两种方式对比与选择

| | 硬件 UART | 半主机(LDCC) |
|---|---|---|
| 依赖 | 板子引出 UART 口 + USB-TTL | iostring 改对 + CDK 调试界面勾 "enable debug print" + **必须在 debug 会话里** |
| 脱机运行 | ✅ 照常输出 | ❌ 离开调试器就没了 |
| 对客户 | 插串口线开助手，直观 | 要懂 CDK 调试配置，门槛高 |

**决策：先问客户"板子有没有引出可用的 UART 口"。**
- **有 → 用硬件 UART(默认推荐)**：`#define _DEBUG_UART_IO` 保持定义；配好 UART 引脚(见下)。
- **没有 → 用半主机**：注释掉 `_DEBUG_UART_IO`；提示客户在 CDK 调试界面勾 "enable debug print"，且只能在 debug 会话里看。

## 配 UART 引脚(硬件 UART 时)
demo 默认 `console_init()` 用 `UART0_TX_PA01`。**问客户板子的串口 TX 在哪个脚**，改成对应引脚：
```c
uart0_pin_config(UART0_TX_PA05);              // 例：TX=PA05
gpio_pull_configure(GPIOA0, 5, PULL_MODE_PULLUP);  // 对应脚上拉
uart_configure(UART0, 94, UART_DATA_8BIT, UART_PARITY_NONE);  // 分频94@24MHz=256000
```
- 引脚复用枚举见 `Driver/inc/top.h`(如 `UART0_TX_PA05`)，该脚须支持 UART0_TX。
- **波特率分频按主频重算**：`94 = 24MHz/256000`；主频若 48MHz 则 `≈188`。保波特率不变、分频随主频变。

## 验证
串口助手(对应波特率，8-N-1)看打印。
- 正常打印、不乱码 → 通道建立，且**间接证明时钟配置正确**。
- **乱码 → 主频与波特率不匹配**，检查分频值是否按实际主频算。

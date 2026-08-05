# 把 demo 掏成白纸骨架

## 为什么
demo 的价值是**骨架**(能编译/能烧录/启动+链接+时钟对/驱动可用)，不是它自带的业务。
迁移起点 = 先把 demo 掏成"白纸骨架"(只剩底座，能编译能跑)，再按客户需要逐个加 IP。

## 白纸骨架 = 保留什么 / 删除什么
- **保留(底座)**：启动文件、链接脚本(.ld)、时钟/中断框架、所有驱动库(Driver/src/*.c)、
  console 打印、中断向量表。
- **删除(应用业务)**：demo 特有的应用逻辑(如触摸扫描→LED 显示→蜂鸣那一套)、业务参数表。

## ★ 关键坑：业务代码渗透在三处，不止 main.c
实测：只掏 main.c，整工程链接报一堆 `undefined reference`。业务钩子藏在：
1. **main.c** — 主循环业务。
2. **interrupt.c** — 中断服务里调的业务函数(如 `led_scan_prg`/`Buzzer_drive_prg`/`tkey_xxx_process`)，
   以及引用的业务变量(如 `byCntMs`/`byBuzzerOutCnt`)。
3. **initial.c** — 初始化里的业务钩子(如 `led_io_config`/`tk_config`/`bt_config`/`delay_nms`)。

三处都要处理干净，否则整工程链接失败。

## 掏的尺度
- 删"应用业务"，留"底座"。
- **中断函数留空壳**(保留函数、清掉里面的业务调用)，**不要删中断向量**，否则加 IP 时还得重建。
- **残留符号逐个清**：删了 main.c 里的业务，interrupt.c 若还 `extern` 引用它们(如 `byCntMs++`)，
  会报 undefined。把这些引用也一并去掉(它们本就是业务用的)。

## 白纸骨架 main.c 参考(最小)
```c
#include "inc.h"
extern void apt32f104_init(void);
extern void syscon_iwdt_reload(void);
extern void my_printf(const char *fmt, ...);

int main(void)
{
    volatile unsigned int cnt = 0;
    apt32f104_init();                 // 时钟/中断/console 底座
    my_printf("skeleton alive\r\n");  // 上电打印一次
    while (1) {
        syscon_iwdt_reload();         // 喂狗
        if (++cnt >= 200000) { cnt = 0; my_printf("heartbeat\r\n"); }
    }
}
```
initial.c 的 `apt32f104_init()` 里，去掉 `led_io_config/tk_config/bt_config` 等业务初始化，只留
`syscon_config()` + `console_init()` + 开中断。

## 判定白纸成立
整工程 make = 0 error → 得到"最小可编译骨架"。烧录后串口应打印心跳、芯片不死机。
这是逐 IP 的起点。**实测：APT32F104 白纸骨架烧录后 UART0@PA05,256000 正常打印 heartbeat。**

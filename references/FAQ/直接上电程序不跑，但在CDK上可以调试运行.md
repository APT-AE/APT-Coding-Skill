# 直接上电程序不跑，但在CDK上可以调试运行

有两个方向可以检查

# 程序使能了半主机调试——debug print，检查iostring\.c中的宏。如果进入else脱离PC会卡在while里。

```C++
void __putchar__ (char ch) 
{
#ifdef _DEBUG_UART_IO
        uart_write_byte(UART0,ch);                        //uart 0
//        uart_write_byte(UART1,ch);                        //uart 1
#else
        //select debug serial Pane
        volatile unsigned int *pdata = (unsigned int *)LDCC_DATA_P;
        while (*pdata & LDCC_BIT_STATUS);        //Waiting for data read.
        *pdata = ch;
#endif
```

# 下面两个条件同时满足时，会因为SRAM初始化问题导致程序无法运行

- CDK \-\> Project Settings \-\> Compiler中勾选了One Elf Section Per Data

- ld文件data段没有包含 \*\(\.data\.\*\)

解决方法：保持勾选 One Elf Section Per Data， ldld文件data段加入 \*\(\.data\.\*\)





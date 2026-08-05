# CountA使用注意事项

本文适用APT32F103x、APT32F173x系列

## 一、问题描述

有客户用APT32F1031芯片，用CountA做定时器并在CountA中断里面翻转IO口，发现IO翻转频率不一致，已排除其他中断影响。主要代码如下：

```C
void COUNTA_CONFIG(void)
{
    GPIO_Init(GPIOB0,7,0); 
    
    COUNT_DeInit();                                                                       //clear all countA Register
    //COUNTA_IO_Init(COUNTA_PB011);                                                       //set PB0.11 as counter IO
    COUNTA_Init(1000,0,Period_H,DIV1,REPEAT_MODE,CARRIER_OFF,OSP_LOW);                    //Data_H=Time*2/(1/sysclock),
    COUNTA_Config(SW_STROBE,PENDREM_OFF,MATCHREM_OFF,REMSTAT_0,ENVELOPE_0);               //countA mode set  
    COUNTA_Start();                                                                       //countA start
    //COUNTA_Stop();                                                                      //countA stop  
    COUNTA_Int_Enable();                                                                  //countA INT enable
}

void CNTAIntHandler(void) 
{
    // ISR content ...
    GPIO_Reverse(GPIOB0,7);
}
```


## 二、解决办法
我们需要去改变RISC V中CountA的中断触发特性：
在CountA初始化后面添加一句“CLIC\-\>CLICINT\[40\]\.ATTR \|= \(0x01 \<\< 1\);”，可解决此问题。

```C
void COUNTA_CONFIG(void)
{
    GPIO_Init(GPIOB0,7,0); 
    
    COUNT_DeInit();                                                                       //clear all countA Register
    //COUNTA_IO_Init(COUNTA_PB011);                                                       //set PB0.11 as counter IO
    COUNTA_Init(1000,0,Period_H,DIV1,REPEAT_MODE,CARRIER_OFF,OSP_LOW);                    //Data_H=Time*2/(1/sysclock),
    COUNTA_Config(SW_STROBE,PENDREM_OFF,MATCHREM_OFF,REMSTAT_0,ENVELOPE_0);               //countA mode set  
    COUNTA_Start();                                                                       //countA start
    //COUNTA_Stop();                                                                      //countA stop  
    COUNTA_Int_Enable();   
    
    CLIC->CLICINT[40].ATTR |= (0x01 << 1);                                                               //countA INT enable
}
```





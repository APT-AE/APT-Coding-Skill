# ADC连续转换模式从Deepsleep模式唤醒后卡死问题原因和解决办法（对外密）

此文档仅供APT内部人员查阅，严禁外发给外部人员查看！！

本文适用于APT32F102x、APT32F171、APT32FS003、APT32F110x系列

## 一、简述卡死原因

ADC的转换由逻辑部分和模拟部分共同控制，当ADC在连续模式下，正常情况下数字部分发送启动命令给模拟部分并等待模拟反馈EOC芯片，模拟接收到命令后开始转换且转换完成后发送EOC信号给数字部分，数字再执行读取ADC值、启动等操作，依次循环操作。

芯片进入Deepsleep模式时模拟部分会Powerdown，而数字部分只是暂停并且进模式前保存的状态继续维持。从Deepsleep模式唤醒数字部分会继续从进Deepsleep模式后的部分代码继续执行，模拟部分相当于复位。

ADC连续转换模式下MCU因为进入Deepsleep模式由客户程序控制，且进入时间点不好把控，所以就会存在数字在睡眠前已经发给模拟部分启动转换命令并等待EOC信号，而模拟部分也接受到命令准备转换或者已经转换一半就进入Powerdown模式了。当此类情况出现时就会导致MCU唤醒后数字部分继续工作且在等模拟部分的EOC信号，而模拟从Powerdown模式唤醒后把前面的转换任务丢弃了不会转换或发送EOC信号给数字部分，从而导致转换环路断掉，数字部分永远等不到EOC信号。

下面以APT32F102x\_StdPeriph\_Lib\_V1\_15库在1023学习板上测试为例，测试部分代码如下：

```C
/**************************************************/
int main(void) 
{
        uint8_t Temp = 0;
        
        delay_nms(3000);                                                        //power on delay if needed
        APT32F102_init();                                                        //102 initial
        printf("*** ADC test ***\n");
        while(1)
        {
                //SYSCON_IWDCNT_Reload();                                 //IWDT Clear
                for(Temp=0;Temp<20;Temp++)
                {
                        ADC0->CSR |= 0x01 << 2; //清溢出中断
                        //ADC12_EOC_wait();
                        ADC12_SEQEND_wait(0);
                        printf("Adc value = %d\n",ADC12_DATA_OUPUT(0));
                }
                
                SYSCON->GCER |= (1 << 12);    //使能低功耗的ISOSC
                SYSCON->GCDR |= (0xCAB << 8); //deepleep模式后禁止PCLK、HCLK、YSTICK、IMOSC、EMOSC、EMO_CKM、EMO_CMRST
                
                printf("Goto deepsleep mode, Press the button to wake up!! \n");
                delay_nms(10);
                PCLK_goto_deepsleep_mode();
                delay_nms(1000);   
                printf("MCU running!!\n");
    }
}
/******************* (C) COPYRIGHT 2019 APT Chip *****END OF FILE****/
```

上面测试程序逻辑：while循环里面连续读并串口打印20次ADC值，然后进Deepsleep模式等待按键唤醒。唤醒后继续转换20次再进Deepsleep模式，不断循环。但实际出现唤醒后只读取一次ADC值后就卡死再ADC的转换完成标志中，无法再往下执行。

程序卡在读标志语句中。

```C
void ADC12_SEQEND_wait(U8_T val)
{
        while(!(ADC0->SR & (0x01ul << (16+val))));                        // EOC wait
}
```

## 二、解决办法

在进Deepsleep模式前把ADC停掉，并读取一次状态值或者读取一次ADC转换值，唤醒后再启动ADC一次。

改动后代码如下：

```C
/**************************************************/
int main(void) 
{
        uint8_t Temp = 0;
        
        delay_nms(3000);                                                        //power on delay if needed
        APT32F102_init();                                                        //102 initial
        printf("*** ADC test ***\n");
        while(1)
        {
                //SYSCON_IWDCNT_Reload();                                 //IWDT Clear
                for(Temp=0;Temp<20;Temp++)
                {
                        ADC0->CSR |= 0x01 << 2; //清溢出中断
                        //ADC12_EOC_wait();
                        ADC12_SEQEND_wait(0);
                        printf("Adc value = %d\n",ADC12_DATA_OUPUT(0));
                }
                
                SYSCON->GCER |= (1 << 12);    //使能低功耗的ISOSC
                SYSCON->GCDR |= (0xCAB << 8); //deepleep模式后禁止PCLK、HCLK、YSTICK、IMOSC、EMOSC、EMO_CKM、EMO_CMRST
                
                printf("Goto deepsleep mode, Press the button to wake up!! \n");
                ADC12_Control(ADC12_STOP);
                ADC12_SEQEND_wait(0);
                delay_nms(10);
                PCLK_goto_deepsleep_mode();
                delay_nms(1000);
                ADC12_Control(ADC12_START);        
                printf("MCU running!!\n");
    }
}
/******************* (C) COPYRIGHT 2019 APT Chip *****END OF FILE****/
```

测试效果如下：




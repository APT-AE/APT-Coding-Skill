# ADC 使用INTVREF\(1V\)注意事项

本文适用STD库

## 一、问题描述

有客户用APT32F1023芯片在电池供电产品上，需要实时监测电池电压，ADC参考源选择INTVREF\(1V\)。客户用STD库默认配置，只配置通道及参考源，发现ADC采样出来ADC\-\>DR\[x\]的数据始终偏离理想值30\-40左右。程序配置如下：

```C
void ADC12_CONFIG(void)
{
        ADC12_Software_Reset();
        ADC12_CLK_CMD(ADC_CLK_CR , ENABLE);     //enable ADC CLK
        ADC12_Configure_Mode(ADC12_12BIT , Continuous_mode ,0, 6 ,2 , 1); //12BIT ADC; Continuous mode; Conversion priority selection 0; Holding cycles=6 ;ADC_CLK=PCLK/2*2=0.2us; Number of Conversions=1
        ADC12_Configure_VREF_Selecte(ADC12_VREFP_INTVREF1000_VREFN_VSS);  //ADC VREF Positive FVR4.096V,negative VSS
        ADC12_ConversionChannel_Config(ADC12_ADCIN0,ADC12_CV_RepeatNum1,ADC12_AVGDIS,0);  //SEQ0 chose ADCIN0, 6 Holding cycles, Average 1 time
        ADC12_CMD(ENABLE);                      //enable ADC
        ADC12_ready_wait();                                                                                                                                 //wait ADC get ready
        ADC12_Control(ADC12_START);                                                                                                                        //ADC convert start
}
```

## 二、问题原因

原因是ADC参考源选择INTVREF\(1V\)，ADC的时钟必须低于2MHz，需要修改ADC时钟分频配置参数（2改成12，即默认48MHz系统主频，经过2\*12分频后，FADC = 2MHz），ADC\-\>DR\[x\]的数据误差在理想范围内（±1）。

```C
void ADC12_CONFIG(void)
{
        ADC12_Software_Reset();
        ADC12_CLK_CMD(ADC_CLK_CR , ENABLE);     //enable ADC CLK
        ADC12_Configure_Mode(ADC12_12BIT , Continuous_mode ,0, 6 ,12 , 1); //12BIT ADC; Continuous mode; Conversion priority selection 0; Holding cycles=6 ;ADC_CLK=PCLK/2*2=0.2us; Number of Conversions=1
        ADC12_Configure_VREF_Selecte(ADC12_VREFP_INTVREF1000_VREFN_VSS);  //ADC VREF Positive FVR4.096V,negative VSS
        ADC12_ConversionChannel_Config(ADC12_ADCIN0,ADC12_CV_RepeatNum1,ADC12_AVGDIS,0);  //SEQ0 chose ADCIN0, 6 Holding cycles, Average 1 time
        ADC12_CMD(ENABLE);                      //enable ADC
        ADC12_ready_wait();                                                                                                                                 //wait ADC get ready
        ADC12_Control(ADC12_START);                                                                                                                        //ADC convert start
}
```




# 使用外部晶振CYOSC\_GM位配置注意事项

本文适用STD库，CSI库有根据晶振频率调整CYOSC\_GM位

## 一、问题描述

有客户用APT32F110系列芯片，采样EMOSC 外接32\.768KHz晶振做RTC的时钟源，出现个别芯片的32\.768K时钟变成194\.7KHz。

```C
void SYSCON_CONFIG(void)
{
//------SYSTEM CLK AND PCLK FUNTION---------------------------/
    SYSCON_RST_VALUE();                        //SYSCON all register clr
    SYSCON_General_CMD(ENABLE,ENDIS_ISOSC);                                                                                //SYSCON enable/disable clock source
//---------------------EMOSC Config---------------------------/
    EMOSC_OSTR_Config(0XAD,0X1f,EM_LFSEL_EN,EM_FLEN_EN,EM_FLSEL_10ns);//EM_CNT=0X3FF,0xAD(36K),EM_GM=0,Low F modedisable,EM filter disable,if enable,cont set 5ns
    SYSCON_General_CMD(ENABLE,ENDIS_EMOSC);
    //ESOSC_OSTR_Config(0XFF,7,EM_SMT_EN);                                                                                //ES_CNT=0XFF,ES_GM=7,Enable STM
    //SYSCON_General_CMD(ENABLE,ENDIS_ESOSC);
    SYSCON_HFOSC_SELECTE(HFOSC_SELECTE_48M);                                                                        //HFOSC selected 48MHz
    SystemCLK_HCLKDIV_PCLKDIV_Config(SYSCLK_HFOSC,HCLK_DIV_1,PCLK_DIV_1,HFOSC_48M);//system clock set, Hclk div ,Pclk div  set system clock=SystemCLK/Hclk div/Pclk div
    SYSCON_IWDCNT_Config(IWDT_TIME_2S,IWDT_INTW_DIV_7);                                              //WDT TIME 1s,WDT alarm interrupt time=1s-1s*1/8=0.875S
    SYSCON_WDT_CMD(ENABLE);                                    //enable WDT                
    SYSCON_IWDCNT_Reload();                                    //reload WDT
    SYSCON_LVD_Config(DISABLE_LVDEN,LVDRST_DIS,INTDET_LVL_2_7V,RSTDET_LVL_1_9V,INTDET_POL_fall);   
                                                                                               //Enable system wakeup INT        
}
```



## 二、问题原因

原因是客户配置代码时未修改SYSCON的CYOSC\_GM位配置，导致频率不对。根据使用手册中参数配置描述，外部晶振接32\.768KHz时，SYSCON的CYOSC\_GM位默认配置是0x1F，应当配置成0x07。

|EMOSC频率|CYO\_GM\[4:0\]|
|---|---|
|16MHz|11111|
|10MHz|11111|
|8MHz|11111|
|4MHz|11111|
|1MHz|11111|
|500KHz|11111|
|32\.768KHz|00111|

```C
void SYSCON_CONFIG(void)
{
//------SYSTEM CLK AND PCLK FUNTION---------------------------/
    SYSCON_RST_VALUE();                        //SYSCON all register clr
    SYSCON_General_CMD(ENABLE,ENDIS_ISOSC);                                                                                //SYSCON enable/disable clock source
//---------------------EMOSC Config---------------------------/
    EMOSC_OSTR_Config(0XAD,0X07,EM_LFSEL_EN,EM_FLEN_EN,EM_FLSEL_10ns);//EM_CNT=0X3FF,0xAD(36K),EM_GM=0,Low F modedisable,EM filter disable,if enable,cont set 5ns
    SYSCON_General_CMD(ENABLE,ENDIS_EMOSC);
    //ESOSC_OSTR_Config(0XFF,7,EM_SMT_EN);                                                                                //ES_CNT=0XFF,ES_GM=7,Enable STM
    //SYSCON_General_CMD(ENABLE,ENDIS_ESOSC);
    SYSCON_HFOSC_SELECTE(HFOSC_SELECTE_48M);                                                                        //HFOSC selected 48MHz
    SystemCLK_HCLKDIV_PCLKDIV_Config(SYSCLK_HFOSC,HCLK_DIV_1,PCLK_DIV_1,HFOSC_48M);//system clock set, Hclk div ,Pclk div  set system clock=SystemCLK/Hclk div/Pclk div
    SYSCON_IWDCNT_Config(IWDT_TIME_2S,IWDT_INTW_DIV_7);                                              //WDT TIME 1s,WDT alarm interrupt time=1s-1s*1/8=0.875S
    SYSCON_WDT_CMD(ENABLE);                                    //enable WDT                
    SYSCON_IWDCNT_Reload();                                    //reload WDT
    SYSCON_LVD_Config(DISABLE_LVDEN,LVDRST_DIS,INTDET_LVL_2_7V,RSTDET_LVL_1_9V,INTDET_POL_fall);   
                                                                                               //Enable system wakeup INT        
}
```

改完SYSCON的CYOSC\_GM位后输出频率正常




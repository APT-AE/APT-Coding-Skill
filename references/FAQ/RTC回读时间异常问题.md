# RTC回读位操作注意事项

本文适用APT32F102x、APT32F110x、APT32S003、APT32F173系列

## 一、问题描述

有客户在配置完RTC模块后，马上去读RTC时间数据出现时间值全为0。比如设定的的时间是9:55:55，读到的数据是00:00:00，持续时间大概1s左右后才恢复正常时间值。出现此现象是因为RTC回读功能一直是使能状态，且CPU时钟比RTC时钟快（RTC要1s更新一次寄存器值），导致读到的数据是RTC还未更新时后的数据。


## 二、解决办法

初始化RTC时先不使能RTC的回读位（RTC\-\>CR，bit 16），只在读时间时才打开回读位，并且读完再禁止回读位，这样初始化完后马上读就是设定的值，规避读到全0情况。

> ```C
> void csi_rtc_get_time(csp_rtc_t *ptRtc, csi_rtc_time_t *ptRtctime)
> {
>     csp_rtc_rb_enable(ptRtc,ENABLE);   //回读使能
>                 
>     ptRtctime->iSec = csp_rtc_read_sec(ptRtc);
>     ptRtctime->iMin = csp_rtc_read_min(ptRtc);
>     ptRtctime->iHour = csp_rtc_read_hour(ptRtc);
>     ptRtctime->iPm = csp_rtc_read_pm(ptRtc);
>     ptRtctime->iMday = csp_rtc_read_mday(ptRtc);
>     ptRtctime->iWday = csp_rtc_read_wday(ptRtc);
>     ptRtctime->iMon = csp_rtc_read_mon(ptRtc);
>     ptRtctime->iYear = csp_rtc_read_year(ptRtc);
>     
>     csp_rtc_rb_enable(ptRtc,DISABLE);  //回读禁止                                    
> }
> ```
> 
> 


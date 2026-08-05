# RTC DeepSleep下同步触发事件中断无法唤醒MCU说明

本文适用APT32F102x、APT32F110x、APT32S003、APT32F171、APT32F173系列

## 一、问题描述

有客户在使用APT32F1023的RTC进Deepsleep模式时，用RTC的CPRD、闹铃中断能正常唤醒Deepsleep模式，但改成同步触发事件中断后无法唤醒MCU。

## 二、问题说明

因为RTC同步触发事件中断会通过ETCB模块输出，并且ETCB模块必须用PCLK时钟源，默认Deepsleep模式下PCLK是关闭状态，导致RTC的触发中断无法唤醒MCU，所以Deepsleep模式下不支持同步触发事件中断来唤醒MCU。同样的道理，芯片所有模块的同步触发事件中断都不支持唤醒Deepsleep模式。




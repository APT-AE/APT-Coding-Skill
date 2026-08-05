# GPTx在各芯片上的特性对比

# 1、概述

简单来说，资源丰富程度：EPT \> GPTB \> GPTA

|产品|GPTA|GPTB|EPT|
|---|---|---|---|
|102x|**Y**|**N**|**Y**|
|103x|**Y**|**N**|**Y**|
|110x|**Y**|**Y**|**Y**|
|171x|**Y**|**N**|**Y**|
|173x|**Y**|**Y**|**N**|
|004x|**Y**|**N**|**Y**|
|104x|**Y**|**Y**|**N**|
|170x|**Y**|**Y**|**Y**|
|031x|**Y**|**Y**|**N**|
|005x|**Y**|**Y**|**N**|



# 2、特性对比

## 2\.1 GPTA

|**特性**|**102x**|**103x**|**110x**|**171x**|173**x**|170x|031x|005x|004x|104x|
|---|---|---|---|---|---|---|---|---|---|---|
|**计数方式**|UP/DN/UP\_DN||||||||UP||
|**PWM通道数**|2路独立输出||||||||1路输出||
|**群脉冲**|支持，CHA/CHB输入|||||支持，IO输入，外部AF功能|||支持，CHA输入||
|波形控制|支持，C1/C2可选择||||||||||
|**波形修改**|仅支持作用到内部PWM||||||||不支持||
|PHSR|不支持|支持||||不支持|||||
|**掩码控制**|不支持||||||||||
|通道回读监测|||||||||||
|**捕获**|SYNC2触发捕获，最大支持2个捕获事件||SYNC2触发捕获，最大支持4个捕获事件||SYNC2/3触发捕获，最大支持4个捕获事件||||SYNC2/3触发捕获，最大支持4个捕获事件，支持波形/捕获不互斥||
|**斩波**|不支持||||||||||
|**死区**|||||||||||
|**紧急模式**|||||||||||
|**SYNCIN**|SYNCIN0：外部Sync事件<br>SYNCIN1：Load触发<br>SYNCIN2：Capture触发事件<br>SYNCIN3：CNT增减一拍触发事件<br>SYNCIN4：T0事件（用于PWM波形输出控制）<br>SYNCIN5：T1事件（用于PWM波形输出控制）||||SYNCIN0：外部Sync事件<br>SYNCIN1：Load触发<br>SYNCIN2：Capture触发事件<br>SYNCIN3：Capture触发事件<br>SYNCIN4：CNT增减一拍触发事件<br>SYNCIN5：T0事件（用于PWM波形输出控制）<br>SYNCIN6：T1事件（用于PWM波形输出控制）|SYNCIN0：外部Sync事件<br>SYNCIN1：Load触发<br>SYNCIN2：Capture触发事件<br>SYNCIN3：Capture触发事件<br>SYNCIN4：CNT增减一拍触发事件|||||
|**TRGOUT**|支持，TRGEV0\~TRGEV1||||||||||
|**IP联动**|仅支持通过ETCB触发||||||||||
|**寄存器链接**|不支持|支持||不支持|支持||||||

## 2\.2 GPTB

|**特性**|**110x**|**173x**|104x|170x|031x|005x|
|---|---|---|---|---|---|---|
|**计数方式**|UP/DN/UP\_DN||||||
|时钟选择|PCLK||||PCLK/PLLCLK|PCLK|
|**PWM通道数**|2路独立或1组互补\+1路独立|||2路独立或1组互补|||
|**群脉冲**|支持，CHAX/CHB输入|||支持，IO输入，外部AF功能|||
|波形控制|支持，C1/C2可选择||||||
|**波形修改**|仅支持作用到内部PWM||支持作用到内部PWM和直接作用到管脚||||
|PHSR|支持|||不支持|||
|**掩码控制**|不支持||支持，比较软件强制赋值和MASK|支持，直接比较管脚和MASK|||
|**通道回读监测**|不支持|||支持|||
|**捕获**|SYNC2触发捕获，最大支持4个捕获事件|SYNC2/3触发捕获，最大支持4个捕获事件|||||
|**斩波**|不支持||||||
|**死区**|支持，Dead Band输入源可选择||||||
|**紧急模式**|支持，EP0\~EP3||支持，EP0\~EP1||支持，EP0\~EP1。\(EMSRC源选择比较特殊，需注意。\)||
|**SYNCIN**|SYNCIN0：外部Sync事件<br>SYNCIN1：Load触发<br>SYNCIN2：Capture触发事件<br>SYNCIN3：CNT增减一拍触发事件<br>SYNCIN4：T0事件（用于PWM波形输出控制）<br>SYNCIN5：T1事件（用于PWM波形输出控制）|SYNCIN0：外部Sync事件<br>SYNCIN1：Load触发<br>SYNCIN2：Capture触发事件<br>SYNCIN3：Capture触发事件<br>SYNCIN4：CNT增减一拍触发事件<br>SYNCIN5：T0事件（用于PWM波形输出控制）<br>SYNCIN6：T1事件（用于PWM波形输出控制）|SYNCIN0：外部Sync事件<br>SYNCIN1： Load触发<br>SYNCIN2： Capture触发事件<br>SYNCIN3： Capture触发事件<br>SYNCIN4： CNT增加一拍触发事件 \+ 软件直接force管脚启动事件<br>SYNCIN5：软件直接force管脚停止事件|SYNCIN0：外部Sync事件<br>SYNCIN1：Load触发<br>SYNCIN2：Capture触发事件<br>SYNCIN3：Capture触发事件<br>SYNCIN4：CNT增减一拍触发事件|||
|**TRGOUT**|支持，TRGEV0\~TRGEV1||||||
|**IP联动**|仅支持通过ETCB触发|||- ETCB触发其他IP<br>- 直接触发CMP|仅支持通过ETCB触发||
|**寄存器链接**|支持||||||




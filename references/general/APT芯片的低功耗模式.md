# APT芯片的低功耗模式

通常情况下APT芯片支持2种低功耗模式：SLEEP 和 DEEPSLEEP，有时候也称为IDLE和STOP。

有些芯片（如APT32F110x系列）在此基础上还支持SNOOZE和SHUTDOWN，这两个是更低的功耗模式。



|**MODE**|**ENTRY**|**WAKEUP**|**CLOCK STATUS**|**POWER STATUS**|
|---|---|---|---|---|
|LP RUN\(2\)|Set valid FLASH WAIT cycle<br>Set OPT1\[LPMD\] bit;|Clear OPT1\[LPMD\] bit;<br>Recover FLASH WAIT cycle|The same as normal|All on|
|SLEEP|DOZE Instruction|Any Interrupt|CPU Clock Off<br>No effect on other clock|All on|
|DEEP\-SLEEP|STOP Instruction|1\.Module: Set WKCR\[xxx\_WKEN\] as   wakeup source; Enable corresponding interrupt,including IWER in NVIC<br>2\.EXI:Set IO as input or any AF with   input function; Set trigger mode; enable EXI interrupt in GPIO,SYSCON and   IWER in NVIC |All Clock is Off, including CPU and   peripheral clock in default;<br>Clock source can be set to keep   working by STPEN bit|All on|
|SNOOZE（3）|WKCR\[SHD\_SNZ\_SEL\]=0xaa;<br>STOP Instruction|Same as DEEPSLEEP|All off except SRAM0/1 and wake up   logic;<br>Clock source can be set to keep   working by STPEN bit|All off except SRAM0/1, LCD, TOUCH,   enabled osc and wake up source|
|SHUTDOWN（4）|WKCR\[SHD\_SNZ\_SEL\]=0xa5;<br>STOP Instruction|1\.Module：Same as DEEPSLEEP<br>2\.ALV IO\(\(1\)\)：set WKCR\[WKENx\]; Set corresponding IO   as input|All off except SRAM1 and wake up   logic;<br>Clock source can be set to keep   working by STPEN bit|All off except SRAM1, enabled osc and   wake up logic|

NOTE

\(1\) Alive GPIO分布请参考 Datasheet Table 2\-1 管脚功能分配。

\(2\) LP RUN其实是低系统时钟的RUN 模式。




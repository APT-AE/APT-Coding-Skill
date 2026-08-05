# 关于EPT/GPTB模块的CHX/CHY输出波形的差异

适用于所有芯片。

# 文档说明

本文档用于说明EPT/GPTB模块的CHX/CHY输出波形之间的差异，以及如何修正差异。



# 差异说明

EPT模块和GPTB模块都带死区控制模块，在不使用死区模块的情况下，EPT/GPTB模块的同一个通道的CHX/CHY输出波形是不一致的，如CHAX和CHAY。死区控制模块的框图如下（下图以171 EPT模块为例）：

在不使用死区功能的情况下，S0、S1开关都是0，也就是将死区功能旁路掉，直接输出波形，此时X通道对应PWM1，Y通道对应PWM2，所以我们在CHX和CHY引脚上看到的波形是不一致的。



# 如何修正

如果需要CHX和CHY的波形一致，则需要通过配置死区控制模块来修正这个问题，将S0设置为1，S1依然是0，然后其他开关及边沿延时的参数设置为0即可，如下：

```C
csi_ept_deadzone_config_t  tEptDeadZoneTime;
tEptDeadZoneTime.byDcksel               = EPT_DB_DPSC;     
tEptDeadZoneTime.hwDpsc                 = 0;   
tEptDeadZoneTime.hwRisingEdgereGister   = 0;             //上升沿-ns
tEptDeadZoneTime.hwFallingEdgereGister  = 0;             //下降沿-ns
tEptDeadZoneTime.byChaDedb              = DB_AR_BF;      //不使用死区双沿
tEptDeadZoneTime.byChbDedb              = DB_AR_BF;
tEptDeadZoneTime.byChcDedb              = DB_AR_BF;
csi_ept_dz_config(EPT0,&tEptDeadZoneTime);

tEptDeadZoneTime.byChxOuselS1S0      = E_DBOUT_BF;      
tEptDeadZoneTime.byChxPolarityS3S2   = E_DAB_POL_DIS;      
tEptDeadZoneTime.byChxInselS5S4      = E_DBCHAIN_AR_AF;   
tEptDeadZoneTime.byChxOutSwapS8S7    = E_CHOUTX_OUA_OUB;   
csi_ept_channelmode_config(EPT0,&tEptDeadZoneTime,EPT_CHANNEL_1);
csi_ept_channelmode_config(EPT0,&tEptDeadZoneTime,EPT_CHANNEL_2);
csi_ept_channelmode_config(EPT0,&tEptDeadZoneTime,EPT_CHANNEL_3);
```


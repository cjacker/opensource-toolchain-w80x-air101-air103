# opensource-toolchain-w80x
Opensource toolchain for HLK w80x series based on T-Head XT-E804

# Draft, not finished, **to be written**


Hardware:
- HLK w80x board or Luat AIR 101/103
- cklink / cklink-lite / sipeed rv debugger plus / stm32f103

Software:
- compiler : csky gcc
- sdk: wm_sdk_w806/w80x
- debugger: t-head debug server / gdb
- programmer : wm_tool in wm_sdk_w806


C-Sky GCC is 6.3.0:

https://occ-oss-prod.oss-cn-hangzhou.aliyuncs.com/resource/1356021/1619529419771/csky-elf-noneabiv2-tools-x86_64-newlib-20210423.tar.gz

https://occ-oss-prod.oss-cn-hangzhou.aliyuncs.com/resource/1356021/1619529111421/csky-elfabiv2-tools-x86_64-minilibc-20210423.tar.gz

https://occ-oss-prod.oss-cn-hangzhou.aliyuncs.com/resource/1356021/1619527806432/csky-linux-uclibc-tools-x86_64-uclibc-linux-4.9.56-20210423.tar.gz


SDk is partial close source.

T-Head debug server is close source.


CK-Link Pin out:
```
 +-------------+
 | TDI ▪ • GND |
 | TDO • • GND |
   TCK • • GND |
    -- • • --  |
  nRST • • TMS |
 |  -- • • --  |
 |VREF • • TRST|
 +-------------+
```


Connection:

| CK-Link  | W806 board |
|----------|------------|
| VRef/3V3 | 3V3        |
| TRST     | RST        |
| TCK      | CLK(PA1)   |
| TMS      | DAT(PA4)   |
| GND      | GND        |

```
CKLink    :     W806
VREF/VCC  ->    3V3
TRST      ->    RST
TCK       ->    CLK(PA1)
TMS       ->    DAT(PA4)
GND       ->    GND
```


Debug server connected:

```
+---                                                    ---+
|  T-Head Debugger Server (Build: Jan  9 2023, Linux)      |
   User   Layer Version : 5.16.07
   Target Layer version : 2.0
|  Copyright (C) 2022 T-HEAD Semiconductor Co.,Ltd.        |
+---                                                    ---+
T-HEAD: CKLink_Lite_V2, App_ver 2.36, Bit_ver null, Clock 2526.316KHz,
       2-wire, With DDC, Cache Flush On, SN CKLink_Lite_V2-650763C7D6.
+--  Debug Arch is CKHAD.  --+
+--  CPU 0  --+
T-HEAD Xuan Tie CPU Info:
        WORD[0]: 0x04a11453
        WORD[1]: 0x11000000
        WORD[2]: 0x21400417
        WORD[3]: 0x30c00005
Target Chip Info:
        CPU Type is CK804FGT, in LITTLE Endian.
        L1ICache size 16KByte.
        Bus type is AHB32.
        Signoff date is 04/0107.
        HWBKPT number is 5, HWWP number is 2.

GDB connection command for CPUs(CPU0):
        target remote 127.0.0.1:1025
        target remote 192.168.2.148:1025
        target remote 192.168.122.1:1025
        target remote 172.17.0.1:1025
```


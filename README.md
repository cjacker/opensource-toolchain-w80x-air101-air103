# opensource toolchain for HLK W80x and Luat AIR101 / 103

W80x are made by [WinnerMicro](https://www.winnermacro.com) and are based on one 240 Mhz XT-E804 core. The core itself is made on top of the C-SKY architecture (Not ARM, Not RISCV) developed by C-SKY Microsystems Co.,Ltd. C-SKY Microsystems was aquired by Ali serveral years ago and renamed to [T-Head](https://www.t-head.cn)).

Somebody may refer these chips as 'HLK-W80x', it's a mistake. 'W80x' is the correct name, 'HLK-W80x-KIT' stands for a series boards made by [Hi-Link](https://www.hlktech.com).

W80x line-up include 3 microcontrollers:
- W806 - XT-E804, QFN56, 6mmx6mm.
- W801 - XT-E804 + Wi-Fi + Bluetooth, QFN56, 6mmx6mm. **W801 = W806 + Wi-Fi + BT**
- W800 - XT-E804 + Wi-Fi + Bluetooth, QFN32, 4mmx4mm. **W800 = W801 - some pins**

All these chips are available as devboards (about 1 $).

Besides W80x, there is another vendor 'Luat' also made MCU based on XT-E804, includes AIR101 and 103.
- AIR101 - XT-E804, QFN32, 4mmx4mm
- AIR103 - XT-E804, QFN56, 6mmx6mm **AIR103 = W806**

All these 2 chips are also availble as devboards (about 1 $)

Let's focus on the base model W806 / AIR103. The specification:

- 32-bit XT-E804 processor, frequency up to 240MHz, built-in DSP, FPU and security engine
- Built-in 1MB Flash, 288KB RAM
- Integrated PSRAM interface, supports up to 64MB external PSRAM memory
- 6-channel UART high-speed interface
- 4-channel 16-bit ADC, the highest sampling rate is 1KHz
- 1 high-speed SPI interface (slave interface), supports up to 50MHz
- master/slave SPI interface
- 1 SDIO_HOST interface, supports SDIO2.0, SDHC, MMC4.2
- 1 SDIO_DEVICE, supports SDIO2.0, the maximum throughput rate is 200Mbps
- 1 I²C controller
- Integrated GPIO controller supporting up to 44 GPIOs
- 5 channel PWM interface
- 1 Duplex I²S controller
- LCD controller, supports 4x32 interface
- 1 7816 interface
- 15 Touch Sensors integrated

Power supply:
- 3.3V single power supply
- Support work, sleep, standby, shutdown modes
- Standby power consumption is less than 10uA


# Hardware prerequiest
- AIR 101 or 103 devboards and HLK-W800-KIT
  + HLK-W806 devboards is not USB type-c and not suite for breadboard.
  + HLK-W801 devboards has some hardware issue with Linux
  
- a CK-Link Debugger
  + CK-Link Pro / Lite (a little bit expensive)
  + Sipeed rv debugger plus can work as a CK-Link lite
  + stm32f103 with CK-Link lite firmware.
  

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


# opensource toolchain for W80x and AIR101 / 103

W80x series MCU are made by [WinnerMicro](https://www.winnermacro.com), these series MCU are based on one 240 Mhz XT-E804 core. The core itself is made on top of the C-SKY architecture (Not ARM, Not RISCV) developed by C-SKY Microsystems. Serveral years ago, C-Sky Microsystems was aquired by Alibaba and renamed to [T-Head](https://www.t-head.cn)).

Somebody may refer these chips as 'HLK-W80x', it's a mistake. 'W80x' is the correct name, 'HLK-W80x-KIT' stands for a series boards made by [Hi-Link](https://www.hlktech.com).

W80x line-up include 3 microcontrollers:
- W806 - XT-E804, QFN56, 6mmx6mm. 1M FLASH, 288K RAM.
- W801 - XT-E804 + Wi-Fi + Bluetooth, QFN56, 6mmx6mm. 2M FLASH, 288K RAM. **W801 = W806 + Wi-Fi + BT**
- W800 - XT-E804 + Wi-Fi + Bluetooth, QFN32, 4mmx4mm. 2M FLASH, 288K RAM. **W800 = W801 - some pins**

All these chips are available as devboards (about 1 $).

Besides W80x, there is another vendor 'Luat' also made MCU based on XT-E804, includes AIR101 and 103.
- AIR101 - XT-E804, QFN32, 4mmx4mm. **2M FLASH, 176K RAM**.
- AIR103 - XT-E804, QFN56, 6mmx6mm. 1M FLASH, 288K RAM. **AIR103 = W806**

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
  + I recommend AIR 101 or AIR 103 for C-Sky XT804. AIR 101 have more flash space, less ram and less pins than AIR 103.
  + HLK-W806-KIT devboards is not USB type-c and not suite for breadboard.
  + HLK-W806-KIT need manual reset when programming.
  + HLK-W801-KIT devboards has some hardware issue with Linux
  + HLK-W800-KIT have base and pro version
  + do NOT buy **AIR105** for XT804, it's based on ARM core and even didn't export the SWD interface.
- CK-Link debugger
  + Option 1: T-Head or HLK CK-Link debugger
  + Option 2: STM32F103 bluepill with modified CK-Link lite firmware.

<img src="https://github.com/cjacker/opensource-toolchain-w80x/raw/main/boards.png" />

# Toolchain overview
- Compiler: C-Sky GNU Toolchain
- SDK : wm_sdk_w806 / wm_sdk_w80x / LuatOS
- Programming: wm_tool in wm_sdk
- Debugging: c-sky debug server / gdb

**NOTE:**
- The APIs of 'wm_sdk_w806' and 'wm_sdk_w80x' differs a lot. 
- 'wm_sdk_w806' can be used with AIR101 / 103 / W806 / W801 / W800, but lack Wi-Fi / BT libraries.
- **'libdsp.a'** (csi-dsp) and 'secboot.bin' in wm_sdk_w806 is **close source**. If you do not use DSP, you can get rid of it. 
- the **libraries of Wi-Fi / BT** in wm_sdk_w80x is **close source**.
- **'C-Sky debug server'** is **close source**.

Since the C-Sky debug server is **close source**, this is not a complete opensource toolchain.

# Compiler

T-Head provide prebuilt C-Sky GNU toolchain (GCC v6.3.0), you can download it from T-Head [Open Chip Community](https://occ.t-head.cn/community/download?id=3885366095506644992), but it may require registering an account first.

To skip the registration process, You can download the C-Sky GNU toolchain from [here](https://occ-oss-prod.oss-cn-hangzhou.aliyuncs.com/resource/1356021/1619529111421/csky-elfabiv2-tools-x86_64-minilibc-20210423.tar.gz) directly. I also upload a copy to github, you can also download it from [here](https://github.com/cjacker/wm_sdk_w80x/releases/download/init/csky-elfabiv2-tools-x86_64-minilibc-20210423.tar.gz).

After download it, extract it to somewhere, for example:

```
mkdir /opt/csky-toolchain
sudo tar zxvf csky-elfabiv2-tools-x86_64-minilibc-20210423.tar.gz -C /opt/csky-toolchain
```

And, do not forget add `/opt/csky-toolchain/bin` to PATH env.

The triplet of C-Sky GNU toolchain is **csky-abiv2-elf**.


# SDK

## wm_sdk
As mentioned above:

- [wm_sdk_w806](https://github.com/IOsetting/wm-sdk-w806) can be used for W806 / AIR101 / AIR103, you can also use it with W801 / W800, but lack Wi-Fi / BT support. 
- [wm_sdk_w80x](https://github.com/cjacker/wm_sdk_w80x) can be used for W801 / W800, it provide Wi-Fi / BT support static library (but without source).

Although, APIs of these 2 SDK differ a lot, but the sdk dir structure, code organization and usage is similar.

Use 'wm_sdk_w806' as example.

```
git clone https://github.com/IOsetting/wm-sdk-w806.git
```


The dir structure of 'wm_sdk_w806' SDK:

```
wm-sdk-w806
├─app              # User application code
├─bin              # Compilation results
├─demo             # Examples
├─include          # SDK header files 
├─ld               # Linker scripts
├─lib              # DSP Libraries in binary form
├─Makefile
├─platform         # SDK source code
└─tools            # Utilities
```

Compare with 'wm-sdk-w806', 'wm-sdk-w80x' SDK also have 'src' dir contains source codes for some static binary libraries (but not all).

The default user application codes of 'wm_sdk_w806' is for 'HLK-W806-KIT' devboard to blink 3 LEDs connected to 'PB0 / PB1 / PB2' (PWM 0 / 1 /2). If you use AIR 103 devboard, 3 LEDs on-board is connected to PB24 / PB25 / PB26, you have to modify the codes.

To build the 'user application codes', type 'make' directly. the target file 'W806.\*' will be generated at 'bin/W806' dir.

You can use `make menuconfig` to change some options of the SDK. for more usage, try `make help`.

## LuatOS

LuatOS is a powerful embedded Lua Engine for IoT devices, with many components and low memory requirements (16K RAM, 128K Flash), it is developed by OpenLuat community and by default support almost all devboards from OpenLuat, such as AIR 101 and AIR 103, also support various ARM based MCU. Since this tutorial is for C-Sky XT804, I will focus on AIR 101 and 103. 

**Note :** LuatOS also can be used with W80X devboard.

### build from source

LuatOS released in source and prebuilt-binary format, if you like to use the prebuilt version for various SOC LuatOS already supported, you can ignore this section.

### install xmake
Building LuatOS requires `xmake` installed, which not shipped by most distributions. you have to install it yourself:
```
$ mkdir xmake-build && cd xmake-build
$ git clone https://github.com/xmake-io/xmake.git
$ git clone https://github.com/xmake-io/xmake-core-lua-cjson.git
$ git clone https://github.com/xmake-io/xmake-core-lua.git
$ git clone https://github.com/xmake-io/xmake-core-luajit.git
$ git clone https://github.com/xmake-io/xmake-core-pdcurses.git
$ git clone https://github.com/xmake-io/xmake-core-sv.git
$ git clone https://github.com/xmake-io/xmake-core-lz4.git
$ git clone https://github.com/tboox/tbox.git

$ cd xmake
$ git submodule init
$ git config submodule.core/src/lua-cjson/lua-cjson.url "$(pwd)/../xmake-core-lua-cjson"
$ git config submodule.core/src/lua/lua.url "$(pwd)/../xmake-core-lua"
$ git config submodule.core/src/luajit/luajit.url "$(pwd)/../xmake-core-luajit"
$ git config submodule.core/src/pdcurses/pdcurses.url "$(pwd)/../xmake-core-pdcurses"
$ git config submodule.core/src/sv/sv.url "$(pwd)/../xmake-core-sv"
$ git config submodule.core/src/lz4/lz4.url "$(pwd)/../xmake-core-lz4"
$ git config submodule.core/src/tbox/tbox.url "$(pwd)/../tbox"
$ git submodule update

$ ./configure --prefix=/usr
$ make
$ sudo make install
```

`xmake` and `xrepo` commands will be installed to `/usr/bin` dir.

### build LuatOS

```
$ mkdir luatos-build && cd luatos-build
$ git clone https://gitee.com/openLuat/LuatOS.git
$ git clone https://gitee.com/openLuat/luatos-soc-air101.git
```
There should have 2 dirs in luatos-build dir, `LuatOS` and `luatos-soc-air101`, don't rename any of them.

To build AIR101 firmware:

```
cd luatos-soc-air101
gcc -o tools/xt804/wm_tool tools/xt804/wm_tool.c
xmake
```

After building finished, the firmwares located at `build/out` dir, what we need is `AIR101.fls` and `LuatOS-SoC_V0016_AIR101.soc`. `soc` file is a 7za file and can be extracted by `7za x`, it can be programed by LuatTools (only have windows support). `AIR101.fls` is the firmware we will used later.

To build AIR103 firmware, you need :

```
cd luatos-soc-air101
sed -i 's/#define AIR101/#define AIR103/g' app/port/luat_conf_bsp.h
gcc -o tools/xt804/wm_tool tools/xt804/wm_tool.c
xmake
```

Because the flash size of AIR103 is 1M and smaller than 2M of AIR101, if you encouter `I-SRAM' overflowed by  XXXX` error, you need edit `app/port/luat_conf_bsp.h` to comment out some features, such as `u8g2/lcd/lvgl`.

### use prebuilt release
LuatOS upstream released a set of pre-built firmwares for various SOC and OpenLuat devboards, you can download it from [LuatOS project](https://gitee.com/openLuat/LuatOS/releases) and use these pre-built firmware directly.

For AIR101 : https://gitee.com/openLuat/LuatOS/releases/download/v0007.air101.v0015/core_V0015.zip
For AIR103 : https://gitee.com/openLuat/LuatOS/releases/download/v0007.air103.v0015/core_V0015.zip

After download and extracted, you will find a `LuatOS-SoC_XXXX.soc` file, it's 7z file and can be extracted as:

```
7za x LuatOS-SoC_XXXX.soc
```

And you will get `AIR101.fls` or `AIR103.fls`, this is the firmware we will used later.


# Programming

## for wm_sdk_w806/w80x

Before start programming, you need connect the devboard to PC and get the USB uart device name, usually, it's `ttyUSB0`. you can find it with `ls /dev/tty*`.

Then, run `make menuconfig`, goto 'Download Configuration', set 'download port' to `ttyUSB0`. Then save / exit menuconfig and program the target firmware to device by:

```
make flash
```

The output looks like:

```
connecting serial...
serial connected.
wait serial sync.........
please manually reset the device.
```

For HLK-W806-KIT, you need press the 'RESET' button here to continue.

```
serial sync sucess.
mac CC-CC-CC-CC-CC-CC.
start download.
0% [###] 100%
download completed.
reset command has been sent.
```

For AIR 101 / 103 devboards and W801 / W800 devboards, the RTS pin of on-board ch34x already connect to RESET pin of MCU, there is no response if you press the RESET button when programming, you need apply a patch to enable auto-reset. To enable it, at the top dir of 'wm_sdk_w806' or 'wm_sdk_w80x', run:

```
sed -i 's/-rs at/-rs rts/g' tools/W806/rules.mk
```


## for LuatOS

LuatOS hasn't a standard programming tool, because it's a software and not a hardware. how to program it depends on specific hardware platform. For ARM based SOC, it can be programmbed by CMSIS-DAP, ST-Link, etc. For AIR101 / 103, since it's C-Sky XT804, LuatOS will use `wm_tool` to program to target device, actually we already use `wm_tool` with wm_sdk.

LuatTools is close source flash tool for Windows provided by LuatOS upstream, and it can not support Linux even with 'wine'. Honestly speaking, the linux supporting from upstream is not very well since they all may use windows as first development platform.

After talked with the author of LuatOS, I decided to write a set of utilities named [luatos-utils](https://github.com/cjacker/luatos-utils) for linux to make things easy.

```
git clone https://github.com/cjacker/luatos-utils.git
cd luatos-utils
make
```

After built successfully, you should have below commands at current dir:
- luac : 32bit ELF build from upstream lua 5.3.6 and with `-DLUA_32BITS`.
- mkscriptbin : convert dir to luadb binary file.
- wm_tool_luatos : firmware pre-process and program tool.
- gen-script-img : generate 'script.img' for AIR101 or AIR103
- flash-script-img : flash script.img to AIR101 or AIR103
- flash-base-firmware: flash base firmware to AIR101 or AIR103

In general, you can treat firmwares for LuatOS as two part:
- the base firmware, we built it in SDK section, such as 'AIR101.fls' and 'AIR103.fls'. 
- the script firmware, lua script you write to run on base firmware.

The base firmware can be programmed once and only need to re-program when it has a new release. The script firmware is what we will write and maybe will program many times.

To program base firmware, connect the TypeC port of AIR101 devboard to PC USB port, using 'AIR101.fls' we built in SDK section as example:

```
./wm_tool_luatos -ds 2M -c ttyUSB0 -ws 115200 -rs rts -dl AIR101.fls
```

In short, use the 'flash-base-firmware' script:
```
./flash-base-firmware AIR101.fls
```

To program lua script, you need:
- create a dir and put all lua scripts and resources into it. note, subdirs is not supported, it also not supported by upstream.
- run `gen-script-img [air101 or air103] <source dir>`, a `script.img` will generated.
- run `flash-script-img [air101 or air103] <script.img>` to program the `script.img` to target device.

Using `lvgl-demo` provide with luat-utils` as example:

```
./gen-script-img air101 lvgl-demo
./flash-script-img air101 script-for-air101.img
```

# Debugging

## For wm_sdk

To debugging XT-E804 based MCU, you have to use a CK-Link debugger and T-Head debug server (close source software).

**If you have any stm32f103 devboard (such as bluepill), there is not neccesary to buy a CK-Link debugger, please refer to below sections to make a CK-Link Lite debugger yourself.**

## Install C-Sky debug server

The T-Head debug server can be downlowed from https://occ-oss-prod.oss-cn-hangzhou.aliyuncs.com/resource//1673423345494/T-Head-DebugServer-linux-x86_64-V5.16.7-20230109.sh.tar.gz

After download finished:

```
tar xf T-Head-DebugServer-linux-x86_64-V5.16.7-20230109.sh.tar.gz
tail -n +282 T-Head-DebugServer-linux-x86_64-V5.16.7-20230109.sh >csky-debug-server.tar.gz
```
Then extract the 'csky-debug-server.tar.gz' to somewhere, for example:
```
mkdir -p /opt/csky-debug-server
sudo tar zxf csky-debug-server.tar.gz -C /opt/csky-debug-server
```

The command of csky-debug-server is `DebugServerConsole.elf`, it depend on some libraries installed at `/opt/csky-debug-server` dir,you have to run it as:

```
cd /opt/csky-debug-server
./DebugServerConsole.elf
```

Or

```
LD_LIBRARY_PATH=/opt/csky-debug-server /opt/csky-debug-server/DebugServerConsole.elf
```

You can save below wrapper script as `csky-debug-server` and put it to `/usr/bin`:

```
#!/usr/bin/bash
cd /opt/csky-debug-server
export LD_LIBRARY_PATH=/opt/csky-debug-server
./DebugServerConsole.elf $@
```

You may also need to create a udev rule '/etc/udev/rules.d/99-csky-cklink.rules' with below contents to set device permission correctly (allow normal user read / write the CK-Link device).

```
# For Hi-Link CK-Link lite
SUBSYSTEM=="usb", ATTR{idVendor}="32bf", ATTR{idProduct}=="b210", MODE="666"
```

After this udev rules saved, please run:
```
udevadm trigger
udevadm control --reload
```

### Option 1 : Use T-Head or HLK CK-Link debugger

The Ck-Link pinout:

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

Official CK-Link debugger from T-Head or HLK do not supply power to target board, you need supply power to target board with another USB cable.

Connect target board to CK-Link (refer to below table) and plug CK-Link to PC USB port:

| CK-Link  |   board    |
|----------|------------|
| VRef     | 3V3        |
| TRST     | RST        |
| TCK      | CLK(PA1)   |
| TMS      | DAT(PA4)   |
| GND      | GND        |


### Option 2 : Make your own CK-Link Lite debugger with STM32F103

C-Sky debug server contains a set of cklink firmware files, if you have a STM32F103 devboard, you can use 'cklink_lite.hex' shipped with C-Sky debug server to make your own CK-Link debugger. 

'cklink_lite_iap.hex' (address range from 0x0800_0000 to 0x0800_4000) is the bootloader and 'cklink_lite.hex' (address range start from 0x0800_4000) is the firmware. the IAP firmware not works with STM32F103 bluepill (due to the circuit differences between CKLink Lite debugger and STM32F103 bluepill), we need modify the 'cklink_lite.hex' first to copy the vector table to the beginning of the flash:

- open 'cklink_lite.hex' in your favorite editor.
- copy the lines before address 0x4100 and paste them to the start of file.
- modify address of all lines just copied, from 0x40XX to 0x00XX.
- fix the checksum of all lines just modified.

I wrote a tool to convert it automatically, you can find it in [cklink-lite-fw-convertor](https://github.com/cjacker/cklink-lite-fw-convertor) repo.

And the pre-converted '[cklink-lite-2.36_for-stm32f103.hex](https://raw.githubusercontent.com/cjacker/cklink-lite-fw-convertor/main/cklink_lite-2.36_for-stm32f103.hex) can programmed to STM32F103 directly. If the firmware version is outdated, you can use `cklink-lite-fw-convertor` to convert the latest firmware yourself.

After STM32F103 programmed, connect STM32F103 with your target devboard as below table and plug STM32F103 to PC USB port.

| STM32F103 | XT-E804  |
|-----------|----------|
| A0        | RESET    |
| A1        | CLK(PA1) |
| A5        | DAT(PA4) |
| 3V3       | 3V3      |
| GND       | GND      |


### Launch C-Sky debug server

Then invoke C-Sky debug server as mentioned above:

```
# here I use wrapper script
csky-debug-server
```

If all good, the output looks like:

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
```

### Debug

Then open new terminal window, and run:

```
csky-abiv2-elf-gdb bin/build/W806/image/W806.elf
```

After '(cskygdb)' show up:

```
(cskygdb) target remote :1025
Remote debugging using :1025
0x08013482 in HAL_GetTick () at wm_cpu.c:108
108         return uwTick;
(cskygdb) b main
Breakpoint 1 at 0x8011988: file main.c, line 46.
(cskygdb) c
```


## For LuatOS

LuatOS debugging heavily depends on UART print out, the default baudrate is 921600, it's defined by LuatOS base firmware for AIR101 and AIR103, I use `tio` and `picocom` as example:

tio:
```
tio -b 921600 /dev/ttyUSB0 -m INLCRNL
```

picocom:
```
picocom -b 921600 /dev/ttyUSB0 --imap lfcrlf
```


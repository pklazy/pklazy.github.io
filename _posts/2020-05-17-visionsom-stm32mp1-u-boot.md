---
layout: post
title: "VisionSOM-STM32MP1 U-Boot 기록"
date: 2020-05-17 03:00:00 +0900
categories: embedded
tags: [STM32MP1, U-Boot]
---

https://github.com/pklazy/u-boot/commit/0a03a79589128fda9b10fc8a9b46533676467c06

```
[0.000002 0.000002]
[0.000224 0.000222] U-Boot SPL 2020.04-00002-g0a03a79589 (May 17 2020 - 01:51:44 +0900)
[0.005901 0.005677] Model: SomLabs VisionSOM-STM32MP1
[0.015813 0.009912] RAM: DDR3-DDR3L 16bits 400000Khz
[0.033797 0.017984] [722905] >>SPL: board_init_r()
[0.037724 0.003927] [726507] spl_init
[0.039011 0.001287] Trying to boot from MMC1
[0.082817 0.043806] [772178] SPL: payload image: U-Boot 2020.04-00002-g0a03a79589 load addr: 0xc00fffc0 size: 790650
[0.153625 0.070809] [842792] Jumping to U-Boot
[0.156511 0.002885] [843752] loaded - jumping to U-Boot...
[0.160022 0.003511] [847224] image entry point: 0xc0100000
[1.207782 1.047760] [850706] initcall: c0113495
[1.210950 0.003168] [850779] initcall: c016ab45
[1.213368 0.002418] [850975] initcall: c011954b
[1.215735 0.002367] [851037] initcall: c011348d
[1.218209 0.002474] [851099] initcall: c011389b
[1.220842 0.002633] [851161] initcall: c01134ad
[1.222949 0.002107] [851222] initcall: c01138c7
[1.225820 0.002871] [851284] initcall: c0101ed9
[1.227933 0.002113] [851347] initcall: c01138ab
[1.230671 0.002738] [851409] initcall: c0113895
[1.232891 0.002220] [1387295] initcall: c01138cf
[1.235880 0.002989] [1387359] initcall: c0100fbb
[1.238745 0.002865] [1387423] initcall: c014a3b9
[1.241374 0.002629] [1387492] initcall: c0113879
[1.243749 0.002375] [1387597] initcall: c0141fad
[1.246018 0.002269] [1896426] initcall: c01188a1
[1.248856 0.002838] [1936506] initcall: c016b631
[1.251750 0.002894]
[1.251813 0.000062]
[1.251850 0.000037] U-Boot 2020.04-00002-g0a03a79589 (May 17 2020 - 01:51:44 +0900)
[1.257709 0.005859]
[1.257753 0.000045] [1945272] initcall: c01136c9
[1.259953 0.002200] [1947876] U-Boot code: C0100000 -> C01A21C4  BSS: -> C01AEA64
[1.265769 0.005816] [1953346] initcall: c01138d3
[1.268601 0.002833] [1955950] initcall: c0113821
[1.371964 0.103363] [2060870] initcall: c0101f61
[1.402785 0.030821] CPU: STM32MP157AAB Rev.B
[1.405536 0.002751] [2092867] initcall: c0113d11
[1.408101 0.002565] Model: SomLabs VisionSOM-STM32MP1
[1.411157 0.003056] Board: stm32mp1 in basic mode (st,stm32mp157)
[1.415219 0.004062] [2102601] initcall: c0113811
[1.417573 0.002354] DRAM:  [2105800] initcall: c010226d
[1.604756 0.187183] [2293700] initcall: c011397d
[1.607204 0.002448] [2294859] Monitor len: 000AEA64
[1.610059 0.002855] [2297724] Ram size: 20000000
[1.612792 0.002733] [2300330] Ram top: E0000000
[1.615165 0.002373] [2302845] initcall: c01134b1
[1.617698 0.002532] [2305450] initcall: c01134e1
[1.620925 0.003227] [2308054] TLB table from dfff0000 to dfff4000
[1.624687 0.003762] [2312133] initcall: c01137f5
[1.626884 0.002196] [2314767] initcall: c01138af
[1.629633 0.002749] [2317342] initcall: c0113671
[1.632531 0.002899] [2319945] Reserving 698k for U-Boot at: dff41000
[1.636720 0.004188] [2324286] initcall: c01135f5
[1.639013 0.002293] [2326890] Reserving 32776k for malloc() at: ddf3f000
[1.643836 0.004823] [2331576] Reserving 1M for noncached_alloc() at: dde00000
[1.648858 0.005022] [2336697] initcall: c01137ad
[1.651702 0.002844] [2339302] Reserving 80 Bytes for Board Info at: dddfffb0
[1.656697 0.004995] [2344337] initcall: c01138b3
[1.659529 0.002832] [2346939] initcall: c01135bd
[1.661806 0.002277] [2349544] Reserving 224 Bytes for Global Data at: dddffed0
[1.666922 0.005116] [2354753] initcall: c011355d
[1.669691 0.002769] [2357357] Reserving 47936 Bytes for FDT at: dddf4390
[1.674585 0.004894] [2362043] initcall: c01138b7
[1.676845 0.002260] [2364648] initcall: c01138bb
[1.679640 0.002795] [2367252] initcall: c01138cb
[1.681906 0.002266] [2369854] initcall: c0113a01
[1.684782 0.002876] [2372460] initcall: c01134c5
[1.687590 0.002808] [2375065] initcall: c01138dd
[1.689888 0.002298] [2377666]
[1.690914 0.001026] RAM Configuration:
[1.692781 0.001867] [2380444] Bank #0: c0000000 512 MiB
[1.695824 0.003043] [2383655]
[1.696842 0.001018] DRAM:  512 MiB
[1.698587 0.001745] [2386086] initcall: c0113539
[1.700823 0.002236] [2388690] New Stack Pointer is: dddf4370
[1.704680 0.003857] [2392336] initcall: c0113781
[1.707564 0.002884] [2395614] initcall: c01138bf
[1.709848 0.002284] [2397545] initcall: c01138c3
[1.712692 0.002843] [2400147] initcall: c01136fd
[1.714937 0.002245] [2402755] Relocation Offset is: 1fe41000
[1.718825 0.003888] [2406397] Relocating to dff41000, new gd at dddffed0, sp at dddf4370
[1.724856 0.006031] [2412474] initcall: c01138a7
[1.756940 0.032084] [2445438] initcall: dff54aa1
[1.759121 0.002181] [2446605] initcall: dff54aa5
[1.762681 0.003560] [2449209] initcall: c0113c55 (relocated to dff54c55)
[1.767021 0.004340] [2455556] initcall: c0113c1d (relocated to dff54c1d)
[1.770893 0.003871] [2458787] initcall: c0113c63 (relocated to dff54c63)
[1.775727 0.004835] [2463462] initcall: c0113bdd (relocated to dff54bdd)
[1.780757 0.005029] [2468149] Pre-reloc malloc() used 0x1308 bytes (4 KB)
[1.812885 0.032128] [2501763] initcall: c0113491 (relocated to dff54491)
[1.817804 0.004919] [2504977] initcall: c0113bcf (relocated to dff54bcf)
[1.822098 0.004294] [2509664] initcall: c0113c67 (relocated to dff54c67)
[1.826699 0.004601] [2514351] initcall: c0113bc5 (relocated to dff54bc5)
[1.831759 0.005060] [2519041] initcall: c0113bb3 (relocated to dff54bb3)
[1.855939 0.024180] [2545063] initcall: c0102ed1 (relocated to dff43ed1)
[1.864939 0.009000] Clocks:
[1.865794 0.000855] - MPU : 800 MHz
[1.866888 0.001094] - MCU : 200 MHz
[1.868605 0.001717] - AXI : 200 MHz
[1.869801 0.001196] - PER : 0 MHz
[1.870981 0.001180] - DDR : 400 MHz
[1.879820 0.008838] [2569292] initcall: c0160451 (relocated to dffa1451)
[1.885031 0.005212] [2572519] initcall: c0113c5f (relocated to dff54c5f)
[1.889779 0.004747] [2577193] initcall: c0113c6b (relocated to dff54c6b)
[1.893919 0.004140] [2581880] initcall: c011be8d (relocated to dff5ce8d)
[1.898897 0.004978] [2586568] initcall: c0113ba9 (relocated to dff54ba9)
[1.919985 0.021088] [2608917] initcall: c0113b85 (relocated to dff54b85)
[1.925308 0.005323] [2612212] Now running in RAM - U-Boot at: dff41000
[1.928957 0.003649] [2616726] initcall: c0113c6f (relocated to dff54c6f)
[1.933924 0.004967] [2621413] initcall: c0113b61 (relocated to dff54b61)
[1.938786 0.004862] NAND:  0 MiB
[1.939837 0.001051] [2627316] initcall: c0113b49 (relocated to dff54b49)
[1.944781 0.004944] MMC:   STM32 SDMMC2: 0
[1.977170 0.032389] [2664334] initcall: c0113b01 (relocated to dff54b01)
[1.981779 0.004609] Loading Environment from EXT4...
[2.158076 0.176296] ** Unable to use mmc 0:auto for loading the env **
[2.162725 0.004650] [2851049] initcall: c0113c75 (relocated to dff54c75)
[2.167360 0.004634] [2854680] initcall: c011bea1 (relocated to dff5cea1)
[2.172639 0.005280] [2859369] initcall: c0113af7 (relocated to dff54af7)
[2.176768 0.004129] [2864056] initcall: c0118931 (relocated to dff59931)
[2.181032 0.004264] In:    serial
[2.182615 0.001583] Out:   serial
[2.183692 0.001077] Err:   serial
[2.184972 0.001280] [2872691] initcall: c0102085 (relocated to dff43085)
[2.189758 0.004786] invalid MAC address in OTP 00:00:00:00:00:00
[2.193756 0.003998] [2881422] initcall: c01018a1 (relocated to dff428a1)
[2.198763 0.005007] [2886015] initcall: c0113aed (relocated to dff54aed)
[2.202945 0.004183] [2890702] initcall: c0113ad5 (relocated to dff54ad5)
[2.207843 0.004898] [2895390] initcall: c01030e5 (relocated to dff440e5)
[2.212608 0.004765] [2900983] initcall: c0113ac1 (relocated to dff54ac1)
[2.216892 0.004284] Net:
[2.236024 0.019132] Warning: ethernet@5800a000 (eth0) using random MAC address - 3e:0b:78:85:ae:85
[2.242833 0.006808] eth0: ethernet@5800a000
[2.244989 0.002156] [2932672] initcall: c0113ab9 (relocated to dff54ab9)
[2.252560 0.007571] STM32MP>
```

```
STM32MP> dm tree
 Class     Index  Probed  Driver                Name
-----------------------------------------------------------
 root          0  [ + ]   root_driver           root_driver
 sysreset      0  [   ]   syscon_reboot         |-- reboot
 simple_bus    0  [ + ]   generic_simple_bus    |-- soc
 serial        0  [ + ]   serial_stm32          |   |-- serial@40010000
 hwspinlock    0  [ + ]   hwspinlock_stm32mp1   |   |-- hwspinlock@4c000000
 nop           0  [ + ]   stm32-rcc             |   |-- rcc@50000000
 clk           0  [ + ]   stm32mp1_clk          |   |   |-- stm32mp1_clk
 reset         0  [ + ]   stm32_rcc_reset       |   |   `-- stm32_rcc_reset
 pmic          0  [ + ]   stm32mp_pwr_pmic      |   |-- pwr@50001000
 regulator     0  [ + ]   stm32mp_pwr_regulato  |   |   |-- reg11
 regulator     1  [ + ]   stm32mp_pwr_regulato  |   |   |-- reg18
 regulator     2  [ + ]   stm32mp_pwr_regulato  |   |   `-- usb33
 syscon        0  [   ]   syscon                |   |-- interrupt-controller@5000d000
 syscon        1  [ + ]   stmp32mp_syscon       |   |-- syscon@50020000
 mmc           0  [ + ]   stm32_sdmmc2          |   |-- sdmmc@58005000
 blk           0  [ + ]   mmc_blk               |   |   `-- sdmmc@58005000.blk
 eth           0  [ + ]   eth_eqos              |   |-- ethernet@5800a000
 misc          0  [ + ]   stm32mp_bsec          |   |-- nvmem@5c005000
 pinctrl       0  [ + ]   pinctrl_stm32         |   |-- pin-controller@50002000
 gpio          0  [ + ]   gpio_stm32            |   |   |-- gpio@50002000
 gpio          1  [ + ]   gpio_stm32            |   |   |-- gpio@50003000
 gpio          2  [ + ]   gpio_stm32            |   |   |-- gpio@50004000
 gpio          3  [ + ]   gpio_stm32            |   |   |-- gpio@50005000
 gpio          4  [ + ]   gpio_stm32            |   |   |-- gpio@50006000
 gpio          5  [   ]   gpio_stm32            |   |   |-- gpio@50007000
 gpio          6  [ + ]   gpio_stm32            |   |   |-- gpio@50008000
 gpio          7  [   ]   gpio_stm32            |   |   |-- gpio@50009000
 gpio          8  [   ]   gpio_stm32            |   |   |-- gpio@5000a000
 gpio          9  [   ]   gpio_stm32            |   |   |-- gpio@5000b000
 gpio         10  [   ]   gpio_stm32            |   |   |-- gpio@5000c000
 pinconfig     0  [   ]   pinconfig             |   |   |-- adc12-ain-0
 pinconfig     1  [   ]   pinconfig             |   |   |-- adc12-usb-cc-pins-0
 pinconfig     2  [   ]   pinconfig             |   |   |-- cec-0
 pinconfig     3  [   ]   pinconfig             |   |   |-- cec-sleep-0
 pinconfig     4  [   ]   pinconfig             |   |   |-- cec-1
 pinconfig     5  [   ]   pinconfig             |   |   |-- cec-sleep-1
 pinconfig     6  [   ]   pinconfig             |   |   |-- dac-ch1
 pinconfig     7  [   ]   pinconfig             |   |   |-- dac-ch2
 pinconfig     8  [   ]   pinconfig             |   |   |-- dcmi-0
 pinconfig     9  [   ]   pinconfig             |   |   |-- dcmi-sleep-0
 pinconfig    10  [   ]   pinconfig             |   |   |-- rgmii-0
 pinconfig    11  [   ]   pinconfig             |   |   |-- rgmii-sleep-0
 pinconfig    12  [   ]   pinconfig             |   |   |-- rgmii-1
 pinconfig    13  [   ]   pinconfig             |   |   |-- rgmii-sleep-1
 pinconfig    14  [   ]   pinconfig             |   |   |-- fmc-0
 pinconfig    15  [   ]   pinconfig             |   |   |-- fmc-sleep-0
 pinconfig    16  [   ]   pinconfig             |   |   |-- i2c1-0
 pinconfig    17  [   ]   pinconfig             |   |   |-- i2c1-1
 pinconfig    18  [   ]   pinconfig             |   |   |-- i2c1-2
 pinconfig    19  [   ]   pinconfig             |   |   |-- i2c1-3
 pinconfig    20  [   ]   pinconfig             |   |   |-- i2c2-0
 pinconfig    21  [   ]   pinconfig             |   |   |-- i2c2-1
 pinconfig    22  [   ]   pinconfig             |   |   |-- i2c2-2
 pinconfig    23  [   ]   pinconfig             |   |   |-- i2c2-3
 pinconfig    24  [   ]   pinconfig             |   |   |-- i2c5-0
 pinconfig    25  [   ]   pinconfig             |   |   |-- i2c5-1
 pinconfig    26  [   ]   pinconfig             |   |   |-- i2s2-0
 pinconfig    27  [   ]   pinconfig             |   |   |-- i2s2-1
 pinconfig    28  [   ]   pinconfig             |   |   |-- ltdc-a-0
 pinconfig    29  [   ]   pinconfig             |   |   |-- ltdc-a-1
 pinconfig    30  [   ]   pinconfig             |   |   |-- ltdc-b-0
 pinconfig    31  [   ]   pinconfig             |   |   |-- ltdc-b-1
 pinconfig    32  [   ]   pinconfig             |   |   |-- m-can1-0
 pinconfig    33  [   ]   pinconfig             |   |   |-- m_can1-sleep-0
 pinconfig    34  [   ]   pinconfig             |   |   |-- pwm2-0
 pinconfig    35  [   ]   pinconfig             |   |   |-- pwm8-0
 pinconfig    36  [   ]   pinconfig             |   |   |-- pwm12-0
 pinconfig    37  [   ]   pinconfig             |   |   |-- qspi-clk-0
 pinconfig    38  [   ]   pinconfig             |   |   |-- qspi-clk-sleep-0
 pinconfig    39  [   ]   pinconfig             |   |   |-- qspi-bk1-0
 pinconfig    40  [   ]   pinconfig             |   |   |-- qspi-bk1-sleep-0
 pinconfig    41  [   ]   pinconfig             |   |   |-- qspi-bk2-0
 pinconfig    42  [   ]   pinconfig             |   |   |-- qspi-bk2-sleep-0
 pinconfig    43  [   ]   pinconfig             |   |   |-- sai2a-0
 pinconfig    44  [   ]   pinconfig             |   |   |-- sai2a-1
 pinconfig    45  [   ]   pinconfig             |   |   |-- sai2b-0
 pinconfig    46  [   ]   pinconfig             |   |   |-- sai2b-1
 pinconfig    47  [   ]   pinconfig             |   |   |-- sai2b-2
 pinconfig    48  [   ]   pinconfig             |   |   |-- sai2b-3
 pinconfig    49  [   ]   pinconfig             |   |   |-- sai4a-0
 pinconfig    50  [   ]   pinconfig             |   |   |-- sai4a-1
 pinconfig    51  [   ]   pinconfig             |   |   |-- sdmmc1-b4-0
 pinconfig    52  [   ]   pinconfig             |   |   |-- sdmmc1-b4-od-0
 pinconfig    53  [   ]   pinconfig             |   |   |-- sdmmc1-b4-sleep-0
 pinconfig    54  [   ]   pinconfig             |   |   |-- sdmmc1-dir-0
 pinconfig    55  [   ]   pinconfig             |   |   |-- sdmmc1-dir-sleep-0
 pinconfig    56  [   ]   pinconfig             |   |   |-- sdmmc1-dir-1
 pinconfig    57  [   ]   pinconfig             |   |   |-- sdmmc1-dir-sleep-1
 pinconfig    58  [   ]   pinconfig             |   |   |-- sdmmc2-b4-0
 pinconfig    59  [   ]   pinconfig             |   |   |-- sdmmc2-b4-od-0
 pinconfig    60  [   ]   pinconfig             |   |   |-- sdmmc2-b4-sleep-0
 pinconfig    61  [   ]   pinconfig             |   |   |-- sdmmc2-d47-0
 pinconfig    62  [   ]   pinconfig             |   |   |-- sdmmc2-d47-sleep-0
 pinconfig    63  [   ]   pinconfig             |   |   |-- sdmmc2-d47-1
 pinconfig    64  [   ]   pinconfig             |   |   |-- sdmmc2-d47-sleep-1
 pinconfig    65  [   ]   pinconfig             |   |   |-- spdifrx-0
 pinconfig    66  [   ]   pinconfig             |   |   |-- spdifrx-1
 pinconfig    67  [   ]   pinconfig             |   |   |-- spi2-0
 pinconfig    68  [   ]   pinconfig             |   |   |-- stusb1600-0
 pinconfig    69  [ + ]   pinconfig             |   |   |-- uart4-0
 pinconfig    70  [   ]   pinconfig             |   |   |-- uart4-1
 pinconfig    71  [   ]   pinconfig             |   |   |-- uart7-0
 pinconfig    72  [ + ]   pinconfig             |   |   |-- sdmmc1_mx-0
 pinconfig    73  [   ]   pinconfig             |   |   |-- sdmmc1_opendrain_mx-0
 pinconfig    74  [   ]   pinconfig             |   |   |-- sdmmc1_sleep_mx-0
 pinconfig    75  [ + ]   pinconfig             |   |   |-- eth1_mx-0
 pinconfig    76  [   ]   pinconfig             |   |   `-- eth1_sleep_mx-0
 pinctrl       1  [ + ]   pinctrl_stm32         |   |-- pin-controller-z@54004000
 gpio         11  [   ]   gpio_stm32            |   |   |-- gpio@54004000
 pinconfig    77  [   ]   pinconfig             |   |   |-- i2c2-0
 pinconfig    78  [   ]   pinconfig             |   |   |-- i2c2-1
 pinconfig    79  [   ]   pinconfig             |   |   |-- i2c4-0
 pinconfig    80  [   ]   pinconfig             |   |   |-- i2c4-1
 pinconfig    81  [   ]   pinconfig             |   |   `-- spi1-0
 ram           0  [   ]   stm32mp1_ddr          |   `-- ddr@5a003000
 simple_bus    1  [   ]   generic_simple_bus    |-- mlahb
 regulator     3  [ + ]   fixed regulator       |-- regulator_vdd
 clk           1  [ + ]   fixed_rate_clock      |-- clk-hse
 clk           2  [ + ]   fixed_rate_clock      |-- clk-hsi
 clk           3  [ + ]   fixed_rate_clock      |-- clk-lse
 clk           4  [ + ]   fixed_rate_clock      |-- clk-lsi
 clk           5  [ + ]   fixed_rate_clock      `-- clk-csi
```

```
...
[1.230671 0.002738] [851409] initcall: c0113895
[1.232891 0.002220] [1387295] initcall: c01138cf
...
[1.243749 0.002375] [1387597] initcall: c0141fad
[1.246018 0.002269] [1896426] initcall: c01188a1
...
```

```
...
.text.initf_dm
                0xc0113894        0x6 common/built-in.o
...
.text.serial_init
                0xc0141fac       0xe0 drivers/serial/built-in.o
                0xc0141fac                serial_init
...
```

initcall 주소가 왜 홀수일까?  
0.5s 정도 지연되는 함수가 2개가 있는데 좀더 확인해봐야 겠다.
---
layout: post
title: "STM32F3Discovery Zephyr MCUBOOT 사용 기록"
date: 2020-09-05 16:42:00 +0900
categories: embedded
tags: [STM32, Zephyr]
---

Zephyr 버전
- 2.3.99 7b8643e373c2ee7e4eb8dad73b1b59a415f72f95

관련 정보
- https://docs.zephyrproject.org/latest/guides/west/sign.html
- https://mynewt.apache.org/latest/newtmgr/index.html

어플리케이션 폴더 생성
```
cd ~/zephyrproject
mkdir testapp
cd testapp
mkdir src
mkdir boards
```

파일 생성

CMakeLists.txt
```
cmake_minimum_required(VERSION 3.13.3)

set(BOARD stm32f3_disco)

find_package(Zephyr)
project(testapp)

target_sources(app PRIVATE src/main.c)
```

prj.conf
```
CONFIG_MCUMGR=y
CONFIG_SYSTEM_WORKQUEUE_STACK_SIZE=2048
CONFIG_BOOTLOADER_MCUBOOT=y
CONFIG_MCUBOOT_SIGNATURE_KEY_FILE="bootloader/mcuboot/root-rsa-2048.pem"
CONFIG_MCUBOOT_EXTRA_IMGTOOL_ARGS="--version 1.0"
CONFIG_FLASH=y
CONFIG_MCUMGR_CMD_IMG_MGMT=y
CONFIG_MCUMGR_CMD_OS_MGMT=y
CONFIG_LOG=y
CONFIG_MCUMGR_SMP_SHELL=y
```

src/main.c
```
#include <zephyr.h>

#include "img_mgmt/img_mgmt.h"
#include "os_mgmt/os_mgmt.h"

void main(void) {
    os_mgmt_register_group();
    img_mgmt_register_group();
}
```

boards/stm32f3_disco.overlay
```
/ {
    chosen {
        zephyr,code-partition = &slot0_partition;
    };
};

&flash0 {
    partitions {
        compatible = "fixed-partitions";
        #address-cells = <1>;
        #size-cells = <1>;

        boot_partition: partition@0{
            label = "mcuboot";
            reg = <0x00000000 0x0000c000>;
            read-only;
        };

        slot0_partition: partition@c000 {
            label = "image-0";
            reg = <0x0000c000 0x00016000>;
        };

        slot1_partition: partition@22000 {
            label = "image-1";
            reg = <0x00022000 0x00016000>;
        };

        scratch_partition: partition@38000 {
            label = "image-scratch";
            reg = <0x00038000 0x00008000>;
        };
    };
};
```

mcuboot 빌드 및 플래시
```
west build -b stm32f3_disco -s ../bootloader/mcuboot/boot/zephyr -d build-mcuboot -- -DDTC_OVERLAY_FILE="~/zephyrproject/testapp/boards/stm32f3_disco.overlay"
west flash -d build-mcuboot
```

UART Console 표시
```
*** Booting Zephyr OS build zephyr-v2.3.0-2610-g7b8643e373c2  ***
[00:00:00.005,000] <inf> mcuboot: Starting bootloader
[00:00:00.006,000] <inf> mcuboot: Primary image: magic=unset, swap_type=0x1, copy_done=0x3, image_ok=0x3
[00:00:00.006,000] <inf> mcuboot: Scratch: magic=unset, swap_type=0x1, copy_done=0x3, image_ok=0x3
[00:00:00.006,000] <inf> mcuboot: Boot source: primary slot
[00:00:00.014,000] <inf> mcuboot: Swap type: none
[00:00:00.014,000] <err> mcuboot: Unable to find bootable image
```

어플리케이션 빌드 및 플래시
```
west build
west flash --hex-file build/zephyr/zephyr.signed.hex
```

UART Console 표시
```
*** Booting Zephyr OS build zephyr-v2.3.0-2610-g7b8643e373c2  ***
[00:00:00.005,000] <inf> mcuboot: Starting bootloader
[00:00:00.006,000] <inf> mcuboot: Primary image: magic=unset, swap_type=0x1, copy_done=0x3, image_ok=0x3
[00:00:00.006,000] <inf> mcuboot: Scratch: magic=unset, swap_type=0x1, copy_done=0x3, image_ok=0x3
[00:00:00.006,000] <inf> mcuboot: Boot source: primary slot
[00:00:00.014,000] <inf> mcuboot: Swap type: none
*** Booting Zephyr OS build zephyr-v2.3.0-2610-g7b8643e373c2  ***


uart:~$
```

이미지 정보 확인
```
mcumgr --conntype serial --connstring "/dev/ttyUSB0" image list
```

결과
```
Images:
 image=0 slot=0
    version: 1.0.0
    bootable: true
    flags: active confirmed
    hash: d4b34caf9c7c488df0b986d74b0e3642376668bb2b604926a47de845444653bb
Split status: N/A (0)
```

prj.conf 수정
```
CONFIG_MCUMGR=y
CONFIG_SYSTEM_WORKQUEUE_STACK_SIZE=2048
CONFIG_BOOTLOADER_MCUBOOT=y
CONFIG_MCUBOOT_SIGNATURE_KEY_FILE="bootloader/mcuboot/root-rsa-2048.pem"
CONFIG_MCUBOOT_EXTRA_IMGTOOL_ARGS="--version 2.0"
CONFIG_FLASH=y
CONFIG_MCUMGR_CMD_IMG_MGMT=y
CONFIG_MCUMGR_CMD_OS_MGMT=y
CONFIG_LOG=y
CONFIG_MCUMGR_SMP_SHELL=y
```

main.c 수정
```
#include <zephyr.h>
#include <sys/printk.h>

#include "img_mgmt/img_mgmt.h"
#include "os_mgmt/os_mgmt.h"

void main(void) {
    os_mgmt_register_group();
    img_mgmt_register_group();

    printk("Hello World!\n");
}
```

빌드 및 이미지 업로드, 확인
```
west build
mcumgr --conntype serial --connstring "/dev/ttyUSB0" image upload build/zephyr/zephyr.signed.bin
mcumgr --conntype serial --connstring "/dev/ttyUSB0" image list
```

결과
```
Images:
 image=0 slot=0
    version: 1.0.0
    bootable: true
    flags: active confirmed
    hash: d4b34caf9c7c488df0b986d74b0e3642376668bb2b604926a47de845444653bb
 image=0 slot=1
    version: 2.0.0
    bootable: true
    flags:
    hash: ac8f066a2984fac811a65b3de88986e3812ace609626e52c55543cd6ca4a7904
Split status: N/A (0)
```

업로드한 이미지 임시 적용
```
mcumgr --conntype serial --connstring "/dev/ttyUSB0" image test ac8f066a2984fac811a65b3de88986e3812ace609626e52c55543cd6ca4a7904
```

결과
```
Images:
 image=0 slot=0
    version: 1.0.0
    bootable: true
    flags: active confirmed
    hash: d4b34caf9c7c488df0b986d74b0e3642376668bb2b604926a47de845444653bb
 image=0 slot=1
    version: 2.0.0
    bootable: true
    flags: pending
    hash: ac8f066a2984fac811a65b3de88986e3812ace609626e52c55543cd6ca4a7904
Split status: N/A (0)
```

보드 리셋
```
mcumgr --conntype serial --connstring "/dev/ttyUSB0" reset
```

UART Console 표시
```
*** Booting Zephyr OS build zephyr-v2.3.0-2610-g7b8643e373c2  ***
[00:00:00.005,000] <inf> mcuboot: Starting bootloader
[00:00:00.006,000] <inf> mcuboot: Primary image: magic=unset, swap_type=0x1, copy_done=0x3, image_ok=0x3
[00:00:00.006,000] <inf> mcuboot: Scratch: magic=unset, swap_type=0x1, copy_done=0x3, image_ok=0x3
[00:00:00.006,000] <inf> mcuboot: Boot source: primary slot
[00:00:00.014,000] <inf> mcuboot: Swap type: test
*** Booting Zephyr OS build zephyr-v2.3.0-2610-g7b8643e373c2  ***
Hello World!


uart:~$
```

이미지 상태 확인
```
mcumgr --conntype serial --connstring "/dev/ttyUSB0" image list
```

결과
```
Images:
 image=0 slot=0
    version: 2.0.0
    bootable: true
    flags: active
    hash: ac8f066a2984fac811a65b3de88986e3812ace609626e52c55543cd6ca4a7904
 image=0 slot=1
    version: 1.0.0
    bootable: true
    flags: confirmed
    hash: d4b34caf9c7c488df0b986d74b0e3642376668bb2b604926a47de845444653bb
Split status: N/A (0)
```

이미지에 문제가 없으면 적용
```
mcumgr --conntype serial --connstring "/dev/ttyUSB0" image confirm
```

결과
```
Images:
 image=0 slot=0
    version: 2.0.0
    bootable: true
    flags: active confirmed
    hash: ac8f066a2984fac811a65b3de88986e3812ace609626e52c55543cd6ca4a7904
 image=0 slot=1
    version: 1.0.0
    bootable: true
    flags:
    hash: d4b34caf9c7c488df0b986d74b0e3642376668bb2b604926a47de845444653bb
Split status: N/A (0)
```
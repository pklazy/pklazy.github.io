---
layout: post
title: "MicroPython M5Stick OLED Driver"
date: 2019-05-04 17:12 +0900
categories: embedded
tags: m5stick
---

![MtStick](/assets/e7c3c369477d60e83090d33037d656fcd820c3e796e26a43bb1604de3f26a425.jpg)

MicroPython M5Stick OLED Driver: https://gist.github.com/pklazy/db683d8a079ed10bd88b63fd880fed7d

OLED 사양

- 1.3 inch
- 64x128
- SH1107

OLED IO 연결 상태

- CS: GPIO14
- DC: GPIO27
- RST(RES): GPIO33
- SCK(D0): GPIO18
- MOSI(D1): GPIO23

참고

- 제품 정보: https://docs.m5stack.com/#/en/core/m5stick
- 회로도: https://github.com/m5stack/M5-Schematic/tree/master/Core/m5stick
- SH1107 datasheet: https://www.displayfuture.com/Display/datasheet/controller/SH1107.pdf
- 제어 참고 코드: https://github.com/olikraus/u8g2/blob/master/csrc/u8x8_d_sh1107.c


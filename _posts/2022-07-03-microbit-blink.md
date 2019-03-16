---
layout: post
title: micro:bit LED blink
category: embedded
tags: microbit rust
date: 2022-07-03 15:23:00 +0900
---

패키지 생성

```shell
$ cargo new microbit-led-blink
$ cd microbit-led-blink
```

빌드시 필요한 메모리 정보인 memory.x 파일 추가

```
/* Linker script for the nRF52833 */
MEMORY
{
  FLASH : ORIGIN = 0x00000000, LENGTH = 512K
  RAM   : ORIGIN = 0x20000000, LENGTH = 128K
}
```

openocd gdb 서버 실행용 openocd.cfg 파일 생성

```ini
source [find interface/cmsis-dap.cfg]
source [find target/nrf52.cfg]
```

gdb 연결용 openocd.gdb 파일 생성

```ini
target extended-remote :3333

# print demangled symbols
set print asm-demangle on

# detect unhandled exceptions, hard faults and panics
break DefaultHandler
break HardFault
break rust_begin_unwind

monitor arm semihosting enable

load

# start the process but immediately halt the processor
stepi
```

빌드 설정을 위해 .cargo/config 파일 생성

```toml
[target.thumbv7em-none-eabihf]
rustflags = ["-C", "link-arg=-Tlink.x"]
runner = "gdb-multiarch -x openocd.gdb"

[build]
target = "thumbv7em-none-eabihf"
```

Cargo.toml에 의존 패키지 추가

```toml
[dependencies]
cortex-m = "0.7"
cortex-m-rt = "0.7"
panic-halt = "0.2"
nrf52833-pac = { git = "https://github.com/pklazy/nrf52833-pac" }
```

MCU는 기본값으로 초기화가 되어 동작할것으로 예상하여 GPIO만 제어해보기로 한다.

main.rs

```rust
#![no_std]
#![no_main]

use cortex_m_rt::entry;
use nrf52833_pac as pac;
use panic_halt as _;

#[entry]
fn main() -> ! {
    let p = pac::Peripherals::take().unwrap();

    let p0 = p.P0;

    // ROW1(source): P0.21, COL1(sink): P0.28
    p0.dirset.write(|w| w.pin21().set_bit().pin28().set_bit());

    loop {
        p0.outset.write(|w| w.pin21().set_bit());
        for _ in 0..100 {
            cortex_m::asm::nop();
        }
        p0.outclr.write(|w| w.pin21().clear_bit_by_one());
        for _ in 0..200000 {
            cortex_m::asm::nop();
        }
    }
}
```

참고
- <https://docs.rust-embedded.org/book/>
- <https://tech.microbit.org/hardware/schematic/>
- <https://infocenter.nordicsemi.com/topic/ps_nrf52833/gpio.html>
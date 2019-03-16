---
layout: post
title: micro:bit UART 테스트
category: embedded
tags: microbit rust
date: 2022-07-11 21:55:00 +0900
---

메모리를 보면 UART 장치에 UART, UARTE 2종류 사용할수 있는데 UARTE에는 EasyDMA가 붙어있다.  
nRF52는 다른 회사 제품과 다르게 주변 장치에 DMA가 각각 붙어있다.  
UART는 Deprecated로 표시되어있어 UARTE를 사용해보기로 했다.

주의사항

- micro:bit v2 회로도에 UART 핀이 반대로 되어있음
- easyDMA는 flash에 액세스 불가능
- 수신중 STOPRX를 할 경우 데이터가 깨짐
- RXD.AMOUNT는 ENDRX 이후에 업데이트됨

처음 생각한건 DMA 계속돌려 링버퍼 처럼 쓰는 구조 였지만 RXD.AMOUNT가 실시간으로 업데이트되지 않고 STOPRX를 할경우 데이터가 깨지는 문제로 간단하게 구현할 수 없었다.

위 방식을 사용하기 위해서는 PPI와 TIMER를 사용해서 RXDRDY가 발생한 수를 카운트 하는 방식으로 구현 하는것으로 보인다.

우선 UART 기능이 동작하는것부터 확인하기 위해 1byte로 echo하는 간단한 코드를 작성했다.

```rust
#![no_std]
#![no_main]

use cortex_m_rt::entry;
use nrf52833_pac as pac;
use panic_halt as _;

fn clock_config(p: &pac::Peripherals) {
    let clock = &p.CLOCK;

    if clock.hfclkstat.read().src().is_xtal()
        && clock.hfclkstat.read().state().is_running()
    {
        return;
    }

    clock
        .tasks_hfclkstart
        .write(|w| w.tasks_hfclkstart().set_bit());

    while !clock.hfclkstat.read().src().is_xtal() {}
    while !clock.hfclkstat.read().state().is_running() {}
}

fn uart_config(dp: &pac::Peripherals) {
    let p0 = &dp.P0;
    let uarte = &dp.UARTE0;

    // micro:bit uart rxd: P1.8 txd: P0.6
    p0.outset.write(|w| w.pin6().set());
    p0.dirset.write(|w| w.pin6().set());

    uarte.psel.rxd.write(|w| {
        unsafe { w.port().set_bit().pin().bits(8) }
            .connect()
            .connected()
    });
    uarte.psel.txd.write(|w| {
        unsafe { w.port().clear_bit().pin().bits(6) }
            .connect()
            .connected()
    });

    uarte.baudrate.write(|w| w.baudrate().baud115200());
    uarte.enable.write(|w| w.enable().enabled());
}

fn uart_write(dp: &pac::Peripherals, data: &[u8]) {
    let uarte = &dp.UARTE0;

    uarte
        .txd
        .ptr
        .write(|w| unsafe { w.ptr().bits(data.as_ptr() as u32) });
    uarte
        .txd
        .maxcnt
        .write(|w| unsafe { w.maxcnt().bits(data.len() as u16) });

    uarte.events_endtx.reset();

    uarte.tasks_starttx.write(|w| w.tasks_starttx().trigger());

    while uarte.events_endtx.read().events_endtx().is_not_generated() {}
}

fn uart_read(dp: &pac::Peripherals, data: &mut [u8]) -> usize {
    let uarte = &dp.UARTE0;

    uarte
        .rxd
        .ptr
        .write(|w| unsafe { w.ptr().bits(data.as_ptr() as u32) });
    uarte
        .rxd
        .maxcnt
        .write(|w| unsafe { w.maxcnt().bits(data.len() as u16) });

    uarte.events_endrx.reset();

    uarte.tasks_startrx.write(|w| w.tasks_startrx().trigger());

    while uarte.events_endrx.read().events_endrx().is_not_generated() {}

    uarte.rxd.amount.read().amount().bits() as usize
}

#[entry]
fn main() -> ! {
    let dp = pac::Peripherals::take().unwrap();

    clock_config(&dp);
    uart_config(&dp);

    let mut buf = [0_u8; 1];

    loop {
        let len = uart_read(&dp, &mut buf);
        uart_write(&dp, &buf[..len]);
    }
}
```

참고

- <https://infocenter.nordicsemi.com/topic/ps_nrf52833/memory.html>
- <https://infocenter.nordicsemi.com/topic/ps_nrf52833/uarte.html>
- <https://infocenter.nordicsemi.com/topic/ps_nrf52833/easydma.html>
- <https://devzone.nordicsemi.com/f/nordic-q-a/43684/why-is-uart-without-dma-deprecated>
- <https://github.com/microbit-foundation/microbit-v2-hardware/issues/5>
- <https://devzone.nordicsemi.com/f/nordic-q-a/36866/stoprx-task-of-uarte-during-reception-of-bytes>
- <https://devzone.nordicsemi.com/f/nordic-q-a/28420/uarte-in-circular-mode>
- <https://www.reddit.com/r/embedded/comments/l9jo56/using_dma/>
- <https://github.com/aws/amazon-freertos/blob/main/vendors/nordic/nRF5_SDK_15.2.0/components/libraries/experimental_libuarte/nrf_libuarte.c>
- <https://github.com/zephyrproject-rtos/zephyr/blob/main/drivers/serial/uart_nrfx_uarte.c>
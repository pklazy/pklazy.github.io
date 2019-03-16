---
layout: post
title: micro:bit 마이크 동작 확인
category: embedded
tags: microbit rust
date: 2022-08-18 12:30:00 +0900
---

micro:bit에 달린 마이크는 아날로그 타입으로 ADC를 통해서 소리를 측정을 하는데 채널 설정을 기본값으로 측정 해보니 측정된 소리가 들리기는 하나 노이즈가 심했다.

micro:bit 기존 코드인 codal 코드를 보니 게인 4배, 레퍼런스 Vdd 1/4, 측정시간 3us로 기본값과 다르게 되어있어 해당 설정을 적용해보니 기본값에 비해 훨신 소리 측정이 잘 되었지만 노이즈는 여전히 어느정도 있다.

ADC를 연속적으로 측정하기 위해서는 2가지 방법 중 한가지인 ADC 자체 타이머를 사용하는 방법이 있는데 자체 타이머가 Sample 동작 시키기 위해서는 Start 테스크 이전 이나 이후에 한번은 Sample 테스크를 한번 실행시켜야 했다.

측정된 데이터를 시리얼로 전송하는데 1Mbps 설정에서도 동작은 하는데 인터페이스칩 내부 버퍼 문제인지 로드가 많이지면 데이터 손실이 발생하여 샘플링레이트를 48kHz에서 32kHz로 변경했다.

```rust
#![no_std]
#![no_main]

use core::sync::atomic::{AtomicU32, Ordering};
use cortex_m_rt::entry;
use cortex_m_rt::exception;
use nrf52833_pac as pac;
use panic_semihosting as _;

static TICK_COUNTER: AtomicU32 = AtomicU32::new(0);

fn clock_config(clock: &pac::CLOCK) {
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

fn systick_config(p: &mut cortex_m::Peripherals) {
    use cortex_m::peripheral::syst::SystClkSource;

    let syst = &mut p.SYST;

    syst.set_clock_source(SystClkSource::Core);
    syst.set_reload((64_000_000 / 1000) - 1); // 1kHz
    syst.clear_current();
    syst.enable_counter();
    syst.enable_interrupt();
}

#[exception]
fn SysTick() {
    TICK_COUNTER.fetch_add(1, Ordering::Relaxed);
}

fn delay_ms(ms: u32) {
    let curr_tick = TICK_COUNTER.load(Ordering::Relaxed);
    while TICK_COUNTER.load(Ordering::Relaxed).wrapping_sub(curr_tick) < ms {}
}

fn io_config(p0: &pac::P0) {
    // uart tx
    p0.outset.write(|w| w.pin6().set());
    p0.dirset.write(|w| w.pin6().set());

    // run_mic
    p0.dirset.write(|w| w.pin20().set());
}

fn serial_config(uarte: &pac::UARTE0) {
    uarte.psel.txd.write(|w| {
        unsafe { w.port().clear_bit().pin().bits(6) }
            .connect()
            .connected()
    });

    uarte.baudrate.write(|w| w.baudrate().baud1m());
    uarte.enable.write(|w| w.enable().enabled());
}

fn serial_send_i16(uarte: &pac::UARTE0, buf: &[i16]) {
    uarte
        .txd
        .ptr
        .write(|w| unsafe { w.ptr().bits(buf.as_ptr() as u32) });
    uarte
        .txd
        .maxcnt
        .write(|w| unsafe { w.maxcnt().bits((buf.len() * 2) as u16) });

    uarte.events_endtx.reset();
    uarte.tasks_starttx.write(|w| w.tasks_starttx().trigger());
    while uarte.events_endtx.read().events_endtx().is_not_generated() {}
}

fn mic_config(adc: &pac::SAADC) {
    adc.enable.write(|w| w.enable().enabled());

    adc.tasks_calibrateoffset
        .write(|w| w.tasks_calibrateoffset().trigger());
    while adc
        .events_calibratedone
        .read()
        .events_calibratedone()
        .is_not_generated()
    {}

    adc.resolution.write(|w| w.val()._14bit());
    // 32kHz
    adc.samplerate.write(|w| unsafe {
        w.cc().bits((16_000_000 / 32_000) as u16).mode().timers()
    });
    adc.ch[0].pselp.write(|w| w.pselp().analog_input3());
    adc.ch[0]
        .config
        .write(|w| w.gain().gain4().refsel().vdd1_4().tacq()._3us());

    adc.tasks_sample.write(|w| w.tasks_sample().trigger());
}

fn mic_record_start(adc: &pac::SAADC, buf: &mut [i16]) {
    adc.result
        .ptr
        .write(|w| unsafe { w.ptr().bits(buf.as_ptr() as u32) });
    adc.result
        .maxcnt
        .write(|w| unsafe { w.maxcnt().bits(buf.len() as u16) });

    adc.events_end.reset();
    adc.events_started.reset();
    adc.tasks_start.write(|w| w.tasks_start().trigger());
    while adc
        .events_started
        .read()
        .events_started()
        .is_not_generated()
    {}
}

fn mic_record_wait(adc: &pac::SAADC) {
    while adc.events_end.read().events_end().is_not_generated() {}
}

fn mic_on(p0: &pac::P0, on: bool) {
    if on {
        p0.outset.write(|w| w.pin20().set());
    } else {
        p0.outclr.write(|w| w.pin20().clear_bit_by_one());
    }
}

#[entry]
fn main() -> ! {
    let mut cp = cortex_m::Peripherals::take().unwrap();
    let dp = pac::Peripherals::take().unwrap();

    clock_config(&dp.CLOCK);
    systick_config(&mut cp);
    io_config(&dp.P0);
    serial_config(&dp.UARTE0);

    mic_config(&dp.SAADC);

    delay_ms(1000);

    mic_on(&dp.P0, true);
    delay_ms(10);

    // 10ms
    let mut mic_buf = [[0; 320]; 2];
    let mut mic_buf_index = 0;

    mic_record_start(&dp.SAADC, &mut mic_buf[mic_buf_index]);
    mic_buf_index = (mic_buf_index + 1) % 2;

    // 10s
    for _ in 0..(1000 - 1) {
        mic_record_wait(&dp.SAADC);

        mic_record_start(&dp.SAADC, &mut mic_buf[mic_buf_index]);
        mic_buf_index = (mic_buf_index + 1) % 2;

        serial_send_i16(&dp.UARTE0, &mic_buf[mic_buf_index]);
    }

    mic_record_wait(&dp.SAADC);
    mic_on(&dp.P0, false);
    mic_buf_index = (mic_buf_index + 1) % 2;

    serial_send_i16(&dp.UARTE0, &mic_buf[mic_buf_index]);

    loop {}
}
```

참고
- <https://infocenter.nordicsemi.com/topic/ps_nrf52833/saadc.html>
- <https://tech.microbit.org/hardware/#microphone>
- <https://tech.microbit.org/hardware/schematic/>
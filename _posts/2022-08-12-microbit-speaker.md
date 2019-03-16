---
layout: post
title: micro:bit 스피커 동작 확인
category: embedded
tags: microbit rust
date: 2022-08-12 23:50:00 +0900
---

micro:bit 스피커 동작 확인을 위해 PWM을 테스트 해봤다.

삽질
- PSEL.OUT 레지스터 기본값이 0xFFFFFFFF 라서 PORT를 설정하지 않으면 핀설정이 P1에 할당된다.
- LOOP 레지스터 설정은 SEQ[0], SEQ[1] 모두 설정되어 있어야 적용된다.

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

fn systick_config(syst: &mut cortex_m::peripheral::SYST) {
    use cortex_m::peripheral::syst::SystClkSource;

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
    // speaker
    p0.dirset.write(|w| w.pin0().set());
}

fn speaker_config(pwm: &pac::PWM0) {
    pwm.psel.out[0].write(|w| unsafe {
        w.port().clear_bit().pin().bits(0).connect().connected()
    });
    pwm.enable.write(|w| w.enable().enabled());
    pwm.mode.write(|w| w.updown().up());
    pwm.loop_.write(|w| w.cnt().disabled());
    pwm.decoder
        .write(|w| w.load().individual().mode().refresh_count());
}

fn speaker_out(pwm: &pac::PWM0, freq: f32) {
    const PWM_CLK: u32 = 16_000_000;

    if freq <= 0.0 {
        return;
    }

    let mut prescaler: u8 = 0;
    let mut pwm_clk;
    loop {
        pwm_clk = PWM_CLK / (1 << prescaler);

        if (pwm_clk as f32 / freq) as u32 <= 0x7fff {
            break;
        }
        prescaler += 1;

        if prescaler > 7 {
            return;
        }
    }

    let count_top = (pwm_clk as f32 / freq) as u16;

    pwm.prescaler.write(|w| w.prescaler().bits(prescaler));
    pwm.countertop
        .write(|w| unsafe { w.countertop().bits(count_top) });

    let seq_ram: [u16; 4] = [count_top / 2, 0, 0, 0];
    pwm.seq0
        .ptr
        .write(|w| unsafe { w.ptr().bits(seq_ram.as_ptr() as u32) });
    pwm.seq0
        .cnt
        .write(|w| unsafe { w.cnt().bits(seq_ram.len() as u16) });
    pwm.seq0.refresh.write(|w| w.cnt().continuous());
    pwm.seq0.enddelay.reset();

    pwm.events_seqend[0].reset();
    pwm.tasks_seqstart[0].write(|w| w.tasks_seqstart().trigger());
    while pwm.events_seqend[0]
        .read()
        .events_seqend()
        .is_not_generated()
    {}
}

#[entry]
fn main() -> ! {
    let mut cp = cortex_m::Peripherals::take().unwrap();
    let dp = pac::Peripherals::take().unwrap();

    clock_config(&dp.CLOCK);
    systick_config(&mut cp.SYST);
    io_config(&dp.P0);

    speaker_config(&dp.PWM0);

    for i in 1..=10 {
        speaker_out(&dp.PWM0, i as f32 * 100.0);
        delay_ms(500);
    }

    dp.PWM0.tasks_stop.write(|w| w.tasks_stop().trigger());

    loop {}
}
```

참고
- <https://infocenter.nordicsemi.com/topic/ps_nrf52833/pwm.html>
---
layout: post
title: micro:bit 로 BLE ADV 데이터 받아보기
category: embedded
tags: microbit rust BLE
date: 2022-08-11 17:10:00 +0900
---

micro:bit를 통해 BLE 신호를 받을수 있는지 확인하기 위해 Advertising physical channel PDU를 받아보기로 했다.

주의사항

- Access address(BASE, PREFIX, RXADDRESSES)를 설정하지 않으면 RX 상태에서 멈춘다.
- LENGTH 필드를 설정(PCNF0.LFLEN)하지 않으면 payload 값이 저장 되지 않는다.

Access address를 설정하지 않으면 수신이 되지 않아 아쉽게도 떠도는 모든 데이터를 수신 할수는 없다.

아래 코드로 Advertising physical channel PDU 데이터를 받아지는것을 확인할수 있었다.

```rust
#![no_std]
#![no_main]

use core::fmt::Write;
use core::sync::atomic::{AtomicU32, Ordering};
use cortex_m_rt::entry;
use cortex_m_rt::exception;
use nrf52833_pac as pac;
use panic_semihosting as _;

static TICK_COUNTER: AtomicU32 = AtomicU32::new(0);

const PDU_TYPE_STR: [&str; 8] = [
    "ADV_IND",
    "ADV_DIRECT_IND",
    "ADV_NONCONN_IND",
    "SCAN_REQ",
    "SCAN_RSP",
    "CONNECT_IND",
    "ADV_SCAN_IND",
    "ADV_EXT_IND",
];

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

fn io_config(p0: &pac::P0) {
    p0.outset.write(|w| w.pin6().set());
    p0.dirset.write(|w| w.pin6().set());
}

fn delay_ms(ms: u32) {
    let curr_tick = TICK_COUNTER.load(Ordering::Relaxed);
    while TICK_COUNTER.load(Ordering::Relaxed).wrapping_sub(curr_tick) < ms {}
}

fn ble_config_adv(dp: &pac::Peripherals) {
    let radio = &dp.RADIO;

    radio.mode.write(|w| w.mode().ble_1mbit());

    // adv header
    radio
        .pcnf0
        .write(|w| unsafe { w.s0len().set_bit().lflen().bits(8) });

    radio
        .pcnf1
        .write(|w| unsafe { w.balen().bits(3) }.whiteen().enabled());

    radio.crccnf.write(|w| w.len().three().skipaddr().skip());
    radio.crcpoly.write(|w| unsafe { w.crcpoly().bits(0x65b) });
    radio
        .crcinit
        .write(|w| unsafe { w.crcinit().bits(0x555555) });

    // 0x8E89BED6
    radio.base0.write(|w| unsafe { w.base0().bits(0x89bed600) });
    radio.prefix0.write(|w| unsafe { w.ap0().bits(0x8e) });
    radio.rxaddresses.write(|w| w.addr0().enabled());

    // frequency: 2402MHz
    radio.frequency.write(|w| unsafe { w.frequency().bits(2) });
    // channel index: 37
    radio
        .datawhiteiv
        .write(|w| unsafe { w.datawhiteiv().bits(37) });

    radio.shorts.write(|w| {
        w.address_rssistart()
            .enabled()
            .disabled_rssistop()
            .enabled()
    });
}

fn ble_recv_adv(dp: &pac::Peripherals, buf: &mut [u8]) {
    let radio = &dp.RADIO;

    if buf.len() < 2 {
        return;
    }

    let pdu_len = buf.len() - 2;
    let pdu_len = if pdu_len > 0xff { 0xff } else { pdu_len as u8 };

    radio
        .packetptr
        .write(|w| unsafe { w.packetptr().bits(buf.as_mut_ptr() as u32) });

    radio
        .pcnf1
        .modify(|_, w| unsafe { w.maxlen().bits(pdu_len) });

    radio.events_ready.reset();
    radio.events_end.reset();
    radio.events_disabled.reset();

    radio.tasks_rxen.write(|w| w.tasks_rxen().trigger());
    while radio.events_ready.read().events_ready().is_not_generated() {}

    radio.tasks_start.write(|w| w.tasks_start().trigger());
    while radio.events_end.read().events_end().is_not_generated() {}

    radio.tasks_disable.write(|w| w.tasks_disable().trigger());
    while radio
        .events_disabled
        .read()
        .events_disabled()
        .is_not_generated()
    {}
}

struct Uart<'a> {
    uart: &'a pac::UART0,
}

impl<'a> Uart<'a> {
    pub fn new(uart: &'a pac::UART0) -> Self {
        uart.psel.rxd.write(|w| unsafe {
            w.port().clear_bit().pin().bits(8).connect().connected()
        });

        uart.psel.txd.write(|w| unsafe {
            w.port().clear_bit().pin().bits(6).connect().connected()
        });

        uart.baudrate.write(|w| w.baudrate().baud115200());
        uart.enable.write(|w| w.enable().enabled());
        uart.tasks_starttx.write(|w| w.tasks_starttx().trigger());

        Self { uart }
    }
}

impl<'a> Write for Uart<'a> {
    fn write_str(&mut self, s: &str) -> core::fmt::Result {
        for c in s.as_bytes() {
            self.uart.events_txdrdy.reset();
            self.uart.txd.write(|w| unsafe { w.txd().bits(*c) });
            while self
                .uart
                .events_txdrdy
                .read()
                .events_txdrdy()
                .is_not_generated()
            {}
        }

        Ok(())
    }
}

#[entry]
fn main() -> ! {
    let mut cp = cortex_m::Peripherals::take().unwrap();
    let dp = pac::Peripherals::take().unwrap();

    clock_config(&dp);
    systick_config(&mut cp);

    io_config(&dp.P0);

    ble_config_adv(&dp);

    let mut loop_count: u32 = 0;

    let mut ble_buf = [0_u8; 255 + 2];

    let radio = &dp.RADIO;

    let mut uart = Uart::new(&dp.UART0);

    loop {
        delay_ms(1000);
        loop_count = loop_count.wrapping_add(1);

        ble_recv_adv(&dp, &mut ble_buf);

        write!(uart, "\r\nloop count: {}\r\n", loop_count,).unwrap();

        if radio.crcstatus.read().crcstatus().is_crcerror() {
            continue;
        }

        let len = ble_buf[1] as usize;

        write!(uart, "payload: {:02x?}\r\n", &ble_buf[..len + 2]).unwrap();

        write!(
            uart,
            "RSSI: -{} dBm\r\n",
            dp.RADIO.rssisample.read().rssisample().bits()
        )
        .unwrap();

        let pdu_type = ble_buf[0] & 0xf;
        let ch_sel = (ble_buf[0] & 0x20) != 0;
        let tx_add = (ble_buf[0] & 0x40) != 0;
        let rx_add = (ble_buf[0] & 0x80) != 0;

        write!(uart, "pdu type: {}\r\n", PDU_TYPE_STR[pdu_type as usize])
            .unwrap();

        write!(
            uart,
            "ChSel: {}, TxAdd: {}, RxAdd: {}\r\n",
            ch_sel, tx_add, rx_add
        )
        .unwrap();

        match pdu_type {
            0 | 1 | 2 | 6 => {
                write!(
                    uart,
                    "AdvA: {:02x}:{:02x}:{:02x}:{:02x}:{:02x}:{:02x}\r\n",
                    ble_buf[7],
                    ble_buf[6],
                    ble_buf[5],
                    ble_buf[4],
                    ble_buf[3],
                    ble_buf[2],
                )
                .unwrap();

                let sub_type = ble_buf[7] >> 6;

                write!(
                    uart,
                    "addr sub-type: {}\r\n",
                    match sub_type {
                        0 => "Non-resolvable private address",
                        1 => "Resolvable private address",
                        2 => "Reserved for future use",
                        3 => "Static device address",
                        _ => panic!(),
                    }
                )
                .unwrap();
            }
            _ => {}
        }

        match pdu_type {
            0 | 2 | 6 => {
                write!(uart, "AdvData: {:02x?}\r\n", &ble_buf[8..len + 2])
                    .unwrap();
            }
            _ => {}
        }
    }
}
```

참고
- <https://www.bluetooth.com/ko-kr/specifications/specs/core-specification-5-3/>
- <https://infocenter.nordicsemi.com/topic/ps_nrf52833/radio.html>

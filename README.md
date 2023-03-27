```
rustup update
rustup target add thumbv7em-none-eabihf
cargo install cargo-binutils
```

```
cargo new rust-stm32-1
cd rust-stm32-1
cargo add cortex-m-rt
cargo add cortex-m
cargo add embedded-hal
cargo add stm32f4xx-hal --features=stm32f401
cargo add nb
cargo add panic-halt
```

```
mkdir .cargo
touch .cargo/config.toml
```

`.cargo/config.toml`

```
[target.thumbv7em-none-eabihf]
runner = 'probe-run --chip STM32F411CEUx'
rustflags = [
  "-C", "link-arg=-Tlink.x",
]

[build]
target = "thumbv7em-none-eabihf"
```

```
touch memory.x
```

`memory.x`

```
MEMORY
{
  /* NOTE K = KiBi = 1024 bytes */
  FLASH : ORIGIN = 0x08000000, LENGTH = 128K 
  RAM : ORIGIN = 0x20000000, LENGTH = 32K
}

/* This is where the call stack will be allocated. */
/* The stack is of the full descending type. */
/* NOTE Do NOT modify `_stack_start` unless you know what you are doing */
_stack_start = ORIGIN(RAM) + LENGTH(RAM);
```

`src/main.rs`

```rust
#![deny(unsafe_code)]
#![allow(clippy::empty_loop)]
#![no_main]
#![no_std]

// Halt on panic
use panic_halt as _; // panic handler

use cortex_m_rt::entry;
use stm32f4xx_hal as hal;

use crate::hal::{pac, prelude::*};

#[entry]
fn main() -> ! {
    if let (Some(dp), Some(cp)) = (
        pac::Peripherals::take(),
        cortex_m::peripheral::Peripherals::take(),
    ) {
        // Set up the LED. On the Nucleo-446RE it's connected to pin PA5.
        let gpioa = dp.GPIOA.split();
        let mut led = gpioa.pa5.into_push_pull_output();

        // Set up the system clock. We want to run at 48MHz for this one.
        let rcc = dp.RCC.constrain();
        let clocks = rcc.cfgr.sysclk(48.MHz()).freeze();

        // Create a delay abstraction based on SysTick
        let mut delay = cp.SYST.delay(&clocks);

        loop {
            // On for 1s, off for 1s.
            led.toggle();
            delay.delay_ms(1000_u32);
        }
    }

    loop {}
}
```

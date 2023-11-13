+++
title = 'Rust Raspberrypi'
date = 2023-11-07T10:45:35Z
draft = false
+++

### 本文主要对学习rust-raspberrypi-OS-tutorials中相关rust的理解

#### 01_wait_forever
``` rust
#[cfg(target_arch = "aarch64")]
#[path = "../_arch/aarch64/cpu/boot.rs"]
mod arch_boot;
```
mod模块的定义可以通过path宏去定义，将默认模块的路径设置到自己想放的位置

``` rust
use core::panic::PanicInfo;

#[panic_handler]
fn panic(_info: &PanicInfo) -> ! {
    unimplemented!()
}
```
对于no std中对panic的处理，需要自己定义panic_handler

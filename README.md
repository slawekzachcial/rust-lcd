# LCD HD44780 Driver for Raspberry Pi

Rust implmentation of Raspberry Pi driver for
[LCD](http://wiki.sunfounder.cc/index.php?title=LCD1602_Module) compatible with
[HD44780](https://www.sparkfun.com/datasheets/LCD/HD44780.pdf).

The implementation is a port of Arduino [LiquidCrystal](https://github.com/arduino-libraries/LiquidCrystal)
library. The integration with Raspberry Pi is based on [gpio-cdev](https://github.com/rust-embedded/gpio-cdev).

## Example

### Connections

Connect LCD display to Raspberry Pi [as follows](https://www.youtube.com/watch?v=cVdSc8VYVBM):
* LCD pin 1 (ground) => Pi ground
* LCD pin 2 (VCC) => Pi 5V
* LCD pin 3 (contrast) => Pi ground
* LCD pin 4 (rs) => Pi GPIO 26
* LCD pin 5 (r/w) => Pi ground
* LCD pin 6 (enable) => Pi GPIO 19
* LCD pin 11 (d4) => Pi GPIO 13
* LCD pin 12 (d5) => Pi GPIO 06
* LCD pin 13 (d6) => Pi GPIO 05
* LCD pin 14 (d7) => Pi GPIO 11
* LCD pin 15 (led+) => Pi 5V
* LCD pin 16 (led-) => Pi ground

> This connection type uses LCD 4-bit mode for data and allows to only write
> to the LCD (r/w => ground).

### Code

```rust
use gpio_cdev::errors;
use lcd::{CharSize, GpioPin::*, Pins, LCD};

fn do_main() -> Result<(), errors::Error> {

    let mut lcd = LCD::new(Pins {
        rs: P26,
        rw: None,
        enable: P19,
        data: [NONE, NONE, NONE, NONE, P13, P06, P05, P11],
    })?;

    lcd.begin(16, 2, CharSize::Dots5x8);

    lcd.set_cursor(0, 0);
    lcd.print("Hello, ...");
    lcd.set_cursor(0, 1);
    lcd.print("... world!");
}

fn main() {
    match do_main() {
        Ok(()) => {}
        Err(e) => {
            eprintln!("Error {:?}", e);
        }
    }
}
```

> Note that `Pxx` are GPIO pins and so, for example, P26 is pin GPIO 26 which
> is on Pi's pin 37 as shown on [this diagram](https://www.raspberrypi.org/documentation/usage/gpio/).

## Building

In order to use this library, it needs to be cross-compiled for Raspberry Pi.

This can be accomplished using [rust-embedded/cross](https://github.com/rust-embedded/cross)
tool. The tool documentation provides details how to install and use it.

Once installed, to compile the library use the following command:
```
cross build --target arm-unknown-linux-musleabihf
```

The project contains a binary that demos all the library functions. Assuming
your Raspberry Pi is available at DNS name `raspberrypi.local` and you set up
SSH server on it, you can copy it to the board using the following command:
```
scp target/arm-unknown-linux-musleabihf/debug/lcd pi@raspberrypi.local:/home/pi
```

Finally, to run the demo binary:
```
ssh pi@raspberrypi.local /home/pi/lcd
```

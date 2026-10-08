# l3gd20-nv

The L3GD20 is ST's three-axis digital gyroscope: it measures the rate at
which it turns about each of three axes and reports it over an SPI or
I2C bus, as its data sheet (DocID022116) describes.  ST's I3G4250D has
the same registers and sensitivities and answers another identification
byte.  This package drives either part on an SPI bus and presents it as
[devices-nv](https://novo-lang.org/packages/devices-nv)'s `Gyroscope`
and `Identifiable`.

## What it is

A **gyroscope** reports the **angular rate**: how fast the board turns
about an axis, in degrees per second (dps).  At rest it reads close to
zero on every axis.  The part measures within a **full scale**, ±250,
±500 or ±2000 dps; a wider scale reaches faster turns and resolves slow
ones more coarsely.  Each axis is a 16-bit count, and the data sheet's
**sensitivity** says what one step of the count is worth at each full
scale (§ 2.1, Table 4).  The driver answers every reading in
milli-degrees per second.

The part samples at one of four **output data rates**, 95, 190, 380 or
760 samples a second, and raises a **data-ready** flag when a sample of
all three axes is complete.  It can drive the flag onto its DRDY/INT2
pin, so a board can see a new sample without asking the part.

On **SPI** a transaction is a command byte and the data inside one
chip-select window: bit 7 of the command reads, bit 6 makes the
register address advance after each byte, and bits 5 to 0 name the
register (§ 5.2).  The part's identification byte, **WHO_AM_I** at
register 0x0F, is 0xD4 on an L3GD20 and 0xD3 on an I3G4250D.

## Install

```
novo pkg add l3gd20-nv
```

## Example

A board names the part in the `devices` block of its declaration, as
the STM32F3DISCOVERY's does:

```novo norun:fragment
    devices
        gyro  L3gd20  from "l3gd20-nv"  on spi0  cs gyro_cs  irq gyro_drdy
```

and a program reads it as a gyroscope:

```novo norun:needs-pkg
use dev_gyro.{ Gyroscope }
use l3gd20

@tier(embedded)
fn main() [hw]
    let gyro = bsp.board.gyro()
    // Brings the part up: ±250 dps, 95 samples a second, data-ready
    // on DRDY/INT2.
    if gyro.begin()
        match gyro.gyro_read()
            // The rate about z, in milli-degrees per second.
            Some(s) => hal.uart.write(str.from_int(s.z))
            None    => hal.uart.write("no sample")
```

## What the package contains

| Module | What is in it |
| --- | --- |
| `l3gd20` | `L3gd20`, the driver: `attach`, `begin`, `set_rate_hz`, `irq_level`, `reg` and `set_reg`, and devices-nv's `Gyroscope` and `Identifiable`; and the same operations as functions over any `SpiBus` and `GpioOut`, with the register map's conversions |

## How to choose an entry point

A program on a board uses `L3gd20`, which the board's device table
builds.  A program that holds its own bus, or a test with a double of
the part, calls the module's functions, which take the bus, the GPIO
and the wiring as arguments.

## The rules a user needs

1. `begin` must run before a reading: the part leaves power-on powered
   down.  It answers `false` for a part that does not answer as an
   L3GD20 or an I3G4250D.
2. The part has no ±1000 dps scale (§ 7.5, Table 30), so
   `gyro_set_range(GyroRange1000dps)` answers `false` and changes
   nothing.
3. A reading is converted at the scale it was taken in: the driver
   reads the scale and the sample in one transaction.
4. At rest a reading is not exactly zero.  The part's zero-rate level is
   within ±10 dps at ±250 dps (§ 2.1, Table 4); a program that
   integrates the rate measures and subtracts it first.
5. The part listens as an I2C target while its chip select is high
   (§ 5).  `begin` spends one read before it reads WHO_AM_I, because
   the first select after other traffic on the bus can answer a bit
   late.

## What is not included

- The I2C interface: the boards that carry the part wire it to SPI.
- The FIFO, the high-pass filter and the interrupt generator's
  thresholds: no trait of devices-nv names them yet; `set_reg` reaches
  the registers.
- The temperature byte, OUT_TEMP, which the data sheet does not
  calibrate.

## Related packages

- [devices-nv](https://novo-lang.org/packages/devices-nv) names the
  `Gyroscope` trait this driver implements, the wiring record and the
  SPI helpers it uses.
- [embedded-hal-nv](https://novo-lang.org/packages/embedded-hal-nv)
  names the `SpiBus` and `GpioOut` traits and the board handles.
- lsm303agr-nv and lsm303dlhc-nv drive the accelerometer and
  magnetometer that share the STM32F3DISCOVERY with this part.

## Tests

`novo test tests` runs the module's functions over a double of the
part's register file: the command byte, the address increment, the
conversions at each scale, the rates, the bring-up and a bus with no
part; and each method of the type `L3gd20` over the host's board
handles, which are absent hardware whose every byte reads 0.  The
driver on hardware is measured by the STM32F3DISCOVERY's suite in the
novo-lang repository,
`orbit/bsp/st/stm32f3/stm32f3discovery/tests/imu.sh`, which reads the
part on the board and the coverage of this module there.

## Licence

Apache-2.0.

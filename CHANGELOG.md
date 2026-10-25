# Changelog

## 0.1.0 (2026-10-08)

The first release: a driver for ST's L3GD20 three-axis gyroscope, and the
register-compatible I3G4250D, on an SPI bus.

- `L3gd20`, the type a board's device table builds from its wiring,
  implementing devices-nv's `Gyroscope` and `Identifiable` (WHO_AM_I
  0xD4 or 0xD3), with its own methods for the bring-up, the sample
  rate, the data-ready line's level and register access.
- The register access as functions over any `SpiBus` and `GpioOut`,
  which the type's methods call.
- The host suite runs the functions over a double of the part's
  register file and the type's methods over the host's board handles.

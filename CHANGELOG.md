# Changelog

## 0.1.2 — 2026-10-28

The dependency requirement is raised: `devices-nv` from ^0.2.0 to ^0.3.2.  The newest version of each builds with no migration warning under the 0.22.0 compiler, and the older one drew E2050, E2052 or E2055 warnings into this package's build.  The tests and the magnetometer wiring carry the new `en` field of `DeviceWiring`, which devices-nv 0.3 added.  No signature and no behaviour changed.

## 0.1.1 — 2026-10-25

A call that passes a variable to a `var` parameter now writes the `var` marker, which the 0.21.1 compiler asks for (E2050).  The package needs Novo 0.21.1 or later.  No signature and no behaviour changed.

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

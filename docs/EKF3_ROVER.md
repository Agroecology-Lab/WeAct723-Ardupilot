# EKF3 on the WeAct H723: ArduRover board

The stock `WeActH723` hwdef (Copter, bootloader, 640 KB firmware region) cannot
hold EKF3. Measured against ArduPilot `24b18174`, GCC 13.2:

| Build | Result |
| --- | --- |
| `WeActH723`, Copter, EKF3 off | Fails to link, 0x520 bytes (about 1.3 KB) over 640 KB |
| `WeActH723`, Rover, EKF3 on | Fails to link, 0x7C40 bytes (about 31 KB) over 640 KB |
| `WeActH723-Rover`, Rover, EKF3 on | Links. 687,876 B of flash used, about 96 KiB free of 768 KiB |

## WeActH723-Rover

`hwdef/WeActH723-Rover/hwdef.dat` includes the base hwdef and then:

- enables EKF3 (EKF2 stays off),
- drops the bootloader (`FLASH_RESERVE_START_KB 0`), so firmware gets sectors
  0-5 (768 KB) and parameters keep sectors 6-7,
- turns off bootloader flashing and the firmware check, which need a bootloader,
- enables `GPS_MOVING_BASELINE`, which `minimize_features.inc` turns off. Without
  it `GPS_TYPE 17`/`18` (two-F9P heading) does not exist.

## Consequences

- No bootloader: flash by DFU at `0x08000000` (hold BOOT0). No USB firmware
  updates, and a bad flash is recovered the same way.
- It replaces the layout in the README. Do not flash `ardurover.bin` at
  `0x08020000`.
- Build with `./waf configure --board WeActH723-Rover && ./waf rover`. The
  `apply_patches.sh` script copies every directory under `hwdef/`.
- Load `params/sowbot-serial.parm` then `params/sowbot-ekf3.parm`.

## Not verified

- Built only. Nothing has run on the board or in SITL.
- Logging, scripting and optical flow stay off. The Hiwonder EMM patches were
  not in the measured build, so their flash cost is unknown (the headroom above
  should cover it, but check).
- With logging off there are no dataflash logs to tune the EKF from.

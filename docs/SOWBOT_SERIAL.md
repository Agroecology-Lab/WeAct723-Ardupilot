# Sowbot serial layout (two F9P + Avaota)

The board has exactly three UARTs besides USB, so this layout uses all of them:

| Port | UART | Pins | Device |
| --- | --- | --- | --- |
| SERIAL1 | USART1 | PA9 TX / PA10 RX | F9P #1, base (GPS1) |
| SERIAL2 | USART2 | PA2 TX / PA3 RX | F9P #2, rover (GPS2) |
| SERIAL3 | UART4 | PB9 TX / PB8 RX | Avaota SBC, MAVLink2 |

Load `params/sowbot-serial.parm`. No hwdef change is needed.

## Consequences

- No UART is left for an RC receiver or a telemetry radio. Both must go through
  the SBC or USB.
- UART4 is declared `NODMA` in the hwdef. Fine for 115200 MAVLink, but check
  for dropped frames under load.
- Cross TX/RX between the board and each device. Check the Avaota's UART logic
  level is 3.3 V.

## Not verified

- Not built or run. The hwdef uses `minimize_features.inc`, which may drop GPS
  drivers. Check that `GPS_TYPE 17`/`18` (u-blox moving baseline) are in the
  build before relying on this.
- EKF2 and EKF3 are disabled in the hwdef, so the heading-from-GPS source
  (`EK3_SRC1_YAW 3` in LEVIA's `rover.parm`) is not set here. Whether DCM
  uses the GPS yaw on this build is untested.

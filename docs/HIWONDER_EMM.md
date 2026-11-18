# Hiwonder 4-channel encoder motor driver

Ported from the PX4 `hiwonder_emm` driver as `AP_HiwonderEMM`, enabled by
`define AP_HIWONDEREMM_ENABLED 1` in the hwdef.

Status: compiles for LEVIA_H7 (Rover) upstream; **not built for WeActH723 at all**. **Not yet run in SITL or on hardware.**
The I2C register map and speed scaling are copied from PX4 and unverified
against Hiwonder's documentation.

## Wiring (WeActH723)

`I2C_ORDER I2C1 I2C2` in the hwdef makes I2C1 (PB6/PB7, the BMP280) ArduPilot
bus 0 and I2C2 (PB10/PB11, the external compass) bus 1. `SERVO_EMM_BUS` defaults
to 1, so the EMM board goes on PB10/PB11 alongside the compass. Address 0x34 does
not clash with the supported compasses. Use bus 0 only if you move the barometer.
Check pull-ups and logic levels on the motor board before connecting.

## Limits on this board

- The hwdef sets `HAL_NAVEKF3_AVAILABLE 0`, `HAL_LOGGING_ENABLED 0` and
  `AP_OPTICALFLOW_ENABLED 0` to fit 640 KB. The EMM log message, and EKF3
  wheel-encoder fusion (`WENC_TYPE 11`), need those back on. That is unlikely to
  fit, so expect to use only the motor output and arming check here.
- The hwdef is a Copter build. The skid-steer set-up below needs ArduRover.

## Parameters

| Name | Default | Meaning |
| --- | --- | --- |
| `SERVO_EMM_MASK` | 0 | Servo channels sent to the board, lowest bit is motor 1, max 4. Reboot after changing. |
| `SERVO_EMM_BUS` | 1 | I2C bus |
| `SERVO_EMM_ADDR` | 52 (0x34) | I2C address |
| `SERVO_EMM_TYPE` | 3 | 0 none, 1 TT, 2 N20, 3 JGB37-520 12V 110RPM |
| `SERVO_EMM_POL` | 0 | Encoder polarity |

Channels in the mask stop producing PWM, and their `SERVOn_MIN/MAX/TRIM` are
forced to 1000/2000/1500. 1500 is stopped; 1000..2000 maps to speed -127..127
(PX4 used 0..255 minus 128, which was asymmetric).

For a skid-steer Rover, set `SERVO1`/`SERVO2` to throttleLeft and
`SERVO3`/`SERVO4` to throttleRight, then `SERVO_EMM_MASK` to 15.

## Behaviour added over PX4

- Motors are commanded to zero if the main loop stops updating for 200 ms.
  This does not cover a hung I2C bus.
- A missing board is retried once a second and blocks arming
  (`EMM not found on bus N`).
- Supply voltage and encoder counts are logged as `EMM` at 10 Hz.

## Bench test before driving

Wheels off the ground. Confirm the disarmed output is stopped, that each motor
follows the right servo channel and direction, and that motors stop when the
autopilot is reset or the I2C cable is pulled.

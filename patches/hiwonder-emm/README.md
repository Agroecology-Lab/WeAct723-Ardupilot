# Hiwonder EMM patches

Five `git format-patch` files adding the `AP_HiwonderEMM` I2C encoder motor
driver, its SRV_Channel output backend, a pre-arm check and a wheel encoder
backend. Taken unchanged from
[Agroecology-Lab/levia-h7-ardupilot](https://github.com/Agroecology-Lab/levia-h7-ardupilot)
(`ardupilot/patches/`, commit 6fdb461).

They target the ArduPilot commit in `UPSTREAM_COMMIT` and apply cleanly there.
Other revisions may drift. Apply with `WITH_EMM=1 scripts/apply_patches.sh`, or
by hand from the ArduPilot root: `git apply patches/hiwonder-emm/*.patch`.

Nothing is enabled until the hwdef has `define AP_HIWONDEREMM_ENABLED 1`.
See `docs/HIWONDER_EMM.md`.

# Field operations and safety rules

## Non-negotiable rules

1. The agent **never pushes to the glider without an explicit per-version YES** from the allowlisted Telegram user.
2. Every pushed image must have **passed the bench-twin test suite** at that exact git hash.
3. OTA only when the glider reports a **safe state** (surfaced, idle, battery OK).
4. Rollback must be possible: **test-swap + confirm** only (see firmware hardening #1).
   Until that is in place, do field OTA only when the glider can be recovered by hand.
5. Every deployment is tagged in git and logged (who approved it, when, hash, result).

## Staged validation before relying on it

1. Bench: twin ESP32 on the Pi, OTA over Wi-Fi at 1 m.
2. Bench: forced-failure image → verify automatic revert.
3. Bucket/pool: glider in water at the surface, OTA from the Pi at a few metres.
4. Open water, tethered/close range: measure the maximum reliable range and transfer time.
5. Full mission with a planned OTA at a surfacing.

## Surfacing procedure (target, automated by the Pi)

1. Watcher sees `Tuba-Glider` SSID → joins, reads `$STATUS`.
2. If an approved update is queued and the state is safe → push → wait for reboot.
3. Reconnect → check version/self-test → `confirm` → report on Telegram.
4. If anything fails → do not confirm (automatic revert) → report → leave the glider on its old image.

## Rollback

- Automatic: unconfirmed image + reset → MCUboot reverts.
- Manual: push the previous `deployed/*` tag's image (built and tested again) through the same approval flow.

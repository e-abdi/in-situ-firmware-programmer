# Roadmap

## Phase 0: iPhone ([guide](docs/02-iphone-setup.md))
- [ ] Telegram + 2-step verification + passcode
- [ ] Bot created via @BotFather; token and own user ID saved
- [ ] Dictation / voice message permissions
- [ ] Tailscale installed; emergency SSH path chosen (iPhone is on iOS 16.7, no Termius)

## Phase 1: Raspberry Pi base ([guide](docs/03-raspberry-pi-setup.md))
- [ ] OS, SSH, Tailscale, repo cloned with submodules
- [ ] Zephyr toolchain; `./build.sh` produces `zephyr.signed.bin` on the Pi
- [ ] Embedded-AI-Harness installed, bench twin detected
- [ ] Second Wi-Fi adapter joins `Tuba-Glider` without breaking the uplink

## Phase 2: Manual → scripted OTA
- [ ] Manual OTA from the Pi to the bench twin (menu 5 + URL)
- [ ] `ota/ota_push.py`: join AP, read status, serve image, trigger OTA, verify new version
- [ ] Measure transfer time and range

## Phase 3: Firmware hardening ([details](docs/04-firmware-hardening.md))
- [ ] Test-swap + self-test + confirm-after-reconnect; watchdog
- [ ] Revert verified on the bench with a deliberately bad image
- [ ] Own signing key (bootloader reflashed over USB)
- [ ] WPA2 + console authentication
- [ ] OTA only in a safe state
- [ ] `$STATUS` line + non-interactive `ota <url>` / `confirm` commands

## Phase 4: AI build/test loop
- [ ] Test plan for the bench twin (simulation mode)
- [ ] Agent skills/tools in `agent/` (build, test_on_twin, diff, status)
- [ ] Agent completes a change end-to-end on the bench from a terminal prompt

## Phase 5: Telegram agent
- [ ] Hermes Agent or OpenClaw installed, Telegram gateway restricted to own user ID
- [ ] Whisper transcription of voice messages, echoed back
- [ ] Approval gate: exact "YES" per version before `ota_push`
- [ ] systemd services; survives reboot

## Phase 6: Field
- [ ] Surfacing watcher + queued updates
- [ ] Deployment tags + logs
- [ ] Staged validation (bench → pool → tethered → mission), see [field ops](docs/05-field-operations.md)

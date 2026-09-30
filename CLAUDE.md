# CLAUDE.md

Context for AI agents working in this repository (on the Raspberry Pi).

## What this repo is
Tooling to reprogram the Tuba underwater glider (ESP32, Zephyr 4.3, MCUboot) at sea:
Telegram request → agent edits `firmware/tuba-firmware` → build → test on the bench twin →
human approval → OTA over the glider's Wi-Fi AP when it surfaces.
Read `README.md`, `docs/01-architecture.md`, and `ROADMAP.md` first; `ROADMAP.md` shows the current phase.

## Hard rules
- **Never trigger an OTA to the real glider without an explicit per-version approval
  from the pilot.** Bench-twin flashing is fine.
- Never commit secrets: `.env`, signing keys (`*.pem`), Telegram tokens, Wi-Fi keys.
- Firmware changes go on a branch inside `firmware/tuba-firmware` (`agent/<topic>`),
  not directly on its main branch. Bump the submodule pin here only after tests pass.
- Safety-critical logic (approval, state checks, OTA push) lives in plain scripts
  (`ota/`, `agent/`), not in prompts.

## Layout
- `firmware/tuba-firmware/`: submodule, glider firmware. Build: `./build.sh`
  (`west build --sysbuild --board=esp32_devkitc/esp32/procpu`), OTA image:
  `build/tuba/zephyr/zephyr.signed.bin`. OTA code: `src/ota_simple.c`, menu: `src/ui_menu.c`.
- `third_party/Embedded-AI-Harness/`: submodule, Pi test bench (serial over RFC2217,
  device API on :8080) and `.claude/skills` for the closed-loop workflow.
- `docs/`: design and setup docs; keep them up to date as steps are completed.

## Known firmware gaps
See `docs/04-firmware-hardening.md`: permanent (non-revertible) upgrades, default signing
key, open Wi-Fi + unauthenticated telnet. Don't do field OTA before these are fixed.

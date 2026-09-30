# Raspberry Pi setup (outline, to be filled in on the Pi)

## Hardware shopping list

- **Raspberry Pi 5, 8 GB** (Zephyr builds are heavy; a Pi 4 works but is slower)
- NVMe SSD via an M.2 HAT (strongly recommended over an SD card for builds), or a good A2 SD card
- Official 27 W PSU (bench) and a 12 V → 5 V 5 A converter (boat)
- **Second USB Wi-Fi adapter** with an external antenna connector
  (2.4 GHz, Linux in-kernel driver, e.g. MediaTek MT7612U/MT7610U-based) +
  a directional/high-gain 2.4 GHz antenna for the glider link
- Internet uplink: 4G/LTE HAT or USB modem, **or** iPhone hotspot, **or** Starlink
- **Bench twin:** spare ESP32 DevKitC WROOM-32U (same as the glider), plus
  ideally the same sensors, or a stubbed/simulated setup
- USB hub (powered) if you use several boards
- Weatherproof enclosure for field use

## Software steps (planned)

1. **OS:** Raspberry Pi OS Lite 64-bit (Bookworm or newer), SSH on, hostname `tuba-pi`.
2. **Access:** install Tailscale (`curl -fsSL https://tailscale.com/install.sh | sh && sudo tailscale up --ssh`),
   and set up the browser terminal for the iOS 16 phone: `ttyd` bound to the
   Tailscale interface with a login (`ttyd -i tailscale0 -c user:pass -W bash`) as a systemd service.
3. **Clone this repo** with `--recurse-submodules` (see README).
4. **Zephyr toolchain:** Python venv, `pip install west`, `west init`/`west update`
   for the Zephyr version tuba uses (4.3), Zephyr SDK for Xtensa ESP32,
   `west blobs fetch hal_espressif`. Verify with `cd firmware/tuba-firmware && ./build.sh`.
5. **Signing key:** `imgtool keygen` → store outside the repo (e.g. `~/keys/`); reference in `.env`.
6. **Embedded-AI-Harness:** `sudo bash third_party/Embedded-AI-Harness/pi/install.sh`,
   plug in the twin, verify `curl http://localhost:8080/api/devices`.
   Check its Wi-Fi AP feature doesn't grab the glider-link adapter.
7. **Networking:** NetworkManager profiles: uplink on `wlan0`/modem, and the
   glider link on `wlan1` joining `Tuba-Glider` (no default route via wlan1!).
8. **Manual OTA test:** reproduce the OTA by hand (serve `zephyr.signed.bin`,
   telnet, menu 5, URL), then script it as `ota/ota_push.py`.
9. **STT:** faster-whisper or whisper.cpp (`small` or `base` model is enough on a Pi 5).
10. **Agent:** install Hermes Agent *or* OpenClaw, configure the Telegram gateway with
    `TELEGRAM_BOT_TOKEN` and restrict it to `TELEGRAM_ALLOWED_USER_ID`, and point it at
    an LLM provider (API key in `.env`).
11. **Agent tools:** expose `build`, `test_on_twin`, `glider_status`, `ota_push`
    (needs approval), `rollback`, and `revert_commit` as scripts/skills in `agent/`.
12. **Voice bridge** for hands-free Siri use: small HTTP service (e.g. FastAPI) on
    `tailscale0:8088` with `/voice`, `/status`, `/approve`, `/cancel`, bearer-token auth
    (`VOICE_BRIDGE_TOKEN`), which forwards to the agent and mirrors everything to Telegram.
    See `docs/02-iphone-setup.md` §4C.
13. **systemd services:** agent gateway, voice bridge, OTA HTTP server, surfacing watcher.

Details are written up here as each step is done on the Pi.

# Architecture

## Components

| Component | Where | Responsibility |
|---|---|---|
| iPhone + Siri Shortcuts | With the pilot | Hands-free voice requests/status/approval via HTTPS to the Pi (over Tailscale) |
| iPhone + Telegram | With the pilot | Log of everything, reviewing diffs on screen, text/voice backup input, `/cancel` |
| Telegram cloud | Internet | Message transport (bot API, long polling from the Pi) |
| Tailscale | Internet | Private network between the iPhone and the Pi (no open ports) |
| Voice bridge | On the Pi, `:8088`, Tailscale interface only | Small HTTP service: `/voice`, `/status`, `/approve`, `/cancel`; token auth; mirrors everything to Telegram |
| Raspberry Pi 5 | Boat, buoy or shore, near the glider | Agent, STT, build, bench test, OTA push |
| Internet uplink | On the Pi | 4G/LTE HAT or USB modem, Starlink, or iPhone hotspot |
| Glider-link Wi-Fi adapter | On the Pi | Second USB Wi-Fi adapter + high-gain/directional antenna; joins `Tuba-Glider` |
| Bench twin ESP32 | USB on the Pi | Same hardware as the glider, running the candidate firmware in simulation mode |
| Glider ESP32 | In the glider | Runs tuba-firmware; MCUboot swaps slots after OTA |

## Request flow

1. Pilot says *"Hey Siri, glider"* → *"make the pump timeout 120 seconds"*. The Siri
   Shortcut POSTs the text to the voice bridge (backup: Telegram text or voice message).
2. The bridge checks the token (Telegram: the sender ID), posts the request to the
   Telegram chat, and hands it to the agent.
3. The agent answers with its interpretation + plan: spoken back through the Shortcut
   (short) and posted to Telegram (full). Voice messages are transcribed with Whisper.
4. Agent creates a branch in `firmware/tuba-firmware`, edits code or parameters.
5. `west build --sysbuild` produces `zephyr.signed.bin` (signed with the private key on the Pi).
6. Agent flashes the **bench twin** over USB and runs the test suite (Embedded-AI-Harness).
7. Agent reports: diff summary, build size, test results, version/git hash.
8. **Pilot approves this exact version**: by voice ("Hey Siri, glider approve" + the
   spoken challenge, e.g. "approve 1 4 3 tango") or YES on Telegram. Anything else = abort.
9. OTA push is queued until the glider surfaces (its AP appears to the glider-link adapter).
10. Pi joins the AP → telnet → checks state is safe → sends the OTA URL (Pi serves the image over HTTP) → glider downloads, verifies, reboots into **test** mode.
11. Pi reconnects, checks the new version + self-test result, sends `confirm`.
    If this doesn't happen in time, the watchdog/MCUboot reverts to the previous image.
12. Agent reports the outcome, tags the commit (`deployed/<date>-<ver>`), logs everything.

## Key design decisions

- **Hands-free first, but approval is deliberate.** Voice approval needs a per-build spoken challenge.
- **Human approval before every glider push.** The agent may freely edit/build/test
  on the bench; it may never push to the vehicle on its own.
- **Bench twin first.** Nothing reaches the glider without passing on identical hardware.
- **Rollback by default.** MCUboot test-swap + confirm-after-reconnect
  (see [04-firmware-hardening.md](04-firmware-hardening.md)).
- **Signing key lives only on the Pi** (plus an offline backup).
- **Surface-only.** The OTA window is one surfacing; the image transfer must fit in it.
- **The agent is swappable.** Hermes Agent and OpenClaw both work; the deterministic
  parts (build, test, OTA push) are plain scripts in this repo that the agent calls
  as tools, so the safety logic lives outside the LLM.

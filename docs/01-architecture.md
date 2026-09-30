# Architecture

## Components

| Component | Where | Responsibility |
|---|---|---|
| iPhone + Telegram | With the pilot | Voice/text requests, reviewing plans and diffs, approving OTA |
| Telegram cloud | Internet | Message transport (bot API, long polling from the Pi) |
| Raspberry Pi 5 | Boat, buoy or shore, near the glider | Agent, STT, build, bench test, OTA push |
| Internet uplink | On the Pi | 4G/LTE HAT or USB modem, Starlink, or iPhone hotspot |
| Glider-link Wi-Fi adapter | On the Pi | Second USB Wi-Fi adapter + high-gain/directional antenna; joins `Tuba-Glider` |
| Bench twin ESP32 | USB on the Pi | Same hardware as the glider, running the candidate firmware in simulation mode |
| Glider ESP32 | In the glider | Runs tuba-firmware; MCUboot swaps slots after OTA |

## Request flow

1. Pilot sends a voice message: *"make the pump timeout 120 seconds"*.
2. The agent's Telegram gateway receives it; the agent checks the sender ID is on the allowlist.
3. Whisper transcribes the audio; the agent replies with the transcription + its plan.
4. Agent creates a branch in `firmware/tuba-firmware`, edits code or parameters.
5. `west build --sysbuild` produces `zephyr.signed.bin` (signed with the private key on the Pi).
6. Agent flashes the **bench twin** over USB and runs the test suite (Embedded-AI-Harness).
7. Agent reports: diff summary, build size, test results, version/git hash.
8. **Pilot replies YES** (exact, per-version approval). Anything else = abort.
9. OTA push is queued until the glider surfaces (its AP appears to the glider-link adapter).
10. Pi joins the AP → telnet → checks state is safe → sends the OTA URL (Pi serves the image over HTTP) → glider downloads, verifies, reboots into **test** mode.
11. Pi reconnects, checks the new version + self-test result, sends `confirm`.
    If this doesn't happen in time, the watchdog/MCUboot reverts to the previous image.
12. Agent reports the outcome, tags the commit (`deployed/<date>-<ver>`), logs everything.

## Key design decisions

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

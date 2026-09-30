# In-Situ Firmware Programmer

Reprogram the **Tuba underwater glider** while it is deployed, by talking to a
Telegram bot from an iPhone, **hands-free with Siri**.

```
 "Hey Siri, glider" → "Increase max dive depth to 80 m"     (hands-free)
        │
   iPhone ── Siri Shortcut (HTTPS over Tailscale) ─┐
          └─ Telegram (log, diffs, backup input) ──┴► Raspberry Pi (boat / shore station)
                            ├─ AI agent (Hermes Agent or OpenClaw) with Telegram gateway
                            ├─ edits tuba-firmware, builds it (Zephyr + west)
                            ├─ tests it on a bench "twin" ESP32 (Embedded-AI-Harness)
                            ├─ asks you to approve (spoken challenge or Telegram)
                            └─ at surfacing: joins glider Wi-Fi → telnet → OTA over HTTP
                                                               │
                                                       Glider ESP32 (MCUboot)
```

Radio does not get through seawater, so **the glider can only be updated while
it is at the surface and within Wi-Fi range of the Pi**. The agent may never push
to the glider without an explicit "YES" from you on Telegram.

## Status

| Component | State |
|---|---|
| Project repo, docs, design | ✅ Done (this repo) |
| iPhone | ▶ **In progress**: see [docs/02-iphone-setup.md](docs/02-iphone-setup.md) |
| Raspberry Pi base setup | Next: [docs/03-raspberry-pi-setup.md](docs/03-raspberry-pi-setup.md) |
| Manual OTA from the Pi (no AI) | To do |
| Firmware hardening (rollback, security) | To do: [docs/04-firmware-hardening.md](docs/04-firmware-hardening.md) |
| AI build/test loop on bench twin | To do |
| Telegram agent + approval gate | To do |
| Siri voice bridge (hands-free) | To do (design in [docs/02-iphone-setup.md](docs/02-iphone-setup.md) §4C) |
| Field operations | To do: [docs/05-field-operations.md](docs/05-field-operations.md) |

The full checklist is in [ROADMAP.md](ROADMAP.md).

## Where we left off

- **Decisions so far**
  - Hands-free voice is the primary input: *"Hey Siri, glider"* → Shortcut → Pi voice
    bridge over Tailscale. Telegram is the log, the diff viewer and the backup input.
  - Pushing to the glider needs a spoken per-build challenge ("approve 1 4 3 tango")
    or YES on Telegram.
  - IOS-Keyboard is not used. tuba-firmware and Embedded-AI-Harness are submodules;
    fork Embedded-AI-Harness when we first change it.
- **Constraints discovered**
  - The pilot's iPhone is on **iOS 16.7** (the latest it supports): no Termius (needs iOS 17),
    and the wake phrase must be "Hey Siri". Emergency SSH goes through a `ttyd` browser terminal,
    the Tailscale web SSH console, or a-Shell.
  - tuba-firmware OTA is **permanent (no rollback)**, uses the **default MCUboot signing
    key**, and has open Wi-Fi + unauthenticated telnet. Fix before any field OTA.
- **Next steps**
  1. Finish the iPhone checklist (Telegram hardening, BotFather bot + user ID,
     "Hey Siri", Tailscale). Shortcuts are built later, once the voice bridge exists.
  2. Get the Pi hardware (Pi 5, second USB Wi-Fi adapter + antenna, spare ESP32 DevKitC twin).
  3. On the Pi: clone this repo and follow `docs/03-raspberry-pi-setup.md`.

## Getting started on the Raspberry Pi

```bash
git clone --recurse-submodules <this-repo-url> in-situ-firmware-programmer
cd in-situ-firmware-programmer
# if you already cloned without submodules:
git submodule update --init --recursive
```

Then follow [docs/03-raspberry-pi-setup.md](docs/03-raspberry-pi-setup.md).
If you use Claude Code on the Pi, [CLAUDE.md](CLAUDE.md) gives the agent
the project context.

## Repository layout

```
README.md                  this file
ROADMAP.md                 phased checklist
CLAUDE.md                  context for an AI agent working in this repo
.env.example               secrets/config template (copy to .env on the Pi, never commit)
docs/
  01-architecture.md       components, data flow, design decisions
  02-iphone-setup.md       apps and settings on the iPhone
  03-raspberry-pi-setup.md Pi hardware, OS, toolchain, agent
  04-firmware-hardening.md changes needed in tuba-firmware before field OTA
  05-field-operations.md   safety rules, procedure, rollback
  upstream-repos.md        all referenced repos, fork strategy
firmware/tuba-firmware/    git submodule – the glider firmware
third_party/Embedded-AI-Harness/  git submodule – Pi test bench + AI closed-loop skills
```

Planned directories (created in later phases): `pi/` (Pi setup scripts),
`ota/` (scripted OTA push), `agent/` (agent tools and skills), `tests/`.

## Referenced projects

| Project | Role | How it's included |
|---|---|---|
| [e-abdi/tuba-firmware](https://github.com/e-abdi/tuba-firmware) | Glider firmware (ESP32, Zephyr 4.3, MCUboot OTA) | submodule |
| [SensorsIot/Embedded-AI-Harness](https://github.com/SensorsIot/Embedded-AI-Harness) | Pi test bench: build → flash → test loop on real hardware | submodule |
| [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) | Agent with a Telegram gateway (option A) | installed on the Pi |
| [openclaw/openclaw](https://github.com/openclaw/openclaw) | Agent with a Telegram gateway (option B) | installed on the Pi |
| [SensorsIot/IOS-Keyboard](https://github.com/SensorsIot/IOS-Keyboard) | Considered, **not used** (see below) | reference only |
| [Zephyr RTOS](https://github.com/zephyrproject-rtos/zephyr), [MCUboot](https://github.com/mcu-tools/mcuboot) | Firmware OS and bootloader | pulled by `west` |
| [whisper.cpp](https://github.com/ggml-org/whisper.cpp) / [faster-whisper](https://github.com/SYSTRAN/faster-whisper) | Speech-to-text for Telegram voice messages | installed on the Pi |

**Why IOS-Keyboard isn't used:** it sends iPhone speech over BLE to an
ESP32-S3 that acts as a USB keyboard for a computer. Here the text goes to
Telegram, and both iOS dictation and Telegram voice messages already cover
that. See [docs/upstream-repos.md](docs/upstream-repos.md) for the fork strategy.

## License

TBD. Note that tuba-firmware is CERN-OHL-W v2; check compatibility before choosing.

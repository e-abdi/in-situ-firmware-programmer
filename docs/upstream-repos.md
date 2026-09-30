# Upstream repositories and fork strategy

| Repo | Include as | Fork? | Reason |
|---|---|---|---|
| [e-abdi/tuba-firmware](https://github.com/e-abdi/tuba-firmware) | submodule `firmware/tuba-firmware` | **No.** It's your own repo | The agent works on branches (`agent/<topic>`); hardening changes are merged there. The submodule pin records exactly which firmware this tool was tested with. |
| [SensorsIot/Embedded-AI-Harness](https://github.com/SensorsIot/Embedded-AI-Harness) | submodule `third_party/Embedded-AI-Harness` | **Fork when you first need to change it** | Its skills/test bench will need tuba-specific changes (Zephyr instead of ESP-IDF/PlatformIO, glider test plans). Forking protects you from upstream changes and lets you commit your changes. |
| [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) | installed on the Pi | No | Use the official installer and pin the version you validated. Our tools live in `agent/`, not in a fork. |
| [openclaw/openclaw](https://github.com/openclaw/openclaw) | installed on the Pi | No | Same as above; choose one of Hermes/OpenClaw. |
| [SensorsIot/IOS-Keyboard](https://github.com/SensorsIot/IOS-Keyboard) | reference only | No | Not needed: iOS dictation / Telegram voice + Whisper covers voice input. |

## Switching a submodule to your fork

After forking on GitHub (e.g. `e-abdi/Embedded-AI-Harness`):

```bash
git submodule set-url third_party/Embedded-AI-Harness https://github.com/e-abdi/Embedded-AI-Harness.git
git submodule sync
cd third_party/Embedded-AI-Harness
git remote add upstream https://github.com/SensorsIot/Embedded-AI-Harness.git
git fetch origin && git checkout main
cd ../.. && git add .gitmodules third_party/Embedded-AI-Harness && git commit -m "Use fork of Embedded-AI-Harness"
```

## Updating a submodule pin

```bash
cd firmware/tuba-firmware && git fetch && git checkout <commit-or-tag> && cd ../..
git add firmware/tuba-firmware && git commit -m "Bump tuba-firmware to <ver>"
```

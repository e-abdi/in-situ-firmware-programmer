# iPhone setup

The iPhone is only the **pilot's console**: you speak or type a request, read
the agent's plan/diff/test results, and approve or reject. Nothing is built or
flashed on the phone.

You need **no custom app**. Everything below uses App Store apps and iOS settings.

## 1. Apps to install

| App | Required? | Purpose |
|---|---|---|
| **Telegram** | Required | Chat with the glider bot (text, dictation, voice messages) |
| **Tailscale** | Strongly recommended | Private VPN to reach the Pi from anywhere without opening ports |
| **Termius** (or Blink Shell) | Recommended | SSH into the Pi over Tailscale: emergency access if the bot/agent is down |
| **GitHub** | Optional | Review the agent's branches/diffs/commits on a bigger view |

## 2. Telegram account hardening

This Telegram account will be able to reflash a deployed vehicle, so treat it
like a root password.

1. Settings → Privacy and Security → **Two-Step Verification** → turn on and set a password.
2. Settings → Privacy and Security → **Passcode Lock** → turn on (Face ID OK).
3. Settings → Devices → end any sessions you don't recognize.

## 3. Create the bot (can be done entirely on the iPhone)

1. In Telegram, open a chat with **@BotFather** (check for the blue verified tick).
2. Send `/newbot`, choose a display name (e.g. `Tuba Glider`) and a username ending
   in `bot` (e.g. `tuba_glider_<yourname>_bot`).
3. BotFather replies with an **HTTP API token** like `123456789:AA...`.
   **Save it somewhere safe** (e.g. iOS Passwords app / your password manager).
   It goes into `.env` on the Pi as `TELEGRAM_BOT_TOKEN`. Anyone who has the token controls the bot.
4. Recommended BotFather settings:
   - `/setjoingroups` → **Disable** (bot cannot be added to groups)
   - `/setprivacy` → Enable
   - `/setdescription`, `/setuserpic`: cosmetic
5. Get **your numeric Telegram user ID**: message **@userinfobot** (or @RawDataBot)
   and note the `Id` number. It goes into `.env` as `TELEGRAM_ALLOWED_USER_ID`.
   The agent will ignore everyone else.

Don't message the bot yet. Nothing is listening until the Pi is running.

## 4. Voice input: pick one (you can use both)

### Option A: iOS dictation into Telegram (simplest)
- Settings → General → Keyboard → **Enable Dictation** = on.
- Settings → General → Keyboard → Dictation Languages → make sure your language is on.
- In the bot chat, tap the **mic key on the keyboard** (not Telegram's mic button),
  speak, check the text, send. On-device dictation works offline for supported languages.

### Option B: Telegram voice messages (recommended for field use)
- Hold Telegram's **mic button** and speak. The Pi transcribes it (Whisper)
  and **echoes back the transcription and its interpretation** before doing anything.
- Better with gloves on a boat, and gives you a record of the exact request.
- Settings → Privacy → Microphone → Telegram = on.

### Optional: Siri, hands-free
- iOS Shortcuts + Telegram's Siri integration can send a message to a chat
  ("Hey Siri, send a Telegram to Tuba Glider…"). Support for sending to *bots*
  varies between iOS/Telegram versions; test it, and don't rely on it.

## 5. Tailscale (remote access to the Pi)

1. Install Tailscale, sign in (GitHub/Google/Apple account). Use the **same
   Tailscale account** later on the Pi.
2. Leave it off until the Pi joins the tailnet; then you can `ssh pi@<pi-name>`
   from Termius even when the Pi is behind a 4G modem's NAT.

## 6. Termius

1. Install, create a new host later with the Pi's Tailscale name/IP.
2. Generate an SSH key in Termius (Keychain → Generate key, ed25519) and later
   add its public key to `~/.ssh/authorized_keys` on the Pi.

## 7. Field considerations

- **iPhone as the Pi's internet uplink:** if the Pi has no 4G modem, iPhone
  **Personal Hotspot** can give the Pi internet on the boat. In that case keep the
  phone charged and turn on Settings → Personal Hotspot → *Maximize Compatibility*
  (2.4 GHz works better with Pi Wi-Fi). The Pi needs a *separate* Wi-Fi adapter
  for the glider link.
- Carry a power bank. Turn off Low Power Mode while waiting for a surfacing
  (it can delay notifications).
- Telegram → Notifications → make sure bot chat notifications are on, so you see
  "glider surfaced, ready to push?" prompts.

## Checklist: iPhone ready when…

- [ ] Telegram installed, 2-step verification + passcode on
- [ ] Bot created; **token** and **your user ID** saved securely
- [ ] Bot group-joining disabled
- [ ] Dictation enabled and/or Telegram mic permission granted
- [ ] Tailscale installed and signed in
- [ ] Termius installed, SSH key generated
- [ ] (Optional) GitHub app signed in

Next: [03-raspberry-pi-setup.md](03-raspberry-pi-setup.md)

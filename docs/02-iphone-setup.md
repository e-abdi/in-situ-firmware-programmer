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
| SSH client (see §6) | Recommended | Emergency access to the Pi over Tailscale if the bot/agent is down |
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

## 4. Voice input

A and B need the screen; **C is hands-free** and is the main mode. Keep A/B as backups.

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

### Option C: Siri, hands-free (primary mode)

Goal: operate without touching the phone. Siri **cannot send messages *to* a
Telegram bot** reliably, and a bot never receives messages it sent itself. So the
hands-free path goes **Siri → Shortcuts → HTTPS request to the Pi over Tailscale**.
The Pi then posts the request and all replies into the Telegram chat, so Telegram
stays the single log and the place to review diffs when you *can* look at the screen.

```
"Hey Siri, glider"  →  Siri: "What should I change?"  →  you speak
   → Shortcut POSTs text to http://tuba-pi:8088/voice   (Tailscale only, bearer token)
   → Pi: posts "🎙 via Siri: …" to Telegram, starts the agent
   → Siri speaks the Pi's short reply: "Understood: pump timeout 120 s. Building, I'll report."
```

#### iOS settings (iOS 16.7)
- Settings → Siri & Search → **Listen for "Hey Siri"** = on (train your voice;
  plain "Siri" without "Hey" needs iOS 17).
- Settings → Siri & Search → **Allow Siri When Locked** = on.
- Settings → Siri & Search → **Siri Responses** → *Always* speak responses.
- **AirPods or a Bluetooth headset** strongly recommended on a noisy boat
  (AirPods 2nd gen+/Pro support "Hey Siri").
- Tailscale must be connected all the time: Tailscale app → settings → turn on
  **VPN On Demand** (or leave the VPN toggled on).
- Some Shortcut actions require unlocking a locked phone. Test with the phone locked;
  if it asks for Face ID/passcode, consider Settings → Face ID & Passcode →
  Auto-Lock timing during operations, or use "Require attention" off.

#### Shortcuts to create (Shortcuts app; built in on iOS 16)
Create these once the Pi's voice bridge is running (Pi phase). The shortcut
**name is the Siri phrase**.

| Shortcut name ("Hey Siri, …") | Actions |
|---|---|
| **Glider** | *Dictate Text* (stop listening: After Pause) → *Get Contents of URL* `http://tuba-pi:8088/voice`, Method POST, Header `Authorization: Bearer <VOICE_BRIDGE_TOKEN>`, JSON body `{"text": <Dictated Text>}` → *Get Dictionary Value* `say` → *Speak Text* |
| **Glider status** | *Get Contents of URL* `http://tuba-pi:8088/status` (same header) → *Get Dictionary Value* `say` → *Speak Text* |
| **Glider approve** | *Dictate Text* → POST to `/approve` with `{"text": …}` → *Speak Text* the reply |
| **Glider cancel** | POST to `/cancel` → *Speak Text* the reply |

#### Hands-free approval: safety design
Voice approval to reflash a vehicle has to be harder to trigger by accident than a tap:
- The Pi reads out a **one-time challenge**, e.g. *"Version 1.4.3 passed 11 of 11 tests.
  To push, say: approve 1 4 3 tango."* ("tango" is a random word chosen per build).
- "Glider approve" must repeat both the version and the word; anything else = rejected.
- Approval expires (e.g. 30 min) and is valid for that exact git hash only.
- Everything is mirrored to Telegram; a Telegram `/cancel` always wins.

#### Hearing replies without touching the phone
- Each shortcut speaks the Pi's short `say` text.
- Progress updates ("build done", "glider surfaced, pushing", "confirmed") arrive as
  Telegram notifications. With AirPods, Settings → Notifications → **Announce
  Notifications** can read them aloud, if Telegram supports it on your iOS/Telegram version
  (test it). Otherwise just ask "Hey Siri, glider status".

## 5. Tailscale (remote access to the Pi)

1. Install Tailscale, sign in (GitHub/Google/Apple account). Use the **same
   Tailscale account** later on the Pi.
2. Keep it connected (VPN On Demand), because the Siri shortcuts need it. Once the Pi joins the tailnet, you can `ssh pi@<pi-name>`
   from Safari (browser terminal), a-Shell, or another SSH app even when the Pi is behind a 4G modem's NAT.

## 6. Emergency SSH access to the Pi

Termius needs iOS 17+. This project assumes the iPhone may be stuck on
**iOS 16.7**, so use one of these instead (check "Compatibility" on each App Store page):

1. **Browser terminal, no app needed (recommended).** The Pi runs `ttyd`
   (a web terminal), reachable only over Tailscale, e.g. `http://tuba-pi:7681`
   in Safari. Protect it with a login (`ttyd -c user:pass`) and bind it to the
   Tailscale interface only. Set up in the Pi phase.
2. **Tailscale SSH console:** from Safari, log in to the Tailscale admin console
   (login.tailscale.com) → Machines → `tuba-pi` → **SSH**. Needs `tailscale up --ssh`
   on the Pi. Also no app.
3. **An SSH app that still supports iOS 16**, e.g. **a-Shell** (free, has `ssh`),
   Blink Shell, or Prompt 3. Check the minimum iOS version before buying.
4. **Older Termius build:** if Termius was ever downloaded with your Apple ID
   (e.g. on a newer device: "Get" it there), the old iPhone's App Store offers to
   *"download the last compatible version"* from Account → Purchased.

The Telegram bot will also get a few safe maintenance commands (`/status`,
`/restart_agent`, `/logs`), so SSH is only needed when the agent itself is broken.

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
- [ ] "Hey Siri" trained, Allow Siri When Locked on, Siri always speaks responses
- [ ] (Recommended) AirPods/Bluetooth headset paired
- [ ] Siri shortcuts created (after the Pi voice bridge exists)
- [ ] Tailscale installed and signed in
- [ ] Emergency SSH path chosen (browser terminal / Tailscale console / a-Shell)
- [ ] (Optional) GitHub app signed in

Next: [03-raspberry-pi-setup.md](03-raspberry-pi-setup.md)

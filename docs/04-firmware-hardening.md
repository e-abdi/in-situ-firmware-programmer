# Firmware hardening required before field OTA

Findings from reviewing `firmware/tuba-firmware` at commit `f9f035f`.
These changes belong in the tuba-firmware repo (make them on a branch there,
then bump the submodule here).

## What already exists

- MCUboot built via sysbuild (`sysbuild.conf`: `SB_CONFIG_BOOTLOADER_MCUBOOT=y`;
  `prj.conf`: `CONFIG_BOOTLOADER_MCUBOOT`, `CONFIG_MCUBOOT_IMG_MANAGER`).
- OTA over HTTP: console menu **5 → type a URL** (`src/ui_menu.c`, `ST_OTA_MENU`).
  `src/ota_simple.c` downloads into `slot1_partition`, checks the MCUboot header,
  then calls `boot_request_upgrade()` and reboots.
- Wi-Fi AP `Tuba-Glider`, telnet console on `192.168.4.1:23` (`src/main.c`).
- Helper scripts in the repo root: `ota_server.py`, `ota_update.sh` (manual, interactive).

## Gaps (highest risk first)

### 1. No rollback: upgrades are permanent 🔴
`ota_simple_reboot()` calls `boot_request_upgrade(1)`. `1` is
`BOOT_UPGRADE_PERMANENT`, and nothing in `src/` calls
`boot_write_img_confirmed()`. A bad image that boots but misbehaves (hangs the
state machine, breaks Wi-Fi, breaks the pump) **cannot be reverted remotely**.
If Wi-Fi breaks, the glider is unreachable until recovered.

**Fix:**
- Call `boot_request_upgrade(BOOT_UPGRADE_TEST)` instead.
- Make sure MCUboot runs in swap mode with revert (not overwrite-only) and has
  enough scratch/slot space.
- On boot, if `!boot_is_img_confirmed()`, run a **post-boot self-test**: sensors
  answer on I2C, depth reading sane, pump and trim encoders respond, Wi-Fi AP up.
  Then call `boot_write_img_confirmed()`, but only after the Pi has reconnected
  over telnet and sent a `confirm` command. That proves the OTA link still works.
- Enable a hardware watchdog so a hang causes a reset, and so a revert to the
  previous image.
- Test the revert path on the bench: deliberately push an image that never
  confirms and check that the old one comes back.

### 2. Default (public) signing key 🔴
No custom key is configured (`SB_CONFIG_BOOT_SIGNATURE_KEY_FILE` isn't set), so
images are signed with MCUboot's **public dev key**. Anyone can sign an image
your glider will accept.

**Fix:** generate your own key on the Pi (`imgtool keygen -k tuba-ecdsa-p256.pem -t ecdsa-p256`),
set `SB_CONFIG_BOOT_SIGNATURE_TYPE_ECDSA_P256=y` and
`SB_CONFIG_BOOT_SIGNATURE_KEY_FILE="<path>"`, keep the key **out of git**
(it's in `.gitignore`), and back it up offline. Flash the new bootloader over USB once.

### 3. Open Wi-Fi + unauthenticated telnet 🟠
`WIFI_SECURITY_TYPE_NONE` and a telnet console with full control, including OTA
and motor commands. Anyone within range can operate the glider.

**Fix:** WPA2-PSK with a strong key (from Kconfig/NVS, not hard-coded in a
public repo), plus a console password/token before menu access. Consider only
accepting OTA URLs from the Pi's IP.

### 4. OTA allowed in any state 🟠
The OTA menu can be entered whenever the console is reachable.

**Fix:** reject OTA unless the glider is surfaced/idle, not mid-mission, battery
above a threshold, and motors stopped. Also make sure OTA can't interrupt a
recovery.

### 5. Machine-readable status 🟡
The Pi needs to parse the state reliably. Add a non-interactive command (or
menu item) that prints one line, e.g.
`$STATUS,ver=1.4.3,git=abc123,slot=0,confirmed=1,state=SURFACE,batt=14.8,depth=0.2`,
plus non-interactive `ota <url>` and `confirm` commands so the Pi doesn't
have to drive menus by keystrokes.

### 6. Transfer robustness 🟡
Check the download handles a dropped link mid-transfer (erase-on-start is fine;
it must never request an upgrade on a partial image). Measure the transfer time
at realistic range, since it has to fit inside a single surfacing.

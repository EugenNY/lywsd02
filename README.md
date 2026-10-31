# LYWSD02 Clock Sync — set the time on a Xiaomi E-Ink clock from your browser

A single-page web app that sets the time and time zone on the **Xiaomi Mijia LYWSD02** E-Ink temperature/humidity clock over **Web Bluetooth**. No Mi Home app, no Xiaomi account, no cloud, nothing to install. Open the page, tap the button, pick the clock, and you're done.

**▶ Open it:** https://eugenny.github.io/lywsd02/ (GitHub Pages; the UI is in German)

## Why

The LYWSD02 has no radio-controlled clock and no buttons for setting the time. Normally it only syncs through the Mi Home app, which requires an account, pairing and a cloud connection. After a battery change or a DST switch it shows the wrong time until something syncs it again. This page does that sync in one click. It uses the current time and UTC offset of the phone or computer you open it on.

## Supported devices

| Device | Status |
|---|---|
| Xiaomi Mijia **LYWSD02** / **LYWSD02MMC**: "Xiaomi Temperature and Humidity Monitor Clock" (rectangular E-Ink clock, 2× CR2032, made by Miaomiaoce) | ✅ supported |
| LYWSD03MMC, MHO-C401, CGG1 and other Xiaomi thermometers without a clock | ❌ no clock to set |

The page lists every BLE device whose advertised name starts with `LYWSD02`.

### Chipset / hardware revisions

Xiaomi's spec sheet lists Bluetooth Low Energy 5.0 and 2× CR2032 cells. Two SoCs have shipped:

| Revision | SoC | Notes |
|---|---|---|
| Earlier units | Dialog / Renesas **DA14585** (BLE, Arm Cortex-M0) | Uses the GATT time protocol below. |
| Newer units (~2023/24 onward) | Telink **TLSR8250** (TLSR8250F512ET32, 512 KB flash) | Reported by the community. Not yet verified with this page. |

You can't tell the revision from the outside. Both are expected to use the same time characteristic.

## Browser support

- ✅ Chrome / Edge / Opera on Android, macOS, Windows, ChromeOS and Linux (on Linux you may need `chrome://flags/#enable-web-bluetooth`)
- ✅ iPhone / iPad via the [Bluefy](https://apps.apple.com/app/bluefy-web-ble-browser/id1492822055) browser app (Safari has no Web Bluetooth)
- ❌ Firefox, Safari

Web Bluetooth needs a secure context: serve the page over HTTPS (e.g. GitHub Pages) or from `localhost`.

## How it works

The page connects to the clock's vendor GATT service and writes 5 bytes to the time characteristic:

- Service: `ebe0ccb0-7a0a-4b0c-8a1a-6ff2997da3a6`
- Time characteristic: `ebe0ccb7-7a0a-4b0c-8a1a-6ff2997da3a6`

| Bytes | Meaning |
|---|---|
| 0–3 | Unix timestamp (seconds, UTC), `uint32` little-endian |
| 4 | UTC offset in whole hours, `int8` (e.g. `+1` CET, `+2` CEST) |

Other characteristics on the same service, which this page doesn't use:

| UUID suffix | Purpose |
|---|---|
| `…ccc1` | Current temperature (`int16` / 100 °C) + humidity (`uint8` %), notify |
| `…ccc4` | Battery level (%) |
| `…ccbe` | Display unit (`0x01` = °F, `0xff` = °C) |
| `…ccbc` / `…ccb9` / `…ccba` | Hourly min/max history records, count, index |

The whole app is the plain HTML + JavaScript in [`index.html`](index.html). There is no build step and no dependencies.

## Limitations

- **Daylight saving time:** the clock stores a fixed offset and doesn't switch DST by itself. Run the page again after the clocks change.
- **Half-hour time zones** (e.g. India UTC+5:30) can't be represented. The offset is truncated to whole hours.
- **Device not found?** Close Mi Home or any other app that holds a connection to the clock, and move closer to it.

## Credits

The protocol details come from the [h4/lywsd02](https://github.com/h4/lywsd02) Python library. Hardware notes come from [pvvx/ATC_MiThermometer#213](https://github.com/pvvx/ATC_MiThermometer/issues/213), [pvvx/BZdevice#5](https://github.com/pvvx/BZdevice/issues/5) and [Gadgetbridge](https://gadgetbridge.org/gadgets/clocks/xiaomi/).

# Device Notes — Zigbee Coordinator — SMLIGHT SLZB-06M

---

## 1. Device Identity

| Field | Value |
|---|---|
| Manufacturer | SMLIGHT |
| Model | SLZB-06M |
| MAC Address | 68:25:DD:47:C9:8C |
| mDNS | SLZB-06M.local |
| IP Address | 192.168.200.232 (DHCP — assign static reservation) |
| Network | Fioptics24719 (WiFi client mode) |
| Web UI | http://192.168.200.232/ |

### Firmware Versions

| Component | Chip | Before | After | Date Updated |
|---|---|---|---|---|
| Core (ESP32) | ESP32-S3 (2× 240MHz, 16MB flash, 520KB RAM) | v2.9.3 | v3.2.0 | 2026-03-23 |
| Zigbee Radio | EFR32MG21 (1× 80MHz, 768KB flash, 96KB RAM) | 20231030 | 20250220 (SDK 8.0.2) | 2026-03-23 |

---

## 2. Settings & Rationale

| Setting | Value | Reason |
|---|---|---|
| Connection mode | WiFi client | Ethernet not yet cabled to this location |
| Zigbee role | Coordinator | Required for ZHA |
| HA Integration | ZHA (not Zigbee2MQTT) | More stable with SLZB-06M; simpler config for beginner setup |
| ZHA socket | `socket://192.168.200.232:6638` | Network coordinator address; port 6638 is SLZB default |
| ZHA radio type | EZSP | Required for EFR32MG21 + SDK 8.0.2 firmware |
| SMLIGHT integration | Added separately | Monitors coordinator health, firmware, LED, uptime in HA |

### Why EZSP and not Zigbee2MQTT
The 20250220 firmware uses SDK 8.0.2 and requires `adapter: ember` in Zigbee2MQTT config.
ZHA handles this natively via EZSP without extra config. For a new setup with no existing
Z2M config, ZHA is the right choice and avoids the known stability issues reported with
this coordinator + Z2M (pre-firmware-fix).

### Why firmware update was critical
The factory firmware (20231030) caused Z2M to produce "Frame(s) in progress cancelled"
errors and required weekly reboots. The 20250220 firmware (SDK 8.0.2) resolves this.
The Core update (v2.9.3 → v3.2.0) was required for OTA stability improvements and to
properly support the new Zigbee firmware.

### Zigbee firmware version shows "Unknown"
After flashing 20250220 (SDK 8.0.2), the web UI and API report the Zigbee version as
"Unknown". This is **expected** — the new firmware reports its version via the EZSP
protocol rather than direct UART queries. The correct version will appear once ZHA
establishes a connection.

---

## 3. Web UI & API — How to Work With This Device

### The UI Problem
The SLZB-06M web UI is a JavaScript SPA that:
- Establishes a persistent WebSocket back to the device on load
- Requires a real browser — `curl` returns gzipped binary, plain HTTP requests return garbage
- Will not reach "networkidle" in headless browsers (WebSocket keeps connection open forever)
- Bootstrap modals don't render properly in headless Chrome via CDP

### Fastest Approach: Playwright with `wait_until="load"`

```python
import asyncio
from playwright.async_api import async_playwright

async def main():
    async with async_playwright() as p:
        browser = await p.chromium.launch(headless=True)
        page = await browser.new_page(viewport={"width": 1280, "height": 900})
        # IMPORTANT: use "load" not "networkidle" — WS connection never closes
        await page.goto("http://192.168.200.232/", wait_until="load", timeout=15000)
        await asyncio.sleep(5)  # Wait for WS data to populate the UI
        text = await page.inner_text("body")
        await page.screenshot(path="/tmp/slzb.png")
        await browser.close()

asyncio.run(main())
```

**Install if needed:** `pip3 install playwright --break-system-packages`
**Note:** Do NOT use port 9222 for Chrome remote debugging — another project uses that port. Use 9223+.

### Direct API Endpoints (much faster than UI automation)

All endpoints use `GET http://192.168.200.232/api2?action=...`

| Action | Endpoint | Returns |
|---|---|---|
| Get Core firmware version | `action=1&param=espRev` | e.g., `v3.2.0` |
| Get Zigbee firmware version | `action=1&param=zbRev` | e.g., `20250220` or `[ZB_FW_unk]` |
| Check internet state | `action=1&param=inetState` | `1` or `ok` |
| Reboot device | `action=1&param=reboot` | `ok` |
| Flash Core firmware (OTA) | `action=8&fwUrl=<url>` | `ok` — flashes in background |
| Flash Zigbee firmware (OTA) | `action=6&fwUrl=<url>` | `ok` — flashes in background |

### Firmware Update URLs (SMLIGHT OTA Server)

```bash
# List available Core (ESP) firmware versions
curl -s "https://updates.smlight.tech/services/api/slzb-06x-ota.php?type=ESP" \
  | python3 -c "import sys,json; [print(f['ver'], f.get('dev',''), f.get('link','')) for f in json.load(sys.stdin).get('fw',[])]"

# List available Zigbee firmware versions (SLZB-06M device type = key '1')
curl -s "https://updates.smlight.tech/services/api/slzb-06x-ota.php?type=ZB&format=slzb" \
  | python3 -c "
import sys, json
d = json.load(sys.stdin)
for entry in d.get('1', []):
    print(entry.get('rev'), entry.get('prod'), entry.get('link'))
"
```

**Known firmware URLs (as of 2026-03-23):**
```
Core v3.2.0:
  https://updates.smlight.tech/firmware/slzb06x/core/slzb-os-v3.2.0-ota.bin

Zigbee Coordinator 20250220 (SDK 8.0.2):
  https://updates.smlight.tech/firmware/slzb06x/zigbee/slzb06m/20250220/slzb06m_zigbee_ncp_8.0.2.0_sw_flow_115200.gbl

Zigbee Router 20250220:
  https://updates.smlight.tech/firmware/slzb06x/zigbee/slzb06m/20250220/slzb06m_zigbee_router_8.0.2.0_115200.gbl
```

### Flash Sequence (for future updates)

```bash
# 1. Check current versions
curl -s "http://192.168.200.232/api2?action=1&param=espRev"
curl -s "http://192.168.200.232/api2?action=1&param=zbRev"

# 2. Flash Core (update Core BEFORE Zigbee)
curl -s "http://192.168.200.232/api2?action=8&fwUrl=<core_url>"
# Wait ~90 seconds for download + flash + reboot

# 3. Confirm Core version
curl -s "http://192.168.200.232/api2?action=1&param=espRev"

# 4. Flash Zigbee coordinator
curl -s "http://192.168.200.232/api2?action=6&fwUrl=<zigbee_coordinator_url>"
# Wait ~90 seconds — zbRev will show [ZB_FW_unk] until ZHA connects (normal for SDK 8.0.2)
```

### Monitor reboot after flash

```bash
for i in $(seq 1 30); do
    sleep 5
    V=$(curl -s --max-time 3 "http://192.168.200.232/api2?action=1&param=espRev" 2>/dev/null)
    echo "$(date +%H:%M:%S) core='$V'"
    [ ! -z "$V" ] && break
done
```

### Navigation in Playwright (v3.2.0 UI)

```python
# Sidebar nav items (click by text):
# Dashboard, Mode, Network, 4G/LTE, Z2M and ZHA, IR Learn & Replay,
# Buzzer, Ambilight, Security, VPN, DDNS, USB, Scripts and Automations,
# Settings and Tools → General settings, Firmware update, LEDs settings,
#                      Time settings, Log and debug

# Close the "What's new?" modal that appears after Core update:
await page.evaluate("""() => {
    document.querySelectorAll('.modal').forEach(m => { m.style.display='none'; m.classList.remove('show'); });
    document.querySelectorAll('.modal-backdrop').forEach(b => b.remove());
}""")

# Navigate to a sub-page (text must match exactly):
await page.evaluate("""() => {
    Array.from(document.querySelectorAll('a, div, span'))
        .find(el => el.textContent.trim() === 'Firmware update')?.click();
}""")

# Click a button inside an accordion (regular .click() often blocked — use JS):
await page.evaluate("document.getElementById('BUTTON_ID').click()")
```

---

## 4. Open Items

- [ ] Assign static IP via DHCP reservation on router (current: 192.168.200.232)
- [ ] Add SMLIGHT integration to HA (Settings → Devices & Services → SMLIGHT)
- [ ] Add ZHA integration to HA (`socket://192.168.200.232:6638`, radio type EZSP)
- [ ] Consider switching to Ethernet once office cabling is in place (more reliable than WiFi)
- [ ] Enable "Automatic Zigbee Update" after ZHA is stable (currently Off)

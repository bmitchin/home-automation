# Equipment Inventory

Verified equipment list with Home Assistant integration status. Last updated: 2026-03-22.

**Status icons:**
- ✅ Supported — works well
- ⚠️ Supported with caveats — see notes
- ⛔ Not supported — no viable HA integration
- 🔲 Not yet set up
- 📦 Not yet arrived

---

## Quick Reference

| Device | Category | Room | Protocol | HA Integration | Status |
|---|---|---|---|---|---|
| SMLIGHT SLZB-06M | Zigbee Coordinator | — | Zigbee 3.0 / Ethernet | ZHA (recommended) | ⚠️ Firmware update needed |
| THIRDREALITY Smart Plug Gen3 ×4 | Smart Plug | TBD | Zigbee 3.0 | ZHA | 🔲 Not set up |
| THIRDREALITY Temp/Humidity ×2 | Sensor | TBD | Zigbee 3.0 | ZHA | 🔲 Not set up |
| Apollo MSR-2 mmWave | Occupancy Sensor | TBD | WiFi / ESPHome | ESPHome | 📦 Arriving Thursday |
| ESP32-S3-BOX-3B ×2 | Voice Satellite | TBD | Wyoming / WiFi | Official HA | 📦 Arriving Tuesday |
| Yamaha TSR-7810 | AV Receiver | Basement Office | YNCA / Network | yamaha_ynca (HACS) | 🔲 Not set up |
| WiiM (model TBD) | Streaming Audio | Basement Office | LinkPlay / WiFi | WiiM (HACS) | 🔲 Not set up |
| Sony STR-DN2010 | AV Receiver | 1F Family Room | Ethernet | ⛔ None | — |
| Vizio E60-C3 | TV | Basement Office | WiFi | SmartCast (official) | ⚠️ 2015 model — verify |
| Samsung UN49MU7000 | TV | 1F Family Room | WiFi / WebSocket | Samsung Smart TV | 🔲 Connected, not set up |
| TiVo Stream 4K IPA1114HDW | Streaming Device | 1F Family Room | Android TV / WiFi | Android TV Remote | 🔲 In use, not set up |
| Philips Hue Bridge | Zigbee Hub | — | Zigbee / Ethernet | Philips Hue (official) | 🔲 Not set up |
| Hue Color Ambiance Slim Downlights 5/6" | Smart Lighting | Kitchen / Family Room | Zigbee via Bridge | Philips Hue (official) | 📦 On hand, not installed |
| Hue Lightstrip Flux 10ft | Smart Lighting | Kitchen | Zigbee via Bridge | Philips Hue (official) | 📦 On hand, not installed |
| Hue bulbs ×2 | Smart Lighting | Basement Office | Zigbee via Bridge | Philips Hue (official) | ✅ In use |
| Magic Home LED strips | LED Strips | Basement Office | WiFi | flux_led (official) | ⚠️ In use, not in HA |

---

## Zigbee Infrastructure

### SMLIGHT SLZB-06M — Zigbee Coordinator

- **Manufacturer**: SMLIGHT
- **Protocol**: Zigbee 3.0 (IEEE 802.15.4); supports Thread/Matter-over-Thread via alternate firmware
- **Connectivity**: Ethernet (primary), WiFi, USB
- **HA Integration**: ZHA (Zigbee Home Automation) — recommended; Zigbee2MQTT also supported
- **Current status**: Under roof — struggling to set up

**Setup notes:**
- **Firmware update is required before connecting to HA.** Versions prior to 20250220
  produce "Frame(s) in progress cancelled" errors in Zigbee2MQTT and require weekly reboots.
  Update via the SMLIGHT web interface (navigate to device IP in browser) before adding to HA.
- **Use ZHA, not Zigbee2MQTT.** ZHA is more stable with this coordinator and simpler to
  configure. This is almost certainly the root cause of current setup difficulties.
- The SMLIGHT integration (separate from ZHA) provides coordinator health monitoring,
  firmware updates, and LED control directly in HA — install both.
- Connect via Ethernet for maximum reliability. WiFi mode is available but Ethernet is preferred.

**Action required**: Update firmware → connect via Ethernet → add to HA via ZHA.

---

## Sensors

### THIRDREALITY Smart Plug Gen3 (×4)

- **Manufacturer**: THIRDREALITY
- **Protocol**: Zigbee 3.0
- **Rating**: 15A
- **HA Integration**: ZHA (preferred) or Zigbee2MQTT
- **Current status**: Under roof, not yet set up

**Features**: Power on/off, real-time energy monitoring (±1% accuracy), power-on state
restoration, timer. Also functions as a Zigbee mesh repeater — extending coordinator range.

**Setup notes**: Pair via ZHA. Community reports more reliable pairing through ZHA than Z2M.

---

### THIRDREALITY Zigbee Temperature & Humidity Sensor (×2)

- **Manufacturer**: THIRDREALITY
- **Model**: 3RTHS24BZ (LCD display) or 3RTHS0224Z (Lite, no display — higher accuracy)
- **Protocol**: Zigbee 3.0
- **Battery**: 2× AAA, ~1 year life
- **HA Integration**: ZHA or Zigbee2MQTT — both fully supported
- **Current status**: Under roof, powered on, not set up in HA

**Specs**:
- LCD model: ±1°C temp accuracy, ±2% RH humidity, 20-second refresh
- Lite model: ±0.3°C temp accuracy, ±2% RH humidity, 5-second refresh

**Setup notes**: No major known issues. Pairs like any standard Zigbee sensor.

---

### Apollo Automation MSR-2 mmWave Multisensor (LD2410B)

- **Manufacturer**: Apollo Automation
- **Protocol**: WiFi (ESP32-C3); **not Zigbee**
- **HA Integration**: ESPHome (official Apollo Automation integration)
- **Current status**: Arriving Thursday

**Sensors included**: mmWave occupancy (still-person detection), LUX, UV, temperature,
pressure, RGB LED, piezo buzzer.

**Setup notes**:
- Connect to "Apollo MSR-2 Hotspot" WiFi on first boot, select home network, device
  auto-discovers in HA via ESPHome integration.
- **Temperature readings require a +3–5°C offset** — the ESP32-C3 generates heat in
  constant WiFi mode which affects the onboard temp sensor. Not recommended as a primary
  temperature source; use the THIRDREALITY sensors for that.
- Excellent for occupancy: the mmWave chip detects stationary people (sitting at desk,
  watching TV) that PIR sensors miss. This is the right sensor for "keep lights on while
  someone is in the room."
- For mesh WiFi networks: manually enter SSID during setup rather than scanning.

---

## Voice Satellites

### ESP32-S3-BOX-3B (×2)

- **Manufacturer**: Espressif
- **Protocol**: Wyoming protocol over WiFi
- **HA Integration**: Official Home Assistant voice satellite
- **Current status**: Arriving Tuesday

**Setup notes**:
- Pre-built firmware available — flash directly from the HA voice assistant web installer
  at `https://voice.home-assistant.io`. No ESPHome build required.
- Supports microWakeWord for fully on-device wake word detection (no cloud).
- Will integrate with the local Wyoming/Whisper pipeline on the HA server for offline
  voice command processing.
- Touchscreen can display custom information (clock, active media, etc.).

---

## Audio

### Yamaha TSR-7810 — Basement Office Receiver

- **Manufacturer**: Yamaha
- **Type**: 7.2-channel AV receiver, Dolby Atmos / DTS:X, 4K
- **Protocol**: YNCA over network (Ethernet)
- **HA Integration**: `yamaha_ynca` custom integration (HACS) — recommended over official Yamaha integration
- **MusicCast**: Yes — supports Yamaha MusicCast for whole-home audio
- **Current status**: Under roof, connected to network, not set up in HA

**Setup notes**:
- Install `yamaha_ynca` via HACS — provides richer control than the built-in Yamaha integration.
- Supports: power, volume, mute, source selection, playback controls.
- MusicCast integration enables multi-room audio coordination with other MusicCast devices.
- YNCA protocol only accepts one connection at a time — HA will hold the connection.

---

### WiiM Streamer (model TBD)

- **Manufacturer**: WiiM (Linkplay Technology)
- **Protocol**: LinkPlay over WiFi
- **HA Integration**: WiiM integration via HACS (official core integration pending as of 2026)
- **Current status**: Qty 1 for testing, more planned as whole-home audio expands

**Setup notes**:
- Install WiiM integration via HACS. Requires HA 2024.12.0+.
- Auto-discovered on the network.
- Supports: playback control, volume, source selection, grouped/synchronized playback.
- Confirm WiiM model number — Mini, Pro, and Amp all use LinkPlay and are supported.

**Open question**: Confirm exact model (WiiM Mini / Pro / Amp / Ultra).

---

### Sony STR-DN2010 — 1F Family Room Receiver

- **Manufacturer**: Sony
- **Type**: 7.1-channel AV receiver
- **Year**: 2010
- **Protocol**: Ethernet / DLNA (streaming only, no control API)
- **HA Integration**: ⛔ None. Sony Songpal integration supports 2014+ models only.
- **Current status**: In use, family room

**Options until replaced**:
- IR blaster (e.g., Broadlink RM4 Pro) for basic power/volume/source control via IR
- Harmony Hub (if already owned) for consolidated remote + HA bridge
- Manual remote only — acceptable since family room is Phase 2 and receiver replacement
  is already planned

**Note**: Replacement with an IP-controllable receiver is already in the project plan.
This device's lack of HA integration does not block Phase 1 work.

---

## Displays & Media

### Vizio E60-C3 — Basement Office TV

- **Manufacturer**: Vizio
- **Type**: 60" 1080p Full HD Smart TV
- **Year**: 2015
- **Protocol**: WiFi (802.11n)
- **HA Integration**: VIZIO SmartCast (official) — **but SmartCast requires 2016+ models**
- **Current status**: In use, basement office

**Likely incompatible** with the HA SmartCast integration. Verify at
`support.vizio.com` — search E60-C3 and confirm SmartCast support.

If not compatible: treat as display-only for now. Basic power/input control may be
achievable via IR blaster if needed.

---

### Samsung UN49MU7000 — 1F Family Room TV

- **Manufacturer**: Samsung
- **Type**: 49" 4K UHD Smart TV
- **Year**: 2016
- **Protocol**: WiFi / WebSocket
- **HA Integration**: Samsung Smart TV (official) — fully supported
- **Current status**: Connected via WiFi, not set up in HA

**Setup notes**:
- HA and TV must be on the **same subnet**. Will not work across VLANs.
- Assign a static IP to the TV (via router DHCP reservation) to prevent connection drops.
- Set TV access permission to "First time only" (Settings > General > External Device Manager)
  to avoid per-session approval prompts.
- Supports: power, volume, source, media state.

---

### TiVo Stream 4K — 1F Family Room (Model IPA1114HDW)

- **Manufacturer**: TiVo / eStream
- **Type**: Android TV streaming media player — **not a traditional cable box**
- **Protocol**: Android TV / WiFi
- **HA Integration**: Android TV Remote (official) — fully supported
- **Current status**: In use, family room

**Note**: This device aggregates streaming services (Netflix, Hulu, etc.) and live TV
apps. It does not decode a cable signal or control a cable subscription directly. It runs
Android TV, which is what HA integrates with.

**Setup notes**: Auto-discovered via Android TV Remote integration. Supports media
controls, app launching, volume. Requires Android TV Remote Service (pre-installed).

---

## Lighting

### Philips Hue Color Ambiance Slim Downlights 5/6" + Hue Bridge

- **Manufacturer**: Philips Hue
- **Protocol**: Zigbee (via Hue Bridge) — bridge connects via Ethernet
- **HA Integration**: Philips Hue (official) — excellent
- **Current status**: On hand, not yet installed (kitchen/family room — post-remodel)

**Setup notes**:
- Hue Bridge connects to router via Ethernet. Add the Philips Hue integration in HA —
  press the button on the bridge when prompted.
- The Hue Bridge runs a separate Zigbee network from the SLZB-06M. Hue devices pair
  directly to the bridge, not to ZHA.
- Supports full color, brightness, scenes, and group control.
- Up to 50 bulbs per bridge.

---

### Philips Hue Lightstrip Flux 10ft

- **Manufacturer**: Philips Hue
- **Protocol**: Zigbee via Hue Bridge
- **Color temperature**: 1,000K–20,000K (very warm to very cool daylight)
- **Brightness**: 4,800–6,000 lumens
- **HA Integration**: Philips Hue (official) — same integration as other Hue devices
- **Current status**: On hand, not yet installed (kitchen under-counter — post-remodel)

---

### Philips Hue Bulbs ×2 — Basement Office Overhead

- **Manufacturer**: Philips Hue
- **Protocol**: Zigbee via Hue Bridge
- **HA Integration**: Philips Hue (official)
- **Current status**: ✅ In use — Phase 1 starting point

---

### Magic Home WiFi LED Strips — Basement Office

- **Manufacturer**: Magic Home (exact model unknown)
- **Protocol**: WiFi (proprietary Magic Home protocol)
- **HA Integration**: `flux_led` (official HA integration) — no firmware flash required
- **Current status**: In use (over cabinets + behind TV), not yet in HA

**Setup notes**:
- Find the device IP address (check router DHCP table or Magic Home app).
- Add via Settings > Integrations > FLUX LED/MagicLight in HA — enter IP address.
- Supports: on/off, color, brightness, effects.
- Assign a static IP (DHCP reservation) to prevent the IP changing after a router restart.
- **Longer-term**: consider replacing with Zigbee/Hue-compatible LED controllers to
  consolidate onto one ecosystem and eliminate WiFi dependency.

**Open question**: Confirm exact Magic Home model number (check device label or app).

---

## Issues & Decisions Log

### 1. SLZB-06M Firmware — Action Required Now
**Problem**: Firmware below 20250220 causes Zigbee coordinator instability.
This is almost certainly the cause of current setup difficulties.
**Resolution**: Update firmware via SMLIGHT web UI → use ZHA (not Zigbee2MQTT).

### 2. Sony STR-DN2010 — No HA Integration
**Problem**: 2010-era receiver, no supported control API.
**Resolution**: Use manual remote until replaced. IR blaster optional interim solution.
Receiver replacement is already in the Phase 2 plan.

### 3. Vizio E60-C3 — Likely Incompatible
**Problem**: SmartCast integration requires 2016+ models; E60-C3 is 2015.
**Resolution**: Verify with Vizio support. If incompatible, treat as display-only.
IR blaster available as fallback for basic control if needed.

### 4. TiVo IPA1114HDW — Clarification
**Note**: This is an Android TV streaming media player, not a cable box. It integrates
well with HA via Android TV Remote.

---

## Open Questions

- [ ] WiiM exact model (Mini / Pro / Amp / Ultra)
- [ ] Magic Home LED strip exact model (check device label)
- [ ] Vizio E60-C3 SmartCast compatibility — verify with Vizio support
- [ ] Hue Bridge — confirm generation (v2 square or v3)
- [ ] Yamaha TSR-7810 — confirm it is connected to the network (Ethernet recommended)
- [ ] SLZB-06M current firmware version — check via SMLIGHT web UI

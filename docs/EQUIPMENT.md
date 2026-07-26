# Equipment Inventory

Verified equipment list with Home Assistant integration status. Last updated: 2026-07-26.

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
| SMLIGHT SLZB-06M | Zigbee Coordinator | — | Zigbee 3.0 / WiFi | ZHA | ✅ Core v3.2.4, Zigbee 20250220 |
| THIRDREALITY Smart Plug Gen3 ×4 | Smart Plug | Basement Office | Zigbee 3.0 | ZHA | ✅ 3 paired (cabinet underlights); 4th spare |
| THIRDREALITY Temp/Humidity ×2 | Sensor | Office + Kitchen | Zigbee 3.0 | ZHA | ✅ Set up |
| Apollo MSR-2 mmWave | Occupancy Sensor | Basement Office | WiFi / ESPHome | ESPHome | ✅ Set up — drives occupancy automations |
| ESP32-S3-BOX-3B ×2 | Voice Satellite | Office + Kitchen | ESPHome / WiFi | Official HA voice | ✅ Both set up, firmware 26.6.1 |
| Yamaha TSR-7810 | AV Receiver | Basement Office | YNCA / Network | yamaha_ynca (HACS) | ✅ Set up — 192.168.200.41 |
| WiiM Sound Lite | Smart Speaker | Basement Office | LinkPlay / WiFi | WiiM (HACS) | ✅ Set up |
| Sony STR-DN2010 | AV Receiver | 1F Family Room | Ethernet | ⛔ None | — |
| Vizio E60-C3 | TV | Basement Office | WiFi | SmartCast (official) | ⚠️ 2015 model — verify |
| Samsung UN49MU7000 | TV | 1F Family Room | WiFi / WebSocket | Samsung Smart TV | 🔲 Connected, not set up |
| TiVo Stream 4K IPA1114HDW | Streaming Device | 1F Family Room | Android TV / WiFi | Android TV Remote | 🔲 In use, not set up |
| Philips Hue Bridge v2 | Zigbee Hub | — | Zigbee / Ethernet | Philips Hue (official) | ✅ Set up — 192.168.200.195 |
| Hue Color Ambiance Slim Downlights 5/6" | Smart Lighting | Kitchen ×5 / Family Room ×6 | Zigbee via Bridge | Philips Hue (official) | ✅ Installed and in use |
| Hue Lightstrip Flux 10ft | Smart Lighting | Kitchen | Zigbee via Bridge | Philips Hue (official) | 📦 On hand, not installed |
| Hue bulbs ×2 | Smart Lighting | Basement Office | Zigbee via Bridge | Philips Hue (official) | ✅ In use |
| Hue tap dial switch | Wall Control | 1F Family Room | Zigbee via Bridge | Philips Hue (official) | ✅ In use |
| Hue dimmer switch | Wall Control | Kitchen | Zigbee via Bridge | Philips Hue (official) | ✅ In use |
| Magic Home LED strips | LED Strips | Basement Office | WiFi | flux_led (official) | ✅ East + South + TV bias in HA |

---

## Zigbee Infrastructure

### SMLIGHT SLZB-06M — Zigbee Coordinator

- **Manufacturer**: SMLIGHT
- **Protocol**: Zigbee 3.0 (IEEE 802.15.4); supports Thread/Matter-over-Thread via alternate firmware
- **Connectivity**: Ethernet (primary), WiFi, USB
- **HA Integration**: ZHA (Zigbee Home Automation)
- **Address**: 192.168.200.232, `socket://192.168.200.232:6638`, EZSP, `flow_control=software`
- **Firmware**: Core v3.2.4, Zigbee 20250220 (SDK 8.0.2)
- **Current status**: ✅ Set up — 111 ZHA entities in service

**Setup notes:**
- **ZHA is the authoritative Zigbee stack here.** Zigbee2MQTT and a Mosquitto broker are also
  installed as HA Apps, but hold **zero devices** — there is no `mqtt` platform in the entity
  registry. Treat ZHA as the only Zigbee integration; the Z2M install is vestigial.
- The Hue Bridge runs a **separate** Zigbee network. Hue devices pair to the bridge, not to ZHA.
- The SMLIGHT integration (separate from ZHA) monitors coordinator health, firmware, and temps.
- **`usePackets` must be ON for ZHA** and is cleared by a factory reset — always re-apply.
- Currently on WiFi. Ethernet is still preferred once office cabling is in place.

See `docs/device notes - zigbee coordinator - SLZB-06M.md` for the full recovery and API reference.

---

## Sensors

### THIRDREALITY Smart Plug Gen3 (×4)

- **Manufacturer**: THIRDREALITY
- **Protocol**: Zigbee 3.0
- **Rating**: 15A
- **HA Integration**: ZHA
- **Model**: 3RSP02064Z
- **Current status**: ✅ 3 paired — North / East / South Cabinet Underlights (Basement Office).
  4th plug is a spare.

**Features**: Power on/off, real-time energy monitoring (±1% accuracy), power-on state
restoration, timer. Also functions as a Zigbee mesh repeater — extending coordinator range.

**Setup notes**:
- Pair via ZHA. Community reports more reliable pairing through ZHA than Z2M.
- The three cabinet plugs are exposed as **lights, not switches**, via Switch-as-X helpers so
  they participate in the `light.office_all_lights` group and respond to lighting automations.

---

### THIRDREALITY Zigbee Temperature & Humidity Sensor (×2)

- **Manufacturer**: THIRDREALITY
- **Model**: 3RTHS24BZ (LCD display, confirmed)
- **Protocol**: Zigbee 3.0
- **Battery**: 2× AAA, ~1 year life
- **HA Integration**: ZHA — fully supported

**Specs**: ±1°C temp accuracy, ±2% RH humidity, 20-second refresh, LCD display

**Pairing**: Press and hold the side button for 5 seconds until the cloud icon blinks on the LCD — sensor is in pairing mode. HA will discover it within the 60-second ZHA pairing window.

---

### Apollo Automation MSR-2 mmWave Multisensor (LD2410B)

- **Manufacturer**: Apollo Automation
- **Protocol**: WiFi (ESP32-C3); **not Zigbee**
- **HA Integration**: ESPHome (official Apollo Automation integration)
- **Current status**: ✅ Set up — Basement Office, device "Office Occupancy Sensor"

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
- **In use by**: `binary_sensor.office_occupancy_sensor_radar_target` drives both office
  occupancy automations in `packages/lighting.yaml`. `radar_target` combines moving + still
  detection, so lights stay on while someone sits still.

---

## Voice Satellites

### ESP32-S3-BOX-3B (×2)

- **Manufacturer**: Espressif
- **Protocol**: ESPHome native API over WiFi
- **HA Integration**: Official Home Assistant voice satellite (`assist_satellite`)
- **Firmware**: 26.6.1 (ESPHome 2026.5.3) — both units
- **Current status**: ✅ Both set up

| Unit | Area | Device name | Entity prefix | MAC |
|---|---|---|---|---|
| #1 | Office | Office Voice Satellite | `office_voice_satellite_*` | `90:e5:b1:d6:4f:e0` |
| #2 | Kitchen | Kitchen Voice Satellite | `kitchen_voice_satellite_*` | `90:e5:b1:d6:50:6c` |

Both run the **Focused local assistant** pipeline with the **Okay Nabu** wake word detected
**on device** (microWakeWord, no cloud).

**Flashing**:
- Use the **Connect** button embedded in
  `https://www.home-assistant.io/voice_control/s3_box_voice_assistant/`, or
  `https://web.esphome.io` → *Install Voice Assistant*. The "Using the ESP32-S3-BOX-3" tab
  covers both the BOX-3 and the BOX-3B — same firmware.
- **`voice.home-assistant.io` is not an installer** — it redirects to a developer docs overview.
- Desktop Chrome/Edge only; WebSerial is unavailable on phone and tablet.
- Plug USB-C **into the box directly, not the docking station**.
- If the box doesn't enumerate, force flash mode: hold `boot` (upper left) while tapping
  `reset` (lower left).
- On Linux the port is `/dev/ttyACM0`, owned `root:dialout` and advertised as
  *"Espressif USB JTAG/serial debug unit"* (`303a:1001`). A user not in `dialout` cannot open
  it — grant access with `sudo setfacl -m u:$USER:rw /dev/ttyACM0`.

**Sensor dock**:
- The ESP32-S3-BOX-3-SENSOR dock is useful as a **stand and power source only**. The stock
  voice-assistant firmware defines no `sensor`, `binary_sensor`, or IR components, so the dock's
  mmWave, temperature/humidity and IR are **not exposed in HA**.
- Unlocking them requires a custom ESPHome build, giving up the precompiled firmware. Not
  worth it here — THIRDREALITY sensors already cover temp/humidity in both rooms, and a sensor
  mounted on a warm always-on box reads high (same self-heating issue as the Apollo MSR-2).

**Behaviour notes**:
- Firmware, WiFi credentials and the wake word model live on the box's flash; device name, area,
  entity IDs and pipeline selection live in HA. A power cycle loses nothing — verified across the
  25.12.2 → 26.6.1 OTA. Reflashing *does* reset the device side.
- Touchscreen can display custom information (clock, active media, etc.).
- On 26.6.1 the `speaker_enable` switch is reported `disabled_by: integration` — expected, not a
  fault.

---

## Audio

### Yamaha TSR-7810 — Basement Office Receiver

- **Manufacturer**: Yamaha
- **Type**: 7.2-channel AV receiver, Dolby Atmos / DTS:X, 4K
- **Protocol**: YNCA over network (Ethernet)
- **HA Integration**: `yamaha_ynca` custom integration (HACS) — recommended over official Yamaha integration
- **MusicCast**: Yes — supports Yamaha MusicCast for whole-home audio
- **Address**: 192.168.200.41 (YNCA over TCP, port 50000)
- **Current status**: ✅ Set up — MAIN zone customized to "Office Receiver" in `packages/audio.yaml`.
  ZONE2 is exposed but unassigned.
- **Reference notes**: `docs/manuals/yamaha-tsr-7810.md` — includes the official manual URL,
  how to find the receiver's IP from the front panel, and network setup notes

**Setup notes**:
- Install `yamaha_ynca` via HACS — provides richer control than the built-in Yamaha integration.
- Supports: power, volume, mute, source selection, playback controls.
- MusicCast integration enables multi-room audio coordination with other MusicCast devices.
- YNCA protocol only accepts one connection at a time — HA will hold the connection.

---

### WiiM Sound Lite — Basement Office Speaker

- **Manufacturer**: WiiM (Linkplay Technology)
- **Model**: WiiM Sound Lite
- **Protocol**: LinkPlay over WiFi
- **HA Integration**: WiiM Audio (HACS, by mjcumming) — integrated via Google Cast
- **Current status**: ✅ Set up — Basement Office
- **Manual**: `docs/manuals/wiim-sound-lite.pdf`

**Setup notes**:
- Integrated via Google Cast discovery — entity: `media_player.basement`
- Supports: playback control, volume, source selection, grouped/synchronized playback.
- Name the device in the WiiM app to control the entity name in HA.
- **Duplicate entity warning**: this speaker currently appears twice — `media_player.basement`
  (Cast) and `media_player.office_speaker` (WiiM Sound Speaker). Two integrations are claiming
  the same hardware. Automations should target `media_player.basement`, which is the one
  `packages/audio.yaml` customizes, until one of the two is disabled.
- Music Assistant does not produce sound through this speaker in a sync group, while the Yamaha
  does — under investigation.

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
- **Bridge**: v2 square, model 3241312018A, **192.168.200.195**
- **Current status**: ✅ Installed — Kitchen ×5 (NW, NE, SW, SE, Sink), Family Room ×6
  (N, NE, NW, S, SE, SW)

**Setup notes**:
- Hue Bridge connects to router via Ethernet. Add the Philips Hue integration in HA —
  press the button on the bridge when prompted.
- The Hue Bridge runs a separate Zigbee network from the SLZB-06M. Hue devices pair
  directly to the bridge, not to ZHA.
- Supports full color, brightness, scenes, and group control.
- Up to 50 bulbs per bridge.

**Room naming convention — matters for voice control**:

A Hue *Room* surfaces in HA as a single group light entity. If the room is named after the area
it sits in, Assist ends up with an entity literally named "Kitchen" inside the Kitchen area, which
collides with area-based voice commands. All three rooms are therefore renamed to
`<Area> Overhead Lights` **at the bridge**:

| Hue room | HA entity | Members |
|---|---|---|
| Office Overhead Lights | `light.office` | 2 bulbs |
| Kitchen Overhead Lights | `light.kitchen` | 5 downlights + dimmer switch |
| Family Room Overhead Lights | `light.family_room` | 6 downlights + tap dial switch |

- Rename **at the bridge**, not as an HA override — that is what propagates the name into HA.
- A rename changes the friendly name only; **entity IDs do not follow**, which is why they remain
  `light.office` / `light.kitchen` / `light.family_room`. Assist matches on friendly name, so this
  is sufficient.
- Direct bridge API (CLIP v2, self-signed cert so `curl -sk`); the application key lives in HA's
  `/config/.storage/core_config_entries` under the `hue` entry:

```bash
curl -sk -X PUT -H "hue-application-key: $KEY" -H "Content-Type: application/json" \
  -d '{"metadata":{"name":"New Name"}}' \
  https://192.168.200.195/clip/v2/resource/room/<room-id>
```

- **Known gap**: the individual downlights are not assigned to any HA area — only the Room group
  is. Area-scoped automations therefore see one light per room, not five or six.

---

### Philips Hue Tap Dial Switch + Dimmer Switch

- **Protocol**: Zigbee via Hue Bridge
- **HA Integration**: Philips Hue (official)
- **Current status**: ✅ In use — tap dial in Family Room, dimmer in Kitchen

Both are physical wall controls and are members of their respective Hue rooms. Their button
behaviours are stored as bridge `behavior_instance` resources that reference rooms and scenes
**by UUID, not by name** — so renaming a Hue room does not disturb them.

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
- **Current status**: ✅ East, South and TV bias in HA. North controller failed — needs replacement.

**Known IPs** (assign DHCP reservations):
- TV Bias Light: 192.168.200.233 (MAC TBD)
- Office East Over Cabinet LED: 192.168.200.215 (MAC TBD)
- Office South Over Cabinet LED: 192.168.200.188 (MAC TBD)
- Office North Over Cabinet LED: controller needs replacement

**Setup notes**:
- Add via Settings > Integrations > Magic Home in HA — enter IP address.
- Supports: on/off, color, brightness, effects.
- Assign a static IP (DHCP reservation) to prevent the IP changing after a router restart.
- **Longer-term**: consider replacing with Zigbee/Hue-compatible LED controllers to
  consolidate onto one ecosystem and eliminate WiFi dependency.

**Open question**: Confirm exact Magic Home model number (check device label or app).

---

## Issues & Decisions Log

### 1. SLZB-06M Firmware — Resolved (2026-03-30)
**Problem**: Firmware below 20250220 caused Zigbee coordinator instability, and the EFR32MG21
radio became unresponsive (port 6638 closed, `zb_version: 0`).
**Resolution**: USB reflash via `smlight.tech/flasher` with the combined image → Core v3.2.4 +
Zigbee 20250220 (SDK 8.0.2). ZHA connected over `socket://192.168.200.232:6638`, EZSP,
`flow_control=software`. Remember `usePackets` must be ON and is cleared by factory reset.

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

### 5. Hue Room Names Collide With Voice Commands — Resolved (2026-04-05, 2026-07-26)
**Problem**: A Hue Room named after its area produces an HA entity named e.g. "Kitchen" sitting
*inside* the Kitchen area, which conflicts with area-based Assist commands.
**Resolution**: All three rooms renamed at the bridge to `<Area> Overhead Lights`. Office first
(2026-04-05, when voice satellite #1 hit the conflict), Kitchen and Family Room on 2026-07-26
ahead of satellite #2.

### 6. Zigbee2MQTT Installed But Unused — Clarification
**Note**: Zigbee2MQTT and Mosquitto are installed as HA Apps but hold zero devices — there is no
`mqtt` platform in the entity registry. **ZHA is the authoritative Zigbee stack.** The Z2M install
is vestigial; don't infer a dual-stack setup from its presence.

---

## Open Questions

- [x] WiiM exact model — confirmed WiiM Sound Lite
- [x] Hue Bridge — confirmed v2 square (model 3241312018A), 192.168.200.195
- [x] Yamaha TSR-7810 — confirmed on the network at 192.168.200.41 and set up in HA
- [x] SLZB-06M firmware version — Core v3.2.4, Zigbee 20250220 (SDK 8.0.2)
- [ ] Magic Home LED strip exact model (check device label)
- [ ] Vizio E60-C3 SmartCast compatibility — verify with Vizio support
- [ ] Assign individual Kitchen/Family Room downlights to their HA areas — currently only the
      Hue Room group carries an area
- [ ] Replace the failed Office North Over Cabinet LED controller

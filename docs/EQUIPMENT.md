# Equipment — Home Assistant integration status

How each device in the house is integrated into Home Assistant: which
integration, how to pair it, what it exposes, and what to watch out for.
Last updated: 2026-08-23.

> **Hardware facts live in the `my-hardware` repository**, not here. Make,
> model, serial, MAC, IP, firmware version, physical location and the physical
> gotchas are recorded in `my-hardware/devices/<id>.md` — see its `INDEX.md`.
>
> This file records **Home Assistant knowledge** only. The two are deliberately
> not duplicated: if you need to know what a thing *is*, look there; if you need
> to know how HA talks to it, look here.
>
> Path: `/home/mitchiner/Documents/projects/my-hardware`

**Status icons:**
- ✅ Supported — works well
- ⚠️ Supported with caveats — see notes
- ⛔ Not supported — no viable HA integration
- 🔲 Not yet set up
- 📦 Not yet arrived

---

## Quick Reference

| Device | Room | HA Integration | Status | Hardware record |
|---|---|---|---|---|
| SMLIGHT SLZB-06M | — | ZHA | ✅ Core v3.2.4, Zigbee 20250220 | `slzb-06m` |
| THIRDREALITY Smart Plug Gen3 ×4 | Basement Office | ZHA | ✅ 3 paired (cabinet underlights); 4th spare | `thirdreality-smart-plug-gen3` |
| THIRDREALITY Temp/Humidity ×2 | Office + Kitchen | ZHA | ✅ Set up | `thirdreality-temp-humidity` |
| Apollo MSR-2 mmWave | Basement Office | ESPHome | ✅ Set up — drives occupancy automations | `apollo-msr-2` |
| ESP32-S3-BOX-3B ×2 | Office + Kitchen | Official HA voice | ✅ Both set up, firmware 26.6.1 | `esp32-s3-box-3b` |
| Yamaha TSR-7810 | Basement Office | yamaha_ynca (HACS) | ✅ Set up | `yamaha-tsr-7810` |
| WiiM Sound Lite | Basement Office | WiiM (HACS) | ⚠️ Set up — duplicate entity, see below | `wiim-sound-lite` |
| Sony STR-DN2010 | 1F Family Room | ⛔ None | — | `sony-str-dn2010` |
| Vizio E60-C3 | Basement Office | SmartCast (official) | ⚠️ 2015 model — verify | `vizio-e60-c3` |
| Samsung UN49MU7000 | 1F Family Room | Samsung Smart TV | 🔲 Connected, not set up | `samsung-un49mu7000` |
| TiVo Stream 4K | 1F Family Room | Android TV Remote | 🔲 In use, not set up | `tivo-stream-4k` |
| Philips Hue Bridge v2 | Basement Office | Philips Hue (official) | ✅ Set up | `hue-bridge-v2` |
| Hue Slim Downlights ×11 | Kitchen ×5 / Family Room ×6 | Philips Hue (official) | ✅ Installed and in use | `hue-downlights` |
| Hue Lightstrip Flux 10ft | Kitchen | Philips Hue (official) | 📦 On hand, not installed | `hue-lightstrip-flux` |
| Hue bulbs ×2 | Basement Office | Philips Hue (official) | ✅ In use | `hue-bulbs` |
| Hue tap dial switch | 1F Family Room | Philips Hue (official) | ✅ In use | `hue-tap-dial` |
| Hue dimmer switch | Kitchen | Philips Hue (official) | ✅ In use | `hue-dimmer` |
| Magic Home LED strips | Basement Office | flux_led (official) | ⚠️ East + South + TV bias; North failed | `magic-home-led` |

---

## Zigbee Infrastructure

### SMLIGHT SLZB-06M — Zigbee Coordinator

- **HA Integration**: ZHA (Zigbee Home Automation)
- **ZHA config**: `socket://192.168.200.232:6638`, radio type **EZSP**,
  `flow_control=software`
- **Current status**: ✅ Set up — 111 ZHA entities in service

**Why EZSP and not Zigbee2MQTT**: the 20250220 firmware uses SDK 8.0.2 and
requires `adapter: ember` in a Z2M config. ZHA handles this natively via EZSP
with no extra configuration. For a new setup with no existing Z2M config, ZHA is
the right choice and avoids the stability issues reported with this coordinator
under Z2M.

**Setup notes:**
- **ZHA is the authoritative Zigbee stack here.** Zigbee2MQTT and a Mosquitto
  broker are also installed as HA Apps, but hold **zero devices** — there is no
  `mqtt` platform in the entity registry. Treat ZHA as the only Zigbee
  integration; the Z2M install is vestigial.
- The Hue Bridge runs a **separate** Zigbee network. Hue devices pair to the
  bridge, not to ZHA.
- The SMLIGHT integration (separate from ZHA) monitors coordinator health,
  firmware and temps.
- **`usePackets` must be ON for ZHA** and is cleared by a factory reset — always
  re-apply before adding ZHA:

  ```bash
  curl -s -X POST "http://192.168.200.232/settings/saveParams" -d "pageId=3&usePackets=on"
  # Response must show: "changes":{"usePackets":true},"needReboot":true
  curl -s "http://192.168.200.232/api2?action=1&param=reboot"
  ```

Firmware history, the API reference, the USB recovery procedure and the reason
the web UI cannot be scraped are all in `my-hardware/devices/slzb-06m.md`.

**Open HA-side items:**
- [ ] Enable "Automatic Zigbee Update" once ZHA is stable (currently Off)

---

## Sensors

### THIRDREALITY Smart Plug Gen3 (×4)

- **HA Integration**: ZHA
- **Current status**: ✅ 3 paired — North / East / South Cabinet Underlights
  (Basement Office). 4th plug is a spare.

**Setup notes**:
- Pair via ZHA. Community reports more reliable pairing through ZHA than Z2M.
- The three cabinet plugs are exposed as **lights, not switches**, via
  Switch-as-X helpers so they participate in the `light.office_all_lights` group
  and respond to lighting automations.
- They report real-time energy use, which is available for energy dashboards.

### THIRDREALITY Zigbee Temperature & Humidity Sensor (×2)

- **HA Integration**: ZHA — fully supported
- **Pairing**: press and hold the side button for 5 seconds until the cloud icon
  blinks on the LCD, then find it within the 60-second ZHA pairing window.

These are the **preferred temperature source** for the office and kitchen. Both
other candidates in those rooms (the Apollo MSR-2 and the voice satellite dock)
self-heat and read high.

### Apollo Automation MSR-2 mmWave Multisensor

- **HA Integration**: ESPHome (official Apollo Automation integration)
- **Current status**: ✅ Set up — Basement Office, device "Office Occupancy Sensor"

**Setup notes**:
- Connect to "Apollo MSR-2 Hotspot" WiFi on first boot, select the home network,
  and the device auto-discovers in HA via ESPHome.
- For mesh WiFi networks, enter the SSID manually rather than scanning.
- **Do not use its temperature reading as a primary source** — it needs a +3–5°C
  offset because the ESP32-C3 self-heats. Use the THIRDREALITY sensors instead.

**In use by**: `binary_sensor.office_occupancy_sensor_radar_target` drives both
office occupancy automations in `packages/lighting.yaml`. `radar_target`
combines moving + still detection, so lights stay on while someone sits still —
this is the whole reason for choosing mmWave over PIR.

---

## Voice Satellites

### ESP32-S3-BOX-3B (×2)

- **HA Integration**: official Home Assistant voice satellite (`assist_satellite`)
- **Current status**: ✅ Both set up

| Unit | Area | Device name | Entity prefix |
|---|---|---|---|
| #1 | Office | Office Voice Satellite | `office_voice_satellite_*` |
| #2 | Kitchen | Kitchen Voice Satellite | `kitchen_voice_satellite_*` |

Both run the **Focused local assistant** pipeline with the **Okay Nabu** wake
word detected **on device** (microWakeWord, no cloud).

**What lives where**: device name, area, entity IDs and pipeline selection live
in HA. Firmware, WiFi credentials and the wake-word model live on the box. A
power cycle loses nothing; a reflash resets the device side.

**Behaviour notes**:
- The touchscreen can display custom information (clock, active media, etc.).
- On 26.6.1 the `speaker_enable` switch reports `disabled_by: integration` —
  expected, not a fault.
- The sensor dock's mmWave, temperature/humidity and IR are **not exposed in
  HA** and cannot be without a custom ESPHome build. Not worth it — see the
  hardware record for why.

The flashing procedure and its gotchas are in
`my-hardware/devices/esp32-s3-box-3b.md`.

---

## Audio

### Yamaha TSR-7810 — Basement Office Receiver

- **HA Integration**: `yamaha_ynca` custom integration (HACS) — richer control
  than the built-in Yamaha integration
- **GitHub**: https://github.com/mvdwetering/yamaha_ynca
- **Current status**: ✅ Set up — MAIN zone customized to "Office Receiver" in
  `packages/audio.yaml`. ZONE2 is exposed but unassigned.
- Supports power, volume, mute, source selection and playback controls.
- MusicCast is also supported and enables multi-room coordination with other
  MusicCast devices.

**YNCA accepts one connection at a time and HA holds it.** Nothing else can talk
to the receiver's control port while the integration is running — stop it first
before debugging with another client.

### WiiM Sound Lite — Basement Office Speaker

- **HA Integration**: WiiM Audio (HACS, by mjcumming) — integrated via Google Cast
- **Current status**: ✅ Set up — entity `media_player.basement`
- Supports playback control, volume, source selection and grouped playback.

**Duplicate entity warning**: this speaker currently appears twice —
`media_player.basement` (Cast) and `media_player.office_speaker` (WiiM Sound
Speaker). Two integrations are claiming the same hardware. Automations should
target `media_player.basement`, which is the one `packages/audio.yaml`
customizes, until one of the two is disabled.

**Open**: Music Assistant does not produce sound through this speaker in a sync
group, while the Yamaha does. Under investigation.

Name the device in the WiiM app to control the entity name in HA.

### Sony STR-DN2010 — 1F Family Room Receiver

- **HA Integration**: ⛔ None. Sony Songpal supports 2014+ models only.
- **Current status**: In use, family room

**Options until replaced**:
- IR blaster (e.g. Broadlink RM4 Pro) for basic power/volume/source control
- Harmony Hub for a consolidated remote plus HA bridge
- Manual remote only — acceptable, since family room is Phase 2 and receiver
  replacement is already planned

This device's lack of an integration does not block Phase 1 work.

---

## Displays & Media

### Vizio E60-C3 — Basement Office TV

- **HA Integration**: VIZIO SmartCast (official) — **but SmartCast requires
  2016+ models**, and this is a 2015 set
- **Current status**: In use, likely display-only

Verify at `support.vizio.com`. If incompatible, an IR blaster is the fallback
for power and input control.

### Samsung UN49MU7000 — 1F Family Room TV

- **HA Integration**: Samsung Smart TV (official) — fully supported
- **Current status**: Connected via WiFi, not yet set up in HA
- Supports power, volume, source and media state.

**Setup notes**:
- HA and the TV must be on the **same subnet**. This will not work across VLANs.
- Assign a static IP via a router DHCP reservation to prevent connection drops.
- Set TV access permission to "First time only"
  (`Settings > General > External Device Manager`) to avoid a per-session
  approval prompt on screen.

### TiVo Stream 4K — 1F Family Room

- **HA Integration**: Android TV Remote (official) — fully supported
- **Current status**: In use, not yet set up
- Auto-discovered via the Android TV Remote integration. Supports media
  controls, app launching and volume. Requires Android TV Remote Service
  (pre-installed).

**This is an Android TV streaming player, not a cable box** — it aggregates
streaming apps and does not decode a cable signal. HA integrates with Android
TV, not with TiVo.

---

## Lighting

### Philips Hue Bridge v2 and the Slim Downlights

- **HA Integration**: Philips Hue (official) — excellent
- **Current status**: ✅ Installed — Kitchen ×5 (NW, NE, SW, SE, Sink),
  Family Room ×6 (N, NE, NW, S, SE, SW)

**Setup notes**:
- Add the Philips Hue integration in HA and press the button on the bridge when
  prompted.
- The Hue Bridge runs a separate Zigbee network from the SLZB-06M. Hue devices
  pair directly to the bridge, not to ZHA.
- Supports full colour, brightness, scenes and group control.

**Room naming convention — matters for voice control**:

A Hue *Room* surfaces in HA as a single group light entity. If the room is named
after the area it sits in, Assist ends up with an entity literally named
"Kitchen" inside the Kitchen area, which collides with area-based voice
commands. All three rooms are therefore renamed to `<Area> Overhead Lights`
**at the bridge**:

| Hue room | HA entity | Members |
|---|---|---|
| Office Overhead Lights | `light.office` | 2 bulbs |
| Kitchen Overhead Lights | `light.kitchen` | 5 downlights + dimmer switch |
| Family Room Overhead Lights | `light.family_room` | 6 downlights + tap dial switch |

- Rename **at the bridge**, not as an HA override — that is what propagates the
  name into HA.
- A rename changes the friendly name only; **entity IDs do not follow**, which
  is why they remain `light.office` / `light.kitchen` / `light.family_room`.
  Assist matches on friendly name, so this is sufficient.
- Direct bridge API (CLIP v2, self-signed cert so `curl -sk`); the application
  key lives in HA's `/config/.storage/core_config_entries` under the `hue` entry:

```bash
curl -sk -X PUT -H "hue-application-key: $KEY" -H "Content-Type: application/json" \
  -d '{"metadata":{"name":"New Name"}}' \
  https://192.168.200.195/clip/v2/resource/room/<room-id>
```

- **Known gap**: the individual downlights are not assigned to any HA area —
  only the Room group is. Area-scoped automations therefore see one light per
  room, not five or six.

### Philips Hue Tap Dial Switch + Dimmer Switch

- **HA Integration**: Philips Hue (official)
- **Current status**: ✅ In use — tap dial in Family Room, dimmer in Kitchen

Both are members of their respective Hue rooms. Their button behaviours are
stored as bridge `behavior_instance` resources that reference rooms and scenes
**by UUID, not by name** — so renaming a Hue room does not disturb them.

### Philips Hue Lightstrip Flux 10ft

- **HA Integration**: Philips Hue (official) — same integration as everything else
- **Current status**: On hand, not yet installed (kitchen under-counter — post-remodel)

### Philips Hue Bulbs ×2 — Basement Office Overhead

- **HA Integration**: Philips Hue (official)
- **Current status**: ✅ In use — Phase 1 starting point

### Magic Home WiFi LED Strips — Basement Office

- **HA Integration**: `flux_led` (official) — no firmware flash required
- **Current status**: ✅ East, South and TV bias in HA. North controller failed
  — needs replacement.

**Setup notes**:
- Add via `Settings > Integrations > Magic Home` and enter the IP address.
- Supports on/off, colour, brightness and effects.
- **These are configured by IP and sit on plain DHCP.** Assign reservations —
  a router restart silently breaks them. The IPs are recorded in
  `my-hardware/devices/magic-home-led.md`.
- **Longer-term**: consider replacing with Zigbee/Hue-compatible controllers to
  consolidate onto one ecosystem and remove the WiFi dependency.

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

### 7. Hardware Facts Split Out To `my-hardware` — 2026-08-23
**Problem**: This file mixed two different kinds of knowledge — what a device *is*
(model, serial, MAC, IP, firmware, physical gotchas) and how HA talks to it. The
first kind belongs with all the other house hardware, most of which HA never sees.
**Resolution**: Hardware identity moved to the `my-hardware` repository, one record
per device under `devices/`. This file keeps the HA-side knowledge and links across.
Nothing is recorded in both places. The `Hardware record` column in the Quick
Reference gives each device's record id.

---

## Open Questions

- [x] WiiM exact model — confirmed WiiM Sound Lite
- [x] Hue Bridge — confirmed v2 square, and its HA area is Office
- [x] Yamaha TSR-7810 — confirmed on the network and set up in HA
- [x] SLZB-06M firmware version — Core v3.2.4, Zigbee 20250220 (SDK 8.0.2)
- [ ] Vizio E60-C3 SmartCast compatibility — verify with Vizio support
- [ ] Assign individual Kitchen/Family Room downlights to their HA areas — currently only the
      Hue Room group carries an area
- [ ] Replace the failed Office North Over Cabinet LED controller
- [ ] Resolve the duplicate WiiM entity (`media_player.basement` vs
      `media_player.office_speaker`)
- [ ] `living_room`, `bedroom` and `family_room` have no floor assigned in the HA
      floor registry, though the family room is documented as 1st floor

> Hardware-side open questions (Magic Home model and MACs, the router's make and
> model, the HA host's Pi model) now live in `my-hardware/TODO.md`.

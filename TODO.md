# TODO

## In Progress

- [ ] Music Assistant multi-room audio — Office Speaker (WiiM) not producing sound in MA; Yamaha works. Investigating WiiM Cast/protocol compatibility with MA sync group.

## Phase 1 — Basement Office (next up)

- [x] Pair THIRDREALITY Smart Plugs ×3 to ZHA (2026-03-30) — 4th plug TBD
- [x] Pair THIRDREALITY Temp/Humidity sensors ×2 to ZHA (2026-03-30) — Desk (Office) + Kitchen
- [x] Set up Apollo MSR-2 occupancy sensor via ESPHome integration (2026-04-05)
  - [x] Paired to HA via ESPHome, assigned to Office area
  - [x] Occupancy automations created — lights on when occupied, off after 10 min vacancy (packages/lighting.yaml)
- [x] Add Philips Hue integration + pair existing 2× Hue office overhead bulbs (2026-03-30)
- [x] Integrate Magic Home TV Bias Light via Magic Home integration (2026-03-30) — 192.168.200.233
- [x] Integrate Magic Home over-cabinet LEDs (2026-03-31) — Office East Over Cabinet LED + Office South Over Cabinet LED
- [ ] Replace Office North Over Cabinet LED controller (hardware failed) — low priority
- [x] Set up Yamaha TSR-7810 via yamaha_ynca (HACS) (2026-03-31) — 192.168.200.41, Office area
- [x] Set up WiiM Sound Lite via WiiM Audio integration (HACS) (2026-03-30) — renamed to "Office Speaker", assigned to Office area (2026-04-05)
- [x] Set up voice satellite #1 — ESP32-S3-BOX-3B (2026-04-04) — Office area, Wyoming local pipeline
  - [x] Flashed via the ESP Web Tools installer (the URL recorded here originally was wrong —
        see `docs/EQUIPMENT.md` for the correct one)
  - [x] Wyoming Faster Whisper + Piper installed and running
  - [x] Rename Hue "Office" group entity → "Office Overhead Lights" (2026-04-05) — resolved voice conflict; "office lights" now targets all 8 lights via Office All Lights group
- [x] Set up voice satellite #2 — ESP32-S3-BOX-3B — **Kitchen area** (2026-07-26)
  - [x] Rename satellite #1 entity IDs `esp32_s3_box_3_d64fe0_*` → `office_voice_satellite_*`
  - [x] Flash box #2, join WiFi, adopt in HA — firmware 26.6.1, MAC `90:e5:b1:d6:50:6c`
  - [x] Assign to Kitchen area + rename entities → `kitchen_voice_satellite_*` (13 registry changes)
  - [x] Resolve Hue "Kitchen" room naming conflict before go-live
  - [x] Pipeline set to "Focused local assistant" (Speech-to-Phrase + Piper `en_US-lessac-medium`),
        wake word "Okay Nabu" on device — matching satellite #1
  - Flashing procedure and gotchas are documented in `docs/EQUIPMENT.md` → Voice Satellites.
    Note `voice.home-assistant.io` is NOT the installer — it 301s to a developer docs overview.
  - Sensor dock: use for power/stand only. The stock `esphome.voice-assistant` firmware defines no
    sensor/binary_sensor/IR components, so the dock's mmWave, temp/humidity and IR are NOT exposed
    in HA — confirmed against the upstream YAML and against satellite #1, which has zero sensor
    entities. Unlocking them requires a custom ESPHome build.
- [ ] Verify satellite #2 end to end — say "Okay Nabu, turn on the kitchen lights" and confirm
      mic, speaker and the `light.kitchen` rename all behave. Not verifiable from the API.
- [x] ~~Satellite #1 went offline 2026-07-26 19:29 and had not returned~~ — **it did return**
      (2026-07-26). It recovered on its own at 15:56:48 EDT and is not unplugged. Now tracked as an
      intermittent-flapping issue under Infrastructure & Housekeeping below.
- [ ] Kitchen has no occupancy/presence sensor — the only real gap the sensor dock would fill.
      Prefer a second Apollo MSR-2 (proven pattern, already used in Office) over a custom ESPHome
      build on the voice satellite. Temp/humidity is already covered by the THIRDREALITY sensor,
      and a dock sensor on a warm always-on box would read high anyway.
- [x] Update satellite #1 firmware 25.12.2 → 26.6.1 (2026-07-26) — now 26.6.1 (ESPHome 2026.5.3);
      area, device name, entity IDs, pipeline and wake word all survived the OTA
- [x] Rename Hue "Kitchen" room → "Kitchen Overhead Lights" (2026-07-26) — renamed at the bridge via
      CLIP v2 API (room 930fbc0d), propagated to HA. No entity is named exactly "Kitchen" any more.
      Entity ID stays `light.kitchen`, matching the Office precedent (`light.office`) — only the
      friendly name changed there too.
- [ ] Assign static DHCP reservations for all WiFi/network devices
  - Yamaha TSR-7810: 192.168.200.41 (get MAC from router)
  - Magic Home TV Bias Light: 192.168.200.233 (get MAC from router)
  - Magic Home East Over Cabinet LED: 192.168.200.215 (get MAC from router)
  - Magic Home South Over Cabinet LED: 192.168.200.188 (get MAC from router)
  - SLZB-06M: 192.168.200.232 (MAC 68:25:DD:47:C9:8C)
  - Hue Bridge v2: 192.168.200.195 (get MAC from router)
  - Office Voice Satellite: 192.168.200.144 (MAC 90:E5:B1:D6:4F:E0)
  - Kitchen Voice Satellite: 192.168.200.78 (MAC 90:E5:B1:D6:50:6C)
  - Apollo MSR-2 (Office Occupancy): 192.168.200.44
- [x] Create packages/ directory structure and first package files (2026-04-04)
  - packages/lighting.yaml — Office All Lights group (8 lights incl. TV bias), occupancy automations
  - packages/audio.yaml — Office audio zones (Office Speaker + Office Receiver)

## Phase 2 — 1st Floor Kitchen & Family Room (partially deployed ahead of remodel)

- [x] Kitchen: Philips Hue downlights ×5 installed (NW, NE, SW, SE, Sink) + Hue dimmer switch
- [x] Family Room: Hue downlights ×6 installed (N, NE, NW, S, SE, SW) + Hue tap dial switch
- [x] Kitchen: voice satellite #2 (2026-07-26)
- [ ] Kitchen: Hue Lightstrip Flux under-counter (on hand, awaiting remodel)
- [ ] Kitchen/Family Room: smart switches for pendant fixtures
- [ ] Family Room: smart outlets for lamps and seasonal
- [ ] Family Room: replace Sony STR-DN2010 with IP-controllable receiver
- [ ] Expand whole-home audio with WiiM speakers
- [ ] Integrate Samsung UN49MU7000 (Samsung Smart TV integration)
- [ ] Integrate TiVo Stream 4K IPA1114HDW (Android TV Remote integration)

## Phase 3 — Whole Home

- [ ] Additional WiiM speakers in remaining rooms
- [ ] Garage door control
- [ ] HVAC integration
- [ ] Leak detection
- [ ] Plant soil moisture monitoring
- [ ] Additional voice satellite deployment

## Infrastructure & Housekeeping

- [ ] Assign static DHCP reservation for SLZB-06M (MAC 68:25:DD:47:C9:8C → 192.168.200.232)
- [ ] Decide sandbox server hardware (x86 mini PC preferred over RPI5 for Whisper)
- [ ] Check Vizio E60-C3 SmartCast compatibility (likely 2015 = incompatible)
- [x] Confirm WiiM exact model — WiiM Sound Lite (2026-04-05)
- [ ] Confirm Magic Home LED strip exact model
- [x] Confirm Hue Bridge generation — v2 square (model 3241312018A)
- [ ] Open pending fixture boxes — check pendant bulb socket type
- [ ] Set up git pre-commit hook for yamllint
- [ ] **`configuration.yaml` and `packages/` are still untracked** — CLAUDE.md states "all
      configuration is managed as code in this git repository", which is not yet true. Commit them.
- [x] **Resolve `packages/audio.yaml` drift between repo and HA** (2026-07-26) — pushed the repo
      version to `/config/packages/audio.yaml`; the deployed copy had targeted
      `media_player.basement_office`, which does not exist. Config check valid, applied with
      `homeassistant.reload_core_config` (no restart needed for `customize:`). Old copy backed up
      to `/config/audio.yaml.bak-20260726`, deliberately outside `packages/` so it is not loaded.
      `lighting.yaml` and `configuration.yaml` were already identical.
- [ ] Consider dropping the `customize:` blocks in `packages/audio.yaml` entirely — "Office Speaker"
      and "Office Receiver" already come from the device registry, so the block changes nothing.
      Fixing the stale target made it correct, not useful.
- [ ] Remove `/config/audio.yaml.bak-20260726` once the change has proven stable
- [ ] **Duplicate media_player entities** — two "Office Speaker" (`media_player.basement` and
      `media_player.office_speaker`) and two "Family Room TV" (`media_player.family_room_tv`
      and `..._tv_2`). Likely Cast and WiiM/native integrations both claiming the same hardware.
      Pick one per device and disable the other.
- [ ] `configuration.yaml` includes `frontend: themes: !include_dir_merge_named themes` but
      `/config/themes/` does not exist. HA is running fine, so it is tolerated — but create the
      directory or drop the block before relying on `ha core check`.
- [x] **Sync docs with live HA** (2026-07-26) — DESIGN.md and EQUIPMENT.md were months behind.
      Reconciled: Kitchen ×5 and Family Room ×6 Hue downlights, Hue tap dial + dimmer switches,
      Hue Bridge v2 at 192.168.200.195, both voice satellites, Apollo MSR-2, ZHA/Yamaha/WiiM status.
  - Resolved the ZHA vs Zigbee2MQTT question: **ZHA is authoritative** (111 entities). Z2M and
    Mosquitto are installed but hold zero devices — there is no `mqtt` platform in the registry.
- [ ] Assign individual Kitchen/Family Room downlights to their HA areas — currently only the Hue
      Room group carries an area, so area-scoped automations see one light per room, not five or six
- [ ] Music Assistant, Speech-to-Phrase and File editor apps are installed but undocumented —
      decide whether they are keepers and note them if so
- [ ] Consider a "Kitchen All Lights" group mirroring `light.office_all_lights`, once the Hue
      Lightstrip Flux is installed under-counter. Today `light.kitchen` covers all 5 downlights,
      so a group adds nothing yet.
- [x] Rename Hue "Family Room" room → "Family Room Overhead Lights" (2026-07-26) — renamed at the
      bridge (room 7f6c0172), propagated to HA. All three Hue rooms now follow the
      "<Area> Overhead Lights" convention. Entity ID stays `light.family_room`.
      Verified the Hue tap dial switch is unaffected: its behavior_instance binds rooms and scenes
      by UUID, not by name, and remains enabled/running.
- [x] Set up SSH access to HA (Advanced SSH & Web Terminal app, key auth) (2026-04-04)
- [x] **Install the ha-mcp server** (2026-07-26) — `homeassistant-ai/ha-mcp` v7.14.2 as an HA app
      (slug `81f33d0f_ha_mcp`, boot: auto). Adds 74 live tools vs the ~24 Assist-scoped intents the
      built-in `/api/mcp` server provides — most importantly automation traces, history and logs,
      which were previously undiagnosable. Installed guarded: tool security policies on, auto-backup
      on, snapshot delete off, and the four hard-delete tools (`ha_remove_device`, `ha_remove_entity`,
      `ha_remove_area_or_floor`, `ha_remove_helpers_integrations`) disabled. Both MCP servers run
      side by side for now.
- [ ] **Revisit the HACS in-process ha-mcp component after an HA Core upgrade to >= 2026.6.0**
      (box is on 2026.3.4). It is the project's recommended install and removes the separate app
      container, but the Core version gate blocks it today.
- [ ] **Decide whether to drop the built-in `/api/mcp` server** after the ha-mcp trial period.
      It costs near-zero idle context (Opus defers tool loading) and is a useful fallback, but it
      is largely redundant with ha-mcp apart from its Assist-exposure scoping.
- [ ] Do not author automations via ha-mcp's `ha_config_set_automation` — it writes to `.storage`,
      which PRINCIPLES.md §4 forbids. Authoring stays in `packages/` via SSH. (Guardrail note, not
      a task; delete if it stops being a live risk.)
- [ ] **Office voice satellite #1 flaps** — ha-mcp history (2026-07-26) shows two self-recovering
      dropouts, 15:17:13→15:17:27 EDT and 15:29:57→15:56:48 EDT. It is not unplugged; it completed
      a full voice interaction at 15:18. Likely the same WiFi instability as the SLZB-06M below —
      chase that first. Note the ESPHome firmware exposes **no WiFi signal/RSSI entity**, so
      signal strength cannot be read from HA; check at the AP/router instead.
- [x] **Fix `light.tv_bias_light` in `packages/lighting.yaml`** (2026-07-26) — the entity does not
      exist; the real one is `light.tv_led_strip` ("TV Bias Light" is only its friendly name). The
      TV bias light was silently dropped from every "Office All Lights" action, and HA logged
      `Referenced entities light.tv_bias_light are missing or not currently available` on each run.
      Corrected, pushed, `ha core check` passed, applied with `homeassistant.reload_all`
      (no restart). Backup at `/config/lighting.yaml.bak-20260726`, outside `packages/`.
- [x] ~~Office voice satellite has duplicate select entities~~ **NOT A BUG** (2026-07-26) — the
      `_2` selects are a real ESP32-S3-BOX-3B firmware feature, not orphans. The firmware exposes
      **two independent wake-word → pipeline slots** (unique_ids `-pipeline` / `-pipeline_2`), and
      both satellites have them. Slot 1 = "Okay Nabu" → Focused local assistant (active);
      slot 2 = `no_wake_word` → Home Assistant pipeline (idle, no wake word assigned).
      Nothing to remove.
- [ ] **Opportunity from the above:** slot 2 is free on both satellites. Could assign
      "Hey Jarvis" → "Full local assistant" (faster_whisper) as a slow-but-smarter fallback,
      keeping "Okay Nabu" → Speech-to-Phrase as the fast daily driver.
- [ ] **🔴 SLZB-06M Zigbee coordinator is in a continuous reconnect loop** (found 2026-07-26) —
      all 57 ZHA entities unavailable. `socket_uptime` resets to ~1 s every 20–30 s while device
      uptime is 17.7 days, so the coordinator is fine but the **EZSP-over-TCP link keeps dropping**.
      HA log: `bellows.ash.NcpFailure: NcpResetCode.ERROR_EXCEEDED_MAXIMUM_ACK_TIMEOUT_COUNT`.
      Root cause: the SLZB-06M is running on **WiFi, not Ethernet** (`"ethernet":false,
      "wifi_connected":true` from its `/ha_sensors` API), and ping jitter is 7.8–77.8 ms
      (mdev 32 ms). EZSP is latency-sensitive and blows its ACK timeout on that jitter.
      **Fix: move the SLZB-06M to wired Ethernet** — it has a PoE-capable port. This is almost
      certainly also why the office voice satellite flaps.

## Completed

- [x] Set up GitHub repository (github.com/bmitchin/home-automation)
- [x] Create CLAUDE.md with HA-specific guidance
- [x] Create docs/PRINCIPLES.md — guiding principles north star
- [x] Create docs/DESIGN.md — system architecture and phased plan
- [x] Create docs/EQUIPMENT.md — verified equipment inventory
- [x] Update SLZB-06M Core firmware: v2.9.3 → v3.2.0 (2026-03-23), then → v3.2.4 via USB reflash (2026-03-30)
- [x] Update SLZB-06M Zigbee firmware: 20231030 → 20250220 SDK 8.0.2 (2026-03-23)
- [x] Create device notes for SLZB-06M
- [x] Add SMLIGHT integration to HA (2026-03-23)
- [x] Add ZHA integration to HA — socket://192.168.200.232:6638, EZSP, flow_control=software (2026-03-30)

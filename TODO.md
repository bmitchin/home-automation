# TODO

## In Progress

- [ ] Connect SLZB-06M to HA via ZHA
  - Add SMLIGHT integration (Settings → Devices & Services → SMLIGHT, auto-discovers at 192.168.200.232)
  - Add ZHA integration (socket://192.168.200.232:6638, radio type EZSP)
  - Confirm Zigbee firmware version reads as 20250220 once ZHA connects

## Phase 1 — Basement Office (next up)

- [ ] Pair THIRDREALITY Smart Plugs ×4 to ZHA (pair these first — they extend the mesh)
- [ ] Pair THIRDREALITY Temp/Humidity sensors ×2 to ZHA
- [ ] Set up Apollo MSR-2 occupancy sensor (arriving Thursday) via ESPHome integration
- [ ] Add Philips Hue integration + pair existing 2× Hue office overhead bulbs
- [ ] Integrate Magic Home LED strips via flux_led integration (find IP, set DHCP reservation)
- [ ] Set up Yamaha TSR-7810 via yamaha_ynca (HACS install required first)
- [ ] Set up WiiM streamer via WiiM integration (HACS)
- [ ] Set up voice satellite — ESP32-S3-BOX-3B ×2 (arriving Tuesday)
  - Flash via HA voice installer at voice.home-assistant.io
  - Configure Wyoming pipeline (local Whisper + Piper)
- [ ] Assign static DHCP reservations for all WiFi/network devices
- [ ] Create packages/ directory structure and first package files

## Phase 2 — 1st Floor Kitchen & Family Room (post-remodel)

- [ ] Kitchen: Philips Hue downlights, Hue Lightstrip Flux under-counter
- [ ] Kitchen/Family Room: smart switches for pendant fixtures
- [ ] Family Room: Hue downlights, smart outlets for lamps and seasonal
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

- [ ] Assign static DHCP reservation for SLZB-06M (currently 192.168.200.232)
- [ ] Decide sandbox server hardware (x86 mini PC preferred over RPI5 for Whisper)
- [ ] Check Vizio E60-C3 SmartCast compatibility (likely 2015 = incompatible)
- [ ] Confirm WiiM exact model (Mini / Pro / Amp / Ultra)
- [ ] Confirm Magic Home LED strip exact model
- [ ] Confirm Hue Bridge generation (v2 or v3)
- [ ] Open pending fixture boxes — check pendant bulb socket type
- [ ] Set up git pre-commit hook for yamllint

## Completed

- [x] Set up GitHub repository (github.com/bmitchin/home-automation)
- [x] Create CLAUDE.md with HA-specific guidance
- [x] Create docs/PRINCIPLES.md — guiding principles north star
- [x] Create docs/DESIGN.md — system architecture and phased plan
- [x] Create docs/EQUIPMENT.md — verified equipment inventory
- [x] Update SLZB-06M Core firmware: v2.9.3 → v3.2.0 (2026-03-23)
- [x] Update SLZB-06M Zigbee firmware: 20231030 → 20250220 SDK 8.0.2 (2026-03-23)
- [x] Create device notes for SLZB-06M

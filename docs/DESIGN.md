# System Design

This document captures the architecture, goals, and design decisions for this home automation
project. It is a living document — update it as decisions are made and the system grows.

---

## System Architecture — Two-Server Model

### Production Server (RPI5 8GB — HA OS)

The "bulletproof" instance. Controls everything the household depends on daily.

- **Purpose**: WAF-critical automations — lights, AV, scenes
- **Philosophy**: Stability over features. Never run experimental integrations here.
  If it breaks, real people notice.
- **Update policy**: Wait 2–4 weeks after HA releases before updating. Read breaking
  changes carefully. Always take a backup first.
- **What lives here**: Finalized automations, production Zigbee mesh, Hue bridge
  integration, family room AV, whole-home audio (once validated on sandbox)

### Sandbox Server (hardware TBD — under $500)

The experimental instance. Break things here so production never breaks.

- **Purpose**: Development, integrations in progress, voice assistant, Claude API experiments
- **Philosophy**: Iterate fast. This is where new ideas are validated before promotion
  to production.
- **Candidates**: Mint Box mini PC (currently in use), second RPI5, or equivalent.
  Decision pending — see [Hardware Decision: Sandbox Server](#hardware-decision-sandbox-server).
- **What lives here**: Voice assistant pipeline, new integrations under test, Claude API
  layer, anything experimental

### Promotion Path

```
Sandbox → tested and stable → Production
```

Config changes that prove reliable on the sandbox are migrated to the production instance.
They are not developed directly on production.

---

## Implementation Phases

### Phase 1 — Basement Office (current focus)

**Goal**: Full working proof-of-concept in one room before the remodel begins.

Scope:
- Lighting control (2x Hue bulbs in overhead fixture, LED strips over cabinets and behind TV)
- Whole-home audio — single room starting point (Yamaha IP receiver)
- Voice assistant — local pipeline + Claude API research layer
- Zigbee mesh foundation (radio, sensors)

Success criteria: lights, audio, and voice all working reliably in the basement office.
Everything validated here before touching the remodel spaces.

### Phase 2 — 1st Floor Kitchen & Family Room (post-remodel)

Timing: After the remodel is complete. New electrical, lighting fixtures, and cabling
will be designed with the smart home plan in mind.

Scope:
- Kitchen lighting: Philips Hue Color Ambiance slim downlights (5/6"), Hue Lightstrip
  Flux under-counter, smart switches for pendant fixtures (island + sink)
- Family room lighting: Same Hue downlights, smart outlets for lamps and seasonal use
- Family room AV: Replace receiver with IP-controllable unit; integrate as whole-home
  audio endpoint
- Expand whole-home audio to kitchen and family room (WiiM speakers)
- Presence-based audio routing (audio follows occupancy; do not interrupt active TV viewing)

### Phase 3 — Whole Home Expansion

Scope (in rough priority order):
- Additional rooms with WiiM speakers
- Garage door control
- HVAC integration
- Leak detection
- Plant soil moisture monitoring
- Additional voice satellite deployment

---

## Domain Goals

### Lighting

- **Hue ecosystem** for all controllable fixtures where the switch stays permanently on
  (recessed pucks, overhead fixtures with smart bulbs)
- **Smart switches** for fixtures with dumb bulbs (pendants, standard fixtures) where
  a physical switch must remain functional
- **Smart outlets** as the starting point for lamps and seasonal items (Christmas tree, etc.)
- **Key constraint**: Hue bulbs require constant power. Never put a smart switch on a
  circuit that controls Hue bulbs — use a Hue-compatible switch (e.g., Lutron Aurora,
  Hue Dimmer) that leaves power intact and sends commands over Zigbee instead.

### Whole-Home Audio

- **Zones**: Each room is an independent audio zone
- **Routing**: Presence-aware — audio activates in occupied rooms, deactivates when empty
- **Constraint**: Family room AV is the household's primary entertainment system.
  Automation must never interrupt active TV viewing. Presence detection here requires
  extra care (TV state + occupancy, not occupancy alone).
- **Starting point**: Yamaha IP receiver in basement office (Phase 1)
- **Expansion path**: WiiM speakers in other rooms → new IP-ready receiver in family room
- **Sync**: Both independent zone control and synchronized whole-home playback supported,
  depending on context

### Voice Assistant

- **Architecture**: Hybrid local + cloud
  - **Local pipeline** (Wyoming): handles all device commands — lights, audio, scenes,
    volume. Works offline. No token cost.
  - **Claude API layer**: available on demand for research queries, complex questions,
    and anything that benefits from LLM reasoning. Internet-dependent, token-aware.
    **Not yet implemented** — see the AI Integration Layer section below.
- **Design goal**: local commands feel instant and reliable; Claude layer is invoked
  explicitly or as a fallback, not for every utterance

**Assist pipelines as configured:**

| Pipeline | Speech-to-text | Text-to-speech | Notes |
|---|---|---|---|
| Focused local assistant | `stt.speech_to_phrase` | `tts.piper` — `en_US-lessac-medium` | **In use by both satellites.** Fast; recognizes structured command patterns only |
| Full local assistant | `stt.faster_whisper` | `tts.piper` — `en_US-libritts-high` | Free-form speech; heavier, noticeably slower on the RPI5 |
| Home Assistant | — | — | Default; text/conversation only |

All three use `conversation.home_assistant` as the conversation engine.

- **Speech-to-Phrase over Whisper for daily use.** Whisper is CPU-hungry and the production
  server is an RPI5; a slow reply in a kitchen is worse than no voice control. Whisper stays
  available as a second pipeline and becomes the default once the x86 sandbox lands.
- **Wake word runs on device** (microWakeWord, "Okay Nabu") — no audio leaves the satellite
  until the wake word fires.
- **Voice is a property of the pipeline, not the satellite.** Both satellites share
  "Focused local assistant", so changing the TTS voice changes every room. Per-room voices
  would require a second pipeline.

**Voice satellites**: deployed per room for coverage — currently Office and Kitchen.
See `docs/EQUIPMENT.md` for hardware, flashing procedure and per-unit details.

### AI Integration Layer (Claude Code ↔ HA)

- **HA MCP Server**: HA exposes a Model Context Protocol server at `http://192.168.200.50:8123/api/mcp`
  - Gives Claude Code native HA tools: query entity states, call services, control devices
  - Strictly better than raw REST API calls — structured, typed, intent-aware
  - Claude Code is configured at `~/.claude.json` (user scope) to connect with Long-Lived Token
- **Two directions**:
  - **Claude Code → HA** (current): Claude reads entity states and calls services directly during
    development, troubleshooting, and automation authoring sessions
  - **HA → Claude** (future, Phase 1): HA routes complex voice assistant questions to Claude API
    as a conversation agent — for anything that requires reasoning beyond local pipeline capability
- **Token policy**: HA → Claude direction is invoked explicitly or as a named fallback, not on
  every utterance. Local Whisper + Piper handles all device commands. Claude answers the hard questions.

### Zigbee Mesh

- **Two independent Zigbee networks, by design:**
  - **SLZB-06M → ZHA** — the project's own mesh: THIRDREALITY plugs and temp/humidity sensors.
    ZHA is the authoritative stack. Zigbee2MQTT and Mosquitto are installed but hold zero
    devices; do not treat this as a dual-stack setup.
  - **Hue Bridge v2** — all Philips Hue lighting and wall controls. Hue devices pair to the
    bridge, never to ZHA. This keeps the Hue app working as a fallback, which matters for WAF.
- Smart switches, temp/humidity sensors, and additional devices on the ZHA mesh
- Mesh strength grows as more devices are added — place routers (always-on devices)
  thoughtfully to build good coverage. The THIRDREALITY plugs are mains-powered and act
  as repeaters.

---

## Rooms

### Basement Office (Phase 1)

- ~12×12 room, U-shaped desk on 3 walls, cabinets over desk
- Overhead fixture: 2× Hue smart bulbs (existing, working)
- Under-cabinet lighting: WiFi LED strips (existing, old — replacement candidate)
- Behind-TV bias lighting: WiFi LED strip (existing, old)
- Audio: Yamaha TSR-7810 + WiiM Sound Lite, both in HA
- Occupancy: Apollo MSR-2 mmWave, driving the lighting automations
- Voice: satellite #1 installed
- Status: Phase 1 goals substantially met — lights, audio, occupancy and voice all working

### 1st Floor Kitchen (Phase 2 — partially deployed ahead of remodel)

- Recessed lighting: **installed** — 5× Hue Color Ambiance slim downlights (NW, NE, SW, SE, Sink)
- Wall control: **installed** — Hue dimmer switch
- Sensors: **installed** — THIRDREALITY temp/humidity
- Voice: **satellite #2 installed** (2026-07-26)
- Under-counter: Hue Lightstrip Flux (10ft) — on hand, not installed
- Pendants: 2× over island, 1× over sink — dumb bulbs on smart switches, pending remodel

### 1st Floor Family Room (Phase 2 — partially deployed ahead of remodel)

- Recessed lighting: **installed** — 6× Hue Color Ambiance slim downlights (N, NE, NW, S, SE, SW)
- Wall control: **installed** — Hue tap dial switch
- Lamps: starting with smart outlets, Zigbee/Hue bulbs longer term
- Seasonal: smart outlet (Christmas tree, etc.)
- AV: existing receiver (to be replaced with IP-controllable unit)
- This is the WAF anchor — must be reliable above all else. Changes here are made
  deliberately, and physical controls must keep working regardless of HA state.

> **Phase drift, recorded honestly**: the phase plan reads "Kitchen and Family Room after the
> remodel", but Hue downlights, wall controls and a voice satellite are already in service in
> both rooms. The remaining Phase 2 work is genuinely remodel-dependent (pendants, smart
> switches, under-counter lightstrip, receiver replacement); the lighting was not.

### Other Rooms (Phase 3)

Rooms TBD. Expansion path via WiiM speakers and additional Zigbee devices.

---

## Key Design Decisions

| Decision | Choice | Reason |
|---|---|---|
| Smart lighting ecosystem | Philips Hue via Hue Bridge v2 → HA Hue integration | Reliability, WAF, Hue app remains functional as fallback; Bridge handles Zigbee |
| Pendant/dumb-fixture control | Smart switches (not smart bulbs) | Physical switch must stay functional; Hue bulbs need constant power |
| Lamp/seasonal control | Smart outlets (starting point) | Flexible, no bulb replacement required |
| Audio platform | Yamaha (Phase 1) + WiiM expansion | Yamaha is IP-ready now; WiiM for multi-room |
| Voice pipeline | Speech-to-Phrase + Piper (daily) — Whisper pipeline held in reserve | Offline-capable, token-efficient; Whisper is too slow on the RPI5 for everyday commands |
| Voice satellite firmware | Precompiled `esphome.voice-assistant` image, not a custom build | No build toolchain to maintain; costs the sensor-dock peripherals, which are redundant here |
| Hue room naming | `<Area> Overhead Lights`, renamed at the bridge | A Hue room named after its area collides with area-scoped Assist commands |
| Zigbee vs Z-Wave | Zigbee | Hue is Zigbee; consolidate on one mesh protocol |
| Zigbee stack | ZHA only (Z2M installed but unused) | One stack to reason about; Hue Bridge handles Hue separately |
| Two-server model | Production (RPI5) + Sandbox (TBD) | Protect WAF-critical systems from experimental work |
| AI integration | HA MCP Server + Claude Code (user scope) | Native HA tools in Claude Code; future path for HA voice → Claude API |

---

## Hardware Decision: Sandbox Server

**Status**: Pending — but no longer blocking. Phase 1 voice work shipped on the production RPI5
using Speech-to-Phrase, which is light enough to run there. The sandbox decision now gates
*Whisper* performance (the "Full local assistant" pipeline), not voice control as a whole.

**Options**:
- Mint Box mini PC (currently in use) — available now, no cost, x86 architecture
  (better for Whisper inference)
- Second RPI5 — familiar platform, matches production, ~$100 + accessories
- Other mini PC under $500 — more CPU/RAM for Whisper, future-proof for LLM work

**Key consideration**: Whisper (local speech-to-text) benefits significantly from CPU
performance. x86 will outperform ARM for inference at the same price point. If voice
assistant performance matters, a mini PC with a modern x86 CPU is the stronger choice
over a second RPI5.

---

## Open Questions

- [ ] Pendant light fixtures (delivered, not yet opened) — confirm bulb socket type
      and whether Hue bulbs fit; if not, smart switch approach is confirmed correct
- [ ] Sandbox server hardware decision (x86 mini PC preferred over RPI5 for Whisper performance)
- [ ] Magic Home LED strip exact model (check device label)
- [ ] Vizio E60-C3 SmartCast compatibility — verify with Vizio; likely 2015 = incompatible
- [ ] Kitchen has no presence sensor — the one real gap. A second Apollo MSR-2 is preferred over
      a custom ESPHome build on the voice satellite to expose its dock's mmWave
- [ ] Individual Kitchen/Family Room downlights carry no HA area — only the Hue Room group does
- [x] WiiM exact model — WiiM Sound Lite
- [x] Hue Bridge generation — v2 square (model 3241312018A); use Hue Bridge → HA Hue integration (not direct ZHA pairing)

## Resolved

- [x] Yamaha receiver model — TSR-7810; uses YNCA + MusicCast; excellent HA integration via yamaha_ynca (HACS)
- [x] WiFi LED strips — Magic Home brand; integrates via flux_led (official); no flash needed
- [x] Voice satellite hardware — ESP32-S3-BOX-3B ×2; both deployed (Office 2026-04-04, Kitchen 2026-07-26) on firmware 26.6.1
- [x] Zigbee radio — SMLIGHT SLZB-06M; use ZHA (not Z2M); firmware update required first
- [x] Family room receiver (Sony STR-DN2010) — 2010 model, no HA integration; replacement planned
- [x] TiVo IPA1114HDW — confirmed Android TV streaming device (not cable box); integrates via Android TV Remote
- [x] SLZB-06M firmware — Core v3.2.4 + Zigbee 20250220 SDK 8.0.2; USB reflash via smlight.tech/flasher resolved unresponsive EFR32MG21 (2026-03-30)
- [x] HA API access — Long-Lived Token generated; MCP server integration planned; REST API confirmed working
- [x] SMLIGHT integration — added to HA; monitors coordinator health, firmware, temps (2026-03-23)
- [x] ZHA integration — active; socket://192.168.200.232:6638, EZSP, flow_control=software (2026-03-30)
- [x] Zigbee stack — ZHA is authoritative (111 entities); Zigbee2MQTT + Mosquitto installed but hold zero devices (2026-07-26)
- [x] Assist pipeline choice — Speech-to-Phrase for daily use, Whisper held in reserve until the x86 sandbox lands (2026-07-26)
- [x] Hue room naming convention — `<Area> Overhead Lights`, renamed at the bridge, to avoid Assist collisions (2026-07-26)
- [x] Voice satellite sensor dock — stand and power only; stock firmware exposes none of its sensors (2026-07-26)

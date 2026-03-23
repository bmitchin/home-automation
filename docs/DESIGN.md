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
  - **Local pipeline** (Whisper + Piper via Wyoming protocol): handles all device
    commands — lights, audio, scenes, volume. Works offline. No token cost.
  - **Claude API layer**: available on demand for research queries, complex questions,
    and anything that benefits from LLM reasoning. Internet-dependent, token-aware.
- **Voice satellites**: deployed per room for coverage
- **Design goal**: local commands feel instant and reliable; Claude layer is invoked
  explicitly or as a fallback, not for every utterance

### Zigbee Mesh

- Zigbee radio connected to HA (hardware TBD — see equipment list)
- Smart switches, temp/humidity sensors, and additional devices on the Zigbee mesh
- Mesh strength grows as more devices are added — place routers (always-on devices)
  thoughtfully to build good coverage

---

## Rooms

### Basement Office (Phase 1)

- ~12×12 room, U-shaped desk on 3 walls, cabinets over desk
- Overhead fixture: 2× Hue smart bulbs (existing, working)
- Under-cabinet lighting: WiFi LED strips (existing, old — replacement candidate)
- Behind-TV bias lighting: WiFi LED strip (existing, old)
- Audio: Yamaha IP-ready receiver (connected to network)
- Voice: satellite(s) on order

### 1st Floor Kitchen (Phase 2 — post-remodel)

- New construction: electrical and cabling to be designed around smart home plan
- Recessed lighting: Philips Hue Color Ambiance slim downlights (5/6") + Hue Bridge
- Under-counter: Hue Lightstrip Flux (10ft)
- Pendants: 2× over island, 1× over sink — dumb bulbs on smart switches

### 1st Floor Family Room (Phase 2 — post-remodel)

- Recessed lighting: Philips Hue Color Ambiance slim downlights (5/6")
- Lamps: starting with smart outlets, Zigbee/Hue bulbs longer term
- Seasonal: smart outlet (Christmas tree, etc.)
- AV: existing receiver (to be replaced with IP-controllable unit)
- This is the WAF anchor — must be reliable above all else

### Other Rooms (Phase 3)

Rooms TBD. Expansion path via WiiM speakers and additional Zigbee devices.

---

## Key Design Decisions

| Decision | Choice | Reason |
|---|---|---|
| Smart lighting ecosystem | Philips Hue (Zigbee) | Reliability, WAF, wide fixture compatibility |
| Pendant/dumb-fixture control | Smart switches (not smart bulbs) | Physical switch must stay functional; Hue bulbs need constant power |
| Lamp/seasonal control | Smart outlets (starting point) | Flexible, no bulb replacement required |
| Audio platform | Yamaha (Phase 1) + WiiM expansion | Yamaha is IP-ready now; WiiM for multi-room |
| Voice pipeline | Wyoming/Whisper + Piper (local) + Claude API (research) | Offline-capable, token-efficient, fast for commands |
| Zigbee vs Z-Wave | Zigbee | Hue is Zigbee; consolidate on one mesh protocol |
| Two-server model | Production (RPI5) + Sandbox (TBD) | Protect WAF-critical systems from experimental work |

---

## Hardware Decision: Sandbox Server

**Status**: Pending. Decision needed before Phase 1 voice work begins.

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
      and whether Hue bulbs fit; if not, confirm smart switch approach
- [ ] Sandbox server hardware decision
- [ ] Yamaha receiver model — confirm API/control method (MusicCast, YNCA, or other)
- [ ] WiFi LED strips in basement office — confirm brand/model; assess whether to
      integrate or replace with Zigbee/Hue-compatible strips
- [ ] Voice satellite hardware — confirm model when they arrive
- [ ] Zigbee radio model — confirm (impacts which HA integration to use: ZHA vs Zigbee2MQTT)

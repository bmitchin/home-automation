# Device Notes — Zigbee Coordinator — SMLIGHT SLZB-06M

Home Assistant side only.

> **Hardware facts, firmware history, the device API, factory reset, USB
> recovery and web-UI automation are in `my-hardware/devices/slzb-06m.md`.**
> This file covers how HA talks to the coordinator and why it is configured the
> way it is.

---

## ZHA Configuration

| Setting | Value | Reason |
|---|---|---|
| HA Integration | ZHA (not Zigbee2MQTT) | More stable with the SLZB-06M; simpler config |
| ZHA socket | `socket://192.168.200.232:6638` | Network coordinator address; 6638 is the SLZB default |
| ZHA radio type | EZSP | Required for EFR32MG21 + SDK 8.0.2 firmware |
| Flow control | `software` | Active as of 2026-03-30 |
| Zigbee role | Coordinator | Required for ZHA |
| `usePackets` | ON | Required for ZHA — cleared by factory reset, always re-apply |
| SMLIGHT integration | Added separately | Monitors coordinator health, firmware, LED and uptime |
| Connection mode | WiFi client | Ethernet not yet cabled to this location |

### Why EZSP and not Zigbee2MQTT

The 20250220 firmware uses SDK 8.0.2 and requires `adapter: ember` in a
Zigbee2MQTT config. ZHA handles this natively via EZSP with no extra
configuration. For a new setup with no existing Z2M config, ZHA is the right
choice and avoids the stability issues reported with this coordinator under Z2M
before the firmware fix.

### Why the firmware update was not optional

The factory Zigbee firmware (20231030) made Z2M produce "Frame(s) in progress
cancelled" errors and needed weekly reboots. 20250220 (SDK 8.0.2) resolved it.
The Core update (v2.9.3 → v3.2.0) was needed for OTA stability and to support
the newer Zigbee firmware properly.

### `usePackets` must be ON before adding ZHA

A factory reset clears it. Re-apply it *before* configuring ZHA or the
integration will not work:

```bash
curl -s -X POST "http://192.168.200.232/settings/saveParams" -d "pageId=3&usePackets=on"
# Response must show: "changes":{"usePackets":true},"needReboot":true
curl -s "http://192.168.200.232/api2?action=1&param=reboot"
```

### The Zigbee version reads "Unknown" — this is expected

After flashing 20250220 (SDK 8.0.2), the web UI, the device API and
`sensor.slzb_06m_zigbee_type` all report the Zigbee version as "Unknown" or
`[ZB_FW_unk]`. The firmware reports its version over EZSP rather than direct
UART queries. Confirmed still reading `unknown` on 2026-08-23 with the
coordinator working normally. It is cosmetic — do not chase it.

---

## Open Items

- [x] Add the SMLIGHT integration to HA (Settings → Devices & Services → SMLIGHT)
- [x] Add ZHA to HA (`socket://192.168.200.232:6638`, EZSP, `flow_control=software`) — 2026-03-30
- [ ] Enable "Automatic Zigbee Update" once ZHA is stable (currently Off)
- [ ] Assign a static IP via DHCP reservation on the router
- [ ] Switch to Ethernet once office cabling is in place — more reliable than WiFi

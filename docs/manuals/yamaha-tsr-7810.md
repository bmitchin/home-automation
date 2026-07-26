# Yamaha TSR-7810 — Reference Notes

## Manual

- **Official PDF**: https://usa.yamaha.com/files/download/other_assets/3/803723/TSR-7810_Manual_English.pdf
- **Online viewer**: https://manual.yamaha.com/av/17/tsr7810/en/

## Known IP Address

- Current IP: 192.168.200.41 (DHCP — assign static reservation)
- Use MusicCast app to find IP if it changes

## Finding the IP Address

On the receiver front panel:
**SETUP → Network → IP Address**

Or check your router DHCP table for a device named "Yamaha" or "TSR-7810".

## Network Setup (Manual p.114)

- Default: DHCP on (receiver gets IP automatically from router)
- To set static IP on the receiver: SETUP → Network → IP Address → set DHCP to Off, then enter IP manually
- Recommended: leave DHCP on and assign a static reservation at the router instead

## HA Integration

- **Integration**: `yamaha_ynca` (HACS)
- **Protocol**: YNCA over TCP (port 50000)
- **MusicCast**: Also supported — yamaha_ynca uses YNCA which gives deeper control
- **GitHub**: https://github.com/mvdwetering/yamaha_ynca

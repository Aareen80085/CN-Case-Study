# Meridian College: Campus LAN Segmentation, Trunking & STP Redesign

Case Study No. 2 for Computer Networks. This repository contains the Packet Tracer
lab, the screenshot evidence, and a script that builds the technical report as a Word document.

## Contents

| File / folder | Purpose |
|---|---|
| `CN case study.pkt` | Cisco Packet Tracer 9.0.1 project (the full network) |
| `generate_report.py` | python-docx script that builds the report |
| `Meridian_College_Case_Study_2_Report.docx` | Generated report (output) |
| `screenshots/` | Evidence images referenced as Figures 1 to 7 |

## Topology

```
                    R1 (Gi0/0/0, ROAS sub-interfaces)
                     |
                 SW-Admin  ---- Admin-PC
                 (root, 4096)
            Po1 /          \ Fa0/24
   (2x Gig, LACP)           \
        SW-Academic ------- SW-Library
        (8192)    Fa0/23   Fa0/24 (blocked, alternate)
        /     \                /    \
  Faculty-PC  Student-PC   Laptop0  Guest-Kiosk
```

Devices: 3 x Catalyst 2960-24TT, 1 router (R1), 5 end hosts.

## Design summary

| Area | Choice |
|---|---|
| VLANs | 10 Faculty, 20 Students, 30 Admin, 40 IT-Mgmt, 900 Guest, 998 Parking, 999 Native-Unused |
| Addressing | 10.10.10.0/24, 10.10.20.0/24, 10.10.30.0/24, 10.10.40.0/24, 192.168.90.0/24 (gateway `.1` on each) |
| Trunking | 802.1Q, native VLAN 999, allowed list `10,20,30,40,900` |
| EtherChannel | LACP (`mode active`), Port-channel 1, SW-Admin Gi0/1-2 to SW-Academic Gi0/1-2 |
| Routing | Router-on-a-stick, R1 Gi0/0/0.10 to Gi0/0/0.900 |
| Spanning tree | Rapid PVST+; SW-Admin 4096, SW-Academic 8192, SW-Library 32768 |
| Edge security | SW-Library Fa0/13: max 2 MACs, sticky, violation `restrict` |
| VTP | Transparent on all switches, manual trunk pruning |

## Requirements

- Cisco Packet Tracer 9.0.1 (to open the `.pkt`)
- Python 3.8+
- `python-docx`

```bash
pip install python-docx
```

## Building the report

1. Put the screenshots in a folder (for example `screenshots/`).
2. Open `generate_report.py` and edit the configuration block at the top:
   - `STUDENT_NAME`, `STUDENT_ID`, `COURSE`, `REPORT_DATE`
   - Port ranges that are not proven by a screenshot: `ADMIN_R1_PORT`, `ADMIN_ACCESS`,
     `ADMIN_PARK`, `LIB_ACCESS`, `LIB_PARK`
3. Run:

```bash
python generate_report.py screenshots
```

Output: `Meridian_College_Case_Study_2_Report.docx` in the current directory.

The script finds images whether they are named `Screenshot 2026-10-06 at 9.35.10 PM.png`
(macOS style) or `Screenshot_2026-10-06_at_9_35_10_PM.png`, as `.png` or `.jpg`. If one is
missing, it prints a warning and leaves a red placeholder in the document.

### Figure map

| Figure | Screenshot time | Content |
|---|---|---|
| 1 | 9.35.10 PM | SW-Academic `show vlan brief` |
| 2 | 9.37.32 PM | SW-Admin `show etherchannel summary` |
| 3 | 9.37.01 PM | Faculty-PC ping to 10.10.30.10 |
| 4a | 9.42.53 PM | Baseline topology |
| 4b | 9.43.26 PM | SW-Library `show spanning-tree vlan 20` (baseline) |
| 5 | 9.25.47 PM | SW-Library `show port-security interface fa0/13` |
| 6 | 9.25.29 PM | Topology with severed link |
| 7 | 9.16.11 PM | SW-Library CLI showing failover to Fa0/24 |

## Reproducing the lab tests

Open the `.pkt` and use these commands.

| Test | Device | Command |
|---|---|---|
| VLAN database | SW-Academic | `show vlan brief` |
| Trunks and native VLAN | SW-Admin | `show interfaces trunk` |
| EtherChannel | SW-Admin | `show etherchannel summary` |
| Gateways | R1 | `show ip interface brief` |
| Routing | Faculty-PC | `ping 10.10.30.10` (first echo times out while ARP resolves; replies show TTL=127) |
| STP baseline | SW-Library | `show spanning-tree vlan 20` (Fa0/23 Root FWD, Fa0/24 Altn BLK, root cost 19) |
| Port security | SW-Library | `show port-security interface fa0/13` |
| Failover | Delete the SW-Admin Fa0/24 to SW-Library Fa0/23 cable, then run the STP command again | Fa0/24 becomes Root FWD, root cost 22 |

## Known limitations

- Only SW-Academic's port map was verified against a screenshot. SW-Admin and SW-Library ranges in the report are assumptions until checked.
- VLAN 900 (Guest) is routed with no ACL, so guests can reach internal subnets at Layer 3.
- Packet Tracer does not timestamp STP transitions, so the report makes no exact convergence time claim.
- Running configs must be saved (`copy running-config startup-config`) before closing the `.pkt`, or the changes are lost on reload.

## Author

[Your Name], [Student ID], October 2026

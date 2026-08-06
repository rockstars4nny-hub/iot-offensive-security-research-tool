# IoT Offensive Security Research Tool

**Aetherverse Intelligence Protocol (AIP)**  
Authorized camera / drone / embedded assessment toolkit — UART, firmware, network ports, MAVLink, BLE.

| | |
|--|--|
| **Tool** | `iot_scanner` v1.1.0 |
| **License** | MIT (see `license`) |
| **Safety** | Devices you **own** or have **explicit written permission** to test |

Companion fabric UI: **Finch Glass** in private [PT](https://github.com/rockstars4nny-hub/PT).

---

## What this repo is

A Python CLI that produces **structured JSON findings** across six modules:

1. **UART** — auto-baud detection + shell-access probe  
2. **Firmware dump** — pull image over UART shell when available  
3. **Firmware analysis** — binwalk + credential/secret string scan  
4. **Network discovery** — common IoT/camera/drone TCP/UDP service ports  
5. **MAVLink** — signing / plaintext telemetry checks; optional benign command-acceptance test  
6. **BLE** — advertising discovery + GATT enum; readable-characteristic credential scan  

Entry points in-tree:

- `iot-scanner` — primary CLI (v1.1)  
- `File` — earlier variant of the same tool family  

---

## Install

```bash
python3 -m venv venv && source venv/bin/activate
pip install pyserial pymavlink bleak
# optional: binwalk on PATH for --analyze
chmod +x iot-scanner
```

---

## Usage

```bash
# UART baud scan + shell probe
python iot-scanner --uart /dev/ttyUSB0 --baud-scan

# Firmware offline analysis
python iot-scanner --firmware firmware.bin --analyze

# Network surface on a device you own
python iot-scanner --ip 192.168.1.100 --scan

# MAVLink listen / assess
python iot-scanner --mavlink-connection udp:0.0.0.0:14550 --mavlink

# BLE advertise scan (+ optional GATT enum)
python iot-scanner --ble-scan
python iot-scanner --ble-address AA:BB:CC:DD:EE:FF --ble-enum

# JSON out
python iot-scanner --ip 192.168.1.100 --scan --output report.json --quiet
```

`--all` runs every module applicable to the targets you supplied.  
`--confirm-injection-test` enables the **benign** MAVLink command-acceptance probe — controlled lab only.

---

## Port catalog (network module)

Includes RTSP (`554`/`8554`/`7070`), MQTT (`1883`/`8883`), HTTP family, Telnet/SSH/FTP, MAVLink TCP/UDP (`5760`, `14550`, `18570`), and common IoT debug ports. Full map is in `COMMON_PORTS` inside `iot-scanner`.

---

## Stack with Finch Glass

For LAN-wide Find IoT / MQTT listen / SENTINEL handoff:

```bash
git clone https://github.com/rockstars4nny-hub/PT.git
cd PT && ./update.sh --start
# Glass → Find IoT / IoT Vulns on an authorized CIDR
```

IoT Attack Surface Pack copy: [docs/aip-iot-sku-x-gumroad.md](https://github.com/rockstars4nny-hub/PT/blob/main/finch/docs/aip-iot-sku-x-gumroad.md)

---

## Disclaimer

Authorized security research and internal assessments only. Do **not** use against devices or networks without ownership or written authorization. AIP is not responsible for misuse.

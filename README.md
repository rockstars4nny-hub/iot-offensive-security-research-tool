# iot-offensive-security-research-tool

# iot-security-scanner

Security assessment tool for IoT cameras and drones/UAVs. Built for pre-shipment
hardware security testing: UART console access, firmware extraction and secret
scanning, network exposure, and MAVLink telemetry/command-injection checks.

> **Authorized testing only.** Do not use on devices you do not own or do not
> have explicit written permission to test.

## What it does

| Module | Checks |
|---|---|
| UART | Auto-detects console baud rate (9600–230400), probes for an unauthenticated shell |
| Firmware | `binwalk` filesystem ID + string-pattern scan for hardcoded keys/passwords/cloud endpoints |
| Network | Port sweep across common IoT/camera/drone services (SSH, Telnet, RTSP, MAVLink, MQTT) |
| MAVLink | MAVLink2 signing verification, plaintext GPS/battery telemetry check, benign command-acceptance test |

Findings are emitted as structured JSON, severity-tagged (`CRITICAL` → `INFO`),
each with a remediation note.

## Install

```bash
sudo apt install binwalk
pip install pyserial pymavlink
```

Both `pyserial` and `pymavlink` are optional — the tool degrades gracefully
(logs an `INFO` finding) if either is missing, rather than crashing.

## Usage

```bash
python3 iot_scanner.py --uart /dev/ttyUSB0 --baud-scan
python3 iot_scanner.py --firmware firmware.bin --analyze
python3 iot_scanner.py --ip 192.168.1.100 --scan
python3 iot_scanner.py --mavlink-connection udpin:0.0.0.0:14550 --mavlink
python3 iot_scanner.py --all --output results.json
```

`--quiet` for JSON-only stdout (pipe into other tooling), `--verbose` for
progress output on stderr.

## A few design notes

- **UART baud detection** doesn't just try every rate and hope — it checks
  response printability ratio and known shell/bootloader signatures
  (`BusyBox`, `U-Boot`, login prompts) to tell a correct baud lock from line
  noise.
- **MAVLink signing check** reads the MAVLink2 incompatibility-flag byte
  directly off the wire (`0xFD` magic byte, flag `0x01`) rather than trusting
  connection metadata, since that's the actual on-wire signal of whether
  signing is enforced.
- **The command-injection test is intentionally non-destructive.** It sends
  `MAV_CMD_REQUEST_MESSAGE` — a benign, non-actuating command — to prove
  whether the link accepts unauthenticated commands at all, without ever
  touching arm/disarm, mode, or movement commands against a real aircraft.
  It's gated behind `--confirm-injection-test` since it actively transmits to
  a flight controller.
- **Firmware string scanning** has a pure-Python fallback if the `strings`
  binary isn't available, so the tool doesn't hard-fail on minimal systems.

## Testing without hardware

The UART and MAVLink modules can be exercised without real hardware:

- UART: pair a virtual serial device with `socat pty,link=/tmp/ttyFAKE0
  pty,link=/tmp/ttyFAKE1`, then run a small Python responder on one end that
  mimics an unauthenticated BusyBox shell.
- MAVLink: use `pymavlink` to send unsigned HEARTBEAT/GPS_RAW_INT/
  BATTERY_STATUS messages over loopback UDP and point `--mavlink-connection`
  at the listening port.

## License

MIT — see [LICENSE](LICENSE).

## About

Built by **AIP** — Web3/smart contract security, AI/LLM red teaming, and
hardware/IoT security research. Portfolio: [github.com/rockstars4nny-hub](https://github.com/rockstars4nny-hub)

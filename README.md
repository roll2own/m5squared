# m5squared aka. wheelchair.py

**Your wheelchair, your rules.**

Python toolkit for the Alber e-motion M25 power-assist wheels. Because paying €595 for a Bluetooth remote that does less than a Python script is absurd.

Presented at 39C3 Hamburg: *["Pwn2Roll: Who Needs a 595€ Remote When You Have wheelchair.py?"](https://media.ccc.de/v/39c3-pwn2roll-who-needs-a-599-remote-when-you-have-wheelchair-py)*

## What This Does

- **Decrypt the "encrypted" Bluetooth protocol:** AES-128-CBC, nothing fancy
- **Replace the €595 ECS remote:** Same features, zero cost, more control
- **Bypass all of the in-app purchases:** All the "premium" features, free
    - Currently we support out-of-the-box: ECS remote, speed increase to 8.5km/h, parking mode. *Cruise* mode and the push counter are possible in theory, but not tested yet. Stay tuned, this will be tested soonish (it's also the ground work for replacing the knob-style remote).
- **Access dealer-only parameters:** Your wheels, your data

## The €595 Remote vs. This Script

| Feature                 | Official ECS Remote | wheelchair.py |
| ----------------------- | ------------------- | ------------- |
| Price                   | €595                | Free          |
| Read battery            | Yes                 | Yes           |
| Change assist level     | Yes                 | Yes           |
| Toggle hill hold        | Yes                 | Yes           |
| Read raw sensor data    | No                  | Yes           |
| Adjust drive parameters | No                  | Yes           |
| Works on Linux          | No                  | Obviously     |

## Quick Start

```bash
# Setup (Ubuntu/Debian)
sudo apt install python3.12-venv python3-bluez
python3 -m venv .venv --system-site-packages
.venv/bin/pip install -e .
source .venv/bin/activate

# Get your AES keys (scan the QR codes on your wheel hubs)
python m25_qr_to_key.py "YourQRCodeHere"

# Talk to your wheels
python m25_ecs.py --left-addr AA:BB:CC:DD:EE:FF --right-addr 11:22:33:44:55:66 \
                  --left-key HEXKEY --right-key HEXKEY
```

## Tools

| Script             | What it does                                      | 
| ------------------ | ------------------------------------------------- |
| `m25_qr_to_key.py` | QR code → AES key (their encoding is... creative) |
| `m25_ecs.py`       | The main event: read status, change settings      |
| `m25_decrypt.py`   | Decrypt captured Bluetooth packets                |
| `m25_encrypt.py`   | Encrypt packets for transmission                  |
| `m25_analyzer.py`  | Make sense of the packet soup                     |
| `m25_parking.py`   | Remote movement control (use responsibly)         |
| `m25_bluetooth.py` | Scan, connect, send/receive                       |

## Getting Your Keys

Each wheel has a QR code sticker. That's your key to the kingdom.

1. Scan the QR code (22 characters of proprietary encoding)
2. Run: `python m25_qr_to_key.py "ABCD1234..."`
3. Get a 16-byte hex key
4. Use it with `--left-key` / `--right-key`

Both wheels have different keys. Yes, you need both.

## Protocol TL;DR

Bluetooth SPP on channel 6. Packets look like:

```
[0xEF] [length:2] [IV encrypted with ECB:16] [payload encrypted with CBC:n] [CRC:2]
```

Why encrypt the IV with ECB first? Nobody knows. But it works.

## Requirements

- Python 3.8+
- `pycryptodome` - For the crypto
- `bluez` / `python3-bluez` - For the Bluetooth

## Legal Stuff

This is for **your own wheels**. Don't be creepy.

- Research and education: Yes
- Your own devices: Yes
- Other people's wheelchairs: Absolutely not

## Links

- Protocol docs: [`doc/` directory](https://github.com/roll2own/m5squared/tree/main/doc)
- Supplementary resources (media, manuals, talk materials and other non-codey stuff: [m5squared-resource repo](https://github.com/roll2own/m5squared-resources)
- 39C3 Talk: *["Pwn2Roll: Who Needs a 595€ Remote When You Have wheelchair.py?"](https://media.ccc.de/v/39c3-pwn2roll-who-needs-a-599-remote-when-you-have-wheelchair-py)*

### Media Coverage

- [[DE] Warum eine Multiple-Sklerose-Erkrankte ihren Rollstuhl hackte](https://www.spiegel.de/netzwelt/hackerkonferenz-39c3-warum-eine-multiple-sklerose-erkrankte-ihren-rollstuhl-hackte-a-169573b3-dbc9-4aed-bc33-1cc39cee6971)
- [[DE] Wie eine Multiple-Sklerose-Erkrankte die Paywall ihres Rollstuhls knackte](https://www.derstandard.at/story/3000000302326/wie-eine-multiple-sklerose-erkrankte-die-paywall-ihres-rollstuhls-knackte)
- [[DE] 39C3: Rollstuhl-Security – Wenn ein QR-Code alle Schutzmechanismen aushebelt](https://www.heise.de/news/39C3-Rollstuhl-Security-Wenn-ein-QR-Code-alle-Schutzmechanismen-aushebelt-11126816.html)
- [[EN] Louis Rossmann: Wheelchairs have paywalls and digital locks now (YouTube video)](https://www.youtube.com/watch?v=5yWcXPDJQ7k)

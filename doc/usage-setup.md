# Setup Guide

Getting this running is easier than convincing your health insurance to pay for the ECS remote.

## Ubuntu/Debian

### Install The Stuff

```bash
sudo apt install python3.12-venv python3-bluez bluez

# Make sure these exist
which rfcomm hcitool bluetoothctl
```

### Python Setup

```bash
# Virtual env with system packages (bluez needs this)
python3 -m venv .venv --system-site-packages

# Install the rest
.venv/bin/pip install -r requirements.txt

# Activate
source .venv/bin/activate

# Sanity check
python3 -c "from Crypto.Cipher import AES; print('Crypto: OK')"
python3 -c "import bluetooth; print('Bluetooth: OK')"
```

### Bluetooth Permissions

Linux and Bluetooth permissions. A tale as old as time.

```bash
# Option 1: Just sudo it
sudo python3 m25_bluetooth.py scan

# Option 2: Add yourself to the club (needs re-login)
sudo usermod -aG bluetooth $USER
```

## Finding Your Wheels

```bash
# They show up as "M25V1_..."
sudo python3 m25_bluetooth.py scan --m25
```

Write down the MAC addresses. You'll need both.

## Getting Your Keys

Each wheel has a QR code sticker. The string it contains is called the "Cyber Security Key". Very dramatic.

1. Scan it with your phone (any QR app works)
2. Get a 22-character string that looks like gibberish
3. Convert it:

```bash
python3 m25_qr_to_key.py "YourQRCodeString"
# Outputs: 20a61dd47a91f690d93ca601d113806c
```

Do this for both wheels. Yes, they have different keys.

## Connecting

```bash
# Bind the RFCOMM device
sudo python3 m25_bluetooth.py connect AA:BB:CC:DD:EE:FF

# Check it worked
python3 m25_bluetooth.py status

# When you're done
python3 m25_bluetooth.py disconnect
```

## Test It

```bash
# Dry run (no Bluetooth needed, just generates packets)
python3 m25_ecs.py --left-key HEXKEY --right-key HEXKEY --dry-run

# Decrypt a packet you captured
echo "ef0024..." | python3 m25_decrypt.py -k YOUR_HEX_KEY
```

If you see sensible output, you're in business.

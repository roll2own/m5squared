# M25 Protocol Documentation

How to talk to wheelchair wheels. It's not rocket science, but it's not exactly simple either.

## The Big Picture

Bluetooth SPP (Serial Port Profile) on channel 6, wrapped in AES-128-CBC encryption.
Nothing exotic, but they did add some... creative choices.

## Packet Structure

### What Goes Over The Wire

```
+--------+--------+--------+------------------+------------------+--------+--------+
|  0xEF  | LEN_HI | LEN_LO |  Encrypted IV    | Encrypted Data   | CRC_HI | CRC_LO |
|  (1)   |  (1)   |  (1)   |      (16)        |    (variable)    |  (1)   |  (1)   |
+--------+--------+--------+------------------+------------------+--------+--------+
```

- **0xEF**: Every packet starts with this. Exciting.
- **Length**: Big-endian, bytes after the header
- **Encrypted IV**: 16 bytes, encrypted with ECB (yes, really)
- **Encrypted Data**: The actual payload, CBC encrypted
- **CRC-16**: Just in case the encryption wasn't enough

### Byte Stuffing

Because 0xEF can appear in data too:
- `EF` in payload → `EF EF` on wire
- Single `EF` = packet header
- Double `EF` = just the byte 0xEF

## What's Inside (Decrypted)

```
+----------+----------+--------+--------+----------+----------+------------+
| Proto ID | Telegram | Source | Dest   | Service  | Param    | Payload    |
|   (1)    |   ID (1) | ID (1) | ID (1) |  ID (1)  |  ID (1)  | (variable) |
+----------+----------+--------+--------+----------+----------+------------+
```

Six bytes of header, then whatever data the command needs.

### Who's Talking

| ID | Device | Notes |
|----|--------|-------|
| 0x01 | M25 Wheel (broadcast) | Both wheels listen |
| 0x02 | Left Wheel | |
| 0x03 | Right Wheel | |
| 0x04 | ECS Remote | The €595 one |
| 0x05 | Smartphone App | That's us now |
| 0x06 | Knob Remote | |
| 0x07 | USB Service Tool | Dealer stuff |

### What They're Talking About

| ID | Service | Purpose |
|----|---------|---------|
| 0x01 | APP_MGMT | Drive modes, assist levels, the fun stuff |
| 0x02 | BATT_MGMT | Battery state |
| 0x03 | VERSION_MGMT | Firmware versions |
| 0x04 | RTC | Real-time clock |
| 0x05 | STATISTICS | Obviously stats |
| 0x06 | MEMORY_MGMT | Raw config memory |
| 0x08 | TANDEM | For paired wheelchairs |

### The Commands You'll Actually Use

#### APP_MGMT (0x01)

| ID | What | Direction |
|----|------|-----------|
| 0x10 | WRITE_SYSTEM_MODE | → Wheel |
| 0x20 | WRITE_DRIVE_MODE | → Wheel |
| 0x21 | READ_DRIVE_MODE | → Wheel |
| 0x22 | STATUS_DRIVE_MODE | ← Wheel |
| 0x30 | WRITE_REMOTE_SPEED | → Wheel |
| 0x40 | WRITE_ASSIST_LEVEL | → Wheel |
| 0x41 | READ_ASSIST_LEVEL | → Wheel |
| 0x42 | STATUS_ASSIST_LEVEL | ← Wheel |

#### BATT_MGMT (0x02)

| ID | What |
|----|------|
| 0x01 | READ_SOC |
| 0x02 | STATUS_SOC |

## The Encryption

### Key Derivation (The Fun Part)

The QR code on each wheel contains your AES key. Encoded in their own special way.

1. 22 characters from a custom 64-character alphabet
2. Each char = 6 bits (22 × 6 = 132 bits)
3. Drop the first 4 bits → 128 bits → AES key

Why not just print the hex key? Where's the fun in that?

### Encrypting a Packet

1. Generate random 16-byte IV
2. Encrypt IV with AES-ECB → Encrypted IV (why? nobody knows)
3. PKCS7-pad the payload
4. Encrypt with AES-CBC using the IV
5. Calculate CRC-16 over the *original* payload
6. Assemble: Header + Encrypted IV + Encrypted Data + CRC

### Decrypting a Packet

1. Strip header, get length
2. Decrypt IV with AES-ECB
3. Decrypt payload with AES-CBC
4. Remove PKCS7 padding
5. Verify CRC-16
6. Parse the SPP data

## CRC-16

Polynomial 0x8005, init 0xFFFF. Standard stuff. See `m25_protocol.py` for the lookup table.

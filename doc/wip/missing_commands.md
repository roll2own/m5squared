# Missing Protocol Implementations

Based on `adb logcat` analysis with several tests, 2026-01-02.
Several tests using Cruise Mode, Dealer Mode features, on/off via ECS Remote feature.


---

## TX Commands not implemented ATM

### Service 0x01 (APP_MGMT)

#### READ_MOUNTING_SIDE (0x71)

Returns which side the DuoDrive remote is mounted on

```
TX: 05 01 01 71
RX: 05 01 72 [side]
    side: 0x01 = RIGHT, 0x02 = LEFT
```

#### READ_AUTO_SHUTOFF_TIME (0x81)

Returns auto power-off timeout in seconds

```
TX: 05 01 01 81
RX: 05 01 82 [time_hi] [time_lo]
    time: uint16_be, seconds (default 3600 = 1 hour)
```

#### WRITE_AUTO_SHUTOFF_TIME (0x80)

Sets auto power-off timeout (seconds)

```
TX: 05 01 01 80 [time_hi] [time_lo]
RX: 05 01 ff 00 (ACK)
```

#### READ_FACTORY_RESET_STATE (0xA1)

Returns factory reset status

```
TX: 05 01 01 a1
RX: 05 01 a2 [state]
    state: 0x00 = NOT_STARTED
           0x01 = IN_PROGRESS
           0x02 = FINISHED
           0x03 = ERROR
```

#### START_FACTORY_RESET (0xA0)

Initiates factory reset of drive parameters, profiles, buzzer, LEDs (via Dealer Mode, otherwise via button on back of ECS remote)

```
TX: 05 01 01 a0
RX: 05 01 ff 00 (ACK)
```

#### READ_DUO_DRIVE_PARAMS (0xF1)

Returns DuoDrive parameters
Note: "Mounting Side" is the mounting side of the DuoDrive remote (`CU` - Control Unit)

```
TX: 05 01 01 f1
RX: 05 01 f2 [mounting_side] [speed_sensibility] [steering_dynamic]
    mounting_side: 0x00 = UNKNOWN, 0x01 = RIGHT, 0x02 = LEFT
    speed_sensibility: uint8
    steering_dynamic: uint8 enum
```

#### WRITE_DUO_DRIVE_PARAMS (0xF0)

Sets DuoDrive parameters (Dealer Mode setting)
Note: "Mounting Side" is the mounting side of the DuoDrive remote (`CU` - Control Unit)

```
TX: 05 01 01 f0 [mounting_side] [speed_sensibility] [steering_dynamic]
RX: 05 01 ff 00 (ACK)
```

#### ANNOUNCE_DISCONNECT (0x10 with payload 0x01)

Proper disconnect sequence. Different from WRITE_SYSTEM_MODE implementation. #TODO needs side-by-side verification

```
TX: 05 01 01 10 01
RX: 05 01 ff 00 (ACK)
```

Note: WRITE_SYSTEM_MODE uses same param 0x10 but with payload 0x01 (connect) or 0x02 (standby)
ANNOUNCE_DISCONNECT specifically uses payload 0x01 in disconnect context

TODO do a clean log of standy/shutdown sequences
---

### Service 0x02 (ACTOR_BUZZER)

#### READ_BUZZER_VOL (0x11)

Returns buzzer volume setting

```
TX: 05 01 02 11
RX: 05 02 12 [volume]
    volume: 0x00 = MIN, 0x01 = MAX
```

#### WRITE_BUZZER_VOL (0x10)

Sets buzzer volume

```
TX: 05 01 02 10 [volume]
RX: 05 02 ff 00 (ACK)
```

#### READ_BUZZER_ON_OFF (0x21)

Returns buzzer enabled state

```
TX: 05 01 02 21
RX: 05 02 22 [state]
    state: 0x00 = OFF, 0x01 = ON
```

#### WRITE_BUZZER_ON_OFF (0x20)

Enables or disables buzzer

```
TX: 05 01 02 20 [state]
RX: 05 02 ff 00 (ACK)
```

---

### Service 0x03 (ACTOR_LEDS)

#### READ_LED_STATE (0x11)

Returns LED settings

```
TX: 05 01 03 11
RX: 05 03 12 [state]
    state: bit  flags
           bit0 (0x01) = LEDs on while charging
           bit1 (0x02) = LEDs on during normal operation
           default: 0x03 (both enabled)
```

#### WRITE_LED_STATE (0x10)

Sets LED behavior

```
TX: 05 01 03 10 [state]
RX: 05 03 ff 00 (ACK)
```

---

### Service 0x08 (BATT_MGMT)

#### READ_DISCHARGE_STATE (0x21)

Returns battery discharge status (used e.g. for storage mode)

```
TX: 05 01 08 21
RX: 05 08 22 [current_soc] [target_soc] [remaining_hi] [remaining_lo]
    current_soc: uint8, current charge %
    target_soc: uint8, target discharge %
    remaining: uint16_be, seconds remaining
```

#### READ_BMS_STATE (0x71)

Returns battery management system state including charging status

```
TX: 05 01 08 71
RX: 05 08 72 [flags] [is_charging]
    flags: uint8, BMS status flags
    is_charging: 0x00 = not charging, non-zero = charging
```

---

### Service 0x09 (MEMORY_MGMT)

#### READ_MEMORY_PREPARATION (0x10)

Prepares memory block for reading. Required before READ_MEMORY_BLOCK (was tested with Dealer Mode > Error Memory Reading)

```
TX: 05 01 09 10 [block_id]
RX: 05 09 ff 00 (ACK)

So far seen block_ids:
    0x03 = PRODUCTION_DATE_PCB
    0x04 = SERIAL_NUMBER_PCB
    0x05 = PRODUCTION_DATE_WHEEL
    0x06 = SERIAL_NUMBER_WHEEL
    0x50 = DEFAULT_DRIVING_PROFILE
    0x91 = DUODRIVE_FLAG
```

#### READ_MEMORY_BLOCK (0x11)

Reads prepared memory block data

```
TX: 05 01 09 11
RX: 05 09 12 [data...]
    data: variable length depending on block type
```

---

### Service 0x0A (VERSION_MGMT)

#### READ_HW_VERSION (0x41)

Returns hardware version number

```
TX: 05 01 0a 41
RX: 05 0a 42 [hw_version]
    hw_version: uint8 (e.g., 0x0C = version 12)
```

---

### Service 0x0C (SPECIAL_MODE_MGMT)

#### READ_TEST_PERIOD_STATE (0x21)

Returns Mobility Plus Package testperiod status

```
TX: 05 01 0c 21
RX: 05 0c 22 [state]
    state: 0x00 = NOT_STARTED
           0x01 = ACTIVE
           0x02 = ELAPSED # This is the permanent settings after the 30 days of test period
           0x03 = ERROR
```

---

### Service 0x0E (RTC)

#### READ_RTC_TIME (0x11)

Returns real-time clock value

```
TX: 05 01 0e 11
RX: 05 0e 12 [year_hi] [year_lo] [month] [day] [hour] [minute]
    year: uint16_be (e.g., 0x07E9 = 2025)
    month: 1-12
    day: 1-31
    hour: 0-23
    minute: 0-59
```

#### WRITE_RTC_TIME (0x10)

Sets real-time clock to custom values (each wheel can have it's own and distinct RTC value - Intellion
  will offer to syncrhonize if RTC's of two wheels diverge)

```
TX: 05 01 0e 10 [year_hi] [year_lo] [month] [day] [hour] [minute]
RX: 05 0e ff 00 (ACK)
```

---

### Service 0x14 (SYS_ERROR_MGMT)

#### RESET_ERROR (0x10)

Resets/clears error state

```
TX: 05 01 14 10 [error_code]
RX: 05 14 ff 00 (ACK)

So far known error_code values:
    0x01 = RESET_ERR_REMOTEMODE_PUSHRIMSENS # This error occurs during parking ('remote') mode disruption by grabbing a wheel.
```

---

### Service 0x7F (GENERAL_ERROR_MGMT)

#### READ_GENERAL_ERROR (0x11)

Returns current error status

```
TX: 05 01 7f 11
RX: 05 7f 12 [error_code]
    error_code: 0x00 = no error
                0x3E = ERR_CHARGER_REMOTEMODE (62) # Remote mode only will work w/o charger connected
                0x3F = ERR_REMOTEMODE_PUSHRIMSENS (63) # Will occur when push rims are grabbed during remote mode. Remote mode works with person sitting on the wheelchair, though
```

---

## RX Response Parsers not implemented ATM

### STATUS_SYSTEM_MODE (Service 0x01, Param 0x12)

Response to READ_SYSTEM_MODE (0x11)

```
Payload: [mode]
    mode: 0x00 = OFF
          0x01 = ON
          0x02 = STANDBY
          0x03 = FLIGHT
```

### STATUS_MOUNTING_SIDE (Service 0x01, Param 0x72)

Response to READ_MOUNTING_SIDE (0x71), this is again the DuoDrive remote mounting side.

```
Payload: [side]
    side: 0x01 = RIGHT
          0x02 = LEFT
```

### STATUS_AUTO_SHUTOFF_TIME (Service 0x01, Param 0x82)

Response to READ_AUTO_SHUTOFF_TIME (0x81). Android app maxes out at 10h, minimum is at 5 minutes.

```
Payload: [time_hi] [time_lo]
    time: uint16_be, seconds
```

### STATUS_FACTORY_RESET (Service 0x01, Param 0xA2)

Response to READ_FACTORY_RESET_STATE (0xA1)

```
Payload: [state]
    state: 0x00 = NOT_STARTED
           0x01 = IN_PROGRESS
           0x02 = FINISHED
           0x03 = ERROR
```

### STATUS_DUO_DRIVE_PARAMS (Service 0x01, Param 0xF2)

Response to READ_DUO_DRIVE_PARAMS (0xF1)
Note: mounting side is the mounting side of the DuoDrive remote again

```
Payload: [mounting_side] [speed_sensibility] [steering_dynamic]
```

### STATUS_BUZZER_VOL (Service 0x02, Param 0x12)

Response to READ_BUZZER_VOL (0x11)

```
Payload: [volume]
```

### STATUS_BUZZER_ON_OFF (Service 0x02, Param 0x22)

Response to READ_BUZZER_ON_OFF (0x21)

```
Payload: [state]
```

### STATUS_LED_STATE (Service 0x03, Param 0x12)

Response to READ_LED_STATE (0x11)

```
Payload: [state]
    bit0 = charging LEDs
    bit1 = normal operation LEDs
```

### STATUS_DISCHARGE (Service 0x08, Param 0x22)

Response to READ_DISCHARGE_STATE (0x21)

```
Payload: [current_soc] [target_soc] [remaining_hi] [remaining_lo]
```

### STATUS_BMS_STATE (Service 0x08, Param 0x72)

Response to READ_BMS_STATE (0x71)

```
Payload: [flags] [is_charging]
```

### READ_MEMORY_DATA (Service 0x09, Param 0x12)

Response to READ_MEMORY_BLOCK (0x11)

```
Payload: [data...]
  (variable length depending on block type)
```

### STATUS_HW_VERSION (Service 0x0A, Param 0x42)

Response to READ_HW_VERSION (0x41)

```
Payload: [hw_version]
```

### STATUS_TEST_PERIOD_STATE (Service 0x0C, Param 0x22)

Response to READ_TEST_PERIOD_STATE (0x21)

```
Payload: [state]
```

### STATUS_RTC_TIME (Service 0x0E, Param 0x12)

Response to READ_RTC_TIME (0x11)

```
Payload: [year_hi] [year_lo] [month] [day] [hour] [minute]
```

### STATUS_GENERAL_ERROR (Service 0x7F, Param 0x12)

Response to READ_GENERAL_ERROR (0x11)

```
Payload: [error_code]
```

---

## Incomplete so far: CRUISE_VALUES

The `parse_cruise_values` function in `m25_ecs.py:244` only extracts `distance_km` so far.
  The full payload is more than that.

Full payload structure (12-13 bytes):

```
Byte  Field              Type       Description
----  -----              ----       -----------
0     drive_mode         uint8      Flags: bit0=auto_hold, bit1=cruise, bit2=remote
1     push_rim_value     int8       Push rim sensor deflection (signed)
2-3   current_speed      uint16_be  Speed in 0.001 km/h units
4     soc                uint8      Battery state of charge %
5-8   overall_distance   uint32_be  Distance in 0.01 meter units
9-10  push_counter       uint16_be  Push rim deflection count since startup
11    wheel_error        uint8      Error code (0 = no error)
12    (optional)         uint8      Extended data if packet length == 22
```

Missing fields so far:
- drive_mode (with breakdown of the flags)
- push_rim_value
- current_speed (multiply by 0.001 for km/h)
- soc (already available via READ_SOC but is duplicated here - can be both for completeness)
- push_counter
- wheel_error

# Proprietary Authentication Protocol - Huawei ESM Batteries

## ⚠️ CRITICAL INFORMATION

This protocol **is NOT documented** in Huawei's standard Modbus specification. It is a proprietary protocol obtained through reverse engineering and analysis of communication traffic.

---

## 1. General Information

**Compatible Models:** ESM-48100B1, ESM-48150B1, ESM-48100B3, ESM-48100U2, and ESM family
**Base Protocol:** Modbus RTU with proprietary extensions
**Requirement:** Mandatory authentication before accessing FC41 functions
**Persistence:** Authentication status is maintained until physical disconnection

### 1.1 Why is Authentication Necessary?

- Huawei ESM batteries implement **software protection**
- Without authentication, they only respond to basic Modbus functions (FC03, FC04)
- The **FC41** function (reading device information) is **blocked** by default
- Authentication unlocks full access to the BMS

---

## 2. Wake-Up Protocol (Optional but Recommended)

### 2.1 Description

Batteries may be in **power-saving mode** and not respond immediately. The wake-up protocol "wakes them up" before authentication.

### 2.2 Implementation

**Command:** Repetitive reading of the basic voltage register
```
[slave_id] 03 00 00 00 01 [CRC]
```

**Wake-Up Parameters:**
```
Maximum attempts: 5
Timeout per attempt: 0.8 seconds
Wait pattern: 1s, 2s, 4s, 8s, 16s (progressive)
Success criterion: Valid response with voltage > 0
```

**Successful Response:**
```
[slave_id] 03 02 [voltage_MSB] [voltage_LSB] [CRC]
```

**Interpretation:** If the voltage obtained is greater than 0V (typically 40-55V), the battery is awake and ready for authentication.

---

## 3. 3-Step Authentication Sequence

### 3.1 Overview

| Step | Function | Purpose | Timeout |

------|---------|-----------|---------|

| 1 | FC03 Special | Unlock Command | 1.0s |

| 2 | FC10 | Date/Time Synchronization | 1.0s |

| 3 | FC41 | Access Validation | 2.0s |

### 3.2 Intermediate Steps

**Between steps:** 0.5-0.8 second pause for stabilization
**After Step 3:** 0.3 seconds before the first actual FC41

---

## 4. Step 1: Special Unlock Command

### 4.1 Description

Uses the FC03 (Read Holding Registers) function with specific parameters to unlock access to the battery.

### 4.2 Command

```
Format: [slave_id] 03 01 06 00 01 [CRC]
```

**Command Breakdown:**
| Byte | Value | Description |

|------|-------|-------------|

| 0 | slave_id | Battery ID (typically 214-218) |

| 1 | 0x03 | Function 03 (Read Holding Registers) |

| 2 | 0x01 | MSB Address (0x0106) |

| 3 | 0x06 | LSB Address |

| 4 | 0x00 | Number of MSBs (1 register) |

| 5 | 0x01 | Number of LSBs |

| 6-7 | CRC | CRC16 Modbus (little-endian) |

### 4.3 Expected Response

```
Format: [slave_id] 03 02 00 XX [CRC]
Length: Exactly 7 bytes
```

**Critical Validations:**
- `response[0] == slave_id` (Correct ID)
- `response[1] == 0x03` (Correct function)
- `response[2] == 0x02` (Byte count = 2)
- `response[3] == 0x00` (Expected pattern)
- `len(response) == 7` (Exact length)

### 4.4 Example of Real Traffic

```
TX: [D9 03 01 06 00 01 E4 2F] // Command for battery ID 217
RX: [D9 03 02 00 03 F8 45] // Successful response
```

---

## 5. Step 2: Date/Time Synchronization

### 5.1 Description

Sends the current system date and time to the battery using FC10 (Write Multiple Registers). **This step is mandatory** even if the battery has its own RTC.

### 5.2 Command

```
Format: [slave_id] 10 10 00 00 06 0C [DATE/TIME] [CRC]
```

**Command Structure:**
| Bytes | Value | Description |

|-------|-------|-------------|

| 0 | slave_id | Battery ID |

| 1 | 0x10 | Function 10 (Write Multiple Registers) |

| 2-3 | 0x10 0x00 | Starting address: 0x1000 |

4-5 | 0x00 0x06 | Number of registers: 6 (12 bytes) |

6 | 0x0C | Byte count: 12 bytes of data |

7-18 | [DATA] | Date/Time (see format below) |

19-20 | CRC | CRC16 Modbus |

### 5.3 Date/Time Format

**12 bytes organized as 6 16-bit (big-endian) registers:**

| Bytes | Register | Description | Format | Example |

|-------|----------|-------------|---------|---------|

7-8 | 1 | Year | YYYY (16-bit) | 0x07E8 = 2024 |

9-10 | 2 | Month | 0x00 + MM | 0x000C = December |
| 11-12 | 3 | Day | 0x00 + DD | 0x001F = 31 |
| 13-14 | 4 | Time | 0x00 + HH | 0x0017 = 23:xx |
| 15-16 | 5 | Minute | 0x00 + mm | 0x003B = xx:59 |
| 17-18 | 6 | Second | 0x00 + ss | 0x003B = xx:xx:59 |

### 5.4 Expected Response

```
Format: [slave_id] 10 10 00 00 06 [CRC]
Length: Exactly 8 bytes
```

**Validations:**
- Echo of bytes 2-5 of the original command
- Confirms writing 6 records to 0x1000

### 5.5 Example of Real Traffic

```
// For 31/Dec/2024 23:59:59
TX: [D9 10 10 00 00 06 0C 07 E8 00 0C 00 1F 00 17 00 3B 00 3B XX XX]
RX: [D9 10 10 00 00 06 XX XX] // Confirmation echo
```

---

## 6. Step 3: Access Validation

### 6.1 Description

Uses the custom function **FC41** to complete validation and enable full access to all subsequent FC41 functions.

### 6.2 Command

```
Format: [slave_id] 41 05 01 04 [CRC]
```

**Command Breakdown:**
| Byte | Value | Description |

|------|-------|-------------|

| 0 | slave_id | Battery ID |

| 1 | 0x41 | Function 41 (Huawei custom) |

| 2 | 0x05 | Subfunction/Parameter |

| 3 | 0x01 | Validation Parameter |

| 4 | 0x04 | Command Code |

| 5-6 | CRC | CRC16 Modbus |

### 6.3 Expected Response

```
Format: [slave_id] 41 05 06 [DATA...] [CRC]
Minimum length: 9 bytes
Typical length: 10-12 bytes
```

**Minimum Validations:**
- `response[0] == slave_id`
- `response[1] == 0x41`
- `len(response) >= 9`
- Bytes 2-3 typically: `0x05 0x06`

### 6.4 Example of Real Traffic

```
TX: [D9 41 05 01 04 XX XX] // Validation Command
RX: [D9 41 05 06 XX XX XX XX XX] // Confirmation Response
```

---

## 7. Post-Authentication Reading with FC41

### 7.1 Available Functions

Once authentication is complete, the following FC41 functions are available:

| Subfunction | Command | Purpose |

------------|---------|-----------|

| 06 03 04 | Device Information | Read manufacturer, model, serial number data |

| 06 03 05 | Historical Logs | Read stored logs |

| 05 01 05 | Initialize History | Prepare history session |

| 0C 01 05 | Close History | End history session |

### 7.2 Reading Device Information

#### 7.2.1 Command

```
Format: [slave_id] 41 06 03 04 00 [index] [CRC]
```

**Parameters:**
- **index:** 0-5 (6 fragments available)

#### 7.2.2 Information by Index

| Index | Typical Content |

|--------|------------------|

| 0 | Manufacturer (VendorName), Model (BoardType) |

| 1 | Serial Number (BarCode) |

| 2 | Date Manufactured |

| 3 | Firmware Version (ArchivesInfoVersion) |

| 4 | Label Version |

| 5 | Extended Information and Description |

#### 7.2.3 Response Format

```
[slave_id] 41 [length] [status] [data_bytes...] [CRC]
```

**Structure:**
- **Bytes 0-1:** slave_id + 0x41
- **Bytes 2-6:** FC41 Response Header
- **Bytes 7 to CRC-2:** ASCII Device Data
- **Last 2 bytes:** CRC

#### 7.2.4 Data Decoding

Data is returned as **ASCII text** in key=value format:

**Example Data Decoded:**
```
VendorName=HUAWEI
BoardType=ESM-48150B1
BarCode=ABC123DEF456
Manufactured=2023-08-15
ArchivesInfoVersion=V1.2
LabelVersion=V2.1
Description=Lithium Battery Management System
```

**Decoding Process:**
1. Extract data bytes (from byte 7 to CRC-2)
2. Decode as ASCII (ignore non-printable characters)
3. Parse lines separated by \n or \r
4. Split each line by '=' to obtain key-value pairs
5. Clean up whitespace and special characters

### 7.3 History Reading

#### 7.3.1 History Reading Sequence

1. **Initialize session:** `[slave_id] 41 05 01 05 [CRC]`
2. **Reset pointer:** `[slave_id] 41 06 03 05 00 00 [CRC]`
3. **Read records:** `[slave_id] 41 06 03 05 [record_MSB] [record_LSB] [CRC]`
4. **Log out:** `[slave_id] 41 0C 01 05 [CRC]`

#### 7.3.2 Historical Record Format

Each historical record contains **32 bytes of data** with the following structure:

| Bytes | Parameter | Factor | Description |

|-------|-----------|--------|-------------|

| 8-9 | Pack Voltage | 0.01 | Total Voltage in V |

| 10-11 | Current | 0.01 | Signed Current (A) |

16 | Minimum Temperature | 1 | Lowest Temperature (°C) |

18 | Maximum Temperature | 1 | Highest Temperature (°C) |

20 | State of Charge (SOC) | 1 | State of Charge (%) |

24-25 | Ah Discharged | 1 | Ampere-hours discharged |

28 | Discharge Cycles | 1 | Number of discharges |

30-31 | Battery Voltage | 0.01 | Battery voltage (V) |

#### 7.3.3 End of History Detection

**Empty Register:** All 32 bytes of data are `0xFF`
```
[slave_id] 41 [length] [status] [FF FF FF ... FF] [CRC]
```

This indicates that the end of the stored history has been reached.

#### 7.3.4 Signed Value Processing

**Current (bytes 10-11):**
- If value > 32767: actual_value = value - 65536
- Factor: 0.01
- Positive: Charge, Negative: Discharge

---

## 8. Post-Authentication Verification

### 8.1 Successful Authentication Test

After the 3 steps, verify success by attempting to read basic information:

```
Command: [slave_id] 41 06 03 04 00 00 [CRC]
```

**Expected Result:** FC41 response with device ASCII data

### 8.2 Success Indicators

Successful authentication is confirmed when:
- All 3 steps are completed without errors
- The first FC41 command returns valid ASCII data
- The data contains fields like "VendorName=HUAWEI"
- No timeout errors on subsequent commands

### 8.3 State Persistence

- **During session:** Authenticated state is maintained
- **After disconnection:** State is lost and must be repeated
- **Baud rate change:** Requires re-authentication
- **Hardware reset:** Requires full reset sequence

---

## 9. Troubleshooting and Common Errors

### 9.1 Communication Errors

| Error | Symptom | Probable Cause | Solution |

|-------|---------|----------------|----------|

| No response in Step 1 | Timeout | Battery in sleep mode | Run wake-up first |

| Incomplete response | Few bytes | Very short timeout | Increase timeout to 1.0s+ |

Incorrect CRC | Validation error | Bus interference | Check RS485 wiring |

Incorrect ID | response[0] != slave_id | Wrong address | Verify correct slave_id |

### 9.2 Protocol Errors

| Error | Symptom | Cause | Solution |

|-------|---------|-------|----------|

| Step 2 fails | Incorrect echo | Date/time format | Verify command construction |

| Step 3 no response | FC41 not responding | Steps 1-2 failed | Restart entire sequence |

| Subsequent FC41 fails | Function unavailable | Incomplete authentication | Re-authenticate |

### 9.3 Hardware Problems

| Problem | Symptom | Verification |

|----------|---------|--------------|

| RS485 cable | No communication | Verify pins A, B, and GND |

| Termination | Corrupted data | Verify 120Ω resistors |

| Power supply | Battery not responding | Verify 48V present |

| Baud rate | Strange characters | Confirm 9600 bps |

--

## 10. Implementation Considerations

### 10.1 Timing Parameters

- **Minimum Timeout:** 0.8 seconds per command
- **Total Time:** 5-7 seconds for complete sequence
- **Retries:** Maximum 3 attempts per step
- **Persistence:** State is maintained throughout the session

### 10.2 Model Compatibility

| Model | Compatibility | Notes |

|--------|----------------|-------|

| ESM-48100B1 | ✅ Confirmed | Standard Protocol |

| ESM-48150B1 | ✅ Confirmed | Standard Protocol |

| ESM-48100B3 | ✅ Confirmed | Standard Protocol |

| ESM-48100U2 | ⚠️ Probable | Not Verified |
| Other ESMs | ⚠️ Likely | Check individually |


### 10.3 Security Considerations

- **Session Authentication:** Must be repeated after each disconnection
- **Not Cryptographic:** The protocol does not include encryption
- **Full Access:** Once authenticated, full access to the BMS
- **No Access Control:** There are no differentiated roles or permissions

### 10.4 System Integration

To integrate with monitoring systems:

1. **Initialize:** Wake-up + Authentication upon connection establishment
2. **Maintain:** Periodically verify connection
3. **Recover:** Automatically re-authenticate after disconnection
4. **Monitor:** Use FC03/FC04 for continuous log reading
5. **Diagnosticate:** Use FC41 for detailed information as needed

---

## 11. References and Resources

### 11.1 Analysis Tools

- **Analyzers Modbus:** To capture traffic and verify commands
- **Oscilloscopes:** To verify RS485 signals
- **Multimeters:** To verify voltage levels

### 11.2 Related Documentation

- Official Modbus RTU Specification
- Huawei ESM Modbus Registers Manual
- RS485 Wiring Guides

### 11.3 Contact Information

- DIYSolar forums for user experiences
- GitHub repositories with reference implementations
- BMS protocol reverse engineering groups

---

**Final Note:** This protocol was developed through reverse engineering and traffic analysis. Huawei does not provide official documentation on this authentication sequence. Use at your own risk and always verify compatibility with specific hardware.

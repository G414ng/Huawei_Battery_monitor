# Gyroscope Disablement Protocol - Huawei ESM-48150B1 Battery

## General Description

Undocumented protocol for disabling the gyroscope integrated into Huawei ESM-48150B1 batteries using Modbus RTU commands over TCP.

## Technical Specifications

### Communication
- **Protocol**: Modbus RTU over TCP
- **TCP Port**: 8235
- **Slave ID**: 214 (0xD6)
- **Recommended Timeout**: 1000ms

### Modbus Registers

| Register | Type | Description | Access |

|----------|------|-------------|--------|

`0x0000` | Holding | General system status | R |

`0x1118` | Holding | Gyroscope status (0=Off, 1=On) | R |

| `0x117C-0x118B` | Holding | Cryptographic Key Area (16 registers) | W |

## Protocol Structure

### Deactivation Sequence

```
1. Read(0x0000)          → Verificar estado del sistema
2. Write(0x117C, clave)  → Escribir clave de 32 bytes
3. Read(0x1118)          → Verificar estado = 1 (activo)
4. Write(0x117C, clave)  → Reescribir clave (confirmación)
5. Read(0x1118)          → Verificar estado = 0 (desactivado)
```

### Frame Modbus RTU

#### Comando Write Multiple Registers (0x10)

```
[Slave ID][Function][Address H][Address L][Quantity H][Quantity L][Byte Count][Data...][CRC]
   D6        10         11         7C         00          10          20      [32 bytes] [2 bytes]
```

**Desglose**:
- `D6`: Slave ID (214 decimal)
- `10`: Function Code (Write Multiple Registers)
- `117C`: Starting Address
- `0010`: Quantity of Registers (16 registers = 32 bytes)
- `20`: Byte Count (32 decimal)
- `[32 bytes]`: Clave criptográfica
- `[CRC]`: Modbus CRC-16

#### Respuesta Write Multiple Registers

```
[Slave ID][Function][Address H][Address L][Quantity H][Quantity L][CRC]
   D6        10         11         7C         00          10      [2 bytes]
```

## Cryptographic Key

### Characteristics

- **Length**: 32 bytes (256 bits)
- **Format**: Unencoded binary
- **Validity**: 24 hours (changes daily)
- **Scope**: Universal (same key for all batteries of the same model)
- **Entropy**: High (~88% unique bytes)

### Key Structure

```
Offset  +0 +1 +2 +3 +4 +5 +6 +7 +8 +9 +A +B +C +D +E +F
0x00    [-------- 32 bytes de datos criptográficos --------]
0x10    [-------------- continuación... ------------------]
```

### Generation Pattern

- Date-based algorithm (changes at midnight)
- Contains no battery identifiers
- Likely SHA-256 with proprietary salt
- No apparent relationship to device parameters

## Frame Examples

### Write Multiple Registers - Request

```
D6 10 11 7C 00 10 20 25 BD 4C CF 7B 23 EF 2F 43 C3 88 3B 76 74 A3 69
05 3E 27 3B 19 A2 92 EC 05 9B BA DB 08 EA C3 DE D1 53
```

### Write Multiple Registers - Respuesta (ACK)

```
D6 10 11 7C 00 10 17 06
```

### Read Holding Register 0x1118 - Solicitud

```
D6 03 11 18 00 01 13 16
```

### Read Holding Register 0x1118 - Respuesta

```
D6 03 02 00 01 0D 96    // Giroscopio activo (valor = 1)
D6 03 02 00 00 CC 56    // Giroscopio inactivo (valor = 0)
```

## Important Notes

1. **Double Write**: The protocol requires writing the key twice to confirm the operation.
2. **Status Verification**: Always verify the 0x1118 register after each write.
3. **Universal Key**: The same key works for all batteries during the validity period.
4. **No Official Documentation**: This protocol is not listed in the manufacturer's manuals.
5. **Daily Change**: The key changes every 24 hours (time zone to be determined).

## Flowchart

```
┌─────────────┐
│   START     │
└──────┬──────┘
       │
       ▼
┌─────────────────┐
│ Read(0x0000)    │
└────────┬────────┘
         │
         ▼
┌──────────────────────┐
│ Write(0x117C, clave) │
└──────────┬───────────┘
           │
           ▼
    ┌──────────────┐
    │ Read(0x1118) │
    └──────┬───────┘
           │
           ▼
      ┌─────────┐
      │ = 0x01? │
      └────┬────┘
           │ Sí
           ▼
┌──────────────────────┐
│ Write(0x117C, clave) │
└──────────┬───────────┘
           │
           ▼
    ┌──────────────┐
    │ Read(0x1118) │
    └──────┬───────┘
           │
           ▼
      ┌─────────┐
      │ = 0x00? │
      └────┬────┘
           │ Sí
           ▼
     ┌─────────┐
     │ ÉXITO   │
     └─────────┘
```

## Implementation Considerations

- Implement a 1-second timeout for each operation
- Handle reconnection in case of communication loss
- Validate key length before sending (exactly 32 bytes)
- Consider retries in case of communication failure
- Detailed logging for troubleshooting

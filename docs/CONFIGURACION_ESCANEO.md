# 🔎 Device Configuration and Scanning

[<< Back to Main README](../README.md) | [Go to User Guide >>](USO.md)

The system includes tools for configuring communication and discovering Modbus devices on the network, specifically designed to identify Huawei ESM-48150B1 batteries.

## Configuration File

The system uses a `config.json` file in the project root to store all configuration parameters and maintain information about the discovered devices.

```json
{
  "serial": {
    "port": "COM8",
    "baudrate": 9600,
    "parity": "N",
    "stopbits": 1,
    "bytesize": 8,
    "timeout": 1.0
  },
  "scanning": {
    "start_id": 214,
    "end_id": 231,
    "max_attempts": 3,
    "progressive_wait": true,
    "scan_timeout": 0.5
  },
  "application": {
    "auto_connect": true,
    "last_connected_id": 214,
    "discovered_devices": [
      // Aquí se almacenan los dispositivos encontrados...
    ]
  },
  "device_types": {
    "huawei_battery": {
      // Definición de parámetros específicos del tipo de dispositivo...
    }
  }
}
```

### config.json Structure

| Section | Description |

---------|-------------|

`serial` | Serial port configuration (COM port, baud rate, parity, etc.) |

`scanning` | Parameters for device scanning (ID range, attempts, etc.) |

`application` | Application configuration and list of discovered devices |

`device_types` | Definition of device types and their characteristics |

## Scanning Tool

The `scan_modbus_devices.py` script allows automatic discovery of Modbus RTU devices on the bus, specifically identifying Huawei batteries.

<div align="center">
  <img src="../static/images/scan_example.jpg" alt="Ejemplo de escaneo" style="width: 80%; max-width: 600px; border: 2px solid #ddd; border-radius: 8px;">
  <p><i>Ejemplo: Resultado de escaneo con dispositivos encontrados</i></p>
</div>
<div align="center">
  <img src="../static/images/escaneo2.jpg" alt="Ejemplo de escaneo" style="width: 80%; max-width: 600px; border: 2px solid #ddd; border-radius: 8px;">
  <p><i>Ejemplo: Resultado de escaneo con dispositivos encontrados</i></p>
</div>


> ⚠️ **Note**: If the image above does not appear, you must capture a screenshot of the scanning process and save it as `static/images/scan_example.png`

### Scanner Features

- **Smart Scanning**: Searches for devices within a configurable ID range
- **Automatic Identification**: Detects Huawei batteries based on response patterns
- **Adaptive Retries**: Implements progressive wait retries for slow devices
- **Full Logging**: Captures key data such as voltage, current, SOC, and SOH
- **Configuration Update**: Automatically saves found devices

### Using the Scanner

```bash
# Run device scan
python scan_modbus_devices.py
```

During the scan:
1. It will connect to the port configured in `config.json`
2. It will search for devices within the specified ID range
3. It will automatically identify the devices found
4. It will update the `config.json` file with the information obtained

> ⚠️ **ATTENTION**: The scan will replace the current list of discovered devices in the configuration.

### Scan Parameters

The scanner's behavior can be customized by modifying the following parameters in the `scanning` section of the `config.json` file:

| Parameter | Description |

|-----------|-------------|

`start_id` | Starting ID for the scan range (default: 214) |

`end_id` | Ending ID for the scan range (default: 231) |

`max_attempts` | Maximum number of attempts per device (default: 3) |

`progressive_wait` | If `true`, the wait time between attempts increases progressively |

`scan_timeout` | Wait time (in seconds) for each scan attempt (default: 0.5) |

### Scan Output

For each device found, the following is recorded:

- **ID**: Modbus identifier of the device
- **Type**: "huawei_battery" or "unknown_device"
- **Voltage**: Battery voltage value (V)
- **Current**: Current value (A)
- **State of Charge (SOC)**: Percentage of charge
- **State of Health (SOH)**: Battery health indicator
  
## Device Data Structure

Each discovered device is saved with the following structure in the `config.json` file:

```json
{
  "id": 214,
  "register_0": 5222,
  "discovery_date": "2025-04-27T11:17:05",
  "last_seen": "2025-04-27T11:17:05",
  "custom_name": "Batería Huawei 214",
  "registers": {
    "battery_voltage": 52.22,
    "pack_voltage": 51.92,
    "current": 11.23,
    "soc": 12,
    "soh": 94
  },
  "raw_values": [5222, 5192, 1123, 12, 94],
  "type": "huawei_battery"
}
```

### Device Data Fields

| Field | Description |

-------|-------------|

`id` | Modbus slave ID |

`register_0` | Raw value of the first register (useful for identification) |

`discovery_date` | Date and time of first discovery |

`last_seen` | Last time the device was seen |

`custom_name` | Customizable name for the device |

`registers` | Processed values ​​of the main registers |

`raw_values` | Raw values ​​of the registers (for debugging) |

`type` | Type of device identified |

## Huawei Battery Identification

The system automatically identifies Huawei ESM-48150B1 batteries based on:

1. **Voltage Patterns**: Typical values ​​between 30V and 60V
2. **Register Structure**: Specific arrangement of data registers
3. **Command Response**: Behavior when read requests are made

Correctly identified batteries are labeled `huawei_battery` in the configuration, while other devices are marked as `unknown_device`.

## Programmatic Usage

The module can be imported and used in custom Python code:

```python
from modbus_app.client import connect_client, disconnect_client, get_client
import json

# Cargar configuración
with open('config.json', 'r') as f:
    config = json.load(f)

# Conectar al cliente
serial_config = config["serial"]
connect_client(serial_config["port"], serial_config["baudrate"])

# Obtener cliente para operaciones personalizadas
client = get_client()

# Realizar operaciones...

# Desconectar al finalizar
disconnect_client()
```

### Example: Programmatic scanning of a specific device

```python
from modbus_app.client import connect_client, disconnect_client, get_client
from datetime import datetime

# Conectar al puerto serial
connect_client("COM8", 9600)
client = get_client()

# Obtener información básica de la batería (ID 217)
slave_id = 217
result = client.read_holding_registers(address=0, count=5, slave=slave_id)

if not result.isError() and hasattr(result, 'registers'):
    voltage = result.registers[0] * 0.01
    current = result.registers[2] * 0.01
    soc = result.registers[3]
    soh = result.registers[4]
    
    print(f"Batería ID {slave_id}:")
    print(f"  Voltaje: {voltage:.2f} V")
    print(f"  Corriente: {current:.2f} A")
    print(f"  SOC: {soc} %")
    print(f"  SOH: {soh} %")

# Desconectar al finalizar
disconnect_client()
```

## Troubleshooting

If you experience difficulties with scanning:

- **Check physical connection**: Ensure the USB-RS485 adapter is properly connected.
- **Check serial configuration**: Confirm that the baud rate, parity, and bit settings match the battery.
- **Adjust timeout**: For slow networks, increase `scan_timeout` in `config.json`.
- **Check ID range**: Verify that the range includes your device IDs (typically 217 for Huawei batteries).
- **Increase attempts**: For devices that take a long time to wake up, increase `max_attempts`.

### Common problems:

1. **No device found**:

- Check the physical connections.

- Confirm that the serial parameters (`baud rate`, `parity`, etc.) match the battery settings.

- Try different ID ranges (Huawei batteries typically use ID 217).

2. **Device Identified as "unknown_device"**:

- The battery may be in deep sleep mode

- The values ​​read may be outside expected ranges

- Attempt to perform a "wake-up" operation before scanning

3. **Timeout error during scanning**:

- Increase the `scan_timeout` value in the settings

- Enable `progressive_wait` to allow more time for retries

- Increase `max_attempts` for slow-response devices

For a more detailed analysis of communication problems, see the [Troubleshooting](TROUBLESHOOT.md) section.

---

[<< Back to Main README](../README.md) | [Go to User Guide >>](USER.md)

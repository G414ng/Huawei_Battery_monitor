# Modbus Web UI for Huawei ESM-48150B1 Batteries

![Python Version](https://img.shields.io/badge/python-3.12+-blue.svg)
![License](https://img.shields.io/badge/license-BSD-green.svg)
![Status](https://img.shields.io/badge/estado-beta-orange.svg)

## 🔋 Introduction

**Advanced monitoring and control system for Huawei ESM-48150B1 batteries** with native Modbus RTU protocol. This project implements a custom Modbus client that replaces PyModbus with a solution optimized specifically for Huawei batteries, including automatic authentication and full support for proprietary functions.

> **⚡ STILL UNDER DEVELOPMENT. SOME FEATURES ARE INCOMPLETE.:** Version 0.5

> **⚡ Key Features:** Simultaneous monitoring of multiple batteries, modern web interface, native Huawei protocol, advanced diagnostic system, and detailed data export.

** ## 📸 System View

<div align="center">

<h3>Main Monitoring Interface</h3>

<img src="static/images/Monitordebaterias.png" alt="Main system interface" style="width: 90%; max-width: 800px; border: 2px solid #ddd; border-radius: 8px;">

<p><i>Main panel with real-time dashboard of multiple batteries</i></p>

</div>

<div align="center">

<h3>Individual Battery Dashboard</h3>

<img src="static/images/Vista moderna.png" alt="Detailed battery dashboard" style="width: 90%; max-width: 800px; border: 2px solid #ddd; border-radius: 8px;">

<p><i>Detailed view with cell data, Graphics and Diagnostics</i></p>
</div>

<div align="center">

<h3>Advanced Diagnostics System</h3>

<img src="static/images/Celdas.png" alt="Diagnostics Panel" style="width: 90%; max-width: 800px; border: 2px solid #ddd; border-radius: 8px;">

<p><i>Complete Log Analysis and Data Export</i></p>
</div>

## 🚀 Main Features

### 🔌 Advanced Communication System
- **Native Modbus Client**: Custom implementation in `core.py` that replaces PyModbus
- **Optimized Huawei Protocol**: Full support for FC41 and authentication sequences
- **Unified Connection**: Simplified single connection system with automatic management
- **Thread-Safe**: Safe concurrent operations for multiple Devices

### 📊 Multi-Battery Monitoring
- **Simultaneous Monitoring**: Real-time tracking of multiple Huawei batteries
- **Intelligent Cache System**: Optimized data management per device with `device_cache.py`
- **Automatic History**: Periodic data recording for time-series analysis
- **Individual Cell Data**: Detailed monitoring of voltages and temperatures per cell

### 🎛️ Modern Web Interface
- **Industrial Dashboard**: Unified view with real-time graphs
- **Tab System**: Modular organization of information (Status, Cells, Diagnostics, Advanced)
- **Advanced Diagnostics**: Structured visualization of all mapped records
- **Data Export**: Multiple formats (JSON, CSV, PDF)

### 🔧 Technical Features
- **Automatic Detection**: Intelligent battery identification The Network
- **State Management**: Advanced connection control, authentication, and monitoring
- **Alert System**: Automatic notifications for critical conditions
- **Full REST API**: Endpoints for integration with external systems

## 🎯 Compatible Devices

### Fully Supported
- **Huawei ESM-48150B1** (Typical ID: 217)

- Automatic authentication

- Extended information reading (FC41)

- Individual cell monitoring

- Manufacturing and diagnostic data

### Basic Support
- **Generic Modbus RTU Devices**

- Standard functions (FC01-FC06, FC15-FC16)

- No authentication or proprietary functions

## 🏗️ System Architecture

### Componentes Principales
```
modbus_app/
├── huawei_client/          # Cliente Modbus nativo
│   ├── core.py            # Cliente principal (reemplaza PyModbus)
│   ├── protocol.py        # Protocolo Modbus RTU
│   └── authentication.py  # Autenticación Huawei
├── battery_monitor.py     # Monitor multi-batería con threading
├── device_cache.py        # Sistema de cache inteligente
├── operations.py          # Mapeo de registros y operaciones
└── logger_config.py       # Configuración de logging
```

### Frontend Modular
```
static/js/
├── main.js                # Inicialización y coordinación
├── modbusApi.js          # API unificada del sistema
├── connectionHandler.js   # Gestión de conexión única
├── battery-components/    # Componentes modulares de batería
│   ├── tabs/             # Sistema de pestañas
│   └── charts/           # Gráficos y visualizaciones
└── vista-industrial/     # Estilos y componentes industriales
```

## 📚 Complete Documentation

For detailed information on every aspect of the system, please consult the following specialized guides:

### 🚀 **Getting Started**
- 🔧 [**Installation and Configuration**](docs/INSTALACION.md) - Step-by-step setup
- 🔌 [**Hardware Configuration**](docs/CONFIGURACION_HARDWARE.md) - RS485 adapters and physical connections
- ▶️ [**User Guide**](docs/USO.md) - Complete web interface manual

### 🏗️ **Architecture and Functionalities**
- 🏛️ [**System Architecture**](docs/ARQUITECTURA_SISTEMA.md) - Native client vs. PyModbus
- 🔋 [**Monitor of Batteries**](docs/MONITOR_BATERIAS.md) - Multi-battery system and threading
- 🔎 [**Device Detection**](docs/DETECCION_DISPOSITIVOS.md) - Automatic scanning and configuration

### 🔧 **Advanced Configuration**
- ⚙️ [**Advanced Configuration**](docs/CONFIGURACION_AVANZADA.md) - Timeouts, optimization, and expert parameters
- 🌐 [**API and Integration**](docs/API_REFERENCIA.md) - Complete endpoint documentation
- 🔋 [**Huawei Protocol**](docs/PROTOCOLO_HUAWEI.md) - Authentication and FC41 technical details

### 📋 **Protocol Technical Documentation**
- 📖 [**Records Modbus Huawei ESM**](docs/REGISTROS_MODBUS.md) - Complete specification of registers, thresholds, and configurations
- 🔐 [**Huawei Authentication Protocol**](docs/Authentificacion.md) - Proprietary 3-step sequence and FC41 functions
- 🔐 [**Gyroscope Protocol**](docs/Giroscopio.md) - Gyroscope deactivation sequence with Liberado Software

### 🛠️ **Diagnostics and Support**
- 🔍 [**Advanced Diagnostics**](docs/DIAGNOSTICOS_AVANZADOS.md) - Analysis of registers and cell data
- ❓ [**Troubleshooting**](docs/SOLUCION_PROBLEMAS.md) - Troubleshooting and common errors
- 🤝 [**Guide to Contribution**](docs/CONTRIBUTIONS.md) - How to collaborate with the project

---

## 📋 Quick Installation

### System Requirements
- **Python 3.10+** (3.12+ recommended)
- **Available COM port** (USB-RS485 or virtual)
- **4GB RAM minimum** (8GB recommended)
- **Internet connection** (for dependencies)

### Automatic Installation
```bash
# Clonar repositorio
git clone https://github.com/williamsioSapo/Huawei_Battery_monitor
cd Huawei_Battery_monitor

# Crear entorno virtual
python -m venv env

# Activar entorno (Windows)
.\env\Scripts\activate
# Activar entorno (Linux/macOS)
source env/bin/activate

# Instalar dependencias
pip install -r requirements.txt

# Ejecutar aplicación
python app.py
```

### Accessing the Application
- **Local URL**: `http://127.0.0.1:5000`
- **Local Network**: `http://[server-IP]:5000`

## 🔧 Quick Setup

### 1. Automatic Setup
On first-time operation, the system will automatically detect:
- Available COM ports
- Optimal communication parameters
- Devices connected to the network

### 2. Typical Parameters for Huawei ESM-48150B1
```json
{
  "port": "COM8",           # ver en panel de control
  "baudrate": 9600,         # Estándar para Huawei
  "parity": "N",            # Sin paridad
  "stopbits": 1,            # 1 bit de parada
  "bytesize": 8,            # 8 bits de datos
  "timeout": 1,             # Timeout en segundos
  "slave_id": 217           # ID típico de Huawei ESM
}
```

## 📊 System Usage

### 🔌 Initial Connection
1. **Configure Port**: Select the correct COM port
2. **Connect System**: Click the "Connect to System" button
3. **Initialize Batteries**: Automatic detection and authentication process
4. **Open Dashboard**: Access to the main monitoring panel

### 📈 Real-Time Monitoring
- **Main Dashboard**: Overview of all batteries
- **Data per Cell**: Individual voltages and temperatures
- **Historical Graphs**: Voltage, current, and SOC trends
- **Automatic Alerts**: Notifications for abnormal conditions

### 🔧 Advanced Operations
- **Read Registers**: Direct access to Modbus registers
- **Write Parameters**: Controlled modification of configurations
- **Comprehensive Diagnostics**: Exhaustive analysis of the system status
- **Data Export**: Reports in multiple formats

## 🛠️ Advanced Technical Features

### Custom Modbus Client
- **PyModbus Elimination**: More efficient native implementation
- **Adaptive Timeouts**: Automatic configuration based on operation type
- **Automatic Reconnection**: Recovery after communication loss
- **Thread Safety**: Safe concurrent operations

### Monitoring System
- **Intelligent Polling**: Adaptive polling frequency based on activity
- **Multi-Level Cache**: Optimized access to frequently used data
- **Persistent History**: Automatic storage for analysis
- **Anomaly Detection**: Algorithms for identifying unusual patterns

### API REST Nativa
```javascript
// Ejemplos de uso de la API
conectarSistema(parametros)           // Conexión única
inicializarBaterias()                 // Autenticación automática
getAvailableBatteries()              // Lista de baterías detectadas
startMultiBatteryMonitoring()        // Monitoreo simultáneo
getAllMappedRegisters(batteryId)     // Datos estructurados completos
```

## 📂 Estructura del Proyecto Actualizada

```
ModbusReader_SR/
├── app.py                     # Aplicación Flask principal
├── config.json               # Configuración centralizada
├── requirements.txt          # Dependencias Python
├── modbus_app/              # Módulo principal de la aplicación
│   ├── huawei_client/       # Cliente Modbus nativo
│   ├── battery_monitor.py   # Monitor multi-batería
│   ├── device_cache.py      # Sistema de cache
│   ├── operations.py        # Mapeo de registros
│   └── logger_config.py     # Configuración de logs
├── static/                  # Recursos web estáticos
│   ├── css/                # Estilos (incluyendo vista industrial)
│   ├── js/                 # JavaScript modular
│   └── images/             # Recursos gráficos
├── templates/              # Plantillas HTML
│   └── index.html         # Interfaz principal
└── docs/                   # Documentación detallada
    ├── INSTALACION.md      # Guía de instalación
    ├── CONFIGURACION_HARDWARE.md
    ├── USO.md              # Manual de usuario
    ├── API_REFERENCIA.md   # Documentación de la API
    ├── REGISTROS_MODBUS.md # Especificación completa de registros
    └── PROTOCOLO_AUTENTICACION.md # Protocolo propietario Huawei
```

## 🔍 Diagnostics and Troubleshooting

### Integrated Diagnostic Tools
- **Connection Monitor**: Real-time communication status
- **Log Analyzer**: Structured data visualization
- **System Log**: Detailed operation log
- **Communication Test**: Automatic connectivity verification

### Common Problems and Solutions

| Problem | Probable Cause | Solution |

|----------|----------------|----------|

| COM port not detected | USB-RS485 driver | Check in Device Manager |

| Communication timeout | Incorrect serial parameters | Use 9600-8N1 for Huawei |

| Authentication failure | Incorrect slave ID | Check ID 217 for ESM-48150B1 |

| Incomplete cell data | Battery in power-saving mode | Wake battery on initial operation |

## 🚀 Improvements from the Previous Version

### ✅ Implemented
- ✨ **Native Modbus Client** - Eliminates dependency on PyModbus
- 🔋 **Multi-Battery Monitoring** - Simultaneous support for multiple devices
- 🎛️ **Unified Interface** - Simplified connection system
- 📊 **Advanced Diagnostics** - Complete structured visualization
- 💾 **Intelligent Cache** - Optimized data management
- 🔄 **Secure Threading** - Stable concurrent operations

### 🔄 Under Development
- 📱 **Mobile App** - Native interface for mobile devices
- ☁️ **Cloud Integration** - Synchronization with external services
- 🤖 **Predictive AI** - Failure prediction algorithms
- 📈 **Advanced Analytics** - Metrics of Performance and Efficiency

## 🤝 Support and Community

### Help Resources
- 📖 **Documentation**: Folder `docs/` with detailed guides
- 🐛 **Issues**: [GitHub Issues](https://github.com/nestorcal/ModbusReader_SR/issues)
- 💬 **Discussions**: [GitHub Discussions](https://github.com/nestorcal/ModbusReader_SR/discussions)
- 📧 **Contact**: [Project contact information]

### Contributions
Contributions are welcome! See [CONTRIBUTIONS.md](docs/CONTRIBUTIONS.md) for:
- 🔧 New features
- 🐛 Bug fixes
- 📚 Improved documentation
- 🧪 Testing with new devices

## 📜 License

This project is licensed under the **BSD License**. See the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- **PyModbus Community** - Foundation for initial development
- **Reverse Engineering** - Analysis of Huawei's proprietary protocol
- **Contributors** - Testing, feedback, and improvements
- **Huawei** - Manufacturer of the ESM-48150B1 hardware

---

> **⚠️ Disclaimer**: This software is independent and not officially affiliated with Huawei. Use it at your own risk in production systems.

**Last Update**: December 2024 | **Version**: 2.0-beta

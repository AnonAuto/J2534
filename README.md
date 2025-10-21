<img width="25%" height="25%" align="right" alt="ANONAUTOSTICKER" src="https://github.com/user-attachments/assets/76bb6c11-2a57-42aa-b36f-8432fdbca5c0" />


# J2534
In 32 bit Python.  


    The Anonymous Automotive Alliance Presents 

             A J2534 Implementation.

              In 32 bit Python.

              Deployed 21/10/2025

           Communication is Essential.

        Without it what would we have?


# J2534 Python Library

A comprehensive Python implementation of the SAE J2534 PassThru API for automotive diagnostics and ECU programming.

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Architecture](#architecture)
- [API Reference](#api-reference)
  - [Core Classes](#core-classes)
  - [Enumerations](#enumerations)
  - [Structures](#structures)
  - [Interfaces](#interfaces)
- [Usage Examples](#usage-examples)
- [Error Handling](#error-handling)
- [Troubleshooting](#troubleshooting)
- [Advanced Topics](#advanced-topics)
- [License](#license)

## Overview

The J2534 Python Library provides a complete Python interface to SAE J2534-compliant PassThru devices. J2534 is a standard API that allows software applications to communicate with vehicle Electronic Control Units (ECUs) through a PassThru device.

This library is a direct Python port from C#, maintaining exact naming conventions and structure for compatibility and ease of migration.

### What is J2534?

J2534 (SAE J2534) is a standard that defines an interface between a computer application and a vehicle communication interface device. It enables:

- **Vehicle Diagnostics**: Read and clear diagnostic trouble codes (DTCs)
- **ECU Programming**: Flash and reprogram vehicle control modules
- **Data Logging**: Monitor real-time vehicle data
- **Network Analysis**: Analyze CAN, ISO-TP, and other automotive protocols

## Features

- ✅ **Complete J2534 API Implementation**: Full support for all J2534 v04.04 and v05.00 functions
- ✅ **Multiple Protocol Support**: CAN, ISO15765, ISO9141, ISO14230, J1850 VPW/PWM, and more
- ✅ **Extended Functions**: Battery voltage reading, buffer management, configuration
- ✅ **Type Safety**: Extensive use of Python enums and type hints
- ✅ **Cross-Platform**: Works on Windows (with native J2534 DLLs) and can be extended for Linux
- ✅ **Memory Management**: Proper ctypes memory handling for safe native interop
- ✅ **Exact C# Compatibility**: Preserved naming conventions for easy migration

## Requirements

### System Requirements

- Python 3.7 or higher
- Windows OS (for native J2534 DLL support)
- A J2534-compliant PassThru device and its driver DLL

### Python Dependencies

- `ctypes` (standard library)
- `enum` (standard library)
- `typing` (standard library)

No external dependencies required!

## Installation

1. **Copy the J2534.py file to your project directory**:

```bash
cp J2534.py /path/to/your/project/
```

2. **Import in your Python code**:

```python
from J2534 import *
```

3. **Ensure your J2534 device drivers are installed** on your system.

## Quick Start

### Basic Connection Example

```python
from J2534 import *

# Create a J2534 device instance
device = J2534Device()
device.Name = "My PassThru Device"
device.FunctionLibrary = "C:\\Path\\To\\J2534Device.dll"

# Create J2534 functions wrapper
j2534 = J2534Functions()

# Load the device library
if j2534.LoadLibrary(device):
    print("Library loaded successfully!")
    
    # Open device
    deviceId = c_uint(0)
    result = j2534.PassThruOpen(c_void_p(0), deviceId)
    
    if result == J2534Err.STATUS_NOERROR:
        print(f"Device opened with ID: {deviceId.value}")
        
        # Connect to CAN bus
        channelId = c_uint(0)
        result = j2534.PassThruConnect(
            deviceId.value,
            ProtocolID.ISO15765,
            ConnectFlag.NONE,
            BaudRate.ISO15765_500000,
            channelId
        )
        
        if result == J2534Err.STATUS_NOERROR:
            print(f"Connected to bus with channel ID: {channelId.value}")
            
            # Your diagnostic operations here...
            
            # Disconnect when done
            j2534.PassThruDisconnect(channelId.value)
        
        # Close device
        j2534.PassThruClose(deviceId.value)
    
    # Free library
    j2534.FreeLibrary()
else:
    print("Failed to load library!")
```

### Sending a Message

```python
# Create a message
msg = PassThruMsg(
    ProtocolID.ISO15765,
    TxFlag.NONE,
    bytes([0x00, 0x00, 0x07, 0xE0, 0x01, 0x00])  # Example OBD-II request
)

# Send the message
msgPtr = Utils.ToIntPtr(msg)
numMsgs = c_int(1)
timeout = 1000  # ms

result = j2534.PassThruWriteMsgs(
    channelId.value,
    msgPtr,
    numMsgs,
    timeout
)

if result == J2534Err.STATUS_NOERROR:
    print(f"Message sent successfully! {numMsgs.value} messages transmitted.")
```

### Reading Messages

```python
# Allocate memory for received messages
numMsgs = c_int(10)
rxMsgs = cast(create_string_buffer(sizeof(PassThruMsg) * numMsgs.value), c_void_p)
timeout = 1000  # ms

result = j2534.PassThruReadMsgs(
    channelId.value,
    rxMsgs,
    numMsgs,
    timeout
)

if result == J2534Err.STATUS_NOERROR:
    messages = Utils.AsMsgList(rxMsgs, numMsgs.value)
    
    for msg in messages:
        print(f"Received message: {msg.GetBytes().hex()}")
```

## Architecture

### Library Structure

```
J2534.py
├── Enumerations
│   ├── ConfigParameter       # Configuration parameters
│   ├── ProtocolID           # Communication protocols
│   ├── BaudRate             # Baud rate definitions
│   ├── J2534Err             # Error codes
│   ├── TxFlag               # Transmission flags
│   ├── RxStatus             # Reception status flags
│   ├── ConnectFlag          # Connection flags
│   ├── FilterType           # Message filter types
│   ├── PinNumber            # Pin numbering
│   ├── PinVoltage           # Voltage levels
│   ├── Ioctl                # IOCTL commands
│   └── GET_DEVICE_INFO_Defines
│
├── Interfaces
│   ├── IJ2534               # Base J2534 interface
│   └── IJ2534Extended       # Extended functionality
│
├── Core Classes
│   ├── J2534Device          # Device information
│   ├── J2534Functions       # Standard J2534 functions
│   ├── J2534FunctionsExtended # Extended functions
│   ├── J2534DllWrapper      # DLL loading and function pointers
│   └── J2534Exception       # Exception handling
│
├── Structures
│   ├── PassThruMsg          # Message structure
│   ├── SConfig              # Configuration structure
│   ├── SConfigList          # Configuration list
│   ├── SParam               # Parameter structure
│   ├── SParamList           # Parameter list
│   ├── SByteArray           # Byte array structure
│   ├── SDEVICE              # Device structure
│
└── Utilities
    ├── NativeMethods        # Native DLL interaction
    └── Utils                # Helper functions
```

## API Reference

### Core Classes

#### J2534Device

Represents a J2534 PassThru device.

```python
class J2534Device:
    def __init__(self):
        self.Vendor = ""              # Device vendor name
        self.Name = ""                # Device name
        self.FunctionLibrary = ""     # Path to device DLL
        self.ConfigApplication = ""   # Configuration application path
        # Protocol support flags
        self.CAN = 0
        self.ISO14230 = 0
        self.ISO15765 = 0
        self.ISO9141 = 0
        self.J1850PWM = 0
        self.J1850VPW = 0
        # ... additional protocol flags
```

**Properties:**
- `Vendor` (str): Manufacturer name
- `Name` (str): Device model name
- `FunctionLibrary` (str): Full path to the J2534 DLL
- Protocol support properties indicate which protocols are supported (0 = not supported, >0 = supported)

#### J2534Functions

Main class for J2534 API operations.

```python
class J2534Functions(IJ2534):
    def LoadLibrary(self, device: J2534Device) -> bool
    def FreeLibrary(self) -> bool
    
    def PassThruOpen(self, name: c_void_p, deviceId: c_uint) -> J2534Err
    def PassThruClose(self, deviceId: c_uint) -> J2534Err
    def PassThruConnect(self, deviceId: c_uint, protocolId: ProtocolID, 
                       flags: ConnectFlag, baudRate: BaudRate, 
                       channelId: c_uint) -> J2534Err
    def PassThruDisconnect(self, channelId: c_int) -> J2534Err
    
    def PassThruReadMsgs(self, channelId: c_int, msgs: c_void_p, 
                        numMsgs: c_int, timeout: c_int) -> J2534Err
    def PassThruWriteMsgs(self, channelId: c_int, msgs: c_void_p, 
                         numMsgs: c_int, timeout: c_int) -> J2534Err
    
    def PassThruStartPeriodicMsg(self, channelId: c_int, msg: c_void_p, 
                                msgId: c_int, timeInterval: c_int) -> J2534Err
    def PassThruStopPeriodicMsg(self, channelId: c_int, msgId: c_int) -> J2534Err
    
    def PassThruStartMsgFilter(self, channelid: c_int, filterType: FilterType,
                              maskMsg: c_void_p, patternMsg: c_void_p,
                              flowControlMsg: c_void_p, filterId: c_int) -> J2534Err
    def PassThruStopMsgFilter(self, channelId: c_int, filterId: c_int) -> J2534Err
    
    def PassThruSetProgrammingVoltage(self, deviceId: c_int, 
                                     pinNumber: PinNumber, 
                                     voltage: c_uint) -> J2534Err
    def PassThruReadVersion(self, deviceId: c_int, firmwareVersion: c_void_p,
                          dllVersion: c_void_p, apiVersion: c_void_p) -> J2534Err
    def PassThruGetLastError(self, errorDescription: c_void_p) -> J2534Err
    def PassThruIoctl(self, channelId: c_int, ioctlID: c_int, 
                     input: c_void_p, output: c_void_p) -> J2534Err
```

**Key Methods:**

##### LoadLibrary(device: J2534Device) -> bool
Loads the J2534 DLL specified in the device configuration.

**Parameters:**
- `device`: J2534Device object with FunctionLibrary path set

**Returns:**
- `True` if library loaded successfully
- `False` if loading failed

##### PassThruOpen(name: c_void_p, deviceId: c_uint) -> J2534Err
Opens a communication connection to the PassThru device.

**Parameters:**
- `name`: Reserved for future use (pass `c_void_p(0)`)
- `deviceId`: Reference to receive the device ID

**Returns:**
- `J2534Err.STATUS_NOERROR` on success
- Error code on failure

##### PassThruConnect(deviceId, protocolId, flags, baudRate, channelId) -> J2534Err
Establishes a logical communication channel using a specific protocol.

**Parameters:**
- `deviceId`: Device ID from PassThruOpen
- `protocolId`: Communication protocol (ProtocolID enum)
- `flags`: Connection flags (ConnectFlag enum)
- `baudRate`: Communication speed (BaudRate enum)
- `channelId`: Reference to receive the channel ID

**Returns:**
- `J2534Err.STATUS_NOERROR` on success
- Error code on failure

##### PassThruWriteMsgs(channelId, msgs, numMsgs, timeout) -> J2534Err
Transmits messages on the specified channel.

**Parameters:**
- `channelId`: Channel ID from PassThruConnect
- `msgs`: Pointer to PassThruMsg array
- `numMsgs`: Reference to number of messages (input/output)
- `timeout`: Timeout in milliseconds

**Returns:**
- `J2534Err.STATUS_NOERROR` on success
- Error code on failure

##### PassThruReadMsgs(channelId, msgs, numMsgs, timeout) -> J2534Err
Receives messages from the specified channel.

**Parameters:**
- `channelId`: Channel ID from PassThruConnect
- `msgs`: Pointer to receive message array
- `numMsgs`: Reference to number of messages (input/output)
- `timeout`: Timeout in milliseconds

**Returns:**
- `J2534Err.STATUS_NOERROR` on success
- `J2534Err.ERR_BUFFER_EMPTY` if no messages available
- Other error codes on failure

#### J2534FunctionsExtended

Extended functionality beyond the standard J2534 API.

```python
class J2534FunctionsExtended(J2534Functions, IJ2534Extended):
    def GetConfig(self, channelId: c_int, config: List[SConfig]) -> J2534Err
    def SetConfig(self, channelId: c_int, config: List[SConfig]) -> J2534Err
    
    def ReadBatteryVoltage(self, deviceId: c_int, voltage: c_int) -> J2534Err
    def ReadProgrammingVoltage(self, deviceId: c_int, voltage: c_int) -> J2534Err
    
    def SW_CAN_BusSpeed(self, ChannelID: c_int, Option: c_int) -> J2534Err
    
    def FiveBaudInit(self, channelId: c_int, targetAddress: c_byte,
                    keyword1: c_byte, keyword2: c_byte) -> J2534Err
    def FastInit(self, channelId: c_int, txMsg: PassThruMsg, 
                rxMsg: PassThruMsg) -> J2534Err
    
    def ClearTxBuffer(self, channelId: c_int) -> J2534Err
    def ClearRxBuffer(self, channelId: c_int) -> J2534Err
    def ClearPeriodicMsgs(self, channelId: c_int) -> J2534Err
    def ClearMsgFilters(self, channelId: c_int) -> J2534Err
    
    def ReadAllMessages(self, channelId: c_int, numMsgs: c_int, 
                       timeout: c_int, messages: List[PassThruMsg],
                       readUntilTimeout: bool = True) -> J2534Err
```

**Extended Methods:**

##### GetConfig(channelId: c_int, config: List[SConfig]) -> J2534Err
Retrieves current configuration parameters.

**Example:**
```python
config = [
    SConfig(ConfigParameter.DATA_RATE, 0),
    SConfig(ConfigParameter.LOOPBACK, 0)
]
result = j2534_ext.GetConfig(channelId.value, config)
if result == J2534Err.STATUS_NOERROR:
    print(f"Data Rate: {config[0].Value}")
    print(f"Loopback: {config[1].Value}")
```

##### SetConfig(channelId: c_int, config: List[SConfig]) -> J2534Err
Sets configuration parameters.

**Example:**
```python
config = [
    SConfig(ConfigParameter.LOOPBACK, 1),  # Enable loopback
    SConfig(ConfigParameter.DATA_RATE, 500000)  # Set 500k baud
]
result = j2534_ext.SetConfig(channelId.value, config)
```

##### ReadBatteryVoltage(deviceId: c_int, voltage: c_int) -> J2534Err
Reads the vehicle battery voltage.

**Example:**
```python
voltage = c_int(0)
result = j2534_ext.ReadBatteryVoltage(deviceId.value, voltage)
if result == J2534Err.STATUS_NOERROR:
    voltage_mv = voltage.value
    voltage_v = voltage_mv / 1000.0
    print(f"Battery Voltage: {voltage_v:.2f}V")
```

##### ReadAllMessages(channelId, numMsgs, timeout, messages, readUntilTimeout) -> J2534Err
Continuously reads messages until timeout or buffer empty.

**Parameters:**
- `channelId`: Channel ID
- `numMsgs`: Maximum messages to read per call
- `timeout`: Timeout in milliseconds
- `messages`: List to populate with received messages
- `readUntilTimeout`: If True, read until timeout; if False, read one batch

**Example:**
```python
messages = []
result = j2534_ext.ReadAllMessages(
    channelId.value,
    10,  # Read up to 10 messages at a time
    1000,  # 1 second timeout
    messages,
    readUntilTimeout=True
)

for msg in messages:
    print(f"Received: {msg.GetBytes().hex()}")
```

### Enumerations

#### ProtocolID

Defines supported communication protocols.

```python
class ProtocolID(IntEnum):
    J1850VPW = 0x01                # J1850 VPW (GM)
    J1850PWM = 0x02                # J1850 PWM (Ford)
    ISO9141 = 0x03                 # ISO 9141
    ISO14230 = 0x04                # ISO 14230 (KWP2000)
    CAN = 0x05                     # Raw CAN
    ISO15765 = 0x06                # ISO 15765 (ISO-TP/UDS)
    SCI_A_ENGINE = 0x07            # Chrysler SCI (Engine)
    SCI_A_TRANS = 0x08             # Chrysler SCI (Transmission)
    SCI_B_ENGINE = 0x09            # Chrysler SCI (Engine)
    SCI_B_TRANS = 0x0A             # Chrysler SCI (Transmission)
    ISO15765_LOGICAL = 0x200       # ISO 15765 Logical
    # J2534-2 protocols
    J1850VPW_PS = 0x8000           # J1850 VPW Passive
    J1850PWM_PS = 0x8001           # J1850 PWM Passive
    ISO9141_PS = 0x8002            # ISO 9141 Passive
    ISO14230_PS = 0x8003           # ISO 14230 Passive
    CAN_PS = 0x8004                # CAN Passive
    ISO15765_PS = 0x8005           # ISO 15765 Passive
    # ... additional protocols
```

#### J2534Err

Error codes returned by J2534 functions.

```python
class J2534Err(IntEnum):
    STATUS_NOERROR = 0x00                      # Success
    ERR_NOT_SUPPORTED = 0x01                   # Function not supported
    ERR_INVALID_CHANNEL_ID = 0x02              # Invalid channel ID
    ERR_INVALID_PROTOCOL_ID = 0x03             # Invalid protocol ID
    ERR_NULL_PARAMETER = 0x04                  # NULL pointer supplied
    ERR_INVALID_FLAGS = 0x06                   # Invalid flags
    ERR_FAILED = 0x07                          # Undefined error
    ERR_ConnectedDevice_NOT_CONNECTED = 0x08   # Device not connected
    ERR_TIMEOUT = 0x09                         # Timeout
    ERR_INVALID_MSG = 0x0A                     # Invalid message
    ERR_INVALID_TIME_INTERVAL = 0x0B           # Invalid time interval
    ERR_EXCEEDED_LIMIT = 0x0C                  # Exceeded limit
    ERR_INVALID_MSG_ID = 0x0D                  # Invalid message ID
    ERR_ConnectedDevice_IN_USE = 0x0E          # Device in use
    ERR_INVALID_IOCTL_ID = 0x0F                # Invalid IOCTL ID
    ERR_BUFFER_EMPTY = 0x10                    # Buffer empty
    ERR_BUFFER_FULL = 0x11                     # Buffer full
    ERR_BUFFER_OVERFLOW = 0x12                 # Buffer overflow
    ERR_PIN_INVALID = 0x13                     # Invalid pin number
    ERR_CHANNEL_IN_USE = 0x14                  # Channel in use
    ERR_MSG_PROTOCOL_ID = 0x15                 # Protocol mismatch
    ERR_INVALID_FILTER_ID = 0x16               # Invalid filter ID
    ERR_NO_FLOW_CONTROL = 0x17                 # No flow control filter
    ERR_NOT_UNIQUE = 0x18                      # Not unique
    ERR_INVALID_BAUDRATE = 0x19                # Invalid baud rate
    ERR_INVALID_ConnectedDevice_ID = 0x1A      # Invalid device ID
    # ... additional error codes
```

#### BaudRate

Common baud rate values for different protocols.

```python
class BaudRate(IntEnum):
    ISO9141_10400 = 10400
    ISO9141_10000 = 10000
    ISO14230_10400 = 10400
    ISO14230_10000 = 10000
    J1850PWM_41600 = 41600
    J1850PWM_83300 = 83300
    J1850VPW_10400 = 10400
    J1850VPW_41600 = 41600
    CAN_125000 = 125000
    CAN_250000 = 250000
    CAN_500000 = 500000
    CAN_33333 = 33333
    CAN_83333 = 83333
    ISO15765_125000 = 125000
    ISO15765_250000 = 250000
    ISO15765_500000 = 500000
    GMUART_8192 = 8192
    GMUART_160 = 160
```

#### TxFlag

Transmission flags for PassThruMsg.

```python
class TxFlag(Flag):
    NONE = 0x00000000
    SCI_TX_VOLTAGE = 0x00800000       # SCI high voltage (12V)
    SCI_MODE = 0x00400000             # SCI mode
    WAIT_P3_MIN_ONLY = 0x00000200     # Wait P3_MIN only
    CAN_29BIT_ID = 0x00000100         # Use 29-bit CAN identifier
    ISO15765_ADDR_TYPE = 0x00000080   # ISO15765 addressing type
    ISO15765_FRAME_PAD = 0x00000040   # Pad ISO15765 frame
    SW_CAN_HV_TX = 0x00000400         # SW-CAN high voltage TX
```

#### RxStatus

Reception status flags for PassThruMsg.

```python
class RxStatus(Flag):
    NONE = 0x00000000
    TX_MSG_TYPE = 0x00000001          # Message is from TX
    START_OF_MESSAGE = 0x00000002     # First message
    RX_BREAK = 0x00000004             # Break received
    TX_INDICATION_SUCCESS = 0x00000008 # TX successful
    ISO15765_PADDING_ERROR = 0x00000010 # Padding error
    ERROR_INDICATION = 0x20           # Error occurred
    BUFFER_OVERFLOW = 0x40            # Buffer overflow
    ISO15765_ADDR_TYPE = 0x00000080   # ISO15765 addressing
    CAN_29BIT_ID = 0x00000100         # 29-bit CAN ID
    TX_FAILED = 0x200                 # TX failed
    SW_CAN_HV_TX = 0x00000400         # SW-CAN HV TX
    SW_CAN_NS_RX = 0x00040000         # SW-CAN normal speed RX
    SW_CAN_HS_RX = 0x00020000         # SW-CAN high speed RX
    SW_CAN_HV_RX = 0x00010000         # SW-CAN high voltage RX
```

#### ConnectFlag

Connection flags for PassThruConnect.

```python
class ConnectFlag(Flag):
    NONE = 0x0000
    ISO9141_K_LINE_ONLY = 0x1000      # Use K-line only (no L-line)
    CAN_ID_BOTH = 0x0800              # 11-bit and 29-bit CAN
    ISO9141_NO_CHECKSUM = 0x0200      # No checksum
    CAN_29BIT_ID = 0x0100             # Use 29-bit CAN IDs
    FULL_DUPLEX = 0x1                 # Full duplex mode
```

#### ConfigParameter

Configuration parameters for GetConfig/SetConfig.

```python
class ConfigParameter(IntEnum):
    DATA_RATE = 0x01                  # Baud rate value
    LOOPBACK = 0x03                   # Loopback mode (0=off, 1=on)
    NODE_ADDRESS = 0x04               # Node address for J1850
    NETWORK_LINE = 0x05               # Network line (0=BUS_NORMAL, 1=BUS_PLUS, 2=BUS_MINUS)
    P1_MIN = 0x06                     # ISO 9141 P1 min timing
    P1_MAX = 0x07                     # ISO 9141 P1 max timing
    P2_MIN = 0x08                     # ISO 9141 P2 min timing
    P2_MAX = 0x09                     # ISO 9141 P2 max timing
    P3_MIN = 0x0A                     # ISO 9141 P3 min timing
    P3_MAX = 0x0B                     # ISO 9141 P3 max timing
    P4_MIN = 0x0C                     # ISO 9141 P4 min timing
    P4_MAX = 0x0D                     # ISO 9141 P4 max timing
    W1 = 0x0E                         # ISO 9141 W1 timing
    W2 = 0x0F                         # ISO 9141 W2 timing
    W3 = 0x10                         # ISO 9141 W3 timing
    W4 = 0x11                         # ISO 9141 W4 timing
    W5 = 0x12                         # ISO 9141 W5 timing
    TIDLE = 0x13                      # Bus idle time
    TINIL = 0x14                      # Initialization time
    TWUP = 0x15                       # Wakeup time
    PARITY = 0x16                     # Parity (0=no parity, 1=odd, 2=even)
    BIT_SAMPLE_POINT = 0x17           # CAN bit sample point (%)
    SYNC_JUMP_WIDTH = 0x18            # CAN sync jump width
    # ISO15765 parameters
    ISO15765_BS = 0x1E                # Block size
    ISO15765_STMIN = 0x1F             # Separation time minimum
    ISO15765_WFT_MAX = 0x25           # Wait frame transmit maximum
    ISO15765_PAD_VALUE = 0X2B         # Padding value
    # ... many more parameters available
```

### Structures

#### PassThruMsg

Main message structure for J2534 communication.

```python
class PassThruMsg(Structure):
    _fields_ = [
        ("ProtocolID", c_int),         # Protocol identifier
        ("RxStatus", c_uint),          # Reception status flags
        ("TxFlags", c_uint),           # Transmission flags
        ("Timestamp", c_uint),         # Message timestamp (microseconds)
        ("DataSize", c_uint),          # Number of data bytes
        ("ExtraDataIndex", c_uint),    # Start of extra data
        ("Data", c_byte * 4128)        # Message data (max 4128 bytes)
    ]
    
    def __init__(self, myProtocolId: ProtocolID = None, 
                 myTxFlag: TxFlag = None, 
                 myByteArray: bytes = None)
    
    def SetBytes(self, myByteArray: bytes)
    def GetBytes(self) -> bytes
    def ToString(self, tab: str) -> str
```

**Usage Example:**
```python
# Create a CAN message
msg = PassThruMsg(
    ProtocolID.CAN,
    TxFlag.CAN_29BIT_ID,
    bytes([0x18, 0xDA, 0xF1, 0x10, 0x02, 0x01, 0x00])
)

# Access message properties
print(f"Protocol: {msg.ProtocolID}")
print(f"Data Size: {msg.DataSize}")
print(f"Data: {msg.GetBytes().hex()}")

# Modify message data
msg.SetBytes(bytes([0x18, 0xDA, 0xF1, 0x10, 0x02, 0x01, 0x01]))
```

#### SConfig

Configuration parameter structure.

```python
class SConfig(Structure):
    _fields_ = [
        ("Parameter", c_int),          # ConfigParameter enum value
        ("Value", c_uint16)            # Parameter value
    ]
```

**Usage Example:**
```python
# Create configuration
config = SConfig()
config.Parameter = ConfigParameter.DATA_RATE
config.Value = 500000

# Or use constructor
config = SConfig(ConfigParameter.LOOPBACK, 1)
```

#### SByteArray

Byte array structure for IOCTL operations.

```python
class SByteArray(Structure):
    _fields_ = [
        ("NumOfBytes", c_int),         # Number of bytes
        ("BytePtr", c_void_p)          # Pointer to byte array
    ]
```

### Utilities

#### Utils Class

Helper methods for working with J2534 structures.

```python
class Utils:
    @staticmethod
    def AsString(ptr: c_void_p) -> str
        """Convert IntPtr to string"""
    
    @staticmethod
    def ToIntPtr(msg: PassThruMsg) -> c_void_p
        """Convert PassThruMsg to IntPtr"""
    
    @staticmethod
    def AsMsgList(ptr: c_void_p, count: int) -> List[PassThruMsg]
        """Convert IntPtr to PassThruMsg list"""
    
    @staticmethod
    def AsList(ptr: c_void_p, count: int, struct_type: type) -> List
        """Convert IntPtr to list of structures"""
    
    @staticmethod
    def AsStruct(ptr: c_void_p, struct_type: type)
        """Convert IntPtr to structure"""
    
    @staticmethod
    def AsNullableStruct(ptr: c_void_p, struct_type: type) -> Optional
        """Convert IntPtr to nullable structure"""
```

## Usage Examples

### Example 1: Read VIN from Vehicle

```python
from J2534 import *

def read_vin():
    # Setup
    device = J2534Device()
    device.FunctionLibrary = "C:\\Program Files\\...\\J2534.dll"
    
    j2534 = J2534Functions()
    
    if not j2534.LoadLibrary(device):
        print("Failed to load library")
        return
    
    # Open device
    deviceId = c_uint(0)
    result = j2534.PassThruOpen(c_void_p(0), deviceId)
    if result != J2534Err.STATUS_NOERROR:
        print(f"Failed to open device: {result}")
        return
    
    # Connect to ISO15765 (CAN)
    channelId = c_uint(0)
    result = j2534.PassThruConnect(
        deviceId.value,
        ProtocolID.ISO15765,
        ConnectFlag.NONE,
        BaudRate.ISO15765_500000,
        channelId
    )
    
    if result != J2534Err.STATUS_NOERROR:
        print(f"Failed to connect: {result}")
        j2534.PassThruClose(deviceId.value)
        return
    
    try:
        # Set up flow control filter (required for ISO15765)
        maskMsg = PassThruMsg(ProtocolID.ISO15765, TxFlag.NONE, 
                             bytes([0x00, 0x00, 0x00, 0x00]))
        patternMsg = PassThruMsg(ProtocolID.ISO15765, TxFlag.NONE,
                                bytes([0x00, 0x00, 0x07, 0xE8]))
        flowMsg = PassThruMsg(ProtocolID.ISO15765, TxFlag.NONE,
                             bytes([0x00, 0x00, 0x07, 0xE0]))
        
        filterId = c_int(0)
        result = j2534.PassThruStartMsgFilter(
            channelId.value,
            FilterType.FLOW_CONTROL_FILTER,
            Utils.ToIntPtr(maskMsg),
            Utils.ToIntPtr(patternMsg),
            Utils.ToIntPtr(flowMsg),
            filterId
        )
        
        if result != J2534Err.STATUS_NOERROR:
            print(f"Failed to start filter: {result}")
            return
        
        # Send VIN request (Service 09, PID 02)
        vinRequest = PassThruMsg(
            ProtocolID.ISO15765,
            TxFlag.NONE,
            bytes([0x00, 0x00, 0x07, 0xDF, 0x09, 0x02])
        )
        
        txMsgPtr = Utils.ToIntPtr(vinRequest)
        numMsgs = c_int(1)
        result = j2534.PassThruWriteMsgs(
            channelId.value,
            txMsgPtr,
            numMsgs,
            1000
        )
        
        if result != J2534Err.STATUS_NOERROR:
            print(f"Failed to send request: {result}")
            return
        
        # Read response
        numMsgs = c_int(10)
        rxMsgs = cast(create_string_buffer(sizeof(PassThruMsg) * numMsgs.value), c_void_p)
        result = j2534.PassThruReadMsgs(
            channelId.value,
            rxMsgs,
            numMsgs,
            2000
        )
        
        if result == J2534Err.STATUS_NOERROR:
            messages = Utils.AsMsgList(rxMsgs, numMsgs.value)
            for msg in messages:
                data = msg.GetBytes()
                # Skip addressing bytes, extract VIN
                if len(data) > 5 and data[4] == 0x49 and data[5] == 0x02:
                    vin_bytes = data[6:]
                    vin = vin_bytes.decode('ascii', errors='ignore')
                    print(f"VIN: {vin}")
        
        # Cleanup filter
        j2534.PassThruStopMsgFilter(channelId.value, filterId.value)
        
    finally:
        # Disconnect and close
        j2534.PassThruDisconnect(channelId.value)
        j2534.PassThruClose(deviceId.value)
        j2534.FreeLibrary()

if __name__ == "__main__":
    read_vin()
```

### Example 2: Monitor CAN Bus

```python
from J2534 import *
import time

def monitor_can_bus(duration_seconds=10):
    device = J2534Device()
    device.FunctionLibrary = "C:\\Path\\To\\J2534.dll"
    
    j2534_ext = J2534FunctionsExtended()
    
    if not j2534_ext.LoadLibrary(device):
        print("Failed to load library")
        return
    
    deviceId = c_uint(0)
    result = j2534_ext.PassThruOpen(c_void_p(0), deviceId)
    if result != J2534Err.STATUS_NOERROR:
        print(f"Failed to open: {result}")
        return
    
    # Connect in passive mode (sniff mode)
    channelId = c_uint(0)
    result = j2534_ext.PassThruConnect(
        deviceId.value,
        ProtocolID.CAN,
        ConnectFlag.NONE,  # Passive listening
        BaudRate.CAN_500000,
        channelId
    )
    
    if result != J2534Err.STATUS_NOERROR:
        print(f"Failed to connect: {result}")
        j2534_ext.PassThruClose(deviceId.value)
        return
    
    try:
        # Set up pass-all filter to receive everything
        maskMsg = PassThruMsg(ProtocolID.CAN, TxFlag.NONE,
                             bytes([0x00, 0x00, 0x00, 0x00]))
        patternMsg = PassThruMsg(ProtocolID.CAN, TxFlag.NONE,
                                bytes([0x00, 0x00, 0x00, 0x00]))
        
        filterId = c_int(0)
        result = j2534_ext.PassThruStartMsgFilter(
            channelId.value,
            FilterType.PASS_FILTER,
            Utils.ToIntPtr(maskMsg),
            Utils.ToIntPtr(patternMsg),
            c_void_p(0),
            filterId
        )
        
        print(f"Monitoring CAN bus for {duration_seconds} seconds...")
        print("CAN ID       Data")
        print("-" * 50)
        
        start_time = time.time()
        message_count = 0
        
        while time.time() - start_time < duration_seconds:
            messages = []
            result = j2534_ext.ReadAllMessages(
                channelId.value,
                50,  # Read up to 50 messages
                100,  # 100ms timeout
                messages,
                readUntilTimeout=False  # Single batch
            )
            
            for msg in messages:
                data = msg.GetBytes()
                can_id = int.from_bytes(data[0:4], 'big') & 0x1FFFFFFF
                payload = data[4:]
                print(f"0x{can_id:08X}   {payload.hex().upper()}")
                message_count += 1
        
        print(f"\nReceived {message_count} messages")
        
        j2534_ext.PassThruStopMsgFilter(channelId.value, filterId.value)
        
    finally:
        j2534_ext.PassThruDisconnect(channelId.value)
        j2534_ext.PassThruClose(deviceId.value)
        j2534_ext.FreeLibrary()

if __name__ == "__main__":
    monitor_can_bus(duration_seconds=30)
```

### Example 3: Read Diagnostic Trouble Codes (DTCs)

```python
from J2534 import *

def read_dtcs():
    """Read DTCs using UDS (ISO 14229) over ISO15765"""
    device = J2534Device()
    device.FunctionLibrary = "C:\\Path\\To\\J2534.dll"
    
    j2534 = J2534Functions()
    
    if not j2534.LoadLibrary(device):
        print("Failed to load library")
        return
    
    deviceId = c_uint(0)
    result = j2534.PassThruOpen(c_void_p(0), deviceId)
    if result != J2534Err.STATUS_NOERROR:
        return
    
    channelId = c_uint(0)
    result = j2534.PassThruConnect(
        deviceId.value,
        ProtocolID.ISO15765,
        ConnectFlag.NONE,
        BaudRate.ISO15765_500000,
        channelId
    )
    
    if result != J2534Err.STATUS_NOERROR:
        j2534.PassThruClose(deviceId.value)
        return
    
    try:
        # Set up filter
        maskMsg = PassThruMsg(ProtocolID.ISO15765, TxFlag.NONE,
                             bytes([0x00, 0x00, 0x00, 0x00]))
        patternMsg = PassThruMsg(ProtocolID.ISO15765, TxFlag.NONE,
                                bytes([0x00, 0x00, 0x07, 0xE8]))
        flowMsg = PassThruMsg(ProtocolID.ISO15765, TxFlag.NONE,
                             bytes([0x00, 0x00, 0x07, 0xE0]))
        
        filterId = c_int(0)
        j2534.PassThruStartMsgFilter(
            channelId.value,
            FilterType.FLOW_CONTROL_FILTER,
            Utils.ToIntPtr(maskMsg),
            Utils.ToIntPtr(patternMsg),
            Utils.ToIntPtr(flowMsg),
            filterId
        )
        
        # Send Read DTC request (Service 0x19, Sub-function 0x02)
        dtcRequest = PassThruMsg(
            ProtocolID.ISO15765,
            TxFlag.NONE,
            bytes([0x00, 0x00, 0x07, 0xE0, 0x19, 0x02, 0x0F])  # Read DTCs by status mask
        )
        
        txMsgPtr = Utils.ToIntPtr(dtcRequest)
        numMsgs = c_int(1)
        j2534.PassThruWriteMsgs(channelId.value, txMsgPtr, numMsgs, 1000)
        
        # Read response
        numMsgs = c_int(20)
        rxMsgs = cast(create_string_buffer(sizeof(PassThruMsg) * numMsgs.value), c_void_p)
        result = j2534.PassThruReadMsgs(channelId.value, rxMsgs, numMsgs, 2000)
        
        if result == J2534Err.STATUS_NOERROR:
            messages = Utils.AsMsgList(rxMsgs, numMsgs.value)
            
            for msg in messages:
                data = msg.GetBytes()
                # Parse DTC response (0x59 is positive response to 0x19)
                if len(data) > 5 and data[4] == 0x59 and data[5] == 0x02:
                    dtc_count = data[6]
                    print(f"Found {dtc_count} DTCs:")
                    
                    # Parse DTCs (3 bytes each)
                    idx = 7
                    for i in range(dtc_count):
                        if idx + 2 < len(data):
                            dtc = (data[idx] << 16) | (data[idx+1] << 8) | data[idx+2]
                            
                            # Convert to standard DTC format (P0xxx, C0xxx, etc.)
                            first_char = ['P', 'C', 'B', 'U'][(dtc >> 14) & 0x3]
                            second_digit = (dtc >> 12) & 0x3
                            remaining = dtc & 0xFFF
                            
                            dtc_string = f"{first_char}{second_digit}{remaining:03X}"
                            print(f"  DTC: {dtc_string}")
                            
                            idx += 3
        
        j2534.PassThruStopMsgFilter(channelId.value, filterId.value)
        
    finally:
        j2534.PassThruDisconnect(channelId.value)
        j2534.PassThruClose(deviceId.value)
        j2534.FreeLibrary()

if __name__ == "__main__":
    read_dtcs()
```

### Example 4: Clear DTCs

```python
from J2534 import *

def clear_dtcs():
    """Clear DTCs using UDS service 0x14"""
    device = J2534Device()
    device.FunctionLibrary = "C:\\Path\\To\\J2534.dll"
    
    j2534 = J2534Functions()
    
    if not j2534.LoadLibrary(device):
        print("Failed to load library")
        return False
    
    deviceId = c_uint(0)
    j2534.PassThruOpen(c_void_p(0), deviceId)
    
    channelId = c_uint(0)
    j2534.PassThruConnect(
        deviceId.value,
        ProtocolID.ISO15765,
        ConnectFlag.NONE,
        BaudRate.ISO15765_500000,
        channelId
    )
    
    try:
        # Set up filter
        maskMsg = PassThruMsg(ProtocolID.ISO15765, TxFlag.NONE,
                             bytes([0x00, 0x00, 0x00, 0x00]))
        patternMsg = PassThruMsg(ProtocolID.ISO15765, TxFlag.NONE,
                                bytes([0x00, 0x00, 0x07, 0xE8]))
        flowMsg = PassThruMsg(ProtocolID.ISO15765, TxFlag.NONE,
                             bytes([0x00, 0x00, 0x07, 0xE0]))
        
        filterId = c_int(0)
        j2534.PassThruStartMsgFilter(
            channelId.value,
            FilterType.FLOW_CONTROL_FILTER,
            Utils.ToIntPtr(maskMsg),
            Utils.ToIntPtr(patternMsg),
            Utils.ToIntPtr(flowMsg),
            filterId
        )
        
        # Send Clear DTC request (Service 0x14)
        clearRequest = PassThruMsg(
            ProtocolID.ISO15765,
            TxFlag.NONE,
            bytes([0x00, 0x00, 0x07, 0xE0, 0x14, 0xFF, 0xFF, 0xFF])
        )
        
        txMsgPtr = Utils.ToIntPtr(clearRequest)
        numMsgs = c_int(1)
        result = j2534.PassThruWriteMsgs(channelId.value, txMsgPtr, numMsgs, 1000)
        
        if result != J2534Err.STATUS_NOERROR:
            print(f"Failed to send clear command: {result}")
            return False
        
        # Read response
        numMsgs = c_int(5)
        rxMsgs = cast(create_string_buffer(sizeof(PassThruMsg) * numMsgs.value), c_void_p)
        result = j2534.PassThruReadMsgs(channelId.value, rxMsgs, numMsgs, 2000)
        
        if result == J2534Err.STATUS_NOERROR:
            messages = Utils.AsMsgList(rxMsgs, numMsgs.value)
            
            for msg in messages:
                data = msg.GetBytes()
                # 0x54 is positive response to 0x14
                if len(data) > 4 and data[4] == 0x54:
                    print("DTCs cleared successfully!")
                    return True
                elif len(data) > 4 and data[4] == 0x7F:
                    # Negative response
                    nrc = data[6] if len(data) > 6 else 0
                    print(f"Failed to clear DTCs. Negative response code: 0x{nrc:02X}")
                    return False
        
        print("No response received")
        return False
        
    finally:
        j2534.PassThruStopMsgFilter(channelId.value, filterId.value)
        j2534.PassThruDisconnect(channelId.value)
        j2534.PassThruClose(deviceId.value)
        j2534.FreeLibrary()

if __name__ == "__main__":
    clear_dtcs()
```

### Example 5: Battery Voltage Monitor

```python
from J2534 import *
import time

def monitor_battery_voltage(duration_seconds=60):
    """Monitor battery voltage over time"""
    device = J2534Device()
    device.FunctionLibrary = "C:\\Path\\To\\J2534.dll"
    
    j2534_ext = J2534FunctionsExtended()
    
    if not j2534_ext.LoadLibrary(device):
        print("Failed to load library")
        return
    
    deviceId = c_uint(0)
    result = j2534_ext.PassThruOpen(c_void_p(0), deviceId)
    if result != J2534Err.STATUS_NOERROR:
        print(f"Failed to open: {result}")
        return
    
    try:
        print(f"Monitoring battery voltage for {duration_seconds} seconds...")
        print("Time (s)    Voltage (V)")
        print("-" * 30)
        
        start_time = time.time()
        
        while time.time() - start_time < duration_seconds:
            voltage = c_int(0)
            result = j2534_ext.ReadBatteryVoltage(deviceId.value, voltage)
            
            if result == J2534Err.STATUS_NOERROR:
                voltage_v = voltage.value / 1000.0
                elapsed = time.time() - start_time
                print(f"{elapsed:6.1f}      {voltage_v:5.2f}")
            else:
                print(f"Error reading voltage: {result}")
            
            time.sleep(1.0)  # Read every second
    
    finally:
        j2534_ext.PassThruClose(deviceId.value)
        j2534_ext.FreeLibrary()

if __name__ == "__main__":
    monitor_battery_voltage(duration_seconds=30)
```

## Error Handling

### Using J2534Exception

```python
from J2534 import *

def safe_operation():
    j2534 = J2534Functions()
    device = J2534Device()
    device.FunctionLibrary = "C:\\Path\\To\\J2534.dll"
    
    try:
        if not j2534.LoadLibrary(device):
            raise J2534Exception(J2534Err.ERR_DLL_NOT_LOADED)
        
        deviceId = c_uint(0)
        result = j2534.PassThruOpen(c_void_p(0), deviceId)
        
        if result != J2534Err.STATUS_NOERROR:
            raise J2534Exception(result)
        
        # ... perform operations ...
        
    except J2534Exception as e:
        print(f"J2534 Error: {e.Error.name} (0x{e.Error.value:02X})")
        print(f"Description: {e}")
    
    finally:
        if j2534.IsDLLLoaded:
            j2534.FreeLibrary()
```

### Error Code Handling

```python
def handle_j2534_error(error_code: J2534Err):
    """Handle different J2534 errors"""
    error_messages = {
        J2534Err.ERR_TIMEOUT: "Operation timed out",
        J2534Err.ERR_BUFFER_EMPTY: "No messages available",
        J2534Err.ERR_BUFFER_FULL: "Buffer is full, cannot send more messages",
        J2534Err.ERR_INVALID_CHANNEL_ID: "Invalid channel ID specified",
        J2534Err.ERR_ConnectedDevice_NOT_CONNECTED: "Device is not connected",
        J2534Err.ERR_FAILED: "General failure occurred"
    }
    
    message = error_messages.get(error_code, f"Unknown error: {error_code}")
    print(f"Error: {message}")
    
    return error_code != J2534Err.STATUS_NOERROR

# Usage
result = j2534.PassThruReadMsgs(channelId.value, rxMsgs, numMsgs, timeout)
if handle_j2534_error(result):
    # Handle error
    pass
```

## Troubleshooting

### Common Issues and Solutions

#### 1. Library Fails to Load

**Problem**: `LoadLibrary()` returns `False`

**Solutions**:
- Verify the DLL path is correct and the file exists
- Check that the DLL is the correct architecture (32-bit vs 64-bit)
- Ensure Python architecture matches DLL architecture
- Verify all DLL dependencies are installed
- Try running as Administrator

```python
import os

dll_path = "C:\\Path\\To\\J2534.dll"
if not os.path.exists(dll_path):
    print(f"DLL not found: {dll_path}")
else:
    print(f"DLL exists: {dll_path}")
```

#### 2. ERR_DEVICE_NOT_CONNECTED

**Problem**: Device not physically connected or drivers not installed

**Solutions**:
- Check physical connection
- Verify device drivers are installed
- Try a different USB port
- Check Device Manager for any issues
- Restart the device

#### 3. ERR_TIMEOUT on Read

**Problem**: No response from vehicle

**Solutions**:
- Increase timeout value
- Verify vehicle is powered on (ignition on)
- Check physical connections (OBD-II cable)
- Verify correct protocol and baud rate
- Ensure message filters are set up correctly

```python
# Increase timeout
result = j2534.PassThruReadMsgs(
    channelId.value,
    rxMsgs,
    numMsgs,
    5000  # 5 second timeout instead of 1 second
)
```

#### 4. ERR_INVALID_MSG

**Problem**: Message format is incorrect

**Solutions**:
- Verify message data length
- Check protocol-specific requirements
- Ensure CAN ID format is correct (11-bit vs 29-bit)
- Verify addressing bytes for ISO15765

```python
# For ISO15765, first 4 bytes are CAN ID
# Byte 0-3: CAN ID (big-endian)
# Byte 4+: UDS/OBD data

msg = PassThruMsg(
    ProtocolID.ISO15765,
    TxFlag.NONE,
    bytes([0x00, 0x00, 0x07, 0xE0,  # CAN ID 0x7E0
           0x01, 0x00])               # Service 0x01, PID 0x00
)
```

#### 5. Memory Access Violations

**Problem**: Application crashes or ERR_ACCESS_VIOLATION

**Solutions**:
- Ensure proper use of ctypes
- Don't free memory managed by the library
- Use `byref()` for references
- Verify structure sizes match

```python
# Correct usage with byref
deviceId = c_uint(0)
result = j2534.PassThruOpen(c_void_p(0), deviceId)  # WRONG - missing byref
result = j2534.PassThruOpen(c_void_p(0), byref(deviceId))  # WRONG - actually this is handled internally

# The library handles byref internally in most cases
# Just pass the c_uint directly
deviceId = c_uint(0)
result = j2534.PassThruOpen(c_void_p(0), deviceId)
```

### Debug Mode

Enable verbose logging for debugging:

```python
import logging

logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger(__name__)

def debug_j2534_call(function_name, result):
    if result == J2534Err.STATUS_NOERROR:
        logger.debug(f"{function_name}: SUCCESS")
    else:
        logger.error(f"{function_name}: ERROR - {result.name} (0x{result.value:02X})")

# Usage
result = j2534.PassThruConnect(deviceId.value, protocol, flags, baud, channelId)
debug_j2534_call("PassThruConnect", result)
```

## Advanced Topics

### Multi-Channel Operation

```python
# Open multiple channels for different protocols
can_channel = c_uint(0)
iso9141_channel = c_uint(0)

# Connect to CAN
j2534.PassThruConnect(
    deviceId.value,
    ProtocolID.CAN,
    ConnectFlag.NONE,
    BaudRate.CAN_500000,
    can_channel
)

# Connect to ISO9141
j2534.PassThruConnect(
    deviceId.value,
    ProtocolID.ISO9141,
    ConnectFlag.NONE,
    BaudRate.ISO9141_10400,
    iso9141_channel
)

# Use both channels...

# Disconnect both
j2534.PassThruDisconnect(can_channel.value)
j2534.PassThruDisconnect(iso9141_channel.value)
```

### Periodic Messages (Tester Present)

```python
# Send periodic keep-alive message
tester_present = PassThruMsg(
    ProtocolID.ISO15765,
    TxFlag.NONE,
    bytes([0x00, 0x00, 0x07, 0xE0, 0x3E, 0x00])  # Tester Present
)

msgId = c_int(0)
timeInterval = 2000  # Every 2 seconds

result = j2534.PassThruStartPeriodicMsg(
    channelId.value,
    Utils.ToIntPtr(tester_present),
    msgId,
    timeInterval
)

# Later, stop the periodic message
j2534.PassThruStopPeriodicMsg(channelId.value, msgId.value)
```

### Message Filtering

```python
# Block specific CAN IDs
blockMask = PassThruMsg(ProtocolID.CAN, TxFlag.NONE,
                       bytes([0xFF, 0xFF, 0xFF, 0xFF]))  # Check all bits
blockPattern = PassThruMsg(ProtocolID.CAN, TxFlag.NONE,
                          bytes([0x00, 0x00, 0x05, 0x00]))  # Block ID 0x500

filterId = c_int(0)
j2534.PassThruStartMsgFilter(
    channelId.value,
    FilterType.BLOCK_FILTER,
    Utils.ToIntPtr(blockMask),
    Utils.ToIntPtr(blockPattern),
    c_void_p(0),
    filterId
)

# Pass specific range of IDs
passMask = PassThruMsg(ProtocolID.CAN, TxFlag.NONE,
                      bytes([0xFF, 0xFF, 0xF0, 0x00]))  # Check upper bits
passPattern = PassThruMsg(ProtocolID.CAN, TxFlag.NONE,
                         bytes([0x00, 0x00, 0x70, 0x00]))  # Pass 0x700-0x7FF

filterId2 = c_int(0)
j2534.PassThruStartMsgFilter(
    channelId.value,
    FilterType.PASS_FILTER,
    Utils.ToIntPtr(passMask),
    Utils.ToIntPtr(passPattern),
    c_void_p(0),
    filterId2
)
```

### Configuration Management

```python
from J2534 import *

def configure_channel(j2534_ext, channelId):
    """Configure channel parameters"""
    
    # Get current configuration
    config_get = [
        SConfig(ConfigParameter.DATA_RATE, 0),
        SConfig(ConfigParameter.LOOPBACK, 0)
    ]
    
    result = j2534_ext.GetConfig(channelId, config_get)
    if result == J2534Err.STATUS_NOERROR:
        print(f"Current baud rate: {config_get[0].Value}")
        print(f"Loopback: {config_get[1].Value}")
    
    # Set new configuration
    config_set = [
        SConfig(ConfigParameter.LOOPBACK, 0),  # Disable loopback
        SConfig(ConfigParameter.ISO15765_BS, 8),  # Block size
        SConfig(ConfigParameter.ISO15765_STMIN, 10)  # 10ms separation time
    ]
    
    result = j2534_ext.SetConfig(channelId, config_set)
    if result == J2534Err.STATUS_NOERROR:
        print("Configuration updated successfully")
    else:
        print(f"Failed to set configuration: {result}")
```

### Programming Voltage Control

```python
# Set programming voltage (for ECU flashing)
result = j2534.PassThruSetProgrammingVoltage(
    deviceId.value,
    PinNumber.PIN_6,
    PinVoltage.FEPS_VOLTAGE  # 18V
)

if result == J2534Err.STATUS_NOERROR:
    print("Programming voltage set to 18V on Pin 6")
    
    # Perform programming operations...
    
    # Turn off programming voltage
    j2534.PassThruSetProgrammingVoltage(
        deviceId.value,
        PinNumber.PIN_6,
        PinVoltage.VOLTAGE_OFF
    )
```

### Reading Device Information

```python
def get_device_version(j2534, deviceId):
    """Read device firmware and API versions"""
    firmware = create_string_buffer(80)
    dll_version = create_string_buffer(80)
    api_version = create_string_buffer(80)
    
    result = j2534.PassThruReadVersion(
        deviceId.value,
        cast(firmware, c_void_p),
        cast(dll_version, c_void_p),
        cast(api_version, c_void_p)
    )
    
    if result == J2534Err.STATUS_NOERROR:
        print(f"Firmware: {firmware.value.decode('ascii', errors='ignore')}")
        print(f"DLL Version: {dll_version.value.decode('ascii', errors='ignore')}")
        print(f"API Version: {api_version.value.decode('ascii', errors='ignore')}")
```

## Best Practices

1. **Always Check Return Values**: Every J2534 function returns a status code. Always check it.

2. **Proper Resource Management**: Use try-finally blocks to ensure resources are cleaned up.

3. **Timeouts**: Set appropriate timeouts based on expected response times.

4. **Message Filtering**: Set up proper filters before reading messages to avoid buffer overflow.

5. **Error Handling**: Implement robust error handling for production code.

6. **Testing**: Test thoroughly with your specific hardware before deployment.

7. **Documentation**: Keep SAE J2534 specification handy for reference.


## Contributing

Contributions are welcome! Please ensure:
- Exact naming conventions are maintained (for C# compatibility)
- All changes are tested with real J2534 hardware
- Documentation is updated for new features
- Error handling is robust

## Support

For issues specific to this Python implementation, please check:
1. This README for solutions
2. The SAE J2534 specification
3. Your J2534 device manufacturer's documentation

## Changelog

### Version 1.0.0 (Initial Release)
- Complete Python port from C#
- All J2534 v04.04 functions implemented
- Extended functions for battery voltage, buffer management
- Full support for all protocols
- Comprehensive error handling
- Utility classes for common operations

---

**Note**: This library requires a physical J2534-compliant PassThru device and its associated drivers to function. It is intended for automotive diagnostics and ECU programming professionals.


# ACS Smart Card I/O iOS Framework & BLETest Codebase Study Guide

> **Author**: Antigravity (iOS Programming Specialist & Code Tutor)  
> **Workspace**: `acssmcio-0.7.0-ios15-20260724`  
> **Framework Version**: ACSSmartCardIO v0.7.0 / SmartCardIO v0.1.10  
> **Target Platform**: iOS 15.0+ | Swift 5.0+ | Xcode 16+ / 26+

---

## 1. Executive Summary & Architecture Overview

Welcome! This workspace contains the official iOS Software Development Kit (SDK) and reference implementation from **Advanced Card Systems Ltd. (ACS)** for communicating with their Bluetooth Low Energy (BLE) Smart Card Readers.

Smart cards (both contact chips and contactless/NFC RFID cards) traditionally communicate via ISO/IEC 7816 standards using **APDUs** (Application Protocol Data Units). On desktops, Java formalized this under **JSR 268** (`javax.smartcardio`). ACS took an elegant architectural approach:
1. **`SmartCardIO.xcframework`**: A pure Swift port of the OpenJDK Java Smart Card I/O API (`javax.smartcardio` and `java.security.Provider`). This provides a standard, platform-independent object model (`TerminalFactory`, `CardTerminal`, `Card`, `CardChannel`, `CommandAPDU`, `ResponseAPDU`, `ATR`).
2. **`ACSSmartCardIO.xcframework`**: The hardware abstraction and BLE driver layer. It implements ACS's proprietary BLE communications, frame packing, AES-128 authentication/encryption, timeouts, battery telemetry, and device info discovery while wrapping `CoreBluetooth` into the standard `SmartCardIO` SPI (Service Provider Interface).
3. **`BLETest`**: A production-grade sample iOS application demonstrating terminal scanning, master key pairing, ATR acquisition, APDU script execution, and card state monitoring.

```
+-----------------------------------------------------------------+
|                       BLETest Sample App                        |
|   (MainViewController, CardStateMonitor, TableView UI, Logger)  |
+-------------------------------+---------------------------------+
                                |
                                v
+-----------------------------------------------------------------+
|                    SmartCardIO.xcframework                      |
|       (JSR-268 Swift Port: TerminalFactory, Card, Channel)      |
+-------------------------------+---------------------------------+
                                |
                                v
+-----------------------------------------------------------------+
|                   ACSSmartCardIO.xcframework                    |
|    (BluetoothTerminalManager, AES Crypto, Protocol Framing)     |
+-------------------------------+---------------------------------+
                                |
                                v
+-----------------------------------------------------------------+
|                      Apple CoreBluetooth                        |
|               (CBCentralManager, CBPeripheral)                  |
+-------------------------------+---------------------------------+
                                |  BLE Radio (2.4 GHz)
                                v
+-----------------------------------------------------------------+
|                 ACS Bluetooth Smart Card Reader                 |
|   (ACR1255U-J1, ACR3901U-S1, AMR220-C, ACR1555U, etc.)          |
+-------------------------------+---------------------------------+
                                |  ISO 7816 / ISO 14443
                                v
+-----------------------------------------------------------------+
|               Smart Card (ACOS3, MIFARE, EMV, etc.)             |
+-----------------------------------------------------------------+
```

---

## 2. Directory Structure & Folder Breakdown

Let's examine every directory and file in the workspace:

```
acssmcio-0.7.0-ios15-20260724/
│
├── ReadMe.txt                     # Official release notes, requirements, and version change logs
│
├── BLETest/                       # Complete Xcode sample project & embedded frameworks
│   ├── ACSSmartCardIO.xcframework # ACS Bluetooth driver framework (Device + Simulator)
│   ├── SmartCardIO.xcframework    # Standard SmartCardIO framework (Device + Simulator)
│   ├── BLETest.xcodeproj          # Xcode project configuration (targets, build phases)
│   ├── BLETest/                   # Swift source code, Storyboards, Assets, Settings
│   ├── BLETestTests/              # Unit tests
│   └── BLETestUITests/            # UI automation tests
│
├── doc/                           # Full Jazzy-generated Swift API documentation (HTML/CSS/JS)
│   ├── index.html                 # Documentation homepage
│   ├── Classes/                   # Class references (BluetoothTerminalManager, ATR, etc.)
│   ├── Protocols/                 # Protocol definitions (Card, CardTerminal, etc.)
│   └── Enums/                     # Enums (CardError, CardState, TerminalType)
│
├── scripts/                       # Sample APDU test script files
│   ├── acos3.txt                  # APDUs for testing ACS ACOS3 contact cards
│   └── mifare.txt                 # APDUs for testing MIFARE Classic 1K RFID cards
│
├── dSYM-iOS/                      # Debug symbols for native iOS devices
│   ├── ACSSmartCardIO.framework.dSYM
│   └── SmartCardIO.framework.dSYM
│
└── dSYM-iOS-Simulator/            # Debug symbols for iOS Simulator (arm64 & x86_64)
    ├── ACSSmartCardIO.framework.dSYM
    └── SmartCardIO.framework.dSYM
```

### Detailed Folder Examination

### 2.1 `ReadMe.txt`
Contains foundational documentation for the SDK:
- **Minimum Requirements**: iOS 15.0 or later; Xcode 16 / 26.
- **Hardware Supported**:
  - `ACR3901U-S1 / ACR3901T-W1`: Bluetooth contact smart card reader (SIM-sized and full-sized cards).
  - `ACR1255U-J1 / ACR1255U-J1 V2`: Bluetooth NFC contactless reader (13.56 MHz, ISO 14443 Type A & B, MIFARE, FeliCa).
  - `AMR220-C`: Bluetooth mPOS terminal with PIN pad, ICC slot, and magnetic stripe/contactless support.
  - `ACR1555U`: Latest high-speed Bluetooth contactless smart card reader.
- **Key Changes in v0.7.0**: Added `bluetoothTerminalManager(_:didDisconnect:error:)` delegate callback to promptly notify apps when readers disconnect or lose signal.

### 2.2 `doc/`
This folder is a standalone HTML documentation website generated using **Jazzy** (a popular Swift documentation tool).
- **Core Namespaces**:
  - `SmartCardIO`: Defines `ATR`, `Card`, `CardChannel`, `CardError`, `CardState`, `CardTerminal`, `CardTerminals`, `CommandAPDU`, `Provider`, `ResponseAPDU`, `TerminalFactory`, `TerminalFactorySpi`.
  - `Bluetooth`: Defines `BluetoothSmartCard`, `BluetoothTerminalManager`, `AcsBluetooth`, `Amr220cConstants`, and `BluetoothFactorySpi`.
  - `ACSSmartCardIO Options`: Defines `TerminalTimeouts` and `TransmitOptions`.

### 2.3 `scripts/`
Contains test scripts with hexadecimal command APDUs and expected response APDUs:
- [`acos3.txt`](file:///d:/Project_Codes/iOS/acssmcio-0.7.0-ios15-20260724/scripts/acos3.txt): Demonstrates card authentication (`80 20 ...`), file selection (`80 A4 ...`), and record write/read (`80 D2`, `80 B2`) on an ACOS3 cryptographic smart card.
- [`mifare.txt`](file:///d:/Project_Codes/iOS/acssmcio-0.7.0-ios15-20260724/scripts/mifare.txt): Demonstrates reading card UID (`FF CA 00 00 00`), loading sector keys (`FF 82 ...`), authenticating sector 1 (`FF 86 ...`), writing 16 bytes to block 4 (`FF D6 ...`), and reading back block 4 (`FF B0 ...`).

### 2.4 `dSYM-iOS/` & `dSYM-iOS-Simulator/`
These folders contain DWARF with dSYM debugging symbol bundles. When building in Release mode or analyzing crash reports from TestFlight/App Store, these allow mapping raw hexadecimal memory addresses in crash stack traces back to readable Swift class names, method names, and line numbers inside the precompiled vendor frameworks.

---

## 3. Deep Dive into `BLETest` Source Code

The `BLETest` project is a fully realized iOS application implementing standard UIKit architecture with delegates, table views, background threads, and document handling.

```
BLETest/BLETest/
├── AppDelegate.swift                  # Application entry point & lifecycle hooks
├── MainViewController.swift          # Core controller: terminal ops, APDUs, logging
├── TerminalListViewController.swift  # Scan, filter by model, select reader
├── MasterKeyViewController.swift      # Configure custom 16-byte AES master keys
├── TerminalTimeoutsViewController.swift# Configure custom connection & APDU timeouts
├── ProtocolViewController.swift       # Card transmission protocol switch (T=0, T=1)
├── SourceViewController.swift         # Script source selector (Direct / iTunes / Picker)
├── InputViewController.swift          # Direct manual Hex APDU entry
├── FileListViewController.swift       # List files loaded via iTunes File Sharing
├── CardStateMonitor.swift             # Dedicated background polling thread for card events
├── Hex.swift                          # High-performance byte array / hex string converter
├── Logger.swift                       # Real-time UI log buffer and file logger
├── Info.plist                         # App bundle permissions & capabilities
└── Settings.bundle                    # iOS Settings integration for APDU options
```

### 3.1 [`Hex.swift`](file:///d:/Project_Codes/iOS/acssmcio-0.7.0-ios15-20260724/BLETest/BLETest/Hex.swift)
A utility class converting between raw byte arrays (`[UInt8]`) and formatted hex strings:
- **`toHexString(buffer: [UInt8]) -> String`**: Converts bytes to space-separated uppercase hex pairs (e.g., `[0xFF, 0xCA, 0x00]` -> `"FF CA 00"`).
- **`toByteArray(hexString: String) -> [UInt8]`**: High-speed hex parser that iterates across Unicode scalars, ignores spaces/invalid characters, and assembles high and low 4-bit nibbles using bitwise shift (`value << 4`) and bitwise OR (`byte |= UInt8(value)`).

### 3.2 [`Logger.swift`](file:///d:/Project_Codes/iOS/acssmcio-0.7.0-ios15-20260724/BLETest/BLETest/Logger.swift)
A real-time logging utility designed specifically for smart card protocol analysis:
- Wraps a `UITextView` and maintains a maximum cap of **1,000 lines** (`maxLines = 1000`) to prevent memory bloating and lag during large APDU transfers.
- Ensures all UI text mutations execute on `DispatchQueue.main.async` and automatically scrolls the view to the latest entry.
- Supports **file logging**: When `openLogFile(name:)` is called, it creates a timestamped log file under `<Application_Home>/Documents/Logs/Log-YYYYMMDDHHMMSS.txt` via `FileHandle` and writes entries synchronously.
- Formats hex data with standard 16 bytes per row (`logBuffer`), matching standard hex editors and network packet analyzers.

### 3.3 [`CardStateMonitor.swift`](file:///d:/Project_Codes/iOS/acssmcio-0.7.0-ios15-20260724/BLETest/BLETest/CardStateMonitor.swift)
A crucial singleton (`CardStateMonitor.shared`) that tracks whether a smart card is currently inserted into or removed from a connected reader:
- **Threading Model**: Spawns an independent `Foundation.Thread` target per terminal (`detectCard(param:)`).
- **Detection Loop**:
  ```swift
  while !Thread.current.isCancelled {
      currState = try terminal.isCardPresent() ? .present : .absent
      if currState != prevState {
          delegate?.cardStateMonitor(self, didChangeState: terminal, ...)
      }
      prevState = currState
      if currState == .absent {
          _ = try terminal.waitForCardPresent(timeout: 1000)
      } else if currState == .present {
          _ = try terminal.waitForCardAbsent(timeout: 1000)
      }
  }
  ```
- **Observer Pattern**: When card status transitions (e.g. absent -> present), it invokes `CardStateMonitorDelegate`, enabling the UI to announce card insertion/removal in real time.

### 3.4 [`TerminalListViewController.swift`](file:///d:/Project_Codes/iOS/acssmcio-0.7.0-ios15-20260724/BLETest/BLETest/TerminalListViewController.swift)
Manages Bluetooth device discovery:
- Presents an Action Sheet (`selectTerminalType`) allowing the user to select their specific reader model (`.acr3901us1`, `.acr1255uj1`, `.amr220c`, `.acr1255uj1v2`, `.acr1555u`).
- Invokes `manager.startScan(terminalType:)`, which instructs `CBCentralManager` to scan only for BLE advertisement packets matching the specified ACS peripheral service UUIDs.
- Populates a `UITableView` with discovered `CardTerminal` objects as `BluetoothTerminalManagerDelegate` receives discovery callbacks.

### 3.5 [`MasterKeyViewController.swift`](file:///d:/Project_Codes/iOS/acssmcio-0.7.0-ios15-20260724/BLETest/BLETest/MasterKeyViewController.swift)
Bluetooth smart card readers must prevent eavesdropping over the air. ACS readers use mutual authentication and AES-128 encryption:
- Users can toggle between the manufacturer default master key or input a custom 16-byte (32-character hexadecimal) AES key.
- Validates text field inputs to only allow valid hexadecimal characters, spaces, and newlines.
- Emits changes to `MasterKeyViewControllerDelegate` to call `manager.setMasterKey(terminal:masterKey:)`.

### 3.6 [`TerminalTimeoutsViewController.swift`](file:///d:/Project_Codes/iOS/acssmcio-0.7.0-ios15-20260724/BLETest/BLETest/TerminalTimeoutsViewController.swift)
Exposes fine-grained timeout controls for smart card operations:
- `connectionTimeout`: BLE peripheral connection establishment timeout (ms).
- `powerTimeout`: Smart card reset / power down timeout (ms).
- `protocolTimeout`: Protocol parameter selection (PPS) timeout (ms).
- `apduTimeout`: Smart card command execution timeout (ms).
- `controlTimeout`: Reader escape / control command timeout (ms).
- Defaults to `TerminalTimeouts.defaultTimeout` (10,000 ms).

### 3.7 [`SourceViewController.swift`](file:///d:/Project_Codes/iOS/acssmcio-0.7.0-ios15-20260724/BLETest/BLETest/SourceViewController.swift) & [`InputViewController.swift`](file:///d:/Project_Codes/iOS/acssmcio-0.7.0-ios15-20260724/BLETest/BLETest/InputViewController.swift)
Added in v0.6.1 to provide flexible command inputs:
- **Option 0 - Input**: Direct typing of APDU commands inside `InputViewController`'s text editor.
- **Option 1 - iTunes File Sharing**: Reads script `.txt` files copied directly into the app bundle's `Documents` folder via Finder/iTunes.
- **Option 2 - Document Picker**: Opens the system `UIDocumentPickerViewController` to import `.txt` files directly from iCloud Drive, Files app, or local device storage.

### 3.8 [`MainViewController.swift`](file:///d:/Project_Codes/iOS/acssmcio-0.7.0-ios15-20260724/BLETest/BLETest/MainViewController.swift)
The central orchestrator of the entire app. It ties together the frameworks, UI, delegates, and hardware:
- **Initialization**:
  - Fetches the shared instances: `BluetoothSmartCard.shared.manager` and `BluetoothSmartCard.shared.factory`.
  - Sets up `UserDefaults` for caching per-terminal settings (`com.acs.BLETest.<TerminalName>`).
  - Registers observers for `UserDefaults.didChangeNotification`.
- **Bluetooth State Delegate (`BluetoothTerminalManagerDelegate`)**:
  - Handles `bluetoothTerminalManagerDidUpdateState`: Inspects `centralManager.state` (`.poweredOn`, `.poweredOff`, `.unauthorized`, etc.) and alerts the user if BLE permissions or Bluetooth power are missing.
  - Handles `didDiscover`: Adds new terminals to `TerminalListViewController` and loads saved master keys and timeout settings.
  - Handles `didDisconnect`: Logs terminal disconnection events and error messages.
- **Card Transmission (`Transmit` Cell Action)**:
  1. Prompts for card insertion with a 5-second countdown using `terminal.waitForCardPresent(timeout: 1000)`.
  2. Connects to the card via `let card = try terminal.connect(protocolString: protocolString)`.
  3. Displays the **ATR (Answer to Reset)** bytes returned by the card.
  4. Obtains the basic logical channel: `let channel = try card.basicChannel()`.
  5. Wraps APDU bytes into `CommandAPDU(apdu: bytes)` and calls `channel.transmit(apdu: commandAPDU)`.
  6. Compares actual response bytes with expected patterns (handling `XX` wildcards).
  7. Calculates and displays data metrics: **Bytes Sent**, **Bytes Received**, **Transfer Time (ms)**, and **Transfer Rate (bytes/sec)**.
  8. Disconnects cleanly: `try card.disconnect(reset: false)`.
- **Escape / Control Commands (`Control` Cell Action)**:
  - Connects to the reader in direct mode: `terminal.connect(protocolString: "direct")`.
  - Sends reader-level control commands via `card.transmitControlCommand(controlCode: controlCode, command: bytes)`.
- **Reader Telemetry**:
  - `manager.batteryStatus(terminal:timeout:)`: Checks if the reader is running on battery, charging, full, or USB plugged.
  - `manager.batteryLevel(terminal:timeout:)`: Obtains battery percentage (0-100%).
  - `manager.deviceInfo(terminal:type:timeout:)`: Reads BLE standard Device Information Service characteristics: System ID, Model, Serial, Firmware Revision, Hardware Revision, Software Revision, and Manufacturer Name.

### 3.9 Permissions & Capabilities (`Info.plist`)
To comply with Apple's strict privacy rules for Bluetooth:
- **`NSBluetoothAlwaysUsageDescription`**: `"$(PRODUCT_NAME) would like to use Bluetooth to connect ACS readers."`
- **`NSBluetoothPeripheralUsageDescription`**: Required for iOS backward compatibility.
- **`UIFileSharingEnabled`**: Set to `true` to allow users to drag-and-drop `.txt` APDU script files into the app via iTunes / macOS Finder.

---

## 4. Understanding Smart Card Concepts for iOS Developers

If you are new to smart cards, here is a quick primer on how the terminology maps to programming concepts:

| Term | Full Name | Explanation |
| :--- | :--- | :--- |
| **APDU** | Application Protocol Data Unit | The message packet exchanged between an app and a smart card. A command APDU contains: `CLA` (Class), `INS` (Instruction), `P1`-`P2` (Parameters), `Lc`/Data (Payload), and `Le` (Expected response length). |
| **ATR** | Answer To Reset | The first string of bytes emitted by a smart card when powered up. Identifies the card's communication capabilities, clock speeds, and manufacturer details. |
| **T=0** | Byte-oriented Half-duplex Protocol | Character-level asynchronous transmission protocol. Classic standard for SIM cards and contact smart cards. |
| **T=1** | Block-oriented Half-duplex Protocol | Packet/block transmission protocol with error detection (checksum/CRC). Standard for modern crypto cards and contactless NFC. |
| **Direct Mode** | Direct / Escape Communication | Communicating directly with the smart card **reader itself** (e.g. to configure reader LEDs, beeper, or antenna) rather than the smart card chip. |
| **SW1 / SW2** | Status Word 1 & 2 | The trailing 2 bytes in every card response. `90 00` indicates **Success**. Values like `6A 82` indicate "File Not Found", `61 XX` indicates "XX bytes still available to read", etc. |

---

## 5. Step-by-Step Code Tutorial: Using the SDK in Your Own App

Here is how you can use this framework in your own iOS project:

### Step 1: Framework Setup
1. Copy `SmartCardIO.xcframework` and `ACSSmartCardIO.xcframework` into your Xcode project.
2. In your Target's **General** settings, set both frameworks to **Embed & Sign**.
3. In `Info.plist`, add `Privacy - Bluetooth Always Usage Description`.

### Step 2: Initialize & Scan for Readers
```swift
import UIKit
import SmartCardIO
import ACSSmartCardIO

class CardScannerService: NSObject, BluetoothTerminalManagerDelegate {
    
    let manager = BluetoothSmartCard.shared.manager
    let factory = BluetoothSmartCard.shared.factory
    var activeTerminal: CardTerminal?

    override init() {
        super.init()
        manager.delegate = self
    }

    func startScanning() {
        // Choose your reader model, e.g., ACR1255U-J1 NFC Reader
        manager.startScan(terminalType: .acr1255uj1)
    }

    func stopScanning() {
        manager.stopScan()
    }

    // MARK: - BluetoothTerminalManagerDelegate
    func bluetoothTerminalManagerDidUpdateState(_ manager: BluetoothTerminalManager) {
        if manager.centralManager.state == .poweredOn {
            print("Bluetooth is powered on and ready!")
        }
    }

    func bluetoothTerminalManager(_ manager: BluetoothTerminalManager, didDiscover terminal: CardTerminal) {
        print("Discovered reader: \(terminal.name)")
        self.activeTerminal = terminal
        stopScanning()
        
        // Connect to card in background
        DispatchQueue.global(qos: .userInitiated).async {
            self.readSmartCard(terminal: terminal)
        }
    }

    func bluetoothTerminalManager(_ manager: BluetoothTerminalManager, didDisconnect terminal: CardTerminal, error: Error?) {
        print("Reader \(terminal.name) disconnected: \(error?.localizedDescription ?? "Normal")")
    }
}
```

### Step 3: Connect to Card & Transmit APDUs
```swift
extension CardScannerService {

    func readSmartCard(terminal: CardTerminal) {
        do {
            print("Waiting for card presentation...")
            // Wait up to 5 seconds for card to enter RFID field or contact slot
            guard try terminal.waitForCardPresent(timeout: 5000) else {
                print("No card detected within timeout.")
                return
            }

            // Connect using auto-negotiated protocol (*)
            let card = try terminal.connect(protocolString: "*")
            print("Card connected! Active protocol: \(card.activeProtocol)")
            print("ATR: \(Hex.toHexString(buffer: card.atr.bytes))")

            // Open logical channel
            let channel = try card.basicChannel()

            // Example: Read Card UID (Mifare / ISO14443 standard APDU: FF CA 00 00 00)
            let getUidCommand = try CommandAPDU(cla: 0xFF, ins: 0xCA, p1: 0x00, p2: 0x00, ne: 0)
            let response = try channel.transmit(apdu: getUidCommand)

            print("Status Word: \(String(format: "%02X %02X", response.sw1, response.sw2))")
            if response.sw == 0x9000 {
                let uid = response.data
                print("Card UID: \(Hex.toHexString(buffer: uid))")
            } else {
                print("Failed to read UID. Error code: \(String(format: "%04X", response.sw))")
            }

            // Disconnect and release card
            try card.disconnect(reset: false)

        } catch {
            print("Card error: \(error.localizedDescription)")
        }
    }
}
```

---

## 6. Engineering Assessment & Modernization Recommendations

The codebase is exceptionally well-structured, clean, modular, and adheres strictly to ISO 7816 and Java SmartCard I/O standards.

As a senior iOS tutor, here are some modern Swift recommendations you can consider when building new features on top of this SDK:

1. **Adopt Swift Concurrency (`async/await`)**:
   - Currently, `MainViewController` and `CardStateMonitor` rely on GCD dispatch queues (`DispatchQueue.global().async`) and manual `Foundation.Thread`.
   - Wrapping `terminal.connect()` and `channel.transmit()` in Swift `async/await` and using `AsyncStream` for card insertion events would make your UI code significantly cleaner and prevent race conditions.
2. **Value Types for APDUs**:
   - `Hex.toByteArray` and `Hex.toHexString` work well. In modern Swift, extending `Data` (`extension Data { var hexEncodedString: String { ... } }`) provides a more idiomatic Swift API.
3. **Combine / Notification Streams**:
   - Replacing delegate patterns in `CardStateMonitor` with `@Published` properties or Combine Publishers would simplify SwiftUI or modern UIKit integration.
4. **Security Scoped Bookmarks**:
   - In `MainViewController.runScript(card:url:)`, security-scoped resource access (`startAccessingSecurityScopedResource`) is properly implemented with `defer { stopAccessingSecurityScopedResource() }`. This demonstrates best practice for iOS document interaction.

---

## 7. Summary Checklist for Your Team

- [x] **SDK Versions**: ACSSmartCardIO 0.7.0, SmartCardIO 0.1.10.
- [x] **Frameworks Format**: Modern XCFrameworks supporting both ARM64 devices and Apple Silicon/Intel Simulators.
- [x] **Documentation**: Full interactive API documentation available in `doc/index.html`.
- [x] **Test Scripts**: Ready-to-run scripts in `scripts/acos3.txt` and `scripts/mifare.txt`.
- [x] **Sample App**: `BLETest` compiles out-of-the-box in Xcode for iOS 15.0+ and demonstrates all real-world flows (Scanning, Authentication, Communication, Monitoring, Logging).

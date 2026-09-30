ACS Smart Card I/O iOS Framework
Advanced Card Systems Ltd.



Introduction
------------

The framework "ACSSmartCardIO" provides classes and interfaces for communicating
with ACS Bluetooth readers. It is based on the service provider interface
(TerminalFactorySpi) from Java Smart Card I/O API defined by JSR 268 [1].

The Java Smart Card I/O API defines a Java API for communication with Smart
Cards using ISO/IEC 7816-4 APDUs. It thereby allows Java applications to
interact with applications running on the Smart Card, to store and retrieve data
on the card, etc.

The framework "SmartCardIO" contains the following source code from OpenJDK
which is ported to Swift:
- javax.smartcardio
- java.security.Provider

To install frameworks to your development environment, see the section
"Installation".

[1] https://jcp.org/en/jsr/detail?id=268



Release Notes
-------------

Version:      0.7.0
Release Date: 24/7/2026

System Requirements

- iOS 15.0 or later

Development Environment

- Xcode 26.6 or later

Supported Readers

- ACR3901U-S1/ACR3901T-W1
- ACR1255U-J1
- ACR1255U-J1 V2
- AMR220-C
- ACR1555U



Installation
------------

1. To use the framework to your project, copy the folder
   "BLETest\SmartCardIO.xcframework" and "BLETest\ACSSmartCardIO.xcframework" to
   your project folder.

2. Select General tab in the Targets and click "+" in Embedded Binaries.

3. When a dialog appears, click on "Add Other..." button to add frameworks to
   your project.



History
-------

Framework: SmartCardIO

v0.1.10 (24/7/2026)
- Update the version to 0.1.10.

v0.1.9 (4/12/2025)
- Update to recommended settings in Xcode 26.1.1.
- Update the version to 0.1.9.

v0.1.8 (22/11/2024)
- Update to recommended settings in Xcode 16.
- Change iOS deployment target to 15.0.
- Update the version to 0.1.8.

v0.1.7 (15/3/2024)
- Update to recommended settings in Xcode 15.3.
- Change iOS deployment target to 12.0.
- Update the version to 0.1.7.

v0.1.6 (18/11/2021)
- Update to recommended settings in Xcode 13.1.
- Change iOS deployment target to 9.0.
- Update the version to 0.1.6.

v0.1.5 (9/3/2020)
- Update the version to 0.1.5.
- Build xcframework which includes iOS and iOS Simulator.

v0.1.4 (6/12/2019)
- Update the version to 0.1.4.

v0.1.3 (15/10/2019)
- Update the version to 0.1.3.

v0.1.2 (28/3/2019)
- Convert to Swift 5.
- Enable Base Internationalization in the project.
- Update the version to 0.1.2.

v0.1.1 (9/10/2018)
- Convert to Swift 4.2.
- Update the version to 0.1.1.

v0.1.0 (30/8/2018)
- New release.
- Import SmartCardIO related classes.



Framework: ACSSmartCardIO

v0.7.0 (24/7/2026)
- Add BluetoothTerminalManagerDelegate.bluetoothTerminalManager(
  _:didDisconnect:error:).
- Run the delegate's bluetoothTerminalManager(_:didDisconnect:error:) when the
  terminal is disconnected.
- Update the documentation.
- Update the version to 0.7.0.

v0.6.2 (4/12/2025)
- Update to recommended settings in Xcode 26.1.1.
- Set the actual slot number to the command when transmitting APDUs to ACR1555U.
- Fix response timeout during time extension in ACR1555U.
- Synchronize the reader sequence number after connecting to ACR1555U.
- Check for response message type during ACR1555U authentication.
- Update frame length check from < 8 to < 10 in ACR1555U.
- Update the version to 0.6.2.

v0.6.1 (22/11/2024)
- Fix the issue of unconditionally getting the slot status during ACR1555U
  initialization.
- Check authentication supported in ACR1555U initialization.
- Add CCID error description to ACR1555U.
- Throw CCID error when transmitting the command to ACR1555U.
- Add ACR1255U-J1 error description for maximum authentication retries exceeded.
- Rename ACR3901U-S1 error description from "Successive authentication failed"
  to "Maximum authentication retries exceeded".
- Add ACR1555U error description for maximum authentication retries exceeded.
- Check the response error when transmitting the command to ACR1555U.
- Update to recommended settings in Xcode 16.
- Change iOS deployment target to 15.0.
- Update the version to 0.6.1.

v0.6.0 (15/3/2024)
- Add the following readers support:
  ACR1555U
- Add missing locks for disconnecting ACR3901U-S1, ACR1255U-J1 and AMR220-C.
- Add missing locks for resetting the card in AMR220-C and ACR1555U.
- Add missing locks for setting the protocol in AMR220-C and ACR1555U.
- Update to recommended settings in Xcode 15.3.
- Change iOS deployment target to 12.0.
- Update the documentation.
- Update the version to 0.6.0.

v0.5.3 (18/11/2021)
- Clear the response before writing NAK frame in AMR220-C.
- Update to recommended settings in Xcode 13.1.
- Change iOS deployment target to 9.0.
- Fix compiler warnings and errors.
- Update the documentation.
- Update the version to 0.5.3.

v0.5.2 (9/3/2020)
- Initialize maximum data rate to 75000 bps in AMR220-C ICC.
- Handle interrupt frame in AMR220-C firmware v2.02.11 or later.
- Initialize the card state to unknown in AMR220-C ICC and PICC.
- Update the card state in AMR220-C initialization.
- Break the waiting operation if the current thread is cancelled.
- Update the version to 0.5.2.
- Build xcframework which includes iOS and iOS Simulator.

v0.5.1 (6/12/2019)
- Update the version to 0.5.1.

v0.5.0 (15/10/2019)
- Lock the peripheral for disconnection in BluetoothTerminalManager.
- Change TerminalTimeouts.defaultTimeout to public.
- Add BluetoothTerminalManager.deviceInfo().
- Update the documentation.
- Update the version to 0.5.0.

v0.4.2 (28/3/2019)
- Convert to Swift 5.
- Enable Base Internationalization in the project.
- Fix close issue if exception occurred in Bluetooth CardChannel.close().
- Update the version to 0.4.2.

v0.4.1 (9/10/2018)
- Convert to Swift 4.2.
- Implement error handling and recovery in AMR220-C.
- Lock the response buffer in AMR220-C initialization.
- Update the version to 0.4.1.

v0.4.0 (30/8/2018)
- Handle the following error codes from ACR3901U-S1:
  successiveAuthenticationFailed (9)
  t1CardOperationFailed (10)
- Lock the reader capabilities in setting the protocol from ACR1255U-J1.
- Implement extended APDU for AMR220-C >= v1.06.
- Fix sequence number synchronization bug in AMR220-C.
- Change the retry count for ACK frame to 10 in AMR220-C.
- Implement authentication for AMR220-C >= v2.00.
- Check the response length in ACR3901U-S1 authentication.
- Fix the wrong response length when transmitting the command to ACR3901U-S1.
- Check the response length when decrypting the response from ACR3901U-S1,
  ACR1255U-J1 and AMR220-C.
- Add BluetoothTerminalManager.batteryStatus().
- Add BluetoothTerminalManager.batteryLevel().
- Add missing synchronization lock in Bluetooth CardTerminal initialization.
- Remove redundant authenticated check in ACR1255U-J1 initialization.
- Remove SmartCardIO related classes.
- Import SmartCardIO.
- Add AcsBluetooth.
- Add "@since 0.3.1" to Crypto.h.
- Update the documentation.
- Update the version to 0.4.0.

v0.3.1 (4/7/2018)
- Verify the response after decrypting the frame in ACR3901U-S1.
- Remove CommonCrypto module dependency.
- Ensure that locks are released properly in AMR220-C, BluetoothTerminalManager,
  Bluetooth CardTerminal and Bluetooth CardTerminals.
- Update the version to 0.3.1.

v0.3.0 (28/6/2018)
- Add the following readers support:
  ACR3901U-S1/ACR3901T-W1
  ACR1255U-J1
  ACR1255U-J1 V2
- Skip invalid frames when transmitting the command to AMR220-C.
- Implement card insertion and removal in Bluetooth CardTerminals.list().
- Guard the concurrent access of current state in Bluetooth CardTerminal.
- Add BluetoothTerminalManager.setMasterKey().
- Add BluetoothTerminalManager.terminalType().
- Update the documentation.
- Update the version to 0.3.0.

v0.2.0 (4/4/2018)
- Add TransmitOptions to modify the default behaviour in transmitting APDUs.
- Replace warm reset with cold reset in Bluetooth Card.connect() and
  Card.disconnect().
- Fix overflow problem in parsing the ATR string.
- Update to recommended settings in Xcode 9.3.
- Update the documentation.
- Update the version to 0.2.0.

v0.1.2 (14/2/2018)
- Set service discovery timeout, characteristics discovery timeout and
  notification timeout from parameter in AMR220-C initialization.
- Check the error code when transmitting APDU to PICC.
- Override the description in Bluetooth CardTerminal.
- Implement CustomStringConvertible in Bluetooth Card and CardChannel.
- Replace string interpolation with concatenation.
- Update the version to 0.1.2.

v0.1.1 (29/1/2018)
- Wait for peripheral disconnection when disconnecting the device.
- Synchronize the sequence number when transmitting the command to AMR220-C.
- Check peripheral state when disconnecting AMR220-C.
- Change service discovery timeout to 5 seconds and characteristics discovery
  timeout to 5 seconds in AMR220-C initialization.
- Return same Card object for direct in Bluetooth CardTerminal.connect().
- Update the version to 0.1.1.

v0.1.0 (22/12/2017)
- New release.



Demo

v0.7.0 (24/7/2026)
- Show the disconnection message when the terminal is disconnected.
- Update the version to 0.7.0.

v0.6.2 (4/12/2025)
- Update to recommended settings in Xcode 26.1.1.
- Wait 5 seconds for card insertion in MainViewController.
- Fix unsaved commands in MainViewController.unwindMain().
- Update the version to 0.6.2.

v0.6.1 (22/11/2024)
- Add SourceViewController to select the source such as Input, iTunes File
  Sharing and Document Picker.
- Add InputViewController to enter commands.
- Register keyPrefCommands in MainViewController.viewDidLoad().
- Load the commands in MainViewController.loadSettings().
- Assign commands to InputViewController in MainViewController.prepare().
- Save the commands in MainViewController.unwindToMain().
- Add MainViewController.sendCommands().
- Set reuseIdentifier to ScriptFile in BLE Test scene.
- Do not list files for iTunes in MainViewController.shouldPerformSegue().
- Show the document picker in MainViewController.tableView().
- Implement UIDocumentPickerDelegate in MainViewController.
- Clear the URL and the filename in SourceViewControllerDelegate.
- Add MainViewController.runScript(card:hScriptFile:send:).
- Add MainViewController.runScript(card:url:send:).
- Transmit APDUs from source in MainViewController.tableView().
- Transmit control commands from source in MainViewController.tableView().
- Add missing constraints to Log text view in Main storyboard.
- Enable capitalization for New Key text field in Main storyboard.
- Allow HEX, space and newline for New Key in MasterKeyViewController.
- Update to recommended settings in Xcode 16.
- Change iOS deployment target to 15.0.
- Update the version to 0.6.1.

v0.6.0 (15/3/2024)
- Add ACR1555U to TerminalListViewController.selectTerminalType().
- Update to recommended settings in Xcode 15.3.
- Change iOS deployment target to 12.0.
- Update the version to 0.6.0.
- Set the build to 1.

v0.5.3 (18/11/2021)
- Add NSBluetoothAlwaysUsageDescription to Info.plist.
- Update to recommended settings in Xcode 13.1.
- Change iOS deployment target to 9.0.
- Update the version to 0.5.3.

v0.5.2 (9/3/2020)
- Fix incorrect default or custom settings shown in MainViewController.
- Add cell "Show Card State" to table view in MainViewController.
- Update the version to 0.5.2.

v0.5.1 (6/12/2019)
- Update the version to 0.5.1.

v0.5.0 (15/10/2019)
- Add TerminalTimeoutsViewController to show the timeouts settings.
- Add cell "Get Device Information" to table view in MainViewController.
- Use StackView in MasterKeyViewController.
- Use StackView in MainViewController.
- Update the version to 0.5.0.

v0.4.2 (28/3/2019)
- Convert to Swift 5.
- Update the version to 0.4.2.

v0.4.1 (9/10/2018)
- Convert to Swift 4.2.
- Update the version to 0.4.1.

v0.4.0 (30/8/2018)
- Add cell "Get Battery Status" to table view in MainViewController.
- Add cell "Get Battery Level" to table view in MainViewController.
- Import SmartCardIO.
- Update the version to 0.4.0.

v0.3.1 (4/7/2018)
- Indent FileListViewControllerDelegate.
- Indent ProtocolViewControllerDelegate.
- Indent TerminalListViewControllerDelegate.
- Update the version to 0.3.1.

v0.3.0 (28/6/2018)
- Fix a bug that the list does not display the terminal after scan.
- Add terminal type selection in TerminalListViewController.
- Add MasterKeyViewController to show the master key settings.
- Fix a bug that default values does not match with Settings.
- Update the version to 0.3.0.

v0.2.0 (4/4/2018)
- Add Settings.bundle to control Transmit Options.
- Update to recommended settings in Xcode 9.3.
- Update the version to 0.2.0.

v0.1.2 (14/2/2018)
- Replace /* ... */ with // in MainViewController.runScript().
- Replace string interpolation with concatenation in MainViewController.
- Update the version to 0.1.2.

v0.1.1 (29/1/2018)
- Fix file not closed on error in MainViewController.runScript().
- Update the version to 0.1.1.

v0.1.0 (22/12/2017)
- New release.



File Contents
-------------

API Documentation:  doc
Sample Application: BLETest
Framework:          BLETest\SmartCardIO.xcframework
                    BLETest\ACSSmartCardIO.xcframework
Debug Symbol:       dSYM-iOS\SmartCardIO.framework.dSYM
                    dSYM-iOS\ACSSmartCardIO.framework.dSYM
                    dSYM-iOS-Simulator\SmartCardIO.framework.dSYM
                    dSYM-iOS-Simulator\ACSSmartCardIO.framework.dSYM
Scripts:            scripts



Support
-------

In case of problem, please contact ACS through:

Web Site: http://www.acs.com.hk/
E-mail: info@acs.com.hk
Tel: +852 2796 7873
Fax: +852 2796 1286



-------------------------------------------------------------------------------
Copyright (c) 2017-2026, Advanced Card Systems Ltd. All rights reserved.

OpenJDK
Copyright (c) 1996-2014, Oracle and/or its affiliates. All rights reserved.

This code is free software; you can redistribute it and/or modify it
under the terms of the GNU General Public License version 2 only, as
published by the Free Software Foundation.  Oracle designates this
particular file as subject to the "Classpath" exception as provided
by Oracle in the LICENSE file that accompanied this code.

This code is distributed in the hope that it will be useful, but WITHOUT
ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or
FITNESS FOR A PARTICULAR PURPOSE.  See the GNU General Public License
version 2 for more details (a copy is included in the LICENSE file that
accompanied this code).

You should have received a copy of the GNU General Public License version
2 along with this work; if not, write to the Free Software Foundation,
Inc., 51 Franklin St, Fifth Floor, Boston, MA 02110-1301 USA.

IOS is a trademark or registered trademark of Cisco in the U.S. and other
countries and is used under license.

Xcode is a trademark of Apple Inc.

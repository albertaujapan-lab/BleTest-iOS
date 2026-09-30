# Summary of Source Code & Configuration Changes for iOS Compilation

This document provides a comprehensive summary of all source code, project configuration, and CI/CD workflow changes made to compile the **BLETest** iOS application successfully without requiring a local Mac or a paid Apple Developer account.

---

## 1. Overview of Challenges & Solutions

The original vendor project (ACS BLE Smart Card Reader SDK v0.7.0) was configured for building exclusively on a developer's local Mac equipped with an active Apple Developer Team and local Xcode user state (`xcuserdata`). 

When migrating this project to build in a headless, cloud CI environment (**GitHub Actions `macos-14`** runner) for Windows 11 sideloading, four main roadblocks had to be resolved:

| # | Roadblock | Cause | Resolution |
|---|-----------|-------|------------|
| **1** | **Missing Scheme in CI** | Xcode schemes were stored in private `xcuserdata/` (ignored by Git). `xcodebuild -scheme BLETest` failed to find the scheme. | Created a shared Xcode scheme at `BLETest.xcodeproj/xcshareddata/xcschemes/BLETest.xcscheme`. |
| **2** | **Code Signing Enforcement** | `CODE_SIGN_STYLE` was `Automatic` with a hardcoded vendor team (`4C5LG9ZP6A`). Cloud runners lack Apple certificates and provisioning profiles. | Set `CODE_SIGN_STYLE = Manual`, cleared `DEVELOPMENT_TEAM`, and set `CODE_SIGNING_ALLOWED=NO` during CI build. |
| **3** | **Embedded Frameworks Codesign Error (Exit 65)** | `SmartCardIO.xcframework` and `ACSSmartCardIO.xcframework` had `CodeSignOnCopy` enabled. Xcode attempted to run `codesign` during copying and failed. | Removed `CodeSignOnCopy` from the `Embed Frameworks` phase in `project.pbxproj`. |
| **4** | **Unsigned IPA Packaging Pipeline** | Need a standard `.ipa` file that sideloading tools (Sideloadly / AltStore) can resign with a free 7-day personal Apple ID on Windows 11. | Configured `xcodebuild clean archive` and scripted standard `Payload/` folder packaging and artifact upload. |

---

## 2. Detailed File Modifications

### 2.1. Shared Xcode Scheme Creation
* **File Added**: [`BLETest/BLETest.xcodeproj/xcshareddata/xcschemes/BLETest.xcscheme`](file:///d:/Project_Codes/iOS/acssmcio-0.7.0-ios15-20260724/BLETest/BLETest.xcodeproj/xcshareddata/xcschemes/BLETest.xcscheme)
* **Purpose**: Makes the `BLETest` build and archive target visible to command-line tools (`xcodebuild`) on any machine or CI runner.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Scheme
   LastUpgradeVersion = "1600"
   version = "1.7">
   <BuildAction
      parallelizeBuildables = "YES"
      buildImplicitDependencies = "YES">
      <BuildActionEntries>
         <BuildActionEntry
            buildForTesting = "YES"
            buildForRunning = "YES"
            buildForProfiling = "YES"
            buildForArchiving = "YES"
            buildForAnalyzing = "YES">
            <BuildableReference
               BuildableIdentifier = "primary"
               BlueprintIdentifier = "56CB898D1FDBC1F300510A0D"
               BuildableName = "BLETest.app"
               BlueprintName = "BLETest"
               ReferencedContainer = "container:BLETest.xcodeproj">
            </BuildableReference>
         </BuildActionEntry>
      </BuildActionEntries>
   </BuildAction>
   ...
   <ArchiveAction
      buildConfiguration = "Release"
      revealArchiveInOrganizer = "YES">
   </ArchiveAction>
</Scheme>
```

> **Why this matters**: In Xcode, targets describe *what* to build, while schemes describe *how* to build, test, and archive targets. Without a shared scheme committed to Git, `xcodebuild -scheme BLETest` outputs `xcodebuild: error: The project 'BLETest' does not contain a scheme named 'BLETest'`.

---

### 2.2. Xcode Project File (`project.pbxproj`)
* **File Modified**: [`BLETest/BLETest.xcodeproj/project.pbxproj`](file:///d:/Project_Codes/iOS/acssmcio-0.7.0-ios15-20260724/BLETest/BLETest.xcodeproj/project.pbxproj)

#### A. Removed `CodeSignOnCopy` from Embedded XCFrameworks
Both `SmartCardIO.xcframework` and `ACSSmartCardIO.xcframework` were configured to be code-signed automatically during the copy phase. In a CI environment without signing certificates, this produced `Process completed with exit code 65`.

```diff
- 567412FD301348310016D8C3 /* SmartCardIO.xcframework in Embed Frameworks */ = {isa = PBXBuildFile; fileRef = 567412FB301348310016D8C3 /* SmartCardIO.xcframework */; settings = {ATTRIBUTES = (CodeSignOnCopy, RemoveHeadersOnCopy, ); }; };
+ 567412FD301348310016D8C3 /* SmartCardIO.xcframework in Embed Frameworks */ = {isa = PBXBuildFile; fileRef = 567412FB301348310016D8C3 /* SmartCardIO.xcframework */; settings = {ATTRIBUTES = (RemoveHeadersOnCopy, ); }; };

- 567413013013483F0016D8C3 /* ACSSmartCardIO.xcframework in Embed Frameworks */ = {isa = PBXBuildFile; fileRef = 567412FF3013483F0016D8C3 /* ACSSmartCardIO.xcframework */; settings = {ATTRIBUTES = (CodeSignOnCopy, RemoveHeadersOnCopy, ); }; };
+ 567413013013483F0016D8C3 /* ACSSmartCardIO.xcframework in Embed Frameworks */ = {isa = PBXBuildFile; fileRef = 567412FF3013483F0016D8C3 /* ACSSmartCardIO.xcframework */; settings = {ATTRIBUTES = (RemoveHeadersOnCopy, ); }; };
```

#### B. Removed Vendor Development Team & Switched to Manual Signing
Removed the vendor's Team ID (`4C5LG9ZP6A`) and changed `CODE_SIGN_STYLE` from `Automatic` to `Manual` for both `Debug` and `Release` build configurations:

```diff
- CODE_SIGN_IDENTITY = "iPhone Developer";
+ CODE_SIGN_IDENTITY = "-";
- DEVELOPMENT_TEAM = 4C5LG9ZP6A;
+ DEVELOPMENT_TEAM = "";

  /* Under Target "BLETest" Debug configuration */
- CODE_SIGN_STYLE = Automatic;
+ CODE_SIGN_STYLE = Manual;

  /* Under Target "BLETest" Release configuration */
- CODE_SIGN_STYLE = Automatic;
+ CODE_SIGN_STYLE = Manual;
```

---

### 2.3. GitHub Actions Build Workflow (`build-ios.yml`)
* **File Modified**: [`.github/workflows/build-ios.yml`](file:///d:/Project_Codes/iOS/acssmcio-0.7.0-ios15-20260724/.github/workflows/build-ios.yml)

The cloud CI script was refined into an end-to-end headless build and IPA packaging pipeline:

```yaml
name: Build BLETest iOS App

on:
  push:
    branches: [ "main" ]
  workflow_dispatch:

env:
  FORCE_JAVASCRIPT_ACTIONS_TO_NODE24: true

jobs:
  build:
    name: Build IPA
    runs-on: macos-14

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Check Xcode Version
        run: |
          xcodebuild -version
          echo "Available Xcode versions:"
          ls -d /Applications/Xcode*

      - name: Build App Archive
        run: |
          xcodebuild clean archive \
            -project BLETest/BLETest.xcodeproj \
            -scheme BLETest \
            -configuration Release \
            -destination "generic/platform=iOS" \
            -archivePath build/BLETest.xcarchive \
            CODE_SIGN_STYLE=Manual \
            DEVELOPMENT_TEAM="" \
            PROVISIONING_PROFILE_SPECIFIER="" \
            CODE_SIGN_IDENTITY="" \
            CODE_SIGNING_REQUIRED=NO \
            CODE_SIGNING_ALLOWED=NO

      - name: Package into IPA
        run: |
          mkdir -p build/Payload
          cp -R build/BLETest.xcarchive/Products/Applications/BLETest.app build/Payload/
          cd build
          zip -r BLETest.ipa Payload/
          cd ..

      - name: Upload IPA Artifact
        uses: actions/upload-artifact@v4
        with:
          name: BLETest-iOS-App
          path: build/BLETest.ipa
          retention-days: 7
```

#### Key Workflow Techniques Explained:
1. **`xcodebuild clean archive` instead of `clean build`**:
   - `clean build` with custom directories often skips standard bundle packaging or causes linker issues.
   - `clean archive` builds the complete canonical Xcode archive (`.xcarchive`) containing compiled binaries, assets, and linked frameworks under `Products/Applications/BLETest.app`.
2. **`destination "generic/platform=iOS"`**:
   - Compiles genuine ARM64 machine code for physical iPhones and iPads, rather than x86_64 or arm64-simulator binaries.
3. **`CODE_SIGNING_ALLOWED=NO` and `CODE_SIGNING_REQUIRED=NO`**:
   - Instructs Xcode's build pipeline to skip all signature checks, certificate lookups, and provisioning profile validations.
4. **Standard iOS IPA Structure**:
   - iOS `.ipa` files are standard zip archives with a top-level directory named `Payload/` containing `<AppName>.app`.
   - By creating `build/Payload`, copying `BLETest.app` into it, and running `zip -r BLETest.ipa Payload/`, the workflow creates a 100% compliant `.ipa` file.
5. **`FORCE_JAVASCRIPT_ACTIONS_TO_NODE24: true`**:
   - Proactively forces GitHub Actions runner steps to execute on Node.js 24 runtime, ensuring forward-compatibility with future runner updates.

---

### 2.4. Documentation Synchronization (`Installation(Win11).md`)
* **File Modified**: [`Installation(Win11).md`](file:///d:/Project_Codes/iOS/acssmcio-0.7.0-ios15-20260724/Installation(Win11).md)
* **Purpose**: Synchronized the documentation so other team members or users on Windows 11 can follow the exact build and sideloading steps without stumbling into code-signing errors.

---

## 3. Git Commit History Summary

Here is the chronological commit trail showing how each issue was identified and resolved:

```
* b3a0e03 Configure xcodebuild archive without code signing and remove CodeSignOnCopy
|   - Fixed exit code 65 by disabling CODE_SIGNING_ALLOWED and removing CodeSignOnCopy on embedded frameworks.
|   - Packaged IPA from canonical .xcarchive bundle.
|
* 87a684e Add shared Xcode scheme and build via scheme in GitHub Actions
|   - Added BLETest.xcscheme to xcshareddata so xcodebuild can find the scheme in CI.
|
* 26f2ce4 Switch to Node 24 and configure manual ad-hoc signing for cloud builds
|   - Set FORCE_JAVASCRIPT_ACTIONS_TO_NODE24 and adjusted manual signing parameters.
|
* 40c4929 Fix Xcode path and configure ad-hoc code signing
|   - Replaced fixed Xcode 15.4 path with default runner Xcode version.
|
* 4f99970 Initial commit of ACS BLE smart card reader SDK and BLETest
    - Original vendor source files and frameworks.
```

---

## 4. How the Sideloading Hand-off Works

```
+------------------------------------------------------------------------+
| 1. GitHub Actions Cloud (macOS-14 Runner)                              |
|    - Swift compiler builds BLETest for ARM64 (physical iPhone)         |
|    - Code signing disabled (CODE_SIGNING_ALLOWED=NO)                   |
|    - Output: Unsigned BLETest.ipa                                      |
+------------------------------------------------------------------------+
                                   |
                          (Download Artifact)
                                   v
+------------------------------------------------------------------------+
| 2. Windows 11 PC (Sideloadly / AltStore)                               |
|    - User connects iPhone via USB                                      |
|    - User inputs free personal Apple ID                                |
|    - Sideloadly generates 7-day development certificate & profile      |
|    - Sideloadly signs:                                                 |
|        - SmartCardIO.framework                                         |
|        - ACSSmartCardIO.framework                                      |
|        - BLETest executable                                            |
|    - Sideloadly installs app onto connected iPhone                     |
+------------------------------------------------------------------------+
                                   |
                                   v
+------------------------------------------------------------------------+
| 3. iPhone (iOS 15 - iOS 18+)                                           |
|    - Go to Settings > General > VPN & Device Management                |
|    - Trust Apple ID Developer Profile                                  |
|    - (iOS 16+) Turn on Developer Mode & Restart                        |
|    - Launch BLETest & Scan for ACS BLE Smart Card Readers              |
+------------------------------------------------------------------------+
```

# Guide: Installing BLETest iOS App on iPhone from Windows 11

> **Author**: Antigravity (iOS Specialist & Code Tutor)  
> **Workspace**: `acssmcio-0.7.0-ios15-20260724`  
> **Target Devices**: iPhone with iOS 15.0 or later  
> **Host OS**: Windows 11 64-bit  
> **Toolchain**: GitHub Actions (Cloud macOS) + Sideloadly

---

## 1. Overview & Strategy

Building native iOS applications requires Apple's **Xcode** and macOS toolchains. Since Windows 11 cannot directly compile Swift apps targeting iOS devices, we use a cloud-and-sideload approach:

```
[Windows 11 PC] 
      │  git push
      ▼
[GitHub Repository]
      │  triggers cloud macOS Runner
      ▼
[GitHub Actions (macOS 14/15)] ──> Compiles & builds BLETest.ipa
      │  download artifact
      ▼
[Windows 11 PC]
      │  Sideloadly (USB / Wi-Fi)
      ▼
[Physical iPhone] ───────────────> App installed & ready to test!
```

---

## 2. Step-by-Step Installation Instructions

### Step 1: Push Your Code to GitHub

1. Open **PowerShell** or **Command Prompt** on Windows 11 in your project folder:
   ```powershell
   cd d:\Project_Codes\iOS\acssmcio-0.7.0-ios15-20260724
   ```
2. Initialize git and make an initial commit:
   ```powershell
   git init
   git add .
   git commit -m "Initial commit of ACS BLE smart card reader SDK and BLETest"
   ```
3. Create a **Private** (or Public) repository on [GitHub](https://github.com/new).
4. Link your remote repository and push your branch:
   ```powershell
   git remote add origin https://github.com/<your-github-username>/<your-repo-name>.git
   git branch -M main
   git push -u origin main
   ```

---

### Step 2: Set Up the GitHub Actions Build Workflow

1. In your project, create the directory structure:  
   `.github\workflows\`
2. Create a file named `build-ios.yml` inside `.github\workflows\` with the following content:

```yaml
name: Build BLETest iOS App

on:
  push:
    branches: [ "main" ]
  workflow_dispatch: # Allows manual trigger from GitHub web UI

jobs:
  build:
    name: Build IPA
    runs-on: macos-14 # Apple Silicon (M1/M2) runner with Xcode pre-installed

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Check Xcode Version
        run: |
          xcodebuild -version
          echo "Available Xcode installations:"
          ls -d /Applications/Xcode*

      - name: Build App Archive
        run: |
          xcodebuild clean build \
            -project BLETest/BLETest.xcodeproj \
            -target BLETest \
            -sdk iphoneos \
            -configuration Release \
            CODE_SIGN_IDENTITY="-" \
            CODE_SIGNING_REQUIRED=NO \
            CODE_SIGNING_ALLOWED=YES \
            CONFIGURATION_BUILD_DIR=build/Release-iphoneos

      - name: Package into IPA
        run: |
          mkdir -p build/Payload
          cp -R build/Release-iphoneos/BLETest.app build/Payload/
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

3. Commit and push this workflow to GitHub:
   ```powershell
   git add .github/workflows/build-ios.yml
   git commit -m "Add GitHub Actions iOS build workflow"
   git push
   ```

---

### Step 3: Download the Compiled `.ipa` from GitHub

1. Open your repository on [GitHub](https://github.com).
2. Click the **Actions** tab at the top.
3. Select **Build BLETest iOS App** on the left menu.
4. Click the latest workflow run (or click **Run workflow** to trigger it manually).
5. The build will take approximately **2 to 3 minutes** to complete.
6. Once completed (green checkmark), scroll down to the **Artifacts** section at the bottom of the page.
7. Click **`BLETest-iOS-App`** to download the zip file.
8. Extract the downloaded zip file on your Windows 11 PC to obtain **`BLETest.ipa`**.

---

### Step 4: Prepare Windows 11 (Install Apple USB Drivers)

For Windows 11 to communicate with your iPhone over USB, Apple mobile device drivers are required.

> [!IMPORTANT]
> **Do NOT install iTunes or iCloud from the Microsoft Store!**  
> Microsoft Store versions run in a sandbox that prevents sideloading tools from accessing the low-level USB drivers. Use the direct Apple standalone installers:

1. Download and install **[iTunes for Windows (64-bit Direct Installer)](https://www.apple.com/itunes/download/win64)**.
2. Download and install **[iCloud for Windows (Direct Installer)](https://updates.cdn-apple.com/2020/windows/001-39935-20200911-1A70AA56-F448-11EA-8109-AE4E41DEE129/iCloudSetup.exe)**.
3. Restart your Windows 11 computer if prompted by the installer.

---

### Step 5: Prepare Your iPhone (Enable Developer Mode)

On iOS 16, 17, and 18+, Apple requires Developer Mode to run sideloaded apps:

1. On your iPhone, open **Settings**.
2. Tap **Privacy & Security**.
3. Scroll to the bottom and tap **Developer Mode**.
4. Toggle the switch to **ON**.
5. Tap **Restart** when prompted.
6. After your iPhone reboots, unlock the screen and tap **Turn On**, then enter your device passcode.

---

### Step 6: Install the App onto iPhone using Sideloadly

1. Download and install **[Sideloadly](https://sideloadly.io/)** on Windows 11.
2. Connect your iPhone to your Windows PC using a USB cable.
   - Unlock your iPhone screen and tap **Trust This Computer** when prompted.
3. Launch **Sideloadly**:
   - Ensure your iPhone appears in the **Device** dropdown.
   - In the **Apple ID** field, enter your personal Apple ID email address.
   - Drag and drop your **`BLETest.ipa`** file onto the large IPA icon on the left.
4. *(Recommended)* Click **Advanced Options**:
   - Change the **Bundle ID** to something personal (e.g., `com.yourname.BLETest`) so it does not collide with the default vendor ID.
5. Click **Start**:
   - Enter your Apple ID password and your 2-Factor Authentication (2FA) verification code when prompted.
   - Sideloadly will contact Apple to generate a free 7-day personal development certificate, sign the IPA, and install it on your device.
6. When the progress log displays **Done**, the **BLETest** app icon will appear on your iPhone's home screen.

---

### Step 7: Trust the Developer Profile on iPhone (First Launch Only)

When you first tap the **BLETest** app icon, iOS will display an *"Untrusted Developer"* prompt. To authorize it:

1. On your iPhone, go to **Settings** -> **General**.
2. Tap **VPN & Device Management** (or *Device Management*).
3. Under **Developer App**, tap your Apple ID email address.
4. Tap **Trust "[Your Apple ID]"** and confirm.
5. Launch the **BLETest** app from your home screen.
6. When prompted, tap **OK** to allow **Bluetooth permissions**.

---

## 3. Important Notes & Troubleshooting

| Topic | Explanation & Solution |
| :--- | :--- |
| **7-Day Expiration** | Apps signed with a free personal Apple ID expire after **7 days**. When expired, open Sideloadly and click **Start** to resign and reinstall. If Sideloadly remains running in the Windows system tray and your PC and iPhone are on the same Wi-Fi, it can automatically refresh in the background. |
| **Why Physical Device?** | Bluetooth Low Energy scanning cannot run on a software Simulator. Physical iPhone hardware is required to communicate with ACS Bluetooth readers. |
| **Paid Developer Alternative** | If you have a paid Apple Developer Account ($99/year), you can configure GitHub Actions with your distribution certificate and push builds directly to **Apple TestFlight**, eliminating the 7-day limit and USB cables completely. |
| **Device Not Detected in Sideloadly** | 1. Ensure iTunes opens and recognizes your iPhone.<br>2. Unlock your iPhone screen and verify you tapped "Trust This Computer".<br>3. Verify you installed the direct Apple version of iTunes/iCloud, not the Microsoft Store version. |

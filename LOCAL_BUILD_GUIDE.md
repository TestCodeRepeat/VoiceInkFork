# VoiceInk Local Build Guide

This document outlines the steps to build VoiceInk with licensing disabled for local development/testing.

## Prerequisites

- macOS with Xcode installed
- Git
- Swift toolchain

Check prerequisites with:
```bash
make check
```

## Step 1: Clone the Repository

```bash
git clone <repository-url>
cd VoiceInkFork
```

## Step 2: Build Whisper Framework

The app requires the whisper.cpp xcframework. Build it:

```bash
make whisper
```

This will:
1. Create `~/VoiceInk-Dependencies/` directory
2. Clone whisper.cpp repository
3. Build `whisper.xcframework`

**Note:** The Xcode project may look for the framework in a different location. If build fails with "XCFramework not found", create a symlink:

```bash
mkdir -p /Users/$(whoami)/Developer/VoiceInk/VoiceInk-Dependencies/whisper.cpp/build-apple
ln -sf ~/VoiceInk-Dependencies/whisper.cpp/build-apple/whisper.xcframework \
       /Users/$(whoami)/Developer/VoiceInk/VoiceInk-Dependencies/whisper.cpp/build-apple/whisper.xcframework
```

## Step 3: Disable Licensing

Edit `VoiceInk/Models/LicenseViewModel.swift`. Replace the `loadLicenseState()` function with:

```swift
private func loadLicenseState() {
    // TEMPORARY: Disable licensing for local development
    // TODO: Remove this bypass before production release
    licenseState = .licensed

    /* Original licensing code - commented out for local development:

    // ... original code here ...

    */
}
```

### Key Files in the Licensing System

| File | Purpose |
|------|---------|
| `VoiceInk/Models/LicenseViewModel.swift` | Main license state management |
| `VoiceInk/Services/LicenseManager.swift` | Keychain storage for license data |
| `VoiceInk/Services/PolarService.swift` | API calls to Polar.sh for validation |
| `VoiceInk/Whisper/WhisperState.swift:388-392` | Watermark injection for trial expired |

### License States

The app uses three license states:
- `trial(daysRemaining: Int)` - 7-day trial period
- `trialExpired` - Trial ended, features restricted
- `licensed` - Full access

## Step 4: Build the App

### Option A: Build with Personal Developer Signing (Recommended)

Build with your Apple Developer account for proper code signing:

```bash
# Build signed Release version
xcodebuild -project VoiceInk.xcodeproj \
           -scheme VoiceInk \
           -configuration Release \
           build

# Or use make target
make build-signed
```

### Option B: Build with Ad-hoc Signing (No Developer Account)

```bash
xcodebuild -project VoiceInk.xcodeproj \
           -scheme VoiceInk \
           -configuration Debug \
           CODE_SIGN_IDENTITY="-" \
           CODE_SIGNING_REQUIRED=NO \
           CODE_SIGNING_ALLOWED=NO \
           DEVELOPMENT_TEAM="" \
           build
```

The built app will be at:
```
~/Library/Developer/Xcode/DerivedData/VoiceInk-*/Build/Products/Release/VoiceInk.app   # Signed
~/Library/Developer/Xcode/DerivedData/VoiceInk-*/Build/Products/Debug/VoiceInk.app     # Ad-hoc
```

## Step 5: Create Installer (DMG)

Copy the app to an installer directory:

```bash
mkdir -p installer
cp -R ~/Library/Developer/Xcode/DerivedData/VoiceInk-*/Build/Products/Release/VoiceInk.app installer/
```

Create a DMG:

```bash
hdiutil create -volname "VoiceInk Local" \
               -srcfolder installer \
               -ov \
               -format UDZO \
               VoiceInk-Local.dmg
```

Or use the make target:

```bash
make installer
```

## Step 6: Install the App

### Quick Install (Direct to Applications)

```bash
# Install directly without DMG
make install

# Or manually:
cp -R ~/Library/Developer/Xcode/DerivedData/VoiceInk-*/Build/Products/Release/VoiceInk.app /Applications/
```

### Install from DMG

1. Open `VoiceInk-Local.dmg`
2. Drag `VoiceInk.app` to `/Applications`
3. For ad-hoc signed builds only: Right-click the app and select "Open" (first time only, to bypass Gatekeeper)

## Reverting Licensing

To restore licensing functionality, edit `VoiceInk/Models/LicenseViewModel.swift` and remove the bypass code:

1. Remove the early `licenseState = .licensed` line
2. Uncomment the original licensing logic inside the `/* ... */` block

## Quick Build Commands

```bash
# Full build from scratch
make all

# Build and run
make dev

# Just build
make build

# Run existing build
make run

# Clean build artifacts
make clean
```

## Troubleshooting

### Build fails with "XCFramework not found"
Create the symlink as described in Step 2.

### Build fails with code signing errors
Ensure you're using the ad-hoc signing flags shown in Step 4.

### App crashes on launch
Check Console.app for crash logs. Common issues:
- Missing entitlements (expected with ad-hoc signing)
- Accessibility permissions not granted

## File Locations

| Item | Path |
|------|------|
| Project | `VoiceInk.xcodeproj` |
| Source Code | `VoiceInk/` |
| Dependencies | `~/VoiceInk-Dependencies/` |
| Build Output | `~/Library/Developer/Xcode/DerivedData/VoiceInk-*/` |
| DMG Installer | `VoiceInk-Local.dmg` |

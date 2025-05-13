# Brawl Stars Auto Dodge

An Android application that automatically detects and dodges projectiles in Brawl Stars using computer vision and accessibility services.

## Project Description

This Android application uses:
- Screen capture to monitor Brawl Stars gameplay in real-time
- OpenCV for computer vision to detect projectiles 
- Accessibility services to execute dodge movements automatically
- Overlay UI to control the dodge functionality while in-game

The app constantly scans the screen during gameplay, identifies incoming projectiles, calculates their trajectory, and automatically performs dodge movements to help you avoid taking damage.

## Getting the APK

### Method 1: Build It Yourself (Recommended)

To build the APK yourself:

1. Clone this repository to your local machine
2. Open the project in Android Studio
3. Connect your Android device or set up an emulator
4. Click "Run" to build and install directly to your device

Alternatively, to just build the APK:
1. In Android Studio, select "Build" -> "Build Bundle(s) / APK(s)" -> "Build APK(s)"
2. The APK will be saved in `app/build/outputs/apk/debug/`

### Method 2: Use the Pre-built APK

If available, you can download the pre-built APK from the releases section of this repository.

## Installation Guide

For detailed installation instructions, see the [TUTORIAL.md](TUTORIAL.md) file.

Visual instructions are available in [VISUAL_GUIDE.md](VISUAL_GUIDE.md).

## Frequently Asked Questions

For answers to common questions, see the [FAQ.md](FAQ.md) file.

## Technical Details

### Core Components

- **ProjectileDetector**: Uses OpenCV to identify and track projectiles
- **DodgeController**: Calculates optimal dodge directions based on projectile trajectories
- **AutoDodgeService**: Accessibility service that executes dodge movements
- **ScreenCaptureManager**: Captures and processes screen frames in real-time
- **OverlayService**: Manages the floating controls

### Permissions Required

- **Overlay Permission**: For floating control panel
- **Accessibility Service**: For performing dodge movements
- **Screen Capture**: For detecting projectiles

## Disclaimer

This app is not affiliated with, endorsed, sponsored, or approved by Supercell. This is an independent, unofficial application. Use at your own risk.

Brawl Stars and all related content, characters, and elements are trademarks and copyrights of Supercell.
# Building Brawl Stars Auto Dodge on PC

This guide will walk you through the process of building the Brawl Stars Auto Dodge app using a PC with Android Studio.

## Step 1: Install Android Studio

1. Download Android Studio from the official website: https://developer.android.com/studio
2. Install Android Studio following the installation wizard
3. During installation, make sure to include:
   - Android SDK
   - Android SDK Platform
   - Android Virtual Device
   - Performance (Intel HAXM)

## Step 2: Set Up the Project

1. Launch Android Studio
2. Select "Open an Existing Project"
3. Navigate to the folder where you extracted the project files
4. Select the root folder of the project (where the `settings.gradle` file is located)
5. Click "Open"
6. Android Studio will start loading the project - this may take a few minutes the first time

## Step 3: Install Required SDK Components

If prompted about missing SDK components:
1. Click "Install missing platforms and sync project"
2. Accept the license agreements
3. Wait for the installation to complete

## Step 4: Build the APK

1. From the top menu, select "Build" → "Build Bundle(s) / APK(s)" → "Build APK(s)"
2. Wait for the build process to complete
3. When finished, a notification will appear at the bottom right
4. Click "locate" in the notification to see where the APK was saved
   - Typically found in `app/build/outputs/apk/debug/app-debug.apk`

## Step 5: Transfer to Phone and Install

1. Connect your phone to your PC with a USB cable
2. Enable file transfer mode on your phone if prompted
3. Copy the APK file to your phone's storage
4. On your phone, use a file manager to locate and tap the APK
5. If prompted about installing from unknown sources, enable this setting
6. Follow the installation prompts

## Step 6: Using the App

1. Open the app on your phone
2. Grant all required permissions:
   - Overlay permission
   - Accessibility permission
3. Tap "START AUTO DODGE"
4. Open Brawl Stars
5. Use the floating control panel to enable/disable the auto dodge feature

## Troubleshooting

### If the build fails:
- Check Android Studio's "Event Log" for specific error messages
- Make sure you have the correct SDK versions installed
- Try selecting "File" → "Sync Project with Gradle Files"

### If the app crashes:
- Check that you've granted all necessary permissions
- Make sure you're using Android 7.0 (API level 24) or higher
- Some devices may have compatibility issues with screen capture functionality
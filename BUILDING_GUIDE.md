# Building the Brawl Stars Auto Dodge APK

This guide provides detailed instructions for building the Brawl Stars Auto Dodge APK using Android Studio on your computer.

## Prerequisites

Before you begin, you'll need:

1. A computer running Windows, macOS, or Linux
2. Internet connection (to download Android Studio and dependencies)
3. At least 8GB of RAM (16GB recommended for smoother development)
4. At least 10GB of free disk space
5. Basic familiarity with using a computer

## Step 1: Download and Install Android Studio

1. Visit the [Android Studio download page](https://developer.android.com/studio)
2. Download the appropriate version for your operating system
3. Install Android Studio by following the installation wizard
   - On Windows: Run the downloaded .exe file and follow the prompts
   - On macOS: Drag Android Studio to the Applications folder
   - On Linux: Extract the downloaded archive and run the studio.sh script

## Step 2: Set Up Android Studio

When you first launch Android Studio, it will guide you through the setup process:

1. If you're opening Android Studio for the first time, it will start the setup wizard
2. Choose "Standard" installation (this will automatically install the Android SDK)
3. Select a UI theme of your choice (Dark or Light)
4. Follow the prompts to download necessary SDK components
5. Wait for the components to download and install (this may take some time)

## Step 3: Download the Project Files

You can download all the project files from this repository:

1. Click the "Code" button at the top of this page
2. Select "Download ZIP"
3. Extract the ZIP file to a location on your computer (e.g., Documents folder)

Alternatively, if you're familiar with Git:

```
git clone <repository-url>
```

## Step 4: Import the Project into Android Studio

1. Open Android Studio
2. Select "Open an Existing Project"
3. Navigate to the folder where you extracted/cloned the project
4. Select the project folder and click "OK"
5. Wait for Android Studio to build the project (this may take a few minutes)
   - Android Studio will automatically download any missing Gradle components

## Step 5: Install OpenCV for Android

Since this project uses OpenCV for computer vision:

1. In Android Studio, go to File > Project Structure
2. Select "Dependencies" from the left menu
3. Click on the "+" button and select "Library Dependency"
4. Search for "opencv-android" and select the version 4.6.0
5. Click "OK" to add it to the project
6. Sync your project by clicking the "Sync Now" link in the notification

## Step 6: Build the APK

Now you're ready to build the APK:

1. In Android Studio, go to Build > Build Bundle(s) / APK(s) > Build APK(s)
2. Wait for the build process to complete
3. A notification will appear when the build is finished - click "locate" to open the folder containing the APK
4. The APK file will be located at: `app/build/outputs/apk/debug/app-debug.apk`

## Step 7: Transfer the APK to Your Android Device

There are several ways to transfer the APK to your Android device:

### Using USB
1. Connect your Android device to your computer with a USB cable
2. Enable file transfer mode on your device if prompted
3. Copy the APK file to your device's storage
4. Disconnect your device from the computer

### Using Email or Cloud Storage
1. Email the APK to yourself as an attachment
2. Open the email on your Android device and download the attachment
3. Alternatively, upload the APK to Google Drive, Dropbox, or similar service
4. Download it to your device using the respective app

## Step 8: Install the APK on Your Android Device

1. On your Android device, use a file manager to navigate to where you saved the APK
2. Tap on the APK file to start the installation
3. If prompted about security settings, you'll need to enable "Install unknown apps" or "Unknown sources"
4. Follow the on-screen instructions to complete the installation

## Troubleshooting

### Common Build Issues

- **Gradle Sync Failed**: Try File > Invalidate Caches / Restart
- **Missing SDK Components**: Go to Tools > SDK Manager and install any missing components
- **Java Version Issues**: Make sure you're using Java 11 or compatible version (File > Project Structure > SDK Location)
- **OpenCV Integration Issues**: Try manually downloading OpenCV Android SDK from opencv.org and importing it as a module

### Installation Issues

- **"App not installed" error**: Make sure you've uninstalled any previous version of the app
- **"Blocked by Play Protect"**: Tap "Install Anyway" if you trust the source
- **Unable to install**: Check that you have enough storage space and that you've allowed installation from unknown sources

## Next Steps

After successfully building and installing the APK, refer to the [TUTORIAL.md](TUTORIAL.md) file for instructions on how to use the app.
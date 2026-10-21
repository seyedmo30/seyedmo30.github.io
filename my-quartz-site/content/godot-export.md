# Pulley — Android Export, Signing & Debugging

A reproducible Ubuntu workflow for exporting the **Pulley** Godot 4.7.2 project, signing a debug APK, and installing it on a Xiaomi Redmi 15.

## 1. Known working environment

| Item | Value |
| --- | --- |
| Project | `/home/ssd/projects/pulley/godot` |
| Godot | `4.7.2.stable` |
| Godot export templates | `/home/ssd/.local/share/godot/export_templates/4.7.2.stable` |
| Java | OpenJDK 17 (`/usr/lib/jvm/java-17-openjdk-amd64`) |
| Android SDK | `/usr/lib/android-sdk` (Ubuntu packages) |
| SDK build tools | `/usr/lib/android-sdk/build-tools/debian` |
| SDK platform tools | `/usr/lib/android-sdk/platform-tools` |
| Debug keystore | `/home/ssd/.local/share/godot/keystores/debug.keystore` |
| Key alias | `androiddebugkey` |
| Device | Redmi 15, connected with ADB |

These are the values observed in the working setup. The Ubuntu SDK packages are older than current Google SDK releases; the exact Android export requirements depend on the selected Godot export configuration.

## 2. One-time dependency setup

```bash
sudo apt update
sudo apt install -y openjdk-17-jdk android-sdk adb
```

Verify installed tools:

```bash
java -version
adb version
command -v aapt
command -v apksigner
command -v zipalign
ls -la /usr/lib/android-sdk
```

Set environment variables for the current terminal session if needed:

```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export ANDROID_HOME=/usr/lib/android-sdk
export ANDROID_SDK_ROOT=/usr/lib/android-sdk
```

The Android SDK variables are **optional for the manual `apksigner`/`adb` steps**; Godot's own Android export settings must point to usable paths. Do not assume that the distro SDK contains modern `sdkmanager`, platform 35/36, or all build tools required for Gradle-based exports.

## 3. Godot templates and Android export configuration

Confirm the installed Godot export templates:

```bash
ls -la "$HOME/.local/share/godot/export_templates/4.7.2.stable"
```

In Godot, open **Editor → Editor Settings → Export → Android** and check:

- Java SDK Path: `/usr/lib/jvm/java-17-openjdk-amd64`
- Android SDK Path: `/usr/lib/android-sdk` **only if Godot accepts this SDK and its tools**.
- Debug keystore path: `/home/ssd/.local/share/godot/keystores/debug.keystore` (where the relevant editor/export setting is present).

Open **Project → Export** and verify an **Android** export preset exists. Check its actual name with:

```bash
cd /home/ssd/projects/pulley/godot
grep -E '^\[preset|^name=|^platform=' export_presets.cfg
```

Use the exact `name=` value in the CLI command. The preset must export project scenes and resources. If using the Godot GUI, select the Android preset, choose **Export Project**, enable **Export With Debug**, and save as `pulley-debug.apk`.

## 4. One-time debug keystore setup

The working keystore already exists. Verify it rather than regenerating it:

```bash
keytool -list -v \
  -keystore "$HOME/.local/share/godot/keystores/debug.keystore" \
  -alias androiddebugkey \
  -storepass android
```

If setting up a **new machine** and the keystore does not exist, create it once:

```bash
mkdir -p "$HOME/.local/share/godot/keystores"
keytool -genkeypair -v \
  -keystore "$HOME/.local/share/godot/keystores/debug.keystore" \
  -alias androiddebugkey \
  -keyalg RSA -keysize 2048 -validity 10000 \
  -storepass android -keypass android \
  -dname 'CN=Android Debug,O=Godot,C=US'
```

**Do not recreate or replace an existing keystore casually.** Updates signed with a different key may not install over the old app. This debug key is for development, not Play Store production releases.

## 5. Export the actual Pulley project

From Godot GUI: **Project → Export → Android → Export Project**, check **Export With Debug**, and output `pulley-debug.apk` into the project directory.

Alternatively, use the command line, provided your Godot executable is available and your export preset is configured:

```bash
cd /home/ssd/projects/pulley/godot
GODOT_BIN="$(command -v godot || command -v godot4)"
"$GODOT_BIN" --headless \
  --path /home/ssd/projects/pulley/godot \
  --export-debug 'Android' \
  /home/ssd/projects/pulley/godot/pulley-debug.apk
```

Replace `'Android'` if `export_presets.cfg` uses another preset name. If `GODOT_BIN` is empty, use the full path to your Godot executable or export through the GUI.

Check the file:

```bash
ls -lh /home/ssd/projects/pulley/godot/pulley-debug.apk
```

**Do not sign** `~/.local/share/godot/export_templates/4.7.2.stable/android_debug.apk` as if it were the game: that is an engine export template and lacks Pulley's exported project data. Installing the signed template previously caused `Unable to setup the Godot engine! Aborting...`.

## 6. Sign the exported APK

```bash
cd /home/ssd/projects/pulley/godot

apksigner sign \
  --ks "$HOME/.local/share/godot/keystores/debug.keystore" \
  --ks-key-alias androiddebugkey \
  --ks-pass pass:android \
  --key-pass pass:android \
  --out pulley-debug-signed.apk \
  pulley-debug.apk
```

Verify **the signed output**, not the unsigned input:

```bash
apksigner verify --verbose --print-certs pulley-debug-signed.apk
```

Expected: `Verifies` and at least one supported APK signature scheme marked `true` (the working setup reported v2 and v3).

Inspect app metadata:

```bash
aapt dump badging pulley-debug-signed.apk | grep -E "^(package:|application-label:|sdkVersion:|targetSdkVersion:)"
```

## 7. Connect the Redmi 15

On the phone, enable **Settings → About phone → tap HyperOS/OS version seven times**. Then enable **Settings → Additional settings → Developer options → USB debugging**. If required by HyperOS, enable **Install via USB** and approve installation prompts. No root or unlocked bootloader is required.

Connect by USB and run:

```bash
adb devices -l
```

Expected: a device line such as `198b3d4b device ... model:25062RN2DA`. If the status is `unauthorized`, approve the USB debugging prompt on the unlocked phone.

## 8. Install and launch

```bash
cd /home/ssd/projects/pulley/godot
adb install -r pulley-debug-signed.apk
```

Expected: `Success`.

Extract the actual package name and launch it:

```bash
PACKAGE="$(aapt dump badging pulley-debug-signed.apk \
  | sed -n "s/package: name='\([^']*\)'.*/\1/p")"
printf 'Package: %s\n' "$PACKAGE"
adb shell monkey -p "$PACKAGE" 1
```

If `INSTALL_FAILED_USER_RESTRICTED` appears, unlock the phone, enable HyperOS **Install via USB**, and approve the prompt. If `INSTALL_FAILED_UPDATE_INCOMPATIBLE` appears, the previously installed app may have been signed with another key; uninstalling it deletes its local app data.

## 9. Debugging

Clear previous logs:

```bash
adb logcat -c
```

Run the game and watch logs:

```bash
adb logcat -v time | grep --line-buffered -Ei 'godot|pulley|fatal exception|crash|AndroidRuntime'
```

Save logs to a file:

```bash
adb logcat -d -v time > /home/ssd/projects/pulley/godot/android-debug.log
```

ADB logs are useful for errors and crashes. For live scene inspection and Godot breakpoints, configure a compatible **debug export** and Godot's remote debugger; logcat alone does not provide those features.

## 10. Repeat after each game change

Run this after exporting a fresh `pulley-debug.apk` from Godot:

```bash
cd /home/ssd/projects/pulley/godot
set -e

apksigner sign \
  --ks "$HOME/.local/share/godot/keystores/debug.keystore" \
  --ks-key-alias androiddebugkey \
  --ks-pass pass:android \
  --key-pass pass:android \
  --out pulley-debug-signed.apk \
  pulley-debug.apk

apksigner verify --verbose pulley-debug-signed.apk
adb install -r pulley-debug-signed.apk

PACKAGE="$(aapt dump badging pulley-debug-signed.apk \
  | sed -n "s/package: name='\([^']*\)'.*/\1/p")"
test -n "$PACKAGE"
adb shell monkey -p "$PACKAGE" 1
```

**Caution:** `--ks-pass pass:android` and `--key-pass pass:android` expose the standard development password in command history and process arguments. Acceptable for a local debug key, but do not use this pattern with production signing secrets.

## 11. Troubleshooting summary

| Symptom | Likely cause / action |
| --- | --- |
| `FileNotFoundException: android_debug.apk` | Wrong working directory and wrong file; export `pulley-debug.apk` first. |
| `Unable to setup the Godot engine` after installing `android_debug_signed.apk` | Installed the generic export template, not the exported Pulley game. |
| `INSTALL_FAILED_USER_RESTRICTED` | Approve phone security prompt; enable HyperOS **Install via USB**. |
| `no devices/emulators found` | Check cable, USB debugging, and `adb devices -l`. |
| `INSTALL_FAILED_UPDATE_INCOMPATIBLE` | Signing key mismatch; preserve data before uninstalling. |
| Godot export complains about SDK | Inspect actual Godot Android export requirements; the Ubuntu distro SDK may be too old. |
| Signature check fails | Verify `pulley-debug-signed.apk`, not the unsigned source. |

# PitchBlack Recovery Project - Realme 11 (chongqing) - Android 15

```
██████╗ ██████╗ ██████╗ ██████╗
██╔══██╗██╔══██╗██╔══██╗██╔══██╗
██████╔╝██████╔╝██████╔╝██████╔╝
██╔═══╝ ██╔══██╗██╔══██╗██╔═══╝
██║     ██████╔╝██║  ██║██║
╚═╝     ╚═════╝ ╚═╝  ╚═╝╚═╝
```

This repository contains the **PitchBlack Recovery Project (PBRP)** device tree for the **Realme 11 (RMX3780)**.

---

## Device Information
- **Device**: Realme 11 5G
- **Tested Version**: RMX378X_15.0.0.1800(EX01)
- **Codename**: chongqing (formerly RE5C6CL1)
- **Model**: RMX3780 / RMX3781 / RMX3782 / RMX3783 / RMX3785
- **SoC**: MediaTek Dimensity 6100+ (MT6835)
- **Android**: 15 (SDK 35)
- **Kernel**: 5.15.180-android13-8
- **Build Date**: July 2026

---

## Key Features & Fixes in this Device Tree

### 1. Security HAL & Decryption
* Packs **`android.hardware.security.keymint@2.0`** and the necessary Trustonic/Mobicore binaries under `/odm/vendor/app/mcRegistry` to support FBE v2 decryption on Android 15 ROMs.
* Redundant duplicate firmware directories (like `/odm/firmware`) were removed to avoid AOSP symlink collisions.

### 2. Display Performance & Hardware Limitations
* Relies on the default `fbdev` software rasterizer for UI rendering. (Hardware OpenGL acceleration is deliberately disabled as the Mali GPU driver is not loaded in recovery, which would otherwise trigger an extremely slow `llvmpipe` software fallback.)
* The stock Realme recovery kernel omits the `cpufreq` driver, locking the CPU to a low bootloader frequency (~800MHz). UI animations may drop frames, but flashing and decryption speed are unaffected.

### 3. Touchscreen Refresh & Touch Lag Fix
* Configured the Oplus touchpanel report rate switch `/proc/touchpanel/game_switch_enable` to `1` on boot to enable high touch panel reporting rate.
* The stock vibrator AIDL HAL service fails on recovery kernels, causing touchscreen lag. This tree overrides it to a dummy instant-exit service to force recovery to use standard sysfs vibration pathways.

### 4. Timezone Sync
* Default timezone is configured to `IST-5:30` in `system.prop`, keeping both recovery and Android clocks perfectly synced.

### 5. CPU Temperature Scaling
* A background loop script `/system/bin/cpu_temp.sh` reads the battery PMIC temperature and scales it to `/tmp/cpu_temp` where it displays correctly on the recovery interface.

---

## Syncing PBRP Source

```bash
# 1. Create a fresh directory (separate from your TWRP/OrangeFox workspace!)
mkdir ~/pbrp && cd ~/pbrp

# 2. Initialize the PBRP 12.1 manifest
repo init --depth=1 -u https://github.com/PitchBlackRecoveryProject/manifest_pb -b android-12.1

# 3. Sync (~15GB download)
repo sync -c -j$(nproc) --force-sync --no-clone-bundle --no-tags
```

---

## Building

```bash
# 1. Copy this device tree into the PBRP workspace
cp -r /mnt/d/recoveries\ dt/pbrp-dt-working/* ~/pbrp/device/realme/chongqing/

# 2. Set up the build environment
cd ~/pbrp
export ALLOW_MISSING_DEPENDENCIES=true LC_ALL="C"
source build/envsetup.sh

# 3. Select device and build
lunch pb_chongqing-eng
mka pbrp -j$(nproc) 2>&1 | tee build.log
```

The output image will be at:
`out/target/product/chongqing/PitchBlack-*-chongqing-*.img`

---

## Flashing

### Flash via Fastboot
```bash
fastboot flash vendor_boot out/target/product/chongqing/PitchBlack-*.img
fastboot reboot recovery
```

### Flash ZIP via Recovery
```bash
adb push out/target/product/chongqing/PitchBlack-*.zip /sdcard/
```

---

## Credits
* **The PitchBlack Team** — for the PBRP recovery source code
* **The TeamWin (TWRP) Team** — for the base recovery framework
* **Ayan** — Device Tree Maintainer

# Compile SDK in Ubuntu 22.04 Environment

## 1. Setting up the Compilation Environment
- Install dependencies:
    ```bash
    sudo apt update

    sudo apt-get install -y git ssh make gcc gcc-multilib g++-multilib module-assistant expect g++ gawk texinfo libssl-dev bison flex fakeroot cmake unzip gperf autoconf device-tree-compiler libncurses5-dev pkg-config bc python-is-python3 passwd openssl openssh-server openssh-client vim file cpio rsync
    ```

- Get latest SDK:
    ```bash
    git clone https://github.com/LuckfoxTECH/luckfox-pico.git
    ```

- Compile SDK
	<p align="center">
	    <img src="../99. Images/Pasted image 20250903212326.png" width="1000">
	</p>

## 2. Configuration file

Luckfox Pico Ultra W SDK configuration file is `BoardConfig-EMMC-Buildroot-RV1106_Luckfox_Pico_Ultra_W-IPC.mk` located in the `project/cfg/BoardConfig_IPC` directory.

Main sections of `BoardConfig-EMMC-Buildroot-RV1106_Luckfox_Pico_Ultra_W-IPC.mk`:

### Board Config

```bash
export LF_ORIGIN_BOARD_CONFIG=BoardConfig-EMMC-Buildroot-RV1106_Luckfox_Pico_Ultra_W-IPC.mk
export RK_CHIP=rv1106
export RK_APP_TYPE=RKIPC_RV1106
export RK_BOOTARGS_CMA_SIZE="1M"
export RK_KERNEL_DTS=rv1106g-luckfox-pico-ultra-w.dts
```

* **LF_ORIGIN_BOARD_CONFIG**: Points to the base Makefile config for this board.
* **RK_CHIP**: The SoC name (Rockchip RV1106).
* **RK_APP_TYPE**: Application type (here: "RKIPC" → IPC camera).
* **RK_BOOTARGS_CMA_SIZE**: Configures the **CMA (Contiguous Memory Allocator)** for the Linux kernel (needed for ISP/video buffers).
* **RK_KERNEL_DTS**: The **Device Tree Source** describing the hardware of the board.

👉 This section defines **chip-level parameters** and the **device tree**.

---

### Boot Medium

```bash
export RK_BOOT_MEDIUM=emmc
export RK_UBOOT_DEFCONFIG_FRAGMENT="rk-emmc.config rv1106-luckfox-rgb-reset.config"
export RK_PARTITION_CMD_IN_ENV="32K(env),512K@32K(idblock),256K(uboot),32M(boot),512M(oem),256M(userdata),6G(rootfs)"
export RK_PARTITION_FS_TYPE_CFG=rootfs@IGNORE@ext4,userdata@/userdata@ext4,oem@/oem@ext4
```

* **RK_BOOT_MEDIUM=emmc**: Boot storage is eMMC.
* **UBOOT_DEFCONFIG_FRAGMENT**: Adds U-Boot config fragments.
* **RK_PARTITION_CMD_IN_ENV**: Defines the **partition layout** in eMMC
* **RK_PARTITION_FS_TYPE_CFG**: Defines filesystem types per partition

👉 This section defines the **storage partitioning and filesystems**.

---

### Target RootFS

```bash
export LF_TARGET_ROOTFS=buildroot
export RK_BUILDROOT_DEFCONFIG=luckfox_pico_w_defconfig
```

* **LF_TARGET_ROOTFS=buildroot**: The root filesystem will be built using **Buildroot**.
* **DEFCONFIG**: Which Buildroot configuration to use.

👉 This determines how the **Linux rootfs (BusyBox, libs, init scripts, apps)** is built.

---

### Defconfigs (Kernel, U-Boot, Toolchain)

```bash
export RK_ARCH=arm
export RK_TOOLCHAIN_CROSS=arm-rockchip830-linux-uclibcgnueabihf
export RK_MISC=wipe_all-misc.img
export RK_UBOOT_DEFCONFIG=luckfox_rv1106_uboot_defconfig
export RK_KERNEL_DEFCONFIG=luckfox_rv1106_linux_defconfig
export RK_KERNEL_DEFCONFIG_FRAGMENT=rv1106-bt.config
```

* **RK_ARCH=arm**: CPU architecture.
* **RK_TOOLCHAIN_CROSS**: Cross-compiler toolchain.
* **RK_MISC**: A misc image (used for wiping).
* **UBOOT_DEFCONFIG**: Base U-Boot configuration.
* **KERNEL_DEFCONFIG**: Base kernel configuration.
* **KERNEL_DEFCONFIG_FRAGMENT**: Extra kernel config (e.g., enable Bluetooth).

👉 This section defines the **core build configs** for U-Boot, Linux kernel, and cross-toolchain.

---

### Camera & ISP Config

```bash
export RK_CAMERA_SENSOR_IQFILES="sc4336_OT01_40IRC_F16.json ..."
export RK_CAMERA_SENSOR_CAC_BIN="CAC_sc4336_OT01_40IRC_F16"
```

* **IQFILES**: ISP (Image Signal Processor) image quality configs for specific camera sensors.
* **CAC_BIN**: Lens calibration files.

👉 These are essential for **IPC camera tuning**.

---

### Applications & Features

```bash
export RK_BUILD_APP_TO_OEM_PARTITION=y
export RK_ENABLE_ROCKCHIP_TEST=y
export RK_ENABLE_WIFI=y
export RK_ENABLE_WIFI_CHIP=AIC8800DC
export LF_WIFI_SSID="Wifi_Name"
export LF_WIFI_PSK="Wifi_Password"
```

* **RK_BUILD_APP_TO_OEM_PARTITION**: Install applications to `/oem`.
* **RK_ENABLE_ROCKCHIP_TEST**: Enable Rockchip test suite.
* **RK_ENABLE_WIFI**: Enable WiFi support (chip: AIC8800DC).
* **LF_WIFI_SSID / LF_WIFI_PSK**: Default WiFi SSID and password for auto-connect.

👉 This section enables **board features and apps**.

---

### Pre/Post Build Scripts

```bash
export RK_PRE_BUILD_OEM_SCRIPT=luckfox-buildroot-oem-pre.sh
export RK_PRE_BUILD_USERDATA_SCRIPT=luckfox-userdata-pre.sh
export RK_POST_OVERLAY="overlay-luckfox-config overlay-luckfox-buildroot-init overlay-luckfox-buildroot-shadow overlay-luckfox-buildroot-rgb"
```

* **Pre-build scripts**: Run before building (e.g., cleanup, copying configs).
* **Post overlays**: Overlay directories added into rootfs after build (configs, init, password shadow, RGB control).

👉 These are **customization hooks** for modifying the final image.

---

### Summarize

- This file is just a **set of environment variables**.
- It defines:

   * **Chip and device tree**
   * **Boot storage and partitioning**
   * **Root filesystem type (Buildroot)**
   * **Kernel/U-Boot toolchain configs**
   * **Camera ISP configs**
   * **WiFi and board features**
   * **Pre/Post build customizations**
   * **Pre/Post build customizations**

- When you run the build (`./build.sh`), these variables guide the build system to produce images:

   * **u-boot.img, kernel.img, rootfs.img, oem.img, userdata.img**
   * which will later be flashed onto the board.

## 3. Compiling the SDK
### 3.1. Compile all images
    ```bash
    ./build.sh lunch
    ./build.sh
    ```

### 3.2. Compile individual images
    ```bash
    ./build.sh clean uboot
    ./build.sh uboot            # Generated image file: output/image/uboot.img

    ./build.sh clean kernel
    ./build.sh kernel           # Generated image file: output/image/boot.img

    ./build.sh clean rootfs
    ./build.sh rootfs           # After compilation, use the ./build.sh firmware command to repack.

    ./build.sh clean media
    ./build.sh media           # Files are stored in the directory: output/out/media_out. After compilation, use the ./build.sh firmware command to repack.

    ./build.sh clean app
    ./build.sh app             # Note 1: app depends on media.
                               # Note 2: After compilation, use the ./build.sh firmware command to repack.

    ./build.sh firmware
    ```

### 3.3. Images storage directory
    ```
    output/
    ├── image
    │   ├── download.bin ---------------- Device-side program for upgrading communication of burning tools, only downloaded to the board's memory
    │   ├── env.img --------------------- Includes partition table and boot parameters
    │   ├── uboot.img ------------------- U-Boot image
    │   ├── idblock.img ----------------- Loader image
    │   ├── boot.img -------------------- Kernel image
    │   ├── rootfs.img ------------------ Kernel image
    │   └── userdata.img ---------------- Userdata image
    └── out
        ├── app_out --------------------- Files compiled after reference application compilation
        ├── media_out ------------------- Files compiled after media-related compilation
        ├── rootfs_xxx ------------------ File system packaging directory
        ├── S20linkmount ---------------- Partition mounting script
        ├── sysdrv_out ------------------ Files compiled after sysdrv compilation
        └── userdata -------------------- Userdata
    ```

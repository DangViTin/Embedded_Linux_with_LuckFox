# Flashing eMMC
## 1. Install driver

- Run [DriverInstall.exe](../../Tools/Driver/DriverAssitant_v5.12/DriverInstall.exe) to install the USB driver. No need to connect anything during this process. **Restart your computer** after the installation is complete.
<p align="center">
    <img src="images/Pasted image 20250820011401.png" width="500">
</p>

## 2. Set device to recovery mode
- Open the flashing tool [SocToolKit.exe](../../Tools/SocToolKit_v1.98/SocToolKit.exe) (right-click and run as administrator), Select RV1106.
<p align="center">
    <img src="images/Pasted image 20250820011207.png" width="700">
</p>

**Recovery mode** allows:
- The host PC to communicate directly with the SoC bootloader/ROM code.
- Bypasses the current OS on the eMMC/SD → to reload the firmware, bootloader or new OS.

In recovery mode of the Luckfox Pico Ultra W, the device will be recognized as a MaskRom device by the Rockchip flashing tool.

<p align="center">
    <img src="images/20250820-0208159.png">
</p>

- Hold down the BOOT button (the ENCODER button works equivalently), then connect the device to your computer. Release the button, and the Rockchip flashing tool will detect the device as a MaskRom device.

<p align="center">
    <img src="images/Pasted image 20250820022242.png" width="300">
</p>

## 3. Flashing

- eMMC firmware for Luckfox Pico Ultra W is located at: [eMMC_Images_0429](../../Tools/eMMC_Images_0429)
- Click `Search Path...` and select the folder containing the firmware files.
<p align="center">
    <img src="images/Pasted image 20250820023719.png" width="700">
</p>

- Then, Click `Yes` to reload the env file
<p align="center">
    <img src="images/Pasted image 20250820022858.png" width="700">
</p>

- Select all items and click `Download` button
<p align="center">
    <img src="images/Pasted image 20250820023433.png" width="700">
</p>

# Download SDK
- Install dependencies:
    ```bash
    sudo apt update

    sudo apt-get install -y git ssh make gcc gcc-multilib g++-multilib module-assistant expect g++ gawk texinfo libssl-dev bison flex fakeroot cmake unzip gperf autoconf device-tree-compiler libncurses5-dev pkg-config bc python-is-python3 passwd openssl openssh-server openssh-client vim file cpio rsync
    ```

- Get SDK:
    ```bash
    git clone https://github.com/LuckfoxTECH/luckfox-pico.git
    ```
# Inspect config file (u-boot config focused)
- This file is a make script provided by the Luckfox team to simplify the build process. `../luckfox-pico/project/cfg/BoardConfig_IPC/BoardConfig-EMMC-Buildroot-RV1106_Luckfox_Pico_Ultra_W-IPC.mk`
    
- This line specifies the defconfig file being used: `luckfox_rv1106_uboot_custom_defconfig`, located in the `../luckfox-pico/sysdrv/source/uboot/u-boot/configs/` folder.
    ```bash
    export RK_UBOOT_DEFCONFIG=luckfox_rv1106_uboot_custom_defconfig
    ```

- This line defines additional fragment files. Their configurations are appended to the final configuration along with the defconfig file.
    ```bash
    export RK_UBOOT_DEFCONFIG_FRAGMENT="rk-emmc.config rv1106-luckfox-rgb-reset.config"
    ```

- These lines specify the target architecture and cross-compiler toolchain. The toolchain is located at `../luckfox-pico/tools/linux/toolchain/arm-rockchip830-linux-uclibcgnueabihf`
    ```bash
    # Target arch
    export RK_ARCH=arm

    # Target Toolchain Cross Compile
    export RK_TOOLCHAIN_CROSS=arm-rockchip830-linux-uclibcgnueabihf
    ```
2. Make our custom defconfig file.
    
    - At `../luckfox-pico/sysdrv/source/uboot/u-boot/` path, run commands
        ```bash
        make ARCH=arm luckfox_rv1106_uboot_defconfig                    (config file in /configs folder)
        make ARCH=arm menuconfig                                        (show config UI, remember to save before exit)
        make ARCH=arm savedefconfig                                     (generate defconfig file)
        mv defconfig configs/luckfox_rv1106_uboot_custom_defconfig      (move new created file back to /configs folder)
        ```

    - For example, we change boot delay parameter to 3 seconds at config UI
        <p>
        <img src="./images/menuconfig.png"/>
        </p>

3. Build u-boot with new defconfig file
    
    - Edit `../luckfox-pico/project/cfg/BoardConfig_IPC/BoardConfig-EMMC-Buildroot-RV1106_Luckfox_Pico_Ultra_W-IPC.mk` file
        ```bash
        # Uboot defconfig
        # export RK_UBOOT_DEFCONFIG=luckfox_rv1106_uboot_defconfig
        export RK_UBOOT_DEFCONFIG=luckfox_rv1106_uboot_custom_defconfig
        ```
    - At `../luckfox-pico/` run
        ```bash
        ./build.sh clean uboot
        ./build.sh uboot
        ```

    - After build done, go to `..luckfox-pico/output/image/` and download 3 files `download.bin`, `idblock.img` and `uboot.img`. Copy `env.img` from vendor image files. Now put 4 files to seprated folder, from now on we will flash with files in this folder.
    
    - Using the flashing tool, write 4 files to eMMC. For this test, we will intentionally corrupt the kernel address by flashing U-Boot to the kernel partition. This prevents the kernel from loading and forces the system to stop at U-Boot. It only needs to be done once, since our focus is on U-Boot for now.

        <p>
        <img src="./images/flashing.png"/>
        </p>

4. Check the UART console
        <p>
        <img src="./images/terminal.png"/>
        </p>

    - You can see that U-Boot waits for 3 seconds before attempting to boot the kernel. Since the kernel cannot load, the boot process fails and control returns to the U-Boot console.

5. Toggle GPIO using u-boot console
    in u-boot console run
    ```bash
    gpio toggle 118
    ```
    This command will toggle onboard red LED (LED inside case, near USB port). Since this kind of hard to see, we run another command.
    ```bash
    gpio toggle 123
    ```
    This command will toggle LCD backlight.

    How to know the number of GPIO, number 118 and 123 coming from ? Inspsect the schematic file, we can see GPIO used to control backlight is GPIO3_D3_d. Now convert that GPIO number to the number that u-boot understand.
        <p>
        <img src="./images/GPIO_backlight.png"/>
        <img src="./images/GPIO_calculate.png"/>
        </p>

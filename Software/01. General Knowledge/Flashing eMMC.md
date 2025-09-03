# Flashing eMMC
## 1. Install driver

- Run [DriverInstall.exe](../../Tools/Driver/DriverAssitant_v5.12/DriverInstall.exe) to install the USB driver. No need to connect anything during this process. **Restart your computer** after the installation is complete.
<p align="center">
    <img src="../99. Images/Pasted image 20250820011401.png" width="500">
</p>

## 2. Set device to recovery mode
- Open the flashing tool [SocToolKit.exe](../../Tools/SocToolKit_v1.98/SocToolKit.exe) (right-click and run as administrator), Select RV1106.
<p align="center">
    <img src="../99. Images/Pasted image 20250820011207.png" width="700">
</p>

**Recovery mode** allows:
- The host PC to communicate directly with the SoC bootloader/ROM code.
- Bypasses the current OS on the eMMC/SD → to reload the firmware, bootloader or new OS.

In recovery mode of the Luckfox Pico Ultra W, the device will be recognized as a MaskRom device by the Rockchip flashing tool. There are two methods to enter recovery mode:

<p align="center">
    <img src="../99. Images/20250820-0208159.png">
</p>

**Method 1:** Hold down the **BOOT button** (the **ENCODER button** works equivalently), then connect the device to your computer. Release the button, and the Rockchip flashing tool will detect the device as a MaskRom device.

**Method 2:** Press the **RESET button**, then continuously press `Ctrl+C` on the serial console to interrupt the U-Boot autoboot process. 
<p align="center">
    <img src="../99. Images/Pasted image 20250821231909.png" width="500">
</p>

From here, you can run the **`download`** command → this also switches the board into **recovery mode**.

<p align="center">
    <img src="../99. Images/Pasted image 20250820022242.png" width="300">
</p>

## 3. Flashing

- eMMC firmware for Luckfox Pico Ultra W is located at: [eMMC_Images_0429](../../Tools/eMMC_Images_0429)
- Click `Search Path...` and select the folder containing the firmware files.
<p align="center">
    <img src="../99. Images/Pasted image 20250820023719.png" width="700">
</p>

- Then, Click `Yes` to reload the env file
<p align="center">
    <img src="../99. Images/Pasted image 20250820022858.png" width="700">
</p>

- Select all items and click `Download` button
<p align="center">
    <img src="../99. Images/Pasted image 20250820023433.png" width="700">
</p>

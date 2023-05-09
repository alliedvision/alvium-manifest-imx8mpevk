# Alvium CSI2 driver for i.MX 8M Plus Evaluation Kit

## Overview
This repository contains the manifest files for building the Allied Vision Alvium reference image for the i.MX 8M Plus Evaluation Kit. 

## Compatibility
This release is compatible with:
- i.MX 8M Plus EVK
- Alvium MIPI CSI-2 cameras with firmware 11.1 or higher

You can use 2 Alvium cameras at the same time if resolution of each camera is less than 4 MP. This is a restriction of the SOM's hardware.

This release supports v4l2 and GenICam for CSI2 access.
For GenICam for CSI2 access VimbaX 2023-2 is required.

## Quick Start
### Prerequisites
-  i.MX 8M Plus EVK
-  Host PC: See NXP's requirements for i.MX Yocto Project BSP Rev. 5.15.52-2.1.0 in the [i.MX Yocto Project User's Guide](https://www.nxp.com/docs/en/user-guide/IMX_YOCTO_PROJECT_USERS_GUIDE.pdf)
-  Alvium camera with Firmware 11.1
-  SD Card 8 GB

### Installation

Tip: For the prebake image, skip steps 1-4 and start with step 5.

To install the driver and layer:

1. Install the repo tool
```shell
    mkdir -p ~/.bin
    export PATH="${HOME}/.bin:${PATH}"
    curl https://storage.googleapis.com/git-repo-downloads/repo > ~/.bin/repo
    chmod a+rx ~/.bin/repo
    echo "export PATH=\"\${HOME}/.bin:\${PATH}\""  >> ~/.bashrc
```
2. Initialize project directory
```shell
    mkdir -p imx8mpevk-yocto
    cd imx8mpevk-yocto
    repo init https://github.com/alliedvision/alvium-manifest-imx8mpevk
    repo sync
```
3. Prepare yocto build
```shell
    source setup-avt-release.sh
```
4. Build the AVT Image with the command:  
```shell
    bitbake avs-image-alvium-validation
```
5. Flash the image to the SD card.  If you have built the image with yocto, you can find the image here:
            <build_dir>/tmp/deploy/images/imx8mpevk/avs-image-alvium-validation-imx8mpevk.rootfs.wic.bz2
6. Boot the board.
7. Check if the camera firmware version is 11.1 or higher. If the camera has an earlier firmware, perform an update with Vimba Firmware Updater.

### Board configuration
The image contains three different device tree configuration.
1. Two cameras connected to the board. Maximum width of both cameras is 2048. The camera on port CSI1 will be listed as "/dev/video3" and the camera on port CSI" as "/dev/video4". 
2. One camera connected to CSI1. Maximum width of the cameras is 4096. The camera will be listed as "/dev/video3".
3. One camera connected to CSI2. Maximum width of the cameras is 4096. The camera will be listed as "/dev/video3".
The active configuration can easily be changed by using the tool "avt-board-config"
All available configurations can be listed using the command:
```shell
    avt-board-config -l
```
The current configuration can be chaned with:
```shell
    avt-board-config -s <index>
```
The index parameter for each configration is returned by the list all configurations command.
After you have changed the configuration a reboot is required. 

## Limitations
General:
-  Maximum width of an image: 4096 pixels when only port 1 is used. Maximum width with both ports: 2048 pixels.
-  When streaming with the pixelformat "AR24", the v4l2 control "Alpha component" must be set before the stream is started. If the value of the v4l2 control is zero, on image will be visible.
-  For changing the camera feature via v4l2 controls, the v4l2 subdevice node of the cameras must be used. In many cases this is "/dev/v4l-subdev2".
-  The digital binning is activated by changing the width and height. You can enter any value for the width and height but the size will be adjusted to the nearsted binning configuration. 
Allied Vision V4L2Viewer:
- When the framerate is limited to ~300 fps, when the frames are displayed.
- The V4L2Viewer does not support all formats, that the hardware supports.
VimbaX:
- The payload size is limited to <value>. To test of the streaming with a configuration is possible please use the following formula to calculate the payload size
    ```
        <width> * <height> * <bpp>
    ```
- When using the weston terminal, you have to manually set the GenICam TL search enviroment variable.
```
    export GENICAM_GENTL64_PATH=<path to VimbaX directory>/cti
```        


## How to start a stream

cat  /sys/bus/i2c/devices/1-003c/device_temperature

e.g. for Alvium 1500 C-500c on Port CSI1 of imx8mp-evk:

```shell
    v4l2-ctl -d /dev/v4l-subdev2 --set-ctrl exposure=20000000,gain=100,brightness=0 --set-crop top=0,left=0,width=1920,height=1080
    gst-launch-1.0 v4l2src device=/dev/video3 ! video/x-raw,width=1920,height=1080,framerate=30/1,io-mode=dmabuf ! waylandsink sync=false -v
```
                
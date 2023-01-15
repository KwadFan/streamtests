# Testing go2rtc

This document describes my test procedures and how to setup my test environment.

## What you need:

-   A Raspberry Pi, i used a Raspberry Pi 4 version 1.4 with 4G Ram
-   A Sdcard minimum size 8GB. I used a generic 32GB Card
-   Putty or similar to ssh into.
-   Rpi Imager to flash a OS Image
-   A copy of Raspberry Pi OS 64bit 'lite'
-   A Raspicam, I used Raspicam V2 (2016 Model)
-   A USB Cam, with inbuilt MJPEG Encoder (most of them provide it)

## Initial setup:

1. Flash your copy of Raspberry Pi OS 64bit 'lite' to your SDcard using rpi imager. Dont forget to enable ssh and setup your wifi credentials and regional settings (locale, keyboard layout, Wifi Country Code)

2. Boot up the pi and ssh into it.

3. Install bare requirements for our task

    `sudo apt update && sudo apt install --yes --no-install-recommends git curl wget`

---

If you discover something like this:

```bash
52 packages can be upgraded. Run 'apt list --upgradable' to see them.
N: Repository 'http://deb.debian.org/debian bullseye InRelease' changed its 'Version' value from '11.5' to '11.6'
```

Please ensure we are using the latest updates, by running:

```bash
sudo apt update --allow-releaseinfo-change && sudo apt upgrade --yes
```

Make sure to reboot the Pi afterwards!

```bash
sudo reboot
```

---

## Prepare ustreamer

1. Install ustreamer dependencies as described in its [README](https://github.com/pikvm/ustreamer/blob/master/README.md).
   for now we want to compile without GPIO and Janus support.

    ```bash
    sudo apt install --yes --no-install-recommends libevent-dev libjpeg62-turbo-dev libbsd-dev
    ```

**_NOTE:_** Discovered that `libjpeg9-dev`is replaced by `libjpeg62-turbo-dev` other than described in ustreamer's README.

2. Clone the ustreamer repo to your `home`directory

    ```bash
    git clone  -b master --depth=1 https://github.com/pikvm/ustreamer.git
    ```

3. Lets ensure we use the same commit (to negotiate errors introdueced by ustreamer)

    ```bash
    git describe --tags --always
    ```

    This should be `c24c1ec`

4. Let's compile ustreamer

    ```bash
    cd ~/ustreamer
    make -j$(nproc)
    ```

5. Run a basic test, to confirm ustreamer is working.

    ```bash
    ./ustreamer -v
    ```

    This should show `5.36` if the commits matches.

6. Confirm ustreamer shows at least a picture

    Determine your `/dev/video*` device by running

    ```bash
    v4l2-ctl --list-devices
    ```

    After that run ustreamer with some arguments

    ```bash
    ./ustreamer --host :: --device=/dev/video2 --format=MJPEG
    ```

    **NOTE:** Your device location might differ ... In my cas its `video2`

    Now head over to your browser and navigate to
    `http://<yourpisipaddress>:8080/stream`

    Seeing a stream of your camera? Fine, lets continue. Hit `CTRL+c`to kill ustreamer.

## Prepare go2rtc

1. Grab go2rtc binary and make it executable

    ```bash
    cd ~
    wget https://github.com/AlexxIT/go2rtc/releases/download/v0.1-rc.8/go2rtc_linux_arm64
    chmod +x ./go2rtc_linux_arm64
    ```

2. Run a basic test.

    ```bash
    ./gortc_linux_arm64
    ```

    Head over to Byour Browser again and navigate to
    `http://<yourpisipaddress>:1984/`

    This should show you that:

    ![go2rtc-api](./assets/go2rtc-api.png)

Did you see that page? Fine. We are finished with the preparation.

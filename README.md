# wsl2_linux_kernel_usbcam_enable_conf
Configuration file to build the kernel to access the USB camera connected to the host PC using USBIP from inside the WSL2.

## 1. Environment
- Windows 11 (OS build: 22000.194+)
- WSL2
- Ubuntu 20.04

## 1. Usage
On WSL2.
```bash
$ uname -r -v
5.10.16.3-microsoft-standard-WSL2 #1 SMP Fri Apr 2 22:23:49 UTC 2021
```
On Windows Terminal.
```shell
C:\> wsl --update
Checking for updates...
Downloading update...
Installing update...
This change will take effect the next time you reboot WSL. To force a reboot, run 'wsl --shutdown'.
Kernel version: 5.10.60.1

C:\> wsl --shutdown
```
On WSL2.
```
$ uname -r -v
5.10.60.1-microsoft-standard-WSL2 #1 SMP ...

$ sudo apt update && sudo apt upgrade -y
$ sudo apt install -y build-essential flex bison \
libgtk2.0-dev libelf-dev libncurses-dev autoconf \
libudev-dev libtool zip unzip v4l-utils libssl-dev \
python3-pip cmake git iputils-ping net-tools

$ cd /usr/src
$ TAGVERNUM=5.10.60.1 && \
TAGVER=linux-msft-wsl-${TAGVERNUM}
$ sudo git clone -b ${TAGVER} \
https://github.com/microsoft/WSL2-Linux-Kernel.git \
${TAGVERNUM}-microsoft-standard && \
cd ${TAGVERNUM}-microsoft-standard

$ sudo cp Microsoft/config-wsl ./.config && \
sudo make clean && sudo make menuconfig
```

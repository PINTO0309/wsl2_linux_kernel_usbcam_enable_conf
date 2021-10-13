# wsl2_linux_kernel_usbcam_enable_conf
Configuration file to build the kernel to access the USB camera connected to the host PC using USBIP from inside the WSL2.

## 1. Environment
- Windows 11 (OS build: 22000.194+)
- WSL2
- Ubuntu 20.04
- https://github.com/microsoft/WSL2-Linux-Kernel

## 2. Usage
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
```bash
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

$ sudo wget -O .config https://github.com/PINTO0309/wsl2_linux_kernel_usbcam_enable_conf/raw/main/${TAGVER}/config && \
sudo chmod 777 .config && \
sudo make clean

$ sudo make -j$(nproc) && \
sudo make modules_install -j$(nproc) && \
sudo make install -j$(nproc)

$ cd tools/usb/usbip/
$ sudo ./autogen.sh && sudo ./configure
$ sudo sed 's/-Werror//g' -i Makefile && \
sudo sed 's/-Werror//g' -i src/Makefile && \
sudo sed 's/-Werror//g' -i libsrc/Makefile
$ sudo make install -j$(nproc)
$ sudo cp libsrc/.libs/libusbip.so.0 /lib/libusbip.so.0
$ sudo rm /mnt/c/Users/<windows username>/vmlinux
$ sudo cp /usr/src/${TAGVERNUM}-microsoft-standard/vmlinux /mnt/c/Users/<windows username>/
$ cat << 'EOT' > /mnt/c/Users/<windows username>/.wslconfig
[wsl2]
kernel=C:\\Users\\<windows username>\\vmlinux
EOT
```
On Windows Terminal.
```shell
C:\> wsl --shutdown
```
On WSL2. If the built kernel has been loaded successfully, you will see **`+`** at the end of the kernel name. **`#n`** is the number of times the kernel was built.
```bash
$ uname -r -v
5.10.60.1-microsoft-standard-WSL2+ #1 SMP Tue Oct 12 12:05:07 JST 2021
```

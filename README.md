# wsl2_linux_kernel_usbcam_enable_conf
Configuration file to build the kernel to access the USB camera connected to the host PC using USBIP from inside the WSL2.
[Windows 11 + WSL2 + USB Camera + Serial](https://zenn.dev/pinto0309/articles/0723ae46501beb)

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
## 3. Note
If you receive the following error, please follow the additional steps: USB bandwidth issues may cause the information exchange with the camera to time out.
- Error Message
```
[ WARN:0] global /tmp/pip-req-build-xw6jtoah/opencv/modules/videoio/src/cap_v4l.cpp (1001) tryIoctl VIDEOIO(V4L2:/dev/video0): select() timeout.
```

Try one or both of the following.

- Try1. Additional command
```bash
$ sudo apt install v4l-utils && \
sudo chmod 777 /dev/video0 && \
v4l2-ctl -d /dev/video0 --all

Driver Info:
	Driver name      : uvcvideo
	Card type        : papalook FHD Camera: papalook F
	Bus info         : usb-0000:00:14.0-11.1
	Driver version   : 5.11.22
	Capabilities     : 0x84a00001
		Video Capture
		Metadata Capture
		Streaming
		Extended Pix Format
		Device Capabilities
	Device Caps      : 0x04200001
		Video Capture
		Streaming
		Extended Pix Format
Media Driver Info:
	Driver name      : uvcvideo
	Model            : papalook FHD Camera: papalook F
	Serial           : 
	Bus info         : usb-0000:00:14.0-11.1
	Media version    : 5.11.22
	Hardware revision: 0x00000100 (256)
	Driver version   : 5.11.22
Interface Info:
	ID               : 0x03000002
	Type             : V4L Video
Entity Info:
	ID               : 0x00000001 (1)
	Name             : papalook FHD Camera: papalook F
	Function         : V4L2 I/O
	Flags         : default
	Pad 0x01000007   : 0: Sink
	  Link 0x02000010: from remote pad 0x100000a of entity 'Extension 3': Data, Enabled, Immutable
Priority: 2
Video input : 0 (Camera 1: ok)
Format Video Capture:
	Width/Height      : 1280/720
	Pixel Format      : 'MJPG' (Motion-JPEG)
	Field             : None
	Bytes per Line    : 0
	Size Image        : 1843200
	Colorspace        : sRGB
	Transfer Function : Rec. 709
	YCbCr/HSV Encoding: ITU-R 601
	Quantization      : Default (maps to Full Range)
	Flags             : 
Crop Capability Video Capture:
	Bounds      : Left 0, Top 0, Width 1280, Height 720
	Default     : Left 0, Top 0, Width 1280, Height 720
	Pixel Aspect: 1/1
Selection Video Capture: crop_default, Left 0, Top 0, Width 1280, Height 720, Flags: 
Selection Video Capture: crop_bounds, Left 0, Top 0, Width 1280, Height 720, Flags: 
Streaming Parameters Video Capture:
	Capabilities     : timeperframe
	Frames per second: 30.000 (30/1)
	Read buffers     : 0
                     brightness 0x00980900 (int)    : min=-64 max=64 step=1 default=0 value=0
                       contrast 0x00980901 (int)    : min=0 max=95 step=1 default=0 value=0
                     saturation 0x00980902 (int)    : min=0 max=100 step=1 default=30 value=30
                            hue 0x00980903 (int)    : min=-2000 max=2000 step=100 default=0 value=0
 white_balance_temperature_auto 0x0098090c (bool)   : default=1 value=1
                          gamma 0x00980910 (int)    : min=100 max=300 step=1 default=100 value=100
           power_line_frequency 0x00980918 (menu)   : min=0 max=2 default=1 value=1
				0: Disabled
				1: 50 Hz
				2: 60 Hz
      white_balance_temperature 0x0098091a (int)    : min=2800 max=6500 step=1 default=4600 value=4600 flags=inactive
                      sharpness 0x0098091b (int)    : min=1 max=7 step=1 default=1 value=1
         backlight_compensation 0x0098091c (int)    : min=0 max=3 step=1 default=2 value=2
```
Check the WIDTH and HEIGHT of the standard resolution displayed, and correct the input resolution listed in **`usbcam_test.py`**.

e.g.

From:
```python
frame = cv2.resize(frame, (640, 480))
```
To:
```python
frame = cv2.resize(frame, (1280, 720))
```
Run it again.
```bash
$ sudo chmod 777 /dev/video0 && python3 ${HOME}/usbcam_test.py
```

- Try2. It may not work well with UVC-compatible USB cameras depending on their compatibility. Try replacing several USB cameras. The following cameras were recognized successfully. I have tried four different USB cameras and only one was successful.

https://www.amazon.co.jp/ELP-USB%E3%82%AB%E3%83%A1%E3%83%A9%E3%83%A2%E3%82%B8%E3%83%A5%E3%83%BC%E3%83%AB-%E3%83%95%E3%83%AA%E3%83%BC%E3%83%89%E3%83%A9%E3%82%A4%E3%83%90%E3%83%BC%E3%82%A6%E3%82%A7%E3%83%96-VR%E3%82%AB%E3%83%A1%E3%83%A9%EF%BC%88%E3%83%A2%E3%83%87%E3%83%AB%EF%BC%9AELP-USB8MP02G-L180-JP%EF%BC%89/dp/B08FDJW3HS

https://www.amazon.co.jp/PAPALOOK-PA452-%E3%82%A6%E3%82%A7%E3%83%96%E3%82%AB%E3%83%A1%E3%83%A9-360%E5%BA%A6%E5%9B%9E%E8%BB%A2%E5%8F%AF%E8%83%BD-%E3%82%A4%E3%83%AB%E3%83%9F%E3%83%8D%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E4%BB%98%E3%81%8D/dp/B01LXHPJ5Z/ref=sr_1_4?__mk_ja_JP=%E3%82%AB%E3%82%BF%E3%82%AB%E3%83%8A&dchild=1&keywords=PA452+pro&qid=1634110776&s=computers&sr=1-4

- Built-in camera for Thinkpad laptops
- Confirmed to work with Windows 11 Home (21H2) OS build: 22000.194
- Confirmed to work with Windows 10 Pro (21H1) OS build: 19043.1237


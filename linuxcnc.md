# Installing LinuxCNC 2.9 on Raspberry Pi 4

I've seen a lot of people, including myself, have trouble getting LinuxCNC running on the Raspberry Pi 4. 

I've just gotten it working, so I thought I'd share my recipe.

## On Linux desktop computer

These commands were tested on Linux Mint 21, but any reasonably recent Debian distro should work.

### Install Raspberry Pi Imager

```bash
sudo apt install rpi-imager
```

If that doesn't work, you can manually download and install from `https://www.raspberrypi.com/software/`

### Build Rasberry Pi OS image with Realtime-RT kernel

```bash
sudo apt install git
git clone --depth 1 -b linuxcnc https://github.com/alangibson/rpi-rt-kernel.git
cd rpi-rt-kernel
make RASPIOS_IMAGE_NAME=raspios_full_arm64
```

The script will spit out the name of the OS image file at the end of its run. 

```
/raspios
  adding: 2022-09-22-raspios-bullseye-arm64-full.img (deflated 65%)
```

### Install image with Rasbperry Pi Imager

- Insert the SD card into your computer.
- Open Raspberry Pi Imager.
- Click "Choose OS" button. Select "Use Custom" from the bottom of the menu. Select the '.img' file you just built in the `rpi-rt-kernel` directory.
- Click "Choose Storage" button. Select your SD card. 
- Click the settings sprocket image button. Set your WLAN credentials, enable ssh, hostname, username/password, and timezone in the settings. I use
    hostname = linuxcnc
    enable ssh = checked
      use password authentication = selected
    set username and password = checked
      username = pi
      password = (password of your choice)
    configure wireless lan = checked
      SSID = (your wifi access point name)
      password = (your wifi password)
    wireless lan country = (your country code)
    set locale settings = checked
      timezone = (your timezone)
- Click the "Write" button

### Boot Raspberry Pi

Insert the micro SD card in your RPi 4 and power it up. If you correctly set your WLAN credentials, it should be available on your Wifi network in about a minute. You can watch for it to appear by running `ping`.

```bash
ping linuxcnc.local
```

### Connect to Raspberry Pi

Once the Rasberry Pi is online, `ssh` in to it

```bash
ssh pi@linuxcnc.local
```

Enter your password when prompted.

## On Rasbperrry Pi

The following instructions are based on `https://gnipsel.com/linuxcnc/debian-11-emc.html`

### Fix locale setting

```bash
echo "export LANGUAGE=$LANG" >> ~/.bashrc
echo "export LC_ALL=$LANG" >> ~/.bashrc
source ~/.bashrc
```

### Build LinuxCNC

```bash
sudo apt install build-essential devscripts autoconf automake  \
  debhelper dh-python libudev-dev bwidget \
  intltool libboost-python-dev libepoxy-dev libgl1-mesa-dev \
  libglu1-mesa-dev libgtk2.0-dev libgtk-3-dev libmodbus-dev \
  libeditreadline-dev libxmu-dev netcat po4a python3-dev \
  python3-tk python3-xlib tcl8.6-dev tclx tk8.6-dev yapps2 \
  asciidoc  docbook-xsl dvipng groff imagemagick  \
  python3-lxml source-highlight w3c-linkchecker xsltproc \
  texlive-extra-utils texlive-font-utils texlive-fonts-recommended \
  texlive-lang-cyrillic texlive-lang-french texlive-lang-german \
  texlive-lang-polish texlive-lang-spanish texlive-xetex \
  texlive-latex-recommended dblatex asciidoc-dblatex \
  libusb-1.0-0-dev graphviz inkscape texlive-lang-european
git clone -b 2.9 --depth 1 https://github.com/LinuxCNC/linuxcnc.git build
cd build
./debian/configure no-docs
dpkg-checkbuilddeps
debuild -uc -us
```

### Install LinuxCNC

```bash
sudo apt install libxml2 gir1.2-gtksource-3.0 libxml2-dev libglew2.1 mesa-utils python3-configobj libgtksourceview-3.0-dev tclreadline
cd ..
sudo dpkg -i linuxcnc-uspace_2.9.0~pre1_arm64.deb
```

### Install Mesaflash

```bash
git clone https://github.com/LinuxCNC/mesaflash.git
sudo apt install libpci-dev libmd-dev pkg-config
cd mesaflash
sudo make install
```

### Install Mesa Configuration Tool

```bash
cd ~
sudo apt install python3-packaging
curl -L -O https://github.com/jethornton/mesact/raw/master/mesact_1.1.2_arm64.deb
sudo dpkg -i mesact_1.1.2_arm64.deb
```

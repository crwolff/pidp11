# Configuration of a Raspberry Pi 3B+ for PiDP11/Bookworm

## OS Installation

1) Insert microSD card into computer
2) Run Raspberry Pi Imager (sudo ./imager_2.0.0_amd64.AppImage)
    - OS - Legacy (bookworm) / 64 bit / Lite
    - Enable SSH
3) Insert microSD card into RPi and power up
4) First boot is pretty slow

## Configuration

### Update system otherwise later installations may fail

```
sudo apt update
sudo apt dist-upgrade -y
```

### Install Minimal LXDE Desktop and TigerVNC

```
sudo apt install lxde-core lxappearance lightdm pipanel raspberrypi-ui-mods -y
sudo apt install xfonts-100dpi xfonts-75dpi xfonts-scalable tigervnc-standalone-server -y
```

### Lightdm Configuration

```
sudo vi /etc/lightdm/lightdm.conf
```
Enable XDMCP (login)
```
[XDMCPServer]
enabled=true
port=177
```
Disable HDMI desktop (if headless)
```
start-default-seat=false
```

### Startup to desktop
```
sudo systemctl set-default graphical.target
```

## Configure TigerVNC to start automatically

Setup systemd socket
```
sudo tee /etc/systemd/system/xvnc1.socket > /dev/null << __EOF__
[Unit]
Description=XVNC Server 1

[Socket]
ListenStream=5901
Accept=yes

[Install]
WantedBy=sockets.target
__EOF__
```
Setup systemd service
```
sudo tee /etc/systemd/system/xvnc1@.service > /dev/null << __EOF__
[Unit]
Description=XVNC Per-Connection Daemon 1

[Service]
ExecStart=-/usr/bin/Xtigervnc -noreset -SecurityTypes None -inetd -query 127.0.0.1 -geometry 1600x1024 -pn -once
StandardInput=socket
StandardOutput=socket
StandardError=journal
__EOF__
```
Enable systemd socket, but not service (runs when socket activates)
```
sudo systemctl daemon-reload
sudo systemctl enable xvnc1.socket
```

### Optional

Disable automatic updates.
```
sudo systemctl mask packagekit
```

### Test VNC

```
sudo shutdown -r now
```
```
vncviewer pidp11:1
```

## Install PiDP-11

Answer 'C' to recompile from source, 'Y' for everything else
```
sudo apt install -y git
cd /opt
sudo git clone https://github.com/crwolff/pidp11.git
/opt/pidp11/install/install.sh
```

## Customizations

```
cp /opt/pidp11/install/pidp11-useroptions.rc /opt/pidp11/pidp11-useroptions.rc
sed -i -e 's/[# ]*export.*PIDP_11_ROTATION=.*/export PIDP_11_ROTATION="FLIP"/' /opt/pidp11/pidp11-useroptions.rc
sed -i -e 's/eth0/wlan0/i' /opt/pidp11/systems/*/*.ini
```

## Reboot

```
sudo shutdown -r now
```

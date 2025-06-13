# Installation Guide for xmm7360-pci on Arch Linux

## 1. Checking for the Device

Run the following command to check if your modem is detected:

```sh
lspci -nn | grep 8086:7360
```

If the output contains a line with `8086:7360`, your device is present.

## 2. Installing Required Packages

Install Python and necessary dependencies:

```sh
sudo pacman -S --needed base-devel git dkms python python-configargparse python-pyroute2 python-dbus
```

```sh
sudo pacman -S python-pip
```

Update and install Linux headers:

```sh
sudo pacman -Syu linux-headers
```
or
```sh
sudo pacman -S linux-lts linux-lts-headers
```

Reboot the system:

```sh
sudo reboot
```

## 3. Verifying Kernel Headers

After rebooting, verify that the kernel headers are correctly installed:

```sh
ls -l /lib/modules/$(uname -r)/build
```

If the directory exists, you can proceed.

## 4. Cloning the Driver Repository

Clone the repository:

```sh
git clone https://github.com/perdoqx/xmm7360-pci.git
```

Navigate to the repository:

```sh
cd xmm7360-pci
```

## 5. Compiling and Loading the Driver

Compile and load the driver:

```sh
make && sudo make load
```

## 6. Configuring the Modem

Copy the sample configuration file:

```sh
cp xmm7360.ini.sample xmm7360.ini
```

Modify the `apn` parameter in `xmm7360.ini` (example for Beeline Russia):


```
apn=internet.beeline.ru

```

Run the following command to establish a connection (replace `apn.url` with your actual APN):

```sh
sudo python3 rpc/open_xdatachannel.py --apn apn.url
```

Set up a DNS server:

```sh
echo "nameserver 1.1.1.1" >> /etc/resolv.conf
```
or
```sh
echo "nameserver 1.1.1.1" | sudo tee -a /etc/resolv.conf
```

Enable the network interface:

```sh
sudo ip link set wwan0 up
```

## 7. Automatic Setup (Recommended)

To simplify the modem setup, use the provided script:
### Installation
After copy the sample configuration file:

```sh
cp xmm7360.ini.sample xmm7360.ini
```

Run the following command to install and set up the modem:

```sh
sudo ./scripts/lte.sh setup
```

### Usage
Once installed, use the following command to bring up the LTE connection:

```sh
sudo lte up
```

This script automates the configuration and connection process, making it easier to manage the modem.

## 8. Automating on Boot (via systemd)

Create a service file:
```sh
sudo nano /etc/systemd/system/lte.service
```

Paste the following (replace YOUR_USERNAME):
```sh
[Unit]
Description=Intel XMM7360 LTE Modem Connection
After=network.target

[Service]
Type=oneshot
WorkingDirectory=/home/YOUR_USERNAME/xmm7360-pci
ExecStart=/home/YOUR_USERNAME/xmm7360-pci/scripts/lte.sh setup
ExecStartPost=/usr/local/bin/lte up
RemainAfterExit=true

[Install]
WantedBy=multi-user.target
```

Replace YOUR_USERNAME with your actual user name.

Then run:
```sh
sudo systemctl daemon-reload
```
```sh
sudo systemctl enable lte.service
```
```sh
sudo systemctl start lte.service
```

Now your modem will automatically initialize and connect at boot!
🧪 Testing

After reboot, run:
```sh
ip addr show wwan0
```
```sh
ping -c 3 1.1.1.1
```
---
Your xmm7360 PCI modem should now be fully functional on Arch Linux! 🚀

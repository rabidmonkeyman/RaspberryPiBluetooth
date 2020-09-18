# RaspberryPiBluetooth
This is a repository I use to document all my learning with Bluetooth capabilities and commands of a raspberrypi.

Using a Raspberry Pi 3 Model B v1.2
- BCM43438 wireless LAN and Bluetooth Low Energy (BLE) on board

## Commands
 ```
 sudo apt update - make sure to run this so you can install the bluetooth package
 sudo apt install bluetooth pi-bluetooth bluez blueman - This downloads the necessary files to run bluetooth, bluetooth doesn't come with the operating system
 sudo apt-get install bluez-tools - This was another bluetooth driver that I downloaded and tried
 sudo apt-get blueman
 systemctl status bluetooth - if Active: inactive (dead) then run the command systemctl start bluetooth
 systemctl start bluetooth
 hciconfig - prints name and basic information about all the Bluetooth devices installed in the system
 hcitool scan - gets the BD address of the pairable devices in range
 sudo bluetoothctl - This sends you into a shell of your bluetooth module
 ```
## Setup Instructions
```
sudo apt update                                        // make sure to run this so you can install the bluetooth package
sudo apt-get dist-upgrade                              // make sure to run this so you can install the bluetooth package
sudo apt install bluetooth pi-bluetooth bluez blueman  // if you get the error "E:" make sure you did sudo apt upgrade
sudo apt-get install pi-bluetooth                      // This is needed to start hciuart will give error "Failed to start hciuart.service: Unit hciuart.service is masked." if you do not do this

sudo reboot
systemctl status bluetooth                             // if Active: inactive (dead) then run the command systemctl start bluetooth
systemctl enable bluetooth
systemctl start bluetooth
systemctl start hciuart                                // This keep giving me errors saying "Jobs..." 
sudo bluetoothctl                                      // This sends you into a shell of your bluetooth module
agent on
default-agent
scan on                                                // If you get the problem "No Default controller available" you have to fix it lol
```
## Useful Links

- https://www.cnet.com/how-to/how-to-setup-bluetooth-on-a-raspberry-pi-3/  
- https://pimylifeup.com/raspberry-pi-bluetooth/
- https://scribles.net/disabling-bluetooth-on-raspberry-pi/
- https://www.linux-magazine.com/Issues/2017/197/Command-Line-bluetoothctl

## Device Tree Stuff Relating to Bluetooth
Located @ /boot/overlays/README
```
Params:
        krnbt                   Set to "on" to enable autoprobing of Bluetooth
                                driver without need of hciattach/btattach
                                (default "off")
                              
                              
Name:   balena-fin
Info:   Overlay that enables WiFi, Bluetooth and the GPIO expander on the
        balenaFin carrier board for the Raspberry Pi Compute Module 3/3+ Lite.
Load:   dtoverlay=balena-fin
Params: <None>


Name:   disable-bt
Info:   Disable onboard Bluetooth on Pi 3B, 3B+, 3A+, 4B and Zero W, restoring
        UART0/ttyAMA0 over GPIOs 14 & 15.
        N.B. To disable the systemd service that initialises the modem so it
        doesn't use the UART, use 'sudo systemctl disable hciuart'.
Load:   dtoverlay=disable-bt
Params: <None>
 
 
Name:   miniuart-bt   //This is also known as pi3-miniuart-bt
Info:   Switch the onboard Bluetooth function on Pi 3B, 3B+, 3A+, 4B and Zero W
        to use the mini-UART (ttyS0) and restore UART0/ttyAMA0 over GPIOs 14 &
        15. Note that this may reduce the maximum usable baudrate.
        N.B. It is also necessary to edit /lib/systemd/system/hciuart.service
        and replace ttyAMA0 with ttyS0, unless using Raspbian or another
        distribution with udev rules that create /dev/serial0 and /dev/serial1,
        in which case use /dev/serial1 instead because it will always be
        correct. Furthermore, you must also set core_freq and core_freq_min to
        the same value in config.txt or the miniuart will not work.
Load:   dtoverlay=miniuart-bt,<param>=<val>
Params: krnbt                   Set to "on" to enable autoprobing of Bluetooth
                                driver without need of hciattach/btattach
                                
```

- This is what /usr/bin/btuart looks like
```#!/bin/sh

HCIATTACH=/usr/bin/hciattach
if grep -q "Pi 4" /proc/device-tree/model; then
  BDADDR=
else
  SERIAL=`cat /proc/device-tree/serial-number | cut -c9-`
  B1=`echo $SERIAL | cut -c3-4`
  B2=`echo $SERIAL | cut -c5-6`
  B3=`echo $SERIAL | cut -c7-8`
  BDADDR=`printf b8:27:eb:%02x:%02x:%02x $((0x$B1 ^ 0xaa)) $((0x$B2 ^ 0xaa)) $((0x$B3 ^ 0xaa))`
fi

if [ -e /sys/class/bluetooth/hci0 ]; then
  # Bluetooth is already enabled
  exit 0
fi

uart0="`cat /proc/device-tree/aliases/uart0`"
serial1="`cat /proc/device-tree/aliases/serial1`"

if [ "$uart0" = "$serial1" ] ; then
        uart0_pins="`wc -c /proc/device-tree/soc/gpio@7e200000/uart0_pins/brcm\,pins | cut -f 1 -d ' '`"
        if [ "$uart0_pins" = "16" ] ; then
                $HCIATTACH /dev/serial1 bcm43xx 3000000 flow - $BDADDR
        else
                $HCIATTACH /dev/serial1 bcm43xx 921600 noflow - $BDADDR
        fi
else
        $HCIATTACH /dev/serial1 bcm43xx 460800 noflow - $BDADDR
fi




```

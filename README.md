# RaspberryPiBluetooth
This is a repository I use to document all my learning with Bluetooth capabilities and commands of a raspberrypi.


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

https://www.cnet.com/how-to/how-to-setup-bluetooth-on-a-raspberry-pi-3/   /n \n
https://pimylifeup.com/raspberry-pi-bluetooth/

# RaspberryPiBluetooth
This is a repository I use to document all my learning with Bluetooth capabilities and commands of a raspberrypi. Using a Raspberry Pi 3B+

## Commands

### General Bluetooth Connections
 ```
 systemctl status bluetooth.service // If Active: inactive (dead) then run the command systemctl start bluetooth.service
 systemctl start bluetooth.service
 
 system status hciuart.service // If you get an error in bluetoothctl saying "No default controller available" then run the command systemctl start hciuart.service
 systemctl start hciuart.service
 
 hciconfig -a // Prints name and basic information about all the Bluetooth devices installed in the system
 sudo hciconfig hci0 name 'New device name' // This is how to change the name of the bluetooth device, I am not sure if you need to restart anything or if this saves after reboot
 
 hcitool scan // Gets the BD address of the pairable devices in range
 
 hcitool dev // Shows you the MAC address of the raspberry pi
 
 sudo bluetoothctl // This sends you into a shell of your bluetooth module
 ```
 
## Setup Instructions for Phone
```
sudo bluetoothctl                                      // Make sure you have the sudo, when connecting you should see "[NEW] Controller XX:XX:XX:XX:XX:XX PiTorch [default]"
agent on
default-agent
scan on                                                 // This is fundimentally the same as "hcitool scan" however this scan repeates every few seconds until told to stop
scan off                                                // This is how you turn off scan
discoverable on                                         // This sets the pi to a "hotspot mode" so i can now connect to it from my phone for example. When connecting to a phone for example you have to say yes on the pi and phone when connecting since you will be prompted with a passcode
```

## Setup Instructions for Headphones

`sudo apt-get install bluealsa pulseaudio                //This is some sort of audio driver for bluetooth on the pi I found`  
In this file:  
`sudo nano /lib/systemd/system/bluetooth.service`  
Change the line   
`ExecStart=/usr/lib/bluetooth/bluetoothd`  
to  
`ExecStart=/usr/lib/bluetooth/bluetoothd --noplugin=sap`  
then  
`sudo reboot`  

In this file:  
`sudo nano /lib/systemd/system/bthelper@.service`  
add the line:  
`ExecStartPre=/bin/sleep 2`  
before  
`ExecStart=/usr/bin/bthelper %I`  
then
`sudo reboot`

Now to check if pulseaudio is running use:  
`ps aux | grep pulseaudio`  
if you do not see something like this:  
```
pi@raspberrypi:~ $ ps aux | grep pulseaudio
pi 544 4.3 1.8 181592 17720 ? Sl 22:02 0:00 pulseaudio --start
pi 564 0.0 0.0 7348 488 pts/0 S+ 22:02 0:00 grep --color=auto pulseaudio
```
then you need to start pulseaudio using this command:  
`pulseaudio --start`  
Now you are ready to connect to your bluetooth headphones doing this:  
```
sudo bluetoothctl
trust XX:XX:XX:XX:XX:XX                     //I am not sure if this is 100% needed to connect
pair XX:XX:XX:XX:XX:XX                      //I am not sure if this is 100% needed to connect
connect XX:XX:XX:XX:XX:XX
```

```
aplay -D bluealsa:HCI=hci0,DEV=F4:7D:EF:12:88:19,PROFILE=a2dp The\ Weeknd\ -\ Blinding\ Lights\ \(Official\ Music\ Video\).mp3
// aplay DOES NOT support mp3 files

sudo apt-get install mpg123  // This is a package i found online that says it supprts mp3 files
mpg123 -a bluealsa:HCI=hci0,DEV=F4:7D:EF:12:88:19,PROFILE=a2dp The\ Weeknd\ -\ Blinding\ Lights\ \(Official\ Music\ Video\).mp3
```




## Useful Links

- https://www.cnet.com/how-to/how-to-setup-bluetooth-on-a-raspberry-pi-3/  
- https://pimylifeup.com/raspberry-pi-bluetooth/
- https://scribles.net/disabling-bluetooth-on-raspberry-pi/
- https://www.linux-magazine.com/Issues/2017/197/Command-Line-bluetoothctl
- https://bluedot.readthedocs.io/en/latest/pairpiandroid.html
- https://peppe8o.com/fixed-connect-bluetooth-headphones-with-your-raspberry-pi/
- http://www.bytran.org/qtrpicrosscompile.htm
- https://doc.qt.io/qt-5/qbluetoothlocaldevice.html
- https://doc.qt.io/qt-5/qtbluetooth-overview.html
- https://api.kde.org/frameworks/bluez-qt/html/classBluezQt_1_1Manager.html

## Device Tree Stuff Relating to Bluetooth
Located @ /boot/overlays/README_
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

## Scripts

BConnect - Bluetooth Connect  
BDisconnect - Bluetooth Disconnect  
BScan - Bluetooth Scan  
scanner - QT QML Bluetooth App


 ## Enabling Bluetooth on Pi for QT

Building Qt Bluetooth  - https://doc.qt.io/qt-5/qtbluetooth-index.html

Despite the fact that the module can be built for all Qt platforms, the module is not ported to all of them. Not supported platforms employ a fake or dummy backend which is automatically selected when the platform is not supported. The dummy backend reports appropriate error messages and values which allow the Qt Bluetooth developer to detect at runtime that the current platform is not supported. The dummy backend is also selected on Linux if BlueZ development headers are not found during build time or Qt was built without Qt D-Bus support.  
 
I am currently having a problem when running the qt bluetooth examples saying `qt.bluetooth: Dummy backend running. Qt Bluetooth module is non-functional`   
Currently I am getting an error with this part in my qt code as well, i think it has to do with the dummy backend running:    
`BluetoothDiscoveryModel.InvalidBluetoothAdapterError`   
Here is the link for the documentation of that code:  
- https://doc.qt.io/qt-5/qml-qtbluetooth-bluetoothdiscoverymodel.html  






I've Installed `sudo apt-get install libqt5bluetooth5 qtconnectivity5-dev`, which didn't work  

- http://www.bytran.org/qtrpicrosscompile.htm

If you plan on using Bluetooth, the following libraries (libbluetooth-dev bluetooth blueman bluez libusb-dev libdbus-1-dev bluez-hcidump bluez-tools) need to be installed onto the Raspbian system after step 6 and before step 8 in the above guide as outlined in the following post.   These libraries must be installed onto the Raspberry Pi before the initial sync of the libs located on the Raspbian system (step 8 in the above guide) to the host cross-compilation computer.   As outlined in the above post, if you fail to install these libraries before you cross-compile, the Bluetooth will not be functional resulting in the "qt.bluetooth: Dummy backend running. Qt Bluetooth module is non-functional." message when you try to run your application. 

I ended up doing as this website says and still not working.  

My only question in my mind is that the Bluetooth App on QT is designed for something other than linux, like maybe windows or android????  

Looking at this page I think maybe its a problem with BlueZ (The driver for bluetooth on the pi) and QT communicating properly. https://doc.qt.io/qt-5/qtbluetooth-attribution-bluez.html

Earbud MAC Address: F4:7D:EF:12:88:19

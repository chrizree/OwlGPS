# OwlGPS

The "OwlGPS" (hoot hoot satellite)

Note! This works on the Hak5 WiFi Pineapple Mark VII as well with the same Bluetooth adapter as mentioned below. Just start on the opkg lines, the Signal Owl payloads aren't needed on the Pineapple.

Originally, I had the idea to hook up a GPS device to the Signal Owl in order to obtain GPS data, but... honestly... who carries around a GPS device these days?! You already have one in your pocket... your phone! OK, so this is how to get GPS data to the Signal Owl using an Android phone and the "official" Hak5 USB Bluetooth dongle (not a dongle really, but a "nano" USB adapter). The Bluetooth dongle is based on the Qualcomm CSR8510 chipset if trying to find alternatives. I guess it's pretty commonly used. Also worth mentioning is that the reason for jumping over the stream to get water is that the Signal Owl is kind of deprecated and will not get updates. That means that there are some problems adding functionality to OpenWrt. It would be easy to just attach a USB based GPS device, but since those devices most often needs the kmod-usb-acm package, it's not possible to use them since you can't easily install the kmod due to the fact that the Signal Owl has a kernel that is too old and causes conflicts when trying to install packages.

Things needed:
- Hak5 Signal Owl with the latest firmware (1.0.1 as of Dec 2020)
- A Qualcomm CSR8510 based USB Bluetooth adapter
- An Android phone with built in GPS capabilities (which one hasn't...), using an old Sony Xperia Z3 here
- "Bluetooth GPS Output" app from the Google Play store (there are other apps that most likely works as well)
  https://play.google.com/store/apps/details?id=com.meowsbox.btgps&hl=en_US&gl=US
- A computer with (preferably) Linux, using Kali Linux here on a Lenovo X220


Start with adding the WiFi-Connect payload (and the extension) to get the Owl online in attack mode (to avoid the need to put the Owl in arming mode all the time)

https://docs.hak5.org/hc/en-us/articles/360034023933-Getting-the-Signal-Owl-Online

https://github.com/hak5/signalowl-payloads/blob/master/payloads/library/wifi/WiFi-Connect/payload.txt

https://github.com/hak5/signalowl-payloads/blob/master/payloads/extensions/wifi_connect.sh


Find out the IP address that the Owl got on the network to which it is connected and then ssh into it and verify internet access

Power the Owl off and connect the USB Bluetooth adapter to the Owl (to the host port), then boot it up again and ssh into it

Most of the tools needed to connect Bluetooth devices should already be available on the Owl, but we might need to set the bps for the "emulated" COM port later on, so let's install a util package to get stty on the system

opkg update

opkg install coreutils-stty

Then run the following command to get information about the Bluetooth adapter:

hciconfig

Note the device name (most likely hci0) and also note the fact that it is probably in the state DOWN, bring the device up using:

hciconfig hci0 up

Run hciconfig again to verify that the state now has been changed to UP (or UP RUNNING PSCAN)

Make sure the device you want to connect to (the Android phone) is in "discovery mode"

Also start the Bluetooth app on the phone and set "Bluetooth Service Controls" to "Enabled"

Run:

bluetoothctl

Then, from the [bluetooth]# prompt, run:

power on

agent on

default-agent

scan on

Devices should now start to pop up on the screen. Discover the Bluetooth MAC address of the Android phone in the list of scanned devices that appears.

scan off

(it's of course also possible to get the bluetooth MAC address from the phone's "About phone" menu)

trust (Bluetooth MAC address)

pair (Bluetooth MAC address)

(if getting errors pairing when it has worked before with the device, run: remove (Bluetooth MAC address) and then trust it again, it can also be a result of an already paried device in the case that you have done it before)

connect (Bluetooth MAC address)

(connect might sometimes throw errors, just ignore it, for some reason it works anyway, just move on)

exit (or quit)

sdptool browse (Bluetooth MAC address)

Identify the channel that the GPS data is coming from/on in the output from the above command, most likely something tagged with "Serial Port"

rfcomm bind hci0 (Bluetooth MAC address) 8

(where: hci0=the BT device, the MAC address is the MAC address of the GPS device, 8=channel)

the command above will create a serial interface on the Owl

Execute rfcomm without parameters and it will output the created device in /dev, such as rfcomm0

Run ls /dev to see the new device rfcomm0

Now it would be possible to just cat the output of /dev/rfcomm0 but most likely it will not show anything, this is due to the fact that you need to specify the port speed in bps (bits per second, not baud as many say in a very wrong fashion). The bps setting depends on the device/app used, it can be 38400, 115200 or something else.

Run

stty -F /dev/rfcomm0 115200

Try to get GPS data output again (abort with Ctrl + C)

cat /dev/rfcomm0

If everything works, you can now continue with a working GPS connected and providing GPS data to the Owl

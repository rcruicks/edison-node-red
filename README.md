# edison-node-red
configure edison for Grove kit, and support for IBM Watson IOT and Cognitive services as well as TI Sensortag

**The following steps assume root login**

Create a repository pointer to current packages:

`echo "src mraa-upm https://iotdk.intel.com/repos/3.5/intelgalactic/opkg/i586" >> /etc/opkg/intel-iotdk.conf`

To bring the Yocto Linux image up to date, enter the following commands: 
```
opkg update 
opkg upgrade
```
Some node packages make use of the `setcap` command - this is not normally installed in the Yocto image

`opkg install libcap-bin`

To fix a fairly common problem with the Yocto USB gadget clashing with commercial/home networks on 192.168.2.* :

`sed -i 's/192.168.2./192.168.233/' /etc/systemd/network/usb0.network`

Install [Node-RED](http://nodered.org):

`npm install -g --unsafe-perm node-red`

Once installed, start Node-RED, waiting for the **Started flows** message, and then stop (using Ctrl-C): 

`node-red`

From the local Node-RED installation directory (*/home/root/.node-red*) install helper packages:
```
cd ./node-red
npm install --unsafe-perm node-red-node-serialport
npm install --unsafe-perm node-red-node-intel-gpio
npm install --unsafe-perm node-red-contrib-scx-ibmiotapp
npm install --unsafe-perm node-red-node-watson
npm install --unsafe-perm node-red-contrib-grove-edison
```

If there is a need to link up with a Texas Instruments [CC2650 SensorTag](http://www.ti.com/ww/en/wireless_connectivity/sensortag/), install the Node-RED pakage, and configure BLE support:
```
npm install --unsafe-perm node-red-node-sensortag
rfkill unblock bluetooth
hciconfig hci0 up
```

To enable BLE at startup, create a service to enable BLE after Bluetooth support has started after boot; this requires a script and a *systemd* service definition:
```
cat <<EOF > /home/root/enable-ble.sh
#!/bin/bash
  rfkill unblock bluetooth
  hciconfig hci0 up 
EOF

chmod u+x /home/root/enable-ble.sh

cat <<EOF > /etc/systemd/system/enable-ble.service
[Unit]
Description = Enable BLE at startup
After = network.service
[Service]
ExecStart=/bin/bash /home/root/enable-ble.sh
[Install]
WantedBy = multi-user.target
EOF

systemctl enable enable-ble.service
```

## Autostart Node-RED when Edison boots
To ensure Node_RED start automatically at boot, install and configure [PM2](http://pm2.keymetrics.io/)
```
npm install -g --unsafe-perm pm2
pm2 start /usr/bin/node-red --node-args="--max-old-space-size=256" -- -v
pm2 save
pm2 startup systemd
```
Reboot, and Node-RED should be available at `http://`*hostname*`.local:1880`

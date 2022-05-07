# Physical Testbed for MITOSYS
## Required Devices
* Samsung SmartThing Hub (model: STH-ETH-250)
* Samsung SmartThings Outlet (ZigBee)
* Yale Assure Lock YRD226 (Z-Wave)
* Nortek S&C HUSBZB-1 dongle (Works as Zigbee & Z-Wave Coordinator)
* Raspberry Pi
* Smartphone

## Required Software
* MITOSYS ([https://github.com/hammadmazhar1/mosquitto](https://github.com/stjohnjohnson/smartthings-mqtt-bridge))
* SmartThings MQTT Bridge ([https://github.com/stjohnjohnson/smartthings-mqtt-bridge](https://github.com/stjohnjohnson/smartthings-mqtt-bridge))
* openHAB

## Configure Proxy (MITOSYS) Side
1. Install Raspberry Pi OS on the Raspberry Pi.
2. Download and compile MITOSYS on the Raspberry Pi.
3. Install openHAB on the Raspberry Pi.
4. Copy files in `openhab/things`, `openhab/items`, and `openhab/rules` to the corresponding openHAB configuration folder (`/etc/openhab`) on the Raspberry Pi. The configuration should be loaded automatically, and you will see Things, Items, and Rules on the openHAB portal. If not, please use `sudo systemctl restart openhab` to restart openHAB.
5. Copy files in `mosquitto` to the mosquitto folder on the Raspberry Pi.
6. Plug in the Nortek dongle to the Raspberry Pi's USB 2.0 port (USB 3.0 port may not work).
7. Open the openHAB management portal, install ZigBee Binding in Things.
8. Then in the ZigBee Binding, add Ember EM35x Coordinator manually. The related configuration can be found at [https://www.openhab.org/addons/bindings/zigbee/](https://www.openhab.org/addons/bindings/zigbee/).
9. Similarly, install Z-Wave Binding, then add Z-Wave Serial Controller manually. The related configuration can be found at [https://community.openhab.org/t/how-to-configure-the-nortek-husbzb-1-zigbee-zwave-radio-on-oh-3/112118/4](https://community.openhab.org/t/how-to-configure-the-nortek-husbzb-1-zigbee-zwave-radio-on-oh-3/112118/4).
10. Pair SmarThings outlet with openHAB. When the outlet is in the pairing mode, click the Scan button in the ZigBee Binding, the device should be discovered automatically.
11. Click the SmartThings outlet we just added in the Things tab, then click Channels, link the Switch and Binary Input channels to the Outlet item, choose Default as Profile for both.
12. Pair Yale lock with openHAB. When the lock is in the pairing mode, click the Scan button in the Z-Wave Binding, the lock should be discorered automatically.
13. Click the Yale lock we just added in the Things tab. We first link the Door Lock channel to the Door Lock item with Default Profile. Then we link the Alarm (raw) channel, but this time choose "Create a new Item" for Item and use DoorLock\_Alarm\_Raw for the Name to match the name in the `.rules` file (check `/etc/openhab/rules/phy_st.rules` for detail). For the Profile, choose Default.
14. Launch MITOSYS with the physical testbed configuration file: `./mosquitto -c phy_st_mosq.conf`.

## Configure Automation (SmartThings) Side
1. Clone the SmartThings MQTT Bridge repo, and replace the `server.js` file with our `smartthings-mqtt-bridge/server.js` file.
2. In the SmartThings MQTT Bridge with our modified `server.js` file. When using Docker, simply create an image using `docker build -t mqtt-bridge .` command in the `smartthings-mqtt-bridge` code folder.
3. Launch the SmartThings MQTT Bridge using our configuration file `mqtt-bridge/config.yml` (replace `host` configuration with the MITOSYS Raspberry Pi's IP address). When using docker, you can place the file at `/opt/mqtt-bridge/config.yml` then execute `sudo docker run -d --name="mqtt-bridge" -v /opt/mqtt-bridge:/config -p 8080:8080 mqtt-bridge` command to start a container.
4. Download SmartThings App on your smartphone and set up the SmartThings hub.
5. Configure the MQTT Bridge follow the Usage section on [https://github.com/stjohnjohnson/smartthings-mqtt-bridge](https://github.com/stjohnjohnson/smartthings-mqtt-bridge) to configure the MQTT Bridge on SmartThings. 
6. In addtion we need to create three simulated devices in the [My Devices IDE](https://graph.api.smartthings.com/device/list) by clicking the "New Device" button on the IDE.
	* A device with following configuration for the "away from home" mode:
		* Name: Away
		* Label: Away
		* Type: Simulated Switch
		* Version: Published
		* Hub: (choose your SmartThings hub)

	* A device with following configuration for the outlet:
		* Name: Outlet
		* Label: Outlet
		* Type: Simulated Switch
		* Version: Published
		* Hub: (choose your SmartThings hub)

	* A device with following configuration for the door lock:
		* Name: DoorLock
		* Label: DoorLock
		* Type: Simulated Lock
		* Version: Published
		* Hub: (choose your SmartThings hub)
7. Add those three device to the Smart App in the SmartThings APP on your phone. 

## Configure IFTTT
1. Download the IFTTT App and register.
2. Connect your SmartThings account to IFTTT.
3. Create an Applet that unlocks the DoorLock with one tap.
4. Create an Applet that turns off the Outlet with on tap.

## Experiments
1. Turn off the Away mode.
2. Turn on/off the outlet on the SmartThings App. The outlet can be turned on/off without any issue and remain its status after each action.
3. Turn on/off the outlet physically by hand. The outlet remains its status after each action.
4. Lock/unlock the door lock on the SmartThings App. The lock can be locked/unlocked without any issue and remain its status after each action.
5. Lock/unlock the door lock physically by hand. The lock can be locked/unlocked without any issue and remain its status after each action.
6. Turn on the Away mode.
7. Turn on the outlet on the SmartThings App. The outlet can be turned on without any issue and remain its its status after each action.
8. Turn on the outlet physically by hand. The outlet can be turned on without any issue and remain its its status after each action.
9. Turn off the outlet on the SmartThings App. The action is blocked by the MITOSYS and the outlet remains on.
10. Turn off the outlet physically by hand. The outlet is quickly turned back on by MITOSYS.
11. Lock the door lock on the SmartThings App. The lock can be locked without any issue and remain its status after each action.
12. Lock the door lock physically by hand. The lock can be locked without any issue and remain its status after each action.
13. Unlock the door lock on the SmartThings App. The action is blocked by the MITOSYS and the door lock remains locked.
14. Unlock the door lock physically by hand. The lock is quickly locked by MITOSYS.
15. Turn off the Away mode.
16. Turn off the outlet using IFTTT. The outlet can be turned off without any issue and remain its status after each action.
17. Unlock the door lock using IFTTT. The door can be unlocked without any issue and remain its status after each action.
18. Turn on the outlet and lock the door lock.
19. Turn on the Away mode.
20. Turn off the outlet using IFTTT. The action is blocked by the MITOSYS and the outlet remains on.
21. Unlock the door lock using IFTTT. The action is blocked by the MITOSYS and the door lock remains locked.

# iRobot create3-ros2-sensors-upgrade
Add Raspberry Pi to create3, plus Lidar and 3D cameras

## Installation

### Setup Create 3

1. Setup the Create 3 (power, update firmware, setup wifi)

Follow the instructions at [Create 3 setup](https://edu.irobot.com/create3-setup) to setup the power, charge the Create 3, update the firmware (very important!), setup WiFi.

Tl;dr press both outer buttons on robot and hold until lights flash, connect a computer to the Create 3 wifi hotspot, and connect your browser to 192.168.10.1. Download latest firmware from [here](https://edu.irobot.com/create3-latest-fw) and in the browser select update and upload the firmware.  Once updated, reconnect wifi and in the browser select `application` then `configuration` and enter a namespace such as `create3-1` for ROS (otherwise any other robots you have will conflict) , select fastrtps for ROS2 humble or cyclone for ROS2 Galactic and click save.  Select `connect` and give the robot a host name, update, then enter wifi connection details and update that.

If the above doesn't make sense then do follow the link above step by step in the order I have indicated.

1. Setup the Raspberry Pi for booting Ubuntu via USB

Follow the instructions at [Boot Raspberry Pi from USB](https://www.tomshardware.com/how-to/boot-raspberry-pi-4-usb)

Tl;dr use the Raspberry Pi imager to burn a micro SSD with the `Misc utility images` `Bootloader` `USB Boot` image.  Insert into RPi and power up.  Leave running for > 10 secs, make sure green light flashing regularly, then power off.  Again use RPi imager and select `Other general purpose OS`, `Ubuntu`, and then either `Ubuntu Server 22.04 64 bit` or `Ubuntu Server 20.04 64 bit`.  *Remember to click the options gear* and turn on ssh, setup wifi, give it a hostname, setup locale and burn that to the USB.  When done insert into the top USB 3 slot (USB 2 are on left, Ethernet on Right, USB 3 in middle)

1. Setup ROS2 (Humble for 22.04, Galactic for 20.04), for this purpose I am assuming [Humble install](https://iroboteducation.github.io/create3_docs/setup/ubuntu2204/) or follow [Setup Galactic Here](https://iroboteducation.github.io/create3_docs/setup/ubuntu2004/)

Ssh to the RPi or connect to screen and keyboard.  Next update the RPi and set it up.

```
sudo apt update
sudo apt upgrade
# if a new kernel is installed, reboot the RPi before continuing
sudo apt install -y net-tools mlocate build-essential zip
echo $LANG # If utf-8 is not displayed or local is wrong setup locale
sudo locale-gen en_GB en_GB.UTF-8  # or en_US etc
sudo update-locale LC_ALL=en_GB.UTF-8 LANG=en_GB.UTF-8 # or en_US etc
export LANG=en_GB.UTF-8 # or en_US etc

# If on terminal and you cannot find your ip address run ifconfig and get inet from the wlan0 output

# Required if using high current usb port from UUGear
wget https://www.uugear.com/repo/MEGA4/install.sh
sudo sh install.sh
rm install.sh
# check it's working if required
# cd mega4
# sudo ./mega4.sh # turn off usb ports and manage card - this is not necessary for it to operate once the drivers are installed


# Setup Humble
sudo apt update && sudo apt install software-properties-common
sudo add-apt-repository universe
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) \ 
signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] \ 
http://packages.ros.org/ros2/ubuntu \
$(source /etc/os-release && echo $UBUNTU_CODENAME) \
main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
sudo apt update && sudo apt upgrade
sudo apt install -y ros-humble-ros-base # or sudo apt install -y ros-humble-ros-perception to add some ros2 opencv and other perception utilities
sudo apt install -y ros-humble-irobot-create-msgs # This is how we talk to the extra functionality of the Create 3
sudo apt install -y python3-colcon-common-extensions # Optionally you can add python3-rosdep ros-humble-rmw-cyclonedds-cpp
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
echo "export RMW_IMPLEMENTATION=rmw_fastrtps_cpp" >> ~/.bashrc
source ~/.bashrc
```

1. Mount the Raspberry Pi, setup battery back power supply, and wire to the Create 3

The high current of some sensors suggest a powered USB Hub

- Remove cover from Create 3
- Switch aux board to USB mode
- Mount a Raspberry Pi base onto the Create 3. Note the RPi should be towards the back to leave room for camera and lidar. Note link to STL files for 3d Printed version #TODO to go here
   - If following the high current USB route, mount the "Mega 4 USB 3 Hub" onto the base using M2.5 spacers and nuts (leaving a screw hole in spacer top) with the USB input facing right.  I bought mine from PiHut.
   - If not following the high current USB route install a non-powered USB 3 hub
- Mount the Raspberry Pi 4 using M2.5 spacers over the USB hub below, and connect to USB hub using the provided USB U link
- Mount a Raspberry Pi low profile (if possible) cooling system on top and connect fan to RPi
- Connect USB Hub output port via USB-C cable to the Create 3 Aux card USB-C
- Connect the Create 3 Aux unregulated power wire to the 7-22 VDC regulated power supply #TODO insert normally on relay between to enable power disconnect when undocked
- Connect the power supply to the Battery Bank micro-usb power input
- Check everything is ready and the Create 3 is powered and put back on the doc
- Close cover (rerouting any wires if necessary)
- Connect the Battery Bank USB outputs to the Raspberry Pi and (optional USB Hub USB-C) power input

The Raspberry Pi should now boot, connect to WiFi, and connect to the Create 3

1. First contact with Create 3

ssh to the raspberry pi and run a few simple tests

`ros2 topic list`

You should see something like the below. 

```
/parameter_events
/rosout
blm@create3-pi-usb:~$ ros2 topic list
/create3_1/battery_state
/create3_1/cmd_audio
/create3_1/cmd_lightring
/create3_1/cmd_vel
/create3_1/dock
/create3_1/hazard_detection
/create3_1/imu
/create3_1/interface_buttons
/create3_1/ir_intensity
/create3_1/ir_opcode
/create3_1/kidnap_status
/create3_1/mobility_monitor/transition_event
/create3_1/mouse
/create3_1/odom
/create3_1/robot_state/transition_event
/create3_1/slip_status
/create3_1/static_transform/transition_event
/create3_1/stop_status
/create3_1/tf
/create3_1/tf_static
/create3_1/wheel_status
/create3_1/wheel_ticks
/create3_1/wheel_vels
/parameter_events
/rosout
```

Note the first part of the topics list from the Create 3 should match whatever namespace you selected on the Create 3 application configuration page.

If you get a ros2 not found error message you need to `source /opt/ros/humble/setup.bash`. If you get only /parameter_events and /rosout then connection with the Create 3 is broken.  Use a browser to the Create 3 IP address and select `Application` `Restart Application`   If it stays like that after the reboot then check your USB hub to Create 3 USB-C cable and connection.  Failing the above, go back through the installation and look for missing steps or error messages.

1. Check ROS2 Create 3 commands

Note do not perform any movement or undock command if the Raspberry Pi is not mounted on the Create 3 and has no cables to a computer or other device away from the Create 3.

`ros2 topic echo /namespace/battery_state` where namespace is the namespace you saw on the topic list command above

## Todo

- Create image of wiring
- Add 3d camera
- Add lidar
- add 3d Print STL of camera and lidar holder
- setup networking to humble host with RBIZ and/or other support tools


# Parts list - note no tags nor affiliate links

- Raspberry Pi 4B (2gb will work, 4gb and 8gb are better, good luck finding them!)
- [XTVTX 7-22 VDC unregulated power to USB power supply](https://www.amazon.co.uk/gp/product/B09K7KHX2C)
- [Mega 4 USB 3 Hub ](https://thepihut.com/collections/uugear/products/mega4-4-port-usb-3-1-ppps-hub-for-raspberry-pi-4)
- [Raspberry Pi low profile cooler ](https://www.amazon.co.uk/gp/product/B0B7QXW3PP)
- [2 USB port 10000 mah Battery Bank](https://www.amazon.co.uk/gp/product/B07TT2VGT6)
- M2.5 spacers, nuts & screws
- Raspberry Pi 3d Printed Base [Download here](/stl/create3%20raspi%20try4%20v4.stl)


## Authors

- [@brianlmerritt](https://github.com/brianlmerritt)

## Reference Material

## Acknowledgements

- [Create3 Documentation and Team](https://iroboteducation.github.io/create3_docs/)
- [Offical ROS2 Tutorials](https://docs.ros.org/en/galactic/Tutorials.html)
- [Boot Raspberry Pi from USB](https://www.tomshardware.com/how-to/boot-raspberry-pi-4-usb)



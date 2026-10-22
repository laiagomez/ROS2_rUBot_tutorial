# **Install micro-ROS on ESP32**

Micro-ROS is an implementation of the Robot Operating System 2 (ROS 2) designed for microcontrollers like the ESP32. 

It enables you to integrate ROS 2 capabilities into resource-constrained devices. 

**Documentation**
- https://github.com/micro-ROS/micro_ros_setup
- https://github.com/micro-ROS/micro_ros_espidf_component
- https://micro.ros.org/docs/tutorials/core/overview/
- Installationvideo: https://www.youtube.com/watch?v=t-ovNiyWbDg

- https://cps.unileoben.ac.at/tag/info/


Here's a general outline of how you can get started with installing Micro-ROS on an ESP32:
## **1. Install micro-ROS**

First of all you need to get acces to COM ports if you are using a docker. 

By using --net=host, -v /dev:/dev, and --privileged, you're granting the Docker container extensive access to your host's system resources, including network and devices.

```shell
docker run --name ROS2_Humble_ports --net=host -v /dev:/dev --privileged -e DISPLAY=host.docker.internal:0.0 --mount src="C:\Users\puigm\Desktop\ROS_github\myPC_shared",dst=/home/myDocker_shared,type=bind -it osrf/ros:humble-desktop-full
```
```shell
docker run --name ROS2_Humble_ports2 --net=host -v /dev:/dev -p "COM3":"ttyUSB3" --privileged -e DISPLAY=host.docker.internal:0.0 --mount src="C:\Users\puigm\Desktop\ROS_github\myPC_shared",dst=/home/myDocker_shared,type=bind -it osrf/ros:humble-desktop-full
```

Follow instructions on: 
- https://micro.ros.org/docs/tutorials/core/first_application_rtos/freertos/
- Video tutorial: https://www.youtube.com/watch?v=CLb9mmJQmrw


Some tricks:
- After colcon build if you have already created /firmware, delete it first:
```shell
sudo rm -rf firmware/
```
- create firmware for ESP32 (this takes a few minutes):
```shell
ros2 run micro_ros_setup create_firmware_ws.sh freertos esp32
```
- Our applications are located in firmware/freertos_apps/apps. This is where we have to save a new application. We will execute the "ping_pong" application.
- Configure the firmware for our application:
```shell
ros2 run micro_ros_setup configure_firmware.sh ping_pong --transport serial
```
- Build the firmware to be able to later upload it to the ESP32
```shell
ros2 run micro_ros_setup build_firmware.sh
```
- Flashing the firmware:
```shell
ros2 run micro_ros_setup flash_firmware.sh
´´´
> Here the problem is to access to the ports within Docker

**1.1. Install ESPIDF tool:**
Follow instructions in: https://github.com/micro-ROS/micro_ros_espidf_component

Clone the ESP-IDF program
```shell
cd /home
git clone --recursive https://github.com/espressif/esp-idf.git
```

Clone the micro_ros_espidf_component repository within your ESP-IDF project directory. This repository contains the Micro-ROS component specifically designed for the ESP-IDF framework. 
```shell
cd esp-idf
git clone -b $ROS_DISTRO https://github.com/micro-ROS/micro_ros_espidf_component.git
```

Make installation:
```shell
./install.sh
```

Set Up the Environment Variables:
```shell
source export.sh
```
Compile a speciffic program code (int32_publisher)
```shell
cd /home/esp-idf/micro_ros_espidf_component/examples/int32_publisher
idf.py menuconfig
```

Compile a speciffic program code (Hello world)
```shell
cd examples/get-started/hello_world
idf.py set-target esp32
```

install additional tools such as the ESP-IDF Extensions for Visual Studio Code

**1.2. Install microros_ws and necessary packages**

```shell
source /opt/ros/humble/setup.bash
cd /home
mkdir microros_ws
cd microros_ws/
git clone -b $ROS_DISTRO https://github.com/micro-ROS/micro_ros_setup.git src/micro_ros_setup
sudo apt update
rosdep update
rosdep install --from-paths src --ignore-src -y
sudo apt install python3-pip
colcon build
source install/local_setup.bash
```

**2. Creating a new firmware workspace for ESP32**
```shell
ros2 run micro_ros_setup create_firmware_ws.sh freertos esp32
```
After some time to compile, identify the local machine IP
```shell
cd microros_ws
hostname -I
```
Note the IP address delivered: 172.17.0.2 to communicate ESP32 to local machine


```shell
ros2 run micro_ros_setup configure_firmware.sh int32_publisher -t udp -i 172.17.0.2 -p 8888
```

**3. Set your wifi credentials for ESP32**

```shell
ros2 run micro_ros_setup build_firmware.sh menuconfig
```
Go to: micro-ROS Transport Settings --> WiFi Configuration, and fill your WiFi SSID and password

Press "S" to save and "Q" to quit

**4. Build firmware**
```shell
ros2 run micro_ros_setup build_firmware.sh
```
Connect your ESP32 to the computer PC with a micro-USB cable

```shell
ls /dev/ttyUSB*
```
I should appear the access to /dev/ttyUSB0

The run:
```shell
ros2 run micro_ros_setup flash_firmware.sh
```

**5. Creating the micro-ROS agent**

Run
```shell
ros2 run micro_ros_setup create_agent_ws.sh
ros2 run micro_ros_setup build_agent.sh
source install/local_setup.bash
```

Run a micro-ROS agent
```shell
ros2 run micro_ros_agent micro_ros_agent udp4 --port 8888
```

Now press the reset button and open a new terminal. You can see in the topic list: /freertos_int32_publisher

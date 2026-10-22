# **Migration to ROS2**

ROS1 is close to finish and you can switch to ROS2
![](./Images/7_ROS2_time.png)
ROS2 structure is based on the architecture:
![](./Images/7_ROS1_ROS2.png)
The main differences are:
![](./Images/7_ROS2_dif.png)

ROS2 is a very good choice.

Interesting references for courses:
- Edouard Renard: https://www.udemy.com/course/ros2-for-beginners/learn/lecture/20260476#overview
- https://www.udemy.com/course/learn-ros2-as-a-ros1-developer-and-migrate-your-ros-projects/learn/lecture/22003074#overview
- https://www.udemy.com/course/ros2-ultimate-mobile-robotics-course-for-beginners-opencv/learn/lecture/28143024#overview
- https://www.udemy.com/course/ros2-self-driving-car-with-deep-learning-and-computer-vision/learn/lecture/28236852#overview

Some interesting projects:
- https://github.com/noshluk2/ROS2-Raspberry-PI-Intelligent-Vision-Robot
- https://github.com/noshluk2/ROS2-Ultimate-Mobile-Robotics-Course-for-Beginners-OpenCV
- https://github.com/noshluk2/ROS2-Self-Driving-Car-AI-using-OpenCV

And the ROS2 reference sites:
- https://docs.ros.org/
- https://docs.ros.org/en/foxy/

## 1. **ROS2 Installation & Tools**
Installation could be in:
- Ubuntu 20
- windows

This installation could be made using Docker tool
## **a) ROS2 in Ubuntu20**
You could have Ubuntu installed:
- In a computer with Ubuntu 20
- Using VirtualBox to create a virtual machine
- with Windows Ubuntu terminal: downloaded from Microsoft Store 

In this last case you will need to:
- Update the windows and all drivers. Restart 2-3 times
- Download Ubuntu 20.04 from Microsoft Store. Run as administrator. Specify user and password
- Verify if WSL2 is installed and working with wsl2 version (open Power Shell)
    ```shell
    wsl -l -v
    wsl --set-version Ubuntu-20.04 2
    ```
-  update Ubuntu 20 (open Ubuntu terminal):
    ```shell
    sudo apt update
    sudo apt upgrade
    sudo apt install python3-pip
    python3 -m pip install -U pydot PyQt5
    sudo apt install gedit
    ```
- Install VcXsrv Windows X Server to manage emerging windows: https://sourceforge.net/projects/vcxsrv/

- Define the DISPLAY env variable:
    ```shell
    export DISPLAY=:0.0
    xhost +
    ```

- To properly install ROS2, follow instructions in:
https://docs.ros.org/en/foxy/Installation/Ubuntu-Install-Debians.html

Some other installations needed:
- python3 pip
    ```shell
    sudo apt install python3-pip
    pip3 install argcomplete
    ```
    > needed for autoformating (press crtl+shift+i)
- Colcon compilation utility (https://docs.ros.org/en/foxy/Tutorials/Colcon-Tutorial.html)
    ```shell
    sudo apt install python3-colcon-common-extensions
    ```
    > To use autocompletion you need to add in ~/.bashrc file this line:
    >- source /opt/ros/foxy/setup.bash
    >- source /usr/share/colcon_argcomplete/hook/colcon-argcomplete.bash
    >
    >type:
    ```shell
    echo "source /opt/ros/foxy/setup.bash" >> ~/.bashrc
    echo "source /usr/share/colcon_argcomplete/hook/colcon-argcomplete.bash" >> ~/.bashrc
    ```

- In Visual Studio Code add some extensions:
    - python 
    - python for VS Code
    - python Intellisense
    - XML (Red-Hat)
    - XML Tools
    - Code runner
    - ROS
    - C/C++
    - cmake

- Install Terminator (https://cheatography.com/svschannak/cheat-sheets/terminator-ubuntu/):
    ```shell
    sudo apt install terminator
    ```
- Finally you have to reboot:
    ```shell
    sudo reboot
    ```

## **b) ROS2 in windows**
Follow instructions in:
http://wiki.ros.org/Installation/Windows

Complete the installation dependencies:
```shell
python -m pip install -U pydot PyQt5
choco install graphviz
```
## **c) ROS2 in windows using Docker tool**
First you need to install Docker:
- https://docs.docker.com/get-docker/

In windows:
- https://docs.docker.com/desktop/install/windows-install/

    >   Open PowerShell terminal and type: systeminfo | find "Tipo de sistema"
    >
    >   The output has to be: x64-based PC

- Then you have to complete your installation with WSL 2 for kernell update in: https://learn.microsoft.com/ca-es/windows/wsl/install-manual#step-4---download-the-linux-kernel-update-package
- Stablish WSL2 as default by opening a powershell and typing: wsl --set-default-version 2
- restart your computer

Then you can download a container with ROS-Noetic-Desktop-full:
- In search item on top menu, type: ROS-Noetic-Desktop-full

- Install VcXsrv Windows X Server to manage emerging windows: https://sourceforge.net/projects/vcxsrv/

- Define the DISPLAY env variable with the IP address of the vEthernet (WSL) adapter (optained in a power Shell with ipconfig):
    ```shell
    export DISPLAY=:0.0
    xhost +172.20.48.1
    ```

## **d) USB Image tool**
This software will be used to create an image of SD card to share and copy to another SD card.
- Download the SW from: 
https://www.alexpage.de/usb-image-tool/download/


## 2. **Create workspace**

You can create a workspace with your desired name (usually finished with ws), for exemple "ROS2_rUBot_ws". Add a subfolder "src" where you will place the packages.

Proper documentation in: https://docs.ros.org/en/foxy/Tutorials/Beginner-CLI-Tools/Configuring-ROS2-Environment.html#

**a) In Ubuntu:**

To work with ROS2 first open your ~/.bashrc and be sure you have sourced ROS2 adding the lines:
- source /opt/ros/foxy/setup.bash
- source /usr/share/colcon_argcomplete/hook/colcon-argcomplete.bash

**b) In Windows:**

In command line add "&& c:\opt\ros\foxy\x64\setup.bat"

## 3. **Create first package**
You can create your first package inside the src folder with a name "ros2_tutorial"
```shell
ros2 pkg create --build-type ament_python ros2_tutorial
cd ..
colcon build --merge-install
```
> If colcon is not installed:
> - In Ubuntu: sudo apt install python3-colcon-common-extensions
> - In windows: pip install -U colcon-common-extensions

Source the ws:
- in Ubuntu: "source ~/Desktop/ros2_rUBot_ws/devel/setup.bash"

- In Windows add in command line: "&& install\local_setup.bat"

Create your first Package: https://docs.ros.org/en/foxy/Tutorials/Creating-Your-First-ROS2-Package.html

## 4. **Create first Publisher and Subscriber nodes**
You can create your first Publisher and Subscriber using some templates.
- Create files "publisher_hello,py" and "subscriber_hello.py"
- Add entry points for Publisher and Subscriber
    
    Reopen setup.py and add the entry point for the subscriber node below the publisher’s entry point. The entry_points field should now look like this:
    ```python
    entry_points={
        'console_scripts': [
                'publisher_node = ros2_tutorial.publisher_hello:main',
                'subscriber_node = ros2_tutorial.subscriber_hello:main',
        ],
    },
    ```
- Compile inside ws: 
    ```shell
    colcon build --merge-install
    ```
- Run:
    ```shell
    ros2 run ros2_tutorial publisher_node
    ros2 run ros2_tutorial subscriber_node
    ```


Create your first Publisher and Subscriber nodes: https://docs.ros.org/en/foxy/Tutorials/Beginner-Client-Libraries/Writing-A-Simple-Py-Publisher-And-Subscriber.html

## 5. **Using parameters in a Class**

The detailed tutorial can be found in: https://docs.ros.org/en/foxy/Tutorials/Beginner-Client-Libraries/Using-Parameters-In-A-Class-Python.html

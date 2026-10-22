# **ROS2 tutorial**

Installation will be made using Docker

Bibliography:
- Edouard Renard: https://www.udemy.com/course/ros2-for-beginners/learn/lecture/21305270#overview


## **1. ROS2 workspace and packages**

The **ROS2 workspace** is the directory in your hard disk where your ROS2 packages reside to be usable by ROS2.

ROS2 uses **packages** to organize its programs. You can think of a package as all the files that a specific ROS2 program contains

In ROS2, you can create two types of packages: Python packages and CMake (C++) packages.

Every **Python package** will have the following structure of files and folders:

- package.xml - File containing meta-information about the package (maintainer of the package, dependencies, etc.).

- setup.py - File containing instructions for how to compile the package.

- setup.cfg - This is a file that defines where the scripts will be installed.

- /<package_name> - This directory will always have the same name as your package. You will put all your Python scripts inside this folder. Note that it already contains an empty __init__.py file.

Some packages might contain extra folders. For instance, the launch folder contains the package's launch files 

We wil use the "ROS2_rUBot_tutorial_ws" as starting workspace for this project.

You open a terminal in this Container using VS Code:
```shell
cd /home
git clone https://github.com/manelpuig/ROS2_rUBot_tutorial_ws
```
**Create a new package**

Every time you want to create a package, you have to be in this directory src. Type into your Webshell the following command:
```shell
ros2 pkg create --build-type ament_python my_package --dependencies rclpy
For exemple
ros2 pkg create --build-type ament_python ros2_tutorial --dependencies rclpy
```
>the <package_dependencies> are the names of other ROS2 packages that your package depends on.

Now you have to build the created ws:
```shell
cd /home/ROS2_rUBot_tutorial_ws
colcon build
```
In ROS2 Humble some errors/warnings arise:
- warnings related to "SetuptoolsDeprecationWarning: setup.py install is deprecated". Install setup tools version 58.2.0 (last version to work with ros2 python packages without any warnings)

We have to downgrade the setuptools version to 58.2.0:
```shell
sudo apt install python3-pip
pip3 list | grep setuptools
pip3 install setuptools==58.2.0
```
- If warnings related to CMAKE_PREFIX_PATH, AMENT_PREFIX_PATH environment variables non existing values, reset them with:
   - export AMENT_PREFIX_PATH=""
   - export CMAKE_PREFIX_PATH=""

Source the workspace. Be sure in .bashrc (with docker is from root) file to have:
```shell
source /opt/ros/humble/setup.bash 
source /usr/share/colcon_argcomplete/hook/colcon-argcomplete.bash
cd /home/ROS2_rUBot_tutorial_ws
source /home/ROS2_rUBot_tutorial_ws/install/setup.bash
```
You are ready to work with this workspace

Some interesting commands:
- ros2 pkg list: Gives you a list of all your ROS system packages.

- ros2 pkg list | grep my_package: Filters, from all of the packages located in the ROS system, the package is named my_package.

**Compile a Package**

Sometimes (for large projects), you will not want to compile all of your packages. This would take such a long time. So instead, you can use the following command to compile only the packages where you have made changes:
```shell
colcon build --packages-select <package_name>
```
You can compile a speciffic package and launch not the executable but the python file in src (we do not need to compile every time we make a change in the python file!):
```shell
colcon build --packages-select ros2_tutorial --symlink-install
```

## **2. ROS2 Nodes**
In ROS2, each node should be responsible for a single module (e.g., one node for controlling wheel motors, one for controlling a LIDAR control, etc.). Each node can communicate with other nodes through different methods.

A full robotic system is comprised of many nodes working together. In ROS2, a single executable (a C++ or Python program, etc.) can contain one node.
ROS2 nodes are:
- Subprograms in your application, responsible for only one thing
- communicate with each other through topics, services and parameters

**Create first ROS2 Program**

You can create a program based on two nodes:
- A Publisher node that publish a string message "Hello world" to a "/pub_topic" topic
- A subscriber node that subscribes to "/pub_topic" and prints on the screen the read message.

Each node is created within a python program in "src/ros2_tutorial/ros2_tutorial" folder:
- "publisher_hello.py": creates a simple_publisher node
- "subscriber_hello.py": creates a simple_subscriber node

You can create Publisher and Subscriber python files using speciffic templates.

Next step is to generate an executable from the python scripts you created:
- In **setup.cfg**: is already specified where do you install the node (install/ros2_tutorial/lib/ros2_tutorial)
- In **setup.py**: contains all the necessary instructions for properly compiling your package. To do that, you work with a dictionary named entry_points. 
    - Inside it, you find an array called console_scripts.

    - Add entry points for Publisher and Subscriber
    
        Open setup.py and add the entry point for the publisher and subscriber nodes. The entry_points field should now look like this:
        ```python
        entry_points={
            'console_scripts': [
                    'publisher_node = ros2_tutorial.publisher_hello:main',
                    'subscriber_node = ros2_tutorial.subscriber_hello:main',
            ],
        },
        ```
        You can see these lines as follows:    '<executable_name> = <package_name>.<script_name>:main'

- Compile inside ws: 
    ```shell
    colcon build
    ```
- Run in a separate terminals:
    ```shell
    ros2 run ros2_tutorial publisher_node
    ros2 run ros2_tutorial subscriber_node
    ```
- You can also run the nodes from the folder where the nodes are installed
    ```shell
    cd install/ros2_tutorial/lib/ros2_tutorial
    ./publisher_node
    ./subscriber_node
    ```
Take into account that, the name of:
- the python file (publisher_hello.py)
- the node (simple_publisher)
- the executable (publisher_node)

Could be different!!

**Debug and monitor nodes**

You have different functions to debug and monitor:
- ros2 node -h
- ros2 node list
- ros2 node info /node

You can run a node with a different name:
- ros2 run pkg node --ros-args --remap __node:=abc

You can compile a speciffic package and launch not the executable but the python file in src (we do not need to compile every time we make a change in the python file!):
- colcon build --packages-select ros2_tutorial --symlink-install

Rqt and rqt_graph tools

You can practice with Turtlesim_node
```shell
ros2 run turtlesim turtlesim_node
ros2 run turtlesim turtle_teleop_key
ros2 run turtlesim turtlesim_node --ros-args -r __node:=my_turtle
```

## **3. Create Launch files**
 Usually we create a "robot_bringup" package to locate the launch files. These launch files can take nodes from all other packages. We will create this package in the next section when we will work in a real robot project. For this first tutorial, we will use the same package to add some launch files.
 
ROS2 programs can be run:
- from executable files
- from launch files. 

When using executable files, we need one terminal to run each node. 

To make easier this process using a unique terminal to launch multiple nodes, we will use launch files.

The launch system in ROS 2 is responsible for helping the user describe the configuration of their system and then execute it as described. Launch files written in Python, XML, or YAML can start and stop different nodes as well as trigger and act on various events. 

To create a launch file we have to:
- create a "launch" folder in "src/ros2_tutorial/"
- Inside create the launch file in different formats (python, XML or YAML)
- Enable the execution of all files in "launch" folder. For that you have to open "setup.py" file and make some modifications to:
    - Import some libraries
    ```shell
    import os
    from glob import glob
    ```
    - share this launch folder to the executable path. Add this line in "data_files"
    ```shell
    (os.path.join('share', package_name), glob('launch/*'))
    ```
- compile again

**Writting launch files**

Launch files can be written in python, XML or YAML formats. We show here the same lauch file in the 3 different formats.
- Launch file in Python format: "hello_pub_sub.launch.py" file
```python
from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():
    return LaunchDescription([
        Node(
            package='ros2_tutorial',
            executable='publisher_node',
            output='screen'),
        Node(
            package='ros2_tutorial',
            executable='subscriber_node',
            output='screen'),
    ])
```
Now you can execute the launch file:
```shell
ros2 launch ros2_tutorial hello_pub_sub.launch.py
```
- Launch file in XML format: "hello_pub_sub.xml" file
```xml
<launch>
<node pkg="ros2_tutorial" exec="publisher_node"/>
<node pkg="ros2_tutorial" exec="subscriber_node"/>
</launch>
```
See documentation: https://docs.ros.org/en/humble/How-To-Guides/Launch-files-migration-guide.html

Now you can execute the launch file:
```shell
ros2 launch ros2_tutorial hello_pub_sub.xml
```
- Launch file in YAML format: "hello_pub_sub.yaml" file
```yaml
launch:

- node:
    pkg: "ros2_tutorial"
    exec: "publisher_node"
    name: "simple_publisher"
- node:
    pkg: "ros2_tutorial"
    exec: "subscriber_node"
    name: "simple_subscriber"
```
Now you can execute the launch file:
```shell
ros2 launch ros2_tutorial hello_pub_sub.yaml
```

**Exercise:**

Make a ROS2 program based on 2 nodes:
- A Publisher node that publish a INT32 number to a "/pub_topic" topic
- A subscriber node that subscribes to "/pub_topic", multiplies this number by 2 and prints on the screen the resulting value.

Construc the corresponding launch file to execute the program


## **4.Client Libraries**
ROS client libraries allow nodes written in various programming languages to communicate. A core ROS client library (RCL) implements the standard functionality needed by various ROS APIs. This makes it easier to write language-specific client libraries.

The ROS2 team currently maintains the following client libraries:

- rclcpp = C++ client library
- rclpy = Python client library

Additionally, other client libraries have been developed by the ROS community. You can find some more information in:
https://docs.ros.org/en/humble/Concepts/About-ROS-2-Client-Libraries.html

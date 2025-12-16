# Franka Workspace (with Git Submodules)

This repository uses Git submodules to include several dependent repositories.

## How to initialize a franka project

### Install libfranka

Actually you don't need to clone the libfranka repository and build it yourself as this won't work.
Just follow the link in readme in this repository. The following steps works for system Ubuntu20.04.

First install dependencies

```bash
sudo apt-get update
sudo apt-get install -y build-essential cmake git libpoco-dev libeigen3-dev libfmt-dev
```
Then download the package and extract it and install
```bash
cd Downloads
wget https://github.com/frankarobotics/libfranka/releases/download/0.18.2/libfranka_0.18.2_focal_amd64.deb
sudo dpkg -i libfranka_0.18.2_focal_amd64.deb
```
After executing this the libfranka package should be installed on your PC. This step is neccessary because
later the franka_ros library need this package to compile.

### Build franka_ros

create a workspace franka_ws, in this folder create a franka_ros_ws, this is the ros catkin workspace used later.
Inside franka_ros_ws add a src folder and clone the forked franka_ros repository on your own github.

```bash
mkdir franka_ws
cd franka_ws
mkdir franka_ros_ws
cd franka_ros_ws
mkdir src
cd src
git clone git@github.com:prominentjohnson/franka_ros.git
```
Before building it's better to install all the required dpendencies. For examples here are the dependencies that was missing on my PC.
```bash
sudo apt install ros-noetic-pinocchio
sudo apt install ros-noetic-combined-robot-hw
sudo apt install ros-noetic-boost-sml
```
Then go back to the ros workspace and build it. 
```bash
cd ..
catkin_make
```
Trouble shooting: When installing you might encounter the following error:
```bash
/usr/include/research_interface/robot/service_types.h:242:24: error: ‘optional’ in namespace ‘std’ does not name a template type 242 | const std::optional<std::vector<double>> &maximum_velocity = std::nullopt)
/usr/include/research_interface/robot/service_types.h:242:19: note: ‘std::optional’ is only available from C++17 onwards 242 | const std::optional<std::vector<double>> &maximum_velocity = std::nullopt)
```
This is because the code was compiled using C++14, however the code is C++17 compliant. So the solution is to modify the CmakeList and force it to compile using C++17. Here's how to do it: Find the CMakeLists.txt which is responsible for the error report, then delete the following two lines
```cmake
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
```
Then add the following line after add_library or add_executable command
```cmake
target_compile_features(franka_hw PUBLIC cxx_std_17)
```
Here are two examples:
```cmake
foreach(executable ${EXECUTABLES})
    add_executable(${executable}
    src/${executable}.cpp
  )
target_compile_features(${executable} PUBLIC cxx_std_17)
```
or
```cmake
add_library(franka_hw
  src/control_mode.cpp
  src/franka_hw.cpp
  src/franka_combinable_hw.cpp
  src/franka_combined_hw.cpp
  src/resource_helpers.cpp
  src/trigger_rate.cpp
)
target_compile_features(franka_hw PUBLIC cxx_std_17)
```
Then run catkin_make again until it can build successfully.

## How to run ROS simulation

source the setup file in the ros workspace
```bash
source ~/workspaces/franka_ws/franka_ros_ws/devel/setup.bash
```
Then run the launch file to start the gazebo simulation
```bash
roslaunch franka_gazebo panda.launch x:=-0.5     world:=$(rospack find franka_gazebo)/world/stone.sdf     controller:=cartesian_impedance_example_controller     rviz:=true
```
Then open another terminal and run
```bash
roslaunch franka_gazebo panda.launch rviz:=true controller:=cartesian_impedance_example_controller
```
and you should be able to drag the arm in the rviz.


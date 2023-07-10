# ROS 2 Kortex
> Kinova® Kortex™ is the common software platform behind all of the products in the Gen3 family (Gen3 and Gen3 lite). It unifies the inner workings of the various robots and their related external tools, like the API. <br />
> https://www.kinovarobotics.com/product/gen3-robots

ROS2 Kortex is the official ROS2 package to interact with Kortex and its related products. It is built upon the Kortex API, documentation for which can be found in the [GitHub Kortex repository](https://github.com/Kinovarobotics/kortex).

## Build status


ROS2 Distro | Humble | Iron | Rolling
:---------: | :----: | :--: | :-----:
| **Branch** | [`main`](https://github.com/PickNikRobotics/ros2_kortex/tree/main) | [`main`](https://github.com/PickNikRobotics/ros2_kortex/tree/main) | [`main`](https://github.com/PickNikRobotics/ros2_kortex/tree/main)
| **Build Status** | [![Humble Binary Build](https://github.com/PickNikRobotics/ros2_kortex/actions/workflows/humble-binary-build.yml/badge.svg?branch=main)](https://github.com/PickNikRobotics/ros2_kortex/actions/workflows/humble-binary-build.yml?branch=main) | :construction: | [![Rolling Binary Build](https://github.com/PickNikRobotics/ros2_kortex/actions/workflows/rolling-binary-build.yml/badge.svg?branch=main)](https://github.com/PickNikRobotics/ros2_kortex/actions/workflows/rolling-binary-build.yml?branch=main)

**Note:** There are several CI jobs checking against future upstream changes see [detailed build status](.github/workflows/README.md) for more information.


## Getting started
<!-- TODO(moriarty) update this when binary package is released getting most users should use binary release -->

1. Install ROS 2.

   If you're a developer, we recommend using Rolling to get the latest features and fixes.

   Rolling Release: [Install ROS2 Rolling](https://docs.ros.org/en/rolling/Installation/Ubuntu-Install-Debians.html)<br/>
   Latest Release: [Install ROS2 Iron](https://docs.ros.org/en/iron/Installation/Ubuntu-Install-Debians.html)</br>
   Stable LTS Release: [Install ROS2 Humble](https://docs.ros.org/en/humble/Installation/Ubuntu-Install-Debians.html)

   After installing a version of ROS, source the setup.bash, which will set the `$ROS_DISTRO` environment variable.

2. Optional: install Cyclone DDS
   If you plan to use MoveIt, it is recommended to install and use Cyclone DDS.
   ```
   sudo apt install ros-$ROS_DISTRO-rmw-cyclonedds-cpp
   export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
   ```

3. Make sure that `colcon`, its extensions, and `vcs` are installed:
   ```
   sudo apt install python3-colcon-common-extensions python3-vcstool
   ```

4. Create a new ROS2 workspace:
   ```
   export COLCON_WS=~/workspace/ros2_kortex_ws
   mkdir -p $COLCON_WS/src
   ```

5. Pull relevant packages, install dependencies, compile, and source the workspace by using:
   ```
   cd $COLCON_WS
   git clone https://github.com/PickNikRobotics/ros2_kortex.git src/ros2_kortex
   vcs import src --skip-existing --input src/ros2_kortex/ros2_kortex.repos
   rosdep install --ignore-src --from-paths src -y -r
   colcon build --cmake-args -DCMAKE_BUILD_TYPE=Release
   source install/setup.bash
   ```

6. To simulate the robot with ignition or gazebo make sure to pull and build additional packages:
   ```
   vcs import src --skip-existing --input src/ros2_kortex/simulation.repos
   rosdep install --ignore-src --from-paths src -y -r
   colcon build --cmake-args -DCMAKE_BUILD_TYPE=Release
   source install/setup.bash
   ```

**Note:** Due to mismatched protobuf version that ships system and used by Gazebo simulator compiling twice may be required.
If errors are encounter you must clean your workspace and run colcon build in two steps:

1. build everything except kortex related packages
2. build the packages that where skipped

```
sudo apt install python3-colcon-clean # if you don't have colcon-clean installed already
colcon clean workspace -y
colcon build --packages-skip-regex '.*kortex.*' '.*gen3.*'
colcon build --packages-select-regex '.*kortex.*' '.*gen3.*'
```

## Usage
<!-- TODO(moriarty) this section is an information overload -->

To launch and view the robots URDF run:

```bash
ros2 launch kortex_description view_robot.launch.py
```

To simulate the 7 DoF Kinova Gen3 robot arm with mock hardware:

```bash
ros2 launch kortex_bringup gen3.launch.py \
  robot_ip:=yyy.yyy.yyy.yyy \
  use_fake_hardware:=true
```

To generate motion plans and execute them with a simulated 7 DoF Kinova Gen3 arm with mock hardware:

```bash
ros2 launch kinova_gen3_7dof_robotiq_2f_85_moveit_config robot.launch.py \
  robot_ip:=yyy.yyy.yyy.yyy \
  use_fake_hardware:=true
```

Alternatively, if you wish to use the Kinova Gen3's 6 DoF variant:

```bash
ros2 launch kortex_bringup gen3.launch.py \
  robot_ip:=yyy.yyy.yyy.yyy \
  use_fake_hardware:=true \
  dof:=6
```

and to bring up the Kinova Gen3 6 DoF with MoveIt:

```bash
ros2 launch kinova_gen3_6dof_robotiq_2f_85_moveit_config robot.launch.py \
  robot_ip:=yyy.yyy.yyy.yyy \
  use_fake_hardware:=true
```

To simulate the 7dof Kinova Gen3 robot with ignition run the following:

```bash
ros2 launch kortex_bringup kortex_sim_control.launch.py \
  dof:=7 \
  use_sim_time:=true \
  launch_rviz:=false
```

and to use MoveIt to command the robot:

```bash
ros2 launch kinova_gen3_7dof_robotiq_2f_85_moveit_config sim.launch.py \
  use_sim_time:=true
```

To work with a physical robot and generate/execute paths with MoveIt run the following:

```bash
ros2 launch kinova_gen3_7dof_robotiq_2f_85_moveit_config robot.launch.py \
  robot_ip:=192.168.1.10
```
**Note: If you have reassigned your physical arm's robot IP address, then you will need to assign that ip address to `robot_ip`**

The Robotiq 2f 85 Gripper will be available on the Action topic:

```bash
/robotiq_gripper_controller/gripper_cmd
```

You can test the gripper by calling the Action server with the following command and setting the desired `position` of thr gripper (`0.0=open`, `0.8=close`)

```bash
ros2 action send_goal /robotiq_gripper_controller/gripper_cmd control_msgs/action/GripperCommand "{command:{position: 0.0, max_effort: 100.0}}"
```

## Contents

The following is a description of the packages included in this repository.

### kortex_description
This package contains the URDF (Unified Robot Description Format), STL and configuration files for the Kortex-compatible robots. For more details, please consult the [README](kortex_description/readme.md) from the package subdirectory.

### kortex_driver
This package implements a ROS node that allows communication between a node and a Kinova Gen3 or Gen3 lite robot. For more details, please consult the [README](kortex_driver/readme.md) from the package subdirectory.

### kortex_moveit_config
This metapackage contains the auto-generated MoveIt! files to use the Kinova Gen3 and Gen3 lite arms with the MoveIt! motion planning framework. For more details, please consult the [README](kortex_moveit_config/readme.md) from the package subdirectory.
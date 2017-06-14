# Mavros-Gazebo

Instructions and scripts for configuring sitl, gazebo and mavros.

## Install Gazebo 7 and Gazebo-ROS Wrapper

TODO: Install Gazebo and Gazebo-ROS for Indigo and upgrade the Gazebo version to 7.X

## Source the SITL-Gazebo Directory from PX4/Firmware repo

Add the following lines into `~/.bashrc` to include the gazebo and ros resources from the PX4 repo

```
source ~/catkin_ws/devel/setup.bash
source ~/src/Firmware/Tools/setup_gazebo.bash ~/src/Firmware ~/src/Firmware/build_posix_sitl_default
export LC_NUMERIC=C
export ROS_PACKAGE_PATH=$ROS_PACKAGE_PATH:~/src/Firmware
export ROS_PACKAGE_PATH=$ROS_PACKAGE_PATH:~/src/Firmware/Tools/sitl_gazebo
```

## Editing Lidar Model to Publish to ROS

TODO

## Import Gazebo Model

TODO

## Configuring for Vision Fusion

The tunnel centroid estimator algorithm outputs the error of the UAV from the centroid. This is published as "vision" position input to the UAV.

Configure the SITL parameters to fuse in the data into PX4 local position estimator accordingly.

Edit the `iris_opt_flow` file located at `~/src/Firmware/posix-config/SITL/init/lpe` and add in the following lines:

```
#Fuse vision position xyz
param set LPE_FUSION 244
#Confidence of the vision position xyz
param set LPE_VIS_XY 0.1
#Fuse vision yaw
param set ATT_EXT_HDG_M 2
```

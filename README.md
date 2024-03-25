# VIO-HAR
VIO-BASED HUMAN ACTIVITY DETECTION

In our method we have created a workflow for acquiring the trajectory out of the human raw activities data.

To acquire the raw data we used the MARS LOGGER Android app. It records both video data and IMU data at the same time. After acquiring the raw data we get 4 different files.

So, our workflow will be like this:
Get raw data from the MARS logger app.
Copy the folder into the working environment which is Ubuntu 18.04
Create rosbag file from the section 1 raw data
Calculate the time difference between the camera and the IMU clock in milliseconds.
For running the rosbag file we need to prepare config.yaml file for the vins-mono
Running the bag file into the vins-mono.
Get the trajectory value CSV file which path is defined in the config.yaml

Step 1: MARS LOGGER

Install MARS Logger android app: Link

Step 2: Install Ubuntu

Setup Ubuntu, in our case we used Virtual Box for the convenient of our working. To setup ubuntu you can follow this youtube tutorials: How to Install Ubuntu in VirtualBox on Windows 10 | Ubuntu 20.04 64bit - YouTube (This tutorial is for ubuntu 20.04 but all the process are same.) We must use Ubuntu 18.04 for our working environment otherwise you might get unnecessary errors while doing the next steps. 



Here are some of the commands for the terminal that will be helpful for the installation process.

Guest Addition CD: sudo apt install build-essential dkms linux-headers-$(uname -r)

Step 3: Install ROS

In this step, we’ll install ROS (Robot Operating System) on ubuntu 18.04 and we must use ros melodic for our workflow. Download link for ROS Melodic: Click Here

Please follow this tutorial playlist for the complete ros intalationa and creating the catkin workspace Intro: Install and Setup ROS Noetic - ROS Tutorial 1 (ROS1) (youtube.com) (Again this tutorial for ros Noetic but all the process is same for ros melodic)

Some of the command which will be helpful for the installation process:
gedit ~/.bashrc
source /opt/ros/melodic/setup.bash

Testing the ros installation:
Turtle Simulation:       rosrun turtlesim turtlesim_node
rosrun turtlesim turtle_teleop_key

Creating the catkin workspace:
source ~/catkin_ws/devel/setup.bash

If you follow that video tutorial step by step you’ll be able to install ros melodic successfully

Step 4: Installing Vins Mono

Follow this link (Click Here) for the vins-mono GitHub page where all theinstructions is available but for the simplicity you can follow this command which will make the process easier. I recommend to read through all the documentation on the vins-mono github page.

First install this Dependencies: run this in the terminal window.

sudo apt-get install ros-melodic-cv-bridge ros-melodic-tf ros-melodic-message-filters ros-melodic-image-transport


Install Ceres Solver using version 1.14.0 for Ubuntu 18.04


To install Ceres Solver version 1.14.0 on Ubuntu 18.04, you can follow these steps:

Update Package Index:

sudo apt-get update

Install Dependencies: Ceres Solver has several dependencies that need to be installed first. Run the following command to install them:

sudo apt-get install -y cmake \
                     g++ \
                     git \
                     libgoogle-glog-dev \
                     libatlas-base-dev \
                     libsuitesparse-dev \
                     libeigen3-dev

Download Ceres Solver Source Code:

You can clone the Ceres Solver repository from GitHub.


git clone https://ceres-solver.googlesource.com/ceres-solver

cd ceres-solver

Checkout Version 1.14.0:

git checkout tags/1.14.0

mkdir build
cd build
cmake ..
make -j$(nproc)

Install Ceres Solver: After building successfully, you can install Ceres Solver:

sudo make install
sudo ldconfig

In our case github cloning for the vinsmono was not working for the network issue. That’s why I manually downloaded the repositories then extracted the repo to the Catkin Workspace under the src folder.
(Please follow my screen recording for this process)

Here’s the command window:
cd catkin_ws
catkin_make

Now we need to set up our environment for creating the bag file and making the configuration file for the Android device. 

Step 5: Installing VIO-Commons For creating the bag file.
Here’s the official documentation for the VIO-Commons: Click Here

First install all the dependencies according to the documentation. We used ROS installation method as we have already installed ros in our system.

Step 6: Creating a new roslaunch vins_estimator android.launch configuration

For this navigate the this path:

/home/your-system-name/catkin_ws/src/VINS-Mono/vins_estimator/launch

In this folder create a new file name: 
android.launch

And paste this code on that file:
'
<launch>
    <arg name="config_path" default = "$(find feature_tracker)/../config/android/pocof1.yaml" />
	  <arg name="vins_path" default = "$(find feature_tracker)/../config/../" />
    <node name="feature_tracker" pkg="feature_tracker" type="feature_tracker" output="log">
        <param name="config_file" type="string" value="$(arg config_path)" />
        <param name="vins_folder" type="string" value="$(arg vins_path)" />
    </node>
    <node name="vins_estimator" pkg="vins_estimator" type="vins_estimator" output="screen">
       <param name="config_file" type="string" value="$(arg config_path)" />
       <param name="vins_folder" type="string" value="$(arg vins_path)" />
    </node>
    <node name="pose_graph" pkg="pose_graph" type="pose_graph" output="screen">
        <param name="config_file" type="string" value="$(arg config_path)" />
        <param name="visualization_shift_x" type="int" value="0" />
        <param name="visualization_shift_y" type="int" value="0" />
        <param name="skip_cnt" type="int" value="0" />
        <param name="skip_dis" type="double" value="0" />
    </node>
</launch>
'

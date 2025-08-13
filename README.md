# VIO-HAR
## VIO-BASED HUMAN ACTIVITY DETECTION

In our method, we have created a workflow for acquiring the trajectory out of the human raw activities data.

To acquire the raw data we used the [**MARS LOGGER**](https://github.com/OSUPCVLab/mobile-ar-sensor-logger/releases/download/v1.0-android/app-fdroid-release.apk). It records both video data and IMU data at the same time. After acquiring the raw data we get 4 different files.

**Our Workflow:**
* Get raw data from the MARS logger app.
* Copy the folder into the working environment which is Ubuntu 18.04
* Create rosbag file from the section 1 raw data
* Calculate the time difference between the camera and the IMU clock in milliseconds.
* For running the rosbag file we need to prepare config.yaml file for the vins-mono
* Running the bag file into the vins-mono.
* Get the trajectory value CSV file which path is defined in the config.yaml

## **Step 1: MARS LOGGER**

Install MARS Logger android app: [**MARS LOGGER**](https://github.com/OSUPCVLab/mobile-ar-sensor-logger/releases/download/v1.0-android/app-fdroid-release.apk)

MARS Logger Wiki Page: [**MARS LOGGER WIKI**](https://github.com/OSUPCVLab/mobile-ar-sensor-logger/wiki)

## **Step 2: Install Ubuntu**

Setup Ubuntu, in our case we used Virtual Box for the convenience of our working. To set up Ubuntu you can follow this youtube tutorial: [**How to Install Ubuntu in VirtualBox on Windows 10 | Ubuntu 20.04 64bit - YouTube**](https://www.youtube.com/watch?v=IOwlnpWPuj0) (This tutorial is for ubuntu 20.04 but all the process are same)
**We must use Ubuntu 18.04 for our working environment otherwise you might get unnecessary errors while doing the next steps.**



Here are some of the commands for the terminal that will be helpful for the installation process.

Guest Addition CD:
```
sudo apt install build-essential dkms linux-headers-$(uname -r)
```

## **Step 3: Install ROS**

In this step, we’ll install ROS (Robot Operating System) on Ubuntu 18.04 and we must use ROS Melodic for our workflow. Download [ROS Melodic](https://wiki.ros.org/melodic/Installation/Ubuntu)

Please follow this full tutorial playlist for the complete ros installation and creating the catkin workspace [Intro: Install and Setup ROS Noetic - ROS Tutorial 1 (ROS1)](https://www.youtube.com/watch?v=Qk4vLFhvfbI&list=PLLSegLrePWgIbIrA4iehUQ-impvIXdd9Q) (Again this tutorial for ros Noetic but all the process is same for ros melodic)

Some of the command which will be helpful for the installation process:
```
gedit ~/.bashrc
```
```
source /opt/ros/melodic/setup.bash
```

**Testing the ros installation:**
* Turtle Simulation:
```
rosrun turtlesim turtlesim_node
```
```
rosrun turtlesim turtle_teleop_key
```

* Creating the catkin workspace: (Source the catkin_ws to the terminal)
```
source ~/catkin_ws/devel/setup.bash
```
If you follow that video tutorial step by step you’ll be able to install ros Melodic successfully

## **Step 4: Installing Vins Mono**

Follow this link [Click Here](https://github.com/HKUST-Aerial-Robotics/VINS-Mono) for the vins-mono GitHub page where all the instructions are available but for simplicity you can follow this command which will make the process easier. You should read through all the documentation on the vins-mono GitHub page.

* **First install these Dependencies: run this in the terminal window.**
```
sudo apt-get install ros-melodic-cv-bridge ros-melodic-tf ros-melodic-message-filters ros-melodic-image-transport
```
* **Install Ceres Solver using version 1.14.0 for Ubuntu 18.04**
To install Ceres Solver version 1.14.0 on Ubuntu 18.04, you can follow these steps:

Update Package Index:
```
sudo apt-get update
```
Install Dependencies: Ceres Solver has several dependencies that must be installed first. Run the following command to install them:
```
sudo apt-get install -y cmake \
                     g++ \
                     git \
                     libgoogle-glog-dev \
                     libatlas-base-dev \
                     libsuitesparse-dev \
                     libeigen3-dev
```

* **Download Ceres Solver Source Code:** You can clone the Ceres Solver repository from GitHub.
```
git clone https://ceres-solver.googlesource.com/ceres-solver
```
```
cd ceres-solver
Checkout Version 1.14.0:
git checkout tags/1.14.0
```
```
mkdir build
cd build
```
```
cmake ..
make -j$(nproc)
```
After building successfully, you can install Ceres Solver:
```
sudo make install
```
```
sudo ldconfig
```
In our case github cloning for the vinsmono was not working for the network issue. That’s why I manually downloaded the repositories then extracted the repo to the Catkin Workspace under the src folder.(Please follow my screen recording for this process)

* Here’s the command window:
```
cd catkin_ws
catkin_make
```
You should test the vins-mono by running [EuRoC MAV Dataset](https://projects.asl.ethz.ch/datasets/doku.php?id=kmavvisualinertialdatasets) by following the vins-mono documentation. 

Now we need to set up our environment for creating the bag file and making the configuration file for the Android device. 


## **Step 5: Prepare config.yaml file**
Navigate to this folder: `/home/your-name/catkin_ws/src/VINS-Mono/config`
_(Your path can be different compare to mine **your-name** refers to your system name)_
Create a new folder you can name the folder anything, in my case I used android. Now navigate to the `/home/your-name/catkin_ws/src/VINS-Mono/config/android` folder and create a new file which extension in `.yaml` in my case I named it `poco.yaml`
Open `poco.yaml` and paste this code initially but remember this parameters values are for poco f1. If you use any different device you might need to change some of the perameters. 
```
%YAML:1.0
#common parameters
imu_topic: "/imu0"
image_topic: "/cam0/image_raw"
output_path: "/home/your/output/path"
#camera calibration 
model_type: PINHOLE
camera_name: camera
image_width: 1280
image_height: 960

distortion_parameters:

   k1: 0e-01

   k2: 0e-02

   p1: 0e-05

   p2: 0e-04

projection_parameters:

   fx: 894.6

   fy: 894.6

   cx: 640.0

   cy: 360.0

# Extrinsic parameter between IMU and Camera.
estimate_extrinsic: 0   # 0  Have an accurate extrinsic parameters. We will trust the following imu^R_cam, imu^T_cam, don't change it.
                        # 1  Have an initial guess about extrinsic parameters. We will optimize around your initial guess.
                        # 2  Don't know anything about extrinsic parameters. You don't need to give R,T. We will try to calibrate it. Do some rotation movement at beginning.                        
#If you choose 0 or 1, you should write down the following matrix.
#Rotation from camera frame to imu frame, imu^R_cam
extrinsicRotation: !!opencv-matrix
   rows: 3
   cols: 3
   dt: d
   data: [0, -1, 0, 
          -1, 0, 0, 
           0, 0, -1]
#Translation from camera frame to imu frame, imu^T_cam
extrinsicTranslation: !!opencv-matrix
   rows: 3
   cols: 1
   dt: d
   data: [0.0, 0.0, 0.0]
#feature traker paprameters
max_cnt: 200            # max feature number in feature tracking
min_dist: 30            # min distance between two features 
freq: 10                # frequence (Hz) of publish tracking result. At least 10Hz for good estimation. If set 0, the frequence will be same as raw image 
F_threshold: 1.0        # ransac threshold (pixel)
show_track: 1           # publish tracking image as topic
equalize: 1             # if image is too dark or light, trun on equalize to find enough features
fisheye: 0              # if using fisheye, trun on it. A circle mask will be loaded to remove edge noisy points
#optimization parameters
max_solver_time: 0.04  # max solver itration time (ms), to guarantee real time
max_num_iterations: 8   # max solver itrations, to guarantee real time
keyframe_parallax: 10.0 # keyframe selection threshold (pixel)
#imu parameters       The more accurate parameters you provide, the better performance
acc_n: 16.0e-2        # accelerometer measurement noise standard deviation. #0.2
gyr_n: 24.0e-3       # gyroscope measurement noise standard deviation.     #0.05
acc_w: 5.5e-4         # accelerometer bias random work noise standard deviation.  #0.02
gyr_w: 2.0e-4       # gyroscope bias random work noise standard deviation.     #4.0e-5
g_norm: 9.81007      # gravity magnitude
#loop closure parameters
loop_closure: 1                 # start loop closure
fast_relocalization: 1          # useful in real-time and large project
load_previous_pose_graph: 0     # load and reuse previous pose graph; load from 'pose_graph_save_path'
pose_graph_save_path: "/home/sajjad/Mars_Log_DS/results" # save and load path

#unsynchronization parameters
estimate_td: 1                      # online estimate time offset between camera and imu
td: 0.042222                        # initial value of time offset. unit: s. readed image clock + td = real image clock (IMU clock)

#rolling shutter parameters
rolling_shutter: 1                  # 0: global shutter camera, 1: rolling shutter camera
rolling_shutter_tr: 0.020           # unit: s. rolling shutter read out time per frame (from data sheet). 

#visualization parameters

save_image: 0                   # save image in pose graph for visualization prupose; you can close this function by setting 0 
visualize_imu_forward: 0        # output imu forward propogation to achieve low latency and high frequence results
visualize_camera_size: 0.4      # size of camera marker in RVIZ

```
Here's also the config file for your convinient [pocof1.yaml](https://github.com/sajjad-hm/VIO-HAR/blob/main/pocof1.yaml)

## **Step 6: Creating a new roslaunch vins_estimator android.launch configuration**
Now, we need to link the new configuration file_poco.yaml_ for launching the vins-mono. For this, first navigate to this path:
`/home/your-name/catkin_ws/src/VINS-Mono/vins_estimator/launch`

_Your path can be different compare to mine **your-name** refers to your system name_

In this folder create a new file like this: 
```
android.launch
```
_you can use any name but the extension should be .launch in my case I used android.launch_

**Now, paste this code on that file:**
NB: make sure you use the correct folder path for this line ` <arg name="config_path" default = "$(find feature_tracker)/../config/android/pocof1.yaml" />`
**In my case my folder path is `/config/android/pocof1.yaml`**
```
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
```
## **Step 7: Installing VIO-Commons For creating the bag file.**
Here’s the official documentation for the VIO-Commons: [Click Here](https://github.com/JzHuai0108/vio_common) 
Follow these steps for installing VIO-Commons:
* Install the dependencies
```
sudo apt-get install libopencv-dev libeigen3-dev
```
* Clone the repositories under catkin_ws/src folder
```
cd catkin_ws/src
git clone https://github.com/sajjadxhm/vio_common.git
```
* Now Catkin build the repositories
```
cd catkin_ws
catkin_make
```

## **Step 8: Convert to Rosbag**

* Find the path under catkin_ws/src/vio_common/python/kalibr_bagcreater.py and make sure this is the right path to put under BAG_PYTHON
* ANDROID_DATA_DIR= Your Android data directory needs to be copied from the Android phone to the PC, please check the [**MARS LOGGER WIKI**](https://github.com/OSUPCVLab/mobile-ar-sensor-logger/wiki) page for more information
```
BAG_PYTHON=catkin_ws/src/vio_common/python/kalibr_bagcreater.py
ANDROID_DATA_DIR=/path/to/android/data/session
python $BAG_PYTHON --video $ANDROID_DATA_DIR/movie.mp4 \
--imu $ANDROID_DATA_DIR/gyro_accel.csv \
--video_time_file $ANDROID_DATA_DIR/frame_timestamps.txt \
--output_bag $ANDROID_DATA_DIR/movie.bag
```

## **Step 9: Calculate the time difference between camera sensor and IMU sensor**
We need to tell the config.yaml file about the time difference between camera and IMU sensor. After collecting data from _MARS LOGGER_ there's four different files in the folder 
* edge_epochs.txt
* frame_timestamps.txt
* gyro_accel.csv
* movie.mp4
* movie_metadata.csv

  **We calculated the time difference between frame_timestamps.txt and gyro_accel.csv**

IMU and camera clock trigger differently. Though MARS Logger try to trigger both of the clock in the same time but still there's some td(time difference). Here you will get timestamps in ns but in the configuration file [pocof1.yaml](https://github.com/sajjad-hm/VIO-HAR/blob/main/pocof1.yaml) we need to provide the td value in ms.



## **Step 10: Running ROS BAG file into VINS-MONO**
**Running our bag file into the vins-mono**
Here's some of the similar terminal command for running your own dataset bag file into the vins-mono. Open three terminal. 
* In the first terminal 
```
roslaunch vins_estimator android.launch
```
* In the second terminal 
```
roslaunch vins_estimator vins_rviz.launch
```
* In the third terminal 
```
rosbag play /your/bag/file/path/movie.bag
```
NB: Make sure you have provided the proper output path in the [pocof1.yaml](https://github.com/sajjad-hm/VIO-HAR/blob/main/pocof1.yaml) configuration file. 
After running successfully you will get three new output file 

* `edge_epochs.txt`
* `vins_result_no_loop.csv`
* `vins_result_loop.csv`

Where you will find timestamp, trajectory XYZ value also, rotation XYZ and a scealer value.

# ROS Wrapper for ZT PILOT&reg; Finder&trade; Device
These are packages for using ZT PILOT Finder camera (F100i Tracking Module) with ROS.

This version supports Melodic and Noetic distributions.

For running in ROS2 environment please switch to the [ros2 branch](https://github.com/ztpilot/finder_ros/tree/ros2). </br>

LibFinder supported version: v0.0.5 (see [finder_camera release notes](https://github.com/ztpilot/finder_ros/releases))

## Installation Instructions

### Ubuntu
   #### Step 1: Install the ROS distribution
   - #### Install [ROS Melodic](http://wiki.ros.org/melodic/Installation/Ubuntu) on Ubuntu 18.04 or [ROS Noetic](http://wiki.ros.org/noetic/Installation/Ubuntu) on Ubuntu 20.04.

### Windows
   #### Step 1: Install the ROS distribution
   - #### Install [ROS Melodic or later on Windows 10](https://wiki.ros.org/Installation/Windows)

### Install finder_camera from source:

* ### Method 1: The Finder&trade; distribution:
     > This option is demonstrated in the [.travis.yml](https://github.com/ztpilot/finder_ros/blob/development/.travis.yml) file. It basically summerize the elaborate instructions in the following 2 steps:


   ### Step 1: Build the latest ZTpilot&reg; Finder&trade; SDK

   - Create a [catkin](http://wiki.ros.org/catkin#Installing_catkin) workspace  

   *Ubuntu*
   ```bash
   mkdir -p ~/catkin_ws/src
   cd ~/catkin_ws/src/
   ```
   *Windows*
   ```batch
   mkdir c:\catkin_ws\src
   cd c:\catkin_ws\src
   ```

   - Clone the latest ZTpilot&reg; Finder&trade; ROS from [here](https://github.com/ztpilot/finder_ros/releases) into 'catkin_ws/src/'
   ```bashrc
   git clone https://github.com/ztpilot/finder_ros.git
   cd finder_ros/
   git checkout `git tag | sort -V | grep -P "^2.\d+\.\d+" | tail -1`
   cd ..
   ```
   - Make sure all dependent packages are installed. You can check .travis.yml file for reference.
   - Specifically, make sure that the ros package *ddynamic_reconfigure* is installed. If *ddynamic_reconfigure* cannot be installed using APT or if you are using *Windows* you may clone it into your workspace 'catkin_ws/src/' from [here](https://github.com/pal-robotics/ddynamic_reconfigure/tree/kinetic-devel)


   ```bash
  catkin_init_workspace
  cd ..
  catkin_make clean
  catkin_make -DCATKIN_ENABLE_TESTING=False -DCMAKE_BUILD_TYPE=Release
  catkin_make install
  ```

  *Ubuntu*
  ```bash
  echo "source ~/catkin_ws/devel/setup.bash" >> ~/.bashrc
  source ~/.bashrc
  ```

  *Windows*
  ```batch
  devel\setup.bat
  ```

## Usage Instructions

### Start the camera node
To start the camera node in ROS:

```bash
roslaunch finder_camera demo_f100i.launch
```

This will stream all camera sensors and publish on the appropriate ROS topics.

Other stream resolutions and frame rates can optionally be provided as parameters to the 'rs_camera.launch' file.

### Published Topics
The published topics differ according to the device and parameters.
After running the above command with D435i attached, the following list of topics will be available (This is a partial list. For full one type `rostopic list`):
- /f100i/color/camera_info
- /f100i/color/image_raw
- /f100i/color/metadata
- /f100i/extrinsics/imu_to_color
- /f100i/extrinsics/gps_to_color
- /f100i/extrinsics/depth_to_color
- /f100i/gyro/imu_info
- /f100i/gyro/metadata
- /f100i/gyro/data
- /f100i/accel/imu_info
- /f100i/accel/metadata
- /f100i/accel/data
- /f100i/imu/data
- /f100i/slam/pose
- /f100i/slam/trajectory
- /f100i/slam/local_map
- /f100i/slam/global_map
- /diagnostics

### Launch parameters
The following parameters are available by the wrapper:
- **serial_no**: will attach to the device with the given serial number (*serial_no*) number. Default, attach to available RealSense device in random.
- **device_type**: will attach to a device whose name includes the given *device_type* regular expression pattern. Default, ignore device type. For example, device_type:=f100 will match F100i. device_type=f500(?!i) will match F500i.

- **rosbag_filename**: Will publish topics from rosbag file.
- **initial_reset**: On occasions the device was not closed properly and due to firmware issues needs to reset. If set to true, the device will reset prior to usage.
- **reconnect_timeout**: When the driver cannot connect to the device try to reconnect after this timeout (in seconds).
- **enable_sync**: gathers closest frames of different sensors, infra red, color and depth, to be sent with the same timetag. This happens automatically when such filters as pointcloud are enabled.
- ***<stream_type>*_width**, ***<stream_type>*_height**, ***<stream_type>*_fps**: <stream_type> can be any of *infra, color, fisheye, depth, gyro, accel, pose, confidence*. Sets the required format of the device. If the specified combination of parameters is not available by the device, the stream will be replaced with the default for that stream. Setting a value to 0, will choose the first format in the inner list. (i.e. consistent between runs but not defined).</br>*Note: for gyro accel and pose, only _fps option is meaningful.
- **enable_*<stream_name>***: Choose whether to enable a specified stream or not. Default is true for images and false for orientation streams. <stream_name> can be any of *infra1, infra2, color, depth, fisheye, fisheye1, fisheye2, gyro, accel, pose, confidence*.
- **tf_prefix**: By default all frame's ids have the same prefix - `camera_`. This allows changing it per camera.
- ***<stream_name>*_frame_id**, ***<stream_name>*_optical_frame_id**, **aligned_depth_to_*<stream_name>*_frame_id**: Specify the different frame_id for the different frames. Especially important when using multiple cameras.
- **base_frame_id**: defines the frame_id all static transformations refers to.
- **odom_frame_id**: defines the origin coordinate system in ROS convention (X-Forward, Y-Left, Z-Up). pose topic defines the pose relative to that system.
- **All the rest of the frame_ids can be found in the template launch file: [nodelet.launch.xml](./finder_camera/launch/includes/nodelet.launch.xml)**
- **unite_imu_method**: The F100i and F500i cameras have built in IMU components which produce 2 unrelated streams: *gyro* - which shows angular velocity and *accel* which shows linear acceleration. Each with it's own frequency. By default, 2 corresponding topics are available, each with only the relevant fields of the message sensor_msgs::Imu are filled out.
Setting *unite_imu_method* creates a new topic, *imu*, that replaces the default *gyro* and *accel* topics. The *imu* topic is published at the rate of the gyro. All the fields of the Imu message under the *imu* topic are filled out.
   - **linear_interpolation**: Every gyro message is attached by the an accel message interpolated to the gyro's timestamp.
   - **copy**: Every gyro message is attached by the last accel message.
- **clip_distance**: remove from the depth image all values above a given value (meters). Disable by giving negative value (default)
- **linear_accel_cov**, **angular_velocity_cov**: sets the variance given to the Imu readings. For the F100i, these values are being modified by the inner confidence value.
- **hold_back_imu_for_frames**: Images processing takes time. Therefor there is a time gap between the moment the image arrives at the wrapper and the moment the image is published to the ROS environment. During this time, Imu messages keep on arriving and a situation is created where an image with earlier timestamp is published after Imu message with later timestamp. If that is a problem, setting *hold_back_imu_for_frames* to *true* will hold the Imu messages back while processing the images and then publish them all in a burst, thus keeping the order of publication as the order of arrival. Note that in either case, the timestamp in each message's header reflects the time of it's origin.
- **topic_odom_in**: For F100i, add wheel odometry information through this topic. The code refers only to the *twist.linear* field in the message.
- **calib_odom_file**: For the F100i to include odometry input, it must be given a [configuration file](https://github.com/ztpilot/finder_ros/blob/master/unit-tests/resources/calibration_odometry.json). Explanations can be found [here](https://github.com/ztpilot/finder_ros/pull/3462). The calibration is done in ROS coordinates system.
- **publish_tf**: boolean, publish or not TF at all. Defaults to True.
- **tf_publish_rate**: double, positive values mean dynamic transform publication with specified rate, all other values mean static transform publication. Defaults to 0 
- **publish_odom_tf**: If True (default) publish TF from odom_frame to pose_frame.

### Available services:
- reset : Cause a hardware reset of the device. Usage: `rosservice call /finder/reset`
- enable : Start/Stop all streaming sensors. Usage example: `rosservice call /finder/enable False"`
- device_info : retrieve information about the device - serial_number, firmware_version etc. Type `osservice type /finder/device_info | rossrv show` for the full list. Call example: `rosservice call /finder/device_info`

### Point Cloud
Here is an example of how to start the camera node and make it publish the point cloud using the pointcloud option.
```bash
roslaunch finder_camera rs_camera.launch filters:=pointcloud
```
Then open rviz to watch the pointcloud:
<p align="center"><img src="https://user-images.githubusercontent.com/17433152/35396613-ddcb1d6c-01f5-11e8-8887-4debf178d0cc.gif" /></p>

### Aligned Depth Frames
Here is an example of how to start the camera node and make it publish the aligned depth stream to other available streams such as color or infra-red.
```bash
roslaunch finder_camera rs_camera.launch align_depth:=true
```
<p align="center"><img width=50% src="https://user-images.githubusercontent.com/17433152/35343104-6eede0f0-0132-11e8-8866-e6c7524dd079.png" /></p>

### Set Camera Controls Using Dynamic Reconfigure Params
The following command allow to change camera control values using [http://wiki.ros.org/rqt_reconfigure].
```bash
rosrun rqt_reconfigure rqt_reconfigure
```
<p align="center"><img src="https://user-images.githubusercontent.com/40540281/55330573-065d8600-549a-11e9-996a-5d193cbd9a93.PNG" /></p>

### Work with multiple cameras
**Important Notice:** Launching multiple F100i cameras is currently not supported. This will be addressed in a later version. 

### About Frame ID
The wrapper publishes static transformations(TFs). The Frame Ids are divided into 3 groups:
- ROS convention frames: follow the format of <tf_prefix>\_<\_stream>"\_frame" for example: camera_depth_frame, camera_infra1_frame, etc.
- Original frame coordinate system: with the suffix of <\_optical_frame>. For example: camera_infra1_optical_frame. Check the device documentation for specific coordinate system for each stream.
- base_link: For example: camera_link. A reference frame for the device. In F100i it is the depth frame. In F100i, the pose frame.


### finder_description package:
For viewing included models, a separate package is included. For example:
```bash
roslaunch finder_description view_f100i_model.launch
```

### Unit tests:
Unit-tests are not available for now.

## Known Issues
* This ROS node does not currently support [ROS Lunar Loggerhead](http://wiki.ros.org/lunar).
* This ROS node currently does not support running multiple f100i cameras at once. This will be addressed in a future update. 

## License
Copyright 2018 ZT Pilot Corporation

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this project except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

**Other names and brands may be claimed as the property of others*

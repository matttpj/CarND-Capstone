# **Capstone Project: Programming a Real Self Driving Car**

## Team:

Full name|Email|GitHub profile|
|--------|---------|-------------|
Bulat Yapparov|byapparov[ at ]gmail.com|[byapparov](https://github.com/byapparov)
Erik Rosen|erik.rosen[ at ]coolit.se|[erik-rosen](https://github.com/erik-rosen)
Matthew Jones|matttpj[ at ]hotmail.com|[matttpj](https://github.com/matttpj)

## Project summary
This is the project repo for our team's final project of the Udacity Self-Driving Car Engineer Nanodegree. The **Capstone Project: Programming a Real Self Driving Car** requires the team to write, test and integrate their own code with existing software packages provided by Udacity and other third parties. The team must then run their project code in the Udacity driving simulator. The test vehicle is required to drive safely in autonomous mode in a simulated driving environment and respect various traffic regulations, such as stopping, when required, at traffic lights. If the project code runs successfully in the simulator then the code is tested again on Udacity's real-world self driving car, Carla. For more information about the full Nanodegree program please follow this link [Udacity Self Driving Car Engineer](https://www.udacity.com/course/self-driving-car-engineer-nanodegree--nd013).

[//]: # (Image References)
__Output from driving simulator__  
<img src="./imgs/CarND-Capstone_v03.gif" width=100% height=100%>
[download video](https://github.com/erik-rosen/CarND-Capstone/raw/master/output_videos/capstone_v02.mp4)

## System architecture
The Capstone Project uses the Robot Operating System or [ROS](https://www.ros.org/) as a framework for its underlying communications infrastructure.
The diagram below illustrates the key ROS nodes and topics used by the project.
<img src="./imgs/final-project-ros-graph-v2.png" width=100% height=100%>

## Explanation of key software packages

### Control

The Udacity self driving car Carla is equipped with a Drive by Wire (DBW) system which enables its throttle, brake and steering to be controlled electronically.    

**Twist Controller** package contains the key files that are responsible for the control of the vehicle, **dbw_node.py** and **twist_controller.py**.  The DBW node subscribes to the */current_velocity* and */twist_cmd* topics to receive target linear and angular velocities and also subscribes to the */vehicle/dbw_enabled* topic to understand if the vehicle is in autonomous mode and under DBW control.  The DBW node publishes throttle, brake and steering commands to the */vehicle/throttle_cmd*, */vehicle/brake_cmd* and */vehicle/steering_cmd* topics.

### Planning

**Waypoint Loader** package loads the static waypoint data and publishes to the */base_waypoints* topic.  

**Waypoint Follower** package subscribes to the */final_waypoints* topic and publishes target vehicle linear and angular velocities in the form of twist commands to the */twist_cmd* topic. The Waypoint Follower package is provided by [Autoware](https://github.com/Autoware-AI/autoware.ai).   

**Waypoint Updater** package contains the file **waypoint_updater.py** which updates the target velocity for each waypoint based on traffic light and obstacle detection data. The Waypoint Updater node publishes a list of waypoints ahead of the car with target velocities to the */final_waypoints* topic.
Input for the nodes comes from subscriptions to these topics:

- */base_waypoints*
- */current_pose*
- */obstacle_waypoint*
- */traffic_waypoint*

### Perception

**Traffic Light Detector** package contains the code for the Traffic Light Detection node **tl_detector.py**.  This node subscribes to */image_color*, */current_pose* and */base_waypoints* topics and publishes the locations to stop for red traffic lights to the */traffic_waypoint* topic.  The */current_pose* topic provides the vehicle's current position and */base_waypoints* provides a complete list of waypoints for the car to follow.  

**TLDetector** class subscribes to the */image_color* topic that holds a stream of images captured by the camera on the front of the vehicle.  When a traffic light is detected in an image, the traffic light is cropped out and the color of the traffic light is detected by calling the get_classification() method of the TLClassifier class. If the traffic light color (red or yellow) requires the vehicle to slow down or stop, an event is published to the */traffic_waypoint* topic to be processed by the Waypoint Updater node.    

**Traffic Light Classification** model is a sub_package defined in file **tl_classifier.py**. **TLClassifier** class uses the **ssd_mobilenet_v1_coco** model from the [TensorFlow 1 Detection Model Zoo](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/tf1_detection_zoo.md) that had been previously trained with the [COCO Dataset](https://cocodataset.org/) of real-world traffic light images captured in San Diego, USA.

Camera image classification is done in two stages:

- Segmentation - first traffic light objects are cropped out of the image using the TensorFlow model.
- Light Detection - using heuristic method red colour are detected in the traffic light crop.  

### System launch

To improve car behaviour on system launch we have introduced `\tl_detector_ready` topic that gets Bool messages
from `tl_detector` node once the first image is processed.

`waypoint_updater` will publish waypoints only once first message is received from `tl_detector` which can take some time
because of the Tensorlfow session initiation. 


### Styx and Styx Messages

**Styx** package contains a server for communicating with the simulator and a bridge to translate and publish simulator messages to ROS topics.    
**Styx Messages** package includes definitions of the custom ROS message types used in the project.

---
## Rosbag test

After submission of the project, the Udacity review team will test our project code on their instance of the driving simulator and Carla, the Udacity real-world self driving car, if  current COVID conditions allow.  Udacity will provide a Rosbag dataset from their version of the driving simulator which we will be able to use to check that a self driving car running our project code drives safely under restricted conditions and responds to real-world traffic light images.

---
## Installation instructions

Please use **one** of the two installation options, either native **or** docker installation.

### Native installation

* Be sure that your workstation is running Ubuntu 16.04 Xenial Xerus or Ubuntu 14.04 Trusty Tahir. [Ubuntu downloads can be found here](https://www.ubuntu.com/download/desktop).
* If using a Virtual Machine to install Ubuntu, use the following configuration as minimum:
  * 2 CPU
  * 2 GB system memory
  * 25 GB of free hard drive space

  The Udacity provided virtual machine has ROS and Dataspeed DBW already installed, so you can skip the next two steps if you are using this.

* Follow these instructions to install ROS
  * [ROS Kinetic](http://wiki.ros.org/kinetic/Installation/Ubuntu) if you have Ubuntu 16.04.
  * [ROS Indigo](http://wiki.ros.org/indigo/Installation/Ubuntu) if you have Ubuntu 14.04.
* Download the [Udacity Simulator](https://github.com/udacity/CarND-Capstone/releases).

### Docker installation
[Install Docker](https://docs.docker.com/engine/installation/)

Build the docker container
```bash
docker build . -t capstone
```

Run the docker file
```bash
docker run -p 4567:4567 -v $PWD:/capstone -v /tmp/log:/root/.ros/ --rm -it capstone
```

### Port forwarding
To set up port forwarding, please refer to the "uWebSocketIO Starter Guide" found in the classroom (see Extended Kalman Filter Project lesson).

### Usage

1. Clone the project repository
```bash
git clone https://github.com/udacity/CarND-Capstone.git
```

2. Install python dependencies
```bash
cd CarND-Capstone
pip install -r requirements.txt
```
3. Make and run styx
```bash
cd ros
catkin_make
source devel/setup.sh
roslaunch launch/styx.launch
```
4. Run the simulator

### Real world testing
1. Download [training bag](https://s3-us-west-1.amazonaws.com/udacity-selfdrivingcar/traffic_light_bag_file.zip) that was recorded on the Udacity self-driving car.
2. Unzip the file
```bash
unzip traffic_light_bag_file.zip
```
3. Play the bag file
```bash
rosbag play -l traffic_light_bag_file/traffic_light_training.bag
```
4. Launch your project in site mode
```bash
cd CarND-Capstone/ros
roslaunch launch/site.launch
```
5. Confirm that traffic light detection works on real life images

### Other library/driver information
Outside of `requirements.txt`, here is information on other driver/library versions used in the simulator and Carla:

Specific to these libraries, the simulator grader and Carla use the following:

|        | Simulator | Carla  |
| :-----------: |:-------------:| :-----:|
| Nvidia driver | 384.130 | 384.130 |
| CUDA | 8.0.61 | 8.0.61 |
| cuDNN | 6.0.21 | 6.0.21 |
| TensorRT | N/A | N/A |
| OpenCV | 3.2.0-dev | 2.4.8 |
| OpenMP | N/A | N/A |

We are working on a fix to line up the OpenCV versions between the two.

---
## Attribution
We make use of data from the LISA Traffic Light dataset [repo](https://www.kaggle.com/mbornoe/lisa-traffic-light-dataset) to test traffic light classifier code. This dataset is licensed under [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)

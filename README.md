# Binpicking and Placing
A project to identify good grasping points on objects placed in bins and to place them on a mobile robot.

## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. See the [documentation](https://github.com/bobejo/binpickingandplacing/tree/master/Documentation/build "Documentation as PDF and HTML") for more information.


### Prerequisites
#### Computer Vision and CNN
* [Opencv](https://pypi.python.org/pypi/opencv-python "Opencv installation")
* [Tensorflow](https://www.tensorflow.org/install/ "Tensorflow installation") - To use the GPU version of Tensorflow [this guide can be used to install the needed package CUDA.](http://gist.github.com/zhanwenchen/e520767a409325d9961072f666815bb8 "CUDA")
* [Keras](https://keras.io/#installation "Keras installation")
* [h5py](http://docs.h5py.org/en/latest/build.html "h5 installation")
#### ROS
* [ROS Kinetic](http://wiki.ros.org/kinetic/Installation "ROS installation")
* [ros_control](https://github.com/ros-controls/ros_control/tree/kinetic-devel "ros_control Github")
* [robotiq](https://github.com/ros-industrial/robotiq/tree/indigo-devel "robotiq Github")
* [ur_modern_driver](https://github.com/ThomasTimm/ur_modern_driver "ur_modern_driver Github")
* [optoforce](https://github.com/shadow-robot/optoforce "Force/Torque sensor node")
* [caramba-vision](https://gitlab.com/diego-gabriel/caramba-vision "Camera node")
* [apriltag](https://github.com/swatbotics/apriltag "apriltag Github")

### Installing

Download the project using:

```
git clone https://github.com/bobejo/binpickingandplacing.git
```
Make sure to change to path of your images, models and camera matrices in the file [paths.py](https://github.com/bobejo/binpickingandplacing/blob/master/paths.py "Paths").

## Convolutional neural network

The computer vision part consist of a neural network that segments the parts in such a way that the best part to pick has a higher probability.
The output image:
![alt text](https://github.com/bobejo/binpickingandplacing/blob/master/Documentation/figures/pred.png "The output from the network.")

shows how some areas have a higher probability (more yellow).

To train the network for new components make sure that the path to your training data is correctly specified in [paths.py](https://github.com/bobejo/binpickingandplacing/blob/master/paths.py "Paths") then run the function *train_model* in [cnn.](https://github.com/bobejo/binpickingandplacing/blob/master/cnn.py "cnn")
To create annotated images use the program [image_annotation.py.](https://github.com/bobejo/binpickingandplacing/blob/master/image_annotation.py "How to create annotated images")

## How to find the grasping point

The position and angle of the grasping point is found in the following way:

1. Each camera takes an image.
2. Both of the images are cropped and propagated through the CNN.
3. The output images is converted to a binary image using thresholding and then dilated. **The thresholding value and dilation size have to be chosen.**
4. In the left image a contour detector is used to find the centre of the largest area and its angle.
5. This centre is used to choose an area in the right image that is used to find the contour and its centre and angle.
6. Triangulation is used to find the 3D coordinate.
7. The coordinate is transformed to base frame and sent, together with the angle.

## How to pick and place a object

The complete process for picking and placing an object is done in the following way:

1. A list of objects that should be picked is input.
2. By scanning an image taken over the work area for the april tags it is possible to find which objects are present and the positions of the bins.
3. The april tag position is transformed to the position of the MiR Map and the MiR then moves to this position.
4. Because of the MiR has a 5 cm accuracy a tag on the MiR is used to more precisely determine its position and orientation.
5. The robot moves over the bin for the first object with the help of the april tag position.
6. The image is input to the CNN and a grasping point and angle is found.
7. The grasping point and angle is converted to the frame of the robot.
5. The robot now pick the object and then moves to the placing area and places it in a specific pose.

### Before running the process
Some ROS nodes must be running

The robot node with the ip of the robot
```
roslaunch ur_modern_driver ur10_bringup_joint_limited.launch robot_ip:=XXX.XXX.XXX.XXX
```
The Optoforce sensor node
```
roslaunch optoforce optoforce.launch
```
The camera node. The calibration file for _camera must be created.

```
rosrun vision stereo_camera_node _camera:=ZED170103 _device:=0 _fps:=15 _res:=2.2k
```
The gripper node with the ip adress of the gripper
```
python SModelTcpNode.py XXX.XXX.XXX.XXX
```





## Authors

* **Mahmoud Hanafy** - hanafy@student.chalmers.se
* **Samuel Ingemarsson** - samuel.ingemarsson@gmail.com	


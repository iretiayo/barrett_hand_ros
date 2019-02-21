# barrett_hand_ros
A meta package for controlling the Barrett hand

## Installation instructions
See [https://github.com/CRLab/staubli_barrett_meta_package](https://github.com/CRLab/staubli_barrett_meta_package) for more information about how to build this package. 

### Sources
This package is a combination of these three packages. They have been archived as a result.
- [https://github.com/CRLab/barrett_hand_common](https://github.com/CRLab/barrett_hand_common)
- [https://github.com/CRLab/barrett_hand](https://github.com/CRLab/barrett_hand)
- [https://github.com/CRLab/barrett_tactile_msgs.git](https://github.com/CRLab/barrett_tactile_msgs.git)

### BHand Specific Installation

## Requirements
- [ROS Indigo](http://wiki.ros.org/indigo)
- [peak-linux-driver-7.15.2.tar.gz](https://www.peak-system.com/fileadmin/media/linux/files/peak-linux-driver-7.15.2.tar.gz)
- [pcan_python](https://github.com/RobotnikAutomation/pcan_python)

Install the peak linux driver:
```bash
$ wget https://www.peak-system.com/fileadmin/media/linux/files/peak-linux-driver-7.15.2.tar.gz
$ tar -xvf peak-linux-driver-7.15.2.tar.gz
$ cd peak-linux-driver-7.15.2
$ make -j$(($(nproc) + 1)) NET=NO_NETDEV RT=NO_RT
$ sudo make install
```

Install pcan_python. You'll need to add `export PYTHONPATH=$PYTHONPATH:/usr/lib` to your .bashrc because that is where pcan_python gets installed by default.
```bash
$ git clone https://github.com/RobotnikAutomation/pcan_python.git
$ cd pcan_python
$ sudo make install
$ echo "export PYTHONPATH=$PYTHONPATH:/usr/lib" >> ~/.bashrc
```


### Trouble-shooting Barrett Hand
Create ros workspace and build:
```bash
mkdir barrett_hand_ws/src -p
cd barrett_hand_ws/src
git clone git@github.com:iretiayo/barrett_hand_ros.git
cd ..
source /opt/ros/indigo/setup.bash
catkin_make
```

Connect the hand using USB/CAN connector. Then check which port is assigned to the hand:
```bash
ls /dev | grep pcan
```
This gives the pcan port e.g. pcan32 which is used in the next step 

Launch the hand controller:
```bash
cd barrett_hand_ws
source devel/setup.bash
roslaunch bhand_controller bhand_controller.launch port:/dev/pcan32
```

Test the hand:

```bash
cd barrett_hand_ws
source devel/setup.bash
```

```python
import rospy
import bhand_controller.msg
import bhand_controller.srv

rospy.init_node('test')

hand_service_proxy = rospy.ServiceProxy('/bhand_node/actions', bhand_controller.srv.Actions)
custom_hand_service_proxy = rospy.ServiceProxy('/bhand_node/custom_actions', bhand_controller.srv.CustomActions)

hand_service_proxy(bhand_controller.msg.Service.INIT_HAND)

spread_angle = 0.2
custom_hand_service_proxy(bhand_controller.msg.Service.SET_SPREAD, [spread_angle,spread_angle])


# this calls the velocity` control, the first three are the target joint angles while the last value is the velocity
# positive velocity closes the hand, negative velocity opens the hand
custom_hand_service_proxy(bhand_controller.msg.Service.CLOSE_HAND_VELOCITY, [0.3, 0.3, 0.3, .1])

custom_hand_service_proxy(bhand_controller.msg.Service.CLOSE_HAND_VELOCITY, [0.01, 0.01, 0.01, -.1])
```
# Autonomous flight of a multi-rotor UAV to search for possible fires

![image](https://github.com/Sandrolek/info-project-clover/blob/master/drone_sim.jpg)

## The project team:
```
Lev Sinyukov - developer of the autonomous flight module
Alexander Polyakov - computer vision specialist
Egor Laptev - backend developer
```

### The following methods and technologies were used within the framework of this project:


```
3D simulator for robotics Gazebo - https://gazebosim.org/home;

Programmable quadcopter COEX Clover4 with PX4 flight stack and Raspberry Pi 4 as a control on-board computer;

Python programming;

Computer vision technology (library) OpenCV;

Cross-platform development environment for Node.js;

Framework for creating bots in telegrams Aiogram;

Working with databases (sqlite)


```

### Realization:

Import all the necessary libraries

```
import json
import requests
from datetime import datetime
from pprint import pprint
import math 

from pyzbar.pyzbar import decode

import rospy 
from clover import srv 
from sensor_msgs.msg import Image 
from std_srvs.srv import Trigger 

import cv2 
from cv_bridge import CvBridge
```

We initialize the ROS node and add the proxies necessary for the flight
```
rospy.init_node('flight')

get_telemetry = rospy.ServiceProxy('get_telemetry', srv.GetTelemetry) 
navigate = rospy.ServiceProxy('navigate', srv.Navigate)
land = rospy.ServiceProxy('land', Trigger) 

bridge = CvBridge() 
```

Creating publishers for debugging topics
```
color = rospy.Publisher('test', Image, queue_size=1)
hsv_topic = rospy.Publisher('hsv', Image, queue_size=1)
orig_topic = rospy.Publisher('orig', Image, queue_size=1)
```
Necessary constants
```
FLIGHT_HEIGHT = 1
FLIGHT_SPEED = 0.7
NUM_FRAME_CHECK = 10
MIN_CNT_AREA = 200
HSV_RED_FILTER = ((0, 120, 196), (20, 167, 255))
```

URL for publishing data to the database
```URL = "https://drone-stats.onrender.com/api/logs"```

### Work algorithm:
1. Takeoff. After takeoff, the starting coordinates are recorded and a qr code is determined, in which the coordinates of the route points are encrypted (read more about working with qr and computer vision - ```https://clover.coex.tech/ru/camera.html```)
2. An optimal route for the flight from the obtained points is developed.
3. Upon arrival at each point, functions are called to determine fires, the result of the study of the area in this position is saved.
4. After flying over all the points on the route, the drone returns to the starting position.
5. The flight report is sent to the server.
6. Landing the drone.
7. Transferring data from the server to the bot database.
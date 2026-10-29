# ROS 2 Mobile Robot: Sensors, Perception and Navigation

ROS 2 package (`erl1`) that builds a simulated mobile robot step by step during the lab classes: robot model, sensors, sensor fusion, ArUco marker detection, mapping, localization, navigation and task planning with PDDL. It also contains the first assignment: a node that finds ArUco markers and visits them one by one, tested in simulation and on a real robot.

> Lab classes and first assignment of the Experimental Robotics Laboratory course (2nd year, 1st semester), MSc Robotics Engineering, University of Genoa.

<p align="center">
  <img src="gifs/2wheel.gif" alt="Marker-visiting task in Gazebo" width="49%">
  <img src="gifs/lab.gif" alt="Marker-visiting task on the real robot" width="49%">
</p>

**Stack:** ROS 2 Jazzy · Gazebo Sim · RViz 2 · `ros_gz` bridge · `robot_localization` · OpenCV / ArUco · Nav2 · PlanSys2

## Assignment 1: ArUco marker visiting

The node `scripts/assignment1.py` makes the robot:
1. rotate on the spot until it has detected every ArUco marker in the scene;
2. visit the markers in ascending ID order: for each one, rotate until the marker is centred in the camera image, then drive towards it until it is close enough;
3. publish an image of the marker with a circle drawn around it on `camera/image_with_circle`;
4. go back to the starting position and move on to the next marker.

The behaviour is a state machine (`STARTING` → `SEARCH_FOR_MARKERS` → `SET_MARKER_GOAL` → `ROTATE_TO_MARKER` → `GO_TO_MARKER` → `ACQUIRE_IMAGE` → `COMING_BACK`), driven by the marker detections (`/aruco_detections`), the odometry (`/odom`) and the camera image.

| Type | Topic | Message |
|---|---|---|
| Subscriber | `/aruco_detections` | `aruco_opencv_msgs/ArucoDetection` |
| Subscriber | `/odom` | `nav_msgs/Odometry` |
| Subscriber | `camera/image/compressed` | `sensor_msgs/CompressedImage` |
| Publisher | `/cmd_vel` | `geometry_msgs/Twist` |
| Publisher | `camera/image_with_circle` | `sensor_msgs/CompressedImage` |

The default world, `worlds/aruco_boxes.sdf`, contains boxes with ArUco markers (`aruco_box/` model).

## Lab classes

The package was extended class by class:

| Step | What was added |
|---|---|
| Robot model | URDF/Xacro of a differential-drive robot, visualized in RViz and spawned in Gazebo with a motion plugin |
| Camera | RGB camera, later replaced by an RGB-D camera (depth image and point cloud) |
| IMU + EKF | IMU sensor, fused with the wheel odometry through an Extended Kalman Filter (`robot_localization`, `config/ekf.yaml`) |
| GPS | NavSat sensor |
| Lidar | 2D laser scanner (`/scan`, point cloud) |
| ArUco markers | Marker detection with OpenCV (`ros_aruco_opencv`) and Gazebo world with marker boxes |
| Mapping | 2D map of the environment, built and saved with Nav2 |
| Localization | Localization on the saved map with a particle filter |
| Navigation | Autonomous navigation to goals set in RViz, with local and global costmaps and paths |
| Task planning | PDDL planning integrated with ROS 2 through PlanSys2 |

## Package structure

```
erl1/
├── launch/erl1_simulation.launch.py   # Gazebo, robot spawn, ros_gz bridges, EKF, RViz, ArUco tracker
├── scripts/assignment1.py             # assignment 1 node
├── urdf/                              # robot model (URDF, Gazebo plugins, materials)
├── worlds/                            # Gazebo worlds (empty, custom, ArUco boxes)
├── aruco_box/                         # Gazebo model of a box with ArUco markers
├── config/ekf.yaml                    # EKF configuration
├── rviz/                              # RViz configurations
└── gifs/                              # demos
```

## Requirements

- ROS 2 Jazzy with Gazebo Sim and the `ros_gz` packages
- Course packages, cloned in the same workspace: [meshes](https://github.com/CarmineD8/meshes), [my_opencv_py](https://github.com/CarmineD8/my_opencv_py), [ros2_navigation](https://github.com/CarmineD8/ros2_navigation), [plansys_interface](https://github.com/CarmineD8/plansys_interface)
- ArUco: [ros_aruco_opencv](https://github.com/fictionlab/ros_aruco_opencv) as a package in the workspace, and [ros2-gazebo-aruco](https://github.com/SaxionMechatronics/ros2-gazebo-aruco) next to the workspace
- PDDL: `sudo apt install "ros-jazzy-plansys2-*"`
- Python packages: `requirements.txt`

## Usage

Build the workspace and start the simulation:

```bash
colcon build
source install/local_setup.bash
ros2 launch erl1 erl1_simulation.launch.py
```

The launch file starts Gazebo with the ArUco world, the robot, the bridges, the EKF, RViz and the ArUco tracker. The assignment node is commented out in the launch file: to run it, uncomment the line that adds `ass_node`.

Mapping, localization and navigation (with the simulation running):

```bash
ros2 launch ros2_navigation mapping.launch.py         # build the map (RViz fixed frame: map)
ros2 run nav2_map_server map_saver_cli -f my_map      # save it
ros2 launch ros2_navigation localization.launch.py    # localize on a saved map
ros2 launch ros2_navigation navigate.launch.py        # navigate with "2D Goal Pose" in RViz
```

In RViz, the particle cloud (`/particle_cloud`) and the pose estimate are visible after setting their *Reliability Policy* to *Best Effort*.

PDDL planning with PlanSys2:

```bash
ros2 launch plansys_interface distributed.launch.py
ros2 run plansys_interface get_plan_and_execute
ros2 run plansys2_terminal plansys2_terminal           # "get domain", "run", "quit"
```

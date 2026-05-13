# Simulation Instructions

This guide provides instructions on how to run the simulation environment for the UGV.

## Environment Setup
Please follow these guides in order and ensure they work before running the code below.
1. https://ardupilot.org/dev/docs/ros2.html
2. https://ardupilot.org/dev/docs/ros2-sitl.html
3. https://ardupilot.org/dev/docs/ros2-gazebo.html
4. https://ardupilot.org/dev/docs/ros2-cartographer-slam.html


## Prerequisites
Clone this repository, you can disregard ardu_ws/
```bash
git clone git@github.com:LI-DC-Long-Island-Deer-Chaser/ugv_ws.git
```

Build this workspace
```bash
colcon build
```

Make sure you have sourced your ROS 2 workspace:
```bash
source install/setup.bash
```

## Running the Simulation

1. **Launch the core SITL and Gazebo Simulation:**
   ```bash
   ros2 launch ugv_description ugv_sitl_gz.launch.py
   ```

2. **Launch SLAM (Cartographer) in Simulation:**
   To run Cartographer mapping in the simulated environment:
   ```bash
   ros2 launch ugv_description cartographer.launch.py
   ```

3. **Launch Mavproxy GCS to communicate with ardupilot SITL**
   To run Mavproxy for simulated Rover:
   ```bash
   mavproxy.py --master=tcp:127.0.0.1:5760 --sitl=127.0.0.1:5501 --out=127.0.0.1:14550
   ```

4. **Launch Navigation in Simulation:**
   To run Nav2 navigation stack in the simulation:
   ```bash
   ros2 launch ugv_description navigation.launch.py
   ```


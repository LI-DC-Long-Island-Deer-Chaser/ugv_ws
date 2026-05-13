# IRL (In Real Life) Instructions

This guide provides instructions on how to run the UGV hardware in real life.

## Prerequisites
Make sure you have sourced your ROS 2 workspace:
```bash
source install/setup.bash
```

Before running any node, ensure your hardware devices (such as the Lidar, flight controller, or Micro-ROS agent) are plugged into the correct ports. 
You will need to edit the respective launch files (`hw_slam.launch.py`, `hw_nav.launch.py`, and `hw_nav_amcl.launch.py`) to match your USB or serial ports (`/dev/ttyUSB0`, `/dev/ttyACM0`, etc.). 

If you get permission errors, you may need to grant read/write permissions to these ports:
```bash
sudo chmod a+rw /dev/ttyUSB*
sudo chmod a+rw /dev/ttyACM*
```

## Running Hardware

1. **Run SLAM on the physical robot:**
   To create a map using the physical robot's sensors (e.g., Slamtec Lidar):
   ```bash
   ros2 launch ugv_description hw_slam.launch.py
   ```

2. **Run Navigation on the physical robot:**
   To run the autonomous navigation stack (Nav2) on the hardware while slam is running. 
   ```bash
   ros2 launch ugv_description hw_nav.launch.py
   ```

## Saving a map after running SLAM

 1. Finish the Cartographer serialisation (writes .pbstream):
    ``` bash
      ros2 service call /finish_trajectory \
        cartographer_ros_msgs/srv/FinishTrajectory "{trajectory_id: 0}"
    ```
    ```bash
      ros2 service call /write_state \
        cartographer_ros_msgs/srv/WriteState \
        "{filename: '/home/odinroast/ugv_ws/src/ugv_description/config/irl/maps/my_map.pbstream', include_unfinished_submaps: true}"
    ```

 2. Convert .pbstream → ROS map (PGM + YAML):
    ``1bash
      ros2 run cartographer_ros cartographer_pbstream_to_ros_map \
        -pbstream_filename=/home/odinroast/ugv_ws/src/ugv_description/config/irl/maps/my_map.pbstream \
        -map_filestem=/home/odinroast/ugv_ws/src/ugv_description/config/irl/maps/my_map \
        -resolution=0.05
    ```

    This creates:
      config/irl/maps/my_map.pgm
      config/irl/maps/my_map.yaml

 3. Launch AMCL navigation on the saved map:
    ```bash
      ros2 launch ugv_description hw_nav.launch.py \
        map:=/home/odinroast/ugv_ws/src/ugv_description/config/irl/maps/my_map.yaml
    ```

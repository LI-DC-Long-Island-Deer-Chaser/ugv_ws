# Manual Control Instructions

This guide explains how to manually control the UGV using an Xbox Controller, allowing you to drive around and aim the camera. This is particularly useful while generating maps during SLAM.

## Running the Joystick Node
Before controlling the UGV, make sure you launch the joystick node:
```bash
ros2 launch ugv_description joystick.launch.py
```

## Controller Mapping and RC Override
When the joystick node (`xbox_teleop.py`) runs, it listens to the standard ROS `/joy` topic and converts the joystick inputs into a new message sent over the `/ap/joy` topic. ArduPilot's DDS (Data Distribution Service) interface subscribes to this `/ap/joy` topic and uses it to perform a direct **RC Override**.

The `/ap/joy` message contains an array of 8 float values, which map directly to ArduPilot's 8 PWM Radio Control channels (Channels 1 through 8). By setting specific indices in the `ap_joy.axes` array (using normalized values from `-1.0` to `1.0`), we override the PWM output of those channels on the flight controller. Any channel set to `NaN` (Not a Number) is ignored by ArduPilot, allowing other automated or default functions to run without interference.

### Primary Driving Controls
* **Throttle (Channel 1):** Controls the forward and backward speed of the UGV.
* **Steering (Channel 3):** Controls the left and right steering/turning angle.

### Camera Controls
* **Swivel Camera (Channel 8):** Controls the azimuth/swivel of the attached camera mechanism. 

Make sure the controller is paired/plugged in and properly recognized by your computer (usually `/dev/input/js0`) before launching it.

### Source code
xbox_teleop.py
```python
#!/usr/bin/env python3
"""Simple joystick to ArduPilot /ap/joy converter"""

import rclpy
from rclpy.node import Node
from sensor_msgs.msg import Joy


class JoyTeleop(Node):
    def __init__(self):
        super().__init__('joy_teleop')
        self.ap_joy_pub = self.create_publisher(Joy, '/ap/joy', 10)
        self.joy_sub = self.create_subscription(Joy, '/joy', self.joy_callback, 10)
        self.get_logger().info('Joy teleop started: axis 1 -> /ap/joy[0] (throttle), axis 3 -> /ap/joy[2] (steering)')
    
    def joy_callback(self, joy_msg: Joy):
        ap_joy = Joy()
        ap_joy.header.stamp = self.get_clock().now().to_msg()
        
        # Create array with at least 4 axes, all initialized to NaN
        ap_joy.axes = [float('nan')] * 8  # ArduPilot supports up to 8 RC channels
        
        if len(joy_msg.axes) > 3:
            ap_joy.axes[0] = -joy_msg.axes[3]   # Left stick Y -> throttle (channel 0)
            ap_joy.axes[2] = joy_msg.axes[1]   # Right stick X -> steering (channel 2)
        
        self.ap_joy_pub.publish(ap_joy)


def main(args=None):
    rclpy.init(args=args)
    node = JoyTeleop()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()


if __name__ == '__main__':
    main()
```
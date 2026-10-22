# Task: Simplify a ROS 2 Go-to-Pose Controller
holaaaaa


## Learning objectives

In this activity, you will build a direct closed-loop controller for Turtlesim. After completing it, you should be able to:

- identify the publisher, subscriber, topics, messages, and parameters of a ROS 2 node;
- use the pose received by a subscriber as feedback for a controller;
- calculate and publish a velocity command periodically;
- organize a node using the standard ROS 2 publisher and subscriber structure;
- use AI-generated code critically and simplify it according to clear requirements.

![](./Images/02_ROS2_tutorial/02_move_turtle.png)

## Direct control with `go_to_pose`

The objective is to create one node that receives a target pose through ROS parameters and directly controls the turtle:

```text
Target parameters
       |
       v
  go_to_pose
   |       |
   |       +--> /turtle1/cmd_vel
   +<---------- /turtle1/pose
```

The node closes the control loop locally:

```text
Read pose --> Calculate errors --> Update velocity --> Publish command
    ^                                                    |
    +----------------------------------------------------+
```

The subscriber provides feedback through `/turtle1/pose`. The controller compares that pose with the target, and the publisher sends velocity commands through `/turtle1/cmd_vel`.

This activity connects the basic publisher and subscriber templates from [02_ROS2_Tutorial.md](02_ROS2_Tutorial.md) with a complete robot-control problem. It focuses only on direct control; ROS 2 services are not required.

Once completed, the controller can be launched with different target parameters without modifying its code:

```bash
ros2 launch ros2_move_turtle go_to_pose.launch.py \
  target_x:=2.0 target_y:=8.0 target_theta_deg:=-90.0
```

The launch starts Turtlesim and the controller. All feedback calculations remain inside the `go_to_pose` node, making the control loop easy to observe and understand.

## Goal

Create a closed-loop controller that moves the turtlesim turtle to a target pose: first to the target position `(x, y)` and then to the target orientation. The node must use:

- a **subscriber** to read the current pose from `/turtle1/pose`;
- a **publisher** to send velocity commands to `/turtle1/cmd_vel`.

The provided [run_pose.py](../src/ros2_move_turtle/ros2_move_turtle/run_pose.py) works, but it was generated with AI from a general request. Without clear requirements and enough ROS 2 knowledge to evaluate the answer, AI-generated code may be correct but unnecessarily complex.

The original program is not optimal for this introductory activity because:

- it is almost 400 lines long;
- it divides a simple controller into many small methods;
- it uses a three-state state machine when two Boolean indicators are sufficient;
- it stores several redundant status variables;
- it separates the pose subscriber from the control calculations;
- it uses a manual loop with `spin_once()` instead of the standard `rclpy.spin(node)` structure;
- it includes extra lifecycle, result-management, and logging logic;
- its callbacks, timer, and manual execution loop make the program flow difficult for a beginner to follow.

The program does not create explicit threads; its complexity comes mainly from its length and organization.

Your task is to rewrite it using the simple publisher and subscriber templates introduced in [02_ROS2_Tutorial.md](02_ROS2_Tutorial.md). You may use AI, but your prompt and your review of its answer must enforce the requirements below.

Keep `run_pose.py` unchanged and complete the provided `go_to_pose_template.py` file. When your solution is finished, rename it to `go_to_pose.py` by removing `_template` from the filename. This final name is required by `setup.py` and `go_to_pose.launch.py`.

## Requirements

Your program must contain **fewer than 100 lines of Python code** and preserve the control equations, speed limits, tolerances, and angle normalization.

### 1. Use this simple structure

| Part | Responsibility |
| --- | --- |
| Constructor | Read the parameters, initialize a `Twist` command, and create the publisher, subscriber, and timer. |
| Subscriber callback | Use each received pose to calculate and store the velocity command. |
| Timer callback | Publish the stored command every 0.05 seconds (20 Hz). |
| `main()` | Initialize ROS, create the node, call `rclpy.spin(node)`, and clean up on exit. |

### 2. Simplify the controller

- Declare and read the target pose and tolerance parameters directly as attributes, for example: `self.target_x = self.declare_parameter('target_x', 8.0).value`.
- Keep the controller gains and maximum speeds as fixed constants; they do not need to be ROS parameters.
- Convert the target angle and angular tolerance from degrees to radians when reading them.
- Initialize `self.command = Twist()` so that the first command is zero.
- Use only two Boolean state indicators: `self.position_reached` and `self.finished`.
- In the subscriber callback, use the received `Pose` directly; do not store it or wait for an initial pose.
- While moving to the position, calculate the distance and heading errors. Keep the cosine factor that reduces forward speed when the turtle is not facing the target.
- Once the position tolerance is reached, command zero velocity and switch to final-orientation control.
- During final-orientation control, keep the linear velocity at zero and rotate through the shortest angular path.
- Once the angular tolerance is reached, store a zero command and set `self.finished = True`.
- The timer callback must only publish `self.command`; it must not calculate the motion.
- Use `rclpy.spin(node)`. Do not use motion loops, `spin_once()`, `sleep()`, extra motion states, or separate helper methods for every calculation.
- On Ctrl+C, cancel the timer, prevent further calculations, publish a final zero command, destroy the node, and shut down ROS safely.

After reaching the target, the node must remain active and continue publishing zero velocity until the user stops it.

## Test

Run turtlesim and check that:

1. The default target is `x = 8.0`, `y = 3.0`, and `theta = 90°`.
2. The turtle turns toward the target, moves to it, stops advancing, and then adjusts its final orientation.
3. The turtle remains stopped after reaching the complete pose.
4. Another target can be selected using ROS parameters.
5. Pressing Ctrl+C during motion stops the turtle safely.

## Delivery

This is an **individual activity**.

Record a short video showing:

- how you launch your `go_to_pose` program;
- the turtle moving to the target position and final orientation;
- the final result after the complete target pose has been reached.

Upload the video to the corresponding assignment in the Virtual Campus.

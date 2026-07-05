# terrian_traversal_robot
This is a **ROS 2 Humble + Gazebo Classic 11** simulation: a 4-wheel skid-steer robot traversing a hilly procedurally-generated heightmap terrain, with tip-over monitoring, optional SLAM+Nav2 autonomy.

## Build

```bash
cd ~/ros2_ws   # merge this src/ into your workspace
colcon build --packages-select skidsteer_terrain_sim
source install/setup.bash
```

## Run — basic sim

```bash
ros2 launch skidsteer_terrain_sim terrain_sim.launch.py
```
Wait for Gazebo to load, then in a second terminal:
```bash
ros2 launch skidsteer_terrain_sim traversal_tools.launch.py
```
This opens RViz and starts `traversal_monitor`, which logs `TIP-OVER RISK` warnings if pitch/roll exceed 30° (tunable via the `tip_over_threshold_deg` param).

## Drive it

Manual (recommended, to test slope handling):
```bash
ros2 run teleop_twist_keyboard teleop_twist_keyboard \
  --ros-args -r cmd_vel:=/skid_steer_controller/cmd_vel_unstamped
```

Or let it wander autonomously:
```bash
ros2 launch skidsteer_terrain_sim traversal_tools.launch.py auto_drive:=true
```

## SLAM + Nav2 autonomy

```bash
# Terminal 1
ros2 launch skidsteer_terrain_sim terrain_sim.launch.py

# Terminal 2 — SLAM (mapping) + full Nav2 stack + RViz
ros2 launch skidsteer_terrain_sim slam_navigation.launch.py
```
In RViz, use **2D Goal Pose** (or the Navigation2 panel) to send a goal — the robot plans around the eight static rock obstacles near the origin and drives there. Drive manually first with teleop to build map coverage before sending distant goals, since Nav2 can only plan through space slam_toolbox has already seen.

Save the map once satisfied:
```bash
ros2 run nav2_map_server map_saver_cli -f ~/my_terrain_map
```

## Other useful commands

```bash
# Run the standalone (ROS-free) tilt math unit tests
cd src/skidsteer_terrain_sim/scripts
pytest test_tilt_math.py -v

# Regenerate the heightmap (edit hill_centers/MAX_HEIGHT/FLAT_RADIUS_PX at the top first)
python3 tools/generate_heightmap.py

# One-shot everything (if you'd rather not run terrain_sim + tools separately)
ros2 launch skidsteer_terrain_sim full_autonomy.launch.py
ros2 launch skidsteer_terrain_sim autonomous_nav.launch.py
```

## Notes

- Robot spawns at `(0, 0, 0.35)`, matching the flattened disk at the heightmap center.
- IMU publishes on `/imu`.
- No AMCL/map_server — this uses live SLAM (slam_toolbox), not localization on a saved map.

Note: needs a real Ubuntu 22.04 + ROS 2 Humble + Gazebo machine — won't run in this sandbox.

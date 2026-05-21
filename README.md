# Visual Safe Human-to-Humanoid Motion Imitation

This repository contains the code, configuration, and experiment scripts for "Visual Safe Human-to-Humanoid Motion Imitation" (Cai et al., 2026). The project implements a vision-based upper-body imitation pipeline with a capsule-based CBF-QP safety filter for self-collision and human--robot collision avoidance.


Repository structure
--------------------
- `docker/` — Docker files and `requirements.txt` listing Python dependencies.
- `g1_configs/` — CycloneDDS and Unitree configuration scripts.
- `g1_real_ws/` — ROS2 workspace containing perception, controllers, CBF module, and experiment scripts:
  - `g1_real_ws/src/mujoco_g1/` — perception, retargeting, controller nodes and `launch/unified_pipeline.launch.py`.
  - `g1_real_ws/src/g1_cbf_ros2/g1_cbf/` — CBF safety filter implementation.
  - `g1_real_ws/src/real_g1/` — Unitree G1 SDK bridge.
  - `g1_real_ws/bdcc_exp/` — experiment scripts, replay/record utilities, sweeps, runs, and figures.
- `g1_repos/` — upstream Unitree repositories used during development.
- `third_party/` — MuJoCo models and simulation assets (`mujoco_menagerie`).

Prerequisites
--------------------

- The `Unitree SDK2` and the `Unitree ROS2` packages are downloaded and built beforehand.
- ROS 2 installed and sourced.
- Python 3 and `colcon` build tools installed.
- If using ZED hardware, the ZED SDK and CUDA must be installed.
- For MuJoCo-based simulation, a valid MuJoCo installation is required.

Installation
--------------------

Key steps

- Build the base Docker image (from `docker/`):

```bash
cd docker
docker build -t g1_real:base .
```

- Create and run the GPU-enabled container (example):

```bash
docker run -it --net=host --gpus all --name g1_real \
  -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix \
  -e ROS_DOMAIN_ID=0 -e ROS_LOCALHOST_ONLY=0 -e RMW_IMPLEMENTATION=rmw_cyclonedds_cpp \
  -v $PWD/g1_real_ws:/ws -v $PWD/g1_repos:/repos -v $PWD/g1_configs:/configs \
  -w /ws g1_real:base bash
```

- Inside the container: source ROS and install rosdeps:

```bash
source /opt/ros/humble/setup.bash
rosdep update
rosdep install --from-paths src --ignore-src -r -y
```

- Build this workspace:

```bash
source /opt/ros/humble/setup.bash
source /repos/unitree_ros2/install/setup.bash
cd /ws
colcon build --packages-select mujoco_g1 g1_cbf g1_description real_g1 --symlink-install
source install/setup.bash
```

- Source:

```bash
source /opt/ros/humble/setup.bash
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
export CMAKE_PREFIX_PATH=/opt/unitree_robotics:$CMAKE_PREFIX_PATH
export LD_LIBRARY_PATH=/opt/unitree_robotics/lib:$LD_LIBRARY_PATH
source /repos/unitree_ros2/install/setup.bash
source /ws/install/setup.bash
```

Launching the Unified Pipeline
--------------------

The main pipeline launch file is:

`g1_real_ws/src/mujoco_g1/launch/unified_pipeline.launch.py`

A typical simulation launch command is:

```bash
ros2 launch mujoco_g1 unified_pipeline.launch.py run_sim:=true run_real:=false use_cbf:=true rviz:=true
```

A typical real-robot launch command is:

```bash
ros2 launch mujoco_g1 unified_pipeline.launch.py run_sim:=false run_real:=true use_cbf:=true rviz:=true
```

#### Simulation vs Real Robot

- Use `run_sim:=true run_real:=false` for a full simulated pipeline.
- Use `run_sim:=false run_real:=true` for real robot execution.
- Both `run_sim` and `run_real` can be configured independently, but the launch file is built around one active path at a time.
- The launch file configures separate topic namespaces for simulation and real robot data, such as `/sim/joint_states` and `/real/joint_states`.

#### Notes
- If your robot hardware or environment differs, customize the launch arguments and topic names in `unified_pipeline.launch.py`.


Reproducing experiments
-----------------------
- Recording and replay utilities are under `g1_real_ws/bdcc_exp/scripts/` and example commands are in `g1_real_ws/bdcc_exp/scripts/command.md`.
- Example record command (paper):

```bash
python3 g1_real_ws/bdcc_exp/scripts/record/record_skeleton_segment.py \
  --scenario S1_self_collision \
  --outdir /ws/bdcc_exp/segments/S1_self_collision \
  --duration 50 --start-on-enter --record-diagnostics
```

- Example replay command (paper):

```bash
python3 g1_real_ws/bdcc_exp/scripts/replay/replay_skeleton_segment.py \
  --segment /ws/bdcc_exp/segments/S1_self_collision --publish-mode filtered \
  --start-delay 3.0 --replay-rate-hz 60 --time-scale 1.0
```

Data availability
-----------------
- Experimental datasets for `g1_real_ws/bdcc_exp/runs`, `g1_real_ws/bdcc_exp/sweeps`, and `g1_real_ws/bdcc_exp/segments` are archived on Zenodo:
  - Record: https://zenodo.org/records/20322803
  - DOI: `10.5281/zenodo.20322802`

Citation
--------
Please cite the manuscript when using this code. Example BibTeX (preprint):

```bibtex
@article{cai2026vision,
  title = {Visual Safe Human-to-Humanoid Motion Imitation},
  author = {Cai, Wenqi and Abanes, John and Evangeliou, Nikolaos and Tzes, Anthony},
  year = {2026},
  note = {Preprint (no DOI)}
}
```


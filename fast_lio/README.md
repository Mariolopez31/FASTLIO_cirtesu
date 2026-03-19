# FASTLIO_cirtesu

ROS 2 package/repository used in CIRTESU as a modified version of FAST-LIO for LiDAR-Inertial odometry and mapping.

This repository is based on the ROS 2 branch of the official FAST-LIO project and is maintained as a CIRTESU-specific adaptation for integration within our robotics stacks.

## Upstream

This repository is derived from:

- Official FAST-LIO repository: `hku-mars/FAST_LIO`
- ROS 2 branch used as base for this adaptation

The goal of this repository is not to replace the original project, but to provide a version adapted to the CIRTESU environment and workflow.

## Notes about this version

Compared to the original project, this repository may include:

* ROS 2 integration fixes or adjustments
* CIRTESU-specific configuration changes
* adaptations for our simulation and/or deployment workflow
* integration with other CIRTESU packages and drivers

## Prerequisites

Recommended environment:

* Ubuntu 22.04
* ROS 2 Humble

### System dependencies

The following libraries/packages are expected to be installed in the system:

* PCL
* Eigen
* `pcl_ros`
* `pcl_conversions`

Typical installation:

```bash
sudo apt update
sudo apt install \
  libpcl-dev \
  libeigen3-dev \
  ros-humble-pcl-ros \
  ros-humble-pcl-conversions
```

### External dependencies to install separately

The following dependencies are **not included here as workspace packages** and must be installed separately.

#### Sophus

A compatible Sophus version is required.

Example installation:

```bash
git clone https://github.com/strasdat/Sophus.git
cd Sophus
git checkout 1.22.10
mkdir build && cd build
cmake .. -DSOPHUS_USE_BASIC_LOGGING=ON
make -j
sudo make install
```

Note: newer Sophus versions may introduce additional logging dependencies. In some setups it may be necessary to add:

```cmake
add_compile_definitions(SOPHUS_USE_BASIC_LOGGING)
```

#### GTSAM

GTSAM is required by some extended modules/workflows derived from the FAST-LIO ROS 2 ecosystem, especially when using pose graph or relocalization-related components.

Install it separately according to your system and version requirements.

#### Livox-SDK2

If your setup requires Livox SDK2, install it separately:

```bash
git clone https://github.com/Livox-SDK/Livox-SDK2.git
cd Livox-SDK2
mkdir build && cd build
cmake .. && make -j
sudo make install
```

### Workspace dependencies

The following repository is expected to be available in the same ROS 2 workspace:

* `livox_ros_driver2`

In the CIRTESU workspace this is typically already included as a submodule, so it usually does not need to be cloned manually.

### Internal dependency

This repository also uses:

* `ikd-Tree` as an internal submodule

## Clone

If cloning this repository directly:

```bash
git clone --recursive https://github.com/Mariolopez31/FASTLIO_cirtesu.git
```

If already cloned without submodules:

```bash
git submodule update --init --recursive
```

## Build

From the workspace root:

```bash
cd ~/your_ws
source /opt/ros/humble/setup.bash
git submodule update --init --recursive
colcon build --packages-select fast_lio
source install/setup.bash
```

If `livox_ros_driver2` has not been built yet in the same workspace, build it as well:

```bash
colcon build --packages-select livox_ros_driver2 fast_lio
```

Configuration files should be adapted depending on the LiDAR, IMU, simulator, and CIRTESU-specific setup.

## Keeping this repository updated

This repository tracks the official FAST-LIO project through an `upstream` remote.

Typical workflow:

* `origin`: CIRTESU fork
* `upstream`: official FAST-LIO repository

To bring changes from upstream:

```bash
git fetch upstream
git checkout ROS2
git merge upstream/ROS2
```

Then push the updated branch to the CIRTESU fork:

```bash
git push
```

## Submodule workflow

When this repository is used as a submodule inside a parent repository:

1. Commit and push changes inside `FASTLIO_cirtesu`
2. Go back to the parent repository
3. Commit the updated submodule reference there

Example:

```bash
cd src/FASTLIO_cirtesu
git add .
git commit -m "Update FASTLIO_cirtesu"
git push
```

Then in the parent repository:

```bash
git add src/FASTLIO_cirtesu
git commit -m "Update FASTLIO_cirtesu submodule"
git push
```

## Acknowledgment

Please refer to the original FAST-LIO project for the main algorithm, publications, and broader context.

This repository is only a CIRTESU-oriented adaptation of that work.

```
```

# FASTLIO_cirtesu

CIRTESU-specific ROS 2 adaptation of FAST-LIO for LiDAR-Inertial odometry, mapping, and relocalization workflows.

This repository is based on the ROS 2 branch of FAST-LIO and adapted for the CIRTESU environment, including integration with our simulation stack, Livox sensors, and external relocalization components.

## Upstream

This work is derived from:

- Official FAST-LIO repository: `hku-mars/FAST_LIO`
- ROS 2 branch used as the base for this adaptation

Please refer to the original FAST-LIO project for the original algorithm, publications, and upstream developments.

## Relocalization stack

The relocalization stack used together with this repository comes from the FASTLIO2_ROS2 project by liangheming, which includes the localizer and interface packages as part of a broader ROS 2 FAST-LIO ecosystem. In that upstream project, localizer provides online relocalization based on a coarse-to-fine ICP pipeline, while interface provides the service definitions used by that workflow.

Repository reference:

https://github.com/liangheming/FASTLIO2_ROS2

In CIRTESU, these packages were adapted so they can be used together with the FAST-LIO version maintained in this repository, instead of depending directly on the full original FASTLIO2_ROS2 stack. In other words, the relocalization services and node structure come from that project, but the code was modified here to work with the official FAST-LIO ROS 2 adaptation used in our workflow.

If relocalization is required, the packages that must be present in the workspace are:

- interface
- localizer

These should be built before fast_lio.

## Recommended environment

- Ubuntu 22.04
- ROS 2 Humble

## Prerequisites and build order

Before building this repository, install the required system and external dependencies first.

If you are using the relocalization workflow, the recommended build order is:

1. `interface`
2. `localizer`
3. `livox_ros_driver2`
4. `fast_lio`

It is recommended to build them separately, especially during the first setup.

## System dependencies

Install the common ROS 2 and point cloud dependencies:

```bash
sudo apt update
sudo apt install \
  libpcl-dev \
  libeigen3-dev \
  ros-humble-pcl-ros \
  ros-humble-pcl-conversions \
  ros-humble-tf2-ros \
  ros-humble-geometry-msgs \
  ros-humble-nav-msgs \
  ros-humble-sensor-msgs \
  ros-humble-std-msgs \
  ros-humble-std-srvs \
  ros-humble-visualization-msgs
````

Depending on the relocalizer version, `yaml-cpp` may also be needed.

## External dependencies

### Sophus

A compatible Sophus version is required.

Example:

```bash
git clone https://github.com/strasdat/Sophus.git
cd Sophus
git checkout 1.22.10
mkdir build && cd build
cmake .. -DSOPHUS_USE_BASIC_LOGGING=ON
make -j
sudo make install
```

In some setups it may also be necessary to compile with:

```cmake
add_compile_definitions(SOPHUS_USE_BASIC_LOGGING)
```

### Livox-SDK2

If your setup uses Livox SDK2 directly, install it separately:

```bash
git clone https://github.com/Livox-SDK/Livox-SDK2.git
cd Livox-SDK2
mkdir build && cd build
cmake ..
make -j
sudo make install
```

## Workspace dependencies

The following packages are typically expected in the same ROS 2 workspace:

* `livox_ros_driver2`
* `localizer` if relocalization is used
* `interface` if relocalization is used

### Important note

When building livox_ros_driver2, the build script expects the ROS distribution to be passed explicitly as an argument. In the upstream instructions for FASTLIO2_ROS2, the driver is built with:

```bash
source /opt/ros/humble/setup.sh
./build.sh humble
```

So the relevant part is not just sourcing ROS 2 Humble, but also passing humble explicitly to the driver build script.

## Internal dependency

This repository uses:

* `ikd-Tree` as an internal submodule

If cloning manually, make sure submodules are initialized.

## Clone

If cloning directly:

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
```

If using relocalization, build in this order:

```bash
colcon build --packages-select interface
source install/setup.bash

colcon build --packages-select localizer
source install/setup.bash

colcon build --packages-select livox_ros_driver2 fast_lio
source install/setup.bash
```

If you only want FAST-LIO mapping, you can build only:

```bash
colcon build --packages-select livox_ros_driver2 fast_lio
source install/setup.bash
```

## Mapping

To generate a map `.pcd`, FAST-LIO must be launched with map saving enabled.

In the FAST-LIO YAML configuration, enable:

```yaml
pcd_save:
  pcd_save_en: true
```

You should also check the publication settings, especially if you want to visualize the map cloud:

```yaml
publish:
  map_en: true
```

### Important note about TF

Correct TF configuration is essential before generating a map.

In practice, you must make sure that:

* the LiDAR cloud is accumulated in the correct global frame
* the `map` frame is consistent in your setup
* the transforms used during mapping are the intended ones

If TF is wrong, the generated `.pcd` will also be wrong.

This is especially important in simulation, where the global transform may come from the simulator or from a dedicated localization setup.

## Relocalization

Once a reference map has been created, it can be used for relocalization.

The relocalization workflow typically consists of:

1. launching FAST-LIO
2. launching `localizer_node`
3. loading a reference `.pcd`
4. calling the relocalization service with:

   * the map path
   * an initial pose guess

```bash
ros2 service call /localizer/relocalize interface/srv/Relocalize \
"{pcd_path: '/absolute/path/to/map.pcd', x: 0.0, y: 0.0, z: 0.0, yaw: 0.0, pitch: 0.0, roll: 0.0}"
```

Relocalization requires a sufficiently good initial pose guess. If the initial guess is too far from the true pose, ICP-based alignment may fail.

### Which map is used?

The map used for relocalization is the `.pcd` passed through the relocalization service request. The relocalizer does not choose a map automatically.

### Map publication

The localizer already publishes the loaded reference map as a point cloud topic. In the current implementation this topic is:

```text
/localizer/map_cloud
```

This can be visualized in RViz together with the live FAST-LIO cloud to check whether alignment is correct.

### Checking convergence

A practical way to check whether relocalization has converged is:

* the relocalization validity service returns success

```bash
ros2 service call /localizer/relocalize_check interface/srv/IsValid "{code: 0}"
```

* the live cloud aligns well with the reference map in RViz
* the estimated pose becomes stable

In simulation, this can also be compared against simulator ground truth.

## Common pitfalls

* forgetting to build `interface` and `localizer` before the relocalization workflow
* building everything at once during the first setup
* wrong TF tree during map generation
* poor initial guess for relocalization
* using the wrong `.pcd` file as reference
* duplicating very large `.pcd` files unnecessarily in the install tree

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

Then push the updated branch:

```bash
git push
```

## Submodule workflow

When this repository is used as a submodule inside a parent repository:

1. commit and push changes inside `FASTLIO_cirtesu`
2. return to the parent repository
3. commit the updated submodule pointer there

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

Please refer to the original FAST-LIO project for the main algorithm, publications, and original implementation context.

This repository is only a CIRTESU-oriented adaptation and integration layer built on top of that work.

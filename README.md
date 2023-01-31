## Flightmare playground

Simple docker container with a ready to use installation of Flightmare, according to the instructions in https://github.com/uzh-rpg/flightmare/wiki/Install-with-ROS


## Usage

Make sure you download the latest binary from https://github.com/uzh-rpg/flightmare/releases and install it within the folder of the `flightrender` package in the Flightmare submodule.

Later, launch the container, build with `catkin_make`, source it and launch:

```bash
roslaunch flightros rotors_gazebo.launch
```
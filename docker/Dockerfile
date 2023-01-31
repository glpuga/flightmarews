ARG BASEIMG=ubuntu:bionic
FROM $BASEIMG

# Set ROS distribution
ARG ROS_DISTRO=melodic

# Base system tools
RUN apt update \
 && apt install -y \
    build-essential \
    ccache \
    clang-format \
    cmake \
    cppcheck \
    curl \
    git \
    locate \
    lsb-release \
    mc \
    pkg-config \
    python-pip \
    python3-pip \
    software-properties-common \
    sudo \
    tmux \
    wget \
 && apt clean

RUN export DEBIAN_FRONTEND=noninteractive \
 && apt update \
 && apt install -y \
    tzdata \
 && ln -fs /usr/share/zoneinfo/America/Los_Angeles /etc/localtime \
 && dpkg-reconfigure --frontend noninteractive tzdata \
 && apt clean

# Missing nvidia libraries
RUN sudo apt update \
 && sudo apt install -y \
    libnvidia-gl-440 \
 && apt clean

# Base ROS melodic system
COPY keys/ros.key /tmp/ros.key
RUN /bin/sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list' \
 && apt-key add /tmp/ros.key \
 && apt update \
 && apt install -y \
    python-rosdep \
    python-catkin-lint \
    python-catkin-tools \
    ros-${ROS_DISTRO}-desktop-full \
 && rosdep init \
 && apt clean

# Get Gazebo 9.
COPY keys/gazebo.key /tmp/gazebo.key
RUN /bin/sh -c 'echo "deb http://packages.osrfoundation.org/gazebo/ubuntu-stable $(lsb_release -cs) main" > /etc/apt/sources.list.d/gazebo-stable.list' \
 && apt-key add /tmp/gazebo.key \
 && apt update \
 && apt upgrade -y \
 && apt clean
# This seems to be needed to launch Gazebo on this setup
# or it fails to launch with a symbol error on gzserver
RUN apt-get update && apt-get upgrade -y libignition-math4

RUN apt update \
 && apt install -y \
    build-essential \
    cmake \
    libzmqpp-dev \
    libopencv-dev \
    libgoogle-glog-dev \
    protobuf-compiler \
    ros-$ROS_DISTRO-octomap-msgs \
    ros-$ROS_DISTRO-octomap-ros \
    ros-$ROS_DISTRO-joy \
    python-vcstool \
 && apt clean

# Set USER and GROUP
ARG USER=developer
ARG GROUP=developer

# Add a user with the same user_id as the user outside the container
# Requires a docker build argument `user_id`.
RUN curl -SsL https://github.com/boxboat/fixuid/releases/download/v0.4/fixuid-0.4-linux-amd64.tar.gz | tar -C /usr/local/bin -xzf - && \
    chown root:root /usr/local/bin/fixuid && \
    chmod 4755 /usr/local/bin/fixuid && \
    mkdir -p /etc/fixuid && \
    printf "user: $USER\ngroup: $GROUP\n" > /etc/fixuid/config.yml

RUN addgroup --gid 1000 $USER && \
    adduser --uid 1000 --ingroup $USER --home /home/$USER --shell /bin/sh --disabled-password --gecos "" $USER

RUN adduser $USER sudo \
 && echo "$USER ALL=NOPASSWD: ALL" >> /etc/sudoers.d/$USER

RUN mkdir -p /home/$USER/ws/src \
 && chown -R $USER: home/$USER/ws

# # Add extra ROS packages here
# RUN apt update \
#  && apt install -y \
#     <some package> \
#  && apt clean

# # Install extra tools tools
# RUN apt update \
#  && apt install -y \
#     <some package> \
#  && apt clean

# Commands below run as the developer user.
USER $USER:$GROUP

RUN rosdep update

# When running a container start in the developer's home folder.
WORKDIR /home/$USER

COPY dependencies.yaml /tmp/dependencies.yaml
RUN cd ~/ws/src/ \
 && mkdir flightmare_deps \
 && cd flightmare_deps \
 && vcs-import < /tmp/dependencies.yaml

# Source all the needed environment files.
RUN /bin/sh -c 'echo ". /opt/ros/${ROS_DISTRO}/setup.bash" >> ~/.bashrc' \
 && /bin/sh -c 'echo ". /usr/share/gazebo/setup.sh" >> ~/.bashrc' \
 && /bin/sh -c 'echo "set-option -g default-shell /bin/bash" >> ~/.tmux.conf' \
 && /bin/sh -c 'echo "export FLIGHTMARE_PATH=~/ws/src/flightmare_deps/RPG_Flightmare" >> ~/.bashrc'

#
# Ccache

# install ccache in the path
RUN /bin/sh -c 'echo "export PATH=/usr/lib/ccache:\$PATH" >> ~/.bashrc'
# sen envvars so that catkin_make finds ccache instead of using absolute paths
ENV CC=gcc
ENV GXX=g++
# Create .ccache cache dir to mount a volume there
RUN mkdir -p /home/$USER/.ccache

WORKDIR /home/$USER/ws

ENTRYPOINT ["fixuid"]

CMD ["/bin/bash"]

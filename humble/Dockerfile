# Copyright 2020-2023 Tiryoh<tiryoh@gmail.com>
# 
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
# 
#     http://www.apache.org/licenses/LICENSE-2.0
# 
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# 
# This Dockerfile is based on https://github.com/AtsushiSaito/docker-ubuntu-sweb
# which is released under the Apache-2.0 license.

FROM ubuntu:jammy-20230816

ARG TARGETPLATFORM
LABEL maintainer="Tiryoh<tiryoh@gmail.com>"

SHELL ["/bin/bash", "-c"]

# Upgrade OS
RUN apt-get update -q && \
    DEBIAN_FRONTEND=noninteractive apt-get upgrade -y && \
    apt-get autoclean && \
    apt-get autoremove && \
    rm -rf /var/lib/apt/lists/*

# Install Ubuntu Mate desktop
RUN apt-get update -q && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y \
        ubuntu-mate-desktop && \
    apt-get autoclean && \
    apt-get autoremove && \
    rm -rf /var/lib/apt/lists/*

# Add Package
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y \
        tigervnc-standalone-server tigervnc-common \
        supervisor wget curl gosu git sudo python3-pip tini \
        build-essential vim sudo lsb-release locales \
        bash-completion tzdata terminator && \
    apt-get autoclean && \
    apt-get autoremove && \
    rm -rf /var/lib/apt/lists/*

# noVNC and Websockify
RUN git clone https://github.com/AtsushiSaito/noVNC.git -b add_clipboard_support /usr/lib/novnc
RUN pip install git+https://github.com/novnc/websockify.git@v0.10.0
RUN ln -s /usr/lib/novnc/vnc.html /usr/lib/novnc/index.html

# Set remote resize function enabled by default
RUN sed -i "s/UI.initSetting('resize', 'off');/UI.initSetting('resize', 'remote');/g" /usr/lib/novnc/app/ui.js

# Disable auto update and crash report
RUN sed -i 's/Prompt=.*/Prompt=never/' /etc/update-manager/release-upgrades
RUN sed -i 's/enabled=1/enabled=0/g' /etc/default/apport

# Enable apt-get completion
RUN rm /etc/apt/apt.conf.d/docker-clean

# Install Firefox
RUN DEBIAN_FRONTEND=noninteractive add-apt-repository ppa:mozillateam/ppa -y && \
    echo 'Package: *' > /etc/apt/preferences.d/mozilla-firefox && \
    echo 'Pin: release o=LP-PPA-mozillateam' >> /etc/apt/preferences.d/mozilla-firefox && \
    echo 'Pin-Priority: 1001' >> /etc/apt/preferences.d/mozilla-firefox && \
    apt-get update -q && \
    apt-get install -y \
    firefox && \
    apt-get autoclean && \
    apt-get autoremove && \
    rm -rf /var/lib/apt/lists/*

# Install VSCodium
RUN wget -qO - https://gitlab.com/paulcarroty/vscodium-deb-rpm-repo/raw/master/pub.gpg \
    | gpg --dearmor \
    | dd of=/usr/share/keyrings/vscodium-archive-keyring.gpg && \
    echo 'deb [ signed-by=/usr/share/keyrings/vscodium-archive-keyring.gpg ] https://download.vscodium.com/debs vscodium main' \
    | tee /etc/apt/sources.list.d/vscodium.list && \
    apt-get update -q && \
    apt-get install -y codium && \
    apt-get autoclean && \
    apt-get autoremove && \
    rm -rf /var/lib/apt/lists/*

# Install ROS
ENV ROS_DISTRO humble
# desktop or ros-base
ARG INSTALL_PACKAGE=desktop

RUN apt-get update -q && \
    apt-get install -y curl gnupg2 lsb-release && \
    curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(lsb_release -cs) main" | tee /etc/apt/sources.list.d/ros2.list > /dev/null && \
    apt-get update -q && \
    apt-get install -y ros-${ROS_DISTRO}-${INSTALL_PACKAGE} \
    python3-argcomplete \
    python3-colcon-common-extensions \
    python3-rosdep python3-vcstool && \
    rosdep init && \
    rm -rf /var/lib/apt/lists/*

# Install ros-dev-tools
RUN apt-get update && apt-get install -y \
    ros-dev-tools \
    && rm -rf /var/lib/apt/lists/*


# Install required TurtleBot3 packages and Gazebo
RUN apt-get update && apt-get install -y \
    ros-${ROS_DISTRO}-gazebo-ros-pkgs \
    ros-${ROS_DISTRO}-turtlebot3-msgs \
    ros-${ROS_DISTRO}-turtlebot3 \
    && rm -rf /var/lib/apt/lists/*

# Set up turtlebot workspace
RUN mkdir -p /home/ubuntu/turtlebot3_ws/src

# Clone turtlebot3_simulations
WORKDIR /home/ubuntu/turtlebot3_ws/src
RUN git clone -b ${ROS_DISTRO}-devel https://github.com/ROBOTIS-GIT/turtlebot3_simulations.git


# Source gazebo setup before building the workspace
RUN echo "source /usr/share/gazebo/setup.sh" >> ~/.bashrc

# Build the workspace
WORKDIR /home/ubuntu/turtlebot3_ws

RUN /bin/bash -c ". /opt/ros/${ROS_DISTRO}/setup.bash; colcon build --symlink-install"

RUN rosdep update

# Install simulation package only on amd64
# Not ready for arm64 for now (July 28th, 2020)
# https://github.com/Tiryoh/docker-ros2-desktop-vnc/pull/56#issuecomment-1196359860
#RUN if [ "$TARGETPLATFORM" = "linux/amd64" ]; then \
#    apt-get update -q && \
#    apt-get install -y \
#    ros-${ROS_DISTRO}-gazebo-ros-pkgs \
#    ros-${ROS_DISTRO}-ros-ign && \
#    rm -rf /var/lib/apt/lists/*; \
#    fi

# Set up colcon workspace
RUN mkdir -p /home/ubuntu/colcon_ws/src
WORKDIR /home/ubuntu/colcon_ws/src

# Install MapViz
RUN git clone -b ros2-devel https://github.com/swri-robotics/mapviz.git

# Dependencies for mapviz
RUN apt-get update
RUN apt-get install ros-$ROS_DISTRO-marti-common-msgs 
RUN apt-get install ros-$ROS_DISTRO-marti-visualization-msgs 
RUN apt-get install ros-$ROS_DISTRO-marti-sensor-msgs 

RUN apt-get install -y ros-$ROS_DISTRO-swri-transform-util 
RUN apt-get install -y ros-$ROS_DISTRO-swri-route-util 
RUN apt-get install -y ros-$ROS_DISTRO-swri-image-util 

# Additional ROS Packages and tools for course
RUN apt-get install -y ros-$ROS_DISTRO-vision-msgs

#WORKDIR /root/colcon_ws/
#RUN rosdep init
#RUN rosdep update
#RUN rosdep install --from-paths src --ignore-src

# Build the workspace
WORKDIR /home/ubuntu/colcon_ws

RUN /bin/bash -c ". /opt/ros/${ROS_DISTRO}/setup.bash; colcon build --symlink-install"


# Sublime
RUN curl -fsSL https://download.sublimetext.com/sublimehq-pub.gpg | sudo apt-key add - 
RUN add-apt-repository "deb https://download.sublimetext.com/ apt/stable/" 
RUN apt install sublime-text


# Docker
# Add Docker's official GPG key:
RUN apt-get update
RUN apt-get install ca-certificates curl gnupg
RUN install -m 0755 -d /etc/apt/keyrings
RUN curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
RUN chmod a+r /etc/apt/keyrings/docker.gpg

# Add the repository to Apt sources:
RUN echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
RUN sudo apt-get update

RUN apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# The ultralytics library for YoloV8
RUN pip install ultralytics

WORKDIR /root

ENV USER ubuntu
ENV PASSWORD tarosbox

COPY ./entrypoint.sh /
ENTRYPOINT [ "/bin/bash", "-c", "/entrypoint.sh" ]




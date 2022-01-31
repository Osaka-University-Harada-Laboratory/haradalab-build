FROM ros:melodic-robot

# nvidia-container-runtime
ENV NVIDIA_VISIBLE_DEVICES ${NVIDIA_VISIBLE_DEVICES:-all}
ENV NVIDIA_DRIVER_CAPABILITIES ${NVIDIA_DRIVER_CAPABILITIES:+$NVIDIA_DRIVER_CAPABILITIES,}graphics

COPY terminator_config /tmp/terminator_config

# install GLX-Gears
RUN apt update && apt install -y --no-install-recommends mesa-utils x11-apps && rm -rf /var/lib/apt/lists/*

# install editors
RUN apt update \
&& apt install -y --no-install-recommends terminator dbus dbus-x11 vim \
&& mkdir -p /root/.config/terminator \
&& touch /root/.config/terminator/config

# create a user
ARG UID=1000
ARG DOCKER_USER=docker
ARG DOCKER_PASSWORD=docker
RUN useradd -m --uid ${UID} --groups sudo ${DOCKER_USER} \
&& echo ${DOCKER_USER}:${DOCKER_PASSWORD} | chpasswd

# copy default terminator config
RUN mkdir -p /home/${DOCKER_USER}/.config/terminator \
&& mv /tmp/terminator_config /home/${DOCKER_USER}/.config/terminator/config \
&& chown -R ${DOCKER_USER}:${DOCKER_USER} /home/${DOCKER_USER}/.config

# install ROS
RUN sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
RUN apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
RUN apt-get update && apt-get install -y --allow-downgrades --allow-remove-essential --allow-change-held-packages \
libpcap-dev \
lsb-release \
git \
wget \
libopenblas-dev \
gstreamer1.0-tools libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev libgstreamer-plugins-good1.0-dev \
ros-melodic-desktop-full python-rosinstall python-rosinstall-generator python-wstool build-essential python-rosdep \
ros-melodic-socketcan-bridge \
ros-melodic-geodesy \
ros-melodic-catkin \
python-catkin-tools \
libboost-dev \
libboost-python-dev \
libboost-system-dev \
libboost-all-dev \
libprotobuf-dev \
protobuf-compiler \
python-setuptools \
python-pip \
python-tk \
# graspit dependencies
libqt4-dev \
libqt4-opengl-dev \
libqt4-sql-psql \
libcoin80-dev \
libsoqt4-dev \
libblas-dev \
liblapack-dev \
libqhull-dev \
libeigen3-dev \
locate \
&& apt-get clean && rm -rf /var/lib/apt/lists/*

# create my workspace for ros
RUN /bin/bash -c "source /opt/ros/melodic/setup.bash \
                  && mkdir -p /home/${DOCKER_USER}/ros_ws/src \
                  && cd /home/${DOCKER_USER}/ros_ws \
                  && catkin init"

# install important ros packages
RUN /bin/bash -c "cd /home/${DOCKER_USER}/ros_ws/src \
                  && git clone https://github.com/graspit-simulator/graspit.git\
                  && git clone https://github.com/shadow-robot/sr_common.git"

# update rosdep and install dependencies
RUN cd /home/${DOCKER_USER}/ros_ws \
&& rosinstall ~/ros_ws/src /opt/ros/melodic \
&& apt update \
&& rosdep fix-permissions \
&& rosdep update \
&& rosdep install --from-paths src --ignore-src --rosdistro=melodic -y \
&& rm -rf /var/lib/apt/lists/*

# catkin build
RUN /bin/bash -c "source /opt/ros/melodic/setup.bash \
                  && cd /home/${DOCKER_USER}/ros_ws \
                  && rm -rf build \
                  && catkin build"

# build graspit!
RUN /bin/bash -c "cd /home/${DOCKER_USER}/ros_ws/src/graspit \
                  && export GRASPIT=$PWD \
                  && mkdir build \
                  && cd build \
                  && cmake .. \
                  && make -j5"

# chown
RUN /bin/bash -c "cd /home/${DOCKER_USER}/ \
                  && chown -R ${DOCKER_USER}:${DOCKER_USER} ros_ws"

# make path to ROS packages
RUN echo "source /home/${DOCKER_USER}/ros_ws/devel/setup.bash" >> /home/${DOCKER_USER}/.bashrc

# change default shell
RUN chsh -s /bin/bash
USER ${DOCKER_USER}


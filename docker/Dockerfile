ARG BASE_IMAGE=nvidia/cudagl:11.3.0-devel-ubuntu20.04
ARG ros_distro=noetic
ARG python_version=python3

FROM ${BASE_IMAGE}

ENV DEBIAN_FRONTEND=noninteractive

SHELL ["/bin/bash", "-c"]

RUN apt-get update -y && \
    apt-get install -y --no-install-recommends \
        wget \
        curl \
        git \
        gnupg2 \
        ca-certificates \
        lsb-release && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# ROS
ARG ros_distro
ENV ROS_DISTRO=${ros_distro}
## https://wiki.ros.org/${ROS_DISTRO}/Installation/Ubuntu
### Setup source.list
RUN sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
### Setup keys
RUN curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | apt-key add -
###  Install ros-${ROS_DISTRO}-base
RUN apt-get update -y && \
    apt-get install -y --no-install-recommends \
        ros-${ROS_DISTRO}-desktop-full && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
###  Dependencies for building packages
ARG python_version
ENV PYTHON_VERSION=${python_version}
RUN apt-get update -y && \
    apt-get install -y --no-install-recommends \
        ${PYTHON_VERSION}-pip \
        ${PYTHON_VERSION}-rosdep \
        ${PYTHON_VERSION}-rosinstall \
        ${PYTHON_VERSION}-rosinstall-generator \
        ${PYTHON_VERSION}-wstool \
        ${PYTHON_VERSION}-catkin-tools \
        build-essential && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
RUN rosdep init && \
    rosdep update

# install ROS sensor pacakge
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        ros-${ROS_DISTRO}-navigation \
        ros-${ROS_DISTRO}-joy \
        ros-${ROS_DISTRO}-gmapping && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# external ROS setting
WORKDIR /root
RUN mkdir -p external_catkin_ws/src && \
    cd external_catkin_ws/src && \
    git clone https://github.com/autonomylab/create_robot.git -b melodic && \
    git clone https://github.com/ryuichiueda/emcl && \
    cd /root/external_catkin_ws && \
    rm -r src/create_robot/create_bringup && \
    rm -r src/create_robot/create_driver && \
    rm -r src/create_robot/create_robot && \
    rosdep update && \
    apt-get update && \
    rosdep install --from-paths src -i -r -y && \
    cd ~/external_catkin_ws && \
    bash -c "source /opt/ros/${ROS_DISTRO}/setup.bash; catkin build"

# install PyTorch
RUN pip install torch==1.10.1+cu111 torchvision==0.11.2+cu111 torchaudio==0.10.1 -f https://download.pytorch.org/whl/torch_stable.html

# install YOLOv8
RUN pip install ultralytics

WORKDIR /root/

COPY ./docker/scripts/. /root/scripts/
RUN chmod -R +x /root/scripts/*

RUN echo "ROOMBA_HSOTNAME=\"TMP_HOSTNAME\"" >> /root/.bashrc
RUN echo "source /root/scripts/initialize-bash-shell.sh" > /root/.bashrc

ENTRYPOINT []

CMD ["/bin/bash"]

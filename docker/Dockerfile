FROM osrf/ros:noetic-desktop-full

ARG UBUNTU_DISTRO=focal
ARG MAX_JOBS=32
ARG DEBIAN_FRONTEND=noninteractive

ENV CERES_VERSION="2.0.0"
ENV CATKIN_WS=/root/catkin_ws
ENV TZ=Asia/Shanghai LANG=C.UTF-8 LC_ALL=C.UTF-8 PIP_NO_CACHE_DIR=1 PIP_CACHE_DIR=/tmp/

RUN sed -i "s/archive.ubuntu.com/mirrors.ustc.edu.cn/g" /etc/apt/sources.list &&\
    sed -i "s/security.ubuntu.com/mirrors.ustc.edu.cn/g" /etc/apt/sources.list &&\
    rm -f /etc/apt/sources.list.d/* &&\
    apt-get update && apt-get upgrade -y &&\
    apt-get install -y --no-install-recommends \
        # DeterminedAI requirements and common tools
        autoconf automake autotools-dev build-essential ca-certificates \
        make cmake ninja-build pkg-config g++ ccache yasm \
        ccache doxygen graphviz plantuml \
        daemontools krb5-user ibverbs-providers libibverbs1 \
        libkrb5-dev librdmacm1 libssl-dev libtool \
        libnuma1 libnuma-dev libpmi2-0-dev \
        openssh-server openssh-client pkg-config nfs-common \
        ## Tools
        git curl wget unzip nano vim-tiny net-tools sudo htop iotop \
        cloc rsync screen tmux xz-utils software-properties-common \
        ## Deps
        ffmpeg \
        libatlas-base-dev \
        libboost-filesystem-dev \
        libboost-graph-dev \
        libboost-program-options-dev \
        libboost-system-dev \
        libboost-test-dev \
        libcereal-dev \
        libcgal-dev \
        libeigen3-dev \
        libflann-dev \
        libfreeimage-dev \
        libgflags-dev \
        libglew-dev \
        libgoogle-glog-dev \
        libhdf5-dev \
        libmetis-dev \
        libprotobuf-dev \
        libqt5opengl5-dev \
        libsqlite3-dev \
        libsuitesparse-dev \
        protobuf-compiler \
        python3-pip \
        qtbase5-dev \
        # available in ubuntu 22.04 (jammy)
        # python-is-python3 \
        # python3.10-dev \
    && rm /etc/ssh/ssh_host_ecdsa_key \
    && rm /etc/ssh/ssh_host_ed25519_key \
    && rm /etc/ssh/ssh_host_rsa_key \
    && sed -i "s/^.*X11Forwarding.*$/X11Forwarding yes/" /etc/ssh/sshd_config \
    && sed -i "s/^.*X11UseLocalhost.*$/X11UseLocalhost no/" /etc/ssh/sshd_config \
    && grep "^X11UseLocalhost" /etc/ssh/sshd_config || echo "X11UseLocalhost no" >> /etc/ssh/sshd_config \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Need to retry if gpg: keyserver receive failed: Cannot assign requested address
RUN curl http://repo.ros2.org/repos.key | sudo apt-key add - &&\
    echo "deb http://mirrors.bfsu.edu.cn/ros/ubuntu/ ${UBUNTU_DISTRO} main" > /etc/apt/sources.list.d/ros-latest.list &&\
    apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key 4B63CF8FDE49746E98FA01DDAD19BAB3CBF125EA &&\
    apt-get update &&\
    apt-get install -y --no-install-recommends \
            ros-${ROS_DISTRO}-catkin \
            # removed in noetic
            # python-catkin-tools \
            ros-${ROS_DISTRO}-cv-bridge \
            ros-${ROS_DISTRO}-image-transport \
            ros-${ROS_DISTRO}-message-filters \
            ros-${ROS_DISTRO}-tf &&\
    apt-get clean &&\
    rm -rf /var/lib/apt/lists/*

ENV ROSDISTRO_INDEX_URL=https://mirrors.bfsu.edu.cn/rosdistro/index-v4.yaml
RUN mkdir -p /etc/ros/rosdep/sources.list.d/ &&\
    curl -o /etc/ros/rosdep/sources.list.d/20-default.list https://mirrors.bfsu.edu.cn/github-raw/ros/rosdistro/master/rosdep/sources.list.d/20-default.list &&\
    rosdep update

WORKDIR /tmp
RUN   git clone https://ceres-solver.googlesource.com/ceres-solver && \
      cd ceres-solver && \
      git checkout tags/${CERES_VERSION} && \
      mkdir build && cd build && \
      cmake .. && \
      make -j install && \
      rm -rf ../../ceres-solver && \
      mkdir -p $CATKIN_WS/src/VINS-Fusion/

# Copy VINS-Fusion
COPY ./ $CATKIN_WS/src/VINS-Fusion/
# use the following line if you only have this dockerfile
# RUN git clone https://github.com/HKUST-Aerial-Robotics/VINS-Fusion.git

# Build VINS-Fusion
WORKDIR $CATKIN_WS
ENV TERM xterm
ENV PYTHONIOENCODING UTF-8
RUN bash -c 'source /opt/ros/${ROS_DISTRO}/setup.bash &&\
            catkin_make -DCMAKE_BUILD_TYPE=Release -j${MAX_JOBS}' &&\
    sed -i '/exec "$@"/i \
            source "/root/catkin_ws/devel/setup.bash"' /ros_entrypoint.sh

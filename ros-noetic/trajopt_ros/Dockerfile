# syntax=docker/dockerfile:1.3
FROM nvidia/opengl:1.2-glvnd-runtime-ubuntu20.04
LABEL maintainer ivo.vatavuk@fer.hr

# Setup environment
ENV LANG C.UTF-8
ENV LC_AL C.UTF-8
ENV ROS_DISTRO noetic
ENV DEBIAN_FRONTEND noninteractive
ENV NVIDIA_VISIBLE_DEVICES all
ENV NVIDIA_DRIVER_CAPABILITIES graphics,utility,compute
ENV TZ=Europe/Zagreb
ENV ROSCONSOLE_FORMAT '[${severity}] [${time}] [${node}]: ${message}'
#ROSCONSOLE_FORMAT=[${severity}] [${time}] [${node}] [${file}] [${thread}] [${function}]: ${message}


# Mitigate interactive prompt for choosing keyboard type
COPY ./keyboard /etc/default/keyboard

# Setup timezone (fix interactive package installation)
RUN ln -snf /usr/share/zoneinfo/${TZ} /etc/localtime && echo ${TZ} > /etc/timezone

RUN apt-get update

# Install necessary packages for ROS and Gazebo
RUN apt-get update &&  apt-get install -q -y \
    apt-utils \
    build-essential \
    bc \
    cmake \
    curl \
    git \
    imagemagick \
    lsb-release \
    libboost-dev \
    sudo \
    nano \
    net-tools \
    tmux \
    wget \
    libncursesw5-dev

# Add sources and keys for Gazebo installation
RUN sh -c 'echo "deb http://packages.osrfoundation.org/gazebo/ubuntu-stable `lsb_release -cs` main" > /etc/apt/sources.list.d/gazebo-stable.list'
RUN wget http://packages.osrfoundation.org/gazebo.key -O - | apt-key add -

RUN apt-get update

# Install gazebo packages
RUN apt-get install -q -y \
    gazebo11 \
    libgazebo11-dev

RUN sh -c 'echo "deb http://packages.ros.org/ros/ubuntu focal main" > /etc/apt/sources.list.d/ros-latest.list'
RUN apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654

RUN apt-get update

# Install ROS
RUN apt-get install -y \
    ros-${ROS_DISTRO}-ros-core \
    ros-${ROS_DISTRO}-robot-state-publisher \
    ros-${ROS_DISTRO}-joint-state-publisher \
    ros-${ROS_DISTRO}-xacro \
    ros-${ROS_DISTRO}-image-transport-plugins \
    ros-${ROS_DISTRO}-teleop-twist-joy \
    ros-${ROS_DISTRO}-map-server \
    ros-${ROS_DISTRO}-octomap-ros \
    ros-${ROS_DISTRO}-octomap-msgs \
    ros-${ROS_DISTRO}-gazebo-plugins \
    ros-${ROS_DISTRO}-gazebo-ros \
    ros-${ROS_DISTRO}-gazebo-ros-pkgs \
    ros-${ROS_DISTRO}-gazebo-ros-control \
    ros-${ROS_DISTRO}-gazebo-dev \ 
    ros-${ROS_DISTRO}-catkin \
    ros-${ROS_DISTRO}-control-toolbox \
    ros-${ROS_DISTRO}-controller-interface \
    ros-${ROS_DISTRO}-costmap-2d \
    ros-${ROS_DISTRO}-plotjuggler \
    ros-${ROS_DISTRO}-ros-control \ 
    ros-${ROS_DISTRO}-ros-controllers \
    ros-${ROS_DISTRO}-position-controllers \
    ros-${ROS_DISTRO}-velocity-controllers \
    ros-${ROS_DISTRO}-collada-urdf \
    ros-${ROS_DISTRO}-trac-ik-kinematics-plugin \
    ros-${ROS_DISTRO}-tf-conversions \
    ros-${ROS_DISTRO}-rosserial \
    ros-${ROS_DISTRO}-canopen-motor-node 

# Instal RQT debug/devel stuff
RUN apt-get install -y \
    ros-${ROS_DISTRO}-rqt-common-plugins \
    ros-${ROS_DISTRO}-rqt-joint-trajectory-controller 


# Install catkin tools
RUN wget http://packages.ros.org/ros.key -O - | sudo apt-key add -
RUN apt-get install -y --no-install-recommends \
    python3-wstool \ 
    python3-catkin-pkg \
    python3-catkin-tools \
    python3-rosdep \
    clang-format-10 \
    libeigen3-dev \
    libtf-conversions-dev

# Add user and add it to sudo group
RUN adduser --disabled-password --gecos '' developer 
RUN adduser developer sudo 
RUN echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

USER developer 
WORKDIR /home/developer

RUN sudo rosdep init
RUN rosdep update 
 
# Init ROS workspace 
RUN mkdir -p moveit_ws/src 
WORKDIR /home/developer/moveit_ws/src
RUN bash -c "source /opt/ros/noetic/setup.bash; catkin_init_workspace"; 
WORKDIR /home/developer/moveit_ws
RUN bash -c "source /opt/ros/noetic/setup.bash; wstool init src" \
    && bash -c "wstool merge -t src https://raw.githubusercontent.com/ros-planning/moveit/master/moveit.rosinstall" \
    && bash -c "wstool update -t src" \
    && bash -c "rosdep install -y --from-paths src --ignore-src --rosdistro ${ROS_DISTRO}" \
    && bash -c "source /opt/ros/noetic/setup.bash; catkin build" 


# Add Cartesian planning -> Descartes planning capability
# ========================================================= TBD
#WORKDIR /home/developer/moveit_ws/src
#RUN git clone https://github.com/PickNikRobotics/descartes_capability.git
#WORKDIR /home/developer/moveit_ws
#RUN bash -c "source /opt/ros/noetic/setup.bash; wstool merge -t src src/descartes_capability/descartes_capability.rosinstall" \
#    && bash -c "wstool update -t src" \
#    && bash -c "rosdep install -y --from-paths src --ignore-src --rosdistro ${ROS_DISTRO}" \
#    && bash -c "source /opt/ros/noetic/setup.bash; catkin build" 

RUN echo "source /opt/ros/melodic/setup.bash" >> ~/.bashrc
RUN echo "source /home/developer/moveit_ws/devel/setup.bash" >> ~/.bashrc 


# Install OpenRAVE for FastIK plugin 
# ========================================================= TBD
# Package denpendencies 
# RUN mkdir openrave-repos 
# WORKDIR /home/developer/openrave-repos
# RUN git clone https://github.com/crigroup/openrave-installation
# To install openrave follow this instructions: https://github.com/crigroup/openrave-installation 

# Currently is TracIK used for inverse kinematics 
# WORKDIR /home/developer
# RUN mkdir repos

# =====================================================

# Build initial ROS workspace
WORKDIR /home/developer/moveit_ws
RUN bash -c "source /opt/ros/noetic/setup.bash; catkin build"  

# ===================== ssh cloning ===================
ENV DOCKER_BUILDKIT=1
RUN sudo apt-get install -y openssh-client
ENV GIT_SSH_COMMAND="ssh -v" 
USER root
RUN --mount=type=ssh,id=default \
    mkdir -p ~/.ssh/ && \
    ssh-keyscan -H github.com >> ~/.ssh/known_hosts 

WORKDIR /home/developer/moveit_ws/src
#RUN --mount=type=ssh id=default git clone git@github.com:larics/schunk_lwa4p_ros.git


RUN sudo apt install ros-${ROS_DISTRO}-pr2-description
WORKDIR /home/developer/moveit_ws/src
RUN git clone -b 0.4.3 https://github.com/tesseract-robotics/tesseract_ros.git
RUN git clone -b 0.2.15 https://github.com/ros-industrial/ros_industrial_cmake_boilerplate.git
RUN git clone -b 0.9.6 https://github.com/tesseract-robotics/tesseract.git
RUN git clone -b 0.2.4 https://github.com/tesseract-robotics/trajopt.git
RUN git clone -b 0.10.0 https://github.com/tesseract-robotics/tesseract_planning.git
RUN git clone -b 0.3.0 https://github.com/swri-robotics/descartes_light.git
RUN git clone -b 0.4.5  https://github.com/Jmeyer1292/opw_kinematics.git
RUN git clone -b 2.1.3  https://github.com/ethz-adrl/ifopt.git

RUN sudo apt-get install -y libeigen3-dev coinor-libipopt-dev libbullet-dev libbullet-extras-dev ros-noetic-fcl
RUN sudo apt-get install -y software-properties-common
RUN sudo add-apt-repository ppa:ros-industrial/ppa
RUN sudo apt-get update
RUN sudo apt-get install -y taskflow
RUN bash -c "catkin build"
WORKDIR /home/developer/moveit_ws
RUN bash -c "source /home/developer/moveit_ws/devel/setup.bash"
RUN bash -c "source /opt/ros/noetic/setup.bash"
RUN echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc
RUN echo "source /home/developer/moveit_ws/devel/setup.bash" >> ~/.bashrc 

WORKDIR /home/developer/moveit_ws/src
RUN sudo apt install -y python3 python3-pip
RUN sudo python3 -m pip install conan
RUN conan config set general.revisions_enabled=1
RUN conan profile new default --detect > /dev/null
RUN conan profile update settings.compiler.libcxx=libstdc++11 default
RUN git clone https://github.com/Kinovarobotics/ros_kortex.git

WORKDIR /home/developer/moveit_ws
RUN rosdep update
RUN rosdep install --from-paths src --ignore-src -y

RUN bash -c "catkin build kortex_gazebo"

CMD ["bash"]


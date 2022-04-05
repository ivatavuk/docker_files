FROM nvidia/opengl:1.0-glvnd-runtime-ubuntu16.04
LABEL maintainer filip.zoric@fer.hr

RUN apt-get update && apt-get install -q -y \
    apt-utils \
    build-essential \
    bc \
    cmake \
    curl \
    git \
    imagemagick \
    locate \
    lsb-release \
    libboost-dev \
    sudo \
    nano \
    net-tools \
    tmux \
    wget \
    mesa-utils \
    libncursesw5-dev

# Add sources for Gazebo installation
RUN sudo sh -c 'echo "deb http://packages.osrfoundation.org/gazebo/ubuntu-stable `lsb_release -cs` main" > /etc/apt/sources.list.d/gazebo-stable.list'
RUN wget https://packages.osrfoundation.org/gazebo.key -O - | sudo apt-key add -

RUN apt-get update

# Install gazebo packages
RUN apt-get install -q -y \
    gazebo9 \
    libgazebo9-dev

RUN sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
RUN sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654

RUN apt-get update

RUN apt-get install -y \
    python-catkin-tools

# Install ROS
RUN apt-get install -y \
    ros-kinetic-ros-core \
    ros-kinetic-robot-state-publisher \
    ros-kinetic-joint-state-publisher \
    ros-kinetic-xacro \
    ros-kinetic-image-transport-plugins \
    ros-kinetic-teleop-twist-joy \
    ros-kinetic-map-server \
    ros-kinetic-octomap-ros \
    ros-kinetic-octomap-msgs \
    ros-kinetic-gazebo9-plugins \
    ros-kinetic-gazebo9-ros \
    ros-kinetic-gazebo9-ros-pkgs \
    ros-kinetic-gazebo9-ros-control \
    ros-kinetic-gazebo9-dev \
    ros-kinetic-catkin \
    ros-kinetic-moveit \
    ros-kinetic-moveit-msgs \
    ros-kinetic-cob-srvs \
    ros-kinetic-control-toolbox \
    ros-kinetic-controller-interface \
    ros-kinetic-costmap-2d \
    ros-kinetic-ros-control \
    ros-kinetic-joint-state-controller \
    ros-kinetic-position-controllers \
    ros-kinetic-velocity-controllers \
    ros-kinetic-effort-controllers \
    ros-kinetic-controller-manager \
    ros-kinetic-ros-controllers \
    ros-kinetic-moveit-visual-tools

# Add user and add it to sudo group
RUN adduser --disabled-password --gecos '' developer
RUN adduser developer sudo
RUN echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

USER developer
WORKDIR /home/developer

RUN sudo rosdep init
RUN rosdep update

# Add default path for Gazebo plugins
ENV GAZEBO_PLUGIN_PATH /opt/ros/kinetic/lib

# source setup.bash
RUN #!/bin/bash source /opt/ros/kinetic/setup.bash
# instead of rosdep bellow
RUN sudo apt-get install -y \
    ros-kinetic-catkin \
    ros-kinetic-tf2-geometry-msgs \
    ros-kinetic-franka-description
# Init ROS workspace
RUN mkdir -p ~/ws_moveit/src
WORKDIR /home/developer/ws_moveit/src
RUN git clone -b kinetic-devel https://github.com/ros-planning/moveit_tutorials.git
RUN git clone -b kinetic-devel https://github.com/ros-planning/panda_moveit_config.git
#RUN rosdep install -y --from-paths . --ignore-src --rosdistro kinetic
WORKDIR /home/developer/ws_moveit

RUN catkin config --extend /opt/ros/kinetic
RUN catkin build
RUN echo 'source ~/ws_moveit/devel/setup.bash' >> ~/.bashrc

#WORKDIR /home/developer/ws_moveit/src
#RUN git clone --branch kinetic_dev https://github.com/ipa320/cob_control.git
#RUN git clone https://github.com/h3ct0r/schunk_lwa4p_gazebo_sim.git

#WORKDIR /home/developer/ws_moveit
#RUN catkin build

# Add newer nano version to have syntax highlighting in nano
WORKDIR /home/developer
COPY install_nano.sh .
RUN sudo chmod +x install_nano.sh \
    && bash -c "./install_nano.sh"


RUN sudo apt-get install -y \
    unzip

WORKDIR /home/developer
RUN curl https://raw.githubusercontent.com/scopatz/nanorc/master/install.sh | sh

# Add Eigen3 and install (installs to user/local) 
WORKDIR /home/developer
RUN git clone https://gitlab.com/libeigen/eigen.git
WORKDIR /home/developer/eigen
RUN mkdir build \
    && cd build \
    cmake ..

WORKDIR /home/developer
RUN git clone https://github.com/crigroup/openrave-installation
WORKDIR /home/developer/openrave-installation
RUN ./install-dependencies.sh
RUN ./install-osg.sh
RUN ./install-fcl.sh
RUN ./install-openrave.sh

WORKDIR /home/developer
RUN git clone https://github.com/openscenegraph/openscenegraph.git -b OpenSceneGraph-3.4.1
WORKDIR /home/developer/openscenegraph
RUN mkdir build \
    && cd build \
    cmake .. -DDESIRED_QT_VERSION=4 \
    make -j \
    sudo make install

RUN export LD_LIBRARY_PATH="/usr/local/lib64:/usr/local/lib:$(openrave-config --python-dir)/openravepy/openravepy:$LD_LIBRARY_PATH"
RUN export OPENTHREADS_INC_DIR="/usr/local/include"
RUN export OPENTHREADS_LIB_DIR="/usr/local/lib64:/usr/local/lib"
RUN export PATH="$OPENTHREADS_LIB_DIR:$PATH"

# install and than remove libopenscenegraph - hack from https://github.com/joschu/trajopt/issues/20
RUN sudo apt-get install -y libopenscenegraph-dev
RUN sudo cp /usr/include/osg/CameraNode /usr/local/include/osg/
RUN sudo apt-get remove -y libopenscenegraph-dev

WORKDIR /home/developer
RUN git clone https://github.com/joschu/trajopt.git
WORKDIR /home/developer/trajopt
RUN mkdir build 
WORKDIR /home/developer/trajopt/build
RUN cmake .. 
RUN make -j

RUN echo 'source /home/developer/ws_moveit/devel/setup.bash' >> ~/.bashrc
RUN echo 'export PYTHONPATH=$PYTHONPATH:/home/developer/trajopt' >> ~/.bashrc
RUN echo 'export PYTHONPATH=$PYTHONPATH:/home/developer/trajopt/build/lib' >> ~/.bashrc


# ros_kortex install

RUN sudo apt-get install -y python3-pip
#install python3.6
WORKDIR /opt
RUN sudo wget https://www.python.org/ftp/python/3.6.3/Python-3.6.3.tgz
RUN sudo tar -xvf Python-3.6.3.tgz

WORKDIR /opt/Python-3.6.3
RUN sudo ./configure
RUN sudo make
RUN sudo make install


RUN sudo pip3 install -U conan

RUN conan config set general.revisions_enabled=1
RUN conan profile new default --detect > /dev/null
RUN conan profile update settings.compiler.libcxx=libstdc++11 default


WORKDIR /home/developer/ws_moveit/src
RUN git clone https://github.com/ivatavuk/ros_kortex.git -b kinetic-devel
WORKDIR /home/developer/ws_moveit
#RUN rosdep install --from-paths src --ignore-src -y

RUN catkin build

CMD ["bash"]
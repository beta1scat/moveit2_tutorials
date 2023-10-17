# syntax = docker/dockerfile:1.3

ARG ROS_DISTRO=humble

FROM ros:${ROS_DISTRO}

RUN mv /etc/apt/sources.list /etc/apt/sources.list.bak && \
    echo "deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse" >> /etc/apt/sources.list && \
    echo "deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse" >> /etc/apt/sources.list && \
    echo "deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse" >> /etc/apt/sources.list && \
    echo "deb http://security.ubuntu.com/ubuntu/ jammy-security main restricted universe multiverse" >> /etc/apt/sources.list

# Update and install moveit gazebo etc.
RUN apt update && apt install -y curl gnupg2 unar && \
    curl -sSL https://ghproxy.com/https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg && \
    mv /etc/apt/sources.list.d/ros2-latest.list /etc/apt/sources.list.d/ros2-latest.list.bak && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] https://mirrors.tuna.tsinghua.edu.cn/ros2/ubuntu jammy main" | tee /etc/apt/sources.list.d/ros2.list > /dev/null && \
    apt update && apt upgrade -y && \
    apt install -y ros-${ROS_DISTRO}-moveit ros-${ROS_DISTRO}-ros2-control ros-${ROS_DISTRO}-ros2-controllers \
    ros-${ROS_DISTRO}-gripper-controllers gazebo ros-${ROS_DISTRO}-gazebo-ros2-control \
    ros-${ROS_DISTRO}-gazebo-ros-pkgs ros-${ROS_DISTRO}-xacro ros-${ROS_DISTRO}-rmw-cyclonedds-cpp \
    ros-${ROS_DISTRO}-moveit-ros-perception ros-${ROS_DISTRO}-moveit-servo \
    ros-${ROS_DISTRO}-rviz-visual-tools ros-${ROS_DISTRO}-moveit-visual-tools ros-${ROS_DISTRO}-rqt* && \
    echo export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp >> ~/.bashrc && \
    echo source /opt/ros/${ROS_DISTRO}/setup.bash >> ~/.bashrc

# Clone and setup IFRA ros2_RobotSimulation
RUN cd ~ && mkdir -p ws_ifra/src && cd ws_ifra/src && \
    git clone https://ghproxy.com/https://github.com/IFRA-Cranfield/ros2_RobotSimulation.git -b ${ROS_DISTRO} && \
    cp ./ros2_RobotSimulation/include/move_group_interface_improved.h /opt/ros/${ROS_DISTRO}/include/moveit/move_group_interface && \
    cd / && rm -rf ~/ws_ifra

# Clone gazebo models
RUN mkdir -p /root/.gazebo && cd /root/.gazebo && \
    git clone https://ghproxy.com/https://github.com/osrf/gazebo_models.git && mv gazebo_models models
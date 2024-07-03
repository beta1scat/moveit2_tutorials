# syntax = docker/dockerfile:1.3

FROM rocm/pytorch:rocm6.1.1_ubuntu22.04_py3.10_pytorch_release-2.1.2

ENV ROS_DISTRO=humble
ENV DEBIAN_FRONTEND=noninteractive

# Updata and install essential packages
RUN mv /etc/apt/sources.list /etc/apt/sources.list.bak && \
    echo "deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse" >> /etc/apt/sources.list && \
    echo "deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse" >> /etc/apt/sources.list && \
    echo "deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse" >> /etc/apt/sources.list && \
    echo "deb http://security.ubuntu.com/ubuntu/ jammy-security main restricted universe multiverse" >> /etc/apt/sources.list && \
    apt update && apt install -y locales && locale-gen en_US en_US.UTF-8 && \
    update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8 && export LANG=en_US.UTF-8 && \
    apt install -y software-properties-common && add-apt-repository universe && \
    apt install -y curl wget gnupg2 unar gedit python3-pip git && \
    git config --global url."https://ghproxy.org/https://".insteadOf https://

# Update and install moveit gazebo etc.
RUN curl -sSL https://mirror.ghproxy.com/https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] https://mirrors.tuna.tsinghua.edu.cn/ros2/ubuntu jammy main" | tee /etc/apt/sources.list.d/ros2.list > /dev/null && \
    apt update && apt upgrade -y && \
    apt install -y ros-${ROS_DISTRO}-desktop ros-dev-tools ros-${ROS_DISTRO}-moveit ros-${ROS_DISTRO}-ros2-control ros-${ROS_DISTRO}-ros2-controllers \
    ros-${ROS_DISTRO}-gripper-controllers gazebo ros-${ROS_DISTRO}-gazebo-ros2-control \
    ros-${ROS_DISTRO}-gazebo-ros-pkgs ros-${ROS_DISTRO}-xacro ros-${ROS_DISTRO}-rmw-cyclonedds-cpp \
    ros-${ROS_DISTRO}-moveit-ros-perception ros-${ROS_DISTRO}-moveit-servo \
    ros-${ROS_DISTRO}-rviz-visual-tools ros-${ROS_DISTRO}-moveit-visual-tools ros-${ROS_DISTRO}-rqt* && \
    # Fix gazebo grasp plugin
    apt install -y libignition-math6-dev  && \
    # Fix gazebo spawn error
    pip install -i https://pypi.tuna.tsinghua.edu.cn/simple lxml && \
    echo export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp >> ~/.bashrc && \
    echo source /opt/ros/${ROS_DISTRO}/setup.bash >> ~/.bashrc && \
    rm /opt/conda/lib/libstdc++.so.6 && ln -s /usr/lib/x86_64-linux-gnu/libstdc++.so.6 /opt/conda/lib/libstdc++.so.6

# Clone and setup IFRA ros2_RobotSimulation, clone gazebo models
RUN cd ~ && mkdir -p ws_ifra/src && cd ws_ifra/src && \
    git clone https://github.com/IFRA-Cranfield/ros2_RobotSimulation.git -b ${ROS_DISTRO} && \
    cp ./ros2_RobotSimulation/include/move_group_interface_improved.h /opt/ros/${ROS_DISTRO}/include/moveit/move_group_interface && \
    cd / && rm -rf ~/ws_ifra  && \
    mkdir -p /root/.gazebo && cd /root/.gazebo && \
    git clone https://github.com/osrf/gazebo_models.git && mv gazebo_models models

# Install SAM, RAM, DINO etc.
RUN cd ~ && mkdir dinoSAM && cd dinoSAM && pip install -U pip && \
    apt update && apt install -y libgtk2.0-dev pkg-config && \
    pip install -i https://pypi.tuna.tsinghua.edu.cn/simple opencv-python open3d spatialmath-python && \
    pip install -i https://pypi.tuna.tsinghua.edu.cn/simple git+https://github.com/xinyu1205/recognize-anything.git && \
    git clone https://github.com/IDEA-Research/Grounded-Segment-Anything.git && cd Grounded-Segment-Anything && git submodule update --init --recursive && \
    pip install -i https://pypi.tuna.tsinghua.edu.cn/simple -e segment_anything && \
    pip install -i https://pypi.tuna.tsinghua.edu.cn/simple --no-build-isolation -e GroundingDINO && \
    pip install -i https://pypi.tuna.tsinghua.edu.cn/simple --upgrade diffusers[torch] && \
    cd grounded-sam-osx && bash install.sh && \
    pip install -i https://pypi.tuna.tsinghua.edu.cn/simple litellm
    # cd /opt/conda/envs/py_3.10/lib/ && \
    # rm libssl.so.3 && ln -s /usr/lib/x86_64-linux-gnu/libssl.so.3 libssl.so.3 && \
    # rm libcurl.so.4.8.0 && ln -s /usr/lib/x86_64-linux-gnu/libcurl.so.4.7.0 libcurl.so.4.8.0 && \
    # rm libstdc++.so.6.0.29 && ln -s /usr/lib/x86_64-linux-gnu/libstdc++.so.6.0.30 .libstdc++.so.6.0.29
RUN pip uninstall opencv-python && \
    pip install -i https://pypi.tuna.tsinghua.edu.cn/simple opencv-python lark && \
    conda uninstall --force openssl libcurl ncurses && \
    apt remove -y python3-colcon-* && \
    pip install -U colcon-common-extensions && \
    ln -s /opt/conda/envs/py_3.10/bin/colcon /usr/bin/ && \
    cd /opt/conda/envs/py_3.10/lib/ && \
    rm libssl.so.3 && ln -s /usr/lib/x86_64-linux-gnu/libssl.so.3 libssl.so.3 && cd ~ && \

# conda uninstall --force openssl libcurl ncurses && \
# docker build . --progress=plain -t ros2-rorch-dino

# 有可能安装了lark以后会出现undefine libgdal的错误？？？
# 问题解决：
# conda uninstall libcurl
# conda uninstall openssl
# conda uninstall ncurses
# apt remove python3-colcon-*
# pip install -U colcon-common-extensions
# pip install empy==3.3.2
# pip install catkin_pkg
# pip install lark
# cd /opt/conda/envs/py_3.10/lib/ && \
# rm libssl.so.3 && ln -s /usr/lib/x86_64-linux-gnu/libssl.so.3 libssl.so.3
# rm libcurl.so.4.8.0 && ln -s /usr/lib/x86_64-linux-gnu/libcurl.so.4.7.0 libcurl.so.4.8.0
# rm libstdc++.so.6.0.29 && ln -s /usr/lib/x86_64-linux-gnu/libstdc++.so.6.0.30 .libstdc++.so.6.0.29

# ln -s ~/ros_ws/src/gazebo_models/* ~/.gazebo/models/ && rm ~/.gazebo/models/YCB_gazebo && \
# ln -s ~/ros_ws/src/gazebo_models/YCB_gazebo* ~/.gazebo/models/
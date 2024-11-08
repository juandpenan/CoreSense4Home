# Dockerfile for ROSCon 2024 Deliberation Workshop

FROM ros:humble AS robocup2024
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Install apt dependencies.
ARG DEBIAN_FRONTEND=noninteractive
# RUN apt-get update && \
#     apt-get upgrade -y && \
#     apt-get install -y \
#     gcc \
#     git \
#     wget \
#     python3 nlohmann-json3-dev python3-pip portaudio19-dev \
#     ros-humble-moveit
RUN apt-get update \
    && apt-get -y --quiet --no-install-recommends install \
    gcc \
    git \
    wget \
    portaudio19-dev \
    python3 \
    python3-pip \
    ros-humble-moveit \
    nlohmann-json3-dev
   

# Create a ROS 2 workspace.
RUN mkdir -p /robocup/src/
WORKDIR /robocup

ADD https://raw.githubusercontent.com/mgonzs13/audio_common/refs/heads/main/requirements.txt /robocup/requirements1.txt
ADD https://raw.githubusercontent.com/mgonzs13/whisper_ros/refs/heads/main/requirements.txt /robocup/requirements2.txt
ADD https://raw.githubusercontent.com/mgonzs13/llama_ros/refs/heads/main/requirements.txt /robocup/requirements3.txt
ADD https://raw.githubusercontent.com/jmguerreroh/yolov8_ros/refs/heads/main/requirements.txt /robocup/requirements4.txt
ADD https://raw.githubusercontent.com/mgonzs13/tts_ros/refs/heads/main/requirements.txt /robocup/requirements5.txt

# Install external dependencies.
RUN pip install -r requirements1.txt
RUN pip install -r requirements2.txt
RUN pip install -r requirements3.txt
RUN pip install -r requirements4.txt
RUN pip install -r requirements5.txt


WORKDIR /robocup/src
RUN git clone https://github.com/juandpenan/CoreSense4Home.git -b humble-devel
RUN vcs import --recursive < ./CoreSense4Home/robocup_bringup/thirdparty.repos

WORKDIR /robocup
RUN rosdep install --from-paths src --ignore-src -r -y

# Remove MatPlotLib display warnings.
RUN mkdir /tmp/runtime-root
ENV XDG_RUNTIME_DIR "/tmp/runtime-root"
RUN chmod -R 0700 /tmp/runtime-root
ENV NO_AT_BRIDGE 1

# Set up the entrypoint.
# Install CUDA nvcc
ARG USE_CUDA
ARG CUDA_VERSION=12-6

RUN if [ "$USE_CUDA" = "1" ]; then \
    wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/cuda-keyring_1.1-1_all.deb && \
    dpkg -i cuda-keyring_1.1-1_all.deb && \
    rm cuda-keyring_1.1-1_all.deb; \
    apt-get update && apt-get install -y cuda-toolkit-$CUDA_VERSION; \
    echo "export PATH=/usr/local/cuda/bin${PATH:+:${PATH}}" >> ~/.bashrc; \
    echo "export LD_LIBRARY_PATH=/usr/local/cuda/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}" >> ~/.bashrc; \
    fi

# Install gdp grasp generator library
WORKDIR /robocup/src/ThirdParty/gpd
RUN mkdir build
WORKDIR /robocup/src/ThirdParty/gpd/build
RUN cmake ..
RUN make -j
RUN make install

# Colcon the ws
WORKDIR /robocup
FROM robocup2024 AS builder
ARG CMAKE_BUILD_TYPE=Release

ENV PATH=/usr/local/cuda/bin${PATH:+:${PATH}}
ENV LD_LIBRARY_PATH=/usr/local/cuda/lib64

RUN source /opt/ros/${ROS_DISTRO}/setup.bash && \
    if [ "$USE_CUDA" = "1" ]; then \
    source ~/.bashrc && \
    colcon build --packages-skip tts_bringup --cmake-args -DGGML_CUDA=ON; \
    else \
    colcon build --packages-skip tts_bringup; \
    fi

# Source the ROS 2 setup file
RUN echo "source /root/robocup/install/setup.bash" >> ~/.bashrc

# Run a default command, e.g., starting a bash shell
CMD ["bash"]
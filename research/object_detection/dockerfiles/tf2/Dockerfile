# using 2.6 as the packages installed after the docker deployment installs 
#2.6, which creates a conflict with other pre-installed packages
FROM tensorflow/tensorflow:2.6.0-gpu

ARG DEBIAN_FRONTEND=noninteractive
# Remove the existing CUDA repo lists
RUN rm /etc/apt/sources.list.d/cuda*.list

# Remove the existing CMake if installed via apt
RUN apt-get purge --auto-remove -y cmake

# Install apt dependencies
RUN apt-get update && apt-get install -y \
    git \
    gpg-agent \
    python3-cairocffi \
    protobuf-compiler \
    python3-pil \
    python3-lxml \
    python3-tk \
    wget

# Install the latest version of CMake
RUN apt-get update && \
    apt-get install -y wget && \
    wget https://github.com/Kitware/CMake/releases/download/v3.25.0/cmake-3.25.0-linux-x86_64.sh && \
    chmod +x cmake-3.25.0-linux-x86_64.sh && \
    ./cmake-3.25.0-linux-x86_64.sh --prefix=/usr/local --skip-license && \
    rm cmake-3.25.0-linux-x86_64.sh

# Install gcloud and gsutil commands
RUN export CLOUD_SDK_REPO="cloud-sdk-$(lsb_release -c -s)" && \
    echo "deb http://packages.cloud.google.com/apt $CLOUD_SDK_REPO main" | tee -a /etc/apt/sources.list.d/google-cloud-sdk.list && \
    curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add - && \
    apt-get update -y && apt-get install google-cloud-sdk -y

# Add new user to avoid running as root
RUN useradd -ms /bin/bash tensorflow
USER tensorflow
WORKDIR /home/tensorflow

# Copy this version of the model garden into the image
COPY --chown=tensorflow . /home/tensorflow/models

# Compile protobuf configs
RUN (cd /home/tensorflow/models/research/ && protoc object_detection/protos/*.proto --python_out=.)

WORKDIR /home/tensorflow/models/research/

# Setup the object detection API
RUN cp object_detection/packages/tf2/setup.py ./
ENV PATH="/home/tensorflow/.local/bin:${PATH}"

# Upgrade pip and install the necessary dependencies, including TensorFlow 2.6.0
RUN python -m pip install -U pip
RUN python -m pip install .

# Set TensorFlow logging level to minimize log output
ENV TF_CPP_MIN_LOG_LEVEL 3


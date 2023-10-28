FROM ubuntu:18.04 as base

FROM base as base-amd64

ENV NVARCH x86_64
ENV NVIDIA_REQUIRE_CUDA "cuda>=10.2 brand=tesla,driver>=418,driver<419"
ENV NV_CUDA_CUDART_VERSION 10.2.89-1

FROM base as base-ppc64le

ENV NVARCH ppc64el
ENV NVIDIA_REQUIRE_CUDA "cuda>=10.2"
ENV NV_CUDA_CUDART_VERSION 10.2.89-1

FROM base-${TARGETARCH}

ARG TARGETARCH

LABEL maintainer "NVIDIA CORPORATION <cudatools@nvidia.com>"

RUN apt-get update && apt-get install -y --no-install-recommends \
    gnupg2 curl ca-certificates && \
    curl -fsSL https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/${NVARCH}/3bf863cc.pub | apt-key add - && \
    echo "deb https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/${NVARCH} /" > /etc/apt/sources.list.d/cuda.list && \
    apt-get purge --autoremove -y curl \
    && rm -rf /var/lib/apt/lists/*

ENV CUDA_VERSION 10.2.89

# For libraries in the cuda-compat-* package: https://docs.nvidia.com/cuda/eula/index.html#attachment-a
RUN apt-get update && apt-get install -y --no-install-recommends \
    cuda-cudart-10-2=${NV_CUDA_CUDART_VERSION} \
    cuda-compat-10-2 \
    && ln -s cuda-10.2 /usr/local/cuda && \
    rm -rf /var/lib/apt/lists/*

# Required for nvidia-docker v1
RUN echo "/usr/local/nvidia/lib" >> /etc/ld.so.conf.d/nvidia.conf && \
    echo "/usr/local/nvidia/lib64" >> /etc/ld.so.conf.d/nvidia.conf

ENV PATH /usr/local/nvidia/bin:/usr/local/cuda/bin:${PATH}
ENV LD_LIBRARY_PATH /usr/local/nvidia/lib:/usr/local/nvidia/lib64

# nvidia-container-runtime
ENV NVIDIA_VISIBLE_DEVICES all
ENV NVIDIA_DRIVER_CAPABILITIES compute,utility


### End of Nvidia build

# Timezone Configuration
ENV TZ=Europe/Moscow
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
ENV DEBIAN_FRONTEND=noninteractive

RUN apt update && apt install -y    apt-utils \
                                    lsb-release \
                                    mesa-utils \
                                    gnupg2 \
                                    net-tools \
                                    iputils-ping \
                                    build-essential \
                                    wget \
                                    unzip \
                                    curl \
                                    git \
                                    mc \
                                    vim \
                                    nano \
                                    ffmpeg \
                                    x11-apps

## GUI tunnel from docker
RUN dpkg --add-architecture i386 && \
    apt-get update && apt-get install -y --no-install-recommends \
        libxau6 libxau6:i386 \
        libxdmcp6 libxdmcp6:i386 \
        libxcb1 libxcb1:i386 \
        libxext6 libxext6:i386 \
        libx11-6 libx11-6:i386 && \
    rm -rf /var/lib/apt/lists/*

RUN apt-get update && apt-get install -y --no-install-recommends \
        libglvnd0 libglvnd0:i386 \
        libgl1 libgl1:i386 \
        libglx0 libglx0:i386 \
        libegl1 libegl1:i386 \
        libgles2 libgles2:i386 && \
    rm -rf /var/lib/apt/lists/*
# Required for non-glvnd setups.
ENV LD_LIBRARY_PATH /usr/lib/x86_64-linux-gnu:/usr/lib/i386-linux-gnu${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}:/usr/local/nvidia/lib:/usr/local/nvidia/lib64

RUN apt update &&  apt install -y make \
                                libssl-dev \
                                zlib1g-dev \
                                libbz2-dev \
                                libreadline-dev \
                                libsqlite3-dev \
                                llvm \
                                libncurses5-dev \
                                libncursesw5-dev \
                                xz-utils \
                                tk-dev \
                                libffi-dev \
                                liblzma-dev \
                                libgdbm-dev \
                                libnss3-dev \
                                libedit-dev \
                                libc6-dev \
                                cmake \
                                unzip \
                                autoconf \
                                autogen \
                                automake \
                                libtool \
                                mlocate


RUN apt update && apt install -y python3-pip python-pip python3-dev python-dev && pip3 install --upgrade setuptools pip && pip install --upgrade setuptools pip
RUN apt update && apt install -y python3-tk python3-numpy python-tk python-numpy



RUN apt update && apt install -y yasm \
                                libtbb2 \
                                libtbb-dev \
                                libjpeg-dev \
                                libpng-dev \
                                libtiff-dev \
                                libtiff5-dev \
                                libavformat-dev \
                                libpq-dev \
                                libxine2-dev \
                                libglew-dev \
                                libavcodec-dev \
                                libavutil-dev \
                                libpostproc-dev \
                                libswscale-dev \
                                libeigen3-dev \
                                libgtk2.0-dev \
                                pkg-config \
                                libqt5gui5 \
                                libv4l-dev \
                                libarmadillo-dev
# install OpenCV
ARG OPENCV_VERSION=3.4.2
# ENV CUDA_TOOLKIT_ROOT_DIR /usr/local/cuda
WORKDIR /opt
RUN wget https://github.com/opencv/opencv/archive/$OPENCV_VERSION.zip &&\
    unzip $OPENCV_VERSION.zip &&\
    rm $OPENCV_VERSION.zip &&\
    wget https://github.com/opencv/opencv_contrib/archive/$OPENCV_VERSION.zip &&\
    unzip ${OPENCV_VERSION}.zip &&\
    rm ${OPENCV_VERSION}.zip &&\
    # Create build folder and switch to it
    mkdir /opt/opencv-${OPENCV_VERSION}/build && cd /opt/opencv-${OPENCV_VERSION}/build &&\
    # Cmake configure
    cmake \
        -DOPENCV_EXTRA_MODULES_PATH=/opt/opencv_contrib-${OPENCV_VERSION}/modules \
        -DWITH_CUDA=ON \
        -DCMAKE_BUILD_TYPE=RELEASE \
        # Install path will be /usr/local/lib (lib is implicit)
        -DCMAKE_INSTALL_PREFIX=/usr/local \
        .. &&\
    # Make
    make -j"$(nproc)" && \
    # Install to /usr/local/lib
    make install && \
    ldconfig &&\
    # Remove OpenCV sources and build folder
    rm -rf /opt/opencv-${OPENCV_VERSION} && rm -rf /opt/opencv_contrib-${OPENCV_VERSION}
# RUN apt update && apt install -y tensorrt

WORKDIR /root
RUN wget https://apt.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB && \
apt-key add GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB && \
rm GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB && \
echo "deb https://apt.repos.intel.com/openvino/2022 bionic main" | tee /etc/apt/sources.list.d/intel-openvino-2022.list
RUN apt update && apt install -y openvino-2022.1.0 
RUN echo "/workspace/PAL-SDK/lib/x86_64/" | tee /etc/ld.so.conf.d/libpal.conf
RUN apt update && apt install -y libusb-1.0-0-dev libudev-dev udev

RUN wget -q -O KEY.gpg https://dreamvu.github.io/ppa/KEY.gpg && apt-key add KEY.gpg && rm KEY.gpg
RUN wget -qO /etc/apt/sources.list.d/dreamvu.list https://dreamvu.github.io/ppa/dreamvu.list
RUN apt update && apt install -y lshw bc sudo rsync
RUN apt update && apt install -y ppa-pal pal

WORKDIR /workspace
ENTRYPOINT ["/bin/bash", "-c"]
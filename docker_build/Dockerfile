FROM --platform=linux/amd64 rapidsai/rapidsai-core:23.02-cuda11.8-base-ubuntu22.04-py3.8 as rapid-runtime

# set LD_LIBRARY_PATH
ENV LD_LIBRARY_PATH=/usr/local/cuda-11.8/compat:$LD_LIBRARY_PATH

ENV CUDA_HOME /usr/local/cuda
ENV LD_LIBRARY_PATH ${LD_LIBRARY_PATH}:${CUDA_HOME}/lib64
ENV LD_LIBRARY_PATH ${LD_LIBRARY_PATH}:/usr/local/lib

ENV OPENCL_LIBRARIES /usr/local/cuda/lib64
ENV OPENCL_INCLUDE_DIR /usr/local/cuda/include

# dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    build-essential \
    curl \
    bzip2 \
    ca-certificates \
    libglib2.0-0 \
    libxext6 \
    libsm6 \
    libxrender1 \
    git \
    vim \
    mercurial \
    subversion \
    cmake \
    libboost-dev \
    libboost-system-dev \
    libboost-filesystem-dev \
    gcc \
    g++

# torch
FROM rapid-runtime as pytorch
ENV TORCH_VERSION=1.10.1+cu111
ENV TORCHVISION_VERSION=0.11.2+cu111
ENV TORCHAUDIO_VERSION=0.10.1
#RUN python3 -m pip install --upgrade pip --no-cache-dir && \ 
#    python3 -m pip --default-timeout=1000 install torch==${TORCH_VERSION} torchvision==${TORCHVISION_VERSION} torchaudio==${TORCHAUDIO_VERSION} -f https://download.pytorch.org/whl/torch_stable.html


RUN conda install pytorch==1.10.1 torchvision==0.11.2 torchaudio==0.10.1 cudatoolkit=11.3 -c pytorch -c conda-forge

# LightGBM
RUN apt install -y ocl-icd-libopencl1 ocl-icd-opencl-dev

RUN cd /usr/local/src && mkdir lightgbm && cd lightgbm && \
    git clone --recursive --branch stable --depth 1 https://github.com/microsoft/LightGBM && \
    cd LightGBM && mkdir build && cd build && \
    cmake -DUSE_GPU=1 .. && \
    make -j1

ENV PATH /usr/local/src/lightgbm/LightGBM:${PATH}
RUN /bin/bash -c "cd /usr/local/src/lightgbm/LightGBM/python-package && python setup.py install --precompile"

# Add OpenCL ICD files for LightGBM
RUN mkdir -p /etc/OpenCL/vendors && \
    echo "libnvidia-opencl.so.1" > /etc/OpenCL/vendors/nvidia.icd

## 时区
FROM pytorch as runtime
# ENV TZ Asia/Shanghai
ENV TZ=Asia/Shanghai \
    DEBIAN_FRONTEND=noninteractive

RUN apt install -y tzdata \
    && ln -fs /usr/share/zoneinfo/${TZ} /etc/localtime \
    && echo ${TZ} > /etc/timezone \
    && dpkg-reconfigure --frontend noninteractive tzdata \
    && rm -rf /var/lib/apt/lists/*

## 字符集
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US.UTF-8
ENV LC_ALL en_US.UTF-8

# System CleanUp
RUN apt-get autoremove -y && apt-get clean && \
    rm -rf /var/lib/apt/lists/* && \
    conda clean -a -y

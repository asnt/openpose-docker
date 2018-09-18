FROM nvidia/cuda:9.2-cudnn7-devel-ubuntu18.04
ARG CUDA_ARCH_BIN="30 35 37 50 52 60 61 70"
ARG CUDA_ARCH_PTX=70
ARG DOWNLOAD_MODELS=OFF
ARG NPROC=4

LABEL maintainer "Alexandre Saint <snt.alex@gmail.com>"

RUN apt-get update -y && apt-get upgrade -y

# Prevent interactive configuration of tzdata.
# https://serverfault.com/questions/347937/how-do-i-ask-apt-get-to-skip-all-post-install-configuration-steps#347953
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y tzdata

## Configure tzdata programmatically.
## https://stackoverflow.com/a/8671492/2528098
## https://bugs.launchpad.net/ubuntu/+source/tzdata/+bug/1554806
#RUN ln -s /usr/share/zoneinfo/Europe/Luxembourg /etc/localtime \
#    && dpkg-reconfigure -f noninteractive tzdata

RUN apt-get install -y \
    apt-utils \
    cmake \
    git \
    lsb-core \
    libatlas-base-dev \
    libgflags-dev \
    libgoogle-glog-dev \
    libhdf5-serial-dev \
    liblmdb-dev \
    libleveldb-dev \
    libopencv-dev \
    libprotobuf-dev \
    libsnappy-dev \
    protobuf-compiler \
    python-opencv \
    python-pip
RUN apt-get install -y --no-install-recommends libboost-all-dev
RUN pip install --upgrade numpy protobuf

# Install Caffe
WORKDIR /
RUN git clone https://github.com/BVLC/caffe.git
WORKDIR /caffe
RUN mkdir build && \
    cd build && \
    cmake .. \
      -DUSE_CUDNN=ON \
      -DCUDA_ARCH_NAME=Manual \
      -DCUDA_ARCH_BIN=${CUDA_ARCH_BIN}
RUN cd build && cmake --build . -- -j ${NPROC}
RUN cd build && make install

# Install OpenPose
WORKDIR /
RUN git clone https://github.com/CMU-Perceptual-Computing-Lab/openpose.git
WORKDIR /openpose
RUN mkdir build && \
    cd build && \
    cmake .. \
        -DCUDA_ARCH=Manual \
        -DCUDA_ARCH_BIN=${CUDA_ARCH_BIN} \
        -DCUDA_ARCH_PTX=70 \
        -DBUILD_CAFFE=OFF \
        -DCaffe_INCLUDE_DIRS=/caffe/build/install/include \
        -DCaffe_LIBS=/caffe/build/install/lib/libcaffe.so \
        -DDOWNLOAD_BODY_25_MODEL=OFF \
        -DDOWNLOAD_BODY_COCO_MODEL=${DOWNLOAD_MODELS} \
        -DDOWNLOAD_BODY_MPI_MODEL=OFF \
        -DDOWNLOAD_FACE_MODEL=OFF \
        -DDOWNLOAD_HAND_MODEL=${DOWNLOAD_MODELS}
RUN cd build && make -j ${NPROC}
ENV PATH="/openpose/build/examples/openpose:${PATH}"
CMD /openpose/build/examples/openpose/openpose.bin

FROM nvidia/cuda:8.0-cudnn5-devel-ubuntu16.04
LABEL maintainer "NVIDIA CORPORATION <cudatools@nvidia.com>"

RUN echo "deb https://developer.download.nvidia.com/compute/machine-learning/repos/ubuntu1604/x86_64 /" > /etc/apt/sources.list.d/nvidia-ml.list

ENV CUDNN_VERSION 6.0.21
ENV CLONE_TAG=master
LABEL com.nvidia.cudnn.version="${CUDNN_VERSION}"

RUN apt-get update && apt-get install -y --no-install-recommends apt-utils \
        libcudnn6=$CUDNN_VERSION-1+cuda8.0 \
        build-essential \
        cmake \
        git \
        wget \
        sudo \
        software-properties-common \
        libssl-dev \
        libzmq3-dev \        
        python-zmq \
        libatlas-base-dev \
        libboost-all-dev \
        libgflags-dev \
        libgoogle-glog-dev \
        libhdf5-serial-dev \
        libleveldb-dev \
        liblmdb-dev \
        libopencv-dev \
        libprotobuf-dev \
        libsnappy-dev \
        libgeos-dev \
        protobuf-compiler \
        python-dev \
        python-numpy \
        python-pip \
        python-scipy \
        python-setuptools \
        libhdf5-dev \
        libhdf5-serial-dev \
        libopenblas-dev \
        python-opencv && \
    rm -rf /var/lib/apt/lists/*

ENV CAFFE_ROOT=/opt/caffe
ENV TORCH_ROOT=/root/torch
WORKDIR $CAFFE_ROOT

# FIXME: clone a specific git tag and use ARG instead of ENV once DockerHub supports this.
ENV CLONE_TAG=master

# Set this with `docker build --build-arg CLONE_REPO=$(git remote get-url --all origin) .`
# Note that will only work for https urls since ssh is not installed in the image

ARG CLONE_REPO

RUN git clone -b ${CLONE_TAG} --depth 1 $CLONE_REPO .

# Required for recent GPUs
RUN pip install --upgrade pip
RUN for req in $(cat python/requirements.txt); do pip install $req; done && \
    mkdir build && cd build && \
    cmake .. -DCUDA_ARCH_NAME=Manual && \
    make -j"$(nproc)"

# HACK: OpenCV can be confused by (the lack of) this driver in some systems
RUN ln /dev/null /dev/raw1394

ENV PYCAFFE_ROOT $CAFFE_ROOT/python
ENV PYTHONPATH $PYCAFFE_ROOT:$PYTHONPATH
ENV PATH $CAFFE_ROOT/build/tools:$PYCAFFE_ROOT:$PATH
ENV CPATH="/usr/include/hdf5/serial/"

RUN echo "$CAFFE_ROOT/build/lib" >> /etc/ld.so.conf.d/caffe.conf && ldconfig

RUN ln -s /usr/local/lib/python2.7/dist-packages/numpy/core/include/numpy /usr/local/include/numpy
RUN pip install --upgrade protobuf
RUN pip install -U numpy

COPY Makefile.config /opt/caffe/Makefile.config
WORKDIR /opt/caffe
RUN make -j4
RUN make py

COPY model_icdar15.caffemodel /opt/caffe/models/model_icdar15.caffemodel
CMD python examples/text/demo_det.py
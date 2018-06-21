FROM scannerresearch/scanner:gpu-9.1-cudnn7
MAINTAINER Sebastien Testeau "s.testeau@nfb.ca"

ENV CUDNN_VERSION 7.1.4.18

RUN apt-get update && apt-get install -y --no-install-recommends apt-utils \
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

ENV CLONE_REPO=https://github.com/moskiteau/TextBoxes_plusplus.git
RUN echo ${CLONE_REPO}
RUN git clone -b ${CLONE_TAG} --depth 1 $CLONE_REPO .

# Required for recent GPUs
ADD requirements.txt .
RUN pip install --upgrade pip
RUN pip install -r requirements.txt
ADD . .

COPY Makefile.config .
RUN make -j"$(nproc)"
RUN make py

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


COPY model_icdar15.caffemodel /opt/caffe/models/model_icdar15.caffemodel
CMD python examples/text/demo_det.py

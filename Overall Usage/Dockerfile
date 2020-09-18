# Last modified 09/17/2020
# Updated torch and torchvision to 1.6 & 0.7 respectively

FROM nvidia/cuda:10.2-cudnn7-devel-ubuntu18.04
LABEL maintainer="Jayoung Lee(starpic414@gmail.com)"

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Needed to install tex
ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update && apt-get install -y \
	ca-certificates git wget \
	cmake ninja-build protobuf-compiler libprotobuf-dev pandoc texlive-xetex
RUN apt-get install -y --no-install-recommends \
        build-essential \
        curl \
        libfreetype6-dev \
        libhdf5-serial-dev \
        libzmq3-dev \
        libtiff5-dev \
        libavformat-dev \
        libavcodec-dev \
        libpng-dev \
        libjpeg-dev \
        libswscale-dev \
        pkg-config \
        software-properties-common

RUN rm -rf /var/lib/apt/lists/*

# Link the libcuda stub to the location where tensorflow is searching for it and reconfigure
# dynamic linker run-time bindings
RUN ln -s /usr/local/cuda/lib64/stubs/libcuda.so /usr/local/cuda/lib64/stubs/libcuda.so.1 \
    && echo "/usr/local/cuda/lib64/stubs" > /etc/ld.so.conf.d/z-cuda-stubs.conf \
    && ldconfig

# See http://bugs.python.org/issue19846
ENV LANG C.UTF-8

RUN apt-get update && apt-get install -y \
    python3 \
    python3-pip

RUN python3 -m pip --no-cache-dir install --upgrade \
    pip \
    setuptools

# Some TF tools expect a "python" binary
RUN ln -s $(which python3) /usr/local/bin/python

# install dependencies
# See https://pytorch.org/ for other options if you use a different version of CUDA
RUN python -m pip --no-cache-dir install torch==1.6 torchvision==0.7 -f https://download.pytorch.org/whl/cu102/torch_stable.html
RUN python -m pip --no-cache-dir install tensorflow \
	pandas \
	scipy \
	scikit-learn \
	scikit-image \
	pillow \
	pydot \
	Cython \
	contextlib2 \
	tqdm \
	matplotlib \
	seaborn \
    missingno

RUN python -m pip --no-cache-dir install jupyter \
    jupyter_http_over_ws \
    jupyterlab

RUN jupyter serverextension enable --py jupyter_http_over_ws

RUN pip install 'git+https://github.com/facebookresearch/fvcore'
# install detectron2
RUN git clone https://github.com/facebookresearch/detectron2 /dl/detectron2_repo

# set FORCE_CUDA because during `docker build` cuda is not accessible
ENV FORCE_CUDA="1"

# This will by default build detectron2 for all common cuda architectures and take a lot more time,
# because inside `docker build`, there is no way to tell which architecture will be used.
ARG TORCH_CUDA_ARCH_LIST="Kepler;Kepler+Tesla;Maxwell;Maxwell+Tegra;Pascal;Volta;Turing"
ENV TORCH_CUDA_ARCH_LIST="${TORCH_CUDA_ARCH_LIST}"

RUN pip install -e /dl/detectron2_repo

# Set a fixed model cache directory.
ENV FVCORE_CACHE="/tmp"

COPY bashrc/.bashrc /etc/bash.bashrc
RUN chmod a+rwx /etc/bash.bashrc

RUN mkdir /dl/work
WORKDIR /dl

EXPOSE 8888
EXPOSE 6006

RUN python -m ipykernel.kernelspec

CMD ["bash", "-c", "source /etc/bash.bashrc && jupyter lab --notebook-dir=/dl --ip 0.0.0.0 --no-browser --allow-root"]

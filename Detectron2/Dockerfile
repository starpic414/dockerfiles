FROM nvidia/cuda:10.1-cudnn7-devel-ubuntu18.04
LABEL maintainer="Jayoung Lee(starpic414@gmail.com)"

# Needed for string substitution
SHELL ["/bin/bash", "-c"]

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

ARG USE_PYTHON_3_NOT_2=True
ARG _PY_SUFFIX=${USE_PYTHON_3_NOT_2:+3}
ARG PYTHON=python${_PY_SUFFIX}
ARG PIP=pip${_PY_SUFFIX}

# See http://bugs.python.org/issue19846
ENV LANG C.UTF-8

RUN apt-get update && apt-get install -y \
    ${PYTHON} \
    ${PYTHON}-pip

RUN ${PIP} install --upgrade \
    pip \
    setuptools

RUN ln -s $(which ${PYTHON}) /usr/local/bin/python

# install dependencies
# See https://pytorch.org/ for other options if you use a different version of CUDA
RUN pip install tensorboard
RUN pip install torch==1.5 torchvision==0.6 -f https://download.pytorch.org/whl/cu101/torch_stable.html
RUN pip install keras \
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
	opencv-contrib-python \
    missingno

RUN pip install jupyter
RUN pip install jupyter_http_over_ws
RUN pip install jupyterlab

RUN jupyter serverextension enable --py jupyter_http_over_ws

RUN pip install --user 'git+https://github.com/facebookresearch/fvcore'
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
# WORKDIR /home/appuser/detectron2_repo

COPY bashrc/.bashrc /etc/bash.bashrc
RUN chmod a+rwx /etc/bash.bashrc

RUN mkdir /dl/work
WORKDIR /dl

EXPOSE 8888
EXPOSE 6006

RUN ${PYTHON} -m ipykernel.kernelspec

CMD ["bash", "-c", "source /etc/bash.bashrc && jupyter lab --notebook-dir=/dl --ip 0.0.0.0 --no-browser --allow-root"]
# run detectron2 under user "appuser":
# wget http://images.cocodataset.org/val2017/000000439715.jpg -O input.jpg
# python3 demo/demo.py  \
	#--config-file configs/COCO-InstanceSegmentation/mask_rcnn_R_50_FPN_3x.yaml \
	#--input input.jpg --output outputs/ \
	#--opts MODEL.WEIGHTS detectron2://COCO-InstanceSegmentation/mask_rcnn_R_50_FPN_3x/137849600/model_final_f10217.pkl

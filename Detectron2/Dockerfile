# Last modified 09/17/2020
# Updated torch and torchvision to 1.6 & 0.7 respectively
# Updated and added non-sudo user.
# Got buggy..
FROM nvidia/cuda:10.1-cudnn7-devel-ubuntu18.04
LABEL maintainer="Jayoung Lee(starpic414@gmail.com)"

ARG NB_USER="jay"
ARG NB_UID="1000"
ARG NB_GID="1000"

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

RUN apt-get install -y python3-opencv python3-dev
RUN apt-get install -y sudo
RUN rm -rf /var/lib/apt/lists/*

RUN ln -s $(which python3) /usr/local/bin/python

# create a non-root user
ENV NB_USER=$NB_USER \
    NB_UID=$NB_UID \
    NB_GID=$NB_GID \
    HOME=/home/$NB_USER

# Copy a script that we will use to correct permissions after running certain commands
COPY fix-permissions /usr/local/bin/fix-permissions
RUN chmod a+rx /usr/local/bin/fix-permissions

RUN echo "auth requisite pam_deny.so" >> /etc/pam.d/su && \
    sed -i.bak -e 's/^%admin/#%admin/' /etc/sudoers && \
    sed -i.bak -e 's/^%sudo/#%sudo/' /etc/sudoers && \
    useradd -m -s /bin/bash -N -u $NB_UID $NB_USER && \
    chmod g+w /etc/passwd && \
    fix-permissions $HOME 

USER $NB_UID
WORKDIR $HOME

RUN mkdir /home/$NB_USER/work && \
    fix-permissions /home/$NB_USER

ENV PATH="${HOME}/.local/bin:${PATH}"
RUN wget https://bootstrap.pypa.io/get-pip.py && \
	python3 get-pip.py --user && \
	rm get-pip.py && \
    fix-permissions /home/$NB_USER

# install dependencies
# See https://pytorch.org/ for other options if you use a different version of CUDA
RUN pip install --user torch==1.6 torchvision==0.7 -f https://download.pytorch.org/whl/cu101/torch_stable.html
RUN pip install --user tensorflow \
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
    missingno && \
    fix-permissions /home/$NB_USER

RUN pip install --user jupyter \
    jupyter_http_over_ws \
    jupyterlab && \
    fix-permissions /home/$NB_USER

RUN jupyter serverextension enable --py jupyter_http_over_ws --user && \
    fix-permissions /home/$NB_USER

RUN pip install --user 'git+https://github.com/facebookresearch/fvcore' && \
    fix-permissions /home/$NB_USER
# install detectron2
RUN git clone https://github.com/facebookresearch/detectron2 /home/jay/work/detectron2_repo && \
    fix-permissions /home/$NB_USER
# set FORCE_CUDA because during `docker build` cuda is not accessible
ENV FORCE_CUDA="1"
# This will by default build detectron2 for all common cuda architectures and take a lot more time,
# because inside `docker build`, there is no way to tell which architecture will be used.
ARG TORCH_CUDA_ARCH_LIST="Kepler;Kepler+Tesla;Maxwell;Maxwell+Tegra;Pascal;Volta;Turing"
ENV TORCH_CUDA_ARCH_LIST="${TORCH_CUDA_ARCH_LIST}"

RUN pip install --user -e /home/jay/work/detectron2_repo && \
    fix-permissions /home/$NB_USER

# Set a fixed model cache directory.
ENV FVCORE_CACHE="/tmp"

RUN mkdir /home/jay/work/python
WORKDIR /home/jay/work/python

EXPOSE 8888
EXPOSE 6006

RUN python3 -m ipykernel.kernelspec --user && \
    fix-permissions /home/$NB_USER

CMD ["bash", "-c", "jupyter lab --notebook-dir=/home/jay/work --ip 0.0.0.0 --no-browser --allow-root"]

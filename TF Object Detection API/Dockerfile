# System Environment
ARG UBUNTU_VERSION=16.04
ARG ARCH=
ARG CUDA=10.0

# Call Base image and Label image
FROM nvidia/cuda${ARCH:+-$ARCH}:${CUDA}-base-ubuntu${UBUNTU_VERSION} as base
LABEL maintainer="Jayoung Lee(starpic414@gmail.com)"

# Recall ARG Again as FROM command overwrites the ARG
ARG ARCH
ARG CUDA
ARG CUDNN=7.5.0.56-1

# Needed to use Shortcuts and Negliate Errors
SHELL ["/bin/bash", "-c"]
RUN echo 'debconf debconf/frontend select Noninteractive' | debconf-set-selections

# Install apt-utils and Run Upgrade
RUN apt-get update -y && \
    apt-get install -y apt-utils && \
    apt-get upgrade -y

# Grab Some Required Tools
RUN apt-get install -y \
    build-essential \
    cuda-command-line-tools-${CUDA/./-} \
    cuda-cublas-${CUDA/./-} \
    cuda-cufft-${CUDA/./-} \
    cuda-curand-${CUDA/./-} \
    cuda-cusolver-${CUDA/./-} \
    cuda-cusparse-${CUDA/./-} \
    libcudnn7=${CUDNN}+cuda${CUDA} \
    git

RUN apt-get install -y \
    libfreetype6-dev \
    libhdf5-serial-dev \
    libjasper-dev \
    libtiff5-dev \
    libavformat-dev \
    libavcodec-dev \
    libv4l-dev \
    libgtk-3-dev \
    libpng-dev \
    libjpeg-dev \
    libzmq3-dev \
    libopenblas-dev \
    libatlas-base-dev \
    gfortran \
    libswscale-dev \
    curl \
    rsync \
    pkg-config \
    software-properties-common \
    unzip

RUN [ ${ARCH} = ppc64le ] || (apt-get update && \
    apt-get install nvinfer-runtime-trt-repo-ubuntu1604-5.0.2-ga-cuda${CUDA} \
    && apt-get update \
    && apt-get install -y --no-install-recommends libnvinfer5=5.0.2-1+cuda${CUDA} \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*)

ENV LD_LIBRARY_PATH /usr/local/cuda/extras/CUPTI/lib64:$LD_LIBRARY_PATH


# Install Python and Pip with required libraries
ARG USE_PYTHON_3_NOT_2=True
ARG _PY_SUFFIX=${USE_PYTHON_3_NOT_2:+3}
ARG PYTHON=python${_PY_SUFFIX}
ARG PIP=pip${_PY_SUFFIX}
ENV LANG C.UTF-8

RUN apt-get update && apt-get install -y \
    ${PYTHON} \
    ${PYTHON}-pip

RUN ${PIP} --no-cache-dir install --upgrade \
    pip \
    setuptools

RUN pip install keras && \
    pip install pandas && \
    pip install scipy && \
    pip install scikit-learn && \
    pip install pillow && \
    pip install pydot && \
    pip install opencv-contrib-python && \
    pip install Cython && \
    pip install contextlib2

# Set PATH for Python
RUN ln -s $(which ${PYTHON}) /usr/local/bin/python

# Install Tensorflow
ARG TF_PACKAGE=tensorflow-gpu
ARG TF_PACKAGE_VERSION=
RUN ${PIP} install ${TF_PACKAGE}${TF_PACKAGE_VERSION:+==${TF_PACKAGE_VERSION}}

COPY bashrc/.bashrc /etc/bash.bashrc
RUN chmod a+rwx /etc/bash.bashrc

# Install and Set Jupyter Notebook Environment
RUN ${PIP} install jupyter matplotlib
RUN ${PIP} install jupyter_http_over_ws
RUN jupyter serverextension enable --py jupyter_http_over_ws

RUN mkdir -p /tf/tensorflow-tutorials && chmod -R a+rwx /tf/
RUN mkdir /.local && chmod a+rwx /.local
RUN apt-get install -y --no-install-recommends wget
WORKDIR /tf/tensorflow-tutorials
RUN wget https://raw.githubusercontent.com/tensorflow/docs/master/site/en/tutorials/keras/basic_classification.ipynb
RUN wget https://raw.githubusercontent.com/tensorflow/docs/master/site/en/tutorials/keras/basic_text_classification.ipynb
WORKDIR /tf
EXPOSE 8888

RUN ${PYTHON} -m ipykernel.kernelspec

CMD ["bash", "-c", "source /etc/bash.bashrc && jupyter notebook --notebook-dir=/tf --ip 0.0.0.0 --no-browser --allow-root"]

# Install Tex for PDF saving of Jupyter Notebook
RUN apt-get install -y pandoc && \
    apt-get install -y texlive-xetex && \
    apt-get install -y protobuf-compiler python-lxml python-pil

# Install Tensorflow Object Detection API
RUN mkdir -p /tf/models
RUN git clone https://github.com/tensorflow/models.git /tf/models

WORKDIR /tf/models/research

RUN git clone https://github.com/cocodataset/cocoapi.git && \
    cd cocoapi/PythonAPI && \
    make && \
    cp -r pycocotools /tf/models/research

RUN wget -O protobuf.zip https://github.com/google/protobuf/releases/download/v3.0.0/protoc-3.0.0-linux-x86_64.zip && \
    unzip protobuf.zip && \
    apt-get autoremove -y && apt-get remove -y wget

RUN ./bin/protoc object_detection/protos/*.proto --python_out=.

RUN export PYTHONPATH=$PYTHONPATH:`pwd`:`pwd`/slim

# End of Code
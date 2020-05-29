FROM tensorflow/tensorflow:latest-gpu-py3-jupyter
MAINTAINER Jayoung Lee (starpic414@gmail.com)

RUN apt-get -y update

RUN apt-get -y upgrade

RUN apt-get install -y texlive
    
RUN apt-get install -y pandoc && \
    apt-get install -y texlive-xetex

RUN apt-get install -y git && \ 
    apt-get install -y python-pil python-lxml python-tk

RUN pip install keras && \
    pip install pandas && \
    pip install scipy && \
    pip install scikit-learn && \
    pip install pillow

RUN pip install pydot && \
    pip install Cython && \
    pip install contextlib2

RUN mkdir -p /tf/models
RUN git clone https://github.com/tensorflow/models.git /tf/models

WORKDIR /tf/models/research

RUN git clone https://github.com/cocodataset/cocoapi.git && \
    cd cocoapi/PythonAPI && \
    make && \
    cp -r pycocotools /tf/models/research

RUN apt-get install -y wget && \
    wget -O protobuf.zip https://github.com/google/protobuf/releases/download/v3.0.0/protoc-3.0.0-linux-x86_64.zip && \
    unzip protobuf.zip

RUN ./bin/protoc object_detection/protos/*.proto --python_out=.

RUN export PYTHONPATH=$PYTHONPATH:`pwd`:`pwd`/slim


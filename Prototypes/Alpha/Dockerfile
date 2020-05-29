FROM tensorflow/tensorflow:latest-gpu-py3-jupyter
MAINTAINER Jayoung Lee (starpic414@gmail.com)

RUN apt-get -y update

RUN apt-get -y upgrade

RUN apt-get install -y texlive
    
RUN apt-get install -y pandoc && \
    apt-get install -y texlive-xetex

RUN apt-get install -y git

RUN pip install keras && \
    pip install pandas && \
    pip install scipy && \
    pip install scikit-learn && \
    pip install pillow

RUN pip install pydot




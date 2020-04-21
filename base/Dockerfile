#FROM alpine:3.10
FROM ubuntu:20.04

LABEL maintainer="Bobp, based on image from Gezim Sejdiu <g.sejdiu@gmail.com>, Giannis Mouchakis <gmouchakis@gmail.com>"

ENV ENABLE_INIT_DAEMON true
ENV INIT_DAEMON_BASE_URI http://identifier/init-daemon
ENV INIT_DAEMON_STEP spark_master_init

ENV SPARK_VERSION=2.4.5
ENV HADOOP_VERSION=2.7
ENV IPP_LIB_URL=http://enggit.systech-int.local:8088/ENG-SRVACT-UNISECURE/IPPLinuxBinary/raw/master/IPP_libs.tgz \
    LOCAL_LIB_URL=http://enggit.systech-int.local:8088/ENG-SRVACT-UNISECURE/OpenCVLinuxBinaries/raw/master/OpenCV_libs.tgz \
    OPENCV4_LIB_URL=http://enggit.systech-int.local:8088/ENG-SRVACT-UNISECURE/OpenCV4.2LinuxBinaries/raw/master/opencv4.2-1604.tgz 

COPY wait-for-step.sh /
COPY execute-step.sh /
COPY finish-step.sh /
COPY libjasper1_1.900.1-debian1-2.4ubuntu1.2_amd64.deb /tmp
COPY libjasper-dev_1.900.1-debian1-2.4ubuntu1.2_amd64.deb /tmp
COPY libpng12.so.0.54.0 /usr/local/lib/libpng12.so.0

#COPY bde-spark.css /css/org/apache/spark/ui/static/timeline-view.css


RUN  DEBIAN_FRONTEND=noninteractive apt-get --yes update && DEBIAN_FRONTEND=noninteractive apt-get --yes install wget curl bash \
         openjdk-8-jre python3 python3-pip libnss3 \
      && ln -s /lib64/ld-linux-x86-64.so.2 /lib/ld-linux-x86-64.so.2 \
      && chmod +x *.sh \
      && wget https://archive.apache.org/dist/spark/spark-${SPARK_VERSION}/spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz \
      && tar -xvzf spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz \
      && mv spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION} spark \
      && rm spark-${SPARK_VERSION}-bin-hadoop${HADOOP_VERSION}.tgz \
      #&& cd /css \
      #&& jar uf /spark/jars/spark-core_2.11-${SPARK_VERSION}.jar org/apache/spark/ui/static/timeline-view.css \
      && cd /

#Give permission to execute scripts
RUN chmod +x /wait-for-step.sh && chmod +x /execute-step.sh && chmod +x /finish-step.sh

# Fix the value of PYTHONHASHSEED
# Note: this is needed when you use Python 3.3 or greater
ENV PYTHONHASHSEED 1

RUN DEBIAN_FRONTEND=noninteractive apt-get --yes install libqt5core5a \
    libqt5opengl5 \
    libqt5test5 \
    libsasl2-dev \
    libxml2 \
    libxml2-dev \
    libxslt-dev \
    libjpeg-dev\
    libfreetype6 \
    zlib1g-dev\
    libxmlsec1-dev\
    libssl-dev \
    libyaml-dev \
    libopencv-dev\
    pkg-config \
    libtiff5-dev\
    libdc1394-22\
    libavcodec-extra \
    libgstreamer1.0-0\
    libgstreamer-plugins-base1.0-0\
    libgtk2.0\
    unzip \
    bzip2 \
    dos2unix \
    nano \
    vim \ 
    logrotate \
    libjsoncpp-dev

RUN DEBIAN_FRONTEND=noninteractive apt-get --yes install /tmp/libjasper1_1.900.1-debian1-2.4ubuntu1.2_amd64.deb && \
    DEBIAN_FRONTEND=noninteractive apt-get --yes install /tmp/libjasper-dev_1.900.1-debian1-2.4ubuntu1.2_amd64.deb

# Bring in eFingerprint dependencies
ADD $IPP_LIB_URL /var/tmp/local_ipp.tgz
ADD $LOCAL_LIB_URL /var/tmp/local_lib.tgz
ADD $OPENCV4_LIB_URL /var/tmp/opencv4.2-1604.tgz
RUN cd /usr/local && gzip -dc /var/tmp/local_ipp.tgz | tar xvf - 
RUN cd /usr/local && gzip -dc /var/tmp/local_lib.tgz | tar xvf - 
RUN cd /usr/local && gzip -dc /var/tmp/opencv4.2-1604.tgz | tar xvf - 
# Copy 16.04 version of libopencv_highgui.so.2.4.3 from /usr/lib/x86_64-linux-gnu as placed by DockerfileCommon
#RUN cd /usr/local && cp /usr/lib/x86_64-linux-gnu/libopencv_highgui.so.2.4.3 lib && \
#    rm /var/tmp/local_ipp.tgz /var/tmp/local_lib.tgz /var/tmp/opencv4.2-1604.tgz
RUN rm /var/tmp/local_ipp.tgz /var/tmp/local_lib.tgz /var/tmp/opencv4.2-1604.tgz
RUN mkdir -p /opt/intel && ln -s /usr/local/ipp /opt/intel/ipp && mkdir /usr/local/efp


# The newest libsecfgprtL has a dependency on libjson_linux-gcc-5.3.0_libmt.so
# For the sake of expediency, we dump it here and put it in place, rather than picking it up from some build or from
# a repository.
ADD libjson_linux-gcc-5.3.0_libmt.so /usr/local/lib/libjson_linux-gcc-5.3.0_libmt.so
RUN chmod 755 /usr/local/lib/libjson_linux-gcc-5.3.0_libmt.so

# /etc/ld.so.conf.d
COPY opencv.conf /etc/ld.so.conf.d/opencv.conf
COPY ipp.conf /etc/ld.so.conf.d/ipp.conf
RUN dos2unix /etc/ld.so.conf.d/opencv.conf && dos2unix /etc/ld.so.conf.d/ipp.conf && ldconfig

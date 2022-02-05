FROM nvidia/cudagl:11.4.2-devel-ubuntu20.04

ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update && apt-get install -y build-essential cmake git pkg-config libgtk-3-dev libavcodec-dev libavformat-dev libswscale-dev libv4l-dev libxvidcore-dev libx264-dev libjpeg-dev libpng-dev libtiff-dev gfortran openexr libatlas-base-dev python3-dev python3-numpy libtbb2 libtbb-dev libdc1394-22-dev libopenexr-dev libgstreamer-plugins-base1.0-dev libgstreamer1.0-dev libboost-all-dev sudo python3-pip htop vim gdb libssl-dev wget unzip libfmt-dev valgrind

# Optional: install ceres
RUN apt-get install -y liblapack-dev libsuitesparse-dev libcxsparse3 libgflags-dev libgoogle-glog-dev libgtest-dev qt5-qmake qt5-default libqglviewer-dev-qt5 libsuitesparse-dev libcxsparse3 libcholmod3

# create a non-root user
ARG USER_ID=1000
RUN useradd -m --no-log-init --system  --uid ${USER_ID} appuser -g sudo
RUN echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
USER appuser
WORKDIR /home/appuser

# OpenCV
RUN wget https://github.com/opencv/opencv/archive/4.5.4.zip
RUN wget https://github.com/opencv/opencv_contrib/archive/4.5.4.zip -O opencv_contrib-4.5.4.zip
RUN unzip ./4.5.4.zip
RUN unzip ./opencv_contrib-4.5.4.zip
RUN mkdir build && cd build && cmake -DOPENCV_ENABLE_NONFREE=ON -DWITH_TBB=ON -DWITH_QT=ON -DWITH_GTK=ON -DWITH_OPENGL=ON -DBUILD_TIFF=ON -DPYTHON_DEFAULT_EXECUTABLE=$(which python3) -DOPENCV_EXTRA_MODULES_PATH=../opencv_contrib-4.5.4/modules ../opencv-4.5.4/ && make -j10 && sudo make install

# Eigen 
RUN wget https://gitlab.com/libeigen/eigen/-/archive/3.4.0/eigen-3.4.0.zip
RUN unzip eigen-3.4.0.zip
RUN cd eigen-3.4.0 && mkdir build && cd build && cmake .. && sudo make install

# GTSAM
RUN pip install pyparsing
RUN git clone -b 4.1.1 https://github.com/borglab/gtsam.git
RUN cd gtsam && mkdir build && cd build && cmake -DGTSAM_BUILD_PYTHON:OPTION=ON -DGTSAM_USE_SYSTEM_EIGEN:OPTION=ON -DCMAKE_BUILD_TYPE=RelWithDebInfo -DGTSAM_WITH_TBB:OPTION=OFF DGTSAM_BUILD_WITH_MARCH_NATIVE:OPTION=ON .. && make -j10 && sudo make install

# Sophus
RUN git clone https://github.com/strasdat/Sophus.git
RUN cd Sophus && git checkout 36b0888 && mkdir build && cd build && cmake .. && make -j10 && sudo make install

# Ceres
RUN git clone https://github.com/ceres-solver/ceres-solver.git
RUN cd ceres-solver && git checkout 7d6524d && mkdir build && cd build && cmake .. && make -j10 && sudo make install

# G2O
RUN git clone https://github.com/RainerKuemmerle/g2o.git
RUN cd g2o && git checkout 73b2f2c && mkdir build && cd build && cmake .. && make -j10 && sudo make install

# Pangolin
RUN git clone --recursive https://github.com/stevenlovegrove/Pangolin.git
RUN cd Pangolin && git checkout bcf41ec && sed -i "s/libeigen3-dev//g" scripts/install_prerequisites.sh && sed -i "s/libc++-dev//g" scripts/install_prerequisites.sh && yes | ./scripts/install_prerequisites.sh recommended && mkdir build && cd build && cmake -DPYTHON_EXECUTABLE=`which python3` .. && cmake --build . -t pypangolin_pip_install && sudo make install

# Optional: Intel Inspector
# download the key to system keyring
RUN wget -O- https://apt.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB | gpg --dearmor | sudo tee /usr/share/keyrings/oneapi-archive-keyring.gpg > /dev/null
# add signed entry to apt sources and configure the APT client to use Intel repository:
RUN echo "deb [signed-by=/usr/share/keyrings/oneapi-archive-keyring.gpg] https://apt.repos.intel.com/oneapi all main" | sudo tee /etc/apt/sources.list.d/oneAPI.list
#RUN sudo apt-get update && sudo apt-get install -y intel-hpckit

RUN echo "LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib" >> ~/.bashrc

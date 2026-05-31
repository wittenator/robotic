# Target architecture: x86_64 (default) or aarch64.
# Used for the base image, the CMake download, and matches `uname -m`.
ARG ARCH=x86_64
FROM quay.io/pypa/manylinux_2_28_${ARCH}
# re-declare so ${ARCH} is usable in RUN steps below the FROM
ARG ARCH

ENV DEBIAN_FRONTEND=noninteractive
ENV HOME="/root"
ENV TERM="xterm"
ENV MAKEFLAGS="-j$(command nproc --ignore 1)"

##########################################################

RUN dnf -y install gcc wget \
    atlas-devel zlib-devel \
    mesa-libGLU-devel libXcursor-devel libXi-devel libXinerama-devel libXrandr-devel \
    clang lapack-static atlas-static \
    glm-devel freetype-devel f2c-devel cmake
    
RUN cd /usr/lib64; ln -s libstdc++.so.6.0.19 libstdc++.so

RUN wget https://github.com/MarcToussaint/rai/raw/master/_make/install.sh; chmod a+x install.sh

# (some of these we compile manually just to ensure the -fPIC, which is missing in yum packages)
RUN ./install.sh -gc jsoncpp
RUN ./install.sh -gc libyaml
RUN ./install.sh -gc libpng
RUN ./install.sh -gc hdf5
RUN ./install.sh -gc freeglut
RUN ./install.sh -gc glew
RUN ./install.sh -gc glfw

RUN ./install.sh -gc eigen
RUN ./install.sh -gc libann
RUN ./install.sh -gc qhull
RUN ./install.sh -gc libccd
RUN ./install.sh -gc fcl
RUN ./install.sh -gc assimp

#RUN ./install.sh -gc bullet
RUN ./install.sh -gc physx

#RUN yum -y install openssl-devel poco-devel libusbx-devel
#RUN ./install.sh -g librealsense
#RUN ./install.sh -g libfranka

RUN python3.10 -m pip install pybind11 pybind11-stubgen numpy
RUN python3.11 -m pip install pybind11 pybind11-stubgen numpy
RUN python3.12 -m pip install pybind11 pybind11-stubgen numpy
RUN python3.13 -m pip install pybind11 pybind11-stubgen numpy
RUN python3.14 -m pip install pybind11 pybind11-stubgen numpy

##########################################################

RUN rm -Rf ${HOME}/git

##########################################################

WORKDIR /root

RUN printf "\necho '*** Robotic Python Lib - docker for wheel build ***'\n" >> $HOME/.bashrc

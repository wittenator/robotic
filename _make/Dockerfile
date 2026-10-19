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

RUN dnf -y install epel-release && \
    dnf -y install --enablerepo=powertools \
        gcc gcc-gfortran wget clang \
        zlib-devel freetype-devel \
        mesa-libGLU-devel \
        libXcursor-devel libXi-devel libXinerama-devel libXrandr-devel libXxf86vm-devel \
        glm-devel \
        openblas-devel && \
    wget https://www.netlib.org/f2c/f2c.h -O /usr/local/include/f2c.h
    
# install old cmake to stay consistent with hitherto used library versions
RUN cd /tmp && wget https://github.com/Kitware/CMake/releases/download/v3.31.9/cmake-3.31.9-linux-${ARCH}.sh
RUN sh /tmp/cmake-3.31.9-linux-${ARCH}.sh --prefix=/usr/local --exclude-subdir --skip-license

RUN wget https://github.com/MarcToussaint/rai/raw/master/_make/install.sh; chmod a+x install.sh
# assimp v5.2.5 doesn't compile cleanly under the newer GCC in the base image
RUN sed -i 's/-DASSIMP_BUILD_TESTS=OFF/-DASSIMP_BUILD_TESTS=OFF -DASSIMP_WARNINGS_AS_ERRORS=OFF/' install.sh
# hdf5 1.10.4 predates GCC 14, which turned implicit declarations and incompatible pointer assignments into hard errors.
RUN sed -i 's|env CFLAGS="-fPIC" CXXFLAGS="-fPIC" ./configure|env CFLAGS="-fPIC -Wno-error=implicit-function-declaration -Wno-error=incompatible-pointer-types" CXXFLAGS="-fPIC" ./configure|' install.sh
# libANN's repo ships prebuilt x86-64 .o files, and a "C++=g++" make argument does not reach $(C++). So without this the archive is just those stale x86 objects repackaged.
RUN sed -i 's|make -C ${lib}/src targets "ANNLIB=libANN.a" "C++=g++"|rm -f ${lib}/src/*.o; env "C++=g++" make -C ${lib}/src targets "ANNLIB=libANN.a"|' install.sh

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

RUN python3.10 -m pip install pybind11 pybind11-stubgen numpy wheel
RUN python3.11 -m pip install pybind11 pybind11-stubgen numpy wheel
RUN python3.12 -m pip install pybind11 pybind11-stubgen numpy wheel
RUN python3.13 -m pip install pybind11 pybind11-stubgen numpy wheel
RUN python3.14 -m pip install pybind11 pybind11-stubgen numpy wheel

##########################################################

RUN rm -Rf ${HOME}/git

##########################################################

WORKDIR /root

RUN printf "\necho '*** Robotic Python Lib - docker for wheel build ***'\n" >> $HOME/.bashrc

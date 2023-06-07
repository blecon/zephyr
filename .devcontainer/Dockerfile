# See here for image contents: https://github.com/devcontainers/images/tree/main/src/cpp
ARG VARIANT="ubuntu-22.04"

FROM mcr.microsoft.com/devcontainers/cpp:0-${VARIANT}

# All ARGs are collected with each FROM command, so this needs to be below
ARG REINSTALL_CMAKE_VERSION_FROM_SOURCE="3.25.1"

# Install additional packages.
RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
     && apt-get -y install --no-install-recommends git cmake ninja-build gperf \
  ccache dfu-util device-tree-compiler wget \
  python3-dev python3-pip python3-setuptools python3-tk python3-wheel python3-venv xz-utils file \
  make gcc libsdl2-dev libmagic1 build-essential libc6-dev

# Reinstall a modern CMake version
COPY ./reinstall-cmake.sh /tmp/
RUN if [ "${REINSTALL_CMAKE_VERSION_FROM_SOURCE}" != "none" ]; then \
        chmod +x /tmp/reinstall-cmake.sh && /tmp/reinstall-cmake.sh ${REINSTALL_CMAKE_VERSION_FROM_SOURCE}; \
    fi \
    && rm -f /tmp/reinstall-cmake.sh

# Install nanopb generator dependencies
RUN pip3 install protobuf grpcio-tools

# Install west
RUN pip3 install west

WORKDIR /opt

# Install Zephyr toolchains
RUN ARCH=`uname -m` \
    && SDK_VERSION=0.16.1 \
    && SDK_MINIMAL_BUNDLE=zephyr-sdk-${SDK_VERSION}_linux-${ARCH}_minimal.tar.xz \
    && wget -q --show-progress --progress=dot:giga -N -O ${SDK_MINIMAL_BUNDLE}  https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v${SDK_VERSION}/${SDK_MINIMAL_BUNDLE} \
    && tar -xJf ${SDK_MINIMAL_BUNDLE} \
    && rm -f ${SDK_MINIMAL_BUNDLE} \
    && cd zephyr-sdk-${SDK_VERSION} \
    && ./setup.sh -t arm-zephyr-eabi -h -c

# Install Zephyr Python dependencies
RUN pip3 install \
		-r https://raw.githubusercontent.com/zephyrproject-rtos/zephyr/main/scripts/requirements.txt

# Install BabbleSim dependencies
RUN export DEBIAN_FRONTEND=noninteractive \
     && apt-get -y install --no-install-recommends repo gcc-multilib libfftw3-dev

# Fetch BabbleSim
WORKDIR /opt/bsim/

RUN repo init -u https://github.com/BabbleSim/manifest.git -m everything.xml -b master \
    && repo sync

# Install BabbleSim
RUN make everything -s -j 8

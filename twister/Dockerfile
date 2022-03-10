FROM ubuntu:20.04

ENV DEBIAN_FRONTEND noninteractive

RUN rm /bin/sh && ln -s /bin/bash /bin/sh

RUN apt-get -y update && \
    apt-get -y upgrade && \
    apt-get install --no-install-recommends -y wget ca-certificates && \
    wget --no-check-certificate https://apt.kitware.com/kitware-archive.sh && \
    bash kitware-archive.sh && \
    apt-get install --no-install-recommends -y git cmake ninja-build gperf \
      ccache dfu-util device-tree-compiler wget \
      python3-dev python3-pip python3-setuptools python3-tk python3-wheel xz-utils file \
      make gcc

# Install multi-lib gcc (x86 only)
RUN if [ "${HOSTTYPE}" = "x86_64" ]; then \
        apt-get install --no-install-recommends -y \
                gcc-multilib \
                g++-multilib \
        ; fi

# Install i386 packages (x86 only)
RUN if [ "${HOSTTYPE}" = "x86_64" ]; then \
        dpkg --add-architecture i386 && \
        apt-get -y update && \
        apt-get -y upgrade && \
        apt-get install --no-install-recommends -y \
                libsdl2-dev:i386 \
        ; fi

RUN cd ~ && \
    wget https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v0.13.1/zephyr-sdk-0.13.1-linux-x86_64-setup.run && \
    chmod +x zephyr-sdk-0.13.1-linux-x86_64-setup.run && \
    ./zephyr-sdk-0.13.1-linux-x86_64-setup.run -- -d ~/zephyr-sdk-0.13.1 && \
    rm zephyr-sdk-0.13.1-linux-x86_64-setup.run

RUN pip3 install -U west && \
    echo 'export PATH=~/.local/bin:"$PATH"' >> ~/.bashrc && \
    . ~/.bashrc && \
    west init /root/zephyrproject && \
    cd /root/zephyrproject && \
    west update && \
    west zephyr-export

RUN cd /root/zephyrproject/zephyr && \
    git checkout v3.0-branch && \
    pip3 install -r scripts/requirements.txt

COPY 0001-twister-hardcode-flash-command-for-96b_nitrogen.patch /root/
RUN cd /root/zephyrproject/zephyr && \
    git config --global user.email "chase.qi@linaro.org" && \
    git config --global user.name "Chase Qi" && \
    git am /root/0001-twister-hardcode-flash-command-for-96b_nitrogen.patch

WORKDIR /root/zephyrproject/zephyr/

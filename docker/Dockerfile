# CUDA (from [3]) ################################
ARG UBUNTU_VERSION=20.04

ARG ARCH=
ARG CUDA=11.3
FROM nvidia/cuda${ARCH:+-$ARCH}:${CUDA}.1-base-ubuntu${UBUNTU_VERSION} as base

ENV TZ=Asia/Tokyo
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

###############################################
#------------------------------------------------
#User Setting
#------------------------------------------------
ARG SUDO_VERSION=*

#install sudo
RUN apt-get update && apt-get install -y --no-install-recommends \
    sudo=${SUDO_VERSION} \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Root user setting
ENV ROOT_USER=root
ARG ROOT_USER_PASSWORD=root

# root password is root
RUN echo ${ROOT_USER}:${ROOT_USER_PASSWORD} | chpasswd
WORKDIR /root

#Ubuntu packages
RUN apt-get update -q \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y \
    wget \
    git \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

#Add default user
ENV DEFAULT_USER=docker
ENV DEFAULT_USER_HOME=/home/${DEFAULT_USER}
ARG DEFAULT_USER_PASSWORD=password
ARG DEFAULT_USER_TEMP_ID=9999
ARG DEFAULT_USER_GROUP_NAME=docker
ARG DEFAULT_USER_GROUP_TEMP_ID=65534

RUN groupadd -g ${DEFAULT_USER_GROUP_TEMP_ID} ${DEFAULT_USER_GROUP_NAME} && \
    useradd -m -s /bin/bash -u ${DEFAULT_USER_TEMP_ID} -g ${DEFAULT_USER_GROUP_TEMP_ID} -G sudo ${DEFAULT_USER} && \
    echo ${DEFAULT_USER}:${DEFAULT_USER_PASSWORD} | chpasswd && \
    echo "${DEFAULT_USER}   ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

USER ${DEFAULT_USER_TEMP_ID}
WORKDIR ${DEFAULT_USER_HOME}
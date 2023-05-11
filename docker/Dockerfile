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
ENV ROOT_HOME=/${ROOT_USER}

RUN echo ${ROOT_USER}:${ROOT_USER_PASSWORD} | chpasswd
WORKDIR /root/

#Ubuntu packages
RUN apt-get update -q \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y \
    wget \
    git \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*


# zsh (from [1]) for root----------------
RUN apt-get update -q \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y \
    wget \
    git \
    zsh \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

SHELL ["/bin/zsh", "-c"]
RUN wget http://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh -O - | zsh

# # pyenv (from [2]) ----------------
ENV DEBIAN_FRONTEND=noninteractive
RUN sudo apt-get update && sudo apt-get install -y \
    build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev \
    libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev \
    xz-utils tk-dev libffi-dev liblzma-dev python-openssl git

RUN curl https://pyenv.run | zsh \
    && echo 'export PATH="${ROOT_HOME}/.pyenv/bin:$PATH"' >> ${ROOT_HOME}/.zshrc \
    && echo 'eval "$(pyenv init -)"' >> ${ROOT_HOME}/.zshrc \
    && echo 'eval "$(pyenv virtualenv-init -)"' >> ${ROOT_HOME}/.zshrc \
    && source ${ROOT_HOME}/.zshrc

RUN source ${ROOT_HOME}/.zshrc && \
    pyenv install 3.8.0 && \
    pyenv global 3.8.0 && \
    pip install -U pip

# X window ----------------
RUN sudo apt-get update && sudo apt-get install -y \
    xvfb x11vnc python-opengl icewm \
    && sudo apt-get clean \
    && sudo rm -rf /var/lib/apt/lists/*


# # utils ----------------
RUN sudo apt-get update -q \
    && DEBIAN_FRONTEND=noninteractive sudo apt-get install -y \
    curl \
    git \
    unzip \
    vim \
    wget \
    tmux \
    && sudo apt-get clean \
    && sudo rm -rf /var/lib/apt/lists/*

COPY .tmux.conf ${ROOT_HOME}/

# DL ----------------
RUN . ${ROOT_HOME}/.zshrc && \
    pip install \
    setuptools \
    jupyterlab \
    widgetsnbextension && \
    echo 'alias jl="DISPLAY=:0 jupyter lab --ip 0.0.0.0 --port 8888 --allow-root &"' >> ${ROOT_HOME}/.bash && \
    echo 'alias tb="tensorboard --logdir runs --bind_all &"' >> ${ROOT_HOME}/.bash

#pytorch install ----------------
RUN . ${ROOT_HOME}/.zshrc && \
    pip install torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/cu113

#experiment management ----------------
RUN . ${ROOT_HOME}/.zshrc && \
    pip install \
    mlflow \
    hydra-core \
    wandb \
    joblib \
    scikit-image

#Install pip install -r requirements.txt
COPY requirements.txt ${ROOT_HOME}/

RUN source ${ROOT_HOME}/.zshrc && \
    pip install -r requirements.txt

RUN sudo apt-get clean && \
    sudo rm -rf /var/lib/apt/lists/*

COPY start.sh ${ROOT_HOME}

RUN echo 'Xvfb :0 -screen 0 1400x900x24 & ' >> ${ROOT_HOME}/Xvfb-run.sh && \
    echo 'x11vnc -display :0 -passwd pass -forever &' >> ${ROOT_HOME}/run-Xvfb.sh && \
    chmod +x ${ROOT_HOME}/run-Xvfb.sh

RUN echo 'DISPLAY=:0 jupyter notebook --allow-root --ip=0.0.0.0 --port 8888 --notebook-dir=${ROOT_HOME} --NotebookApp.token="password" &' >> ${ROOT_HOME}/run-jupyter.sh && \
    chmod +x ${ROOT_HOME}/run-jupyter.sh



# auto start tmux and zsh
ENTRYPOINT tmux new \; \
            send-keys 'Xvfb :0 -screen 0 1400x900x24 & ' Enter \; \
	    send-keys 'x11vnc -display :0 -passwd 0123 -forever &' Enter \; \
            split-window -v  \; \
            send-keys "zsh" Enter \; \
            send-keys "jupyter nbextension enable --py widgetsnbextension --sys-prefix" Enter \; \
            send-keys "bash ${ROOT_HOME}/run-jupyter.sh" Enter \; \
	    new-window \; \
            send-keys "zsh" Enter \; \
    	    send-keys clear C-m \;
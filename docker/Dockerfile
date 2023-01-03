# base image
ARG REPO_LOCATION=""
FROM ${REPO_LOCATION}ubuntu:18.04

# user
ARG USER_NAME=edgeai
ARG USER_ID=1000
ARG USER_GID=$USER_ID
ENV HOME_DIR=/home/$USER_NAME
ARG PROJECT_NAME="modelmaker"

# paths
ENV APPS_PATH=${HOME_DIR}/apps
ENV DOWNLOADS_PATH=${HOME_DIR}/downloads
ENV CODE_PATH=${HOME_DIR}/code

# proxy
ARG PROXY_LOCATION=""
ENV DEBIAN_FRONTEND=noninteractive
ENV http_proxy=${PROXY_LOCATION}
ENV https_proxy=${PROXY_LOCATION}
ENV no_proxy=ti.com
RUN if [ ! -z $PROXY_LOCATION ]; then echo "Acquire::http::proxy \"${PROXY_LOCATION}\";" > /etc/apt/apt.conf; fi && \
    if [ ! -z $PROXY_LOCATION ]; then echo "Acquire::https::proxy \"${PROXY_LOCATION}\";" >> /etc/apt/apt.conf; fi

# baseline
RUN apt update && apt install -y sudo git iputils-ping wget cmake build-essential libgtk2.0 apt-utils

# lang
RUN echo >> /etc/environment
RUN echo "LC_CTYPE=en_US.UTF-8" >> /etc/environment
RUN echo "LANG=en_US.UTF-8" >> /etc/environment
RUN echo "LANGUAGE=en_US.UTF-8" >> /etc/environment
RUN echo "LC_ALL=en_US.UTF-8" >> /etc/environment
RUN apt update && apt install -y locales locales-all && locale-gen en_US.UTF-8 && dpkg-reconfigure locales

# additional packages
RUN apt install -y curl libjpeg-dev zlib1g-dev graphviz graphviz-dev libbz2-dev \
                   build-essential xz-utils libreadline-dev libncurses5-dev libssl-dev \
                   libsqlite3-dev liblzma-dev

# add user, inspired by: https://code.visualstudio.com/remote/advancedcontainers/add-nonroot-user
RUN groupadd --gid $USER_GID $USER_NAME && \
    useradd --uid $USER_ID --gid $USER_GID --create-home $USER_NAME && \
    echo $USER_NAME ALL=\(root\) NOPASSWD:ALL >> /etc/sudoers.d/$USER_NAME && \
    chmod 400 /etc/sudoers.d/$USER_NAME

# Change working directory permission to non-root user
RUN chown -R ${USER_ID}:${USER_GID} ${HOME_DIR}

# switch user, workdir, default permissions
USER $USER_NAME
WORKDIR ${HOME_DIR}
RUN echo "umask u=rwx,g=rwx,o=rx" >> ${HOME_DIR}/.bashrc

# pyenv install
ENV PYENV_ROOT $HOME_DIR/.pyenv
RUN mkdir ${DOWNLOADS_PATH} && \
    curl -L https://github.com/pyenv/pyenv-installer/raw/master/bin/pyenv-installer | bash

ENV PATH :$PYENV_ROOT/bin:$PATH
RUN eval "$($PYENV_ROOT/bin/pyenv init --path)" && \
    eval "$($PYENV_ROOT/bin/pyenv virtualenv-init -)" && \
    $PYENV_ROOT/bin/pyenv install 3.6 && \
    $PYENV_ROOT/bin/pyenv global 3.6 && \
    $PYENV_ROOT/bin/pyenv rehash

RUN $PYENV_ROOT/shims/pip install --upgrade pip
RUN $PYENV_ROOT/shims/pip install --upgrade setuptools

RUN echo "export PATH=:$PYENV_ROOT/bin:$PATH" >> ${HOME_DIR}/.bashrc && \
    echo "eval $(pyenv init --path)" >> ${HOME_DIR}/.bashrc && \
    echo "eval $(pyenv virtualenv-init -)" >> ${HOME_DIR}/.bashrc && \
    echo " " >> ${HOME_DIR}/.bashrc
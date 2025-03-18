FROM nvcr.io/nvidia/pytorch:24.11-py3

RUN apt-get update && \
    DEBIAN_FRONTEND="noninteractive" apt-get install --yes \
    bash \
    build-essential \
    htop \
    locales \
    man \
    python3 \
    python3-pip \
    software-properties-common \
    sudo \
    systemd \
    systemd-sysv \
    tmux \
    unzip tar \
    vim nano \
    tzdata \
    wget curl \
    fzf \
    openssh-server \
    git

# Set locales
RUN locale-gen en_US.UTF-8

# Install Node.JS & Global dependencies
RUN mkdir -p /usr/local/nvm
ENV NVM_DIR /usr/local/nvm
ENV NODE_VERSION v22.14.0

RUN curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
RUN /bin/bash -c "source $NVM_DIR/nvm.sh && nvm install $NODE_VERSION && nvm use --delete-prefix $NODE_VERSION"

ENV NODE_PATH $NVM_DIR/versions/node/$NODE_VERSION/bin
ENV PATH $NODE_PATH:$PATH
RUN npm install -g yarn http-server
RUN ln -sf $NODE_PATH/node /usr/bin/node
RUN ln -sf $NODE_PATH/npm /usr/bin/npm
RUN ln -sf $NODE_PATH/yarn /usr/bin/yarn

# IntelliJ IDEA & PyCharm Setting
WORKDIR /etc/jetbrains
RUN wget -O idea.tar.gz https://download.jetbrains.com/idea/ideaIU-2024.3.4.tar.gz && \
    tar -xzvf idea.tar.gz && \
    rm idea.tar.gz && \
    sh $(find ./ -maxdepth 1 -name "idea*")/bin/remote-dev-server.sh registerBackendLocationForGateway && \
    sed -i  "s/-Xmx[0-9]\+m/-Xmx8192m/g" $(find $(find ./ -maxdepth 1 -name "idea*")/bin/ -name "*.vmoptions")
# sh $(find ./ -maxdepth 1 -name "idea*")/bin/remote-dev-server.sh installPlugins /usr/src 18824-codeglance-pro

RUN wget -O pycharm.tar.gz https://download.jetbrains.com/python/pycharm-professional-2024.3.4.tar.gz && \
    tar -xzvf pycharm.tar.gz && \
    rm pycharm.tar.gz && \
    sh $(find ./ -maxdepth 1 -name "pycharm*")/bin/remote-dev-server.sh registerBackendLocationForGateway && \
    sed -i  "s/-Xmx[0-9]\+m/-Xmx8192m/g" $(find $(find ./ -maxdepth 1 -name "pycharm*")/bin/ -name "*.vmoptions")

# Install code server
WORKDIR /root
RUN curl -fsSL https://code-server.dev/install.sh | sh | tee code-server-install.log

# Set PyTorch and OpenCV environment variables
ENV TORCH_CUDA_ARCH_LIST="8.6"
ENV OPENCV_IO_ENABLE_OPENEXR="1"

# Install Python and PyTorch packages
RUN pip install --upgrade pip && \
    pip install kornia matplotlib opencv-python pandas pytorch-ignite pytorch-lightning ray scikit-image tensorboard torchmetrics torchsummary tqdm

# SSH server setting
RUN sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config

# Entrypoint script
RUN printf "#!/bin/bash\n" >> /usr/sbin/startup && \
    printf "#!/bin/bash\nservice ssh start\n/bin/bash /usr/sbin/startup" >> /usr/sbin/entrypoint

# root password setting
RUN echo "root:password" | chpasswd

CMD ["/bin/bash", "-c" , "/bin/bash /usr/sbin/entrypoint && tail -f /dev/null"]
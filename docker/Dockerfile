# Base image
FROM nvidia/cudagl:11.3.0-devel-ubuntu20.04
LABEL authors="walfrido"

# Configurar zona horaria y prevenir interacciones
ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=Etc/UTC

# Setup basic packages
RUN apt-get update && apt-get install -y --no-install-recommends \
    tzdata \
    build-essential \
    git \
    curl \
    vim \
    ca-certificates \
    libjpeg-dev \
    libpng-dev \
    libglfw3-dev \
    libglm-dev \
    libx11-dev \
    libomp-dev \
    libegl1-mesa-dev \
    libassimp-dev \
    libsdl2-dev \
    ffmpeg \
    pkg-config \
    wget \
    net-tools \
    zip \
    unzip &&\
    rm -rf /var/lib/apt/lists/*

# Configurar la zona horaria sin interacción
RUN ln -fs /usr/share/zoneinfo/$TZ /etc/localtime && \
    dpkg-reconfigure --frontend noninteractive tzdata

# Install conda
RUN curl -L -o ~/miniconda.sh https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh &&\
    chmod +x ~/miniconda.sh &&\
    ~/miniconda.sh -b -p /opt/conda &&\
    rm ~/miniconda.sh &&\
    /opt/conda/bin/conda install numpy pyyaml scipy ipython mkl mkl-include &&\
    /opt/conda/bin/conda clean -ya

# Add conda to path
ENV PATH=/opt/conda/bin:$PATH

# Crear usuario con UID definidos
RUN groupadd -g 1007 walfrido
RUN useradd -u 1006 -g 1007 --create-home walfrido
WORKDIR /home/walfrido
RUN mkdir code/
USER walfrido

# Conda environment
RUN conda create -n habitat python=3.9 cmake=3.22.1

# Setup habitat-sim
RUN git clone --branch v0.3.1 https://github.com/facebookresearch/habitat-sim.git
RUN /bin/bash -c ". activate habitat; cd habitat-sim; pip install -r requirements.txt; python setup.py install --headless --bullet"

# Install specific habitat-lab
RUN git clone --branch stable https://github.com/facebookresearch/habitat-lab.git
RUN /bin/bash -c ". activate habitat; cd habitat-lab; pip install -e habitat-lab/; pip install -e habitat-baselines/"

# Install wandb
RUN /bin/bash -c ". activate habitat; pip install wandb"

# Silence habitat-sim logs
ENV GLOG_minloglevel=2
ENV MAGNUM_LOG="quiet"
ENV HABITAT_SIM_LOG=quiet

# Copiar y ejecutar entrypoint
COPY entrypoint.sh /home/walfrido
ENTRYPOINT [ "/home/walfrido/entrypoint.sh" ]
